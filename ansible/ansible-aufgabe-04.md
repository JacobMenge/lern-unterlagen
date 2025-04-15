# Ansible Praxis: Terraform und Ansible für AWS-Infrastruktur kombinieren

Diese Übung zeigt dir, wie du Terraform und Ansible zusammen nutzen kannst, um eine einfache Infrastruktur in AWS zu erstellen und zu konfigurieren. Dabei lernst du, wie diese beiden mächtigen Tools sich ergänzen und eine vollständige Infrastruktur-as-Code (IaC) Lösung bilden.

> **Hinweis:** Für diese Übung solltest du bereits mit den Grundlagen von Ansible vertraut sein und idealerweise die früheren Übungen abgeschlossen haben. Ein AWS-Konto mit entsprechenden Zugriffsrechten ist erforderlich.

## Was du in dieser Übung lernen wirst

In dieser Übung wirst du:
1. Terraform zur Erstellung von AWS-Ressourcen einsetzen
2. Ein dynamisches Ansible-Inventory aus Terraform-Outputs generieren
3. Zwei EC2-Instanzen mit unterschiedlichen Rollen konfigurieren
4. Die Kommunikation zwischen den konfigurierten Instanzen sicherstellen
5. Den gesamten Prozess mit einer einzigen Ausführung automatisieren

**Neue Konzepte in dieser Übung:**

1. **Integration von Terraform und Ansible**: Die komplementäre Nutzung beider Tools zur vollständigen Automatisierung.

2. **Dynamische Inventory-Generierung**: Automatische Erstellung des Ansible-Inventorys basierend auf den Terraform-Outputs.

3. **AWS-Ressourcen**: Praktische Erfahrung mit der Erstellung und Konfiguration von AWS-Infrastruktur.

4. **Vollständiger End-to-End-Workflow**: Von der Provisionierung bis zur Konfiguration in einem Prozess.

## Warum Terraform und Ansible kombinieren?

**Terraform** und **Ansible** haben komplementäre Stärken:

**Terraform** ist spezialisiert auf:
- Provisionierung von Infrastruktur (Instanzen, Netzwerke, Datenbanken)
- Verwaltung von Ressourcenabhängigkeiten
- Nachverfolgung des Infrastrukturzustands
- Deklarative Ressourcendefinition

**Ansible** ist besser für:
- Konfigurationsmanagement innerhalb der Instanzen
- Anwendungsbereitstellung
- Orchestrierung komplexer Aufgaben
- Beschreibung gewünschter Systemzustände

Durch die Kombination beider Tools erhältst du eine Ende-zu-Ende-Lösung für Infrastructure as Code:
1. **Terraform** erstellt die grundlegende Infrastruktur
2. **Ansible** konfiguriert die erstellten Ressourcen

## Voraussetzungen

Für diese Übung benötigst du:
- Ein AWS-Konto mit Zugriffsberechtigungen (IAM-Benutzer mit programmiertem Zugriff)
- AWS CLI, konfiguriert mit deinen Zugangsdaten
- Terraform (Version 1.5 oder höher)
- Ansible (Version 2.15 oder höher)
- Ein SSH-Schlüsselpaar zum Zugriff auf die EC2-Instanzen

**Warum brauchen wir Terraform?** Terraform ist ein Infrastructure-as-Code-Tool, das es uns ermöglicht, Cloud-Ressourcen wie EC2-Instanzen, Netzwerke und Sicherheitsgruppen in einer deklarativen Syntax zu definieren. Anstatt manuell im AWS-Management-Console zu klicken oder komplizierte AWS CLI-Befehle zu schreiben, können wir unsere gewünschte Infrastruktur in Terraform-Konfigurationsdateien beschreiben.

Überprüfe die Installation:
```bash
terraform --version
```

Stelle sicher, dass deine AWS-Anmeldeinformationen konfiguriert sind:
```bash
aws configure
```

**Was passiert bei aws configure?** Dieser Befehl fordert dich zur Eingabe deiner AWS-Zugangsdaten auf:
- AWS Access Key ID: Dein IAM-Benutzerzugriffsschlüssel
- AWS Secret Access Key: Dein geheimer Zugriffsschlüssel
- Default region name: Die AWS-Region, in der du arbeiten möchtest (z.B. eu-central-1 für Frankfurt)
- Default output format: Das Format für die Befehlsausgabe (normalerweise json)

Diese Informationen werden in `~/.aws/credentials` und `~/.aws/config` gespeichert und von Terraform automatisch verwendet, um sich bei AWS zu authentifizieren.

## 1. Projektstruktur einrichten

Beginnen wir mit der Erstellung einer passenden Verzeichnisstruktur für unser Projekt:

```bash
# Erstelle und wechsle in das Projektverzeichnis
mkdir -p ~/terraform-ansible-aws
cd ~/terraform-ansible-aws

# Erstelle Unterverzeichnisse für Terraform und Ansible
mkdir -p terraform ansible/roles/{webserver,database}/{tasks,templates,files,defaults,handlers}
```

**Erklärung der Verzeichnisstruktur:**
- `terraform/`: Enthält alle Terraform-Konfigurationsdateien
- `ansible/`: Enthält das Ansible-Playbook und die Rollen
  - `roles/webserver/`: Rolle für die Webserver-Instanz
    - `tasks/`: Enthält die auszuführenden Aufgaben für den Webserver
    - `templates/`: Enthält Jinja2-Vorlagen für dynamische Dateien wie HTML und Nginx-Konfiguration
    - `files/`: Enthält statische Dateien, die unverändert kopiert werden
    - `defaults/`: Enthält Standardvariablen für die Webserver-Rolle
    - `handlers/`: Enthält Handler, die auf Änderungen reagieren (z.B. Dienst-Neustarts)
  - `roles/database/`: Rolle für die Datenbankserver-Instanz mit ähnlicher Struktur

Die Aufteilung in zwei separate Rollen (webserver und database) ermöglicht eine klare Aufgabentrennung und fördert die Wiederverwendbarkeit. Jede Rolle enthält nur die für ihre spezifische Funktion notwendigen Aufgaben und Konfigurationen.

## 2. Terraform-Konfiguration erstellen

Wir erstellen nun die Terraform-Konfiguration, um zwei EC2-Instanzen zu provisionieren. Terraform verwendet eine deklarative Sprache, in der du den gewünschten Endzustand beschreibst, und Terraform kümmert sich darum, diesen Zustand zu erreichen.

```bash
# Wechsle in das Terraform-Verzeichnis
cd ~/terraform-ansible-aws/terraform
```

### 2.1 Hauptkonfigurationsdatei

Erstelle die Hauptkonfigurationsdatei:

```bash
nano main.tf
```

Füge folgenden Inhalt hinzu:

```hcl
# main.tf - Hauptkonfiguration für unsere AWS-Ressourcen

# Provider-Konfiguration - AWS in der Region eu-central-1 (Frankfurt)
provider "aws" {
  region = "eu-central-1"
}

# Variablen
variable "instance_type" {
  description = "EC2-Instance-Typ"
  type        = string
  default     = "t2.micro"
}

variable "ami_id" {
  description = "AMI-ID für Ubuntu 22.04 LTS in eu-central-1"
  type        = string
  default     = "ami-05b3b9303c7072b5a"  # Ubuntu 22.04 LTS (eu-central-1, April 2025)
}

variable "ssh_key_name" {
  description = "Name des SSH-Schlüssels in AWS"
  type        = string
  default     = "my-aws-key"  # Ändere dies zu deinem Schlüsselnamen
}

variable "my_ip" {
  description = "Deine IP-Adresse für SSH-Zugriff"
  type        = string
  default     = "0.0.0.0/0"  # Ersetze mit deiner IP für bessere Sicherheit, z.B. "123.123.123.123/32"
}

# VPC für unsere Infrastruktur
resource "aws_vpc" "main" {
  cidr_block           = "10.0.0.0/16"
  enable_dns_hostnames = true
  enable_dns_support   = true

  tags = {
    Name = "terraform-ansible-vpc"
  }
}

# Internet Gateway für den VPC
resource "aws_internet_gateway" "main" {
  vpc_id = aws_vpc.main.id

  tags = {
    Name = "terraform-ansible-igw"
  }
}

# Öffentliches Subnetz
resource "aws_subnet" "public" {
  vpc_id                  = aws_vpc.main.id
  cidr_block              = "10.0.1.0/24"
  map_public_ip_on_launch = true
  availability_zone       = "eu-central-1a"

  tags = {
    Name = "terraform-ansible-public-subnet"
  }
}

# Routing-Tabelle für das öffentliche Subnetz
resource "aws_route_table" "public" {
  vpc_id = aws_vpc.main.id

  route {
    cidr_block = "0.0.0.0/0"
    gateway_id = aws_internet_gateway.main.id
  }

  tags = {
    Name = "terraform-ansible-public-rt"
  }
}

# Zuordnung der Routing-Tabelle zum Subnetz
resource "aws_route_table_association" "public" {
  subnet_id      = aws_subnet.public.id
  route_table_id = aws_route_table.public.id
}

# Tipp für mehr Sicherheit: Für Produktionsumgebungen solltest du die Sicherheitsgruppen weiter einschränken und nur die spezifischen Ports öffnen, die wirklich benötigt werden.
resource "aws_security_group" "webserver" {
  name        = "webserver-sg"
  description = "Webserver-Sicherheitsgruppe"
  vpc_id      = aws_vpc.main.id

  # SSH-Zugriff von deiner IP
  ingress {
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = [var.my_ip]
  }

  # HTTP-Zugriff von überall
  ingress {
    from_port   = 80
    to_port     = 80
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  # Ausgehender Verkehr erlaubt
  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }

  tags = {
    Name = "terraform-ansible-webserver-sg"
  }
}

# Sicherheitsgruppe für Datenbankserver
resource "aws_security_group" "database" {
  name        = "database-sg"
  description = "Datenbankserver-Sicherheitsgruppe"
  vpc_id      = aws_vpc.main.id

  # SSH-Zugriff von deiner IP
  ingress {
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = [var.my_ip]
  }

  # Datenbank-Zugriff nur vom Webserver
  ingress {
    from_port       = 27017
    to_port         = 27017
    protocol        = "tcp"
    security_groups = [aws_security_group.webserver.id]
  }

  # Ausgehender Verkehr erlaubt
  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }

  tags = {
    Name = "terraform-ansible-database-sg"
  }
}

# EC2-Instanz für Webserver
resource "aws_instance" "webserver" {
  ami                    = var.ami_id
  instance_type          = var.instance_type
  key_name               = var.ssh_key_name
  vpc_security_group_ids = [aws_security_group.webserver.id]
  subnet_id              = aws_subnet.public.id

  tags = {
    Name = "terraform-ansible-webserver"
    Role = "webserver"
  }
}

# EC2-Instanz für Datenbankserver
resource "aws_instance" "database" {
  ami                    = var.ami_id
  instance_type          = var.instance_type
  key_name               = var.ssh_key_name
  vpc_security_group_ids = [aws_security_group.database.id]
  subnet_id              = aws_subnet.public.id

  tags = {
    Name = "terraform-ansible-database"
    Role = "database"
  }
}

# Ausgabe der öffentlichen IPs, um sie für Ansible zu verwenden
output "webserver_public_ip" {
  value = aws_instance.webserver.public_ip
}

output "database_public_ip" {
  value = aws_instance.database.public_ip
}

# Ausgabe der privaten IPs für die Konfiguration
output "webserver_private_ip" {
  value = aws_instance.webserver.private_ip
}

output "database_private_ip" {
  value = aws_instance.database.private_ip
}

# Output für das dynamische Ansible-Inventory
output "ansible_inventory" {
  value = <<-EOT
    [webservers]
    webserver ansible_host=${aws_instance.webserver.public_ip} private_ip=${aws_instance.webserver.private_ip}

    [databases]
    database ansible_host=${aws_instance.database.public_ip} private_ip=${aws_instance.database.private_ip}

    [all:vars]
    ansible_user=ubuntu
    ansible_ssh_private_key_file=~/.ssh/id_rsa
    ansible_python_interpreter=/usr/bin/python3
    db_host=${aws_instance.database.private_ip}
  EOT
}
```

**Erklärung der Terraform-Konfiguration:**

1. **Provider-Konfiguration**: Wir verwenden den AWS-Provider und die Region eu-central-1 (Frankfurt). Der Provider ist die Schnittstelle zwischen Terraform und der AWS-API.

2. **Variablen**: Wir definieren Variablen für wichtige Werte, die leicht angepasst werden können:
   - `instance_type`: Der Typ der EC2-Instanz (t2.micro ist im Free Tier enthalten)
   - `ami_id`: Die ID des Amazon Machine Image (Ubuntu 22.04 LTS)
   - `ssh_key_name`: Der Name deines SSH-Schlüssels in AWS
   - `my_ip`: Deine IP-Adresse für SSH-Zugriff (aus Sicherheitsgründen solltest du hier deine eigene IP angeben)

3. **Netzwerkressourcen**: 
   - `aws_vpc`: Ein virtuelles privates Netzwerk in der AWS-Cloud
   - `aws_internet_gateway`: Ermöglicht Kommunikation zwischen dem VPC und dem Internet
   - `aws_subnet`: Ein Subnetz innerhalb des VPC mit öffentlichen IP-Adressen
   - `aws_route_table` und `aws_route_table_association`: Definieren, wie der Netzwerkverkehr geroutet wird

4. **Sicherheitsgruppen**: 
   - Webserver: Erlaubt SSH von deiner IP und HTTP von überall
   - Datenbankserver: Erlaubt SSH von deiner IP und MongoDB-Zugriff (Port 27017) nur vom Webserver

5. **EC2-Instanzen**: Wir erstellen zwei Instanzen (webserver und database) mit Tags zur besseren Identifikation.

6. **Outputs**: Wir definieren Ausgaben, die nach der Bereitstellung angezeigt werden:
   - Die öffentlichen IP-Adressen für den direkten Zugriff
   - Die privaten IP-Adressen für die interne Kommunikation
   - Ein formatiertes Ansible-Inventory im INI-Format

Diese Konfiguration erstellt eine vollständige Infrastruktur mit Netzwerk, Sicherheit und Rechenressourcen, alles in einer einzigen Datei beschrieben.

### 2.2 Erstelle ein Skript zur Generierung des Ansible-Inventarys

Nun benötigen wir ein Skript, das die Terraform-Outputs in ein Ansible-Inventory umwandelt:

```bash
nano create_inventory.sh
```

Füge folgenden Inhalt hinzu:

```bash
#!/bin/bash
# Dieses Skript generiert ein Ansible-Inventory aus den Terraform-Outputs

# Wechsle in das Terraform-Verzeichnis
cd $(dirname "$0")

# Hole den Output und speichere ihn in der Inventardatei
terraform output -raw ansible_inventory > ../ansible/inventory.ini

echo "Ansible-Inventory wurde erfolgreich erstellt in: ../ansible/inventory.ini"
echo "Inhalt des Inventarys:"
cat ../ansible/inventory.ini
```

Mache das Skript ausführbar:

```bash
chmod +x create_inventory.sh
```

**Erklärung:**
- Das Skript verwendet `terraform output -raw`, um den vordefinierten Output `ansible_inventory` zu extrahieren
- Es speichert diesen Output direkt in der Datei `inventory.ini` im Ansible-Verzeichnis
- Anschließend zeigt es den Inhalt des generierten Inventarys an

Dieses Skript ist ein wichtiger Verbindungspunkt zwischen Terraform und Ansible. Es ermöglicht, dass Ansible die von Terraform erstellten Ressourcen automatisch finden und konfigurieren kann, ohne dass manuelle Eingriffe nötig sind.

## 3. Ansible-Konfiguration erstellen

Jetzt erstellen wir die Ansible-Rollen und Playbooks, um unsere Instanzen zu konfigurieren:

```bash
# Wechsle in das Ansible-Verzeichnis
cd ~/terraform-ansible-aws/ansible
```

### 3.1 Ansible-Konfigurationsdatei

Zuerst erstellen wir eine Konfigurationsdatei für Ansible, die einige Standardeinstellungen festlegt:

```bash
nano ansible.cfg
```

Füge folgenden Inhalt hinzu:

```ini
[defaults]
host_key_checking = False  # Deaktiviert die SSH-Host-Key-Prüfung (nützlich für neue Instanzen)
inventory = inventory.ini  # Pfad zur Inventory-Datei
roles_path = ./roles       # Pfad zu den Ansible-Rollen
```

Diese Konfiguration:
- Deaktiviert die Host-Key-Prüfung, sodass Ansible nicht nach Bestätigung fragt, wenn es sich zum ersten Mal mit einem Server verbindet
- Definiert den Pfad zur Inventory-Datei, die wir dynamisch aus Terraform generieren werden
- Legt den Pfad fest, in dem Ansible nach Rollen suchen soll

### 3.2 Webserver-Rolle erstellen

Jetzt erstellen wir die Rolle für den Webserver. Der Webserver wird Nginx und eine einfache HTML-Seite bereitstellen:

```bash
# Erstelle die Standardvariablen für die Webserver-Rolle
nano roles/webserver/defaults/main.yml
```

```yaml
---
# Defaults für Webserver-Rolle
http_port: 80                    # Der Port, auf dem Nginx hören soll
app_directory: /var/www/app      # Das Verzeichnis für unsere Webanwendung
app_environment: production      # Die Umgebung für unsere Anwendung (development, production, etc.)
```

**Warum verwenden wir Standardvariablen?** Standardvariablen (`defaults`) sind einer der Vorteile von Ansible-Rollen. Sie ermöglichen es uns, die Rolle mit vernünftigen Standardwerten zu definieren, die aber bei Bedarf von außen überschrieben werden können. So bleibt die Rolle flexibel und wiederverwendbar.

Erstelle die Hauptaufgabendatei für den Webserver:

```bash
nano roles/webserver/tasks/main.yml
```

```yaml
---
# Tasks für die Webserver-Rolle

# System aktualisieren
- name: Update apt cache
  apt:
    update_cache: yes
    cache_valid_time: 3600     # Cache ist für 1 Stunde gültig
  become: yes                  # Mit Root-Rechten ausführen

# Installiere erforderliche Pakete
- name: Install required packages
  apt:
    name:
      - nginx                  # Webserver
      - nodejs                 # JavaScript-Laufzeitumgebung
      - npm                    # Node.js-Paketmanager
    state: present             # Stelle sicher, dass die Pakete installiert sind
  become: yes

# Erstelle App-Verzeichnis
- name: Create application directory
  file:
    path: "{{ app_directory }}"  # Verwende die Variable aus defaults/main.yml
    state: directory             # Stelle sicher, dass das Verzeichnis existiert
    owner: www-data              # Besitzer ist der Webserver-Benutzer
    group: www-data              # Gruppe ist die Webserver-Gruppe
    mode: '0755'                 # Leserechte für alle, Schreibrechte für Besitzer
  become: yes

# Erstelle eine einfache Beispiel-App
- name: Create simple test application
  template:
    src: index.html.j2           # Quelle (Jinja2-Template)
    dest: "{{ app_directory }}/index.html"  # Ziel
    owner: www-data
    group: www-data
    mode: '0644'                 # Leserechte für alle, Schreibrechte nur für Besitzer
  become: yes
  notify: Restart Nginx          # Benachrichtige den Handler

# Konfiguriere Nginx
- name: Configure Nginx to serve the application
  template:
    src: nginx-site.conf.j2      # Quelle (Jinja2-Template)
    dest: /etc/nginx/sites-available/default  # Standardkonfigurationsdatei
    owner: root
    group: root
    mode: '0644'
  become: yes
  notify: Restart Nginx          # Benachrichtige den Handler

# Teste die Verbindung zum Datenbankserver
- name: Test connection to database server
  wait_for:
    host: "{{ db_host }}"        # Verwende die Variable aus dem Inventory
    port: 27017                  # MongoDB-Standardport
    timeout: 5                   # Timeout nach 5 Sekunden
  register: db_connection        # Speichere das Ergebnis in einer Variable
  ignore_errors: yes             # Ignoriere Fehler, falls der Test fehlschlägt

- name: Show database connection status
  debug:
    msg: "Verbindung zum Datenbankserver {{ db_host }}: {{ 'Erfolgreich' if db_connection is succeeded else 'Fehlgeschlagen' }}"
```

**Was passiert in dieser Task-Datei?** 
- Zuerst aktualisieren wir den Paket-Cache und installieren die notwendigen Pakete (Nginx, Node.js)
- Dann erstellen wir ein Verzeichnis für unsere Webanwendung und platzieren eine einfache HTML-Datei darin
- Wir konfigurieren Nginx, um diese Anwendung zu bedienen
- Schließlich testen wir die Verbindung zum Datenbankserver und zeigen das Ergebnis an

Die `notify`-Anweisungen sind besonders wichtig: Sie teilen Ansible mit, dass der "Restart Nginx"-Handler ausgeführt werden soll, aber nur, wenn der Task tatsächlich etwas geändert hat. Dies verhindert unnötige Neustarts.

Erstelle die Handler für den Webserver:

```bash
nano roles/webserver/handlers/main.yml
```

```yaml
---
# Handler für Webserver-Rolle
- name: Restart Nginx
  service:
    name: nginx
    state: restarted
  become: yes
```

Erstelle die Templates für den Webserver:

```bash
nano roles/webserver/templates/index.html.j2
```

```html
<!DOCTYPE html>
<html>
<head>
    <title>Terraform & Ansible AWS Demo</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            line-height: 1.6;
            margin: 0;
            padding: 20px;
            max-width: 800px;
            margin: 0 auto;
            color: #333;
        }
        h1 {
            color: #0066cc;
            border-bottom: 1px solid #ddd;
            padding-bottom: 10px;
        }
        .info-box {
            background-color: #f8f9fa;
            border: 1px solid #ddd;
            padding: 15px;
            border-radius: 5px;
            margin-top: 20px;
        }
        .success {
            color: green;
            font-weight: bold;
        }
        .error {
            color: red;
            font-weight: bold;
        }
    </style>
</head>
<body>
    <h1>Terraform & Ansible AWS Demo</h1>
    <p>Diese Seite wurde automatisch mit Terraform und Ansible bereitgestellt.</p>
    
    <div class="info-box">
        <h2>Server-Informationen</h2>
        <ul>
            <li><strong>Hostname:</strong> {{ ansible_hostname }}</li>
            <li><strong>IP-Adresse:</strong> {{ ansible_default_ipv4.address }}</li>
            <li><strong>Umgebung:</strong> {{ app_environment }}</li>
            <li><strong>Betriebssystem:</strong> {{ ansible_distribution }} {{ ansible_distribution_version }}</li>
            <li><strong>Datenbankserver-IP:</strong> {{ db_host }}</li>
            <li><strong>Datenbankverbindung:</strong> 
                <span class="{% if hostvars['database'] is defined %}success{% else %}error{% endif %}">
                    {% if hostvars['database'] is defined %}Verfügbar{% else %}Nicht verfügbar{% endif %}
                </span>
            </li>
        </ul>
    </div>
    
    <footer>
        <p>Bereitgestellt durch Terraform und Ansible am {{ ansible_date_time.date }}</p>
    </footer>
</body>
</html>
```

```bash
nano roles/webserver/templates/nginx-site.conf.j2
```

```nginx
server {
    listen {{ http_port }} default_server;
    listen [::]:{{ http_port }} default_server;

    root {{ app_directory }};
    index index.html index.htm;

    server_name _;

    location / {
        try_files $uri $uri/ =404;
    }
}
```

### 3.3 Datenbank-Rolle erstellen

Nun erstellen wir die Rolle für den Datenbankserver, der MongoDB bereitstellen wird:

```bash
# Erstelle die Standardvariablen für die Datenbank-Rolle
nano roles/database/defaults/main.yml
```

```yaml
---
# Defaults für Datenbankserver-Rolle
mongodb_port: 27017               # Der Standardport für MongoDB
mongodb_bind_ip: "0.0.0.0"        # Binde an alle IP-Adressen (für die Übung) 
mongodb_dbpath: /var/lib/mongodb  # Das Verzeichnis für die MongoDB-Daten
```

**Warum binden wir an 0.0.0.0?** Für diese Übung verwenden wir `0.0.0.0`, was bedeutet, dass MongoDB auf alle Netzwerkschnittstellen hört. In einer Produktionsumgebung würdest du dies auf bestimmte IPs einschränken (z.B. nur das interne Netzwerk) und zusätzlich Authentifizierung konfigurieren.

Erstelle die Hauptaufgabendatei für den Datenbankserver:

```bash
nano roles/database/tasks/main.yml
```

```yaml
---
# Tasks für die Datenbankserver-Rolle

# System aktualisieren
- name: Update apt cache
  apt:
    update_cache: yes
    cache_valid_time: 3600
  become: yes

# MongoDB Repository-Schlüssel importieren
- name: Import MongoDB repository key
  apt_key:
    url: https://www.mongodb.org/static/pgp/server-7.0.asc
    state: present
  become: yes

```yaml
# MongoDB Repository hinzufügen
- name: Add MongoDB repository
  apt_repository:
    repo: deb [ arch=amd64,arm64 ] https://repo.mongodb.org/apt/ubuntu jammy/mongodb-org/7.0 multiverse
    state: present
    filename: mongodb-org-7.0
  become: yes

# MongoDB installieren
- name: Install MongoDB
  apt:
    name: mongodb-org
    state: present
  become: yes

# Erstelle MongoDB-Konfigurationsdatei
- name: Create MongoDB configuration
  template:
    src: mongod.conf.j2
    dest: /etc/mongod.conf
    owner: root
    group: root
    mode: '0644'
  become: yes
  notify: Restart MongoDB

# MongoDB-Verzeichnis erstellen
- name: Ensure MongoDB data directory exists
  file:
    path: "{{ mongodb_dbpath }}"
    state: directory
    owner: mongodb
    group: mongodb
    mode: '0755'
  become: yes

# MongoDB-Dienst starten und aktivieren
- name: Ensure MongoDB is running and enabled
  service:
    name: mongod
    state: started
    enabled: yes
  become: yes
```

**Was tun diese Tasks?**
1. Zuerst aktualisieren wir den Paket-Cache
2. Dann importieren wir den MongoDB-Repository-Schlüssel und fügen das Repository hinzu
3. Wir installieren MongoDB aus dem offiziellen Repository
4. Wir erstellen eine benutzerdefinierte Konfiguration mit unseren Variablen
5. Wir stellen sicher, dass das Datenverzeichnis existiert und die richtigen Berechtigungen hat
6. Schließlich starten wir den MongoDB-Dienst und aktivieren ihn für den automatischen Start beim Booten

**Warum verwenden wir das offizielle MongoDB-Repository?** Die MongoDB-Version in den Standard-Ubuntu-Repositories kann veraltet sein. Durch die Verwendung des offiziellen MongoDB-Repositories erhalten wir Zugriff auf die neueste stabile Version mit allen Funktionen und Sicherheitspatches.

Erstelle die Handler für den Datenbankserver:

```bash
nano roles/database/handlers/main.yml
```

```yaml
---
# Handler für Datenbankserver-Rolle
- name: Restart MongoDB
  service:
    name: mongod
    state: restarted
  become: yes
```

Erstelle das Template für die MongoDB-Konfiguration:

```bash
nano roles/database/templates/mongod.conf.j2
```

```yaml
# MongoDB Konfiguration von Ansible verwaltet
# Generiert für {{ ansible_hostname }}

storage:
  dbPath: {{ mongodb_dbpath }}
  journal:
    enabled: true

systemLog:
  destination: file
  logAppend: true
  path: /var/log/mongodb/mongod.log

net:
  port: {{ mongodb_port }}
  bindIp: {{ mongodb_bind_ip }}

processManagement:
  timeZoneInfo: /usr/share/zoneinfo
```

### 3.4 Hauptplaybook erstellen

Als letzten Schritt in der Ansible-Konfiguration erstellen wir das Hauptplaybook, das unsere Rollen zusammenfasst und die Ausführungsreihenfolge definiert:

```bash
nano site.yml
```

Füge folgenden Inhalt hinzu:

```yaml
---
# Hauptplaybook zur Konfiguration von Webserver und Datenbankserver

- name: Configure Database Server
  hosts: databases
  become: yes  # Führe Befehle mit Root-Rechten aus (sudo)
  roles:
    - database  # Wende die database-Rolle auf alle Hosts in der "databases"-Gruppe an

- name: Configure Web Server
  hosts: webservers
  become: yes  # Führe Befehle mit Root-Rechten aus (sudo)
  roles:
    - webserver  # Wende die webserver-Rolle auf alle Hosts in der "webservers"-Gruppe an
```

**Erklärung:**
- Das Playbook besteht aus zwei Plays: eines für Datenbankserver und eines für Webserver
- Jedes Play wendet die entsprechende Rolle auf die zugehörige Host-Gruppe an
- Die Reihenfolge ist wichtig: Wir konfigurieren zuerst die Datenbank, damit sie bereit ist, wenn der Webserver darauf zugreifen möchte
- `become: yes` bedeutet, dass Ansible Befehle mit erweiterten Rechten (sudo) ausführt, was für Installationen und Systemkonfigurationen nötig ist

Dieses Playbook ist absichtlich einfach gehalten, um die Kernkonzepte zu verdeutlichen. In realen Projekten könntest du zusätzliche Plays für andere Komponenten oder pre_tasks und post_tasks für Vor- und Nachbereitungsschritte hinzufügen.

## 4. Integration von Terraform und Ansible

Nachdem wir nun sowohl die Terraform-Konfiguration für die Infrastruktur als auch die Ansible-Konfiguration für die Serverkonfiguration erstellt haben, müssen wir einen Weg finden, diese beiden Prozesse zu verbinden und zu automatisieren. Dafür erstellen wir ein Skript, das alles zusammenführt:

```bash
# Wechsle ins Hauptverzeichnis
cd ~/terraform-ansible-aws

nano deploy.sh
```

```bash
#!/bin/bash
# Dieses Skript führt die Terraform-Provisionierung und Ansible-Konfiguration aus

set -e  # Beende das Skript bei Fehlern

echo "===== Starte Deployment mit Terraform und Ansible ====="

# Wechsle in das Terraform-Verzeichnis
cd terraform

echo "===== Initialisiere Terraform ====="
terraform init

echo "===== Führe Terraform Plan aus ====="
terraform plan -out=tfplan

echo "===== Führe Terraform Apply aus ====="
terraform apply -auto-approve tfplan

echo "===== Generiere Ansible-Inventory ====="
./create_inventory.sh

echo "===== Warte 30 Sekunden, bis die Instanzen bereit sind ====="
sleep 30

# Wechsle in das Ansible-Verzeichnis
cd ../ansible

echo "===== Führe Ansible-Playbook aus ====="
ansible-playbook site.yml

echo "===== Deployment abgeschlossen ====="
echo "Webserver ist erreichbar unter: http://$(cd ../terraform && terraform output -raw webserver_public_ip)"
```

Mache das Skript ausführbar:

```bash
chmod +x deploy.sh
```

**Warum dieses Skript so wichtig ist:** Dieses Skript ist das Herzstück unserer Integration zwischen Terraform und Ansible. Es automatisiert den gesamten Workflow vom Erstellen der Infrastruktur bis zur Konfiguration der Server. Ohne dieses Skript müsstest du jeden Schritt manuell ausführen, was fehleranfällig und zeitaufwändig wäre.

**Was macht das Skript im Detail?**

1. `set -e`: Dieser Befehl sorgt dafür, dass das Skript sofort beendet wird, wenn ein Befehl fehlschlägt, was hilft, Fehler frühzeitig zu erkennen.

2. Die Terraform-Befehle führen nacheinander aus:
   - `terraform init`: Lädt Provider und Module herunter
   - `terraform plan -out=tfplan`: Erstellt einen Ausführungsplan und speichert ihn
   - `terraform apply -auto-approve tfplan`: Wendet den Plan an und erstellt die Ressourcen

3. `./create_inventory.sh`: Ruft das vorher erstellte Skript auf, um das Ansible-Inventory aus Terraform-Outputs zu generieren.

4. `sleep 30`: Eine einfache, aber wichtige Pause, die den EC2-Instanzen Zeit gibt, vollständig zu starten und ihre SSH-Dienste bereitzustellen. Ohne diese Pause würde Ansible möglicherweise versuchen, sich zu verbinden, bevor die Server bereit sind.

5. `ansible-playbook site.yml`: Führt das Ansible-Playbook aus, um die Server zu konfigurieren.

6. Die letzte Zeile gibt die URL des Webservers aus, sodass du leicht auf die erstellte Webseite zugreifen kannst.

## 5. Führe das Deployment aus

Jetzt können wir das komplette Deployment ausführen. Dieser Schritt kombiniert das Terraform-Provisioning und die Ansible-Konfiguration in einem einzigen Prozess:

```bash
# Im Hauptverzeichnis
cd ~/terraform-ansible-aws
./deploy.sh
```

**Was passiert bei der Ausführung?**

1. **Terraform-Initialisierung**:
   ```
   terraform init
   ```
   Dies lädt die notwendigen Provider und Module herunter und bereitet das Arbeitsverzeichnis vor.

2. **Terraform-Plan**:
   ```
   terraform plan -out=tfplan
   ```
   Dies erstellt einen Ausführungsplan, der zeigt, welche Ressourcen erstellt oder geändert werden. Die Option `-out=tfplan` speichert diesen Plan für die spätere Anwendung.

3. **Terraform-Apply**:
   ```
   terraform apply -auto-approve tfplan
   ```
   Dies wendet den Plan an und erstellt die AWS-Ressourcen. Die Option `-auto-approve` überspringt die Bestätigungsaufforderung.

4. **Warten auf Instanzen**:
   ```
   sleep 30
   ```
   Wir warten 30 Sekunden, damit die Instanzen starten und bereit für die Konfiguration sind.

5. **Ansible-Playbook-Ausführung**:
   ```
   ansible-playbook site.yml
   ```
   Dies führt das Ansible-Playbook aus, das die Instanzen konfiguriert.

**Hinweis:** Die Ausführung kann einige Minuten dauern, da das Erstellen von AWS-Ressourcen und die Erstinstallation von Paketen Zeit benötigt. Sei geduldig und beobachte die Ausgabe auf Fehler.

**Fehlerbehebung:**
- Falls Terraform-Fehler auftreten, überprüfe deine AWS-Anmeldeinformationen und Region
- Falls Ansible-Fehler auftreten, überprüfe die SSH-Verbindung und warte etwas länger, bis die Instanzen bereit sind
- Falls die EC2-Instanzen nicht erreichbar sind, überprüfe die Sicherheitsgruppen und Netzwerkeinstellungen

## 6. Überprüfe die Ergebnisse

Nach erfolgreicher Ausführung solltest du die Webseite über die IP-Adresse des Webservers erreichen können. Das Skript gibt dir bereits die URL aus, aber du kannst sie auch manuell erhalten:

```bash
# Hole die IP-Adresse des Webservers
cd ~/terraform-ansible-aws/terraform
echo "http://$(terraform output -raw webserver_public_ip)"
```

Öffne diese URL in deinem Browser. Du solltest eine einfache Webseite sehen, die bestätigt, dass die Installation erfolgreich war und Informationen über den Server anzeigt.

**Was solltest du auf der Webseite sehen?**
- Den Titel "Terraform & Ansible AWS Demo"
- Informationen über den Webserver (Hostname, IP-Adresse, Betriebssystem)
- Die IP-Adresse des Datenbankservers
- Den Status der Datenbankverbindung

**Warum ist diese Überprüfung wichtig?** Sie bestätigt, dass:
1. Die EC2-Instanz erfolgreich erstellt wurde
2. Die Sicherheitsgruppen korrekt konfiguriert sind (Port 80 ist erreichbar)
3. Nginx korrekt installiert und konfiguriert wurde
4. Die Anwendung korrekt bereitgestellt wurde
5. Die Verbindung zwischen Webserver und Datenbankserver funktioniert

Du kannst dich auch per SSH mit den Instanzen verbinden, um weitere Tests durchzuführen oder Probleme zu beheben:

```bash
# Verbinde mit dem Webserver
ssh -i ~/.ssh/id_rsa ubuntu@$(terraform output -raw webserver_public_ip)

# Verbinde mit dem Datenbankserver
ssh -i ~/.ssh/id_rsa ubuntu@$(terraform output -raw database_public_ip)
```

## 7. Erkläre die Workflowschritte

Lass uns den Workflow, den wir gerade durchgeführt haben, im Detail erklären:

1. **Terraform-Provisioning**:
   - `terraform init`: Initialisiert das Arbeitsverzeichnis und lädt den AWS-Provider herunter
   - `terraform plan`: Erstellt einen Ausführungsplan, der zeigt, welche Ressourcen erstellt werden
   - `terraform apply`: Wendet den Plan an und erstellt die tatsächlichen Ressourcen in AWS

2. **Ansible-Inventory-Generierung**:
   - Terraform erzeugt Outputs mit den IP-Adressen und weiteren Details
   - Unser Skript extrahiert diese Informationen und erstellt ein Ansible-Inventory

3. **Ansible-Konfiguration**:
   - `ansible-playbook`: Führt das Playbook aus, das die Rollen anwendet
   - Das Playbook konfiguriert zuerst den Datenbankserver, dann den Webserver
   - Beide Server werden entsprechend ihren Rollen konfiguriert

Dieser Workflow zeigt die komplementäre Natur von Terraform und Ansible:
- **Terraform** kümmert sich um die Infrastruktur: Server, Netzwerke, Sicherheitsgruppen
- **Ansible** kümmert sich um die Serverkonfiguration: Software, Dienste, Dateien

## 8. Infrastruktur bereinigen

Wenn du mit der Übung fertig bist, solltest du die AWS-Ressourcen bereinigen, um unnötige Kosten zu vermeiden:

```bash
# Erstelle ein Bereinigungsskript
cd ~/terraform-ansible-aws
nano cleanup.sh
```

```bash
#!/bin/bash
# Dieses Skript entfernt alle AWS-Ressourcen

echo "===== Bereinige AWS-Ressourcen mit Terraform ====="
cd terraform
terraform destroy -auto-approve

echo "===== Bereinigung abgeschlossen ====="
```

Mache das Skript ausführbar und führe es aus:

```bash
chmod +x cleanup.sh
./cleanup.sh
```

**Wichtig:** Dieser Befehl löscht alle in dieser Übung erstellten AWS-Ressourcen. Stelle sicher, dass du keine wichtigen Daten darauf hast.

## 9. Erweiterungsmöglichkeiten

Hier sind einige Ideen, wie du diese Übung erweitern könntest:

1. **Auto Scaling Group**: Erweitere die Terraform-Konfiguration, um eine Auto Scaling Group für den Webserver zu erstellen.

2. **Load Balancer**: Füge einen Application Load Balancer hinzu, um den Verkehr auf mehrere Webserver zu verteilen.

3. **RDS-Datenbank**: Verwende Amazon RDS anstelle einer selbst verwalteten MongoDB-Instanz.

4. **Terraform-Module**: Refaktoriere den Terraform-Code in wiederverwendbare Module.

5. **Ansible-Variablen aus Terraform**: Übergebe mehr Variablen von Terraform an Ansible für eine dynamischere Konfiguration.

## 10. Zusammenfassung

In dieser Übung hast du:

1. **Terraform zur Erstellung von AWS-Ressourcen eingesetzt**
   - EC2-Instanzen, VPC, Sicherheitsgruppen und andere Netzwerkkomponenten
   - Deklarative Konfiguration der gewünschten Infrastruktur

2. **Ein dynamisches Ansible-Inventory aus Terraform-Outputs generiert**
   - Automatische Weitergabe von Informationen zwischen den Tools
   - Keine manuellen Konfigurationsschritte erforderlich

3. **Ansible-Rollen erstellt**
   - Webserver-Rolle mit Nginx und einfacher Webanwendung
   - Datenbankserver-Rolle mit MongoDB
   - Klare Trennung der Verantwortlichkeiten

4. **Einen automatisierten Workflow implementiert**
   - Ein einziges Skript für den gesamten Prozess
   - Reproduzierbare Umgebungen mit minimalem manuellen Aufwand

5. **Die Ergebnisse überprüft und die Infrastruktur bereinigt**
   - Funktionstest der erstellten Ressourcen
   - Saubere Bereinigung, um unnötige Kosten zu vermeiden

### Die Vorteile dieses Ansatzes

Die Kombination von Terraform und Ansible bietet einen leistungsstarken Ansatz für Infrastructure as Code:

- **Vollständige Automatisierung**: Von der Provisionierung bis zur Anwendungskonfiguration
- **Wiederholbarkeit**: Identische Umgebungen können jederzeit neu erstellt werden
- **Versionskontrolle**: Die gesamte Infrastruktur kann im Git verwaltet werden
- **Dokumentation als Code**: Die Konfiguration selbst dokumentiert die Infrastruktur
- **Modularität**: Klare Trennung zwischen Infrastruktur und Konfiguration
- **Skalierbarkeit**: Der Ansatz kann auf Hunderte oder Tausende von Servern skaliert werden

> **Tipp für produktive Umgebungen**: Für echte Produktionsumgebungen solltest du zusätzliche Sicherheitsmaßnahmen implementieren, wie z.B. strengere Berechtigungen, Netzwerksegmentierung und verschlüsselte Kommunikation. Außerdem solltest du die Terraform-Zustandsdatei in einem Remote-Backend speichern, um die Zusammenarbeit im Team zu erleichtern.

## Wichtige Begriffe erklärt:

### AWS-Konzepte
- **AWS (Amazon Web Services)**: Eine Cloud-Computing-Plattform, die verschiedene Services für Rechenleistung, Speicher und Netzwerke anbietet
- **EC2 (Elastic Compute Cloud)**: AWS-Dienst für virtuelle Server in der Cloud mit skalierbarer Rechenkapazität
- **VPC (Virtual Private Cloud)**: Isoliertes virtuelles Netzwerk in der AWS-Cloud mit definierbaren IP-Adressbereich
- **Security Groups**: Virtuelle Firewalls, die den eingehenden und ausgehenden Datenverkehr zu AWS-Ressourcen steuern
- **Subnet**: Ein Segment eines VPC mit eigenem IP-Adressbereich, kann öffentlich oder privat sein
- **Internet Gateway**: Ermöglicht die Kommunikation zwischen Ressourcen in einem VPC und dem Internet
- **Route Table**: Regelsatz, der den Netzwerkverkehr innerhalb eines VPC lenkt
- **AMI (Amazon Machine Image)**: Vorkonfigurierte Vorlage für EC2-Instanzen mit Betriebssystem und Software

### Terraform-Konzepte
- **Terraform**: IaC-Tool (Infrastructure as Code) zur Provisionierung und Verwaltung von Cloud-Infrastruktur
- **Provider**: Plugin für Terraform, das die API eines Service-Anbieters (z.B. AWS, Azure) bereitstellt
- **Resource**: Infrastrukturkomponente, die von Terraform verwaltet wird (z.B. EC2-Instanz, VPC)
- **Module**: Wiederverwendbare Terraform-Konfigurationen für häufig verwendete Ressourcengruppen
- **State**: Terraform-Zustandsdatei, die die aktuelle Konfiguration der verwalteten Infrastruktur speichert
- **Plan**: Ausführungsplan, der anzeigt, welche Änderungen Terraform vornehmen wird
- **Apply**: Prozess, der den Terraform-Plan umsetzt und die Infrastruktur erstellt oder ändert
- **Output**: Informationen, die nach einer Terraform-Ausführung zurückgegeben werden
- **Variable**: Parametrisierbare Werte in Terraform-Konfigurationen
- **Remote Backend**: Speicherort für Terraform-State-Dateien, der Teamarbeit ermöglicht (z.B. S3)
- **HCL (HashiCorp Configuration Language)**: Deklarative Sprache, in der Terraform-Konfigurationen geschrieben werden

### Ansible-Konzepte
- **Ansible**: Agentenlose Automatisierungsplattform für Konfigurationsmanagement und Anwendungsbereitstellung
- **Inventory**: Datei oder Skript, das die zu verwaltenden Hosts und Gruppen definiert
- **Playbook**: YAML-Datei, die Ansible-Aufgaben in geordneter Reihenfolge definiert
- **Role**: Wiederverwendbare Struktur zur Organisation von Playbooks, Tasks und Variablen
- **Task**: Einzelne Aktion in Ansible (z.B. Paket installieren, Dienst starten)
- **Handler**: Spezielle Tasks, die nur bei Änderungen durch andere Tasks ausgeführt werden
- **Module**: Funktionseinheiten in Ansible, die bestimmte Aufgaben ausführen (z.B. apt, service)
- **Template (Jinja2)**: Dynamische Dateien, in denen Variablen zur Laufzeit ersetzt werden
- **Facts**: Automatisch gesammelte Systeminformationen über verwaltete Hosts
- **Become**: Mechanismus zum Ausführen von Befehlen mit erhöhten Rechten (sudo)
- **Vault**: Werkzeug zur Verschlüsselung sensibler Daten in Ansible
- **Galaxy**: Repository für vorgefertigte Ansible-Rollen und -Collections
- **Idempotenz**: Eigenschaft von Ansible-Tasks, bei wiederholter Ausführung den gleichen Endzustand zu erreichen
- **Ad-hoc Commands**: Einzelne Ansible-Befehle für schnelle Aufgaben ohne Playbook

### Integration von Terraform und Ansible
- **Dynamic Inventory**: Automatisch generiertes Ansible-Inventory aus Cloud-Provider-Daten
- **Provisioner**: Mechanismus in Terraform zum Aufruf externer Tools wie Ansible nach Ressourcenerstellung
- **Infrastructure as Code (IaC)**: Ansatz zur Verwaltung von Infrastruktur durch maschinenlesbare Definitionsdateien
- **Configuration as Code**: Verwaltung von Konfigurationen durch versionierte Code-Dateien
- **GitOps**: Workflow, bei dem Git als Single Source of Truth für Infrastruktur und Konfiguration dient
