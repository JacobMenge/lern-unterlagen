# Terraform Tutorial: Infrastruktur als Code verstehen und anwenden

*Ein praktischer Leitfaden für Einsteiger*

## Einleitung

In diesem Tutorial nehme ich dich an die Hand und zeige dir, wie du mit Terraform deine IT-Infrastruktur verwalten kannst. 

Stell dir vor, du müsstest jeden Server, jedes Netzwerk und jede Datenbank manuell einrichten – das wäre wie ein Haus mit Hammer und Nagel zu bauen, ohne Bauplan. Terraform ist dein Bauplan für die Cloud, der dir hilft, deine gesamte Infrastruktur zu automatisieren.

## Was ist Terraform eigentlich?

Terraform ist ein Open-Source-Tool von HashiCorp, mit dem du deine Infrastruktur als Code (IaC) definieren kannst. Das bedeutet, du beschreibst in Textdateien, welche Ressourcen du haben möchtest – Server, Datenbanken, Netzwerke – und Terraform kümmert sich darum, dass alles genau so eingerichtet wird.

Das Coole daran? Du kannst diese Dateien in deiner Versionskontrolle speichern, mit Kollegen teilen und jederzeit reproduzierbare Umgebungen erstellen.

## Warum solltest du Terraform nutzen?

- **Zeitersparnis**: Einmal geschrieben, kannst du deine Infrastruktur immer wieder mit einem Befehl aufsetzen
- **Fehlerreduzierung**: Keine manuellen Klicks mehr in Cloud-Konsolen, die zu Fehlern führen
- **Dokumentation inklusive**: Dein Code zeigt genau, wie deine Infrastruktur aussieht
- **Multi-Cloud-Fähigkeit**: Egal ob AWS, Azure, Google Cloud oder andere – Terraform spricht mit allen

## Installation: Lass uns loslegen!

Bevor wir richtig starten können, musst du Terraform auf deinem Rechner installieren:

### Für Windows:
1. Geh auf die [Terraform-Download-Seite](https://www.terraform.io/downloads.html)
2. Lade die Windows-Version herunter
3. Entpacke die ZIP-Datei
4. Füge den Pfad zur terraform.exe zu deinen Umgebungsvariablen hinzu

### Für MacOS:
```bash
brew install terraform
```

### Für Linux mit Brians Installationsskript:

Speichere zunächst das folgende Skript in einer Datei namens `install_terraform.sh`:

```bash
#! bin/bash
sudo apt-get update && sudo apt-get install -y gnupg software-properties-common
wget -O- https://apt.releases.hashicorp.com/gpg | \
gpg --dearmor | \
sudo tee /usr/share/keyrings/hashicorp-archive-keyring.gpg > /dev/null
gpg --no-default-keyring \
--keyring /usr/share/keyrings/hashicorp-archive-keyring.gpg \
--fingerprint
echo "deb [signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] \
https://apt.releases.hashicorp.com $(lsb_release -cs) main" | \
sudo tee /etc/apt/sources.list.d/hashicorp.list
# Tatsächlichen Fingerabdruck des gespeicherten Schlüssels abrufen
ACTUAL_FINGERPRINT=$(gpg --no-default-keyring \
  --keyring /usr/share/keyrings/hashicorp-archive-keyring.gpg \
  --fingerprint | grep -Eo "([0-9A-F]{4} ){9}[0-9A-F]{4}")
# Fingerabdruck vergleichen
if [[ "$ACTUAL_FINGERPRINT" != "$EXPECTED_FINGERPRINT" ]]; then
    echo "Fehler: Der GPG-Schlüssel stimmt nicht mit dem erwarteten Fingerabdruck überein!"
    exit 1
fi
echo "GPG-Schlüssel erfolgreich überprüft!"
sleep 3
sudo apt update
sudo apt-get install terraform
terraform -v
sleep 2
echo "Terraform ist erfolgreich installiert worden"
terraform -install-autocomplete
echo "Terraform Auto Complete installiert"
```

Führe dann folgende Befehle aus, um das Skript ausführbar zu machen und zu starten:
```bash
chmod +x install_terraform.sh
./install_terraform.sh
```

### Alternative manuelle Installation für Linux:
```bash
sudo apt-get update && sudo apt-get install -y gnupg software-properties-common
wget -O- https://apt.releases.hashicorp.com/gpg | gpg --dearmor | sudo tee /usr/share/keyrings/hashicorp-archive-keyring.gpg
echo "deb [signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] https://apt.releases.hashicorp.com $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/hashicorp.list
sudo apt-get update && sudo apt-get install terraform
```

Überprüfe die Installation mit:
```bash
terraform version
```

## Einrichtung mit der Techstarter AWS Sandbox

Bevor wir mit unserem ersten Terraform-Projekt starten, müssen wir Terraform für die Arbeit mit der Techstarter AWS Sandbox konfigurieren.

### AWS Zugang einrichten - Detaillierte Anleitung

So erhältst du deine AWS Zugangsdaten von der Techstarter AWS Sandbox:

1. **Zugang zur AWS Management Console**:
   - Öffne den von Techstarter bereitgestellten Link zur AWS Sandbox
   - Gib die Anmeldedaten ein, die du von deinem Ausbilder erhalten hast
   - Nach erfolgreicher Anmeldung befindest du dich in der AWS Management Console

2. **Erstellen eines IAM-Benutzers mit Zugriffsschlüsseln**:
   - Klicke oben rechts auf deinen Kontonamen und wähle "Security Credentials" aus dem Dropdown-Menü
   - Scrolle nach unten zum Abschnitt "Access keys"
   - Klicke auf "Create access key" (Zugriffsschlüssel erstellen)
   - Wähle als Anwendungsfall "Command Line Interface (CLI)" aus
   - Setze den Haken bei "I understand..." und klicke auf "Next"
   - Gib eine Beschreibung ein (z.B. "Terraform-Zugang") und klicke auf "Create access key"
   - **WICHTIG**: Die Seite mit "Access key created" zeigt dir jetzt einmalig die Zugangsdaten an:
     * Access key ID (Zugriffsschlüssel-ID)
     * Secret access key (Geheimer Zugriffsschlüssel)
   - Lade die .csv-Datei herunter oder kopiere beide Werte und bewahre sie sicher auf. Die geheimen Schlüssel werden dir nur EINMAL angezeigt!

3. **Konfiguriere die AWS CLI mit deinen Zugangsdaten**:
   ```bash
   aws configure
   ```
   
   Bei der Eingabeaufforderung gibst du folgende Informationen ein:
   - AWS Access Key ID: [Deine Access Key ID aus Schritt 2]
   - AWS Secret Access Key: [Dein Secret Access Key aus Schritt 2]
   - Default region name: eu-central-1 (für Frankfurt)
   - Default output format: json

4. **Alternativ: Umgebungsvariablen setzen**:
   Wenn du die Zugangsdaten nur temporär für die aktuelle Terminal-Sitzung verwenden möchtest:
   ```bash
   export AWS_ACCESS_KEY_ID="deine-access-key-id"
   export AWS_SECRET_ACCESS_KEY="dein-secret-access-key"
   export AWS_DEFAULT_REGION="eu-central-1"
   ```
   
   Für Windows Command Prompt:
   ```cmd
   set AWS_ACCESS_KEY_ID=deine-access-key-id
   set AWS_SECRET_ACCESS_KEY=dein-secret-access-key
   set AWS_DEFAULT_REGION=eu-central-1
   ```
   
   Für Windows PowerShell:
   ```powershell
   $env:AWS_ACCESS_KEY_ID="deine-access-key-id"
   $env:AWS_SECRET_ACCESS_KEY="dein-secret-access-key"
   $env:AWS_DEFAULT_REGION="eu-central-1"
   ```

5. **Überprüfen der Konfiguration**:
   Um zu testen, ob deine Konfiguration funktioniert, führe folgenden Befehl aus:
   ```bash
   aws sts get-caller-identity
   ```
   
   Bei erfolgreicher Konfiguration erhältst du eine Ausgabe mit deiner Konto-ID, Benutzer-ID und ARN.

### Beachte die Sandbox-Einschränkungen

Die Techstarter AWS Sandbox hat einige Einschränkungen:
- Begrenzte Ressourcennutzung
- Temporärer Zugang (wird nach einer bestimmten Zeit zurückgesetzt)
- Möglicherweise eingeschränkte Berechtigungen für bestimmte Dienste

Daher ist es wichtig, nach Abschluss der Übungen alle erstellten Ressourcen mit `terraform destroy` zu löschen, um unnötige Kosten zu vermeiden.

## Dein erstes Terraform-Projekt

Okay, jetzt wird's spannend! Lass uns ein einfaches Projekt erstellen. Wir werden einen Server in der AWS Sandbox erstellen.

### Schritt 1: Projekt-Ordner erstellen
Erstelle einen neuen Ordner für dein Projekt:

```bash
mkdir mein-erstes-terraform
cd mein-erstes-terraform
```

### Schritt 2: Provider konfigurieren
Erstelle eine Datei namens `main.tf` und füge folgenden Code ein:

```hcl
provider "aws" {
  region = "eu-central-1"  # Frankfurt
  
  # Optional: Falls du Probleme mit der Sandbox hast, kannst du den Default-Timeout erhöhen
  # retries {
  #   max_attempts = 10
  #   min_timeout  = 10
  # }
}
```

Dieser Code sagt Terraform, dass du mit AWS arbeiten willst und zwar in der Region Frankfurt.

### Schritt 3: Ressourcen definieren
Füge diesem `main.tf` folgende Zeilen hinzu:

```hcl
resource "aws_instance" "mein_server" {
  ami           = "ami-0c55b159cbfafe1f0"  # Ubuntu Server 20.04 LTS
  instance_type = "t2.micro"  # Kostenloser Tier in AWS
  
  tags = {
    Name = "MeinErsterServer"
  }
}
```

Hier definierst du eine EC2-Instance (einen virtuellen Server) in AWS mit einem Ubuntu-Betriebssystem und einem kleinen Instance-Typ.

### Schritt 4: Terraform initialisieren
Im Terminal führst du aus:

```bash
terraform init
```

Dieser Befehl lädt alle notwendigen Provider (in diesem Fall AWS) herunter.

### Schritt 5: Plan erstellen
Jetzt schaust du dir an, was Terraform machen würde:

```bash
terraform plan
```

Terraform zeigt dir, welche Ressourcen es erstellen, ändern oder löschen würde. Das ist wie eine Vorschau deiner Änderungen.

### Schritt 6: Infrastruktur erstellen
Wenn du mit dem Plan zufrieden bist, führst du aus:

```bash
terraform apply
```

Terraform fragt dich zur Sicherheit noch einmal, ob du die Änderungen wirklich durchführen willst. Tippe "yes" ein und drücke Enter.

Glückwunsch! Du hast gerade deinen ersten Server mit Terraform erstellt!

## Terraform-Grundkonzepte

Bevor wir weitermachen, lass uns kurz die wichtigsten Begriffe klären:

### Provider
Provider sind Plugins, die Terraform benutzt, um mit verschiedenen Cloud-Anbietern zu kommunizieren. Es gibt Provider für AWS, Azure, Google Cloud, aber auch für Dienste wie GitHub, Cloudflare und viele mehr.

```hcl
provider "aws" {
  region = "eu-central-1"
}
```

### Ressourcen
Ressourcen sind die Dinge, die du erstellen willst: Server, Datenbanken, Netzwerke usw.

```hcl
resource "aws_instance" "beispiel" {
  # Eigenschaften...
}
```

### Variablen
Variablen machen deinen Code flexibler und wiederverwendbarer.

```hcl
variable "region" {
  description = "AWS Region für unsere Ressourcen"
  default     = "eu-central-1"
  type        = string
  
  validation {
    condition     = contains(["eu-central-1", "eu-west-1"], var.region)
    error_message = "Erlaubte Regionen sind nur eu-central-1 und eu-west-1."
  }
}

provider "aws" {
  region = var.region
}
```

### Outputs
Mit Outputs kannst du wichtige Informationen anzeigen lassen, nachdem Terraform fertig ist.

```hcl
output "server_ip" {
  value = aws_instance.mein_server.public_ip
}
```

### Data Sources
Mit Data Sources kannst du Informationen über bestehende Ressourcen abrufen:

```hcl
data "aws_ami" "ubuntu" {
  most_recent = true
  
  filter {
    name   = "name"
    values = ["ubuntu/images/hvm-ssd/ubuntu-focal-20.04-amd64-server-*"]
  }
  
  filter {
    name   = "virtualization-type"
    values = ["hvm"]
  }
  
  owners = ["099720109477"] # Canonical
}

resource "aws_instance" "web" {
  ami           = data.aws_ami.ubuntu.id
  instance_type = "t2.micro"
}
```

### Locals
Lokale Werte sind wie temporäre Variablen innerhalb deiner Konfiguration:

```hcl
locals {
  common_tags = {
    Project     = "Terraform-Demo"
    Environment = var.environment
    Owner       = "DevOps-Team"
  }
}

resource "aws_instance" "example" {
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = "t2.micro"
  
  tags = merge(local.common_tags, {
    Name = "Web-Server"
  })
}
```

## Ein komplexeres Beispiel: Webserver mit Datenbank

Lass uns jetzt etwas Anspruchsvolleres bauen: einen Webserver mit einer Datenbank dahinter.

Erstelle eine neue Datei `webserver.tf`:

```hcl
# Sicherheitsgruppe für den Webserver
resource "aws_security_group" "web" {
  name        = "web-server-sg"
  description = "Erlaubt HTTP-Verkehr"

  ingress {
    from_port   = 80
    to_port     = 80
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }
}

# Der Webserver selbst
resource "aws_instance" "web" {
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = "t2.micro"
  
  security_groups = [aws_security_group.web.name]
  
  user_data = <<-EOF
              #!/bin/bash
              apt-get update
              apt-get install -y apache2
              systemctl start apache2
              systemctl enable apache2
              echo "<h1>Hallo von Terraform!</h1>" > /var/www/html/index.html
              EOF
  
  tags = {
    Name = "WebServer"
  }
}

# Datenbank-Instanz
resource "aws_db_instance" "default" {
  allocated_storage    = 10
  engine               = "mysql"
  engine_version       = "5.7"
  instance_class       = "db.t2.micro"
  name                 = "mydb"
  username             = "admin"
  password             = "password123"  # In der Praxis: Verwende Secrets!
  parameter_group_name = "default.mysql5.7"
  skip_final_snapshot  = true
}

# Output für die Server-IP
output "web_server_ip" {
  value = aws_instance.web.public_ip
}
```

Dieses Beispiel erstellt:
1. Eine Sicherheitsgruppe, die HTTP-Verkehr erlaubt
2. Einen Webserver mit Apache vorinstalliert
3. Eine MySQL-Datenbank
4. Gibt die IP-Adresse des Webservers aus, wenn alles fertig ist

## Terraform-Zustand verstehen

Ein wichtiger Aspekt von Terraform ist der "State" (Zustand). Terraform speichert Informationen über deine erstellten Ressourcen in einer Datei (standardmäßig `terraform.tfstate`).

Dieser Zustand ist entscheidend, denn er ermöglicht Terraform zu wissen, was es bereits erstellt hat und was es bei einer Änderung aktualisieren muss.

In einem Team solltest du diesen Zustand nicht lokal speichern, sondern an einem gemeinsamen Ort, zum Beispiel in einem S3-Bucket oder Terraform Cloud:

```hcl
terraform {
  backend "s3" {
    bucket = "mein-terraform-zustand"
    key    = "prod/terraform.tfstate"
    region = "eu-central-1"
  }
}
```

## Terraform-Module

Module sind wiederverwendbare Komponenten in Terraform – denk an sie wie an Funktionen in einer Programmiersprache. Sie helfen dir, deinen Code zu organisieren und zu wiederholen.

Hier ist ein einfaches Modul-Beispiel. Erstelle einen Ordner `module/webserver` und darin eine Datei `main.tf`:

```hcl
variable "server_name" {
  description = "Name des Webservers"
}

resource "aws_instance" "web" {
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = "t2.micro"
  
  tags = {
    Name = var.server_name
  }
}

output "instance_id" {
  value = aws_instance.web.id
}
```

Dann kannst du dieses Modul in deinem Hauptcode verwenden:

```hcl
module "webserver_prod" {
  source = "./module/webserver"
  server_name = "Produktion"
}

module "webserver_test" {
  source = "./module/webserver"
  server_name = "Test"
}

output "prod_server_id" {
  value = module.webserver_prod.instance_id
}
```

## Terraform HCL-Syntax im Detail

Terraform verwendet eine eigene Sprache namens HCL (HashiCorp Configuration Language). Hier sind die wichtigsten Elemente dieser Syntax:

### Blöcke und Argumente

Die Grundstruktur in Terraform sind Blöcke und Argumente:

```hcl
block_type "label" "name_label" {
  key = value
  nested_block {
    nested_key = nested_value
  }
}
```

Beispiel:
```hcl
resource "aws_instance" "web" {
  ami           = "ami-123456"
  instance_type = "t2.micro"
}
```

### Datentypen

Terraform unterstützt folgende Datentypen:

- **Zeichenketten**: `"Hallo Welt"`
- **Zahlen**: `42`, `3.14`
- **Booleans**: `true`, `false`
- **Listen**: `["a", "b", "c"]`
- **Maps**: `{ name = "Server1", env = "Produktion" }`
- **Null**: `null`

### Interpolation und Ausdrücke

Du kannst Werte in Zeichenketten einbetten:

```hcl
name = "${var.prefix}-server"
```

Oder Ausdrücke verwenden:

```hcl
instance_count = var.environment == "prod" ? 3 : 1
```

### Kommentare

Terraform unterstützt verschiedene Kommentararten:

```hcl
# Einzeiliger Kommentar

// Auch ein einzeiliger Kommentar

/*
  Mehrzeiliger 
  Kommentar
*/
```

### Funktionen

Terraform bietet zahlreiche eingebaute Funktionen:

```hcl
locals {
  upper_name = upper(var.name)
  timestamp  = timestamp()
  server_ids = concat(aws_instance.app[*].id, aws_instance.db[*].id)
}
```

## Aufbau eines Terraform-Moduls

Ein gut strukturiertes Terraform-Modul enthält typischerweise diese Dateien:

### main.tf
Die Hauptkonfigurationsdatei, die die Kernressourcen enthält.

```hcl
# main.tf
resource "aws_vpc" "this" {
  cidr_block = var.vpc_cidr
  tags       = var.tags
}
```

### variables.tf
Definiert alle Eingabevariablen für das Modul.

```hcl
# variables.tf
variable "vpc_cidr" {
  description = "CIDR-Block für die VPC"
  type        = string
  default     = "10.0.0.0/16"
}

variable "tags" {
  description = "Tags für alle Ressourcen"
  type        = map(string)
  default     = {}
}
```

### outputs.tf
Definiert, welche Werte das Modul zurückgeben soll.

```hcl
# outputs.tf
output "vpc_id" {
  description = "ID der erstellten VPC"
  value       = aws_vpc.this.id
}
```

### versions.tf
Legt fest, welche Terraform- und Provider-Versionen erforderlich sind.

```hcl
# versions.tf
terraform {
  required_version = ">= 1.0.0"
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = ">= 3.0.0"
    }
  }
}
```

### README.md
Dokumentation, wie das Modul verwendet wird.

### examples/
Ordner mit Beispielen für die Verwendung des Moduls.

## Best Practices für Terraform

Hier sind einige Tipps, die dir helfen, erfolgreich mit Terraform zu arbeiten:

1. **Versioniere deinen Code**: Nutze Git oder ein anderes Versionierungssystem
2. **Strukturiere deine Dateien**: 
   - `main.tf` - Hauptkonfiguration
   - `variables.tf` - Variablendefinitionen
   - `outputs.tf` - Output-Definitionen
   - `terraform.tfvars` - Variablenwerte (nicht in Git einchecken für sensible Daten!)
3. **Verwende Module**: Für wiederverwendbare Komponenten
4. **Remote State**: Speichere den Terraform-Zustand remote für Teamarbeit
5. **Plan vor Apply**: Überprüfe immer `terraform plan` vor dem Anwenden von Änderungen
6. **Terraform-Version festlegen**: Definiere die benötigte Terraform-Version in deinem Code:
   ```hcl
   terraform {
     required_version = ">= 1.0.0"
   }
   ```

## Multi-Cloud und Provider-Konfiguration

Terraform kann mit mehreren Cloud-Providern gleichzeitig arbeiten. So konfigurierst du mehrere Provider:

```hcl
# AWS Provider
provider "aws" {
  region = "eu-central-1"
  # Zugangsdaten besser über Umgebungsvariablen oder AWS CLI konfigurieren
}

# Azure Provider
provider "azurerm" {
  features {}
  # Zugangsdaten besser über Azure CLI oder Umgebungsvariablen
}

# Ressource in AWS
resource "aws_s3_bucket" "example" {
  bucket = "mein-beispiel-bucket"
}

# Ressource in Azure
resource "azurerm_resource_group" "example" {
  name     = "example-resources"
  location = "West Europe"
}
```

## Provisioner für Konfigurationsmanagement

Manchmal musst du nach dem Erstellen einer Ressource zusätzliche Aktionen ausführen. Dafür kannst du Provisioner verwenden:

```hcl
resource "aws_instance" "web" {
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = "t2.micro"
  
  # SSH-Schlüssel für den Zugriff
  key_name = aws_key_pair.my_key.key_name

  # Remote-Exec-Provisioner führt Befehle auf der Instanz aus
  provisioner "remote-exec" {
    inline = [
      "sudo apt-get update",
      "sudo apt-get install -y nginx",
      "sudo systemctl start nginx"
    ]
    
    connection {
      type        = "ssh"
      user        = "ubuntu"
      private_key = file("~/.ssh/id_rsa")
      host        = self.public_ip
    }
  }
  
  # Local-Exec-Provisioner führt Befehle auf deinem lokalen Computer aus
  provisioner "local-exec" {
    command = "echo 'Server-IP: ${self.public_ip}' > server_ip.txt"
  }
}
```

## CI/CD-Integration mit Terraform

Hier ist ein einfaches Beispiel, wie du Terraform in eine CI/CD-Pipeline integrieren kannst:

### GitHub Actions Workflow-Beispiel

```yaml
name: Terraform CI/CD

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  terraform:
    runs-on: ubuntu-latest
    
    steps:
    - uses: actions/checkout@v2
    
    - name: Setup Terraform
      uses: hashicorp/setup-terraform@v1
      
    - name: Terraform Init
      run: terraform init
      
    - name: Terraform Format
      run: terraform fmt -check
      
    - name: Terraform Validate
      run: terraform validate
      
    - name: Terraform Plan
      run: terraform plan
      if: github.event_name == 'pull_request'
      
    - name: Terraform Apply
      run: terraform apply -auto-approve
      if: github.ref == 'refs/heads/main' && github.event_name == 'push'
```

## Import bestehender Infrastruktur

Wenn du bereits Ressourcen in deiner Cloud hast und diese mit Terraform verwalten möchtest, kannst du sie importieren:

1. Erstelle zuerst die Terraform-Konfiguration für die Ressource:

```hcl
resource "aws_instance" "beispiel" {
  # Minimale Konfiguration, wird nach dem Import vervollständigt
  ami           = "ami-1234567890abcdef0"
  instance_type = "t2.micro"
}
```

2. Führe den Import-Befehl aus:

```bash
terraform import aws_instance.beispiel i-1234567890abcdef0
```

3. Aktualisiere deine Konfiguration mit den tatsächlichen Ressourcendetails:

```bash
terraform state show aws_instance.beispiel
```

Kopiere die relevanten Attribute in deine Konfigurationsdatei.

## Hochverfügbarkeits-Architektur mit Terraform

Hier ist ein Beispiel für eine HA-Architektur in AWS:

```hcl
# VPC einrichten
resource "aws_vpc" "main" {
  cidr_block = "10.0.0.0/16"
  
  tags = {
    Name = "main-vpc"
  }
}

# Subnets in verschiedenen Availability Zones
resource "aws_subnet" "subnet_a" {
  vpc_id            = aws_vpc.main.id
  cidr_block        = "10.0.1.0/24"
  availability_zone = "eu-central-1a"
}

resource "aws_subnet" "subnet_b" {
  vpc_id            = aws_vpc.main.id
  cidr_block        = "10.0.2.0/24"
  availability_zone = "eu-central-1b"
}

# Launch Configuration für Auto Scaling
resource "aws_launch_configuration" "web" {
  name_prefix     = "web-"
  image_id        = "ami-0c55b159cbfafe1f0"
  instance_type   = "t2.micro"
  security_groups = [aws_security_group.web.id]
  
  lifecycle {
    create_before_destroy = true
  }
}

# Auto Scaling Group
resource "aws_autoscaling_group" "web" {
  name                 = "web-asg"
  min_size             = 2
  max_size             = 5
  desired_capacity     = 2
  launch_configuration = aws_launch_configuration.web.id
  vpc_zone_identifier  = [aws_subnet.subnet_a.id, aws_subnet.subnet_b.id]
  
  tag {
    key                 = "Name"
    value               = "web-server"
    propagate_at_launch = true
  }
}

# Load Balancer
resource "aws_lb" "web" {
  name               = "web-lb"
  internal           = false
  load_balancer_type = "application"
  security_groups    = [aws_security_group.lb.id]
  subnets            = [aws_subnet.subnet_a.id, aws_subnet.subnet_b.id]
}

# Listener für den Load Balancer
resource "aws_lb_listener" "front_end" {
  load_balancer_arn = aws_lb.web.arn
  port              = "80"
  protocol          = "HTTP"
  
  default_action {
    type             = "forward"
    target_group_arn = aws_lb_target_group.web.arn
  }
}

# Target Group für den Load Balancer
resource "aws_lb_target_group" "web" {
  name     = "web-lb-tg"
  port     = 80
  protocol = "HTTP"
  vpc_id   = aws_vpc.main.id
}

# Auto Scaling an Load Balancer anbinden
resource "aws_autoscaling_attachment" "web" {
  autoscaling_group_name = aws_autoscaling_group.web.id
  alb_target_group_arn   = aws_lb_target_group.web.arn
}
```

## Workspaces und Umgebungsmanagement

Terraform bietet Workspaces, um mehrere Umgebungen (wie Entwicklung, Staging und Produktion) mit dem gleichen Terraform-Code zu verwalten:

```bash
# Anzeigen der verfügbaren Workspaces
terraform workspace list

# Neuen Workspace erstellen
terraform workspace new produktion

# Workspace wechseln
terraform workspace select entwicklung
```

Du kannst den aktuellen Workspace in deinem Code verwenden:

```hcl
resource "aws_instance" "beispiel" {
  count = terraform.workspace == "produktion" ? 3 : 1
  
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = terraform.workspace == "produktion" ? "t2.medium" : "t2.micro"
  
  tags = {
    Name        = "server-${terraform.workspace}-${count.index + 1}"
    Environment = terraform.workspace
  }
}
```

Eine alternative Methode ist die Verwendung von Verzeichnisstrukturen:

```
project/
├── common/
│   ├── main.tf
│   └── outputs.tf
├── produktion/
│   ├── main.tf
│   └── terraform.tfvars
└── entwicklung/
    ├── main.tf
    └── terraform.tfvars
```

## Dynamische Blöcke und Meta-Argumente

Dynamische Blöcke ermöglichen dir, wiederholende Strukturen zu generieren:

```hcl
variable "ingress_rules" {
  type = list(object({
    port        = number
    protocol    = string
    cidr_blocks = list(string)
  }))
  default = [
    {
      port        = 80
      protocol    = "tcp"
      cidr_blocks = ["0.0.0.0/0"]
    },
    {
      port        = 443
      protocol    = "tcp"
      cidr_blocks = ["0.0.0.0/0"]
    }
  ]
}

resource "aws_security_group" "web" {
  name = "web-sg"
  
  dynamic "ingress" {
    for_each = var.ingress_rules
    content {
      from_port   = ingress.value.port
      to_port     = ingress.value.port
      protocol    = ingress.value.protocol
      cidr_blocks = ingress.value.cidr_blocks
    }
  }
}
```

Meta-Argumente steuern das Verhalten von Ressourcen:

```hcl
resource "aws_instance" "cluster" {
  # For_each erstellt mehrere Ressourcen basierend auf einem Map oder Set
  for_each = {
    "web-1" = { type = "t2.micro", az = "eu-central-1a" }
    "web-2" = { type = "t2.micro", az = "eu-central-1b" }
    "db"    = { type = "t2.medium", az = "eu-central-1a" }
  }
  
  ami               = "ami-0c55b159cbfafe1f0"
  instance_type     = each.value.type
  availability_zone = each.value.az
  
  tags = {
    Name = each.key
  }
  
  # Lifecycle-Konfigurationen
  lifecycle {
    create_before_destroy = true
    prevent_destroy       = false
    ignore_changes        = [tags]
  }
  
  # Depends_on für explizite Abhängigkeiten
  depends_on = [aws_vpc.main, aws_subnet.example]
}
```

## Terraform zerstören

Wenn du mit deinem Projekt fertig bist oder von vorne anfangen willst, kannst du alle erstellten Ressourcen mit einem Befehl entfernen:

```bash
terraform destroy
```

Auch hier fragt Terraform zur Sicherheit nach, ob du wirklich alles löschen willst.

## Häufige Fehler und wie du sie löst

### Problem: Zugriffsdaten fehlen
```
Error: NoCredentialProviders: no valid providers in chain
```

**Lösung**: Stelle sicher, dass du deine AWS-Zugangsdaten konfiguriert hast, entweder durch:
- AWS CLI: `aws configure`
- Umgebungsvariablen: `AWS_ACCESS_KEY_ID` und `AWS_SECRET_ACCESS_KEY`
- Direkt im Provider-Block (nicht für Produktionsumgebungen empfohlen)

### Problem: Ressource kann nicht gelöscht werden
```
Error: Error deleting resource: resource is in use
```

**Lösung**: Finde heraus, welche Abhängigkeiten die Ressource hat. Oft musst du zuerst andere Ressourcen löschen, die von dieser abhängen.

### Problem: Terraform-Zustand ist veraltet
```
Error: Resource already exists
```

**Lösung**: Wenn sich die Infrastruktur außerhalb von Terraform geändert hat, kannst du den Zustand aktualisieren:
```bash
terraform import [RESSOURCENTYP].[NAME] [ID]
```

## Wichtige Terraform-Befehle im Überblick

Hier findest du eine Übersicht der wichtigsten Terraform-Befehle, die du kennen solltest:

| Befehl | Beschreibung |
|--------|--------------|
| `terraform init` | Initialisiert ein Terraform-Projekt und lädt Provider |
| `terraform plan` | Zeigt einen Ausführungsplan (welche Änderungen würden gemacht) |
| `terraform apply` | Wendet die Änderungen an und erstellt/aktualisiert die Infrastruktur |
| `terraform destroy` | Löscht alle von Terraform verwalteten Ressourcen |
| `terraform validate` | Überprüft die Syntax der Terraform-Dateien |
| `terraform fmt` | Formatiert die Terraform-Dateien einheitlich |
| `terraform show` | Zeigt den aktu
