# Ansible Praxis: Einfache Nginx-Webserver Einrichtung

Diese Übung baut auf deinen neu erworbenen Ansible-Grundlagen aus der "Hello World"-Übung auf. Wir werden einen einfachen Nginx-Webserver einrichten und dabei einige zentrale Ansible-Konzepte praktisch anwenden.

> **Hinweis:** Für diese Übung solltest du bereits die Ansible "Hello World"-Übung abgeschlossen haben und mit den Grundlagen von Ansible vertraut sein. Zusätzliches Hintergrundwissen findest du in den [Ansible-Unterlagen](https://github.com/JacobMenge/lern-unterlagen/blob/main/ansible/ansible-basics.md).



## Was du in dieser Übung lernen wirst

In dieser Übung wirst du:
1. Eine einfache Ansible-Rolle erstellen
2. Templates für Konfigurationsdateien verwenden
3. Variablen in Ansible effektiv nutzen
4. Handler für Service-Neustarts implementieren
5. Eine kleine HTML-Seite mit Ansible bereitstellen

## Was ist Ansible nochmal und warum verwenden wir es?

Bevor wir starten, frischen wir kurz dein Wissen auf:

**Ansible** ist ein Automatisierungstool, das die Konfiguration von Servern und Anwendungen vereinfacht. Es ist:
- **Agentenlos**: Es benötigt keine vorinstallierten Programme auf den Zielservern
- **Deklarativ**: Du beschreibst den gewünschten Zustand, nicht die einzelnen Schritte
- **Idempotent**: Du kannst Ansible-Code mehrfach ausführen, ohne unerwünschte Effekte

**Idempotenz** ist ein wichtiges Konzept: Ein Playbook kann beliebig oft ausgeführt werden, und Ansible ändert nur, was tatsächlich geändert werden muss. Wenn du es zum zweiten Mal ausführst, erkennt Ansible, dass bestimmte Zustände bereits erreicht sind, und überspringt diese Schritte. Das macht Ansible-Code sicher und wiederholbar.

In dieser Übung werden wir diese Prinzipien anwenden, um einen Nginx-Webserver einzurichten.

## 1. Projektstruktur einrichten

Beginnen wir mit der Erstellung einer einfachen Verzeichnisstruktur:

```bash
# erstelle und wechsle in das projektverzeichnis
mkdir -p ~/ansible-nginx-basic
cd ~/ansible-nginx-basic

# erstelle die ansible-verzeichnisstruktur
mkdir -p roles/nginx/{tasks,templates,defaults,handlers}
```

**Erklärung der Verzeichnisstruktur:**
- `roles/nginx/tasks`: Enthält die auszuführenden Aufgaben (was getan werden soll)
- `roles/nginx/templates`: Enthält Jinja2-Vorlagen für dynamische Dateien (wie HTML oder Konfigurationsdateien)
- `roles/nginx/defaults`: Enthält Standardvariablen für die Rolle (Werte, die angepasst werden können)
- `roles/nginx/handlers`: Enthält Handler, die auf Änderungen reagieren (z.B. Dienst-Neustarts)

**Was ist eine Ansible-Rolle und warum benutzen wir sie?**

Eine **Rolle** in Ansible ist eine Art von Paket oder Modul, das zusammengehörige Konfigurationen und Aufgaben bündelt. Statt alle Anweisungen in einem einzigen, langen Playbook zu haben, organisieren wir verwandte Aufgaben in Rollen. Das macht deinen Code:
- Übersichtlicher und leichter zu verstehen
- Wiederverwendbar für verschiedene Projekte
- Einfacher zu warten und zu aktualisieren

In unserer Übung erstellen wir eine `nginx`-Rolle, die alles enthält, was zur Installation und Konfiguration eines Nginx-Webservers nötig ist.

## 2. Inventory-Datei erstellen

Für diese Übung verwenden wir wieder den lokalen Server:

```bash
# erstelle das inventory
nano inventory.ini
```

Füge folgenden Inhalt hinzu:

```ini
[webserver]
localhost ansible_connection=local

[all:vars]
ansible_python_interpreter=/usr/bin/python3
```

**Erklärung des Inventorys:**
- `[webserver]` definiert eine Gruppe namens "webserver"
- `localhost ansible_connection=local` gibt an, dass wir auf dem lokalen System arbeiten und keine SSH-Verbindung benötigen
- `ansible_python_interpreter=/usr/bin/python3` sorgt dafür, dass Ansible Python 3 nutzt

## 3. Nginx-Rolle erstellen

### 3.1 Standardvariablen definieren

```bash
# wechsle in das nginx-rollen-verzeichnis
cd ~/ansible-nginx-basic/roles/nginx

# erstelle standardvariablen
nano defaults/main.yml
```

Füge folgende einfache Standardvariablen hinzu:

```yaml
---
# Einfache Variablen für unseren Nginx-Server

# Der Name unserer Website
site_name: "Meine Ansible Demo-Seite"

# Eine kurze Beschreibung
site_description: "Diese Seite wurde mit Ansible bereitgestellt"

# Die Farbe der Überschrift
heading_color: "#2573CF"

# Der Port, auf dem Nginx laufen soll
nginx_port: 80

# Der Pfad zum Webroot
web_root: /var/www/html
```

**Was sind Standardvariablen und wozu dienen sie?**

Standardvariablen (defaults) in Ansible:
- Definieren Werte, die später in Playbooks und Templates verwendet werden
- Können überschrieben werden, wenn nötig
- Machen deinen Code flexibel und anpassbar

Diese Variablen werden später in unseren Templates verwendet, um die Webseite und die Nginx-Konfiguration anzupassen. Durch die Speicherung in einer separaten Datei kannst du diese Werte leicht ändern, ohne den eigentlichen Code ändern zu müssen.

### 3.2 Tasks für die Nginx-Installation erstellen

```bash
# erstelle die hauptaufgabendatei
nano tasks/main.yml
```

Füge folgende Tasks hinzu:

```yaml
---
# Tasks zur Installation und Konfiguration von Nginx

# Task 1: APT-Paketindex aktualisieren
# Dieser Schritt stellt sicher, dass wir die neuesten Paketversionen erhalten
- name: Update apt cache
  apt:
    update_cache: yes       # entspricht 'apt update'
    cache_valid_time: 3600  # Cache ist für 1 Stunde gültig
  become: yes               # mit Root-Rechten ausführen

# Task 2: Nginx installieren
# Mit diesem Task installieren wir den Nginx-Webserver
- name: Install Nginx
  apt:
    name: nginx             # Name des zu installierenden Pakets
    state: present          # 'present' bedeutet, dass das Paket installiert sein soll
  become: yes               # mit Root-Rechten ausführen

# Task 3: Eine benutzerdefinierte index.html-Datei erstellen
# Wir verwenden ein Template, um dynamische Inhalte einzufügen
- name: Create custom index.html
  template:
    src: index.html.j2      # Quelle (Jinja2-Template)
    dest: "{{ web_root }}/index.html"  # Ziel auf dem Server
    owner: www-data         # Besitzer der Datei (Nginx-Benutzer)
    group: www-data         # Gruppe der Datei
    mode: '0644'            # Dateirechte (lesbar für alle, schreibbar für Besitzer)
  become: yes               # mit Root-Rechten ausführen
  notify: Restart Nginx     # benachrichtigt den Handler "Restart Nginx"
                            # dieser wird nur ausgeführt, wenn die Datei geändert wurde

# Task 4: Eine einfache Nginx-Konfigurationsdatei erstellen
# Wir überschreiben die Standard-Konfiguration mit unserer eigenen
- name: Configure Nginx
  template:
    src: nginx.conf.j2      # Quelle (Jinja2-Template)
    dest: /etc/nginx/sites-available/default  # Ziel auf dem Server
    owner: root             # Besitzer der Datei
    group: root             # Gruppe der Datei
    mode: '0644'            # Dateirechte
  become: yes               # mit Root-Rechten ausführen
  notify: Restart Nginx     # benachrichtigt den Handler "Restart Nginx"

# Task 5: Stelle sicher, dass Nginx läuft und beim Systemstart aktiviert ist
- name: Ensure Nginx is running and enabled
  service:
    name: nginx             # Name des Dienstes
    state: started          # Dienst soll laufen
    enabled: yes            # Dienst soll beim Systemstart aktiviert sein
  become: yes               # mit Root-Rechten ausführen
```

**Wichtige Konzepte in diesem Task-File:**

1. **become: yes** - Dieser Parameter bedeutet, dass Ansible den Befehl mit erhöhten Rechten (sudo) ausführt. Dies ist nötig, weil Aktionen wie Paketinstallation und das Schreiben in bestimmte Verzeichnisse Root-Rechte erfordern.

2. **notify: Restart Nginx** - Dieser Parameter "benachrichtigt" den Handler "Restart Nginx", aber nur, wenn der Task tatsächlich etwas geändert hat. Der Handler wird erst am Ende des Playbooks ausgeführt, was effizienter ist als sofortiges Neustarten nach jeder Änderung.

3. **template-Modul** - Dieses Modul kopiert eine Datei auf den Zielserver, ersetzt aber vorher alle Variablen und Ausdrücke mit ihren tatsächlichen Werten. Templates haben üblicherweise die Erweiterung `.j2` (für Jinja2), dies ist aber nur eine Konvention, keine technische Anforderung.

4. **Variablen in {{ }}** - Ausdrücke in doppelten geschweiften Klammern werden durch die entsprechenden Werte ersetzt. Zum Beispiel wird `{{ web_root }}` durch `/var/www/html` ersetzt.

**Das apt-Modul und Paketmanagement in Ansible:**

Das `apt`-Modul ist eines der am häufigsten verwendeten Module in Ansible, besonders auf Debian-basierten Systemen wie Ubuntu. Es ist wichtig zu verstehen, was `state: present` bedeutet:

- `present`: Das Paket soll installiert sein, aber nicht unbedingt auf dem neuesten Stand
- `latest`: Das Paket soll installiert und auf dem neuesten Stand sein
- `absent`: Das Paket soll entfernt werden

Wir verwenden `present`, da wir nur sicherstellen wollen, dass Nginx installiert ist, unabhängig von der Version. In Produktionsumgebungen möchtest du vielleicht eine bestimmte Version angeben, um konsistente Umgebungen zu gewährleisten.

### 3.3 Handler erstellen

```bash
# erstelle handler-datei
nano handlers/main.yml
```

Füge einen einfachen Handler hinzu:

```yaml
---
# Handler für Nginx

# Dieser Handler wird aufgerufen, wenn ein Task mit "notify: Restart Nginx" beendet wird
# und der Task tatsächlich eine Änderung bewirkt hat.
# Er startet den Nginx-Dienst neu, damit Änderungen wirksam werden.

- name: Restart Nginx
  service:
    name: nginx             # Name des Dienstes
    state: restarted        # Aktion: Dienst neustarten
  become: yes               # mit Root-Rechten ausführen
```

**Was ist ein Handler und wann wird er verwendet?**

Ein Handler in Ansible ist eine spezielle Art von Task, der auf Benachrichtigungen reagiert:
- Er wird nur ausgeführt, wenn er mit `notify` aufgerufen wird
- Er wird nur ausgeführt, wenn der aufrufende Task tatsächlich etwas geändert hat
- Er wird am Ende des aktuellen Blocks von Tasks ausgeführt, nicht sofort

In unserem Fall verwenden wir den Handler, um Nginx neu zu starten, nachdem wir Konfigurationsdateien geändert haben. Dies ist effizienter als ein sofortiger Neustart nach jeder Änderung.

**Hinweis:** Für Nginx gibt es auch den Befehl `reload`, der laufende Verbindungen nicht unterbricht. In unserer einfachen Übung verwenden wir `restart`, aber in Produktionsumgebungen ist oft `reload` die bessere Wahl.

### 3.4 Templates erstellen

Erstellen wir die Templates für unsere Webseite und die Nginx-Konfiguration:

```bash
# erstelle das index.html template
nano templates/index.html.j2
```

Füge einen einfachen HTML-Code hinzu:

```html
<!DOCTYPE html>
<html>
<head>
    <title>{{ site_name }}</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            line-height: 1.6;
            margin: 0;
            padding: 20px;
            max-width: 800px;
            margin: 0 auto;
        }
        h1 {
            color: {{ heading_color }};
            border-bottom: 1px solid #ddd;
            padding-bottom: 10px;
        }
        .info-box {
            background-color: #f9f9f9;
            border: 1px solid #ddd;
            padding: 15px;
            border-radius: 5px;
            margin-top: 20px;
        }
        footer {
            margin-top: 30px;
            color: #777;
            font-size: 0.9em;
            text-align: center;
        }
    </style>
</head>
<body>
    <h1>{{ site_name }}</h1>
    <p>{{ site_description }}</p>
    
    <div class="info-box">
        <h2>Server-Informationen</h2>
        <ul>
            <li><strong>Hostname:</strong> {{ ansible_hostname }}</li>
            <li><strong>IP-Adresse:</strong> {{ ansible_default_ipv4.address }}</li>
            <li><strong>Betriebssystem:</strong> {{ ansible_distribution }} {{ ansible_distribution_version }}</li>
            <li><strong>Datum der Bereitstellung:</strong> {{ ansible_date_time.date }}</li>
        </ul>
    </div>
    
    <footer>
        Diese Seite wurde automatisch mit Ansible erstellt.
    </footer>
</body>
</html>
```

```bash
# erstelle die nginx.conf template
nano templates/nginx.conf.j2
```

Füge eine einfache Nginx-Konfiguration hinzu:

```nginx
# Einfache Nginx-Konfiguration für unsere Demo
# Erstellt mit Ansible auf {{ ansible_hostname }}

server {
    # Höre auf Port {{ nginx_port }}
    listen {{ nginx_port }} default_server;
    listen [::]:{{ nginx_port }} default_server;

    # Root-Verzeichnis für die Webseite
    root {{ web_root }};
    
    # Standard-Dateien, die gesucht werden sollen
    index index.html index.htm;

    # Server-Name (localhost)
    server_name _;

    # Konfiguration für den Root-Pfad
    location / {
        # Versuche zuerst die angeforderte URI, dann die URI mit einem Slash am Ende,
        # bei Misserfolg gib einen 404-Fehler zurück
        try_files $uri $uri/ =404;
    }
}
```

**Was sind Jinja2-Templates und wie funktionieren sie?**

Jinja2 ist die Template-Engine, die Ansible verwendet. Mit Templates kannst du:
- Dynamische Inhalte in statische Dateien einfügen
- Variablen und Ausdrücke verwenden (z.B. `{{ variable }}`)
- Logik in deine Dateien einbauen (z.B. Bedingungen, Schleifen)

In unseren Templates verwenden wir zwei Arten von Variablen:
1. **Benutzerdefinierte Variablen** wie `site_name` und `heading_color`, die wir in unserer `defaults/main.yml` definiert haben
2. **Ansible Facts** wie `ansible_hostname` und `ansible_distribution`, die automatisch von Ansible gesammelt werden, wenn `gather_facts: yes` gesetzt ist

**Was sind Ansible Facts?**

Ansible Facts sind automatisch gesammelte Informationen über den Zielserver. Sie umfassen:
- Betriebssystem und Version
- IP-Adressen und Netzwerkkonfiguration
- Verfügbarer Speicher und CPU-Informationen
- Aktuelle Uhrzeit und Datum
- Und vieles mehr...

Du kannst Facts in deinen Playbooks und Templates verwenden, wie wir es mit `{{ ansible_hostname }}` und anderen getan haben. Facts werden gesammelt, wenn du `gather_facts: yes` in deinem Playbook angibst.

## 4. Hauptplaybook erstellen

Jetzt erstellen wir das Hauptplaybook, das unsere Nginx-Rolle verwendet:

```bash
# wechsle ins projektverzeichnis
cd ~/ansible-nginx-basic

# erstelle das hauptplaybook
nano nginx-setup.yml
```

Füge folgenden Inhalt hinzu:

```yaml
---
# Einfaches Playbook zur Installation und Konfiguration von Nginx

- name: Setup Nginx Server
  hosts: webserver       # Zielgruppe aus dem Inventory
  gather_facts: yes      # Sammle Informationen über den Zielserver
  
  # Abschnitt 1: Pre-Tasks - werden vor den Rollen ausgeführt
  pre_tasks:
    - name: Ankündigung
      debug:
        msg: "Starte Nginx-Installation und -Konfiguration..."
  
  # Abschnitt 2: Rollen - die Hauptfunktionalität
  roles:
    - nginx
  
  # Abschnitt 3: Post-Tasks - werden nach den Rollen ausgeführt
  post_tasks:
    - name: Erfolg melden
      debug:
        msg: "Nginx wurde konfiguriert und läuft. Du kannst die Seite unter http://localhost aufrufen."
```

**Was passiert in diesem Playbook?**

Unser Playbook definiert eine strukturierte Ausführung in drei Abschnitten:

1. **hosts und gather_facts** - Der Kopfbereich definiert:
   - Auf welchen Hosts das Playbook ausgeführt werden soll (entspricht der Gruppe in unserem Inventory)
   - Dass Ansible Informationen über den Server sammeln soll (die wir später als Facts verwenden)

2. **pre_tasks** - Dies sind Aufgaben, die vor den Rollen ausgeführt werden. Sie eignen sich gut für:
   - Vorbereitende Schritte
   - Prüfungen, ob Voraussetzungen erfüllt sind
   - Anzeigen von Statusmeldungen

3. **roles** - Dies ist der Hauptblock, der angibt, welche Rollen angewendet werden sollen. In unserem Fall:
   - Nur die 'nginx' Rolle, die wir vorher erstellt haben
   - Rollen können auch Parameter übergeben bekommen, um ihr Verhalten anzupassen

4. **post_tasks** - Dies sind Aufgaben, die nach den Rollen ausgeführt werden. Sie eignen sich gut für:
   - Aufräumarbeiten
   - Statusprüfungen
   - Anzeigen von Erfolgsmeldungen oder nächsten Schritten

**Die Ausführungsreihenfolge ist wichtig zu verstehen:** 
1. Zuerst pre_tasks
2. Dann alle Tasks aus allen angegebenen Rollen
3. Anschließend post_tasks
4. Zum Schluss alle benachrichtigten Handler

Diese Strukturierung hilft dabei, Playbooks übersichtlich und logisch aufzubauen, besonders wenn sie komplexer werden.

## 5. Führe das Playbook aus

> **Hinweis:**  
> Beim Ausführen von Ansible-Playbooks, die `sudo`-Rechte benötigen, kann folgender Fehler auftreten:
>
> ```
> fatal: [localhost]: FAILED! => {"changed": false, "module_stderr": "sudo: a password is required\n", ...}
> ```
>
> Dieser Fehler tritt auf, wenn vorab kein `sudo` verwendet wurde und Ansible daher keine Berechtigung hat.  
> Um das zu vermeiden, sollte das Playbook mit dem Parameter `--ask-become-pass` ausgeführt werden:
>
> ```bash
> ansible-playbook playbook.yml --ask-become-pass
> ```
>
> Dadurch wird beim Start des Playbooks nach dem Passwort für `sudo` gefragt.


Jetzt können wir das Playbook ausführen:

```bash
# führe das playbook aus
ansible-playbook -i inventory.ini nginx-setup.yml
```

Um die Änderungen zu sehen, die an den Dateien vorgenommen wurden, können wir den `--diff`-Modus verwenden:

```bash
# führe das playbook im diff-modus aus
ansible-playbook -i inventory.ini nginx-setup.yml --diff
```

**Was passiert bei der Ausführung?**

Wenn du das Playbook ausführst, durchläuft Ansible diese Phasen:

**Phase 1: Vorbereitung**
- Ansible liest das Inventory und identifiziert die Zielserver
- Es überprüft die SSH-Verbindung (oder lokale Verbindung in unserem Fall)
- Es sammelt Facts über diese Server (weil wir `gather_facts: yes` gesetzt haben)

**Phase 2: Ausführung**
- Es führt die pre_tasks aus (unsere Ankündigung)
- Es führt die Aufgaben in der nginx-Rolle aus:
  - APT-Cache aktualisieren
  - Nginx installieren
  - index.html erstellen
  - Nginx konfigurieren
  - Sicherstellen, dass Nginx läuft

**Phase 3: Handler und Abschluss**
- Wenn eine Aufgabe einen Handler benachrichtigt hat, wird dieser ausgeführt
  - In unserem Fall startet der "Restart Nginx" Handler den Nginx-Server neu
- Es führt die post_tasks aus (unsere Erfolgsmeldung)

**Wichtig zu wissen: Idempotenz in Aktion**

Dank der Idempotenz kannst du das Playbook mehrfach ausführen, ohne Schaden anzurichten. Bei wiederholter Ausführung wirst du sehen, dass weniger oder keine Änderungen ("changed") gemeldet werden, da der gewünschte Zustand bereits erreicht ist.

Beispiel:
- Beim ersten Ausführen wird "changed=5" angezeigt (alle Tasks haben etwas geändert)
- Beim zweiten Ausführen könnte "changed=0" angezeigt werden (nichts musste geändert werden)

Der `--diff`-Modus ist besonders hilfreich, um genau zu sehen, welche Änderungen vorgenommen wurden. Dies ist besonders nützlich beim Debugging und beim Lernen, wie Ansible arbeitet.

**Fehlerbehebung: Häufige Probleme**

Wenn du auf Fehler stößt, hier einige häufige Probleme und ihre Lösungen:

1. **Permission denied**: 
   ```bash
   # Setze Berechtigungen für das Webverzeichnis
   sudo chmod -R 755 /var/www/html
   sudo chown -R $(whoami):$(whoami) /var/www/html
   ```
   **Warum tritt das auf?** Der Ansible-Benutzer benötigt Schreibrechte für `/var/www/html`, selbst wenn er `become: yes` verwendet.

2. **Port 80 bereits in Verwendung**:
   ```bash
   # Finde heraus, welcher Prozess Port 80 verwendet
   sudo netstat -tlnp | grep :80
   
   # Stoppe den Prozess (z.B. wenn es ein anderer Webserver ist)
   sudo systemctl stop apache2  # falls es Apache ist
   ```
   **Warum tritt das auf?** Viele Ubuntu-Installationen haben Apache vorinstalliert, der auch Port 80 verwendet.

3. **Nginx startet nicht**:
   ```bash
   # Prüfe die Syntax der Konfiguration
   sudo nginx -t
   
   # Schaue in die Logs
   sudo tail -f /var/log/nginx/error.log
   ```
   **Warum tritt das auf?** Fehler in der Nginx-Konfiguration verhindern den Start des Dienstes.

**Ansible-Kommandozeilenoptionen verstehen:**

Hier einige nützliche Optionen, die du mit `ansible-playbook` verwenden kannst:

- `-v`, `-vv`, `-vvv`: Erhöht die Ausführlichkeit der Ausgabe (hilfreich zum Debuggen)
- `--check`: Führt einen "Trockentest" durch, ohne tatsächliche Änderungen vorzunehmen
- `--tags TAG`: Führt nur Tasks mit bestimmten Tags aus
- `--skip-tags TAG`: Überspringt Tasks mit bestimmten Tags
- `-l HOSTS`: Begrenzt die Ausführung auf bestimmte Hosts

## 6. Teste den Webserver

Nachdem das Playbook erfolgreich ausgeführt wurde, öffne einen Browser und navigiere zu:

```
http://localhost
```

Du solltest nun unsere einfache Webseite sehen, die mit den von uns definierten Variablen und den automatisch gesammelten Ansible-Facts erstellt wurde.

**Was genau passiert beim Aufruf im Browser?**

Wenn du die URL aufrufst:
1. Dein Browser sendet eine HTTP-Anfrage an Port 80 auf deinem lokalen Computer
2. Nginx empfängt diese Anfrage
3. Nginx sucht im konfigurierten Root-Verzeichnis (`/var/www/html`) nach der Datei `index.html`
4. Es findet die Datei, die wir mit unserem Ansible-Template erstellt haben
5. Nginx sendet die Datei als Antwort an deinen Browser
6. Dein Browser rendert den HTML-Code und zeigt die Webseite an

**Was kann ich tun, wenn die Seite nicht angezeigt wird?**

Folge diesem schrittweisen Debuggingprozess:

1. **Schritt 1: Prüfe, ob Nginx läuft**
   ```bash
   sudo systemctl status nginx
   ```
   Suche nach `active (running)` in der Ausgabe. Falls nicht, starte Nginx:
   ```bash
   sudo systemctl start nginx
   ```

2. **Schritt 2: Prüfe, ob Port 80 offen ist**
   ```bash
   sudo netstat -tlnp | grep :80
   ```
   Du solltest eine Zeile sehen, die auf `nginx` verweist. Falls nicht, könnte ein anderer Dienst den Port blockieren.

3. **Schritt 3: Prüfe, ob die Dateien korrekt erstellt wurden**
   ```bash
   # Prüfe, ob die HTML-Datei existiert
   ls -la /var/www/html/
   
   # Zeige den Inhalt der HTML-Datei an
   cat /var/www/html/index.html
   
   # Prüfe die Nginx-Konfiguration
   cat /etc/nginx/sites-available/default
   ```
   
   Falls die Datei nicht existiert oder leer ist, könnte es ein Problem mit den Berechtigungen geben.

4. **Schritt 4: Prüfe die Nginx-Fehlerprotokolle**
   ```bash
   sudo tail -f /var/log/nginx/error.log
   ```
   Suche nach Fehlermeldungen, die auf Probleme hindeuten.

5. **Schritt 5: Validiere die Nginx-Konfiguration**
   ```bash
   sudo nginx -t
   ```
   Dies prüft die Syntax deiner Nginx-Konfiguration.

Die meisten Probleme lassen sich durch diese systematische Herangehensweise identifizieren und beheben.

**Tip für den Browser-Test:** Wenn du mehrere Änderungen vornimmst und die Seite im Browser aktualisierst, verwende Strg+F5 (oder Cmd+Shift+R auf Mac), um den Browser-Cache zu umgehen und sicherzustellen, dass du die neueste Version der Seite siehst.

## 7. Experimentiere mit Variablen

Jetzt, da alles funktioniert, kannst du mit den Variablen experimentieren, um zu sehen, wie Ansible darauf reagiert:

1. **Schritt 1: Ändere die Variablen**
   Öffne die Datei `roles/nginx/defaults/main.yml` und ändere einige Werte:

   ```bash
   # öffne die variablendatei
   nano roles/nginx/defaults/main.yml
   ```

   Ändere die Werte:

   ```yaml
   # Ändere diese Werte
   site_name: "Meine angepasste Ansible-Seite"
   heading_color: "#E63946"
   ```

2. **Schritt 2: Führe das Playbook erneut aus**
   ```bash
   # mit diff-option für bessere sichtbarkeit der änderungen
   ansible-playbook -i inventory.ini nginx-setup.yml --diff
   ```

3. **Schritt 3: Betrachte die Ergebnisse**
   Aktualisiere die Seite im Browser, um die Änderungen zu sehen.

**Was passiert hier genau?**

Dieser Experimentier-Workflow zeigt den wahren Wert von Ansible und Infrastructure as Code:

1. **Bei der Änderung der Variablen**: Du änderst nur einen zentralen Konfigurationswert, nicht den eigentlichen Code oder die Templates.

2. **Bei der erneuten Ausführung**:
   - Ansible prüft den aktuellen Zustand des Systems
   - Es erkennt, dass sich die Variablenwerte geändert haben
   - Es aktualisiert nur die Dateien, die von diesen Variablen abhängen (in diesem Fall die index.html)
   - Der Nginx-Handler wird benachrichtigt, da sich die Datei geändert hat
   - Nginx wird neu gestartet, damit die Änderungen wirksam werden

3. **Im Browser**: Du siehst die aktualisierten Werte, ohne manuell irgendwelche HTML- oder Konfigurationsdateien bearbeiten zu müssen.

**Variablenprioritäten in Ansible verstehen**

Ein wichtiges Konzept in Ansible ist, dass Variablen eine Priorität haben. Die Reihenfolge (von niedrigster zu höchster Priorität) ist etwa:

1. Rollen-Defaults (wie in unserer `defaults/main.yml`)
2. Inventory-Variablen
3. Playbook-Variablen
4. Kommandozeilenargumente

Dies bedeutet, dass du die Werte aus `defaults/main.yml` überschreiben kannst, ohne den Code zu ändern. Zum Beispiel könntest du eine Variable direkt in der Kommandozeile überschreiben:

```bash
# überschreibe eine variable in der kommandozeile
ansible-playbook -i inventory.ini nginx-setup.yml -e "heading_color=#336699"
```

Dieser Ansatz ermöglicht es, die gleiche Rolle in verschiedenen Umgebungen mit unterschiedlichen Werten zu verwenden - ein Grundprinzip von "Infrastructure as Code".

## 8. Bereinigen

Wenn du mit der Übung fertig bist, kannst du Nginx einfach wieder entfernen:

```bash
# erstelle ein cleanup-playbook
nano cleanup.yml
```

Füge folgenden Inhalt hinzu:

```yaml
---
# Einfaches Playbook zum Entfernen von Nginx

- name: Remove Nginx
  hosts: webserver
  become: yes
  
  tasks:
    - name: Stop Nginx service
      service:
        name: nginx
        state: stopped
      
    - name: Remove Nginx package
      apt:
        name: nginx
        state: absent
        
    - name: Remove index.html
      file:
        path: /var/www/html/index.html
        state: absent
```

Führe das Cleanup-Playbook aus:

```bash
ansible-playbook -i inventory.ini cleanup.yml
```

## 9. Zusammenfassung

In dieser Übung hast du:

1. Eine einfache **Ansible-Rolle für Nginx** erstellt
2. **Templates** für HTML und Nginx-Konfiguration verwendet
3. Mit **Variablen und Facts** gearbeitet
4. Einen **Handler** für den Nginx-Neustart implementiert
5. Die Rolle in einem **Playbook** angewendet

Diese Grundlagen sind der Schlüssel für die Arbeit mit Ansible und bilden die Basis für komplexere Automatisierungen in der Zukunft. Du hast gesehen, wie du mit wenigen Zeilen Code einen vollständigen Webserver einrichten kannst - und das auf eine Weise, die wiederholbar, verständlich und anpassbar ist.

**Wo kannst du von hier aus weitermachen?**

- Erweitere die Nginx-Konfiguration um HTTPS-Support
- Füge weitere Variablen hinzu, um die Seite anzupassen
- Erstelle eine Rolle für eine Datenbank und verbinde sie mit deinem Webserver
- Erstelle eine Rolle für eine vollständige Webanwendung

> **Tipp:** Um tiefer in Ansible einzusteigen, schau dir in den [Unterlagen](https://github.com/JacobMenge/lern-unterlagen/blob/main/ansible/ansible-basics.md) besonders die Abschnitte 2.4 "Ansible-Rollen" und 3.2 "Templates mit Jinja2" an.
