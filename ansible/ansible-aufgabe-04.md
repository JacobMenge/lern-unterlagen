# Ansible und Terraform: Infrastruktur und Konfiguration automatisieren

In dieser praxisorientierten Übung lernst du, wie Terraform und Ansible zusammenarbeiten, um eine vollständig automatisierte Infrastruktur zu erstellen. Du wirst einen NGINX-Webserver auf einer AWS EC2-Instanz bereitstellen und dabei die Stärken beider Tools kennenlernen - Terraform für die Infrastrukturbereitstellung und Ansible für die Konfiguration.

> **Hinweis:** Für diese Übung solltest du bereits mit den Grundlagen von Ansible und AWS vertraut sein. Alle benötigten AWS-Zugangsdaten sollten bereits konfiguriert sein.

## Was du in dieser Übung lernen wirst

In dieser Übung wirst du:
1. Eine EC2-Instanz mit Terraform definieren und bereitstellen
2. Terraform-Outputs für Ansible nutzbar machen
3. Ein Ansible-Playbook zur Konfiguration des NGINX-Webservers erstellen
4. Einen End-to-End-Automatisierungsworkflow implementieren
5. Best Practices für die Kombination beider Tools kennenlernen

## Der vollständige Automatisierungsablauf

Bevor wir mit der praktischen Umsetzung beginnen, ist es wichtig zu verstehen, wie der gesamte Automatisierungsablauf funktioniert:

1. **Terraform erstellt die Infrastruktur**:
   - Terraform liest die Konfigurationsdateien (*.tf) und plant die Erstellung der Ressourcen
   - Es erstellt die EC2-Instanz mit der angegebenen AMI und Konfiguration
   - Es richtet Sicherheitsgruppen ein, die den Netzwerkverkehr steuern
   - Es wartet, bis die Instanz vollständig hochgefahren ist und SSH-Zugriff möglich ist

2. **Terraform generiert Outputs**:
   - Nach der erfolgreichen Erstellung der Infrastruktur erzeugt Terraform Ausgabewerte
   - Diese Outputs enthalten wichtige Informationen wie die IP-Adresse des Servers
   - Besonders wichtig ist der Output "ansible_inventory", der die Verbindungsdaten im JSON-Format enthält

3. **Ein Zwischenskript verbindet Terraform mit Ansible**:
   - Das generate_inventory.sh-Skript wird ausgeführt
   - Es extrahiert die Terraform-Outputs und konvertiert sie in ein Ansible-Inventory
   - Dieses Inventory wird als JSON-Datei gespeichert, die Ansible lesen kann

4. **Ansible konfiguriert den Server**:
   - Ansible liest das generierte Inventory und stellt eine Verbindung zur EC2-Instanz her
   - Es führt das Playbook aus, das NGINX installiert und konfiguriert
   - Das Playbook erstellt Verzeichnisse, kopiert Dateien und startet Dienste
   - Nach Abschluss ist der Webserver vollständig konfiguriert und betriebsbereit

5. **Das Hauptskript (deploy.sh) orchestriert den gesamten Ablauf**:
   - Es führt alle oben genannten Schritte nacheinander aus
   - Es behandelt Fehler und stellt sicher, dass jeder Schritt erfolgreich abgeschlossen wird
   - Es zeigt am Ende die URL an, unter der der Webserver erreichbar ist

Dieser Workflow zeigt die perfekte Ergänzung von Terraform und Ansible:
- **Terraform** kümmert sich um die Infrastruktur (WAS bereitgestellt wird)
- **Ansible** kümmert sich um die Konfiguration (WIE es konfiguriert wird)
- Das **Verbindungsskript** sorgt für den nahtlosen Übergang zwischen beiden Tools

Der besondere Vorteil dieses Ansatzes ist, dass du mit einem einzigen Befehl (`./deploy.sh`) eine vollständig funktionsfähige, konfigurierte Infrastruktur erstellen kannst, die wiederholbar und konsistent ist - unabhängig davon, wie oft du sie ausführst.

## Voraussetzungen

Bevor du mit dieser Übung beginnst, stelle sicher, dass folgende Tools installiert sind:

- Terraform (Version 0.14 oder höher)
- Ansible (Version 2.12 oder höher)
- AWS CLI (konfiguriert mit entsprechenden Berechtigungen)
- Ein SSH-Schlüsselpaar in AWS --> https://github.com/JacobMenge/lern-unterlagen/blob/main/aws-ssh-key-einrichten.md
  > **Hinweis für WSL-Nutzer**: 
  > 
  > Der in dieser Übung angegebene SSH-Schlüsselpfad (`~/.ssh/mein-aws-schluessel.pem`) muss an deine spezifische Umgebung angepasst werden:
  > 
  > - **Bei Schlüsseln im WSL-Dateisystem**: Der Pfad könnte wie im Beispiel `~/.ssh/mein-schluessel.pem` bleiben
  > 
  > - **Bei Schlüsseln im Windows-Dateisystem**: Der Pfad muss auf das Windows-Laufwerk verweisen, z.B.:
  >   ```
  >   private_key_path = "/mnt/c/Users/DeinWindowsNutzer/.ssh/mein-schluessel.pem"
  >   ```
  >
  > Stelle sicher, dass die Berechtigungen für den Schlüssel korrekt gesetzt sind. In WSL kannst du dies mit folgendem Befehl tun:
  > ```bash
  > chmod 600 /pfad/zu/deinem/schluessel.pem
  > ```

Du kannst die installierten Versionen mit folgenden Befehlen überprüfen:

```bash
terraform --version
ansible --version
aws --version
```

Stelle außerdem sicher, dass deine AWS-Anmeldeinformationen korrekt konfiguriert sind:

```bash
aws configure
# Oder überprüfe vorhandene Konfiguration:
aws sts get-caller-identity
```

## 1. Projektstruktur einrichten

Beginnen wir mit der Erstellung einer geeigneten Verzeichnisstruktur für unser Projekt:

```bash
# Erstelle und wechsle in das Projektverzeichnis
mkdir -p ~/terraform-ansible-demo
cd ~/terraform-ansible-demo

# Erstelle Unterverzeichnisse
mkdir -p terraform ansible/playbooks
```

Diese Struktur trennt klar zwischen Terraform-Konfigurationen (Infrastruktur) und Ansible-Playbooks (Konfiguration), was die Lesbarkeit und Wartbarkeit des Codes verbessert.

## 2. Terraform-Konfiguration erstellen

### 2.1 Terraform-Hauptkonfiguration

Erstelle zunächst die Hauptkonfigurationsdatei:

```bash
mkdir -p terraform
nano terraform/main.tf
```

**Hinweis:** Die folgende Konfiguration verwendet Terraform Version 0.14 oder höher mit AWS Provider Version 5.x.

Füge folgenden Inhalt hinzu:

```hcl
# Provider-Konfiguration
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}

provider "aws" {
  region = "eu-central-1"  # Anpassen an deine bevorzugte Region
  # Wenn du mehrere AWS-Profile hast, kannst du hier das spezifische Profil angeben:
  # profile = "default" 
}

# Data Source für neueste Amazon Linux 2023 AMI
data "aws_ami" "amazon_linux_2023" {
  most_recent = true
  owners      = ["amazon"]

  filter {
    name   = "name"
    values = ["al2023-ami-*-kernel-6.1-x86_64"]
  }

  filter {
    name   = "virtualization-type"
    values = ["hvm"]
  }
}
```

**Warum ein AMI Data Source statt einer festen AMI-ID?**

Die Verwendung eines Data Source für die AMI-Auswahl bietet entscheidende Vorteile:
- **Automatische Aktualisierung**: Deine Infrastruktur nutzt immer die neueste AMI-Version mit Sicherheitspatches
- **Portabilität**: Dein Code funktioniert regionsübergreifend ohne Anpassungen
- **Wartbarkeit**: Kein manuelles Nachschlagen und Aktualisieren von AMI-IDs mehr nötig
- **Reproduzierbarkeit**: Eindeutige Filterkriterien statt kryptischer IDs machen deinen Code verständlicher

In Produktionsumgebungen könntest du zusätzlich nach bestimmten AMI-Tags oder Versionsnummern filtern, um noch mehr Kontrolle über die verwendeten Images zu haben.

Füge auch diesen code hier noch in die main.tf hinzu:

```hcl
# Ressource: EC2-Instanz
resource "aws_instance" "web_server" {
  ami                    = data.aws_ami.amazon_linux_2023.id
  instance_type          = "t3.micro"  # Free Tier-berechtigt, bessere Performance als t2.micro
  vpc_security_group_ids = [aws_security_group.web_sg.id]
  key_name               = var.key_name

  tags = {
    Name = "terraform-ansible-demo"
    Environment = "demo"
    ManagedBy = "terraform"
    CreatedAt = formatdate("YYYY-MM-DD", timestamp())
  }

  # Warten auf SSH-Verfügbarkeit, bevor die Bereitstellung als abgeschlossen gilt
  provisioner "remote-exec" {
    inline = ["echo 'SSH is up and running!'"]

    connection {
      type        = "ssh"
      user        = "ec2-user"
      private_key = file(var.private_key_path)
      host        = self.public_ip
    }
  }
}

# Ressource: Security Group für Webserver
resource "aws_security_group" "web_sg" {
  name        = "web-server-sg"
  description = "Security group for web server"

  # SSH-Zugriff
  ingress {
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]  # In produktiven Umgebungen einschränken!
  }

  # HTTP-Zugriff
  ingress {
    from_port   = 80
    to_port     = 80
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  # Ausgehenden Verkehr erlauben
  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }
}
```

Die wichtigsten Aspekte dieser Konfiguration:

- Wir verwenden einen **Data Source** für die AMI, um automatisch die neueste Amazon Linux 2023-Version zu finden
- Die EC2-Instanz wird mit einer **t3.micro**-Instance erstellt (Free Tier-berechtigt mit besserer Performance)
- Eine **Security Group** erlaubt SSH (Port 22) und HTTP (Port 80) Zugriff
- Der **Remote-Exec Provisioner** stellt sicher, dass SSH verfügbar ist, bevor Terraform die Bereitstellung als abgeschlossen meldet

### 2.2 Variablendefinitionen

Erstelle eine Datei für Variablendefinitionen:

```bash
nano terraform/variables.tf
```

```hcl
variable "key_name" {
  description = "Name des SSH-Schlüsselpaars für EC2"
  type        = string
}

variable "private_key_path" {
  description = "Pfad zur privaten SSH-Schlüsseldatei"
  type        = string
}
```

Diese Variablen ermöglichen die flexible Konfiguration des SSH-Zugriffs ohne Änderung des Hauptcodes.

### 2.3 Output-Definitionen

Erstelle eine Datei für Output-Werte:

```bash
nano terraform/outputs.tf
```

```hcl
# Output: Öffentliche IP-Adresse des Webservers
output "web_server_public_ip" {
  value = aws_instance.web_server.public_ip
}

# Output: Instance-ID
output "instance_id" {
  value = aws_instance.web_server.id
}

# Output: Ansible-Inventory im JSON-Format
output "ansible_inventory" {
  value = {
    webservers = {
      hosts = {
        web = {
          ansible_host = aws_instance.web_server.public_ip
          ansible_user = "ec2-user"
        }
      }
    }
  }
}
```

**Die Brücke zwischen Terraform und Ansible verstehen**

Der `ansible_inventory`-Output ist der entscheidende Verbindungspunkt zwischen beiden Tools:

- Die Struktur folgt exakt dem Ansible JSON-Inventory-Format, mit Gruppen (`webservers`), Hosts und Variablen
- Durch die strukturierte JSON-Ausgabe können wir dynamisch generierte Infrastruktur nahtlos an Ansible übergeben
- Jede Instanz bekommt automatisch die richtigen Verbindungsparameter zugewiesen
- Dieser Ansatz skaliert hervorragend: Ob 1 oder 100 Server - das Prinzip bleibt gleich

In größeren Umgebungen würde dieses Inventory noch mehr Details enthalten: verschiedene Host-Gruppen (z.B. webserver, database), zusätzliche Verbindungsvariablen oder auch gruppenspezifische Ansible-Variablen. Die Grundstruktur bleibt jedoch immer gleich, was die Integration vorhersehbar und wartbar macht.

### 2.4 Terraform-Variablenwerte festlegen

```bash
nano terraform/terraform.tfvars
```

```hcl
key_name         = "mein-aws-schluessel"  # Name deines AWS-Schlüsselpaars
private_key_path = "~/.ssh/terraform-ansible-demo.pem"  # Pfad zu deinem privaten Schlüssel! ändere ihn, falls er bei dir wo andedrs liegt oder anders heißt
```

**Wichtig:** 
1. Ersetze die Werte durch deine eigenen SSH-Schlüsselinformationen
2. Stelle sicher, dass der genannte Schlüssel in der AWS-Region existiert, die du im Provider definiert hast
3. Setze die korrekten Berechtigungen für deinen SSH-Schlüssel:
   ```bash
   chmod 600 ~/.ssh/mein-aws-schluessel.pem
   ```

Wenn du mehrere AWS-Profile verwendest, kannst du in der `main.tf` das spezifische Profil angeben:
```hcl
provider "aws" {
  region  = "eu-central-1"
  profile = "default"  # Ändere zu deinem AWS CLI Profil
}
```

## 3. Ansible-Konfiguration erstellen

### 3.1 Ansible-Konfigurationsdatei

```bash
nano ansible/ansible.cfg
```

```ini
[defaults]
host_key_checking = False
inventory = ./inventory.json
interpreter_python = auto_silent
deprecation_warnings = False  # Unterdrücke Warnungen zu veralteten Features
```

Diese Konfiguration optimiert Ansible für die Verwendung mit temporären Cloud-Instanzen und verbessert die Stabilität der Verbindungen.

### 3.2 Dynamisches Inventory-Skript

Das folgende Skript ist ein entscheidender Bestandteil unserer Integration - es fungiert als "Übersetzer" zwischen Terraform und Ansible:

```bash
nano ansible/generate_inventory.sh
```

```bash
#!/bin/bash

# Prüfe, ob jq installiert ist
if ! command -v jq &> /dev/null; then
  echo "jq ist nicht installiert. Versuche jq zu installieren..."
  if command -v apt-get &> /dev/null; then
    sudo apt-get update && sudo apt-get install -y jq
  elif command -v yum &> /dev/null; then
    sudo yum install -y jq
  elif command -v dnf &> /dev/null; then
    sudo dnf install -y jq
  else
    echo "Paketmanager nicht erkannt."
    echo "Erstelle Inventory manuell ohne jq..."
    
    # Wechsle ins Terraform-Verzeichnis
    cd ../terraform
    
    # Hole die IP-Adresse direkt
    SERVER_IP=$(terraform output -raw web_server_public_ip)
    
    # Erstelle ein einfaches JSON-Inventory manuell
    cat > ../ansible/inventory.json << EOF
{
  "webservers": {
    "hosts": {
      "web": {
        "ansible_host": "$SERVER_IP",
        "ansible_user": "ec2-user"
      }
    }
  }
}
EOF
    
    echo "Ansible-Inventory wurde manuell erstellt."
    exit 0
  fi
fi

# Wenn jq installiert ist oder wurde, verwende es
# Wechsle ins Terraform-Verzeichnis
cd ../terraform

# Hole die Ausgabe des Ansible-Inventory und speichere sie als JSON
terraform output -json ansible_inventory | jq > ../ansible/inventory.json

echo "Ansible-Inventory wurde generiert aus Terraform-Output."
```

Mache das Skript ausführbar:

```bash
chmod +x ansible/generate_inventory.sh
```

**Warum diese Integrationsmethode?**

Dieses Skript ist der Schlüssel zur nahtlosen Integration zwischen Terraform und Ansible. Es gibt verschiedene Möglichkeiten, diese Tools zu verbinden:

1. **Manuelle Konfiguration**: IP-Adressen von Terraform manuell kopieren und in Ansible eintragen - fehleranfällig und nicht automatisierbar
2. **Dynamische Inventories**: Ansible kann AWS direkt abfragen - erfordert aber zusätzliche Konfiguration und AWS-Berechtigungen
3. **Terraform-Outputs zu Ansible-Inventory**: Unser Ansatz - einfach, flexibel und ohne zusätzliche Abhängigkeiten

Unser Skript bietet mehrere Vorteile:
- Es hat integrierte Fehlerbehandlung (Fallback für fehlende Tools)
- Es ist sehr portabel (funktioniert überall, wo Bash verfügbar ist)
- Du kannst es leicht an andere Inventory-Formate anpassen
- Die manuelle Inventory-Generierung als Fallback macht die Lösung robust

In größeren Umgebungen würde man vielleicht auf spezialisierte Tools wie Ansible Tower/AWX oder Terraform Cloud setzen, aber unser Ansatz bietet eine perfekte Balance zwischen Einfachheit und Funktionalität für die meisten Anwendungsfälle.

### 3.3 NGINX-Konfigurationsplaybook

```bash
nano ansible/playbooks/setup_nginx.yml
```

```yaml
---
# Playbook zur Konfiguration eines NGINX-Webservers

- name: Konfiguriere NGINX auf den Webservern
  hosts: webservers
  become: yes

  vars:
    nginx_site_template: "templates/nginx_site.conf.j2"
    website_name: "terraform-ansible-demo"
    website_domain: "example.com"  # Nur für Demonstrationszwecke

  tasks:
    - name: Aktualisiere Paketlisten
      dnf:
        update_cache: yes
      changed_when: false

    - name: Installiere EPEL-Repository
      dnf:
        name: "https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm"
        state: present
        disable_gpg_check: yes
      register: epel_result
      until: epel_result is succeeded
      retries: 3
      delay: 5
      ignore_errors: yes
      
    - name: Alternative Methode - Aktiviere Amazon Extras (falls vorhanden)
      shell: amazon-linux-extras enable nginx1
      register: extras_result
      changed_when: extras_result.rc == 0
      failed_when: false
      when: epel_result is failed

    - name: Installiere NGINX
      dnf:
        name: nginx
        state: present

    - name: Erstelle Webserver-Verzeichnis
      file:
        path: "/var/www/{{ website_name }}"
        state: directory
        owner: nginx
        group: nginx
        mode: '0755'

    - name: Erstelle Beispiel-Index-Seite
      copy:
        dest: "/var/www/{{ website_name }}/index.html"
        content: |
          <!DOCTYPE html>
          <html>
          <head>
            <title>Erfolgreich mit Terraform + Ansible!</title>
            <style>
              body {
                width: 70%;
                margin: 0 auto;
                font-family: Tahoma, Verdana, Arial, sans-serif;
              }
              .container {
                margin-top: 40px;
                padding: 20px;
                background-color: #f7f7f7;
                border-radius: 5px;
                box-shadow: 0 1px 3px rgba(0,0,0,0.12), 0 1px 2px rgba(0,0,0,0.24);
              }
              h1, h2 {
                color: #0066cc;
              }
              .tech-info {
                background-color: #e6f3ff;
                padding: 15px;
                border-radius: 3px;
                margin-top: 20px;
              }
            </style>
          </head>
          <body>
            <div class="container">
              <h1>Terraform + Ansible Demo</h1>
              <p>Diese Seite wurde automatisch bereitgestellt durch:</p>
              <ul>
                <li>Terraform für die Infrastrukturerstellung</li>
                <li>Ansible für die Serverkonfiguration</li>
              </ul>
              
              <div class="tech-info">
                <h2>Server-Informationen:</h2>
                <p>Hostname: {{ ansible_hostname }}</p>
                <p>IP-Adresse: {{ ansible_default_ipv4.address }}</p>
                <p>Betriebssystem: {{ ansible_distribution }} {{ ansible_distribution_version }}</p>
                <p>Bereitgestellt am: {{ ansible_date_time.date }}</p>
              </div>
            </div>
          </body>
          </html>
        owner: nginx
        group: nginx
        mode: '0644'

    - name: Konfiguriere NGINX-Default-Site
      copy:
        dest: /etc/nginx/conf.d/default.conf
        content: |
          server {
              listen 80 default_server;
              listen [::]:80 default_server;
              
              root /var/www/{{ website_name }};
              index index.html;
              
              server_name _;
              
              location / {
                  try_files $uri $uri/ =404;
              }
          }
        owner: root
        group: root
        mode: '0644'
      notify: Restart NGINX

    - name: Starte NGINX und aktiviere beim Systemstart
      service:
        name: nginx
        state: started
        enabled: yes

  handlers:
    - name: Restart NGINX
      service:
        name: nginx
        state: restarted
```

**Idempotenz in der Praxis - Das Geheimnis erfolgreicher Automatisierung**

Das NGINX-Playbook demonstriert perfekt, was Idempotenz in der Praxis bedeutet:

Idempotenz beschreibt die Eigenschaft einer Operation, dass sie - egal wie oft ausgeführt - immer zum gleichen Ergebnis führt. Dies ist ein fundamentales Prinzip bei der Infrastrukturautomatisierung:

- Wenn du das Playbook zum ersten Mal ausführst, installiert es NGINX, erstellt Verzeichnisse und Konfigurationen
- Bei wiederholter Ausführung prüft Ansible den aktuellen Zustand und führt nur die Änderungen durch, die notwendig sind
- Wenn bereits alles wie gewünscht konfiguriert ist, werden keine Änderungen vorgenommen
- Diese "Status-Prüfung" macht Ansible zu einem zuverlässigen Werkzeug für die Konfigurationsverwaltung

Beachte besonders den `notify`-Mechanismus im Playbook: NGINX wird nur neu gestartet, wenn sich die Konfiguration tatsächlich geändert hat - ein elegantes Beispiel für effiziente Idempotenz.

In nicht-idempotenten Systemen müsstest du komplexe Bedingungen einbauen, um zu verhindern, dass bereits vorhandene Konfigurationen überschrieben oder Dienste unnötig neu gestartet werden. Ansible nimmt dir diese Arbeit ab und macht die Automatisierung zuverlässiger.

## 4. End-to-End-Workflow-Skript erstellen

Jetzt erstellen wir ein Hauptskript, das den gesamten Workflow orchestriert.

**Hinweis:** Das folgende Skript setzt eine Unix/Linux-Umgebung voraus. Für Windows-Benutzer empfehlen wir die Verwendung von WSL (Windows Subsystem for Linux) oder Git Bash.

```bash
nano deploy.sh
```

```bash
#!/bin/bash
set -e  # Beende bei Fehlern

# Farbkonstanten für bessere Lesbarkeit
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
BLUE='\033[0;34m'
NC='\033[0m'  # No Color

echo -e "${YELLOW}=== Terraform + Ansible Demo-Deployment ===${NC}"

# 1. Terraform Initialisierung und Bereitstellung
echo -e "${BLUE}1. Starte Infrastrukturbereitstellung mit Terraform...${NC}"
cd terraform
terraform init
terraform apply -auto-approve
cd ..

# 2. Warte auf vollständige SSH-Verfügbarkeit und teste Verbindung
echo -e "${BLUE}2. Warte auf vollständige SSH-Verfügbarkeit...${NC}"

# Setze absolute Pfade für sicherere Ausführung
SERVER_IP=$(cd "$(dirname "$0")/terraform" && terraform output -raw web_server_public_ip)
SSH_KEY_PATH=$(cd "$(dirname "$0")/terraform" && terraform output -raw private_key_path | tr -d '"')

echo "Server IP: $SERVER_IP"
echo "SSH Key: $SSH_KEY_PATH"

# Robusterer SSH-Bereitschaftstest
for i in {1..12}; do  # 2 Minuten Timeout (12 x 10 Sekunden)
  echo "Versuche SSH-Verbindung (Versuch $i/12)..."
  if ssh -o StrictHostKeyChecking=no -o ConnectTimeout=5 -i "$SSH_KEY_PATH" ec2-user@"$SERVER_IP" "echo SSH-Verbindung erfolgreich"; then
    echo "SSH ist bereit!"
    break
  fi
  
  if [ $i -eq 12 ]; then
    echo "Timeout beim Warten auf SSH. Bitte überprüfe die Infrastruktur manuell."
    exit 1
  fi
  
  echo "Warte 10 Sekunden vor dem nächsten Versuch..."
  sleep 10
done

# 3. Generiere Ansible-Inventory aus Terraform-Output
echo -e "${BLUE}3. Generiere Ansible-Inventory aus Terraform-Output...${NC}"
cd ansible
./generate_inventory.sh

# 4. Führe Ansible-Playbook aus
echo -e "${BLUE}4. Konfiguriere Server mit Ansible...${NC}"
ansible-playbook playbooks/setup_nginx.yml
cd ..

# 5. Hole Server-IP für den Zugriff
SERVER_IP=$(cd terraform && terraform output -raw web_server_public_ip)

echo -e "${GREEN}=== Deployment erfolgreich abgeschlossen! ===${NC}"
echo -e "Webserver erreichbar unter: http://${SERVER_IP}"
echo -e "Zugriff per SSH: ssh -i ~/.ssh/dein-schluessel.pem ec2-user@${SERVER_IP}"
echo -e "${YELLOW}HINWEIS: Vergiss nicht, die Ressourcen nach dem Test zu löschen:${NC}"
echo -e "cd terraform && terraform destroy -auto-approve"
```

Mache das Skript ausführbar:

```bash
chmod +x deploy.sh
```

## 5. Führe den End-to-End-Workflow aus

Jetzt können wir den gesamten Workflow mit einem einzigen Befehl ausführen:

```bash
# Setze Ausführungsrechte für das Script
chmod +x deploy.sh

# Führe das Deployment aus
./deploy.sh
```

**Wichtiger Hinweis:** Bei der ersten Ausführung kann der Prozess 5-10 Minuten dauern, da Terraform den AWS-Provider herunterladen und die Infrastruktur erstellen muss. Sei geduldig!

**Die Bedeutung der Workflow-Orchestrierung**

Das deploy.sh-Skript erfüllt eine zentrale Rolle in unserem Automatisierungsansatz - es orchestriert den gesamten Ablauf und sorgt für einen reibungslosen Übergang zwischen den einzelnen Phasen:

- Es implementiert einen **sequentiellen Prozess**, der sicherstellt, dass jeder Schritt erfolgreich abgeschlossen ist, bevor der nächste beginnt
- Die **SSH-Verfügbarkeitsprüfung** ist besonders wichtig, da sie den Übergang von der Infrastrukturerstellung zur Konfiguration markiert
- Die **Fehlerbehandlung** mit Timeouts und Retries macht den Prozess robust gegen temporäre Netzwerkprobleme
- Die **visuelle Fortschrittsanzeige** mit farbigen Ausgaben gibt sofortiges Feedback zum aktuellen Status

In professionellen DevOps-Umgebungen würde dieser Workflow typischerweise in einer CI/CD-Pipeline implementiert werden, etwa mit GitHub Actions, GitLab CI, Jenkins oder AWS CodePipeline. Dort würden zusätzliche Schritte wie automatische Tests, Sicherheitsprüfungen und mehrstufige Genehmigungsprozesse hinzukommen.

Das Script zeigt dir, dass ein Deployment nicht nur aus den Tools selbst besteht, sondern auch aus der Art und Weise, wie sie zusammenarbeiten und wie der Workflow gestaltet ist.

## 6. Änderungen demonstrieren

### 6.1 Infrastrukturänderung mit Terraform

Um zu demonstrieren, wie Änderungen an der Infrastruktur vorgenommen werden, ändere die EC2-Instance-Tags:

```bash
nano terraform/main.tf
```

Ändere die Tags-Sektion:

```hcl
  tags = {
    Name = "terraform-ansible-demo"
    Environment = "staging"  # Geändert von "demo" zu "staging"
    ManagedBy = "terraform"
    UpdatedAt = formatdate("YYYY-MM-DD", timestamp())  # Dynamisches Datum
  }
```

Führe Terraform erneut aus:

```bash
cd terraform
terraform apply -auto-approve
cd ..
```

**Was passiert hier? Die Idempotenz von Terraform**

Beobachte die Terraform-Ausgabe genau - du wirst sehen, dass nur die Tags aktualisiert werden, ohne die Instanz neu zu erstellen. Dies zeigt die Idempotenz von Terraform in Aktion:

- Terraform erstellt ein **Modell des gewünschten Zustands** basierend auf deiner Konfiguration
- Es vergleicht diesen mit dem **aktuellen Zustand** (gespeichert in terraform.tfstate)
- Es berechnet den **minimalen Satz an Änderungen**, der notwendig ist, um vom aktuellen zum gewünschten Zustand zu gelangen
- Es führt nur diese Änderungen durch, nicht mehr und nicht weniger

Diese Eigenschaft macht Terraform besonders wertvoll für die Infrastrukturverwaltung:
- Ressourcen werden nicht unnötig neu erstellt (was zu Ausfallzeiten führen könnte)
- Der Prozess ist effizienter und schneller
- Das Risiko von Datenverlusten oder Konfigurationsfehlern wird minimiert

Idempotenz ist nicht nur ein theoretisches Konzept, sondern ein praktisches Werkzeug, das dir hilft, Infrastruktur sicher und vorhersehbar zu verwalten.

### 6.2 Konfigurationsänderung mit Ansible

Um zu demonstrieren, wie Änderungen an der Konfiguration vorgenommen werden, aktualisiere das Playbook:

```bash
nano ansible/playbooks/setup_nginx.yml
```

Ändere die Beispiel-Index-Seite:

```yaml
    - name: Erstelle Beispiel-Index-Seite
      copy:
        dest: "/var/www/{{ website_name }}/index.html"
        content: |
          <!DOCTYPE html>
          <html>
          <head>
            <title>Erfolgreich mit Terraform + Ansible!</title>
            <!-- Restlicher HTML-Code bleibt gleich -->
          </head>
          <body>
            <div class="container">
              <h1>Terraform + Ansible Demo (Aktualisiert)</h1>
              <p>Diese Seite wurde automatisch bereitgestellt und aktualisiert durch:</p>
              <!-- Restlicher HTML-Code bleibt gleich -->
              
              <div class="tech-info">
                <!-- Bestehender Info-Block bleibt gleich -->
              </div>
              
              <div class="update-info" style="background-color: #e8f5e9; padding: 15px; border-radius: 3px; margin-top: 20px;">
                <h2>Update-Informationen:</h2>
                <p>Letzte Aktualisierung: {{ ansible_date_time.iso8601 }}</p>
                <p>Dies demonstriert, wie einfach Konfigurationsänderungen mit Ansible sind!</p>
              </div>
            </div>
          </body>
          </html>
```

Führe nur das Ansible-Playbook erneut aus:

```bash
cd ansible
ansible-playbook playbooks/setup_nginx.yml
cd ..
```

Beobachte, wie Ansible nur die geänderte Konfiguration aktualisiert, ohne die Infrastruktur zu beeinflussen. Dies demonstriert die idempotente Natur von Ansible.

## 7. Ressourcen bereinigen

Nach Abschluss der Übung solltest du die AWS-Ressourcen löschen, um unnötige Kosten zu vermeiden:

```bash
cd terraform
terraform destroy -auto-approve
cd ..
```

Dieses Kommando entfernt alle von Terraform erstellten Ressourcen aus deinem AWS-Konto.

**Wichtiger Hinweis:** Die Terraform-State-Datei (`terraform.tfstate`) wird lokal gespeichert und ist wichtig, um die Ressourcen korrekt zu verwalten. Für Produktionsumgebungen wäre ein Remote-State empfehlenswert, aber für diese Übung ist die lokale Speicherung ausreichend.

**Der vollständige Lebenszyklus der Infrastruktur**

Der `terraform destroy`-Befehl schließt einen wichtigen Aspekt der Infrastructure as Code ab: den vollständigen Lebenszyklus von Ressourcen. Diese Fähigkeit, Infrastruktur sowohl zu erstellen als auch sauber zu entfernen, bringt mehrere wesentliche Vorteile:

- **Kostenkontrolle**: Ressourcen werden nur für die tatsächlich benötigte Zeit bereitgestellt
- **Umweltbereinigung**: Keine "vergessenen" Ressourcen, die unnötig weiterlaufen
- **Reproduzierbarkeit**: Du kannst jederzeit eine frische Umgebung erstellen und nach Gebrauch wieder entfernen
- **Testumgebungen**: Ideal für zeitlich begrenzte Test- oder Entwicklungsumgebungen

In der DevOps-Praxis werden oft kurzlebige Umgebungen eingesetzt, die nur für eine bestimmte Zeit existieren:
- Temporäre Testumgebungen für Feature-Branches
- Umgebungen für automatisierte Tests
- On-Demand-Entwicklungsumgebungen
- Sandbox-Umgebungen zum Experimentieren

Diese "Ephemeral Infrastructure"-Ansätze wären ohne Tools wie Terraform kaum praktikabel, da der manuelle Erstellungs- und Bereinigungsaufwand zu hoch wäre.

## 8. Fehlerbehebung

Bei der Durchführung dieser Übung könnten folgende Probleme auftreten:

### Terraform-Fehler

1. **AWS-Authentifizierungsprobleme**:
   ```
   Error: No valid credential sources found
   ```
   **Lösung**: Führe `aws configure` aus und stelle sicher, dass deine AWS-Anmeldeinformationen korrekt sind.

2. **Resource-Timing-Probleme**:
   ```
   Error: timeout while waiting for state to become 'running'
   ```
   **Lösung**: Dieses Problem kann bei Netzwerk- oder AWS-Kapazitätsproblemen auftreten. Führe `terraform apply` erneut aus.

3. **Terraform-Provider-Fehler**:
   ```
   Error: Provider configuration not present
   ```
   **Lösung**: Führe `terraform init` aus, um den AWS-Provider herunterzuladen.

### Ansible-Fehler

1. **SSH-Verbindungsprobleme**:
   ```
   fatal: [web]: UNREACHABLE! => {"changed": false, "msg": "Failed to connect to the host", "unreachable": true}
   ```
   **Lösung**: Überprüfe Security Group-Regeln und SSH-Schlüsselberechtigungen. Stelle sicher, dass die Instanz läuft.

2. **NGINX-Installationsprobleme**:
   ```
   fatal: [web]: FAILED! => {"changed": false, "msg": "No package nginx available."}
   ```
   **Lösung**: Das Playbook enthält Fallback-Mechanismen für die NGINX-Installation. Wenn der Fehler dennoch auftritt, führe folgenden Befehl manuell auf der Instanz aus:
   ```
   sudo amazon-linux-extras install nginx1 -y
   ```

3. **Berechtigungsprobleme für die SSH-Schlüsseldatei**:
   ```
   @@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
   @         WARNING: UNPROTECTED PRIVATE KEY FILE!          @
   @@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
   ```
   **Lösung**: Setze die Berechtigungen des SSH-Schlüssels:
   ```
   chmod 600 ~/.ssh/mein-aws-schluessel.pem
   ```

**Systematisches Debugging in IaC-Umgebungen**

Bei der Arbeit mit Infrastruktur als Code und mehreren Tools ist ein systematischer Debugging-Ansatz entscheidend:

1. **Schichtweises Debugging**: Gehe von unten nach oben vor
   - Prüfe zuerst die Netzwerkverbindung und SSH-Zugriff
   - Dann Berechtigungen und Ressourcenverfügbarkeit
   - Schließlich die eigentliche Konfiguration

2. **Log-Analyse verstehen**:
   - Terraform-Logs zeigen den Zustand der Infrastruktur
   - Ansible-Logs zeigen den Zustand der Konfiguration
   - AWS-Konsole oder CloudWatch kann zusätzliche Einblicke liefern

3. **Testen mit minimalen Änderungen**:
   - Füge einzelne Komponenten hinzu, um Fehlerquellen zu isolieren
   - Verwende `terraform plan` und Ansible-Optionen wie `--check` oder `--diff`

4. **Grundursachenanalyse**: 
   - Frage immer "Warum ist dieser Fehler aufgetreten?" statt nur die Symptome zu behandeln
   - Suche nach Mustern bei wiederkehrenden Problemen

In größeren Umgebungen würden zusätzliche Monitoring- und Logging-Tools eingesetzt, um Probleme proaktiv zu erkennen und zu beheben, bevor sie kritisch werden.
