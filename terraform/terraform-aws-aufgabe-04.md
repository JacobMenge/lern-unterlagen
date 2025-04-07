# Terraform Module Mini-Challenge: Erweitere deinen Webserver

## Einführung

Hier ist eine kleine zusätzliche Challenge, um dein neues Wissen anzuwenden und zu vertiefen. Diese Aufgabe baut direkt auf der bereits durchgeführten Übung auf. Erledige die Übung gerne für dich oder reiche sie in dem google-form als Upload ein! :) 

## Was du lernst

Mit dieser Mini-Challenge vertiefst du:
- Die Verwendung des `count`-Parameters für mehrere ähnliche Ressourcen
- Die Anpassung von Outputs für Listen von Ressourcen
- Die Validierung von Input-Variablen
- Die dynamische Generierung von Ressourcen-Namen
- Die Verwendung von Data Sources für aktuelle AMIs

Viel Erfolg mit der Challenge! Falls du Fragen hast, zögere nicht, sie zu stellen.

## Die Challenge

Erweitere das bestehende Webserver-Modul aus der Übung um folgende Funktionen:

1. Eine Variable zur Steuerung der Anzahl von Webservern (1-3)
2. Eine einfache benutzerdefinierte Startseite mit unterschiedlichem Inhalt je nach Umgebung

## Voraussetzungen

- Du hast die grundlegende Terraform-Modul-Übung bereits durchgeführt
- Die Projektstruktur aus der Übung ist noch vorhanden

## Schritt 1: Erweiterung des Webserver-Moduls

### Schritt 1.1: variables.tf erweitern

Öffne die Datei `modules/webserver/variables.tf` und füge folgende Variablen hinzu:

```hcl
variable "instance_count" {
  description = "Anzahl der zu erstellenden Webserver-Instanzen"
  type        = number
  default     = 1
  
  validation {
    condition     = var.instance_count >= 1 && var.instance_count <= 3
    error_message = "Die Anzahl der Instanzen muss zwischen 1 und 3 liegen."
  }
}

variable "webserver_content" {
  description = "HTML-Inhalt für die Webserver-Startseite"
  type        = string
  default     = "<h1>Willkommen auf dem Webserver!</h1>"
}
```

### Schritt 1.2: main.tf anpassen

Ändere in der Datei `modules/webserver/main.tf` nur die EC2-Instance-Ressource, um mehrere Instanzen zu unterstützen. Die Security Group bleibt unverändert:

```hcl
resource "aws_instance" "web" {
  count                  = var.instance_count
  ami                    = var.ami_id
  instance_type          = var.instance_type
  subnet_id              = var.subnet_id
  vpc_security_group_ids = [aws_security_group.web.id]
  
  # Webserver mit benutzerdefiniertem Inhalt
  user_data = <<-EOF
              #!/bin/bash
              yum update -y
              yum install -y httpd
              systemctl start httpd
              systemctl enable httpd
              
              cat > /var/www/html/index.html << 'HTMLCONTENT'
              ${var.webserver_content}
              <p>Server Nummer ${count.index + 1} in der ${var.environment}-Umgebung</p>
              <p>Erstellt am: $(date)</p>
              HTMLCONTENT
              EOF

  tags = {
    Name        = "webserver-${var.environment}-${count.index + 1}"
    Environment = var.environment
  }
}
```

### Schritt 1.3: outputs.tf anpassen

Update die Datei `modules/webserver/outputs.tf`, um mit mehreren Instanzen umzugehen:

```hcl
output "instance_ids" {
  description = "IDs der erstellten EC2-Instances"
  value       = aws_instance.web[*].id
}

output "public_ips" {
  description = "Öffentliche IP-Adressen der EC2-Instances"
  value       = aws_instance.web[*].public_ip
}
```

## Schritt 2: Entwicklungsumgebung anpassen

Passe die Datei `environments/dev/main.tf` an:

```hcl
provider "aws" {
  region = "eu-central-1"
}

# Neuestes Amazon Linux 2 AMI finden (statt hardcoded AMI)
data "aws_ami" "amazon_linux" {
  most_recent = true
  owners      = ["amazon"]

  filter {
    name   = "name"
    values = ["amzn2-ami-hvm-*-x86_64-gp2"]
  }
}

# Zugriff auf die gemeinsame VPC
data "terraform_remote_state" "shared" {
  backend = "local"
  config = {
    path = "../shared/terraform.tfstate"
  }
}

# Verwendung des Webserver-Moduls
module "dev_webserver" {
  source = "../../modules/webserver"
  
  environment   = "dev"
  instance_type = "t2.micro"
  ami_id        = data.aws_ami.amazon_linux.id
  vpc_id        = data.terraform_remote_state.shared.outputs.vpc_id
  subnet_id     = data.terraform_remote_state.shared.outputs.subnet_id
  
  # Neue Parameter
  instance_count   = 2  # Zwei Instanzen für Entwicklung
  webserver_content = <<-HTML
    <!DOCTYPE html>
    <html>
    <head>
      <title>Dev Webserver</title>
      <style>
        body { font-family: Arial, sans-serif; background-color: #f0f8ff; padding: 20px; }
        h1 { color: #0066cc; }
      </style>
    </head>
    <body>
      <h1>Entwicklungsumgebung</h1>
      <p>Dies ist ein Testserver für die Entwicklung.</p>
    </body>
    </html>
  HTML
}

# Outputs
output "dev_instance_ips" {
  value = module.dev_webserver.public_ips
}
```

## Schritt 3: Produktionsumgebung anpassen

Passe die Datei `environments/prod/main.tf` an:

```hcl
provider "aws" {
  region = "eu-central-1"
}

# Neuestes Amazon Linux 2 AMI finden (statt hardcoded AMI)
data "aws_ami" "amazon_linux" {
  most_recent = true
  owners      = ["amazon"]

  filter {
    name   = "name"
    values = ["amzn2-ami-hvm-*-x86_64-gp2"]
  }
}

# Zugriff auf die gemeinsame VPC
data "terraform_remote_state" "shared" {
  backend = "local"
  config = {
    path = "../shared/terraform.tfstate"
  }
}

# Verwendung des Webserver-Moduls
module "prod_webserver" {
  source = "../../modules/webserver"
  
  environment   = "prod"
  instance_type = "t3a.micro"
  ami_id        = data.aws_ami.amazon_linux.id
  vpc_id        = data.terraform_remote_state.shared.outputs.vpc_id
  subnet_id     = data.terraform_remote_state.shared.outputs.subnet_id
  
  # Neue Parameter
  instance_count   = 3  # Drei Instanzen für Produktion (mehr Kapazität)
  webserver_content = <<-HTML
    <!DOCTYPE html>
    <html>
    <head>
      <title>Production Webserver</title>
      <style>
        body { font-family: Arial, sans-serif; background-color: #f5f5f5; padding: 20px; }
        h1 { color: #006600; }
        .prod-badge { 
          background-color: #006600; 
          color: white; 
          padding: 5px 10px; 
          border-radius: 5px; 
          display: inline-block; 
        }
      </style>
    </head>
    <body>
      <h1>Produktionsumgebung</h1>
      <div class="prod-badge">PRODUKTION</div>
      <p>Dies ist ein Live-Server der Produktionsumgebung.</p>
    </body>
    </html>
  HTML
}

# Outputs
output "prod_instance_ips" {
  value = module.prod_webserver.public_ips
}
```

## Schritt 4: Testen der Änderungen

### 4.1: Die gemeinsame Infrastruktur sollte bereits vorhanden sein

Wenn du die ursprüngliche Übung abgeschlossen hast, sollte die gemeinsame VPC bereits existieren. Falls nicht, führe diese Befehle aus:

```bash
cd environments/shared
terraform init
terraform apply
```

### 4.2: Aktualisiere die Entwicklungsumgebung

```bash
cd ../dev
terraform init
terraform plan
```

Überprüfe den Plan, um sicherzustellen, dass alles wie erwartet aussieht. Du solltest sehen, dass Terraform zwei Instances erstellen will.

```bash
terraform apply
```

### 4.3: Teste die Webserver

Öffne einen Browser und besuche die IP-Adressen der erstellten Server:

```
http://<dev_instance_ip_1>
http://<dev_instance_ip_2>
```

Du solltest die benutzerdefinierte Webseite mit der Information über die Entwicklungsumgebung sehen.

### 4.4: Aktualisiere optional die Produktionsumgebung

```bash
cd ../prod
terraform init
terraform plan
terraform apply
```

## Schritt 5: Aufräumen nicht vergessen!

Nach Abschluss der Übung solltest du alle erstellten Ressourcen wieder löschen, um unnötige AWS-Kosten zu vermeiden:

```bash
cd ../prod
terraform destroy

cd ../dev
terraform destroy

cd ../shared
terraform destroy
```

Bestätige jeweils mit "yes", wenn du gefragt wirst.

## Tipps

- Der Splat-Operator `[*]` in den Outputs ist eine Terraform-Syntax, um alle Elemente aus einer Liste zu extrahieren
- Die Validierung der Variablen stellt sicher, dass nur sinnvolle Werte übergeben werden
- Durch die Verwendung von `data` für AMIs vermeidest du veraltete AMI-IDs
- Bei der Verwendung von `count` denke daran, dass die Indizes bei 0 beginnen
