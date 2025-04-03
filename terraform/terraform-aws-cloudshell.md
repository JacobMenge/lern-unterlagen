# Terraform Hello World in AWS CloudShell

Dieses Tutorial zeigt dir, wie du mit Terraform in der AWS CloudShell schnell einen einfachen Webserver erstellst.

## Was werden wir tun?

- Terraform in der AWS CloudShell installieren
- Mit Terraform eine EC2-Instance (virtuellen Server) erstellen
- Auf diesem Server einen einfachen Webserver einrichten
- Eine "Hello World"-Webseite anzeigen
- Alles wieder aufräumen, wenn wir fertig sind

> **Hinweis für die Praxis**: In professionellen Umgebungen verwendet man Terraform normalerweise nur für die Infrastruktur (Server, Netzwerke, etc.) und separate Tools wie Ansible für die Konfiguration und Software-Installation. In diesem einfachen Tutorial kombinieren wir beides, um dir einen schnellen Einstieg zu ermöglichen.

## 1. CloudShell starten und Region prüfen

Die AWS CloudShell ist eine Kommandozeile direkt im Browser, mit der du AWS-Ressourcen verwalten kannst.

1. Melde dich bei deiner AWS-Sandbox an: [https://techstarter-sandboxes.awsapps.com/start](https://techstarter-sandboxes.awsapps.com/start)

2. **Wichtig: Prüfe deine AWS-Region** in der oberen rechten Ecke der AWS-Konsole (z.B. "Frankfurt" oder "Ireland"). Diese Information brauchst du später.

3. Klicke auf das CloudShell-Symbol in der oberen Navigationsleiste (>_)

4. Warte etwa 30 Sekunden, bis die CloudShell vollständig geladen ist.

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

Du solltest eine Ausgabe wie `Terraform v1.6.6` sehen.

## 3. Region bestimmen

Wir müssen die richtige AWS-Region für unseren Code verwenden. Führe diesen Befehl aus, um deine aktuelle Region zu ermitteln:

```bash
aws configure get region
```

Die Ausgabe (z.B. `eu-central-1` für Frankfurt oder `eu-west-1` für Irland) notieren wir uns für den nächsten Schritt.

Falls der Befehl keine Ausgabe liefert, kannst du die Region in der AWS-Konsole oben rechts ablesen und in eine der folgenden Regionen übersetzen:
- Frankfurt = eu-central-1
- Irland = eu-west-1
- London = eu-west-2
- Paris = eu-west-3
- N. Virginia = us-east-1

## 4. Projekt erstellen

Wir erstellen ein Verzeichnis für unser Terraform-Projekt:

```bash
mkdir terraform-hello-world
cd terraform-hello-world
```

## 5. Terraform-Datei erstellen

Jetzt erstellen wir eine Datei, die beschreibt, was wir bauen wollen:

```bash
nano main.tf
```

Füge diesen Inhalt ein (ersetze "eu-central-1" mit deiner Region aus Schritt 3):

```hcl
# Provider konfigurieren - sagt Terraform, dass wir mit AWS arbeiten
provider "aws" {
  region = "eu-central-1"  # WICHTIG: Ersetze dies mit deiner Region aus Schritt 3
}

# Aktuelles Datum/Zeit für eindeutige Namen generieren
locals {
  timestamp = formatdate("YYMMDDhhmmss", timestamp())
}

# AMI finden - sucht nach dem Amazon Linux Betriebssystem-Image
# Wir bieten zwei Optionen: Amazon Linux 2 oder Amazon Linux 2023
# Wenn eine Option nicht funktioniert, kannst du die anderen ausprobieren

# Option 1: Amazon Linux 2 (funktioniert in den meisten Regionen)
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
  name        = "terraform-hello-sg-${local.timestamp}"  # Eindeutiger Name mit Zeitstempel
  description = "Erlaubt HTTP-Verkehr"

  # Erlaubt eingehenden Webverkehr (HTTP)
  ingress {
    from_port   = 80            # Port 80 ist der Standard-HTTP-Port
    to_port     = 80            # Gleicher Port
    protocol    = "tcp"         # TCP-Protokoll für HTTP
    cidr_blocks = ["0.0.0.0/0"]  # Von überall erreichbar (0.0.0.0/0 bedeutet "alle IP-Adressen")
  }

  # Erlaubt allen ausgehenden Verkehr
  egress {
    from_port   = 0             # Alle Ports
    to_port     = 0             # Alle Ports
    protocol    = "-1"          # Alle Protokolle ("-1" ist ein Platzhalter für "alles")
    cidr_blocks = ["0.0.0.0/0"]  # Zu allen IP-Adressen
  }

  # Falls eine Standard-VPC vorhanden ist, ist keine weitere Konfiguration nötig
  # Falls nicht, können wir das später im Fehlerbehebungsabschnitt beheben
}

# Webserver erstellen - ein virtueller Server (EC2-Instance)
resource "aws_instance" "web" {
  # Verwendung des gefundenen AMI
  ami           = data.aws_ami.amazon_linux.id
  
  # Kleine, kostengünstige Servergröße
  instance_type = "t2.micro"
  
  # Verbindung mit unserer Security Group
  vpc_security_group_ids = [aws_security_group.web.id]

  # Dieses Skript läuft beim ersten Start des Servers
  user_data = <<-EOF
              #!/bin/bash
              # Aktualisiert das System (wie apt-get update)
              yum update -y
              # Installiert den Apache Webserver
              yum install -y httpd
              # Startet den Webserver
              systemctl start httpd
              # Richtet automatischen Start ein
              systemctl enable httpd
              # Erstellt eine einfache Webseite
              echo "<html><body><h1>Hello World von Terraform!</h1><p>Erfolgreich erstellt am ${local.timestamp}</p></body></html>" > /var/www/html/index.html
              EOF

  tags = {
    Name = "terraform-hello-${local.timestamp}"  # Eindeutiger Name mit Zeitstempel
  }
  
  # AWS benötigt ein paar Minuten, um den Server zu starten und zu konfigurieren
  # Wir informieren Terraform, dass es bis zu 5 Minuten warten soll
  timeouts {
    create = "5m"
  }
}

# Falls Option 1 einen Fehler verursacht, entfernen Sie die Kommentarzeichen vor dieser Option:
# Option 2: Feste AMI-IDs nach Region
/*
locals {
  # AMI-IDs für Amazon Linux 2 in verschiedenen Regionen
  ami_ids = {
    "eu-central-1" = "ami-0648880541a3156f7"  # Frankfurt
    "eu-west-1"    = "ami-0b752bf1df193a6c4"  # Irland
    "eu-west-2"    = "ami-06672d07f62285d1d"  # London
    "eu-west-3"    = "ami-0ebc281c20e89ba4b"  # Paris
    "us-east-1"    = "ami-0cff7528ff583bf9a"  # N. Virginia
  }
}

resource "aws_instance" "web" {
  # Direkte AMI-ID-Auswahl basierend auf der Region
  ami           = local.ami_ids[var.region]
  # ...Rest des Codes bleibt gleich...
}
*/

# Output für Webserver-URL - wird angezeigt, wenn Terraform fertig ist
output "website_url" {
  value = "http://${aws_instance.web.public_ip}"  # URL zur erstellten Webseite
}

# Wir zeigen auch einen Hinweis zur Wartezeit an
output "hinweis" {
  value = "Es kann 2-3 Minuten dauern, bis der Webserver vollständig gestartet ist und die Seite erreichbar ist."
}
```

Speichere die Datei mit STRG+O, bestätige mit Enter und verlasse den Editor mit STRG+X.

**WICHTIG**: Ersetze "eu-central-1" mit deiner eigenen Region, die du in Schritt 3 ermittelt hast!

## 6. Terraform ausführen

Jetzt führen wir Terraform aus, um unsere Infrastruktur zu erstellen:

```bash
terraform init
```
Dieser Befehl initialisiert Terraform und lädt den AWS-Provider herunter. ⏱️ **Dauer**: ca. 30 Sekunden

```bash
terraform plan
```
Dieser Befehl zeigt dir, was Terraform tun wird, ohne tatsächlich Änderungen vorzunehmen. ⏱️ **Dauer**: ca. 15 Sekunden

```bash
terraform apply
```
Dieser Befehl führt die eigentliche Erstellung der Ressourcen durch. ⏱️ **Dauer**: ca. 2-3 Minuten

Wenn du gefragt wirst, gib `yes` ein und drücke Enter.

Du solltest jetzt sehen, wie:
1. Die Security Group erstellt wird (wenige Sekunden)
2. Die EC2-Instance erstellt wird (1-2 Minuten)
3. Die Outputs angezeigt werden

## 7. Website testen

Nach erfolgreicher Anwendung wird eine URL angezeigt:
```
website_url = "http://34.123.45.67"
```

Kopiere diese URL und öffne sie in deinem Browser. **Wichtig**: Es kann bis zu 2-3 Minuten dauern, bis die Website erreichbar ist, da der Server noch startet und den Webserver einrichtet.

Falls die Website nach 3 Minuten noch nicht erreichbar ist, prüfe den Status deiner EC2-Instance in der AWS-Konsole:
1. Öffne die AWS-Konsole in einem neuen Tab
2. Suche nach "EC2" und klicke darauf
3. Klicke auf "Instances (running)"
4. Prüfe, ob der Status deiner "terraform-hello-..." Instance "running" ist

## 8. Aufräumen

Wenn du fertig bist, lösche die erstellten Ressourcen, um keine unnötigen Kosten zu verursachen:

```bash
terraform destroy
```

Gib `yes` ein, wenn du gefragt wirst. ⏱️ **Dauer**: ca. 1-2 Minuten

## Umgang mit CloudShell-Unterbrechungen

Falls deine CloudShell-Sitzung unterbrochen wird (nach ca. 20-30 Minuten Inaktivität):

1. Starte die CloudShell erneut und navigiere zu deinem Projektverzeichnis:
   ```bash
   cd terraform-hello-world
   ```

2. Alle Befehle (apply oder destroy) können von da aus fortgesetzt werden.
   ```bash
   terraform destroy  # Um aufzuräumen
   ```

## Fehlerbehebung

### "Error: No matching AMI found"

Wenn das AMI nicht gefunden wird, bearbeite `main.tf` und ändere den `aws_instance`-Block zur Option 2 (feste AMI-ID):

```bash
nano main.tf
```

1. Kommentiere den `data "aws_ami"` Block aus (füge `/*` am Anfang und `*/` am Ende hinzu)
2. Entferne die Kommentare bei Option 2 (entferne `/*` und `*/`)
3. Speichere die Datei (STRG+O, Enter, STRG+X)
4. Führe `terraform apply` erneut aus

### "Error: Error creating security group"

Falls die Security Group nicht erstellt werden kann, sollte der Zeitstempel im Namen bereits Konflikte vermeiden. Wenn dennoch Probleme auftreten, versuche:

```bash
cd ..
rm -rf terraform-hello-world
mkdir terraform-hello-world-neu
cd terraform-hello-world-neu
```

Dann starte mit Schritt 5 neu.

### "Error: VPC nicht gefunden"

In einigen Sandbox-Umgebungen ist die Standard-VPC möglicherweise nicht verfügbar. In diesem Fall:

1. Ermittle verfügbare VPCs und Subnets:
   ```bash
   aws ec2 describe-vpcs --query "Vpcs[*].{VpcId:VpcId,CidrBlock:CidrBlock,Name:Tags[?Key=='Name'].Value|[0]}"
   aws ec2 describe-subnets --query "Subnets[*].{SubnetId:SubnetId,VpcId:VpcId,CidrBlock:CidrBlock,AvailabilityZone:AvailabilityZone}"
   ```

2. Bearbeite `main.tf` und füge deiner EC2-Instance ein Subnet hinzu:
   ```hcl
   resource "aws_instance" "web" {
     # ...bisherige Konfiguration...
     subnet_id = "subnet-abc123"  # Ersetze mit einer SubnetId aus dem obigen Befehl
   }
   ```

### "Error: TimeoutError: timeout while waiting for state to become 'success'"

Falls Terraform mit einem Timeout endet:

1. Prüfe in der AWS-Konsole, ob die Ressourcen trotzdem erstellt wurden
2. Falls ja, führe `terraform import` aus, um sie in den Terraform-Zustand zu übernehmen
3. Falls nein, versuche es mit einem kleineren Instance-Typ:
   ```hcl
   resource "aws_instance" "web" {
     # ...bisherige Konfiguration...
     instance_type = "t2.nano"  # Noch kleinerer Instance-Typ
   }
   ```

## Was haben wir gelernt?

- Terraform in der AWS CloudShell zu installieren
- Eine Terraform-Konfigurationsdatei zu erstellen
- Die grundlegenden Terraform-Befehle zu verwenden: init, plan, apply, destroy
- Infrastruktur als Code zu definieren: einen Webserver mit Security Group
- AWS-Ressourcen mit Terraform zu erstellen und zu löschen
- Mit typischen Herausforderungen in AWS-Sandbox-Umgebungen umzugehen

## Best Practices für reale Projekte

In echten Projekten würdest du:

1. **Infrastruktur und Konfiguration trennen**: 
   - Terraform für Infrastruktur
   - Ansible, Chef oder Puppet für die Software-Installation

2. **Modularen Ansatz verwenden**:
   - Wiederverwendbare Terraform-Module erstellen
   - Verschiedene Umgebungen (Entwicklung, Test, Produktion) separat verwalten

3. **State remote speichern**:
   - S3-Bucket für gemeinsamen Zugriff auf den Terraform-Zustand
   - State-Locking mit DynamoDB

4. **Variablen und Outputs umfangreicher nutzen**:
   - Konfigurationen anpassbar gestalten
   - Wichtige Informationen als Outputs bereitstellen

Für heute hast du aber schon einen wichtigen ersten Schritt gemacht: Du hast erfolgreich Infrastruktur als Code mit Terraform erstellt! Melde dich bei mir auf Slack und schreibe die Nachricht "Moin, ich habe die Aufgabe zu Ende gelesen".
