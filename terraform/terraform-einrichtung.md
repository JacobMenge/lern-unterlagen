# Terraform mit AWS - Einrichtung und erste Schritte

## Einleitung

In diesem praktischen Teil lernst du, wie du Terraform mit deinem AWS-Account verbindest und deine ersten Schritte damit machst. Wir gehen jeden Schritt detailliert durch, von der Einrichtung der Umgebung bis zur Erstellung einer einfachen "Hello World"-Anwendung in AWS.

## Voraussetzungen

- Terraform ist bereits installiert (wie gestern gemeinsam durchgeführt) --> https://github.com/BrianR-Back2Code/Terraform 
- Du kannst in der WSL (Windows Subsystem for Linux), PowerShell oder deiner bevorzugten Shell arbeiten, wo Terraform installiert ist
- Ein AWS-Account mit Zugriff auf die AWS Management Console
- Grundlegende Kenntnisse über Kommandozeilenbefehle

## 1. Überprüfen der Terraform-Installation

Bevor wir starten, lass uns sicherstellen, dass Terraform korrekt installiert ist:

```bash
terraform --version
```

Du solltest eine Ausgabe ähnlich dieser sehen:
```
Terraform v1.5.7
on linux_amd64
```

Wenn dieser Befehl funktioniert und eine Version anzeigt, ist Terraform korrekt installiert und wir können fortfahren.

## 2. AWS-Zugangsdaten einrichten

Für die Arbeit mit AWS benötigt Terraform Zugangsdaten. Wir werden einen programmatischen Zugang einrichten.

### 2.1 IAM-Benutzer erstellen

1. Melde dich in der [AWS Management Console](https://console.aws.amazon.com/) an

2. Suche nach "IAM" und klicke auf den IAM-Service

3. Klicke im linken Menü auf "Users" und dann auf "Create user"

4. Gib einen Benutzernamen ein (z.B. "terraform-user") und klicke auf "Next"

5. Wähle "Attach policies directly" und suche nach "AdministratorAccess"
   > **Hinweis**: In einer Produktionsumgebung solltest du eingeschränktere Berechtigungen verwenden, aber für dieses Tutorial verwenden wir AdministratorAccess zur Vereinfachung

6. Wähle die "AdministratorAccess"-Policy aus und klicke auf "Next"

7. Überprüfe die Einstellungen und klicke auf "Create user"

### 2.2 Access Keys erstellen

1. Klicke auf den neu erstellten Benutzer in der IAM-Konsole

2. Wechsle zum Tab "Security credentials"

3. Scrolle zum Abschnitt "Access keys" und klicke auf "Create access key"

4. Wähle "Command Line Interface (CLI)" als Anwendungsfall aus

5. Setze optional ein Tag mit dem Schlüssel "Purpose" und dem Wert "Terraform" und klicke auf "Next"

6. Klicke auf "Create access key"

7. **WICHTIG**: Lade die .csv-Datei herunter und/oder notiere dir die "Access key ID" und den "Secret access key". Dies ist der einzige Zeitpunkt, zu dem du auf den Secret Access Key zugreifen kannst!

### 2.3 AWS CLI konfigurieren

Wir werden die AWS CLI verwenden, um die Zugangsdaten zu konfigurieren:

1. Installiere die AWS CLI, falls noch nicht geschehen:

   - **Windows (PowerShell)**:
     ```powershell
     msiexec.exe /i https://awscli.amazonaws.com/AWSCLIV2.msi
     ```

   - **WSL/Linux**:
     ```bash
     curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
     unzip awscliv2.zip
     sudo ./aws/install
     ```

2. Konfiguriere die AWS CLI:

   ```bash
   aws configure
   ```

3. Gib die folgenden Informationen ein, wenn du dazu aufgefordert wirst:
   - AWS Access Key ID: [Deine Access Key ID]
   - AWS Secret Access Key: [Dein Secret Access Key]
   - Default region name: eu-central-1 (oder deine bevorzugte Region)
   - Default output format: json

Diese Konfiguration wird in `~/.aws/credentials` und `~/.aws/config` gespeichert. Terraform wird diese automatisch finden und verwenden.

## 3. Projekt-Verzeichnis erstellen

Jetzt können wir ein Verzeichnis für unser Terraform-Projekt anlegen:

```bash
mkdir terraform-aws-hello-world
cd terraform-aws-hello-world
```

## 4. Erste Terraform-Dateien erstellen

Wir erstellen jetzt die grundlegenden Dateien für unser Projekt.

### 4.1 providers.tf - AWS Provider konfigurieren

Erstelle eine Datei `providers.tf`:

```bash
touch providers.tf
```

Öffne diese Datei in deinem bevorzugten Texteditor und füge Folgendes ein:

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
  region = "eu-central-1"  # Frankfurt, ändere dies bei Bedarf
  
  default_tags {
    tags = {
      Project   = "TerraformHelloWorld"
      ManagedBy = "Terraform"
    }
  }
}
```

Diese Datei definiert:
- Den AWS-Provider, den wir verwenden werden
- Die erforderliche Terraform-Version
- Die AWS-Region, in der wir arbeiten werden
- Standard-Tags, die auf alle Ressourcen angewendet werden

### 4.2 variables.tf - Variablen definieren

Erstelle eine Datei `variables.tf`:

```bash
touch variables.tf
```

Füge Folgendes ein:

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

Diese Datei definiert Variablen, die wir in unserem Projekt verwenden können. Beachte, dass der SSH-Schlüssel optional ist - wir werden ihn in diesem Tutorial nicht verwenden, aber die Variable ist enthalten, falls du später SSH-Zugriff einrichten möchtest.

### 4.3 main.tf - Hauptkonfiguration

Erstelle eine Datei `main.tf`:

```bash
touch main.tf
```

Füge Folgendes ein:

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

# Security Group für den Webserver
resource "aws_security_group" "web" {
  name        = "terraform-hello-world-sg"
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
              echo '<html><head><title>Terraform Hello World</title></head><body><h1>Hello World from Terraform on AWS!</h1><p>Diese Seite wurde automatisch durch Terraform erstellt.</p></body></html>' > /var/www/html/index.html
              EOF

  tags = {
    Name = var.instance_name
  }
}
```

Diese Datei:
- Sucht nach dem neuesten Amazon Linux 2023 AMI
- Erstellt eine Security Group, die HTTP-Verkehr (Port 80) erlaubt
- Erstellt eine EC2-Instance mit einem einfachen Bootstrap-Skript, das einen Webserver installiert und eine "Hello World"-Seite einrichtet

### 4.4 outputs.tf - Ausgaben definieren

Erstelle eine Datei `outputs.tf`:

```bash
touch outputs.tf
```

Füge Folgendes ein:

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

Diese Datei definiert Ausgaben, die nach der Erstellung der Infrastruktur angezeigt werden. Besonders nützlich ist die Website-URL, die wir später verwenden werden.

## 5. Terraform-Workflow ausführen

Jetzt führen wir die Terraform-Befehle aus, um unsere Infrastruktur zu erstellen.

### 5.1 Terraform initialisieren

Im Projektverzeichnis führe aus:

```bash
terraform init
```

Dieser Befehl:
- Initialisiert das Terraform-Projektverzeichnis
- Lädt den AWS-Provider herunter
- Richtet die interne Terraform-Konfiguration ein

Du solltest eine Ausgabe ähnlich dieser sehen:
```
Initializing the backend...

Initializing provider plugins...
- Finding hashicorp/aws versions matching "~> 5.0"...
- Installing hashicorp/aws v5.19.0...
- Installed hashicorp/aws v5.19.0 (signed by HashiCorp)

Terraform has been successfully initialized!

You may now begin working with Terraform. Try running "terraform plan" to see
any changes that are required for your infrastructure. All Terraform commands
should now work.
```

### 5.2 Terraform-Plan erstellen

Erstelle einen Ausführungsplan:

```bash
terraform plan
```

Dieser Befehl zeigt an, welche Ressourcen Terraform erstellen wird. Überprüfe den Plan sorgfältig. Du solltest sehen, dass Terraform eine EC2-Instance und eine Security Group erstellen wird.

### 5.3 Infrastruktur anwenden

Jetzt wenden wir die Konfiguration an und erstellen die Ressourcen:

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

Öffne einen Webbrowser und navigiere zur URL, die in der Ausgabe als `website_url` angegeben ist. Du solltest eine einfache Webseite mit der Nachricht "Hello World from Terraform on AWS!" sehen.

Herzlichen Glückwunsch! Du hast erfolgreich:
1. Terraform mit deinem AWS-Account verbunden
2. Eine EC2-Instance mit einem Webserver erstellt
3. Eine Security Group konfiguriert, um HTTP-Verkehr zuzulassen
4. Eine einfache "Hello World"-Webseite bereitgestellt

## 7. Ressourcen aufräumen

Wenn du mit dem Experiment fertig bist, solltest du die erstellten Ressourcen löschen, um unerwartete AWS-Kosten zu vermeiden:

```bash
terraform destroy
```

Terraform zeigt die Ressourcen an, die gelöscht werden, und fragt nach einer Bestätigung. Gib `yes` ein, um fortzufahren.

Nach Abschluss werden alle von Terraform erstellten Ressourcen gelöscht.

## 8. Nächste Schritte

Jetzt, da du die Grundlagen der Arbeit mit Terraform und AWS kennst, könntest du:

1. **Komplexere Infrastrukturen erstellen**: Füge weitere Ressourcen wie S3-Buckets, RDS-Datenbanken oder Elastic Load Balancer hinzu

2. **Terraform-Module verwenden**: Erstelle wiederverwendbare Module für häufig verwendete Ressourcengruppen

3. **Remote State konfigurieren**: Richte einen S3-Bucket für den Remote State ein, um die Zusammenarbeit im Team zu erleichtern

4. **CI/CD-Integration**: Integriere Terraform in deine CI/CD-Pipeline, um Infrastructure-as-Code-Praktiken zu automatisieren

5. **Terraform Cloud erkunden**: Untersuche Terraform Cloud für eine verbesserte Zusammenarbeit und Workflow-Management

## Fehlerbehebung

### "Error: No valid credential sources found"

Falls du diese Fehlermeldung siehst:
1. Überprüfe, ob du `aws configure` ausgeführt hast
2. Stelle sicher, dass deine Zugangsdaten korrekt sind
3. Überprüfe, ob die AWS-Region in deiner Konfiguration korrekt ist

### "Error: Error creating security group"

Möglicherweise existiert die Security Group bereits:
1. Ändere den Namen der Security Group in der `main.tf`-Datei
2. Oder lösche die bestehende Security Group über die AWS-Konsole

### Webseite ist nicht erreichbar

Wenn die Webseite nach einigen Minuten nicht erreichbar ist:
1. Überprüfe, ob die EC2-Instance läuft (AWS-Konsole → EC2 → Instances)
2. Überprüfe die Security Group-Regeln (AWS-Konsole → EC2 → Security Groups)
3. SSH dich in die Instance ein (falls du einen SSH-Schlüssel konfiguriert hast) und überprüfe den Status des Webservers: `systemctl status httpd`

## Zusammenfassung

In diesem praktischen Tutorial hast du gelernt, wie du:
- Terraform mit deinem AWS-Account verbindest
- Eine einfache Infrastruktur mit Terraform Code definierst
- Den Terraform-Workflow (init, plan, apply, destroy) ausführst
- Eine einfache Webanwendung auf AWS bereitstellst

Diese Grundlagen bilden das Fundament für komplexere Infrastructure-as-Code-Projekte und ermöglichen dir, deine AWS-Infrastruktur effizient zu verwalten und zu automatisieren.
