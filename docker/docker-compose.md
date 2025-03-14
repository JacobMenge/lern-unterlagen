# 🐳 Der Docker Compose Praxis-Guide: Professionelle Anwendungsentwicklung und Container-Orchestrierung

👋 Willkommen zu meinem Docker Compose Praxis-Guide!

Du bist Entwickler, DevOps-Enthusiast oder einfach neugierig auf moderne Softwareentwicklung? Dann bist du hier genau richtig! Dieses Tutorial ist deine Roadmap durch die Welt der Container-Orchestrierung mit Docker Compose.

## 💡 Lernziele

- Docker Compose vollständig verstehen
- Mehrschichtige Webanwendungen containerisieren
- Sichere und skalierbare Entwicklungsumgebungen aufbauen
- Container-Workflows professionell gestalten

## 📚 Empfohlene Vorkenntnisse

- Grundlagen Docker, schau dir hierzu auch gerne meinen Docker Praxis-Guide an
- JavaScript-Entwicklung
- Grundverständnis von Containern
- Basis-Kenntnisse in Webentwicklung


## Kapitel

### [1. 📘 Grundlagen](#-1-grundlagen)
- [Was ist Docker Compose und wozu dient es?](#-was-ist-docker-compose-und-wozu-dient-es)
- [Unterschied zu einfachem Docker](#-unterschied-zu-einfachem-docker)
- [Systemvoraussetzungen](#-systemvoraussetzungen)

### [2. 📝 Die docker-compose.yml Datei](#-2-die-docker-composeyml-datei)
- [Grundlegende Struktur und Syntax](#-grundlegende-struktur-und-syntax)
- [Kernkonzepte: Services, Networks, Volumes](#-kernkonzepte-services-networks-volumes)
  - [Services (Dienste)](#-services-dienste)
  - [Networks (Netzwerke)](#-networks-netzwerke)
  - [Volumes (Datenvolumen)](#-volumes-datenvolumen)
- [Beispiel einer einfachen Compose-Datei](#-beispiel-einer-einfachen-compose-datei)

### [3. 🖥️ Wichtigste Docker Compose Befehle](#-3-wichtigste-docker-compose-befehle)
- [Starten und Stoppen von Containern](#-starten-und-stoppen-von-containern)
  - [`docker compose up`](#-docker-compose-up)
  - [`docker compose down`](#-docker-compose-down)
- [Überwachung und Diagnose](#-überwachung-und-diagnose)
  - [`docker compose ps`](#-docker-compose-ps)
  - [`docker compose logs`](#-docker-compose-logs)
- [Container-Verwaltung und Interaktion](#-container-verwaltung-und-interaktion)
  - [`docker compose exec`](#-docker-compose-exec)
  - [`docker compose restart`](#-docker-compose-restart)
- [Builds und Images](#-builds-und-images)
  - [`docker compose build`](#-docker-compose-build)
  - [`docker compose pull`](#-docker-compose-pull)
- [Aufräumen und Wartung](#-aufräumen-und-wartung)
  - [`docker compose rm`](#-docker-compose-rm)
  - [`docker system prune`](#-docker-system-prune)

### [4. 🖥️ Praktisches Beispiel: Web-Anwendung mit Datenbank](#-4-praktisches-beispiel-web-anwendung-mit-datenbank)
- [Projektziel und Überblick](#-projektziel-und-überblick)
- [Technische Architektur](#-technische-architektur)
- [Aufbau der Compose-Datei](#-services-definieren)
- [Services definieren](#-services-definieren)
- [Netzwerke und Kommunikation](#-netzwerke-und-kommunikation)
- [Volumes für Datenpersistenz](#-volumes-für-datenpersistenz)

### [5. 🛠️ Wichtige Konfigurationstechniken](#-5-wichtige-konfigurationstechniken)
- [Umgebungsvariablen und .env Dateien](#-umgebungsvariablen-und-env-dateien)
- [Port-Mapping](#-port-mapping)
- [Abhängigkeiten zwischen Services](#-abhängigkeiten-zwischen-services)
- [Restart-Policies](#-restart-policies)

### [6. 🛡️ Best Practices für Docker Compose in JavaScript-Projekten](#-6-best-practices-für-docker-compose-in-javascript-projekten)
- [Sicherheitsaspekte](#-sicherheitsaspekte)
  - [Dependency Sicherheit](#-dependency-sicherheit)
  - [Secrets Management](#-secrets-management)
- [Organisation von Compose-Dateien](#-organisation-von-compose-dateien)
- [Entwicklung vs. Produktion](#-entwicklung-vs-produktion)
  - [Entwicklungs-Konfiguration](#-entwicklungs-konfiguration)
  - [Produktions-Konfiguration](#-produktions-konfiguration)
- [Performance-Optimierungen](#-performance-optimierungen)
- [Monitoring und Logging](#-monitoring-und-logging)

---

## 📘 1. Grundlagen

### 🔍 Was ist Docker Compose und wozu dient es?

Docker Compose ist ein Werkzeug, mit dem du **mehrere Docker-Container gleichzeitig verwalten** kannst. Stell dir vor, deine Anwendung besteht aus einer Webseite, einer Datenbank und vielleicht noch einem Cache-Server. Mit Docker Compose kannst du all diese Teile in einer einzigen Datei beschreiben und mit einem Befehl starten.

Das Besondere an Docker Compose:
-  Du definierst deine gesamte Anwendungsumgebung in einer YAML-Datei (`docker-compose.yml`)
-  Mit nur einem Befehl (`docker-compose up`) startest du alle Container
-  Die Container sind automatisch miteinander vernetzt
-  Deine Konfiguration ist als Code gespeichert und versionierbar

Docker Compose macht es super einfach, komplexe Anwendungen zu starten, ohne dass du dir eine lange Liste von Docker-Befehlen merken musst.

> **💡 Kurz gesagt:** Docker Compose ist ein Tool, das mehrere Docker-Container als eine einzige Anwendung verwaltet und dir viel manuelle Arbeit erspart.

---

### 🔄 Unterschied zu einfachem Docker

| **Merkmal** | **Docker** | **Docker Compose** |
|-------------|------------|-------------------|
| **Verwaltung** | Einzelne Container | Mehrere Container als Stack |
| **Starten** | `docker run` für jeden Container | Ein `docker-compose up` für alle |
| **Netzwerk** | Manuell erstellen & verbinden | Automatisch erstellt |
| **Konfiguration** | Lange Befehlszeilen | Übersichtliche YAML-Datei |
| **Komplexität** | Steigt mit jedem Container | Bleibt überschaubar |

**Docker (allein):**
- Verwaltet einzelne Container
- Startet jeden Container mit einem separaten `docker run`-Befehl
- Du musst manuell Netzwerke erstellen, wenn Container miteinander kommunizieren sollen
- Befehle werden schnell lang und kompliziert:
  ```bash
  docker run -d -p 80:80 --name webserver --network my_network -v ./html:/usr/share/nginx/html nginx
  ```

**Docker Compose:**
- Verwaltet mehrere zusammengehörige Container als einen "Stack"
- Alle Container werden mit einem Befehl gestartet: `docker-compose up`
- Erstellt automatisch ein gemeinsames Netzwerk für deine Container
- Konfiguration ist übersichtlich in einer YAML-Datei zusammengefasst:
  ```yaml
  services:
    webserver:
      image: nginx
      ports:
        - "80:80"
      volumes:
        - ./html:/usr/share/nginx/html
  ```

Das folgende Diagramm zeigt anschaulich, wie Docker Compose funktioniert:

![Docker Compose Überblick](https://github.com/user-attachments/assets/f4812f80-5807-45e8-9a3f-24a1bbe761c0)

*Quelle: https://medium.com/@laurap_85411/docker-compose-stop-vs-down-e4e8d6515a85*

Auf dem Bild siehst du:
- **Links**: Die `docker-compose.yml` Datei (Config YAML), in der du alle deine Dienste, Volumes und andere Einstellungen definierst
- **Mitte**: Docker Compose (mit dem Docker-Oktopus), das deine Konfiguration verarbeitet
- **Rechts**: Was Docker Compose für dich einrichtet:
  - **Images**: Werden aus deinen Definitionen gebaut
  - **Containers**: Werden aus den Images erstellt und ausgeführt
  - **Volumes**: Speichern Daten dauerhaft, auch wenn Container neu gestartet werden
  - **Network**: Verbindet deine Container automatisch miteinander

Mit Docker Compose brauchst du nicht mehr für jeden Container eigene Befehle. Du schreibst einmal die Konfiguration, und Docker Compose kümmert sich um alles andere - das Erstellen oder Laden der Images, das Starten der Container, die Vernetzung und die Datenspeicherung.

> **💡 Kurz gesagt:** Mit Docker allein verwaltest du einzelne Container, mit Docker Compose verwaltest du ganze Anwendungen, die aus mehreren Containern bestehen.

---

## 🛠️ Installation und Überprüfung

### Systemvoraussetzungen

| **Betriebssystem** | **Mindestanforderungen** |
|-------------------|--------------------------|
| **Windows** | Windows 10 64-bit oder neuer, WSL 2 empfohlen |
| **macOS** | macOS 10.15 (Catalina) oder neuer |
| **Linux** | Kernel 3.10 oder neuer, 64-bit |

### 💻 Windows und macOS

Auf Windows und macOS ist Docker Compose automatisch dabei, wenn du Docker Desktop installierst:

1. Lade [Docker Desktop](https://www.docker.com/products/docker-desktop/) herunter
2. Führe die Installation aus
3. Starte Docker Desktop
4. Überprüfe, ob das Symbol in der Taskleiste (Windows) oder Menüleiste (macOS) erscheint

> **💡 Tipp:** Auf Windows brauchst du WSL 2 (Windows Subsystem for Linux) für die beste Performance. Docker Desktop bietet dir während der Installation an, WSL 2 zu aktivieren, falls noch nicht geschehen.

### 🐧 Linux

Auf Linux gibt es mehrere Möglichkeiten:

#### **Option 1: Mit dem Docker Compose Plugin (empfohlen)**

Zuerst musst du Docker Engine installieren:

```bash
# Repository einrichten
sudo apt update
sudo apt install ca-certificates curl gnupg
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg

# Docker Repository hinzufügen (hier für Ubuntu)
echo \
  "deb [arch="$(dpkg --print-architecture)" signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  "$(. /etc/os-release && echo "$VERSION_CODENAME")" stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# Docker Engine installieren
sudo apt update
sudo apt install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

# Docker-Gruppe hinzufügen (damit du Docker ohne sudo nutzen kannst)
sudo usermod -aG docker $USER
# Du musst dich ab- und wieder anmelden, damit diese Änderung wirksam wird
```

Für andere Linux-Distributionen wie Fedora, CentOS oder Arch, folge bitte der [offiziellen Anleitung](https://docs.docker.com/engine/install/).

#### **Option 2: Als separates Paket (ältere Methode)**

```bash
# Lade die neueste Version herunter
sudo curl -L "https://github.com/docker/compose/releases/latest/download/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose

# Mache die Datei ausführbar
sudo chmod +x /usr/local/bin/docker-compose

# Erstelle einen symbolischen Link (optional)
sudo ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose
```

### ✅ Überprüfung der Installation

Ganz gleich, welches System du nutzt, überprüfe deine Installation mit:

```bash
# Für Docker Compose Plugin (neuere Methode)
docker compose version

# Für separate Installation (ältere Methode)
docker-compose version
```

Du solltest eine Ausgabe wie diese sehen:
```
Docker Compose version v2.24.5
```

### 🧪 Test mit einem einfachen Beispiel

So überprüfst du, ob Docker Compose wirklich funktioniert:

1. Erstelle einen neuen Ordner und darin eine Datei namens `docker-compose.yml`:
   ```bash
   mkdir compose-test
   cd compose-test
   ```

2. Füge folgenden Inhalt in die `docker-compose.yml` ein:
   ```yaml
   services:
     hello-world:
       image: hello-world
   ```

3. Starte das Beispiel:
   ```bash
   docker compose up
   ```

4. Du solltest die "Hello World"-Nachricht sehen. Das bedeutet, dass Docker Compose korrekt funktioniert!

### ⚠️ Häufige Probleme bei der Installation

- **Permission denied**: Auf Linux musst du entweder `sudo` verwenden oder deinen Benutzer zur `docker`-Gruppe hinzufügen.
- **Command not found**: Stelle sicher, dass der Installationspfad in deiner PATH-Variable enthalten ist.
- **Falsche Version**: Bei älteren Linux-Distributionen könnte eine veraltete Version in den Repositories sein. Verwende dann die manuelle Installation.

> **💡 Kurz gesagt:** Docker Desktop bringt Docker Compose auf Windows und macOS automatisch mit. Auf Linux installierst du es entweder als Plugin oder als eigenständiges Paket.


---

## 📝 2. Die docker-compose.yml Datei

### 🧱 Grundlegende Struktur und Syntax

Die `docker-compose.yml` Datei ist das Herzstück von Docker Compose. Sie definiert, wie deine gesamte Anwendung aufgebaut ist und funktioniert. Die Datei verwendet das YAML-Format (YAML Ain't Markup Language), das für seine Lesbarkeit bekannt ist.

**Wichtigste Regeln der YAML-Syntax:**
- Einrückung mit Leerzeichen (keine Tabs) bestimmt die Hierarchie
- Listen werden mit Bindestrichen (`-`) gekennzeichnet
- Schlüssel-Wert-Paare werden mit Doppelpunkt getrennt (`key: value`)
- Kommentare beginnen mit einem Rautezeichen (`#`)

Jede `docker-compose.yml` beginnt mit der Versionsangabe und enthält dann die Hauptabschnitte:

```yaml
version: '3.8'  # Die Compose-Dateispezifikation, die du verwendest

services:       # Definiert alle Container deiner Anwendung
  # Hier kommen deine Services hin

volumes:        # Definiert dauerhafte Datenspeicher (optional)
  # Hier kommen deine Volumes hin

networks:       # Definiert Netzwerke für die Kommunikation (optional)
  # Hier kommen deine Netzwerke hin
```

> **💡 Tipp:** Ab Docker Compose v2 ist die `version`-Zeile optional. Wenn du sie weglässt, wird die neueste unterstützte Version verwendet.

### 🔍 Kernkonzepte: Services, Networks, Volumes

#### **Services (Dienste)**

Services sind die Container, aus denen deine Anwendung besteht. Jeder Service wird durch einen Namen identifiziert und kann verschiedene Konfigurationsoptionen haben:

| **Option** | **Beschreibung** | **Beispiel** |
|------------|------------------|--------------|
| `image` | Das Docker-Image, das verwendet werden soll | `image: nginx:latest` |
| `build` | Pfad zum Dockerfile, um ein eigenes Image zu bauen | `build: ./app` |
| `ports` | Port-Mapping zwischen Host und Container | `ports: - "8080:80"` |
| `volumes` | Verknüpft Volumes oder Host-Verzeichnisse mit dem Container | `volumes: - ./data:/app/data` |
| `environment` | Umgebungsvariablen für den Container | `environment: - DB_HOST=db` |
| `depends_on` | Abhängigkeiten zwischen Services | `depends_on: - db` |
| `restart` | Neustart-Richtlinie | `restart: always` |
| `networks` | Netzwerke, mit denen der Service verbunden ist | `networks: - backend` |

#### **Networks (Netzwerke)**

Netzwerke ermöglichen die Kommunikation zwischen deinen Containern. Docker Compose erstellt automatisch ein Standard-Netzwerk, aber du kannst auch eigene definieren:

```yaml
networks:
  frontend:  # Netzwerk für Frontend-Services
    driver: bridge
  backend:   # Netzwerk für Backend-Services
    driver: bridge
```

Die wichtigsten Netzwerk-Treiber sind:
- `bridge`: Standard-Treiber für Container-Kommunikation auf demselben Host
- `host`: Verwendet direkt das Netzwerk des Host-Systems
- `overlay`: Für Container-Kommunikation über mehrere Docker-Hosts hinweg (in Swarm-Umgebungen)
- `none`: Deaktiviert die Netzwerkfunktionalität

#### **Volumes (Datenvolumen)**

Volumes sind der bevorzugte Mechanismus für dauerhafte Datenspeicherung in Docker. Sie bleiben bestehen, auch wenn Container gestoppt oder gelöscht werden:

```yaml
volumes:
  db-data:     # Name des Volumes
    driver: local
  cache-data:  # Ein weiteres Volume
    driver: local
```

Arten von Volumes in Docker Compose:
- **Benannte Volumes**: Werden im `volumes`-Abschnitt definiert und bleiben bestehen
- **Host-Volumes**: Verknüpfen ein Host-Verzeichnis mit einem Container-Pfad
- **Anonyme Volumes**: Werden nur im Service definiert und bei `docker-compose down -v` entfernt

### 📋 Beispiel einer einfachen Compose-Datei

Hier ist ein vollständiges Beispiel für eine moderne JavaScript-Webanwendung mit Node.js, Express und MongoDB:

```yaml
version: '3.8'

services:
  # Frontend mit React
  frontend:
    build: ./frontend  # Annahme: Enthält Dockerfile für React-App
    ports:
      - "3000:3000"
    volumes:
      - ./frontend:/app
      - /app/node_modules
    environment:
      - REACT_APP_API_URL=http://localhost:4000
    depends_on:
      - backend
    networks:
      - frontend-network

  # Backend mit Node.js und Express
  backend:
    build: ./backend  # Annahme: Enthält Dockerfile für Node.js/Express-App
    ports:
      - "4000:4000"
    volumes:
      - ./backend:/app
      - /app/node_modules
    environment:
      - MONGODB_URI=mongodb://db:27017/myapp
      - PORT=4000
      - NODE_ENV=development
    depends_on:
      - db
    networks:
      - frontend-network
      - backend-network
    command: npm run dev  # Startet den Server im Entwicklungsmodus

  # MongoDB Datenbank
  db:
    image: mongo:latest
    ports:
      - "27017:27017"
    volumes:
      - mongo-data:/data/db
    networks:
      - backend-network
    restart: always

networks:
  frontend-network:  # Für die Kommunikation zwischen Frontend und Backend
  backend-network:   # Für die Kommunikation zwischen Backend und Datenbank

volumes:
  mongo-data:   # Persistente Speicherung für Datenbankdaten
```

Dieses Beispiel definiert:
- Drei Services: `frontend` (React-Anwendung), `backend` (Node.js/Express-API) und `db` (MongoDB-Datenbank)
- Zwei Netzwerke: `frontend-network` und `backend-network`
- Ein Volume: `mongo-data` für dauerhafte Speicherung der Datenbankdaten

Das Diagramm zeigt, wie die Services interagieren:

```
     [Browser]
        |
        | Port 3000         Port 4000
        ↓                   ↓
    +----------+        +-----------+
    | frontend |        |  backend  |
    +----------+        +-----------+
        |                    |
        +-- frontend-network-+
                             |
                             +-- backend-network --+
                                                   |
                                               +-------+
                                               |  db   |
                                               +-------+
                                                   |
                                             +------------+
                                             | mongo-data |
                                             +------------+
```

So funktioniert es:
1. Der Browser kann auf den `web`-Service über Port 80 und auf `phpmyadmin` über Port 8080 zugreifen
2. Beide Services können mit der Datenbank über das `backend`-Netzwerk kommunizieren
3. Die Datenbankdaten werden dauerhaft im `db-data`-Volume gespeichert

### 🔄 So starten und stoppen wir die Anwendung

Mit diesen einfachen Befehlen verwaltest du den gesamten Stack:

```bash
# Starten im Hintergrund
docker compose up -d

# Status überprüfen
docker compose ps

# Logs anzeigen
docker compose logs

# Alles stoppen, aber Volumes behalten
docker compose down

# Alles stoppen und Volumes löschen
docker compose down -v
```

> **💡 Zusammenfassung:** Die `docker-compose.yml` Datei ist deine Anwendungsblaupause: Sie definiert Services (Container), Netzwerke für die Kommunikation und Volumes für die Datenspeicherung. Mit ihr kannst du komplexe Anwendungen mit einem einzigen Befehl starten und stoppen.

---

## 🖥️ 3. Wichtigste Docker Compose Befehle

Docker Compose bietet zahlreiche Befehle für die Verwaltung deiner Container. Hier lernst du die wichtigsten Befehle kennen, die du täglich brauchen wirst.

### 🚀 Starten und Stoppen von Containern

#### **`docker compose up`**

Mit diesem Befehl startest du alle in deiner `docker-compose.yml` definierten Services:

```bash
# Standard-Start (im Vordergrund mit Logs)
docker compose up

# Im Hintergrund starten (detached mode)
docker compose up -d

# Nur bestimmte Services starten
docker compose up frontend db

# Services neu erstellen (auch wenn sie bereits existieren)
docker compose up --build

# Skalieren einzelner Services
docker compose up --scale web=3 --scale worker=2
```

> **💡 Tipp:** Im detached Mode (`-d`) läuft Docker Compose im Hintergrund und du behältst die Kontrolle über dein Terminal.

#### **`docker compose down`**

Dieser Befehl stoppt und entfernt alle Container, Netzwerke und standardmäßig auch die anonymen Volumes:

```bash
# Standard-Stopp (behält benannte Volumes)
docker compose down

# Stoppt alles und entfernt auch alle Volumes
docker compose down -v

# Stoppt alles und entfernt auch alle Images
docker compose down --rmi all

# Stoppt alles, aber behält Container (nur für Netzwerke)
docker compose down --remove-orphans
```

> **⚠️ Vorsicht:** `docker compose down -v` löscht alle Volumes - deine persistenten Daten gehen dabei verloren!

### 📊 Überwachung und Diagnose

#### **`docker compose ps`**

Zeigt den Status aller Container an, die in der `docker-compose.yml` definiert sind:

```bash
# Alle Services anzeigen
docker compose ps

# Nur laufende Services anzeigen
docker compose ps --services --filter "status=running"

# Status in kurzer Form
docker compose ps -a
```

Die Ausgabe enthält wichtige Informationen wie:
- Service-Name
- Container-ID
- Status (running, exited, etc.)
- Ports
- Befehle, die beim Start ausgeführt wurden

#### **`docker compose logs`**

Mit diesem Befehl greifst du auf die Logs aller Container zu:

```bash
# Logs aller Services anzeigen
docker compose logs

# Logs im "follow"-Modus (wie tail -f)
docker compose logs -f

# Logs eines bestimmten Services
docker compose logs backend

# Letzte 10 Log-Zeilen anzeigen
docker compose logs --tail=10

# Logs mit Zeitstempel
docker compose logs --timestamps
```

> **💡 Tipp:** Mit `docker compose logs -f service_name` kannst du die Logs eines bestimmten Services in Echtzeit verfolgen - perfekt für das Debugging!

### 🔄 Container-Verwaltung und Interaktion

#### **`docker compose exec`**

Mit diesem Befehl führst du Befehle innerhalb eines laufenden Containers aus:

```bash
# Shell in einem Container öffnen
docker compose exec web bash

# Einmaligen Befehl ausführen
docker compose exec db psql -U postgres

# Als bestimmter Benutzer ausführen
docker compose exec --user www-data web php artisan migrate

# Umgebungsvariable setzen
docker compose exec -e DEBUG=true web npm test
```

Anwendungsbeispiele:

| **Use Case** | **Befehl** |
|--------------|------------|
| Datenbank-CLI öffnen | `docker compose exec db mysql -u root -p` |
| Django-Migrationen ausführen | `docker compose exec web python manage.py migrate` |
| Laravel-Artisan nutzen | `docker compose exec app php artisan make:model User` |
| Node.js-Tests starten | `docker compose exec frontend npm test` |

#### **`docker compose restart`**

Startet Container neu, ohne sie neu zu erstellen:

```bash
# Alle Services neustarten
docker compose restart

# Spezifischen Service neustarten
docker compose restart web

# Mit Timeout (warten vor hartem Kill)
docker compose restart --timeout 30 web
```

### 🔨 Builds und Images

#### **`docker compose build`**

Baut (oder rebuilt) die Images für Services mit einem definierten `build`-Parameter:

```bash
# Alle Services bauen
docker compose build

# Spezifischen Service bauen
docker compose build backend

# Images ohne Cache bauen (komplett neu)
docker compose build --no-cache

# Mit Build-Args (für das Dockerfile)
docker compose build --build-arg VERSION=1.0 web
```

Wann du `build` verwenden solltest:
- Nach Änderungen am Code deiner Anwendung
- Nach Änderungen am Dockerfile
- Wenn du ein Update der Base-Images erzwingen möchtest
- Bei CI/CD-Pipelines für frische Builds

#### **`docker compose pull`**

Lädt die neuesten Versionen der Images für deine Services herunter:

```bash
# Alle Images aktualisieren
docker compose pull

# Image für bestimmten Service aktualisieren
docker compose pull redis

# Ohne Ausgabe (für Scripts)
docker compose pull --quiet
```

> **💡 Tipp:** Kombiniere `pull` und `up`: `docker compose pull && docker compose up -d` für einen Update-Workflow.

### 📂 Daten und Volumes

#### **`docker compose cp`**

Kopiert Dateien zwischen dem Host und den Containern:

```bash
# Datei in Container kopieren
docker compose cp ./local-file.txt web:/app/

# Datei aus Container kopieren
docker compose cp db:/var/lib/mysql/data.sql ./backup.sql

# Verzeichnis kopieren
docker compose cp ./assets/ frontend:/usr/share/nginx/html/assets/
```

#### **`docker compose config`**

Überprüft und zeigt die effektive Compose-Konfiguration:

```bash
# Validieren und anzeigen
docker compose config

# Nur validieren, ohne Ausgabe
docker compose config --quiet

# In JSON-Format konvertieren
docker compose config --format json

# Services auflisten
docker compose config --services
```

### 🧹 Aufräumen und Wartung

#### **`docker compose rm`**

Entfernt gestoppte Container:

```bash
# Alle gestoppten Container entfernen
docker compose rm

# Ohne Bestätigung
docker compose rm -f

# Auch Volumes entfernen
docker compose rm -v

# Bestimmten Service entfernen
docker compose rm worker
```

#### **`docker system prune`**

Ein nützlicher Docker-Befehl (nicht spezifisch für Compose), der hilft, Speicherplatz freizugeben:

```bash
# Alle nicht verwendeten Ressourcen entfernen
docker system prune

# Auch nicht verwendete Volumes entfernen
docker system prune --volumes

# Alles löschen, auch benutzte Images (vorsichtig!)
docker system prune -a
```

### 📋 Cheat Sheet der wichtigsten Befehle

| **Befehl** | **Beschreibung** | **Häufige Optionen** |
|------------|------------------|-----------------------|
| `docker compose up` | Startet Services | `-d` (detached), `--build`, `--no-deps` |
| `docker compose down` | Stoppt und entfernt Services | `-v` (Volumes löschen), `--rmi all` |
| `docker compose ps` | Zeigt Status der Container | `-a` (alle), `--services` |
| `docker compose logs` | Zeigt Logs | `-f` (follow), `--tail=n`, `--timestamps` |
| `docker compose exec` | Führt Befehle im Container aus | `--user`, `-e` (env vars) |
| `docker compose build` | Baut Images | `--no-cache`, `--pull` |
| `docker compose restart` | Startet Container neu | `--timeout` |
| `docker compose pull` | Aktualisiert Images | `--quiet`, `--ignore-pull-failures` |
| `docker compose rm` | Entfernt gestoppte Container | `-f` (force), `-v` (mit Volumes) |
| `docker compose config` | Validiert und zeigt Konfiguration | `--services`, `--volumes` |

### 🌐 Häufige Workflows

#### **Entwicklungs-Workflow**

```bash
# Start mit Debug-Logs
docker compose up

# Code ändern... dann:
docker compose restart web

# Debugging: In Container einloggen
docker compose exec web bash

# Wenn Dockerfile geändert wurde:
docker compose up --build
```

#### **Produktions-Workflow**

```bash
# Neueste Images holen
docker compose pull

# Services im Hintergrund starten
docker compose up -d

# Status überprüfen
docker compose ps

# Logs für Fehlersuche
docker compose logs -f web

# Update einzelner Services
docker compose up -d --no-deps --build web
```

#### **Aufräum-Workflow**

```bash
# Nicht benötigte Container entfernen
docker compose rm

# Volumes sichern (wenn nötig)
docker compose exec db pg_dump -U postgres > backup.sql

# Alles herunterfahren
docker compose down

# System aufräumen (außerhalb von Compose)
docker system prune
```

> **💡 Zusammenfassung:** Mit diesen Compose-Befehlen hast du die volle Kontrolle über den Lebenszyklus deiner Container. Von der Erstellung über die Überwachung bis hin zur Fehlersuche - Docker Compose macht die Arbeit mit Container-Stacks einfach und effizient.

---

## 🖥️ 4. Praktisches Beispiel: Web-Anwendung mit Datenbank

In diesem Kapitel werden wir ein praktisches Beispiel durcharbeiten: Eine moderne Web-Anwendung mit React, Node.js, Express und MongoDB. Dieses Beispiel zeigt dir, wie du Docker Compose im Alltag einsetzen kannst, um eine mehrteilige JavaScript-Anwendung zu verwalten.

### 📑 Projektziel und Überblick

Wir erstellen eine einfache To-Do-Anwendung mit:
- **Frontend**: Eine React-Anwendung mit modernem UI
- **Backend**: Eine Node.js/Express API
- **Datenbank**: MongoDB für die Datenspeicherung

## 🌟 Hauptfunktionen

- Aufgaben erstellen
- Aufgaben anzeigen
- Aufgaben als erledigt markieren
- Aufgaben löschen

## 🔧 Technische Architektur

Die Anwendung nutzt drei Hauptkomponenten:
1. **Benutzeroberfläche**: Interaktive React-Anwendung
2. **Backend-Service**: API zur Verarbeitung von Aufgaben
3. **Datenbank**: Persistente Speicherung der Aufgaben

```
+---------------------+     +---------------------+     +---------------------+
| Web-Browser         |     | React Frontend      |     | Node.js Backend     |
|                     | ←→  |                     |  ←→ |                     |
| HTTP-Requests       |     | API-Kommunikation   |     |Datenbank-Operationen|
| (Senden/Empfangen)  |     | (Anfragen/Antworten)|     |(Lesen/Schreiben)    |
+---------------------+     +---------------------+     +----------+----------+
                                                                  ↓ ↑
                                                        +---------------------+
                                                        |  MongoDB            |
                                                        |  Datenbank          |
                                                        +---------------------+
```

So wird unsere Anwendungsstruktur aussehen:

```
todo-app/
├── docker-compose.yml     # Unsere Hauptkonfigurationsdatei
├── .env                   # Umgebungsvariablen für Compose
├── frontend/              # React-Frontend
│   ├── Dockerfile         # Frontend-Build
│   ├── package.json
│   ├── public/
│   └── src/
│       ├── components/
│       ├── App.js
│       └── index.js
└── backend/               # Node.js/Express-Backend
    ├── Dockerfile         # Backend-Build
    ├── package.json
    ├── index.js
    ├── routes/
    └── models/
```

### 🏗️ Aufbau der Compose-Datei

Beginnen wir mit der `docker-compose.yml` Datei, die das Herzstück unseres Projekts ist:

```yaml
version: '3.8'

services:
  # React Frontend
  frontend:
    build:
      context: ./frontend
    container_name: todo_frontend
    ports:
      - "3000:3000"
    volumes:
      - ./frontend:/app
      - /app/node_modules
    depends_on:
      - backend
    environment:
      - REACT_APP_API_URL=http://localhost:4000
    networks:
      - frontend-network
    stdin_open: true  # Für die React-Entwicklungsumgebung

  # Node.js/Express Backend
  backend:
    build:
      context: ./backend
    container_name: todo_backend
    ports:
      - "4000:4000"
    volumes:
      - ./backend:/app
      - /app/node_modules
    depends_on:
      - db
    environment:
      - MONGODB_URI=mongodb://db:27017/tododb
      - PORT=4000
      - NODE_ENV=development
    networks:
      - frontend-network
      - backend-network
    command: npm run dev  # Verwendet nodemon für Entwicklung

  # MongoDB Datenbank
  db:
    image: mongo:latest
    container_name: todo_db
    restart: always
    volumes:
      - db_data:/data/db
    ports:
      - "27017:27017"
    networks:
      - backend-network

networks:
  frontend-network:
    driver: bridge
  backend-network:
    driver: bridge

volumes:
  db_data:
```

### 🧩 Services definieren

Schauen wir uns jeden Service im Detail an:

#### **React Frontend**

```yaml
frontend:
  build:
    context: ./frontend
  container_name: todo_frontend
  ports:
    - "3000:3000"
  volumes:
    - ./frontend:/app
    - /app/node_modules
  depends_on:
    - backend
  environment:
    - REACT_APP_API_URL=http://localhost:4000
  networks:
    - frontend-network
  stdin_open: true  # Für die React-Entwicklungsumgebung
```

- **`build: context: ./frontend`**: Baut das Frontend aus dem lokalen `frontend`-Verzeichnis
- **`container_name: todo_frontend`**: Gibt dem Container einen benutzerfreundlichen Namen
- **`ports: - "3000:3000"`**: Leitet den React-Entwicklungsserver-Port weiter
- **`volumes:`**: 
  - **`./frontend:/app`**: Verbindet den lokalen Code mit dem Container für Hot-Reloading
  - **`/app/node_modules`**: Erstellt ein anonymes Volume für die `node_modules`, um Konflikte zu vermeiden
- **`depends_on: - backend`**: Stellt sicher, dass der Backend-Container vor dem Frontend-Container gestartet wird
- **`environment:`**: Setzt die API-URL als Umgebungsvariable für das Frontend
- **`stdin_open: true`**: Hält STDIN offen, notwendig für die React-Entwicklungsumgebung

#### **Node.js/Express Backend**

```yaml
backend:
  build:
    context: ./backend
  container_name: todo_backend
  ports:
    - "4000:4000"
  volumes:
    - ./backend:/app
    - /app/node_modules
  depends_on:
    - db
  environment:
    - MONGODB_URI=mongodb://db:27017/tododb
    - PORT=4000
    - NODE_ENV=development
  networks:
    - frontend-network
    - backend-network
  command: npm run dev  # Verwendet nodemon für Entwicklung
```

- **`build: context: ./backend`**: Baut das Backend aus dem lokalen `backend`-Verzeichnis
- **`ports: - "4000:4000"`**: Leitet den Express-Server-Port weiter
- **`volumes:`**: 
  - **`./backend:/app`**: Verbindet den lokalen Code mit dem Container
  - **`/app/node_modules`**: Erstellt ein anonymes Volume für die `node_modules`
- **`depends_on: - db`**: Stellt sicher, dass die Datenbank vor dem Backend gestartet wird
- **`environment:`**: Setzt Umgebungsvariablen für die Verbindung zur Datenbank
- **`networks:`**: Verbindet den Container mit beiden Netzwerken
- **`command: npm run dev`**: Startet das Backend mit nodemon für automatisches Neuladen bei Codeänderungen

#### **MongoDB-Datenbank**

```yaml
db:
  image: mongo:latest
  container_name: todo_db
  restart: always
  volumes:
    - db_data:/data/db
  ports:
    - "27017:27017"
  networks:
    - backend-network
```

- **`image: mongo:latest`**: Verwendet das offizielle MongoDB-Image
- **`restart: always`**: Stellt sicher, dass die Datenbank nach einem Absturz automatisch neu gestartet wird
- **`volumes: - db_data:/data/db`**: Speichert Datenbank-Dateien in einem benannten Volume
- **`ports: - "27017:27017"`**: Ermöglicht den Zugriff auf die Datenbank von außerhalb
- **`networks: - backend-network`**: Verbindet nur mit dem Backend-Netzwerk

### 🔄 Netzwerke und Kommunikation zwischen Containern

In unserem Beispiel haben wir zwei Netzwerke definiert:

```yaml
networks:
  frontend-network:
    driver: bridge
  backend-network:
    driver: bridge
```

Diese Netzwerke haben folgende Funktionen:

1. **`frontend-network`**:
   - Verbindet das React-Frontend mit dem Node.js-Backend
   - Sowohl der `frontend`- als auch der `backend`-Service sind diesem Netzwerk zugeordnet

2. **`backend-network`**:
   - Ein internes Netzwerk für die Kommunikation zwischen Backend und Datenbank
   - Sowohl der `backend`- als auch der `db`-Service sind diesem Netzwerk zugeordnet
   - Dieses Netzwerk bietet zusätzliche Sicherheit, da die Datenbank nicht direkt vom Frontend erreichbar ist

Die Kommunikation zwischen Containern erfolgt über ihre Service-Namen. So kann der `backend`-Container die Datenbank über den Hostnamen `db` erreichen. Das ist auch in den Umgebungsvariablen des Backend-Containers zu sehen:

```yaml
environment:
  - MONGODB_URI=mongodb://db:27017/tododb  # Verwendet den Service-Namen als Hostnamen
```

Dieses Diagramm veranschaulicht die Netzwerkarchitektur:

```
[Benutzer Browser] <---> [Port 3000] <---> frontend <---> backend <---> db
                             |                |            |
                    frontend-network    frontend-network   |
                                            |              |
                                       backend-network ----+
```

### 📦 Volumes für Datenpersistenz

In unserem Beispiel verwenden wir drei Arten von Volumes:

1. **Benanntes Volume** für Datenbankdaten:
   ```yaml
   volumes:
     db_data:  # Definiert ein benanntes Volume
   ```
   
   Dieses Volume wird im Service verwendet:
   ```yaml
   volumes:
      - db_data:/data/db  # Speichert MongoDB-Daten persistent
   ```

2. **Host-Volumes** (Bind Mounts) für den Code:
   ```yaml
   volumes:
      - ./frontend:/app  # Verbindet lokalen Frontend-Code mit Container
      - ./backend:/app   # Verbindet lokalen Backend-Code mit Container
   ```

3. **Anonyme Volumes** für `node_modules`:
   ```yaml
   volumes:
      - /app/node_modules  # Verhindert Überschreiben der installierten Module
   ```

Die Vorteile dieser Volume-Konfiguration:

- **Datenpersistenz**: Selbst wenn der Datenbank-Container gelöscht wird, bleiben deine Daten erhalten
- **Entwicklungsfreundlichkeit**: Änderungen am Code werden sofort wirksam ohne den Container neu zu starten
- **Performanz**: Die `node_modules` verbleiben im Container, was Leistungsprobleme auf Windows/macOS vermeidet

### 📄 Dateien für das Projekt

Um das Beispiel vollständig zu machen, schauen wir uns die wichtigsten Dateien an:

#### **backend/Dockerfile**

```dockerfile
FROM node:18-alpine

WORKDIR /app

COPY package*.json ./

RUN npm install

COPY . .

EXPOSE 4000

CMD ["npm", "start"]
```

#### **frontend/Dockerfile**

```dockerfile
FROM node:18-alpine

WORKDIR /app

COPY package*.json ./

RUN npm install

COPY . .

EXPOSE 3000

CMD ["npm", "start"]
```

#### **backend/package.json**

```json
{
  "name": "todo-backend",
  "version": "1.0.0",
  "description": "Todo App Backend",
  "main": "index.js",
  "scripts": {
    "start": "node index.js",
    "dev": "nodemon index.js"
  },
  "dependencies": {
    "cors": "^2.8.5",
    "express": "^4.18.2",
    "mongoose": "^7.5.0",
    "morgan": "^1.10.0"
  },
  "devDependencies": {
    "nodemon": "^3.0.1"
  }
}
```

#### **backend/index.js**

```javascript
const express = require('express');
const mongoose = require('mongoose');
const cors = require('cors');
const morgan = require('morgan');
const taskRoutes = require('./routes/tasks');

const app = express();
const PORT = process.env.PORT || 4000;
const MONGODB_URI = process.env.MONGODB_URI || 'mongodb://localhost:27017/tododb';

// Middleware
app.use(cors());
app.use(morgan('dev'));
app.use(express.json());

// Connect to MongoDB
mongoose.connect(MONGODB_URI, {
  useNewUrlParser: true,
  useUnifiedTopology: true,
})
.then(() => console.log('MongoDB connected'))
.catch(err => console.error('MongoDB connection error:', err));

// Routes
app.use('/api/tasks', taskRoutes);

// Health check route
app.get('/health', (req, res) => {
  res.status(200).json({ status: 'ok' });
});

// Start server
app.listen(PORT, () => {
  console.log(`Server running on port ${PORT}`);
});
```

#### **backend/models/Task.js**

```javascript
const mongoose = require('mongoose');

const taskSchema = new mongoose.Schema({
  title: {
    type: String,
    required: true,
    trim: true,
  },
  description: {
    type: String,
    trim: true,
  },
  completed: {
    type: Boolean,
    default: false,
  },
}, {
  timestamps: true, // Adds createdAt and updatedAt
});

module.exports = mongoose.model('Task', taskSchema);
```

#### **backend/routes/tasks.js**

```javascript
const express = require('express');
const router = express.Router();
const Task = require('../models/Task');

// Get all tasks
router.get('/', async (req, res) => {
  try {
    const tasks = await Task.find().sort({ createdAt: -1 });
    res.json(tasks);
  } catch (err) {
    res.status(500).json({ message: err.message });
  }
});

// Create a new task
router.post('/', async (req, res) => {
  const task = new Task({
    title: req.body.title,
    description: req.body.description,
  });

  try {
    const newTask = await task.save();
    res.status(201).json(newTask);
  } catch (err) {
    res.status(400).json({ message: err.message });
  }
});

// Get a single task
router.get('/:id', async (req, res) => {
  try {
    const task = await Task.findById(req.params.id);
    if (!task) return res.status(404).json({ message: 'Task not found' });
    res.json(task);
  } catch (err) {
    res.status(500).json({ message: err.message });
  }
});

// Update a task
router.patch('/:id', async (req, res) => {
  try {
    const task = await Task.findByIdAndUpdate(
      req.params.id, 
      req.body, 
      { new: true }
    );
    if (!task) return res.status(404).json({ message: 'Task not found' });
    res.json(task);
  } catch (err) {
    res.status(400).json({ message: err.message });
  }
});

// Delete a task
router.delete('/:id', async (req, res) => {
  try {
    const task = await Task.findByIdAndDelete(req.params.id);
    if (!task) return res.status(404).json({ message: 'Task not found' });
    res.json({ message: 'Task deleted' });
  } catch (err) {
    res.status(500).json({ message: err.message });
  }
});

module.exports = router;
```

#### **frontend/package.json**

```json
{
  "name": "todo-frontend",
  "version": "0.1.0",
  "private": true,
  "dependencies": {
    "axios": "^1.5.0",
    "react": "^18.2.0",
    "react-dom": "^18.2.0",
    "react-scripts": "5.0.1"
  },
  "scripts": {
    "start": "react-scripts start",
    "build": "react-scripts build",
    "test": "react-scripts test",
    "eject": "react-scripts eject"
  },
  "eslintConfig": {
    "extends": [
      "react-app",
      "react-app/jest"
    ]
  },
  "browserslist": {
    "production": [
      ">0.2%",
      "not dead",
      "not op_mini all"
    ],
    "development": [
      "last 1 chrome version",
      "last 1 firefox version",
      "last 1 safari version"
    ]
  }
}
```

#### **frontend/src/App.js**

```javascript
import React, { useState, useEffect } from 'react';
import axios from 'axios';
import TaskForm from './components/TaskForm';
import TaskList from './components/TaskList';
import './App.css';

const API_URL = process.env.REACT_APP_API_URL || 'http://localhost:4000';

function App() {
  const [tasks, setTasks] = useState([]);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);

  // Fetch tasks on component mount
  useEffect(() => {
    fetchTasks();
  }, []);

  const fetchTasks = async () => {
    try {
      setLoading(true);
      const response = await axios.get(`${API_URL}/api/tasks`);
      setTasks(response.data);
      setError(null);
    } catch (err) {
      console.error('Error fetching tasks:', err);
      setError('Failed to fetch tasks. Please try again later.');
    } finally {
      setLoading(false);
    }
  };

  const addTask = async (taskData) => {
    try {
      const response = await axios.post(`${API_URL}/api/tasks`, taskData);
      setTasks([response.data, ...tasks]);
    } catch (err) {
      console.error('Error adding task:', err);
      setError('Failed to add task. Please try again.');
    }
  };

  const toggleTaskCompletion = async (id, completed) => {
    try {
      const response = await axios.patch(`${API_URL}/api/tasks/${id}`, {
        completed: !completed,
      });
      
      setTasks(tasks.map(task => 
        task._id === id ? { ...task, completed: !completed } : task
      ));
    } catch (err) {
      console.error('Error updating task:', err);
      setError('Failed to update task. Please try again.');
    }
  };

  const deleteTask = async (id) => {
    try {
      await axios.delete(`${API_URL}/api/tasks/${id}`);
      setTasks(tasks.filter(task => task._id !== id));
    } catch (err) {
      console.error('Error deleting task:', err);
      setError('Failed to delete task. Please try again.');
    }
  };

  return (
    <div className="app-container">
      <header>
        <h1>Todo App</h1>
      </header>
      
      <main>
        <TaskForm onAddTask={addTask} />
        
        {error && <div className="error-message">{error}</div>}
        
        {loading ? (
          <div className="loading">Loading tasks...</div>
        ) : (
          <TaskList 
            tasks={tasks} 
            onToggleComplete={toggleTaskCompletion} 
            onDeleteTask={deleteTask} 
          />
        )}
      </main>

      <footer>
        <p>Docker Compose Todo App Example</p>
      </footer>
    </div>
  );
}

export default App;
```

#### **frontend/src/components/TaskForm.js**

```javascript
import React, { useState } from 'react';

function TaskForm({ onAddTask }) {
  const [title, setTitle] = useState('');
  const [description, setDescription] = useState('');

  const handleSubmit = (e) => {
    e.preventDefault();
    
    if (!title.trim()) return;
    
    onAddTask({
      title,
      description
    });
    
    // Reset form
    setTitle('');
    setDescription('');
  };

  return (
    <div className="task-form">
      <h2>Add New Task</h2>
      <form onSubmit={handleSubmit}>
        <div className="form-group">
          <label htmlFor="title">Title</label>
          <input
            type="text"
            id="title"
            value={title}
            onChange={(e) => setTitle(e.target.value)}
            placeholder="Enter task title"
            required
          />
        </div>
        
        <div className="form-group">
          <label htmlFor="description">Description</label>
          <textarea
            id="description"
            value={description}
            onChange={(e) => setDescription(e.target.value)}
            placeholder="Enter task description (optional)"
            rows="3"
          />
        </div>
        
        <button type="submit" className="btn-submit">
          Add Task
        </button>
      </form>
    </div>
  );
}

export default TaskForm;
```

#### **frontend/src/components/TaskList.js**

```javascript
import React from 'react';
import TaskItem from './TaskItem';

function TaskList({ tasks, onToggleComplete, onDeleteTask }) {
  if (tasks.length === 0) {
    return (
      <div className="empty-list">
        <p>No tasks yet. Add a new task to get started!</p>
      </div>
    );
  }

  return (
    <div className="task-list">
      <h2>My Tasks</h2>
      <ul>
        {tasks.map(task => (
          <TaskItem
            key={task._id}
            task={task}
            onToggleComplete={onToggleComplete}
            onDeleteTask={onDeleteTask}
          />
        ))}
      </ul>
    </div>
  );
}

export default TaskList;
```

#### **frontend/src/components/TaskItem.js**

```javascript
import React from 'react';

function TaskItem({ task, onToggleComplete, onDeleteTask }) {
  return (
    <li className={`task-item ${task.completed ? 'completed' : ''}`}>
      <div className="task-content">
        <div className="task-header">
          <h3>{task.title}</h3>
          <div className="task-actions">
            <button 
              className="btn-toggle" 
              onClick={() => onToggleComplete(task._id, task.completed)}
            >
              {task.completed ? 'Undo' : 'Complete'}
            </button>
            <button 
              className="btn-delete" 
              onClick={() => onDeleteTask(task._id)}
            >
              Delete
            </button>
          </div>
        </div>
        {task.description && (
          <p className="task-description">{task.description}</p>
        )}
        <div className="task-meta">
          <small>Created: {new Date(task.createdAt).toLocaleString()}</small>
        </div>
      </div>
    </li>
  );
}

export default TaskItem;
```

#### **frontend/src/App.css**

```css
* {
  box-sizing: border-box;
  margin: 0;
  padding: 0;
}

body {
  font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
  line-height: 1.6;
  color: #333;
  background-color: #f5f5f5;
}

.app-container {
  max-width: 800px;
  margin: 0 auto;
  padding: 20px;
}

header {
  text-align: center;
  margin-bottom: 40px;
}

header h1 {
  color: #2c3e50;
  font-size: 2.5rem;
}

main {
  background: white;
  border-radius: 8px;
  box-shadow: 0 2px 10px rgba(0, 0, 0, 0.1);
  padding: 20px;
}

.task-form {
  margin-bottom: 30px;
  border-bottom: 1px solid #eee;
  padding-bottom: 20px;
}

.task-form h2 {
  margin-bottom: 15px;
  color: #2c3e50;
}

.form-group {
  margin-bottom: 15px;
}

.form-group label {
  display: block;
  margin-bottom: 5px;
  font-weight: 600;
}

input, textarea {
  width: 100%;
  padding: 10px;
  border: 1px solid #ddd;
  border-radius: 4px;
  font-size: 1rem;
}

.btn-submit {
  background-color: #3498db;
  color: white;
  border: none;
  padding: 10px 15px;
  border-radius: 4px;
  cursor: pointer;
  font-size: 1rem;
  transition: background-color 0.3s;
}

.btn-submit:hover {
  background-color: #2980b9;
}

.task-list h2 {
  margin-bottom: 15px;
  color: #2c3e50;
}

.task-list ul {
  list-style: none;
}

.task-item {
  padding: 15px;
  border-bottom: 1px solid #eee;
  display: flex;
  align-items: center;
}

.task-item:last-child {
  border-bottom: none;
}

.task-item.completed .task-header h3 {
  text-decoration: line-through;
  color: #7f8c8d;
}

.task-content {
  flex-grow: 1;
}

.task-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 5px;
}

.task-header h3 {
  font-size: 1.2rem;
  color: #2c3e50;
}

.task-description {
  color: #7f8c8d;
  margin-bottom: 5px;
}

.task-meta {
  font-size: 0.8rem;
  color: #95a5a6;
}

.task-actions {
  display: flex;
}

.btn-toggle, .btn-delete {
  padding: 5px 10px;
  border: none;
  border-radius: 4px;
  margin-left: 10px;
  cursor: pointer;
  font-size: 0.9rem;
}

.btn-toggle {
  background-color: #2ecc71;
  color: white;
}

.btn-toggle:hover {
  background-color: #27ae60;
}

.btn-delete {
  background-color: #e74c3c;
  color: white;
}

.btn-delete:hover {
  background-color: #c0392b;
}

.loading, .error-message, .empty-list {
  text-align: center;
  padding: 20px;
  color: #7f8c8d;
}

.error-message {
  color: #e74c3c;
}

footer {
  text-align: center;
  margin-top: 40px;
  color: #7f8c8d;
  font-size: 0.9rem;
}
```

### 🚀 Projekt starten und testen

Nachdem du alle Dateien erstellt hast, kannst du das Projekt mit folgenden Schritten starten:

1. Navigiere zum Projektverzeichnis:
   ```bash
   cd todo-app
   ```

2. Starte die Container:
   ```bash
   docker compose up -d
   ```

3. Überprüfe, ob alles läuft:
   ```bash
   docker compose ps
   ```

4. Öffne diese URLs in deinem Browser:
   - React-Frontend: http://localhost:3000
   - Express-API: http://localhost:4000/api/tasks

### 🔧 Tipps und Troubleshooting

#### **Fehlerbehebung**

- **Problem**: React-App zeigt keine Daten an.
  **Lösung**: Überprüfe die Netzwerkkonsole im Browser auf CORS-Fehler. Stelle sicher, dass `REACT_APP_API_URL` korrekt ist.

- **Problem**: Express-Backend kann keine Verbindung zur Datenbank herstellen.
  **Lösung**: Überprüfe die Logs mit `docker compose logs backend` und stelle sicher, dass die MongoDB-URL korrekt ist.

- **Problem**: Änderungen am Code werden nicht übernommen.
  **Lösung**: Bei React sollte Hot-Reloading automatisch funktionieren. Für das Backend verwenden wir nodemon, was ebenfalls automatisches Neuladen bietet.

#### **Erweiterungen**

Du kannst dieses Beispiel erweitern, indem du:

1. **Redis für Caching** hinzufügst:
   ```yaml
   # In der docker-compose.yml
   redis:
     image: redis:alpine
     container_name: todo_redis
     networks:
       - backend-network
   ```

2. **Nginx als Reverse Proxy** für Produktion hinzufügst:
   ```yaml
   # In der docker-compose.yml
   nginx:
     image: nginx:alpine
     container_name: todo_nginx
     ports:
       - "80:80"
     volumes:
       - ./nginx/default.conf:/etc/nginx/conf.d/default.conf
       - ./frontend/build:/usr/share/nginx/html
     depends_on:
       - frontend
       - backend
     networks:
       - frontend-network
   ```

3. **Environment-Datei** für bessere Konfigurierbarkeit nutzt:
   ```
   # .env
   MONGO_INITDB_DATABASE=tododb
   BACKEND_PORT=4000
   FRONTEND_PORT=3000
   NODE_ENV=development
   ```

   In der `docker-compose.yml` referenzierst du diese Variablen dann:
   ```yaml
   environment:
     - PORT=${BACKEND_PORT}
   ```

### 🎓 Was wir gelernt haben

In diesem praktischen Beispiel haben wir:

1. Eine **moderne JavaScript-Stack-Anwendung** mit Docker Compose aufgesetzt
2. **Services definiert** und konfiguriert (React-Frontend, Express-Backend, MongoDB)
3. Die **Kommunikation zwischen Containern** über Netzwerke eingerichtet
4. **Datenpersistenz** mit Volumes sichergestellt
5. Eine optimierte **Entwicklungsumgebung** mit Hot-Reloading

Diese Grundlagen kannst du auf größere und komplexere JavaScript-Projekte übertragen - das Prinzip bleibt das gleiche!

> **💡 Tipp:** Bewahre dieses Beispiel als Vorlage für zukünftige React/Node.js-Projekte auf. Es bietet eine gute Basis, die du für spezifische Anforderungen anpassen kannst.

---

## 🛠️ 5. Wichtige Konfigurationstechniken

### 🌐 Umgebungsvariablen und .env Dateien

Umgebungsvariablen sind entscheidend für die Konfiguration deiner Docker Compose-Anwendung. Sie ermöglichen eine flexible und sichere Konfiguration über verschiedene Umgebungen hinweg.

#### 📋 .env Datei Grundlagen

Erstelle eine `.env`-Datei im Stammverzeichnis deines Projekts:

```env
# Datenbank-Konfiguration
MONGO_DATABASE=tododb
MONGO_USER=todoapp
MONGO_PASSWORD=supersecretpassword

# Backend-Konfiguration
NODE_ENV=development
BACKEND_PORT=4000
API_PREFIX=/api/v1

# Frontend-Konfiguration
REACT_APP_API_URL=http://localhost:4000
REACT_APP_FEATURE_FLAGS={"darkMode":true}

# Logging-Level
LOG_LEVEL=info
```

#### 🔧 Docker Compose Integration

Aktualisiere deine `docker-compose.yml`, um Umgebungsvariablen zu nutzen:

```yaml
version: '3.8'

services:
  backend:
    build: ./backend
    environment:
      - MONGO_URI=mongodb://${MONGO_USER}:${MONGO_PASSWORD}@db:27017/${MONGO_DATABASE}
      - NODE_ENV=${NODE_ENV}
      - PORT=${BACKEND_PORT}
      - LOG_LEVEL=${LOG_LEVEL}
    env_file:
      - .env

  frontend:
    build: ./frontend
    environment:
      - REACT_APP_API_URL=${REACT_APP_API_URL}
      - REACT_APP_FEATURE_FLAGS=${REACT_APP_FEATURE_FLAGS}
    env_file:
      - .env

  db:
    environment:
      - MONGO_INITDB_DATABASE=${MONGO_DATABASE}
      - MONGO_INITDB_ROOT_USERNAME=${MONGO_USER}
      - MONGO_INITDB_ROOT_PASSWORD=${MONGO_PASSWORD}
```

#### 💡 Beispiel: Dynamische Konfiguration in Node.js

```javascript
// backend/config.js
const dotenv = require('dotenv');
dotenv.config();

module.exports = {
  port: process.env.PORT || 4000,
  mongoUri: process.env.MONGO_URI || 'mongodb://localhost:27017/defaultdb',
  logLevel: process.env.LOG_LEVEL || 'warn',
  featureFlags: JSON.parse(process.env.REACT_APP_FEATURE_FLAGS || '{}')
};
```

#### 🔒 Sicherheitshinweise

- Füge `.env` zur `.gitignore` hinzu
- Nutze `.env.example` als Template ohne echte Anmeldedaten
- Verwende komplexe, zufällige Passwörter

### 🚪 Port-Mapping

Port-Mapping ermöglicht die Kommunikation zwischen Containern und dem Host-System.

#### Syntax in Docker Compose

```yaml
services:
  frontend:
    ports:
      - "3000:3000"     # Host-Port:Container-Port
  backend:
    ports:
      - "4000:4000"
  adminer:
    ports:
      - "8080:8080"     # Zusätzliche Services
```

#### 🧩 Dynamische Port-Zuordnung

```yaml
services:
  backend:
    ports:
      - "${BACKEND_PORT}:4000"  # Flexibel über Umgebungsvariable
```

### 🔗 Abhängigkeiten zwischen Services

Definiere Abhängigkeiten, um eine korrekte Startreihenfolge zu gewährleisten:

```yaml
services:
  db:
    image: mongo:latest
    
  backend:
    build: ./backend
    depends_on:
      - db
    # Wartet, bis 'db' gestartet ist
    
  frontend:
    build: ./frontend
    depends_on:
      - backend
    # Stellt sicher, dass Backend läuft
```

#### Healthcheck für robustere Abhängigkeiten

```yaml
services:
  backend:
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:4000/health"]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 40s
    depends_on:
      db:
        condition: service_healthy
```

### 🔄 Restart-Policies

Definiere Neustart-Verhalten für verschiedene Szenarien:

```yaml
services:
  backend:
    restart: on-failure  # Nur bei Fehler neu starten
    
  db:
    restart: always      # Immer neu starten
    
  frontend:
    restart: unless-stopped  # Startet neu, außer manuell gestoppt
```

#### Restart-Policy Optionen

| Policy | Beschreibung |
|--------|--------------|
| `no` | Container startet nicht automatisch (Standard) |
| `always` | Immer neu starten, auch nach Docker-Neustart |
| `on-failure` | Nur bei Fehler-Exit neu starten |
| `unless-stopped` | Startet immer neu, außer manuell gestoppt |

### 🚀 Praxisbeispiel: Komplexe Konfiguration

```yaml
version: '3.8'

services:
  frontend:
    build: 
      context: ./frontend
      args:
        - NODE_ENV=${NODE_ENV}
    ports:
      - "${FRONTEND_PORT}:3000"
    environment:
      - REACT_APP_API_URL=${API_URL}
    restart: unless-stopped
    depends_on:
      - backend

  backend:
    build: 
      context: ./backend
      args:
        - NODE_ENV=${NODE_ENV}
    ports:
      - "${BACKEND_PORT}:4000"
    environment:
      - MONGO_URI=mongodb://${MONGO_USER}:${MONGO_PASSWORD}@db/${MONGO_DATABASE}
      - JWT_SECRET=${JWT_SECRET}
    restart: on-failure
    depends_on:
      db:
        condition: service_healthy
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:4000/health"]
      interval: 30s
      timeout: 10s
      retries: 3

  db:
    image: mongo:latest
    volumes:
      - mongodb_data:/data/db
    environment:
      - MONGO_INITDB_DATABASE=${MONGO_DATABASE}
      - MONGO_INITDB_ROOT_USERNAME=${MONGO_USER}
      - MONGO_INITDB_ROOT_PASSWORD=${MONGO_PASSWORD}
    restart: always
    healthcheck:
      test: echo 'db.runCommand("ping").ok' | mongosh localhost:27017/test
      interval: 10s
      timeout: 5s
      retries: 3

volumes:
  mongodb_data:
```

### 💡 Praxis-Tipps

1. Verwalte Umgebungsvariablen zentral
2. Nutze Healthchecks für robuste Service-Abhängigkeiten
3. Wähle passende Restart-Policies
4. Dokumentiere Konfigurationsoptionen
5. Trenne Entwicklungs- und Produktionskonfigurationen

> **🚀 Zusammenfassung:** Fortgeschrittene Docker Compose-Konfigurationen ermöglichen flexible, sichere und robuste JavaScript-Anwendungen durch intelligente Nutzung von Umgebungsvariablen, Port-Mapping, Service-Abhängigkeiten und Restart-Strategien.

Schaue dir diese Unterlagen genau an und analysiere sie. Erste anschließend eine Markdown mit dem Folgenden Kapitel, welche quasi direk anknüpgt an diese. gebe mir dann aber nur den neuen Teil aus als mardown datei. Es werden als technologien ausschließlich JAvascript, html, nodejs, react und express verwendet

## 🛡️ 6. Best Practices für Docker Compose in JavaScript-Projekten

### 🔒 Sicherheitsaspekte

#### Dependency Sicherheit

##### 1. Package Vulnerabilities Prüfen

Integriere Sicherheits-Checks in deine Entwicklungspipeline:

```json
{
  "scripts": {
    "security-check": "npm audit",
    "update-deps": "npm update",
    "audit-fix": "npm audit fix"
  }
}
```

##### 2. Docker Image Sicherheit

```dockerfile
# Verwende offizielle und geprüfte Basis-Images
FROM node:18-alpine AS base

# Führe als Nicht-Root-Benutzer aus
USER node

# Minimiere Image-Größe und Angriffsfläche
RUN apk add --no-cache tini
ENTRYPOINT ["/sbin/tini", "--"]
```

##### 3. Docker Compose Sicherheitskonfiguration

```yaml
services:
  backend:
    # Begrenzte Berechtigungen
    cap_drop:
      - ALL
    cap_add:
      - NET_BIND_SERVICE
    
    # Read-Only Filesystem
    read_only: true
    
    # Sicherheits-Optionen
    security_opt:
      - no-new-privileges:true
```

#### 🔐 Secrets Management

```yaml
services:
  backend:
    secrets:
      - db_password
      - jwt_secret

secrets:
  db_password:
    file: ./secrets/db_password.txt
  jwt_secret:
    external: true
```

#### 📂 Organisation von Compose-Dateien

##### Modularisierung

```
docker/
├── base.yml            # Gemeinsame Basis-Konfiguration
├── development.yml     # Entwicklungs-spezifische Konfigurationen
├── production.yml      # Produktions-Konfigurationen
└── docker-compose.yml  # Hauptkompositionsdatei
```

###### Beispiel für modulare Komposition

```bash
# Entwicklungs-Umgebung starten
docker compose \
  -f docker/base.yml \
  -f docker/development.yml \
  up -d

# Produktions-Umgebung starten
docker compose \
  -f docker/base.yml \
  -f docker/production.yml \
  up -d
```

#### Compose-Datei-Struktur

```yaml
version: '3.8'

x-common-variables: &common-vars
  NODE_ENV: ${NODE_ENV:-development}
  LOG_LEVEL: ${LOG_LEVEL:-info}

services:
  backend: &backend-defaults
    build: 
      context: ./backend
      args:
        - NODE_ENV=${NODE_ENV}
    <<: *backend-defaults
    environment:
      <<: *common-vars
      MONGO_URI: ${MONGO_URI}
```

### 🐛 Häufige Fehler und deren Behebung

#### Typische Docker Compose Probleme

1. **Netzwerk-Konnektivität**
```yaml
services:
  backend:
    # Explizite Netzwerk-Konfiguration
    networks:
      - app_network
    
    # Debugging-Optionen
    command: sh -c "nc -z db 27017 || exit 1"
```

2. **Volume-Berechtigungen**
```yaml
services:
  frontend:
    volumes:
      # Explizite Berechtigungen
      - ./frontend:/app:delegated
      - /app/node_modules
```

3. **Abhängigkeits-Probleme**
```yaml
services:
  backend:
    depends_on:
      db:
        condition: service_healthy
    
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:4000/health"]
      interval: 30s
      timeout: 10s
      retries: 5
```

### 🚀 Entwicklung vs. Produktion

#### Entwicklungs-Konfiguration

```yaml
# docker/development.yml
services:
  backend:
    volumes:
      - ./backend:/app
      - /app/node_modules
    environment:
      - NODE_ENV=development
      - DEBUG=*
    command: npm run dev

  frontend:
    volumes:
      - ./frontend:/app
      - /app/node_modules
    environment:
      - REACT_APP_API_URL=http://localhost:4000
      - CHOKIDAR_USEPOLLING=true
```

#### Produktions-Konfiguration

```yaml
# docker/production.yml
services:
  backend:
    build:
      target: production
    environment:
      - NODE_ENV=production
      - LOG_LEVEL=error
    command: npm run start

  frontend:
    build:
      target: production
    volumes:
      - frontend-build:/app/build
    environment:
      - REACT_APP_API_URL=https://api.example.com

  nginx:
    image: nginx:alpine
    volumes:
      - frontend-build:/usr/share/nginx/html
    ports:
      - "80:80"

volumes:
  frontend-build:
```

#### Build-Strategien

```dockerfile
# Multi-Stage Dockerfile für optimierte Builds
FROM node:18-alpine AS base
WORKDIR /app
COPY package*.json ./
RUN npm ci

FROM base AS development
ENV NODE_ENV=development
COPY . .
CMD ["npm", "run", "dev"]

FROM base AS production
ENV NODE_ENV=production
COPY . .
RUN npm run build
CMD ["npm", "start"]
```

### 💡 Performance-Optimierungen

1. **Caching-Strategien**
```yaml
services:
  backend:
    cache_from:
      - my-registry.com/myapp/backend:latest
```

2. **Resource-Limits**
```yaml
services:
  backend:
    deploy:
      resources:
        limits:
          cpus: '0.50'
          memory: 512M
        reservations:
          cpus: '0.25'
          memory: 256M
```

### 🔍 Monitoring und Logging

```yaml
services:
  backend:
    logging:
      driver: "json-file"
      options:
        max-size: "200k"
        max-file: "10"
```

### 💻 Entwickler-Workflow

1. Lokale Entwicklung mit Hot-Reloading
2. Continuous Integration (CI) Builds
3. Produktions-Deployment

> **🚀 Zusammenfassung:** Professionelle Docker Compose-Konfigurationen erfordern durchdachte Sicherheits-, Organisations- und Deployment-Strategien. Trenne Entwicklungs- und Produktionsumgebungen, minimiere Sicherheitsrisiken und optimiere deine Build-Prozesse.
