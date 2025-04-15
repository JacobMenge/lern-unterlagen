# Ansible Praxis: NoSQL-Datenbank mit MongoDB einrichten

Diese Übung baut auf deinen neu erworbenen Ansible-Grundlagen auf und führt dich in die Automatisierung von NoSQL-Datenbanken ein. Wir werden einen MongoDB-Server einrichten und dabei wichtige Ansible-Konzepte in der Praxis anwenden.

> **Hinweis:** Für diese Übung solltest du bereits mit den Grundlagen von Ansible vertraut sein und idealerweise die Ansible "Hello World"- und "Nginx"-Übungen abgeschlossen haben. Zusätzliches Hintergrundwissen findest du in den [Ansible-Unterlagen](https://github.com/JacobMenge/lern-unterlagen/blob/main/ansible/ansible-basics.md).

## Was du in dieser Übung lernen wirst

In dieser Übung wirst du:
1. Eine Ansible-Rolle für MongoDB erstellen und konfigurieren
2. Mit externen Paketquellen in Ansible arbeiten
3. Sicherheitseinstellungen für MongoDB verwalten
4. Datenbanken und Benutzer mit Ansible erstellen
5. Mit systemd-Services arbeiten
6. Verschiedene Umgebungskonfigurationen implementieren

## Warum MongoDB und warum Automatisierung?

**MongoDB** ist eine dokumentenorientierte NoSQL-Datenbank, die sich hervorragend für:
- Web-Anwendungen mit variablen Datenstrukturen
- Anwendungen, die hohe Skalierbarkeit erfordern
- Projekte, die mit JSON-ähnlichen Dokumenten arbeiten
- Rapid Prototyping und agile Entwicklung

eignet.

Die **Automatisierung der MongoDB-Installation** mit Ansible bringt folgende Vorteile:
- **Konsistenz**: Gleiche Konfiguration über verschiedene Umgebungen hinweg
- **Reproduzierbarkeit**: Einfache Wiederherstellung oder Skalierung der Datenbankinfrastruktur
- **Dokumentation als Code**: Die Konfiguration ist selbstdokumentierend
- **Zeitersparnis**: Schnelle Bereitstellung neuer Instanzen

## 1. Projektstruktur einrichten

Beginnen wir mit der Erstellung einer passenden Verzeichnisstruktur:

```bash
# erstelle und wechsle in das projektverzeichnis
mkdir -p ~/ansible-mongodb-basic
cd ~/ansible-mongodb-basic

# erstelle die ansible-verzeichnisstruktur
mkdir -p roles/mongodb/{tasks,templates,defaults,handlers,files}
```

**Erklärung der Verzeichnisstruktur:**
- `tasks`: Enthält die auszuführenden Aufgaben
- `templates`: Enthält Jinja2-Vorlagen für dynamische Konfigurationsdateien
- `defaults`: Enthält Standardvariablen für die Rolle
- `handlers`: Enthält Handler, die auf Änderungen reagieren
- `files`: Enthält statische Dateien, die ohne Änderungen kopiert werden

Im Vergleich zur Nginx-Übung haben wir ein `files`-Verzeichnis hinzugefügt. Dies ist ein wichtiges Konzept in Ansible-Rollen, daher solltest du den Unterschied zwischen `files` und `templates` verstehen:
- `files`: Statische Dateien, die ohne Änderungen kopiert werden (z.B. Zertifikate, Skripte)
- `templates`: Dynamische Dateien, in denen Variablen ersetzt werden (z.B. Konfigurationsdateien)

## 2. Inventory-Datei erstellen

Wie in der vorherigen Übung verwenden wir den lokalen Server:

```bash
# erstelle das inventory
nano inventory.ini
```

Füge folgenden Inhalt hinzu:

```ini
[dbservers]
localhost ansible_connection=local

[all:vars]
ansible_python_interpreter=/usr/bin/python3
```

**Erklärung:**
- `[dbservers]` definiert eine Gruppe namens "dbservers"
- `localhost ansible_connection=local` gibt an, dass wir auf dem lokalen System arbeiten
- `ansible_python_interpreter=/usr/bin/python3` sorgt dafür, dass Ansible Python 3 verwendet

## 3. MongoDB-Rolle erstellen

### 3.1 Standardvariablen definieren

```bash
# wechsle in das mongodb-rollen-verzeichnis
cd ~/ansible-mongodb-basic/roles/mongodb

# erstelle standardvariablen
nano defaults/main.yml
```

Füge folgende Variablen hinzu:

```yaml
---
# Standardvariablen für die MongoDB-Konfiguration

# MongoDB-Version
mongodb_version: "7.0"
mongodb_package: mongodb-org

# Systemkonfiguration
mongodb_user: mongodb
mongodb_group: mongodb
mongodb_data_dir: /var/lib/mongodb
mongodb_log_dir: /var/log/mongodb

# Netzwerkkonfiguration
mongodb_port: 27017
mongodb_bind_ip: "127.0.0.1"  # nur lokale Verbindungen erlauben

# Authentifizierung
mongodb_auth_enabled: true
mongodb_admin_user: "admin"
mongodb_admin_password: "changeMe123"  # In Produktion durch Vault ersetzen!

# Datenbanknutzer und -datenbanken
mongodb_users:
  - {
      name: "appuser",
      password: "appPassword123",
      database: "appdb",
      roles: "readWrite"
    }

# Leistungsoptimierung
mongodb_wiredtiger_cache_size: 0.5  # 50% des freien RAM
mongodb_journal_enabled: true
```

**Wichtiger Hinweis zur Sicherheit:**
In einer echten Produktionsumgebung würdest du sensible Daten wie Passwörter niemals im Klartext speichern. Stattdessen würdest du Ansible Vault verwenden, um diese Informationen zu verschlüsseln. Für diese Übung verwenden wir der Einfachheit halber Klartext.

### 3.2 Dateien für externe Paketquellen vorbereiten

Da MongoDB nicht in den Standardrepositorien von Ubuntu/Debian enthalten ist, müssen wir die offiziellen MongoDB-Repositorys hinzufügen. Dafür erstellen wir eine Datei im `files`-Verzeichnis:

```bash
# erstelle verzeichnis für verschiedene ubuntu/debian-versionen
mkdir -p files/apt
```

Erstelle die Repositorydatei für Debian/Ubuntu:

```bash
nano files/apt/mongodb-org-6.0.list
```

Füge folgenden Inhalt hinzu:

```
deb [ arch=amd64,arm64 ] https://repo.mongodb.org/apt/ubuntu jammy/mongodb-org/7.0 multiverse
```

**Hinweis:** Dies ist für Ubuntu 22.04 (Jammy). Für andere Distributionen müsstest du die entsprechende Datei anpassen.

### 3.3 Tasks für die MongoDB-Installation erstellen

```bash
# erstelle die hauptaufgabendatei
nano tasks/main.yml
```

Füge folgende Tasks hinzu:

```yaml
---
# Tasks zur Installation und Konfiguration von MongoDB

# Überprüfung des Betriebssystems
- name: Überprüfe unterstützte Distribution
  assert:
    that:
      - ansible_distribution == "Ubuntu" 
      - ansible_distribution_major_version is version('20.04', '>=')
    msg: "Diese Rolle unterstützt nur Ubuntu 20.04 oder neuer"

# Vorbereitende Aufgaben
- name: Installiere benötigte Pakete für die Repositoryverwaltung
  apt:
    name:
      - gnupg
      - curl
    state: present
    update_cache: yes
  become: yes

- name: MongoDB-Schlüssel importieren
  apt_key:
    url: https://www.mongodb.org/static/pgp/server-{{ mongodb_version }}.asc
    state: present
  become: yes

- name: MongoDB-Repository hinzufügen
  copy:
    src: apt/mongodb-org-{{ mongodb_version }}.list
    dest: /etc/apt/sources.list.d/mongodb-org-{{ mongodb_version }}.list
    owner: root
    group: root
    mode: '0644'
  become: yes
  register: repo_added

- name: Aktualisiere APT-Cache nach Repository-Hinzufügung
  apt:
    update_cache: yes
  become: yes
  when: repo_added.changed

# MongoDB-Installation
- name: Installiere MongoDB-Pakete
  apt:
    name: "{{ mongodb_package }}"
    state: present
  become: yes
  register: mongodb_installed

# Verzeichnisse konfigurieren
- name: Stelle sicher, dass MongoDB-Datenverzeichnis existiert
  file:
    path: "{{ mongodb_data_dir }}"
    state: directory
    owner: "{{ mongodb_user }}"
    group: "{{ mongodb_group }}"
    mode: '0755'
  become: yes

- name: Stelle sicher, dass MongoDB-Logverzeichnis existiert
  file:
    path: "{{ mongodb_log_dir }}"
    state: directory
    owner: "{{ mongodb_user }}"
    group: "{{ mongodb_group }}"
    mode: '0755'
  become: yes

# Konfiguration
- name: Erstelle MongoDB-Konfigurationsdatei
  template:
    src: mongod.conf.j2
    dest: /etc/mongod.conf
    owner: root
    group: root
    mode: '0644'
  become: yes
  notify: Restart MongoDB

# Erste Nutzerkonfiguration (nur bei Neuinstallation)
- name: Stelle sicher, dass MongoDB gestartet ist für die erste Benutzerkonfiguration
  service:
    name: mongod
    state: started
    enabled: yes
  become: yes

- name: Warte, bis MongoDB bereit ist
  wait_for:
    port: "{{ mongodb_port }}"
    delay: 5
    timeout: 30
  when: mongodb_installed.changed

- name: Erstelle MongoDB-Admin-Benutzer
  shell: >
    mongosh --host {{ mongodb_bind_ip }} --port {{ mongodb_port }} admin --eval '
    db.createUser({
      user: "{{ mongodb_admin_user }}",
      pwd: "{{ mongodb_admin_password }}",
      roles: [ { role: "root", db: "admin" } ]
    })'
  # Nur ausführen, wenn Authentifizierung noch nicht aktiviert ist und MongoDB neu installiert wurde
  when: mongodb_installed.changed and mongodb_auth_enabled
  register: mongodb_admin_created
  changed_when: mongodb_admin_created.rc == 0
  failed_when: false

- name: Aktiviere Authentifizierung
  template:
    src: mongod.conf.j2
    dest: /etc/mongod.conf
    owner: root
    group: root
    mode: '0644'
  become: yes
  notify: Restart MongoDB
  when: mongodb_admin_created is defined and mongodb_admin_created.changed

# Ausführung verzögern, damit MongoDB mit Authentifizierung neu starten kann
- name: Warte, bis MongoDB mit Authentifizierung neu gestartet ist
  wait_for:
    port: "{{ mongodb_port }}"
    delay: 5
    timeout: 30
  when: mongodb_admin_created is defined and mongodb_admin_created.changed

# Datenbanken und Benutzer erstellen
- name: Erstelle Anwendungsdatenbank und -benutzer
  shell: >
    mongosh --host {{ mongodb_bind_ip }} --port {{ mongodb_port }} 
    -u "{{ mongodb_admin_user }}" -p "{{ mongodb_admin_password }}" 
    --authenticationDatabase admin --eval '
    db = db.getSiblingDB("{{ item.database }}");
    db.createUser({
      user: "{{ item.name }}",
      pwd: "{{ item.password }}",
      roles: [ { role: "{{ item.roles }}", db: "{{ item.database }}" } ]
    })'
  with_items: "{{ mongodb_users }}"
  when: mongodb_auth_enabled and mongodb_users | length > 0
  register: mongodb_app_users
  changed_when: mongodb_app_users.rc == 0
  failed_when: false
  # Hinweis: In der Produktion sollten wir die Ausgabe verbergen, da sie Passwörter enthalten könnte

# Stelle sicher, dass der Service aktiviert und gestartet ist
- name: Stelle sicher, dass MongoDB läuft und beim Systemstart aktiviert ist
  service:
    name: mongod
    state: started
    enabled: yes
  become: yes
```

**Neue Konzepte in diesem Task-File:**

1. **apt_key**: Verwaltet APT-GPG-Schlüssel, die für Paketrepositorys benötigt werden.

2. **copy vs. template**: `copy` kopiert eine statische Datei, während `template` eine Jinja2-Vorlage verarbeitet.

3. **wait_for**: Wartet, bis ein Dienst verfügbar ist (in diesem Fall, bis MongoDB läuft und Verbindungen annimmt).

4. **mongodb_user**: Ein spezielles Modul für die Verwaltung von MongoDB-Benutzern.

5. **with_items**: Schleife zum Iterieren über eine Liste von Elementen.

Diese Tasks führen die Installation und Konfiguration von MongoDB in einer logischen Reihenfolge durch, wobei sichergestellt wird, dass die Authentifizierung korrekt eingerichtet ist, bevor Datenbankbenutzer erstellt werden.

### 3.4 MongoDB-Konfigurationstemplate erstellen

```bash
# erstelle das mongodb-konfigurationstemplate
nano templates/mongod.conf.j2
```

Füge folgende Konfiguration hinzu:

```yaml
# MongoDB-Konfigurationsdatei
# Erstellt durch Ansible auf {{ ansible_hostname }}
# MongoDB-Version {{ mongodb_version }}

# Wo und wie der Prozess Daten speichert
storage:
  dbPath: {{ mongodb_data_dir }}
  journal:
    enabled: {{ mongodb_journal_enabled | lower }}
  wiredTiger:
    engineConfig:
      cacheSizeGB: {{ mongodb_wiredtiger_cache_size }}

# Wo Logs gespeichert werden
systemLog:
  destination: file
  logAppend: true
  path: {{ mongodb_log_dir }}/mongod.log

# Netzwerk-Schnittstellen
net:
  port: {{ mongodb_port }}
  bindIp: {{ mongodb_bind_ip }}

# Authentifizierung
{% if mongodb_auth_enabled %}
security:
  authorization: enabled
{% endif %}

# Prozessmanagement-Optionen
processManagement:
  timeZoneInfo: /usr/share/zoneinfo
  fork: true  # Fork the process in the background
```

Dieses Template verwendet YAML-Syntax für die MongoDB-Konfiguration und fügt dynamische Werte basierend auf unseren Variablen ein. Der Abschnitt `security` wird nur hinzugefügt, wenn `mongodb_auth_enabled` auf `true` gesetzt ist.

### 3.5 Handler erstellen

```bash
# erstelle handler-datei
nano handlers/main.yml
```

Füge folgenden Handler hinzu:

```yaml
---
# Handler für MongoDB

- name: Restart MongoDB
  service:
    name: mongod
    state: restarted
  become: yes
```

Dieser Handler wird aufgerufen, wenn Änderungen an der MongoDB-Konfiguration vorgenommen werden, und sorgt dafür, dass der MongoDB-Server mit der neuen Konfiguration neu gestartet wird.

## 4. Hauptplaybook erstellen

Jetzt erstellen wir das Hauptplaybook, das unsere MongoDB-Rolle verwendet:

```bash
# wechsle ins projektverzeichnis
cd ~/ansible-mongodb-basic

# erstelle das hauptplaybook
nano mongodb-setup.yml
```

Füge folgenden Inhalt hinzu:

```yaml
---
# Playbook zur Installation und Konfiguration von MongoDB

- name: Setup MongoDB Server
  hosts: dbservers
  gather_facts: yes

  # Systemvoraussetzungen prüfen
  pre_tasks:
    - name: Prüfe minimalen erforderlichen Speicher
      assert:
        that:
          - ansible_memtotal_mb >= 1024
        msg: "MongoDB benötigt mindestens 1 GB RAM für optimale Leistung"

    - name: Prüfe verfügbaren Festplattenspeicher
      shell: "df -h / | awk 'NR==2 {print $4}' | sed 's/G//'"
      register: available_disk
      changed_when: false
      
    - name: Prüfe, ob genügend Festplattenspeicher vorhanden ist
      assert:
        that:
          - available_disk.stdout | float >= 5
        msg: "MongoDB benötigt mindestens 5 GB freien Speicherplatz"
      
    - name: Ankündigung
      debug:
        msg: "Starte MongoDB-Installation und -Konfiguration..."

  # Die MongoDB-Rolle anwenden
  roles:
    - mongodb
  
  # Aufgaben nach der Rollenausführung
  post_tasks:
    - name: Teste die MongoDB-Verbindung
      shell: >
        mongosh --host {{ mongodb_bind_ip }} --port {{ mongodb_port }} 
        -u "{{ mongodb_admin_user }}" -p "{{ mongodb_admin_password }}" 
        --authenticationDatabase admin
        --quiet --eval "db.runCommand({ connectionStatus: 1 })"
      register: mongodb_status
      changed_when: false
      ignore_errors: yes  # Falls mongosh nicht verfügbar ist oder Authentifizierung fehlschlägt

    - name: Zeige MongoDB-Verbindungsstatus
      debug:
        var: mongodb_status.stdout
      when: mongodb_status is succeeded

    - name: Installiere MongoDB Shell, falls nicht verfügbar
      apt:
        name: mongodb-mongosh
        state: present
      become: yes
      when: mongodb_status is failed

    - name: Erstelle ein einfaches Test-Dokument in der Anwendungsdatenbank
      shell: >
        mongosh --host {{ mongodb_bind_ip }} --port {{ mongodb_port }} 
        -u "{{ mongodb_users[0].name }}" -p "{{ mongodb_users[0].password }}" 
        --authenticationDatabase {{ mongodb_users[0].database }}
        --quiet --eval 'db.test_collection.insertOne({
          "name": "Ansible Testdokument",
          "erstellt_am": new Date(),
          "status": "aktiv",
          "eigenschaften": {
            "automatisiert": true,
            "system": "{{ ansible_distribution }} {{ ansible_distribution_version }}"
          }
        })'
      when: mongodb_status is succeeded
      register: mongo_insert
      changed_when: mongo_insert.rc == 0
      failed_when: false

    - name: Zeige MongoDB-Testdaten
      debug:
        msg: "Test-Dokument wurde erstellt: {{ mongo_insert.changed }}"
      when: mongo_insert is succeeded

    - name: Erfolgreiche Installation
      debug:
        msg: |
          MongoDB wurde erfolgreich installiert und konfiguriert!
          
          Verbindungsinformationen:
          - Host: {{ mongodb_bind_ip }}
          - Port: {{ mongodb_port }}
          - Admin-Benutzer: {{ mongodb_admin_user }}
          - Anwendungs-DB: {{ mongodb_users[0].database }}
          - Anwendungs-Benutzer: {{ mongodb_users[0].name }}
          
          Verbindungsbeispiel:
          mongosh --host {{ mongodb_bind_ip }} --port {{ mongodb_port }} -u {{ mongodb_users[0].name }} -p --authenticationDatabase {{ mongodb_users[0].database }}
```

**Neue Elemente in diesem Playbook:**

1. **Komplexere assert-Bedingungen** zur Prüfung der Systemvoraussetzungen.

2. **shell-Modul** für Befehle, die nicht einfach mit dem command-Modul ausgeführt werden können.

3. **mongodb_shell-Modul** zum Ausführen von MongoDB-Shell-Befehlen.

4. **Mehrzeilige Ausgabe** im debug-Modul mit der Pipe-Syntax (`|`).

5. **Bedingungsausdrücke** wie `is succeeded` und `is failed` für elegantere Bedingungen.

Diese Elemente machen das Playbook robuster und informativer für den Benutzer.

## 5. Führe das Playbook aus

Jetzt können wir das Playbook ausführen:

```bash
# führe das playbook aus
ansible-playbook -i inventory.ini mongodb-setup.yml
```

Oder mit zusätzlichen Optionen für mehr Informationen:

```bash
# führe das playbook im diff-modus und mit zusätzlichen informationen aus
ansible-playbook -i inventory.ini mongodb-setup.yml --diff -v
```

**Hinweis:** Die erste Ausführung kann einige Minuten dauern, da Pakete heruntergeladen und installiert werden müssen. Bei weiteren Ausführungen sollten die meisten Tasks als "ok" (keine Änderungen) markiert sein, was die **Idempotenz** von Ansible demonstriert.

**Wichtig:** Bei der ersten Ausführung könnten einige Fehler beim Erstellen von Benutzern auftreten, das ist normal, da wir die Authentifizierung erst einrichten müssen. Das Playbook ist so gestaltet, dass es trotz dieser initialen Fehler erfolgreich abschließt.

**Fehlerbehebung:** Falls du auf Probleme stößt, überprüfe Folgendes:
1. MongoDB-Dienststatus: `sudo systemctl status mongod`
2. MongoDB-Logs: `sudo cat /var/log/mongodb/mongod.log`
3. Netzwerkverbindung: `netstat -tulpn | grep 27017`
4. Firewall-Einstellungen: `sudo ufw status`

## 6. Test der MongoDB-Installation

Nach der erfolgreichen Ausführung des Playbooks können wir die MongoDB-Installation manuell testen:

```bash
# verbinde mit mongodb als admin
mongosh --host 127.0.0.1 --port 27017 -u admin -p --authenticationDatabase admin
```

Gib das in `mongodb_admin_password` definierte Passwort ein.

Innerhalb der MongoDB-Shell können wir einige Befehle ausführen:

```javascript
// Zeige verfügbare Datenbanken an
show dbs

// Wechsle zur Anwendungsdatenbank
use appdb

// Zeige vorhandene Collections an
show collections

// Zeige das Test-Dokument an
db.test_collection.find()

// Füge weitere Test-Dokumente hinzu
db.test_collection.insertOne({
  name: "Weiterer Testdatensatz",
  wert: 42,
  tags: ["test", "ansible", "automation"]
})

// Führe eine einfache Abfrage aus
db.test_collection.find({name: /Test/})

// Beende die MongoDB-Shell
exit
```

Nun können wir auch die Verbindung mit dem Anwendungsbenutzer testen:

```bash
# verbinde mit mongodb als anwendungsbenutzer
mongosh --host 127.0.0.1 --port 27017 -u appuser -p --authenticationDatabase appdb
```

Gib das in `mongodb_users` definierte Passwort ein.

```javascript
// Zeige verfügbare Datenbanken an
show dbs

// Wechsle zur Anwendungsdatenbank
use appdb

// Sieh dir die Test-Collection an
db.test_collection.find()

// Beende die MongoDB-Shell
exit
```

## 7. Erweitere die Rolle um Umgebungskonfigurationen

Jetzt können wir unsere Rolle erweitern, um verschiedene Umgebungen zu unterstützen (Entwicklung, Produktion, etc.). Dafür erstellen wir umgebungsspezifische Verzeichnisse mit eigenen Variablen:

```bash
# erstelle verzeichnisse für verschiedene umgebungen
mkdir -p environments/{dev,prod}/group_vars/dbservers
```

Erstelle Entwicklungsumgebungsvariablen:

```bash
nano environments/dev/group_vars/dbservers/vars.yml
```

```yaml
---
# Entwicklungsumgebung (weniger restriktiv)
mongodb_bind_ip: "0.0.0.0"  # Erlaube Verbindungen von überall (nur für Entwicklung!)
mongodb_wiredtiger_cache_size: 0.25  # Weniger Speicher für Entwicklung
mongodb_admin_password: "devPassword123"  # Einfacheres Passwort für Entwicklung
mongodb_users:
  - {
      name: "devuser",
      password: "devPass123",
      database: "devdb",
      roles: "readWrite"
    }
```

Erstelle Produktionsumgebungsvariablen:

```bash
nano environments/prod/group_vars/dbservers/vars.yml
```

```yaml
---
# Produktionsumgebung (sicherer und leistungsstärker)
mongodb_bind_ip: "127.0.0.1"  # Nur lokale Verbindungen (verwende Reverse-Proxy für externe Zugriffe)
mongodb_wiredtiger_cache_size: 0.6  # Mehr Cache für Produktion
mongodb_admin_password: "Prod#Secure$Password@456"  # Stärkeres Passwort für Produktion
mongodb_users:
  - {
      name: "appuser",
      password: "App#Secure$Password@789",
      database: "production_db",
      roles: "readWrite"
    }
  - {
      name: "readonly_user",
      password: "Read#Only$Password@321",
      database: "production_db",
      roles: "read"
    }
```

### 7.1 Umgebungsspezifische Inventories erstellen

```bash
# erstelle umgebungsspezifische inventories
cp inventory.ini environments/dev/inventory.ini
cp inventory.ini environments/prod/inventory.ini
```

### 7.2 Führe das Playbook in verschiedenen Umgebungen aus

```bash
# für entwicklungsumgebung
ansible-playbook -i environments/dev/inventory.ini mongodb-setup.yml

# für produktionsumgebung
ansible-playbook -i environments/prod/inventory.ini mongodb-setup.yml
```

Beobachte die Unterschiede in der MongoDB-Konfiguration, besonders die IP-Bindung, Cachegröße und Benutzereinstellungen.

## 8. Erweitere die Rolle um Backup-Funktionalität

Als zusätzliche Übung können wir das Playbook um eine einfache Backup-Lösung erweitern:

```bash
# erstelle backup-skript-template
nano roles/mongodb/templates/mongodb_backup.sh.j2
```

Füge folgenden Inhalt hinzu:

```bash
#!/bin/bash
# MongoDB Backup-Skript
# Erstellt durch Ansible am {{ ansible_date_time.date }}

# Konfiguration
MONGO_USER="{{ mongodb_admin_user }}"
MONGO_PASSWORD="{{ mongodb_admin_password }}"
MONGO_HOST="{{ mongodb_bind_ip }}"
MONGO_PORT="{{ mongodb_port }}"
BACKUP_DIR="/var/backups/mongodb"
DATE=$(date +%Y%m%d-%H%M%S)
RETENTION_DAYS=7

# Sicherstellen, dass das Backup-Verzeichnis existiert
mkdir -p $BACKUP_DIR

# Durchführen des Backups mit mongodump
mongodump --host $MONGO_HOST --port $MONGO_PORT -u $MONGO_USER -p $MONGO_PASSWORD --authenticationDatabase admin --out $BACKUP_DIR/$DATE

# Komprimieren des Backups
cd $BACKUP_DIR
tar -czf $DATE.tar.gz $DATE
rm -rf $DATE

# Löschen alter Backups
find $BACKUP_DIR -name "*.tar.gz" -type f -mtime +$RETENTION_DAYS -delete

echo "MongoDB Backup abgeschlossen: $BACKUP_DIR/$DATE.tar.gz"
exit 0
```

Jetzt fügen wir Tasks für das Backup hinzu:

```bash
nano roles/mongodb/tasks/backup.yml
```

```yaml
---
# Tasks für MongoDB-Backups

- name: Stelle sicher, dass Backup-Verzeichnis existiert
  file:
    path: /var/backups/mongodb
    state: directory
    owner: root
    group: root
    mode: '0750'
  become: yes

- name: Installiere benötigte Pakete für Backups
  apt:
    name:
      - cron
      - tar
      - gzip
    state: present
  become: yes

- name: Erstelle MongoDB-Backup-Skript
  template:
    src: mongodb_backup.sh.j2
    dest: /usr/local/bin/mongodb_backup.sh
    owner: root
    group: root
    mode: '0755'
  become: yes

- name: Richte MongoDB-Backup-Cronjob ein
  cron:
    name: "MongoDB Backup"
    hour: "1"
    minute: "0"
    job: "/usr/local/bin/mongodb_backup.sh"
  become: yes
```

Füge die Backup-Tasks in der Hauptaufgabendatei ein:

```bash
nano roles/mongodb/tasks/main.yml
```

Am Ende der Datei folgendes hinzufügen:

```yaml
# Backup-Konfiguration einbinden (nur für Produktionsumgebungen)
- name: Konfiguriere Backups
  import_tasks: backup.yml
  when: ansible_env.ENVIRONMENT is defined and ansible_env.ENVIRONMENT == 'prod'
```

Um die Umgebungsvariable zu setzen, führen wir das Playbook so aus:

```bash
# Für Produktionsumgebung mit Backup-Konfiguration
ENVIRONMENT=prod ansible-playbook -i environments/prod/inventory.ini mongodb-setup.yml
```

## 9. Bereinigen

Wenn du mit der Übung fertig bist, kannst du MongoDB wieder entfernen:

```bash
# erstelle ein cleanup-playbook
nano cleanup.yml
```

Füge folgenden Inhalt hinzu:

```yaml
---
# Einfaches Playbook zum Entfernen von MongoDB

- name: Remove MongoDB
  hosts: dbservers
  become: yes
  
  tasks:
    - name: Stop MongoDB service
      service:
        name: mongod
        state: stopped
      
    - name: Remove MongoDB packages
      apt:
        name: 
          - mongodb-org
          - mongodb-org-server
          - mongodb-org-shell
          - mongodb-org-mongos
          - mongodb-org-tools
          - mongodb-mongosh
        state: absent
        purge: yes
        
    - name: Remove MongoDB data directory
      file:
        path: "{{ item }}"
        state: absent
      with_items:
        - /var/lib/mongodb
        - /var/log/mongodb
        - /var/backups/mongodb
        
    - name: Remove MongoDB configuration
      file:
        path: "{{ item }}"
        state: absent
      with_items:
        - /etc/mongod.conf
        - /usr/local/bin/mongodb_backup.sh
        
    - name: Remove MongoDB repository
      file:
        path: /etc/apt/sources.list.d/mongodb-org-*.list
        state: absent
        
    - name: Update apt cache
      apt:
        update_cache: yes
```

Führe das Cleanup-Playbook aus:

```bash
ansible-playbook -i inventory.ini cleanup.yml
```

## 10. Zusammenfassung

In dieser Übung hast du:

1. Eine Ansible-Rolle für MongoDB erstellt und konfiguriert
2. Mit externen Paketquellen in Ansible gearbeitet
3. Die Verwendung von Shell-Befehlen für MongoDB-Administration gelernt
4. Bedingte Ausführung und Fehlerbehandlung implementiert
5. Umgebungsspezifische Konfigurationen erstellt
6. Ein einfaches Backup-System eingerichtet

Die Verwendung von Ansible zur Automatisierung deiner MongoDB-Infrastruktur bietet mehrere Vorteile:
- **Reproduzierbarkeit**: Identische Konfigurationen über verschiedene Server hinweg
- **Selbstdokumentation**: Deine Infrastruktur ist als Code definiert und damit dokumentiert
- **Schnelles Deployment**: Neue Server können schnell und zuverlässig bereitgestellt werden
- **Wartbarkeit**: Änderungen können systematisch und konsistent auf allen Servern vorgenommen werden

**Wo kannst du von hier aus weitermachen?**

- Implementiere eine MongoDB-Replikation mit mehreren Servern
- Füge Sicherheitsmaßnahmen wie TLS/SSL-Verschlüsselung hinzu
- Integriere eine Monitoring-Lösung wie Prometheus mit MongoDB-Exporter
- Verbinde diese MongoDB-Rolle mit deiner Nginx-Rolle, um einen MEAN- oder MERN-Stack zu erstellen
- Erweitere deine Kenntnisse mit der offiziellen [MongoDB-Dokumentation](https://docs.mongodb.com/) und [Ansible-Dokumentation](https://docs.ansible.com/)

> **Tipp:** Um tiefer in Ansible und Datenbankautomatisierung einzusteigen, schau dir in den [Unterlagen](https://github.com/JacobMenge/lern-unterlagen/blob/main/ansible/ansible-basics.md) besonders die Abschnitte zu Variablenprioritäten, bedingten Ausführungen und Rollenwiederverwendung an.
