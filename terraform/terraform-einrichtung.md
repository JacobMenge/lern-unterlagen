# Terraform mit AWS - Einrichtung und erste Schritte in CloudShell

## Einleitung

In diesem praktischen Tutorial lernst du, wie du Terraform in der AWS CloudShell nutzen kannst, um deine erste Infrastruktur als Code zu erstellen. Die Verwendung der CloudShell bietet mehrere Vorteile, insbesondere in einer verwalteten AWS-Sandbox-Umgebung:

- Keine lokale Installation von Terraform erforderlich
- Automatische AWS-Authentifizierung (keine Access Keys nötig)
- Konsistente Arbeitsumgebung für alle Teilnehmer
- Direkte Integration mit deiner AWS-Umgebung

Wir werden Schritt für Schritt durch den Prozess gehen und ein einfaches "Hello World"-Webserver-Projekt erstellen.

## Voraussetzungen

- Zugang zu deiner AWS-Sandbox über [https://techstarter-sandboxes.awsapps.com/start](https://techstarter-sandboxes.awsapps.com/start)
- Grundlegende Kenntnisse über Kommandozeilenbefehle
- Keine lokale Terraform-Installation erforderlich - wir verwenden die AWS CloudShell!

## 1. AWS CloudShell starten

1. Melde dich bei deiner AWS-Sandbox an: [https://techstarter-sandboxes.awsapps.com/start](https://techstarter-sandboxes.awsapps.com/start)

2. Nach erfolgreicher Anmeldung befindest du dich in der AWS Management Console.

3. Starte die CloudShell, indem du auf das CloudShell-Symbol in der oberen Navigationsleiste klickst (es sieht aus wie ein Kommandozeilen-Symbol >_).

4. Die CloudShell wird in einem neuen Browser-Tab oder -Fenster geöffnet. Die Initialisierung kann einige Momente dauern.

## 2. Terraform in CloudShell installieren

AWS CloudShell hat viele Tools vorinstalliert, aber Terraform ist möglicherweise noch nicht darunter. Du kannst es schnell und einfach selbst installieren:

```bash
mkdir -p ~/bin
wget -q -O terraform.zip https://releases.hashicorp.com/terraform/1.5.7/terraform_1.5.7_linux_amd64.zip
unzip terraform.zip -d ~/bin
rm terraform.zip
export PATH=$PATH:~/bin
echo 'export PATH=$PATH:~/bin' >> ~/.bashrc
```

Überprüfe die Installation mit:

```bash
terraform version
```

Du solltest eine Ausgabe sehen, die etwa so aussieht:
```
Terraform v1.5.7
on linux_amd64
```

## 3. Projekt-Verzeichnis erstellen

Erstelle ein Verzeichnis für dein Terraform-Projekt:

```bash
mkdir terraform-aws-hello-world
cd terraform-aws-hello-world
```

## 4. Terraform-Dateien erstellen

Jetzt erstellen wir die notwendigen Terraform-Dateien direkt in der CloudShell. Du kannst dafür den integrierten Editor oder Kommandozeilen-Editoren wie `nano` oder `vim` verwenden.

### 4.1 providers.tf - AWS Provider konfigurieren

Um den integrierten Editor zu verwenden:
1. Klicke auf "Actions" in der CloudShell
2. Wähle "New File"
3. Gib den Dateinamen "providers.tf" ein
4. Füge folgenden Inhalt ein:

```hcl
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
  
  required_version = ">= 1.5.0"
}

provider "aws" {
  # Region der AWS-Sandbox - passe diese ggf. an
  region = "eu-central-1"
  
  default_tags {
    tags = {
      Project   = "TerraformHelloWorld"
      ManagedBy = "Terraform"
    }
  }
}
```

Alternativ kannst du auch den Kommandozeilen-Editor `nano` verwenden:

```bash
nano providers.tf
# Füge den obigen Inhalt ein, speichere mit Ctrl+O, beende mit Ctrl+X
```

**Wichtig**: Stelle sicher, dass du die richtige AWS-Region angibst. Die Region deiner AWS-Sandbox findest du in der oberen rechten Ecke der AWS Management Console.

### 4.2 variables.tf - Variablen definieren

Erstelle eine Datei `variables.tf`:

```bash
nano variables.tf
```

Füge folgenden Inhalt ein:

```hcl
variable "instance_name" {
  description = "Name der EC2-Instance"
  type        = string
  default     = "terraform-hello-world"
}

variable "instance_type" {
  description = "EC2-Instance-Typ"
  type        = string
  default     = "t2.micro"
}

variable "ssh_key_name" {
  description = "Name des SSH-Schlüssels für die EC2-Instance (optional)"
  type        = string
  default     = null
}
```

### 4.3 main.tf - Hauptkonfiguration

Erstelle eine Datei `main.tf`:

```bash
nano main.tf
```

Füge folgenden Inhalt ein:

```hcl
# Neuestes Amazon Linux 2023 AMI finden
data "aws_ami" "amazon_linux" {
  most_recent = true
  owners      = ["amazon"]

  filter {
    name   = "name"
    values = ["al2023-ami-*-x86_64"]
  }

  filter {
    name   = "virtualization-type"
    values = ["hvm"]
  }
}

# ALTERNATIVE: Falls Amazon Linux 2023 in deiner Region nicht verfügbar ist,
# kommentiere den obigen Block aus und entferne die Kommentare vom folgenden Block:
# data "aws_ami" "amazon_linux" {
#   most_recent = true
#   owners      = ["amazon"]
#
#   filter {
#     name   = "name"
#     values = ["amzn2-ami-hvm-*-x86_64-gp2"]
#   }
#
#   filter {
#     name   = "virtualization-type"
#     values = ["hvm"]
#   }
# }

# Security Group für den Webserver
resource "aws_security_group" "web" {
  name        = "terraform-hello-world-sg-${formatdate("YYMMDDhhmmss", timestamp())}"  # Eindeutiger Name
  description = "Erlaubt HTTP-Verkehr"

  ingress {
    from_port   = 80
    to_port     = 80
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
    description = "HTTP von überall erlauben"
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
    description = "Ausgehenden Verkehr erlauben"
  }
}

# EC2 Instance
resource "aws_instance" "web" {
  ami                    = data.aws_ami.amazon_linux.id
  instance_type          = var.instance_type
  vpc_security_group_ids = [aws_security_group.web.id]
  key_name               = var.ssh_key_name

  user_data = <<-EOF
              #!/bin/bash
              dnf update -y
              dnf install -y httpd
              systemctl start httpd
              systemctl enable httpd
              echo '<html><head><title>Terraform Hello World</title></head><body><h1>Hello World from Terraform on AWS!</h1><p>Diese Seite wurde automatisch durch Terraform in der AWS CloudShell erstellt.</p><p>Dies ist deine erste AWS-Ressource, die mit Terraform in deiner Sandbox erstellt wurde.</p></body></html>' > /var/www/html/index.html
              EOF

  tags = {
    Name = var.instance_name
  }
}
```

### 4.4 outputs.tf - Ausgaben definieren

Erstelle eine Datei `outputs.tf`:

```bash
nano outputs.tf
```

Füge folgenden Inhalt ein:

```hcl
output "instance_id" {
  description = "ID der erstellten EC2-Instance"
  value       = aws_instance.web.id
}

output "public_ip" {
  description = "Öffentliche IP-Adresse der EC2-Instance"
  value       = aws_instance.web.public_ip
}

output "website_url" {
  description = "URL zur erstellten Website"
  value       = "http://${aws_instance.web.public_ip}"
}
```

## 5. Terraform-Workflow ausführen

Jetzt führen wir die Terraform-Befehle in der CloudShell aus, um unsere Infrastruktur zu erstellen.

### 5.1 Terraform initialisieren

```bash
terraform init
```

Dieser Befehl:
- Initialisiert das Terraform-Projektverzeichnis
- Lädt den AWS-Provider herunter
- Richtet die interne Terraform-Konfiguration ein

Du solltest eine erfolgreiche Initialisierungsmeldung sehen.

### 5.2 Terraform-Plan erstellen

```bash
terraform plan
```

Dieser Befehl zeigt an, welche Ressourcen Terraform erstellen wird. Überprüfe den Plan sorgfältig.

### 5.3 Infrastruktur anwenden

```bash
terraform apply
```

Terraform zeigt den Plan erneut an und fragt nach einer Bestätigung. Gib `yes` ein, um fortzufahren.

Die Ausführung kann einige Minuten dauern. Am Ende solltest du eine Ausgabe mit den definierten Outputs sehen, einschließlich der URL zur Website:

```
Apply complete! Resources: 2 added, 0 changed, 0 destroyed.

Outputs:

instance_id = "i-0123456789abcdef0"
public_ip = "3.123.45.67"
website_url = "http://3.123.45.67"
```

## 6. Website testen

Öffne einen neuen Browser-Tab und navigiere zur URL, die in der Ausgabe als `website_url` angegeben ist. Du solltest eine einfache Webseite mit der Nachricht "Hello World from Terraform on AWS!" sehen.

## 7. Ressourcen aufräumen

Wenn du mit dem Experiment fertig bist, solltest du die erstellten Ressourcen löschen, um dein Kontingent in der Sandbox-Umgebung nicht unnötig zu belasten:

```bash
terraform destroy
```

Terraform zeigt die Ressourcen an, die gelöscht werden sollen, und fragt nach einer Bestätigung. Gib `yes` ein, um fortzufahren.

**Hinweis für Sandbox-Umgebungen**: In vielen AWS-Sandbox-Umgebungen werden Ressourcen automatisch gelöscht, wenn die Sitzung beendet wird oder nach Ablauf eines vordefinierten Zeitraums. Es ist dennoch eine gute Praxis, `terraform destroy` auszuführen, um deine Ressourcen sauber zu entfernen und um innerhalb der Kontingentgrenzen deiner Sandbox zu bleiben.

## 8. CloudShell-Tipps

### Persistenter Speicher
Deine Dateien in der CloudShell werden im 1GB großen persistenten Speicher ($HOME) gespeichert und bleiben auch nach dem Schließen erhalten.

### Dateien hochladen/herunterladen
Du kannst Dateien in die CloudShell hochladen oder aus ihr herunterladen:
1. Klicke auf "Actions" in der CloudShell
2. Wähle "Upload file" oder "Download file"

### Inaktivitäts-Timeout
Die CloudShell-Sitzung wird nach 20-30 Minuten Inaktivität automatisch beendet. Speichere deine Arbeit regelmäßig.

### Text kopieren/einfügen
- **Kopieren**: Text in der CloudShell markieren und Ctrl+C oder Cmd+C drücken
- **Einfügen**: Ctrl+V, Cmd+V oder Rechtsklick verwenden

## Fehlerbehebung

### "Error: Unable to find AMI"

Falls das AMI nicht gefunden wird:
1. Überprüfe die Region in der AWS-Konsole (oben rechts)
2. Verwende die alternative AMI-Konfiguration in main.tf (Amazon Linux 2 statt Amazon Linux 2023)

### "Error: Error creating security group"

Möglicherweise gibt es Probleme mit der Security Group:
1. Überprüfe, ob dein Sandbox-Benutzer Berechtigungen zur Erstellung von Security Groups hat
2. Die Konfiguration fügt bereits einen Zeitstempel hinzu, um eindeutige Namen zu generieren

### "Error: Error launching source instance"

In den Sandbox-Umgebungen gibt es manchmal Beschränkungen:
1. Überprüfe, ob du das Kontingent für EC2-Instances erreicht hast
2. Wechsle zu einem kleineren Instance-Typ in der variables.tf (z.B. von t2.micro zu t2.nano)
3. Überprüfe in der AWS-Konsole, ob in deiner Region zusätzliche Einschränkungen bestehen

### "Error: VPC not found" oder VPC-bezogene Fehler

In Sandbox-Umgebungen könnte die VPC-Konfiguration anders sein:
1. Öffne die AWS-Konsole und navigiere zum VPC-Service
2. Notiere die ID der Default-VPC oder einer anderen verfügbaren VPC
3. Füge diese VPC-ID explizit zu deiner EC2-Instance-Konfiguration in main.tf hinzu:
   ```hcl
   resource "aws_instance" "web" {
     # ... andere Konfiguration
     subnet_id = "subnet-xxxxxxxx"  # Eine Subnet-ID aus deiner VPC
   }
   ```

## Zusammenfassung

In diesem Tutorial hast du gelernt, wie du:
- Die AWS CloudShell für Terraform-Projekte nutzt
- Terraform in der CloudShell installierst
- Eine einfache Infrastruktur mit Terraform Code definierst
- Den Terraform-Workflow (init, plan, apply, destroy) ausführst
- Eine "Hello World"-Webanwendung auf AWS bereitstellst

Die Verwendung der CloudShell vereinfacht den Einstieg in Terraform erheblich, da du keine lokale Installation benötigst und die AWS-Authentifizierung automatisch eingerichtet ist. Dieser Ansatz ist besonders nützlich in verwalteten AWS-Sandbox-Umgebungen.
