# Einführung in Docker: Grundlagen für Einsteiger

## Was ist Docker?

Docker ist eine Plattform, die es ermöglicht, Anwendungen in isolierten Umgebungen – sogenannten "Containern" – zu verpacken, zu verteilen und auszuführen. Diese Container enthalten alles, was eine Anwendung zum Laufen braucht: Code, Laufzeitumgebung, Systembibliotheken und Einstellungen.

Stelle dir einen Container wie einen standardisierten Versandcontainer vor: So wie physische Container Waren sicher von A nach B transportieren, sorgt Docker dafür, dass deine Anwendung überall gleich funktioniert – vom Entwicklungslaptop bis zum Produktionsserver.

## Warum Docker?

### Das klassische Problem

Eines der größten Probleme in der Softwareentwicklung ist die Inkonsistenz zwischen verschiedenen Umgebungen. Entwickler kennen den frustrierenden Moment:

"Bei mir läuft's aber!"

Diese Probleme entstehen durch unterschiedliche:
- Betriebssystemversionen
- Installierte Bibliotheken
- Umgebungsvariablen
- Konfigurationen

### Die Docker-Lösung

Docker löst dieses Problem durch Containerisierung. Ein Docker-Container kapselt eine Anwendung mit ihrer gesamten Umgebung, sodass sie unabhängig vom darunterliegenden System gleich ausgeführt wird.

#### Vorteile:

1. **Konsistente Entwicklungsumgebungen**
   - Jeder arbeitet mit exakt der gleichen Umgebung
   - "Works on my machine" wird zu "Works on any machine"

2. **Isolation von Anwendungen**
   - Jede Anwendung läuft in ihrer eigenen Umgebung
   - Keine Konflikte zwischen verschiedenen Anwendungen

3. **Effizienter Ressourceneinsatz**
   - Container teilen sich den Kernel des Host-Systems
   - Deutlich ressourcenschonender als virtuelle Maschinen

4. **Schnelligkeit**
   - Container starten in Sekunden (statt Minuten bei VMs)
   - Schnellere Entwicklungszyklen und Tests

![Container vs VM](https://github.com/user-attachments/assets/d654ec2a-d13b-482e-97d5-27d13698a241)

quelle: https://jfrog.com/de/devops-tools/article/beginners-guide-to-docker/

## Die technischen Grundlagen

### Container vs. Virtuelle Maschinen

Der Unterschied zwischen Containern und virtuellen Maschinen (VMs) ist entscheidend:

#### Virtuelle Maschinen:
- Enthalten ein vollständiges Gastbetriebssystem
- Virtualisieren die komplette Hardware
- Benötigen viel Speicher
- Starten langsam (Minuten)

#### Container:
- Teilen sich den Kernel des Host-Betriebssystems
- Virtualisieren nur die Anwendungsebene
- Sind leichtgewichtig (MB statt GB)
- Starten in Sekundenbruchteilen

### Docker-Architektur

Die Docker-Plattform besteht aus mehreren Kernkomponenten:

#### 1. Docker Engine
Das Herzstück von Docker ist die Docker Engine – ein Dienst, der auf dem Host-System läuft und:
- Container erstellt und verwaltet
- Images verwaltet
- Mit Registries kommuniziert

#### 2. Docker Client
Der Docker Client (`docker`) ist das Tool, mit dem Benutzer mit Docker interagieren. 

#### 3. Docker Registry
Eine Docker Registry speichert Docker-Images. Docker Hub ist die öffentliche Registry mit tausenden öffentlichen Images.

### Container vs. Virtuelle Maschinen: Der grundlegende Unterschied

![Container vs VMs Detailliert](https://github.com/user-attachments/assets/e66fdbcf-416b-4c3d-b5f9-81b18cbf1d3d)

quelle: https://jfrog.com/de/devops-tools/article/beginners-guide-to-docker/

Das Bild zeigt einen fundamentalen Vergleich zwischen zwei Virtualisierungstechnologien: **Container** (links) und **Virtuelle Maschinen** (rechts).

#### Container-Architektur (linke Seite):
- **Containerized Applications (App A bis App F)**: Mehrere isolierte Anwendungen laufen als Container
- **Container Engine**: Die Laufzeitumgebung (z.B. Docker Engine), die Container verwaltet
- **Host Operating System**: Das Betriebssystem auf dem physischen Rechner
- **Infrastructure**: Die zugrunde liegende Hardware

#### Virtuelle Maschinen-Architektur (rechte Seite):
- **Virtual Machine mit App A, B und C**: Jede VM enthält eine komplette Betriebssysteminstanz
- **Guest Operating System**: Jede VM hat ihr eigenes vollständiges Betriebssystem
- **Hypervisor**: Software-Layer, der die Erstellung und Verwaltung von VMs ermöglicht
- **Host Operating System**: Das Betriebssystem des physischen Rechners
- **Infrastructure**: Die zugrunde liegende Hardware

#### Hauptunterschiede auf einen Blick:

| Aspekt | Container | Virtuelle Maschinen |
|--------|-----------|---------------------|
| Betriebssystem | Teilen das Host-OS | Jede VM hat ein eigenes OS |
| Größe | Megabytes | Gigabytes |
| Startzeit | Sekunden | Minuten |
| Isolation | Prozess-Ebene | Hardware-Ebene |
| Ressourcennutzung | Sehr effizient | Ressourcenintensiver |
| Anwendungsfokus | Eine Anwendung pro Container | Kann mehrere Anwendungen enthalten |

Diese unterschiedlichen Ansätze haben jeweils ihre eigenen Vorteile und Anwendungsbereiche. In modernen Umgebungen werden oft beide Technologien kombiniert, um ihre jeweiligen Stärken zu nutzen.

### Zentrale Docker-Konzepte

#### 1. Images
Ein Docker-Image ist eine unveränderliche (read-only) Vorlage, die alles enthält, was zum Starten eines Containers benötigt wird. Du kannst es dir wie eine Schablone oder eine Blaupause vorstellen. Aus einem Image können viele Container erstellt werden.

Ein Docker-Image:
- Besteht aus mehreren Schichten (Layers)
- Wird mit einem Dockerfile definiert
- Wird in Registries gespeichert
- Ist unveränderlich (read-only)

#### 2. Container
Ein Container ist eine laufende Instanz eines Images. Container:
- Haben ihre eigenen Dateisysteme
- Sind voneinander isoliert
- Können miteinander kommunizieren

#### 3. Dockerfile
Ein Dockerfile ist eine einfache Textdatei mit Anweisungen, die beschreiben, wie ein Docker-Container erstellt werden soll. Es ist wie ein Rezept oder eine Bauanleitung für ein Docker-Image:
- Es listet alle benötigten "Zutaten" (Programme, Dateien) auf
- Es beschreibt, wie diese "Zutaten" zusammengesetzt werden
- Es legt fest, was passiert, wenn der Container startet

## Die wichtigsten Dockerfile-Befehle

Hier sind die grundlegenden Dockerfile-Befehle, die du kennen solltest:

### FROM

```dockerfile
FROM nginx:alpine
```

**Erklärung in einfachen Worten:**
- "Nimm dieses fertige Paket als Grundlage"
- Jedes Dockerfile muss mit FROM beginnen
- Es gibt viele fertige Images auf Docker Hub (wie nginx, ubuntu, python)
- "alpine" ist eine besonders kleine Linux-Version

### COPY

```dockerfile
COPY index.html /usr/share/nginx/html/
```

**Erklärung in einfachen Worten:**
- "Kopiere diese Datei von meinem Computer in den Container"
- Links: Die Datei auf deinem Computer
- Rechts: Wohin sie im Container kopiert werden soll
- In unserem Beispiel: Kopiere die HTML-Datei dorthin, wo der Webserver sie finden kann

### EXPOSE

```dockerfile
EXPOSE 80
```

**Erklärung in einfachen Worten:**
- "Dieser Container wird auf Port 80 erreichbar sein"
- Dies ist nur eine Information/Dokumentation
- Um den Port wirklich zugänglich zu machen, brauchst du später den "-p" Parameter

### CMD

```dockerfile
CMD ["nginx", "-g", "daemon off;"]
```

**Erklärung in einfachen Worten:**
- "Führe diesen Befehl aus, wenn der Container startet"
- Dies ist der Hauptprozess des Containers
- Wenn dieser Prozess beendet wird, stoppt auch der Container
- In unserem Beispiel: Starte den Nginx-Webserver im Vordergrund

### WORKDIR

```dockerfile
WORKDIR /app
```

**Erklärung in einfachen Worten:**
- "Wechsle in dieses Verzeichnis für die nächsten Befehle"
- Wie ein "cd" in der Kommandozeile
- Erstellt das Verzeichnis automatisch, falls es nicht existiert

### ENV

```dockerfile
ENV NAME="Wert"
```

**Erklärung in einfachen Worten:**
- "Speichere diese Information im Container"
- Setzt eine Umgebungsvariable
- Programme im Container können darauf zugreifen
- Zum Beispiel: `ENV PORT=8080` setzt die Variable PORT auf den Wert 8080

### RUN

```dockerfile
RUN echo "Hallo" > /tmp/test.txt
```

**Erklärung in einfachen Worten:**
- "Führe diesen Befehl während des Baus des Images aus"
- Wird nur einmal ausgeführt, wenn das Image erstellt wird
- Gut für Installation von Software oder Vorbereitung des Images

### Beispiel eines vollständigen Dockerfiles

```dockerfile
# Basis-Image definieren
FROM node:14

# Arbeitsverzeichnis im Container
WORKDIR /app

# Abhängigkeiten installieren
COPY package.json ./
RUN npm install

# Anwendungscode kopieren
COPY . .

# Port freigeben
EXPOSE 3000

# Startbefehl definieren
CMD ["npm", "start"]
```

## Docker-Befehle in der Kommandozeile

Diese Befehle wirst du häufig von der Kommandozeile aus verwenden:

### docker build

```bash
docker build -t meine-webseite .
```

**Erklärung in einfachen Worten:**
- "Erstelle ein Image aus dem Dockerfile"
- `-t meine-webseite`: Gib dem Image einen Namen (Tag)
- `.`: Suche das Dockerfile im aktuellen Verzeichnis

### docker run

```bash
docker run -d -p 8080:80 --name mein-webserver meine-webseite
```

**Erklärung in einfachen Worten:**
- "Starte einen Container aus diesem Image"
- `-d`: Lasse ihn im Hintergrund laufen (detached)
- `-p 8080:80`: Verbinde Port 8080 auf deinem Computer mit Port 80 im Container
- `--name mein-webserver`: Gib dem Container einen Namen
- `meine-webseite`: Verwende dieses Image

### docker stop

```bash
docker stop mein-webserver
```

**Erklärung in einfachen Worten:**
- "Halte diesen Container an"
- Der Container wird gestoppt, aber nicht gelöscht
- Du kannst ihn später wieder starten

### docker rm

```bash
docker rm mein-webserver
```

**Erklärung in einfachen Worten:**
- "Lösche diesen Container"
- Entfernt den Container vollständig
- Die Daten im Container gehen verloren, wenn sie nicht woanders gespeichert wurden

### docker logs

```bash
docker logs mein-webserver
```

**Erklärung in einfachen Worten:**
- "Zeige mir alle Ausgaben dieses Containers"
- Nützlich, um Probleme zu finden oder zu sehen, was im Container passiert

### docker exec

```bash
docker exec -it mein-webserver sh
```

**Erklärung in einfachen Worten:**
- "Öffne eine Kommandozeile im laufenden Container"
- `-it`: Macht die Verbindung interaktiv (du kannst Befehle eingeben)
- `sh`: Starte eine Shell im Container
- Du kannst dann Befehle im Container ausführen, wie wenn du direkt darin wärst

### docker images

```bash
docker images
```

**Erklärung in einfachen Worten:**
- "Zeige mir alle Images, die ich auf meinem Computer habe"
- Listet alle lokal gespeicherten Images auf

### docker ps

```bash
docker ps
```

**Erklärung in einfachen Worten:**
- "Zeige mir alle laufenden Container"
- Mit `-a` werden auch gestoppte Container angezeigt: `docker ps -a`

### docker pull

```bash
docker pull nginx
```

**Erklärung in einfachen Worten:**
- "Lade dieses Image herunter, ohne es zu starten"
- Nützlich, um Images vorab zu laden

## Wichtige Konzepte für die Übung

### Basis-Images

Docker-Images bauen aufeinander auf. In unserer Übung verwenden wir `nginx:alpine` als Basis. Dies ist ein fertiges Image mit dem Nginx-Webserver auf Basis von Alpine Linux (einer sehr kleinen Linux-Distribution).

### Container vs. Images

- **Image**: Die unveränderliche Vorlage (wie eine ISO-Datei)
- **Container**: Eine laufende Instanz eines Images (wie ein installiertes Betriebssystem)

Ein wichtiges Konzept: Du kannst ein Image erstellen, aber nicht mehr verändern. Um Änderungen zu machen, musst du ein neues Image bauen.

### Port-Weiterleitung

Container haben ihr eigenes Netzwerk. Wenn ein Programm im Container auf Port 80 läuft, ist es zunächst nur innerhalb des Containers erreichbar. Mit der Port-Weiterleitung `-p 8080:80` sagst du Docker:

"Wenn jemand auf meinem Computer Port 8080 aufruft, leite die Anfrage an Port 80 im Container weiter."

### Container-Lebenszyklus

1. **Erstellen**: `docker build` erstellt ein Image
2. **Starten**: `docker run` startet einen Container aus dem Image
3. **Stoppen**: `docker stop` hält den Container an
4. **Entfernen**: `docker rm` löscht den Container

Wichtig: Ein gestoppter Container behält seinen Zustand bei, aber wenn du einen Container löschst, gehen alle Änderungen verloren, die nicht in einem Volume oder auf andere Weise gespeichert wurden.

## Ein einfaches praktisches Beispiel

Hier ist ein Beispiel für eine einfache Web-Anwendung mit Docker:

1. Erstellen einer `index.html`:
```html
<!DOCTYPE html>
<html>
<head>
    <title>Docker Demo</title>
</head>
<body>
    <h1>Hallo Docker!</h1>
    <p>Diese Seite wird in einem Docker-Container ausgeliefert.</p>
</body>
</html>
```

2. Erstellen eines Dockerfiles:
```dockerfile
FROM nginx:alpine
COPY index.html /usr/share/nginx/html/index.html
```

3. Image bauen:
```bash
docker build -t demo-webserver .
```

4. Container starten:
```bash
docker run -d -p 8080:80 demo-webserver
```

5. Im Browser öffnen: http://localhost:8080
