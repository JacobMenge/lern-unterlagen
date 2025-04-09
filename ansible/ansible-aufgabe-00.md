# Ansible "Hello World" - Eine praktische Einführung

Diese Übung führt dich Schritt für Schritt durch deine ersten Erfahrungen mit Ansible in einer WSL-Umgebung (Windows Subsystem for Linux). Du wirst lernen, wie du Ansible installierst, ein einfaches Playbook erstellst und es ausführst. Solltest du während der Übung Probleme haben oder Fragen aufkommen, melde dich gerne jederzeit per Slack bei mir. Ich helfe dir gerne weiter!

> **Hinweis:** Detaillierte Hintergrundinformationen zu Ansible findest du in den Unterlagen unter [ansible-basics.md](https://github.com/JacobMenge/lern-unterlagen/blob/main/ansible/ansible-basics.md). Besonders die Abschnitte 1.2 "Was ist Ansible und warum solltest du es nutzen?" und 1.4 "Kernkonzepte: Playbooks, Rollen, Inventories, Module" sind hilfreich für diese Übung.

## 1. Installation von Ansible in WSL

### 1.1 Ansible in WSL Ubuntu installieren

Öffne dein WSL Ubuntu-Terminal und führe folgende Befehle aus:

```bash
# system-pakete aktualisieren
# dieser schritt stellt sicher dass die paketlisten deines systems auf dem neuesten stand sind
sudo apt update

# optional installierte pakete aktualisieren
# dies bringt alle installierten pakete auf den neuesten stand
sudo apt upgrade -y

# methode 1: installation über das offizielle ppa (empfohlen für ubuntu 20.04)
# benötigte abhängigkeiten installieren
sudo apt install -y software-properties-common

# ansible ppa hinzufügen
# fügt das offizielle ansible-repository hinzu um die neueste version zu bekommen
sudo apt-add-repository --yes --update ppa:ansible/ansible

# ansible installieren
sudo apt install -y ansible

# falls das ppa einen fehler wirft, kannst du methode 2 verwenden:
# methode 2: direkte installation aus den ubuntu-repositories
# sudo apt install -y ansible
```

**Warum dieser Installationsweg?** 
- Der PPA-Installationsweg stellt sicher, dass du immer Zugriff auf die neueste stabile Ansible-Version hast
- Es ist der von Ansible empfohlene Weg für Ubuntu-basierte Systeme
- Alle Abhängigkeiten werden automatisch installiert und verwaltet
- Spätere Updates können einfach über `apt upgrade` durchgeführt werden

**Hinweis:** In neueren Ubuntu-Versionen (ab 22.04) ist die direkte Installation ohne PPA eventuell vorzuziehen, da beide Methoden ähnlich aktuelle Versionen liefern.

### 1.2 Alternative Installationsmethode: Pip (Python-Paketmanager)

Falls du aus irgendeinem Grund die Paketmanager-Installation nicht verwenden kannst oder möchtest, gibt es auch die Möglichkeit, Ansible über Pip zu installieren:

```bash
# python3 und pip installieren
sudo apt update
sudo apt install -y python3 python3-pip python3-venv

# optional aber empfohlen: virtuelle umgebung erstellen
# isoliert die installation und vermeidet konflikte mit systempaketen
python3 -m venv ~/ansible-env
source ~/ansible-env/bin/activate

# ansible über pip installieren
pip3 install ansible

# wenn du keine virtuelle umgebung nutzt, installiere für den aktuellen benutzer
# pip3 install --user ansible
# echo 'export PATH="$HOME/.local/bin:$PATH"' >> ~/.bashrc
# source ~/.bashrc
```

**Hinweis:** Die Pip-Installation mit virtueller Umgebung bietet die aktuellste Ansible-Version und vermeidet Konflikte. Bei Problemen mit der Pfadkonfiguration führe `which ansible` aus, um zu prüfen, ob Ansible im PATH ist.

### 1.3 Überprüfe die Installation

Nach der Installation solltest du prüfen, ob Ansible korrekt installiert wurde:

```bash
# prüfe die installierte ansible-version
ansible --version

# wenn du ein "command not found" siehst, schließe dein terminal und öffne es neu
# oder führe aus: source ~/.bashrc
```

Du solltest eine Ausgabe ähnlich dieser sehen:
```
ansible [core 2.16.x]
  config file = /etc/ansible/ansible.cfg
  configured module search path = ['/home/username/.ansible/plugins/modules', '/usr/share/ansible/plugins/modules']
  ansible python module location = /usr/lib/python3/dist-packages/ansible
  ansible collection location = /home/username/.ansible/collections:/usr/share/ansible/collections
  executable location = /usr/bin/ansible
  python version = 3.10.x (main, Mar 01 2023, 12:34:56) [GCC 11.3.0]
  jinja version = 3.0.3
  libyaml = True
```

**Was diese Ausgabe bedeutet:**
- Die erste Zeile zeigt die installierte Ansible-Version
- `config file` zeigt den Pfad zur Hauptkonfigurationsdatei
- `module search path` zeigt, wo Ansible nach Modulen sucht
- Die restlichen Informationen betreffen die zugrunde liegenden Komponenten und ihre Standorte

## 2. Ein einfaches Ansible-Projekt erstellen

### 2.1 Projektverzeichnis anlegen

```bash
# erstelle ein verzeichnis für dein erstes ansible-projekt
mkdir -p ~/ansible-hello-world
cd ~/ansible-hello-world
```

**Warum?** Eine saubere Verzeichnisstruktur hilft dir, den Überblick zu behalten und folgt den Best Practices für Ansible-Projekte. Die `-p` Option erstellt auch übergeordnete Verzeichnisse, falls sie nicht existieren.

### 2.2 Inventory-Datei erstellen

Für unsere erste Übung werden wir Ansible verwenden, um einen Befehl auf dem lokalen System (localhost) auszuführen. Erstelle eine Inventory-Datei:

```bash
# erstelle und öffne die inventory-datei
nano inventory.ini
```

Füge folgenden Inhalt hinzu:

```ini
[local]
localhost ansible_connection=local
```

Speichere die Datei mit STRG+O, bestätige mit Enter und verlasse den Editor mit STRG+X.

**Erklärung:**
- `[local]` definiert eine Gruppe namens "local"
- `localhost` ist der Name des Hosts
- `ansible_connection=local` weist Ansible an, keine SSH-Verbindung herzustellen, sondern direkt lokale Befehle auszuführen

### 2.3 Einfaches Playbook erstellen

Jetzt erstellen wir ein einfaches Playbook, das einen "Hello World"-Befehl ausführt:

```bash
# erstelle und öffne die playbook-datei
nano hello-world.yml
```

Füge folgenden Inhalt hinzu:

```yaml
---
- name: Hello World Playbook
  hosts: local
  gather_facts: yes  # stellt sicher dass ansible facts verfügbar sind
  
  tasks:
    - name: Echo Hello World
      command: echo "Hello, Ansible World!"
      register: hello_output
    
    - name: Zeige die Ausgabe an
      debug:
        var: hello_output.stdout
```

Speichere die Datei mit STRG+O, bestätige mit Enter und verlasse den Editor mit STRG+X.

**Erklärung:**
- `name: Hello World Playbook` gibt dem Playbook einen beschreibenden Namen
- `hosts: local` legt fest, dass dieses Playbook auf allen Hosts in der Gruppe "local" ausgeführt werden soll
- `gather_facts: yes` ist standardmäßig aktiviert und sammelt Informationen über die Zielsysteme
- Die erste Task verwendet das `command`-Modul, um den Echo-Befehl auszuführen
- `register: hello_output` speichert die Ausgabe des Befehls in einer Variable
- Die zweite Task verwendet das `debug`-Modul, um die Ausgabe anzuzeigen

> **Tipp:** Mehr über die YAML-Syntax und Playbook-Struktur findest du in Abschnitt 2.1 "YAML-Syntax" und 2.2 "Ansible-Workflow: Playbook-Ausführung" in den [Unterlagen](https://github.com/JacobMenge/lern-unterlagen/blob/main/ansible/ansible-basics.md).

## 3. Playbook ausführen

### 3.1 Syntax-Check

Bevor wir das Playbook ausführen, ist es eine gute Praxis, die Syntax zu überprüfen:

```bash
# überprüfe die syntax des playbooks
ansible-playbook -i inventory.ini hello-world.yml --syntax-check
```

**Warum?** Der Syntax-Check hilft, einfache Fehler frühzeitig zu erkennen, bevor das Playbook ausgeführt wird.

### 3.2 Playbook ausführen

Wenn der Syntax-Check erfolgreich war, können wir das Playbook ausführen:

```bash
# führe das playbook aus
ansible-playbook -i inventory.ini hello-world.yml
```

Du solltest eine Ausgabe ähnlich dieser sehen:

```
PLAY [Hello World Playbook] ******************************************

TASK [Gathering Facts] **********************************************
ok: [localhost]

TASK [Echo Hello World] *********************************************
changed: [localhost]

TASK [Zeige die Ausgabe an] *****************************************
ok: [localhost] => {
    "hello_output.stdout": "Hello, Ansible World!"
}

PLAY RECAP *********************************************************
localhost                  : ok=3    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

**Erklärung der Ausgabe:**
- "Gathering Facts" ist ein automatischer Task, der Informationen über den Zielhost sammelt
- "Echo Hello World" zeigt den Status unseres ersten Tasks
- "Zeige die Ausgabe an" zeigt die registrierte Ausgabe
- Die "PLAY RECAP" fasst die Ausführungsergebnisse zusammen

## 4. Erweitere dein Playbook

Lass uns das Playbook erweitern, um mehr über Ansible-Konzepte zu lernen:

```bash
# öffne das playbook erneut
nano hello-world.yml
```

Erweitere es mit folgendem Inhalt:

```yaml
---
- name: Hello World Playbook
  hosts: local
  vars:
    greeting: "Hello, Ansible World!"
    current_time: "{{ ansible_date_time.iso8601 }}"
  
  tasks:
    - name: Echo Hello World
      command: echo "{{ greeting }}"
      register: hello_output
    
    - name: Zeige die Ausgabe an
      debug:
        var: hello_output.stdout
    
    - name: Zeige aktuelle Zeit an
      debug:
        msg: "Die aktuelle Zeit ist {{ current_time }}"
    
    - name: Erstelle eine Hello-World-Datei
      copy:
        content: |
          {{ greeting }}
          Diese Datei wurde mit Ansible erstellt am {{ current_time }}.
        dest: ~/hello-ansible.txt
        mode: '0644'
```

Speichere und führe das Playbook erneut aus:

```bash
# führe das erweiterte playbook aus
ansible-playbook -i inventory.ini hello-world.yml
```

**Erklärung der neuen Elemente:**
- `vars:` definiert Variablen, die im Playbook verwendet werden können
- `{{ ansible_date_time.iso8601 }}` ist ein Beispiel für einen "Fact" - Informationen, die Ansible automatisch sammelt
- Das `copy`-Modul erstellt eine Datei mit angegebenem Inhalt
- `mode: '0644'` setzt die Berechtigungen für die Datei

> **Hinweis:** Mehr über Variablen und Facts in Ansible findest du im Abschnitt 3.1 "Variablen und Facts" der [Unterlagen](https://github.com/JacobMenge/lern-unterlagen/blob/main/ansible/ansible-basics.md). Module wie das hier verwendete `copy`-Modul werden in Abschnitt 1.4 "Kernkonzepte" beschrieben.

### 4.1 Überprüfe das Ergebnis

```bash
# prüfe ob die datei erstellt wurde
cat ~/hello-ansible.txt
```

## 5. Idempotenz verstehen

Führe das Playbook ein zweites Mal aus:

```bash
# führe das playbook erneut aus
ansible-playbook -i inventory.ini hello-world.yml
```

Beachte, dass jetzt weniger Tasks als "changed" markiert sind. Dies demonstriert das wichtige Konzept der **Idempotenz** in Ansible: Ein Playbook kann mehrfach ausgeführt werden, ohne unerwünschte Nebenwirkungen zu verursachen.

Beim zweiten Durchlauf erkennt Ansible, dass die Datei bereits existiert und ihren gewünschten Zustand hat, daher muss sie nicht erneut erstellt werden.

> **Wichtig:** Das Konzept der Idempotenz ist eines der Kernprinzipien von Ansible und wird in Abschnitt 1.2 "Was ist Ansible und warum solltest du es nutzen?" der [Unterlagen](https://github.com/JacobMenge/lern-unterlagen/blob/main/ansible/ansible-basics.md) genauer erklärt. Dort heißt es: "Ansible-Operationen können mehrfach ausgeführt werden, ohne unerwünschte Nebenwirkungen zu verursachen. Das System prüft, ob der gewünschte Zustand bereits erreicht ist."

## 6. Weiterführende Übungen

Du hast jetzt erfolgreich:
1. Ansible in WSL installiert
2. Ein einfaches Inventory erstellt
3. Ein Playbook geschrieben und ausgeführt
4. Variablen und Module verwendet
5. Das Konzept der Idempotenz kennengelernt

Hier sind einige Ideen für nächste Schritte:

### 6.1 Erweitere dein Inventory

Füge einen weiteren Host hinzu, wenn du einen hast, oder erstelle eine neue Gruppe:

```bash
# öffne die inventory-datei
nano inventory.ini
```

Erweitere sie mit:

```ini
[local]
localhost ansible_connection=local

[development]
# hier könntest du einen entfernten server hinzufügen wenn verfügbar
# dev-server ansible_host=192.168.1.100 ansible_user=dein_benutzername
```

### 6.2 Erstelle eine einfache Rolle

```bash
# erstelle das hauptverzeichnis für rollen falls es nicht existiert
mkdir -p roles

# erstelle die rollenstruktur
mkdir -p roles/hello-role/{tasks,vars,defaults}

# erstelle die hauptaufgabendatei für die rolle
nano roles/hello-role/tasks/main.yml
```

Mit dem Inhalt:

```yaml
---
- name: Echo Hello von der Rolle
  command: echo "{{ role_greeting }}"
  register: role_output

- name: Zeige Rollen-Ausgabe
  debug:
    var: role_output.stdout
```

Erstelle Standardvariablen für die Rolle:

```bash
# erstelle standardvariablen
nano roles/hello-role/defaults/main.yml
```

Mit dem Inhalt:

```yaml
---
role_greeting: "Hello von der Ansible-Rolle!"
```

Aktualisiere dein Playbook, um die Rolle zu verwenden:

```bash
# erstelle ein neues playbook für die rolle
nano hello-world-with-role.yml
```

Mit dem Inhalt:

```yaml
---
- name: Hello World mit Rolle
  hosts: local
  roles:
    - hello-role
```

Führe das neue Playbook aus:

```bash
# führe das neue playbook mit rolle aus
ansible-playbook -i inventory.ini hello-world-with-role.yml
```

**Hinweis:** Wenn du einen Fehler wie "Role 'hello-role' not found" erhältst, stelle sicher, dass dein aktuelles Verzeichnis das Projekt-Hauptverzeichnis ist (`~/ansible-hello-world`). Du kannst mit `pwd` prüfen, in welchem Verzeichnis du dich befindest.

> **Tipp:** Die vollständige Struktur und Funktionsweise von Ansible-Rollen wird in Abschnitt 2.4 "Ansible-Rollen" der [Unterlagen](https://github.com/JacobMenge/lern-unterlagen/blob/main/ansible/ansible-basics.md) ausführlich erklärt. Dort findest du auch, wie eine typische Rollenstruktur aussieht und wie du Rollen optimal nutzen kannst.

## 7. Zusammenfassung

In dieser Übung hast du:
- Ansible in einer WSL-Umgebung installiert
- Ein einfaches Inventory und Playbook erstellt
- Variablen und Module verwendet
- Ein Playbook mehrfach ausgeführt und Idempotenz beobachtet
- Die Grundstruktur einer Ansible-Rolle kennengelernt

### 7.1 Fehlerbehebung

Hier sind einige häufige Probleme und ihre Lösungen:

- **"command not found"** nach der Installation: Schließe das Terminal und öffne es neu, oder führe `source ~/.bashrc` aus
- **Fehler beim Hinzufügen des PPAs**: Verwende die direkte Installation mit `sudo apt install ansible`
- **Probleme mit Python-Versionen**: Stelle sicher, dass Python 3 verwendet wird (Standard in neueren Ubuntu-Versionen)
- **"Role not found"**: Prüfe mit `pwd`, ob du im richtigen Verzeichnis bist, und stelle sicher, dass das Rollenverzeichnis korrekt angelegt wurde
- **Permission denied**: Überprüfe die Dateiberechtigungen mit `ls -la`

### 7.2 Nächste Schritte

Diese Grundlagen bilden das Fundament für komplexere Automatisierungsaufgaben mit Ansible. Als nächstes könntest du:
- Mehrere Server verwalten (siehe Abschnitt 2.3 "Inventory-Management" in den [Unterlagen](https://github.com/JacobMenge/lern-unterlagen/blob/main/ansible/ansible-basics.md))
- Komplexere Rollen entwickeln (Abschnitt 2.4 "Ansible-Rollen")
- Templates mit Jinja2 verwenden (Abschnitt 3.2 "Templates mit Jinja2")
- Bedingte Ausführung (when-Klauseln) einsetzen
- Mit Ansible Vault sensible Daten schützen

> **Tipp:** Um tiefer in die Ansible-Konzepte einzusteigen, arbeite die vollständigen [Unterlagen](https://github.com/JacobMenge/lern-unterlagen/blob/main/ansible/ansible-basics.md) durch. Sie enthalten zahlreiche Beispiele und detaillierte Erklärungen zu allen wichtigen Ansible-Funktionen.

Viel Erfolg auf deinem Weg mit Ansible!
