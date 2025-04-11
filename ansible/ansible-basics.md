# Ansible Grundlagen

Dieses Tutorial führt dich in die Grundlagen von Ansible ein. Du lernst, wie du deine Systemkonfiguration und Anwendungsbereitstellung automatisieren kannst - eine essentielle Fähigkeit für moderne IT-Operations und DevOps-Praktiken.

Schaut auch gerne in die Doku: https://docs.ansible.com/

## Inhaltsverzeichnis
- [1. Einführung in Ansible](#1-einführung-in-ansible)
  - [1.1 Was ist Configuration Management?](#11-was-ist-configuration-management)
  - [1.2 Was ist Ansible und warum solltest du es nutzen?](#12-was-ist-ansible-und-warum-solltest-du-es-nutzen)
  - [1.3 Ansible vs. andere Configuration Management Tools](#13-ansible-vs-andere-configuration-management-tools)
  - [1.4 Kernkonzepte: Playbooks, Rollen, Inventories, Module](#14-kernkonzepte-playbooks-rollen-inventories-module)
- [2. Ansible-Architektur](#2-ansible-architektur)
  - [2.1 YAML-Syntax](#21-yaml-syntax)
  - [2.2 Ansible-Workflow: Playbook-Ausführung](#22-ansible-workflow-playbook-ausführung)
  - [2.3 Inventory-Management](#23-inventory-management)
  - [2.4 Ansible-Rollen](#24-ansible-rollen)
- [3. Fortgeschrittene Ansible-Konzepte](#3-fortgeschrittene-ansible-konzepte)
  - [3.1 Variablen und Facts](#31-variablen-und-facts)
  - [3.2 Templates mit Jinja2](#32-templates-mit-jinja2)
  - [3.3 Berechtigungsmanagement](#33-berechtigungsmanagement)
  - [3.4 Best Practices](#34-best-practices)

## 1. Einführung in Ansible

### 1.1 Was ist Configuration Management?

Stell dir vor, du müsstest jeden Server manuell einrichten, jede Software installieren und konfigurieren und das bei jeder Änderung oder für jeden neuen Server wiederholen. Klingt nach einem Albtraum, oder? Genau hier kommt Configuration Management ins Spiel.

**Configuration Management** bedeutet, dass du den Zustand deiner IT-Systeme automatisiert definierst, anwendest und überwachst. Statt manuelle Befehle auszuführen oder Konfigurationsdateien von Hand zu ändern, definierst du den gewünschten Zustand deiner Systeme in maschinenlesbaren Dateien.

Der große Vorteil: Was einmal definiert ist, kannst du:
- **Automatisiert anwenden** auf beliebig viele Systeme
- **Dokumentieren** durch Code (selbstdokumentierende Infrastruktur)
- **Versionieren** (mit Git oder ähnlichen Tools)
- **Konsistent halten** über alle Umgebungen hinweg
- **Testen** vor dem Ausrollen in Produktion
- **Kommunikation mit Expertensystemen** z.B. Claude, Gemini oder ChatGPT

Configuration Management verhindert das berüchtigte "Aber bei mir funktioniert es doch!"-Problem, indem es sicherstellt, dass alle Systeme gleich konfiguriert sind und Änderungen nachvollziehbar dokumentiert werden.

### 1.2 Was ist Ansible und warum solltest du es nutzen?

**Ansible** ist ein Open-Source-Automatisierungstool, das Configuration Management, Anwendungsbereitstellung und Task-Automatisierung vereinfacht. Es wurde mit dem Ziel entwickelt, möglichst einfach und zugänglich zu sein, ohne dabei an Leistung einzubüßen.

**Warum Ansible?**

1. **Agentenlos**: Anders als viele andere Tools benötigt Ansible keine vorinstallierten Agenten auf den zu verwaltenden Systemen. Es nutzt SSH für Linux/Unix und WinRM für Windows.

2. **Einfache Syntax**: Ansible verwendet YAML, eine menschenlesbare Auszeichnungssprache, für seine Playbooks. So ist der Code auch für Nicht-Programmierer verständlich.

3. **Idempotenz**: Ansible-Operationen können mehrfach ausgeführt werden, ohne unerwünschte Nebenwirkungen zu verursachen. Das System prüft, ob der gewünschte Zustand bereits erreicht ist.

4. **Breite Unterstützung**: Ansible funktioniert mit praktisch allen Betriebssystemen und Plattformen und hat Tausende vorgefertigter Module für verschiedenste Aufgaben.

5. **Deklarativ**: Du definierst den Zielzustand, nicht die Schritte dorthin. Ansible ermittelt selbst, was zu tun ist.

6. **Geringer Overhead**: Da keine Agenten oder Datenbanken nötig sind, ist Ansible sehr leichtgewichtig und schnell einzurichten.

Hier ein einfaches Beispiel, wie Ansible-Code aussieht:

```yaml
---
- name: Webserver installieren und konfigurieren
  hosts: webservers
  become: yes
  
  tasks:
    - name: Neueste Version von Apache installieren
      apt:
        name: apache2
        state: latest
    
    - name: Apache-Dienst starten und aktivieren
      service:
        name: apache2
        state: started
        enabled: yes
```

Dieser Code sorgt dafür, dass auf allen Servern in der Gruppe "webservers" Apache installiert, gestartet und so konfiguriert ist, dass er beim Systemstart automatisch startet. Die Stärke liegt darin, dass Ansible dieses Playbook gegen hunderte Server gleichzeitig ausführen kann und es wird immer den gleichen Zustand herstellen.

### 1.3 Ansible vs. andere Configuration Management Tools

Auf dem Markt gibt es mehrere Configuration Management Tools. Wie unterscheidet sich Ansible von diesen?

**Puppet**:
- Verwendet eine eigene deklarative Sprache (Puppet DSL)
- Benötigt einen Agent auf jedem verwalteten System
- Master-Agent-Architektur
- Etablierte Enterprise-Lösung mit starker Unterstützung
- Steilere Lernkurve als Ansible

**Chef**:
- Verwendet Ruby für seine "Rezepte"
- Benötigt einen Agent auf jedem System
- Erfordert mehr Programmierkenntnisse
- Ist stärker imperativer Natur (wie man etwas tun soll)
- Gut für komplexe Umgebungen mit Ruby-Kenntnissen

**SaltStack**:
- Python-basiert mit YAML-Konfigurationen
- Kann sowohl agentenbasiert als auch agentenlos arbeiten
- Höhere Geschwindigkeit durch optimierte Kommunikation
- Etwas komplexer in der Einrichtung als Ansible

**Terraform** (eigentlich IaC, nicht CM):
- Primär für Infrastruktur-Provisionierung statt Konfiguration
- Gut für Multi-Cloud-Umgebungen
- Verwaltet State extern
- Ergänzt Ansible oft, anstatt es zu ersetzen

**Ansible-Vorteile**:
- Keine Agenten nötig (einfacher Einstieg)
- Flache Lernkurve
- YAML-basierte, leicht verständliche Syntax
- Große Community und umfangreiches Ökosystem
- Breite Integrationen mit anderen Tools

In der Praxis werden oft mehrere Tools kombiniert: Terraform für die Infrastruktur und Ansible für die Konfiguration der erstellten Server.

### 1.4 Kernkonzepte: Playbooks, Rollen, Inventories, Module

Um Ansible effektiv zu nutzen, solltest du diese vier Kernkonzepte verstehen:

**1. Inventories**

Inventories definieren, welche Hosts Ansible verwalten soll. Sie können statisch (als Datei) oder dynamisch (aus Cloud-Providern, CMDB, etc. generiert) sein.

Beispiel für ein einfaches Inventory:

```ini
# inventory.ini
[webservers]
web1.example.com
web2.example.com

[dbservers]
db1.example.com

[production:children]
webservers
dbservers

[production:vars]
environment=production
```

Dieses Inventory definiert zwei Gruppen von Servern (webservers und dbservers), fasst sie in einer übergeordneten Gruppe zusammen und weist dieser eine Variable zu.

**2. Playbooks**

Playbooks sind das Herzstück von Ansible – YAML-Dateien, die definieren, was auf den Zielsystemen geschehen soll. Sie bestehen aus einem oder mehreren Plays, die wiederum aus Tasks bestehen.

```yaml
---
- name: Webserver einrichten
  hosts: webservers
  become: yes  # Als root ausführen
  
  tasks:
    - name: Apache installieren
      apt:
        name: apache2
        state: present
    
    - name: Konfigurationsdatei kopieren
      copy:
        src: files/apache2.conf
        dest: /etc/apache2/apache2.conf
      notify: Apache neustarten
  
  handlers:
    - name: Apache neustarten
      service:
        name: apache2
        state: restarted
```

Ein Playbook kann mehrere Plays enthalten, um verschiedene Aspekte deiner Infrastruktur zu konfigurieren.

**3. Module**

Module sind die Arbeitseinheiten in Ansible – sie führen die eigentlichen Aktionen aus. Ansible kommt mit Hunderten integrierter Module für alles von Paketmanagement bis hin zu Cloud-Ressourcen.

Einige häufig verwendete Module:
- `apt`/`yum`/`dnf` für Paketmanagement
- `copy`/`template` zum Kopieren/Rendern von Dateien
- `service` zur Verwaltung von Diensten
- `user`/`group` für Benutzer und Gruppen
- `file` für Dateien und Verzeichnisse
- `git` für Git-Repositories
- `docker_container` für Docker-Container

Module akzeptieren Parameter und führen idempotente Operationen aus – sie ändern nur, was geändert werden muss.

**4. Rollen**

Rollen sind ein Weg, Playbooks in wiederverwendbare, modulare Einheiten zu organisieren. Sie bündeln Dateien, Variablen, Tasks, Handler und andere Ressourcen für eine bestimmte Funktion.

Eine typische Rollenstruktur sieht so aus:

```
webserver/
├── defaults/     # Standard-Variablenwerte
│   └── main.yml
├── files/        # Statische Dateien
├── handlers/     # Handler-Definitionen
│   └── main.yml
├── meta/         # Metadaten der Rolle
│   └── main.yml
├── tasks/        # Hauptaufgaben
│   └── main.yml
├── templates/    # Jinja2-Templates
└── vars/         # Feste Variablen
    └── main.yml
```

Verwendung einer Rolle im Playbook:

```yaml
---
- hosts: webservers
  roles:
    - webserver
```

Diese Kernkonzepte bilden das Fundament für alles, was du mit Ansible tun wirst.

## 2. Ansible-Architektur

### 2.1 YAML-Syntax

Ansible-Konfigurationen werden in YAML (YAML Ain't Markup Language) geschrieben – einer menschenlesbaren Datenstruktursprache, die einfach zu erlernen ist.

**Grundelemente von YAML:**

1. **Listen**:
   ```yaml
   fruits:
     - apple
     - banana
     - cherry
   ```

2. **Dictionaries (Schlüssel-Wert-Paare)**:
   ```yaml
   user:
     name: john
     age: 30
     roles:
       - admin
       - developer
   ```

3. **Verschachtelung durch Einrückung**:
   ```yaml
   company:
     departments:
       it:
         employees: 50
         manager: Jane
       hr:
         employees: 10
         manager: Bob
   ```

4. **Multiline-Strings**:
   ```yaml
   # Block-Stil (behält Zeilenumbrüche)
   description: |
     Dies ist eine
     mehrzeilige
     Beschreibung.
   
   # Gefalteter Stil (wandelt Zeilenumbrüche in Leerzeichen um)
   message: >
     Dies ist eine lange Nachricht,
     die zur besseren Lesbarkeit
     über mehrere Zeilen verteilt ist.
   ```

5. **Anker und Referenzen** (für wiederverwendbare Elemente):
   ```yaml
   defaults: &defaults
     timeout: 60
     retries: 3
   
   production:
     <<: *defaults  # Übernimmt alle Werte aus defaults
     environment: production
   ```

Wichtige Regeln für YAML in Ansible:

- Einrückung ist bedeutsam und sollte konsistent sein (meist 2 Leerzeichen)
- Leerzeichen, nicht Tabulatoren verwenden
- Listen-Elemente beginnen mit einem Bindestrich und Leerzeichen (`- `)
- Kommentare beginnen mit `#`
- Playbooks beginnen typischerweise mit `---` (YAML-Dokumentstart)

Vorteile von YAML für Ansible:
- Weniger Klammern und Quotes als JSON oder XML
- Menschenlesbar auch für Nicht-Programmierer
- Gute Unterstützung für komplexe Datenstrukturen
- Klare visuelle Hierarchie

### 2.2 Ansible-Workflow: Playbook-Ausführung

Der Ansible-Workflow ist im Vergleich zu anderen Configuration-Management-Tools relativ einfach. Lass uns die Schritte durchgehen, wie ein Playbook ausgeführt wird:

**1. Playbook-Start**

```bash
$ ansible-playbook -i inventory.ini webserver.yml

PLAY [Webserver einrichten] *****************************************
```

Dieser Befehl startet die Ausführung des Playbooks:
- `-i inventory.ini` gibt die Inventory-Datei an
- `webserver.yml` ist das auszuführende Playbook

**2. Gathering Facts**

```
TASK [Gathering Facts] **********************************************
ok: [web1.example.com]
ok: [web2.example.com]
```

Standardmäßig sammelt Ansible zunächst Informationen über jeden Host (Betriebssystem, Hardware, Netzwerk etc.). Diese "Facts" können später im Playbook verwendet werden.

**3. Task-Ausführung**

```
TASK [Apache installieren] ******************************************
changed: [web1.example.com]
ok: [web2.example.com]

TASK [Konfigurationsdatei kopieren] *********************************
changed: [web1.example.com]
changed: [web2.example.com]
```

Ansible führt jeden Task nacheinander auf allen angegebenen Hosts aus. Die Ausgabe zeigt den Status:
- `changed`: Der Task hat eine Änderung vorgenommen
- `ok`: Der System-Zustand entsprach bereits dem gewünschten Zustand
- `failed`: Der Task ist fehlgeschlagen
- `skipped`: Der Task wurde übersprungen (z.B. durch Bedingungen)

**4. Handler-Ausführung**

```
RUNNING HANDLER [Apache neustarten] *********************************
changed: [web1.example.com]
changed: [web2.example.com]
```

Handler werden nur ausgeführt, wenn sie durch einen Task mit `notify` benachrichtigt wurden und der Task den Status `changed` hat.

**5. Play-Zusammenfassung**

```
PLAY RECAP *********************************************************
web1.example.com     : ok=4    changed=3    unreachable=0    failed=0
web2.example.com     : ok=4    changed=2    unreachable=0    failed=0
```

Am Ende gibt Ansible eine Zusammenfassung für jeden Host:
- `ok`: Anzahl erfolgreich ausgeführter Tasks
- `changed`: Anzahl der Tasks, die Änderungen vorgenommen haben
- `unreachable`: Anzahl der Hosts, die nicht erreichbar waren
- `failed`: Anzahl fehlgeschlagener Tasks

**Weitere wichtige Ausführungsoptionen:**

- `--check`: Simulationsmodus (ohne tatsächliche Änderungen)
- `--diff`: Zeigt Unterschiede bei geänderten Dateien
- `--limit webserver1`: Beschränkt die Ausführung auf bestimmte Hosts
- `--tags deploy`: Führt nur Tasks mit bestimmten Tags aus
- `--skip-tags upgrade`: Überspringt Tasks mit bestimmten Tags
- `-v` bis `-vvvv`: Unterschiedliche Detailstufen für die Ausgabe

Dieser Workflow macht Ansible vorhersehbar und transparent – du siehst genau, was auf jedem System passiert.

### 2.3 Inventory-Management

Das Inventory ist das Verzeichnis aller Systeme, die Ansible verwalten soll. Es definiert nicht nur die Hosts, sondern auch ihre Gruppierung und spezifische Variablen.

**Statisches Inventory:**

Das einfachste Inventory ist eine INI- oder YAML-Datei:

```ini
# Einfaches INI-Format
[webservers]
web1.example.com ansible_host=192.168.1.101
web2.example.com

[dbservers]
db1.example.com
db2.example.com

[webservers:vars]
http_port=80
proxy_port=8080

[all:children]
webservers
dbservers

[all:vars]
ansible_user=deployer
```

Das gleiche Inventory in YAML:

```yaml
all:
  children:
    webservers:
      hosts:
        web1.example.com:
          ansible_host: 192.168.1.101
        web2.example.com:
      vars:
        http_port: 80
        proxy_port: 8080
    dbservers:
      hosts:
        db1.example.com:
        db2.example.com:
  vars:
    ansible_user: deployer
```

**Host-Muster:**

Ansible unterstützt auch Muster für ähnlich benannte Hosts:

```ini
[webservers]
web[01:50].example.com      # web01.example.com bis web50.example.com
db-[a:f].example.com        # db-a.example.com bis db-f.example.com
```

**Inventory-Variablen:**

Es gibt mehrere Wege, Host- oder Gruppenvariablen zu definieren:

1. **Direkt im Inventory** (wie oben gezeigt)

2. **In separaten Dateien** (empfohlen für komplexere Setups):
   ```
   inventory/
   ├── hosts                # Hauptinventory
   ├── group_vars/
   │   ├── all.yml          # Variablen für alle Hosts
   │   ├── webservers.yml   # Variablen für die Webserver-Gruppe
   │   └── dbservers.yml    # Variablen für die Datenbankserver-Gruppe
   └── host_vars/
       ├── web1.example.com.yml  # Spezifische Variablen für web1
       └── db1.example.com.yml   # Spezifische Variablen für db1
   ```

**Dynamisches Inventory:**

Für Umgebungen, in denen Hosts dynamisch erstellt und gelöscht werden (wie Cloud-Umgebungen), bietet Ansible dynamische Inventory-Skripte:

```bash
# Beispiel für die Verwendung des AWS EC2-Inventory-Skripts
ansible-playbook -i inventory/aws_ec2.yml playbook.yml
```

Diese Skripte fragen die aktuelle Infrastruktur direkt ab (z.B. von AWS, Azure, GCP, VMware) und generieren das Inventory zur Laufzeit.

**Inventory-Plugins:**

Moderne Ansible-Versionen verwenden Inventory-Plugins statt Skripte:

```yaml
# aws_ec2.yml
plugin: aws_ec2
regions:
  - eu-central-1
  - eu-west-1
keyed_groups:
  - key: tags.Environment
    prefix: env
  - key: instance_type
    prefix: type
```

Diese Konfiguration gruppiert EC2-Instances automatisch nach ihren Tags und Instance-Typen.

**Best Practices für Inventories:**

1. **Einheitliche Namenskonvention** für Hosts und Gruppen
2. **Gruppenhierarchien sinnvoll nutzen** (Umgebung > Funktion > Subtyp)
3. **Variablen in externen Dateien** für bessere Übersichtlichkeit
4. **Hosts nach Funktion gruppieren**, nicht nach physischem Standort
5. **Dynamische Inventories für Cloud-Umgebungen** verwenden
6. **Inventory-Dateien versionieren** (z.B. mit Git)

Ein gut strukturiertes Inventory ist die Grundlage für skalierbare Ansible-Deployments.

### 2.4 Ansible-Rollen

Rollen sind Ansible's Weg, Playbooks in wiederverwendbare, modulare Einheiten zu zerlegen. Sie folgen einer festgelegten Verzeichnisstruktur und machen komplexe Automatisierungen übersichtlicher und wartbarer.

**Anatomie einer Rolle:**

```
roles/
└── webserver/
    ├── defaults/       # Standardvariablen (niedrigste Priorität)
    │   └── main.yml
    ├── files/          # Statische Dateien für copy-Modul
    │   └── htaccess
    ├── handlers/       # Handler-Definitionen
    │   └── main.yml
    ├── meta/           # Abhängigkeiten und Metadaten
    │   └── main.yml
    ├── tasks/          # Hauptaufgaben der Rolle
    │   ├── main.yml
    │   └── install.yml
    ├── templates/      # Jinja2-Templates für template-Modul
    │   └── vhost.conf.j2
    ├── vars/           # Feste Variablen (höhere Priorität)
    │   └── main.yml
    └── README.md       # Dokumentation
```

Jedes Verzeichnis hat einen bestimmten Zweck:

- **defaults/main.yml**: Standardwerte für Variablen, die überschrieben werden können
- **files/**: Statische Dateien, die ohne Änderung auf Zielserver kopiert werden
- **handlers/main.yml**: Handler, die auf Benachrichtigungen von Tasks reagieren
- **meta/main.yml**: Rollenmetadaten wie Autor, Abhängigkeiten, Plattformen
- **tasks/main.yml**: Haupteinstiegspunkt für Tasks
- **templates/**: Jinja2-Templates, die vor dem Kopieren gerendert werden
- **vars/main.yml**: Feste Variablen, die schwerer zu überschreiben sind

**Beispiel für eine einfache Rolle:**

```yaml
# roles/webserver/tasks/main.yml
---
- name: Include OS-specific variables
  include_vars: "{{ ansible_os_family }}.yml"

- name: Install Apache
  package:
    name: "{{ apache_package_name }}"
    state: present

- name: Ensure Apache is running
  service:
    name: "{{ apache_service_name }}"
    state: started
    enabled: yes

- name: Deploy virtual host configuration
  template:
    src: vhost.conf.j2
    dest: "{{ apache_conf_dir }}/vhost.conf"
  notify: Restart Apache
```

```yaml
# roles/webserver/handlers/main.yml
---
- name: Restart Apache
  service:
    name: "{{ apache_service_name }}"
    state: restarted
```

```yaml
# roles/webserver/defaults/main.yml
---
# Standardwerte, die überschrieben werden können
apache_listen_port: 80
apache_document_root: /var/www/html
```

**Verwendung einer Rolle in einem Playbook:**

```yaml
---
- hosts: webservers
  roles:
    - webserver              # Einfache Verwendung

- hosts: appservers
  roles:
    - role: webserver        # Erweiterte Verwendung
      apache_listen_port: 8080
      tags: ['web', 'http']
```

**Abhängigkeiten zwischen Rollen:**

```yaml
# roles/webapp/meta/main.yml
---
dependencies:
  - role: webserver
  - role: database
    when: deploy_database
```

**Rollensammlungen teilen:**

Ansible Galaxy ist ein zentrales Repository für Ansible-Rollen:

```bash
# Rolle aus Galaxy installieren
ansible-galaxy install geerlingguy.apache

# Rolle in requirements.yml definieren
# requirements.yml
---
- src: geerlingguy.apache
  version: 3.1.0

- src: https://github.com/company/ansible-role-app.git
  scm: git
  version: master
  name: webapp

# Alle Rollen aus requirements.yml installieren
ansible-galaxy install -r requirements.yml
```

**Vorteile von Rollen:**

1. **Wiederverwendbarkeit**: Einmal erstellen, überall nutzen
2. **Modularität**: Komplexe Aufgaben in überschaubare Teile zerlegen
3. **Übersichtlichkeit**: Klare Struktur statt riesiger Playbooks
4. **Gemeinsame Nutzung**: Einfacher Austausch mit anderen Teams
5. **Testbarkeit**: Einfacher zu testen als monolithische Playbooks

Rollen sind der Schlüssel zu einer skalierbaren Ansible-Codebase und sollten für alle sich wiederholenden Konfigurationsaufgaben verwendet werden.

## 3. Fortgeschrittene Ansible-Konzepte

### kommt noch! 
