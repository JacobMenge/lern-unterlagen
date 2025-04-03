# Terraform Hello World in AWS CloudShell

Dieses Tutorial zeigt dir, wie du mit Terraform in der AWS CloudShell schnell einen einfachen Webserver erstellst.

## Was werden wir tun?

- Terraform in der AWS CloudShell installieren
- Mit Terraform eine EC2-Instance (virtuellen Server) erstellen
- Auf diesem Server einen einfachen Webserver einrichten
- Eine "Hello World"-Webseite anzeigen
- Alles wieder aufräumen, wenn wir fertig sind

## 1. CloudShell starten

Die AWS CloudShell ist eine Kommandozeile direkt im Browser, mit der du AWS-Ressourcen verwalten kannst. Sie hat den Vorteil, dass sie bereits mit deinem AWS-Account verbunden ist.

1. Melde dich bei deiner AWS-Sandbox an: [https://techstarter-sandboxes.awsapps.com/start](https://techstarter-sandboxes.awsapps.com/start)

2. Klicke auf das CloudShell-Symbol in der oberen Navigationsleiste (>_)

## 2. Terraform installieren

Terraform ist ein Tool, mit dem du Infrastruktur als Code definieren und verwalten kannst. Wir installieren es in der CloudShell:

```bash
mkdir -p ~/bin
wget -q -O terraform.zip https://releases.hashicorp.com/terraform/1.6.6/terraform_1.6.6_linux_amd64.zip
unzip terraform.zip -d ~/bin
rm terraform.zip
export PATH=$PATH:~/bin
echo 'export PATH=$PATH:~/bin' >> ~/.bashrc
```

Diese Befehle:
- Erstellen ein Verzeichnis für Terraform
- Laden Terraform herunter
- Entpacken die Dateien
- Machen Terraform im System verfügbar

Prüfe, ob Terraform funktioniert:

```bash
terraform version
```

## 3. Projekt erstellen

Wir erstellen ein Verzeichnis für unser Terraform-Projekt:

```bash
mkdir terraform-hello-world
cd terraform-hello-world
```

## 4. Terraform-Datei erstellen

Jetzt erstellen wir eine Datei, die beschreibt, was wir bauen wollen:

```bash
nano main.tf
```

Füge diesen Inhalt ein:

```hcl
# Provider konfigurieren - sagt Terraform, dass wir mit AWS arbeiten
provider "aws" {
  region = "eu-central-1"  # Ändere dies zu deiner Region, falls nötig
}

# AMI finden - sucht nach dem Amazon Linux Betriebssystem-Image
data "aws_ami" "amazon_linux" {
  most_recent = true
  owners      = ["amazon"]

  filter {
    name   = "name"
    values = ["amzn2-ami-hvm-*-x86_64-gp2"]
  }

  filter {
    name   = "virtualization-type"
    values = ["hvm"]
  }
}

# Security Group erstellen - definiert Firewall-Regeln
resource "aws_security_group" "web" {
  name        = "terraform-hello-world-sg"
  description = "Erlaubt HTTP-Verkehr"

  # Erlaubt eingehenden Webverkehr (HTTP)
  ingress {
    from_port   = 80
    to_port     = 80
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]  # Von überall erreichbar
  }

  # Erlaubt allen ausgehenden Verkehr
  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"  # Alle Protokolle
    cidr_blocks = ["0.0.0.0/0"]  # Zu überall hin
  }
}

# Webserver erstellen - ein virtueller Server (EC2-Instance)
resource "aws_instance" "web" {
  ami                    = data.aws_ami.amazon_linux.id  # Verwendet das gefundene AMI
  instance_type          = "t2.micro"  # Kleine, kostengünstige Servergröße
  vpc_security_group_ids = [aws_security_group.web.id]  # Verbindet mit unserer Security Group

  # Dieses Skript läuft beim ersten Start des Servers
  user_data = <<-EOF
              #!/bin/bash
              yum update -y                                  # Aktualisiert das System
              yum install -y httpd                           # Installiert den Apache Webserver
              systemctl start httpd                          # Startet den Webserver
              systemctl enable httpd                         # Richtet automatischen Start ein
              echo "<html><body><h1>Hello World from Terraform!</h1></body></html>" > /var/www/html/index.html  # Erstellt eine einfache Webseite
              EOF

  tags = {
    Name = "terraform-hello-world"  # Name für den Server in der AWS-Konsole
  }
}

# Output für Webserver-URL - wird angezeigt, wenn Terraform fertig ist
output "website_url" {
  value = "http://${aws_instance.web.public_ip}"  # URL zur erstellten Webseite
}
```

Speichere die Datei mit STRG+O und verlasse den Editor mit STRG+X.

## 5. Terraform ausführen

Jetzt führen wir Terraform aus, um unsere Infrastruktur zu erstellen:

```bash
terraform init
```
Dieser Befehl initialisiert Terraform und lädt alle benötigten Provider herunter (in diesem Fall den AWS-Provider).

```bash
terraform plan
```
Dieser Befehl zeigt dir, was Terraform tun wird, ohne tatsächlich etwas zu verändern - wie eine Vorschau.

```bash
terraform apply
```
Dieser Befehl führt die eigentliche Erstellung der Ressourcen durch. Terraform zeigt noch einmal den Plan an und fragt dich nach einer Bestätigung.

Wenn du gefragt wirst, gib `yes` ein und drücke Enter.

Jetzt:
- Erstellt Terraform eine Security Group (Firewall-Regeln)
- Startet einen virtuellen Server (EC2-Instance)
- Installiert und konfiguriert einen Webserver
- Erstellt eine einfache Webseite

## 6. Website testen

Nach erfolgreicher Anwendung wird eine URL angezeigt, beispielsweise:
```
website_url = "http://3.123.45.67"
```

Kopiere diese URL und öffne sie in deinem Browser, um deine "Hello World"-Webseite zu sehen.

## 7. Aufräumen

Wenn du fertig bist, lösche die erstellten Ressourcen, um keine unnötigen Kosten zu verursachen:

```bash
terraform destroy
```

Gib `yes` ein, wenn du gefragt wirst.

Dieser Befehl entfernt alle Ressourcen, die von Terraform erstellt wurden, in der richtigen Reihenfolge.

## Was haben wir gelernt?

- Terraform in der AWS CloudShell zu installieren
- Eine Terraform-Konfigurationsdatei zu erstellen
- Die grundlegenden Terraform-Befehle zu verwenden: init, plan, apply, destroy
- Infrastruktur als Code zu definieren: einen Webserver mit Security Group
- AWS-Ressourcen mit Terraform zu erstellen und zu löschen

## Fehlerbehebung

**Problem: AMI nicht gefunden**

Wenn das AMI nicht gefunden wird, verwende stattdessen eine feste AMI-ID:

```hcl
resource "aws_instance" "web" {
  ami                    = "ami-0648880541a3156f7"  # Amazon Linux 2 in eu-central-1
  # Rest der Konfiguration bleibt gleich
}
```

**Problem: Security Group-Name bereits verwendet**

Falls die Security Group nicht erstellt werden kann, füge deinen Namen hinzu:

```hcl
resource "aws_security_group" "web" {
  name        = "terraform-hello-world-sg-DEINNAME"
  # Rest der Konfiguration bleibt gleich
}
```

**Problem: Instance kann nicht erstellt werden**

Versuche einen kleineren Instance-Typ (besonders wichtig in Sandbox-Umgebungen):

```hcl
resource "aws_instance" "web" {
  instance_type          = "t2.nano"  # Kleinerer Instance-Typ
  # Rest der Konfiguration bleibt gleich
}
```
