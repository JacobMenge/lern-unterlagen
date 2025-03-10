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

![image](https://github.com/user-attachments/assets/e66fdbcf-416b-4c3d-b5f9-81b18cbf1d3d)

quelle: https://jfrog.com/de/devops-tools/article/beginners-guide-to-docker/

Das Bild zeigt einen fundamentalen Vergleich zwischen zwei Virtualisierungstechnologien: **Container** (links) und **Virtuelle Maschinen** (rechts).

### Container-Architektur (linke Seite)

Die linke Seite zeigt die Container-Architektur:

- **Containerized Applications (App A bis App F)**: Mehrere isolierte Anwendungen laufen als Container
- **Container Engine**: Die Laufzeitumgebung (z.B. Docker Engine), die Container verwaltet
- **Host Operating System**: Das Betriebssystem auf dem physischen Rechner
- **Infrastructure**: Die zugrunde liegende Hardware

#### Schlüsselmerkmale von Containern:
- Teilen sich den Kernel des Host-Betriebssystems
- Enthalten nur die Anwendung und ihre Abhängigkeiten
- Sehr leichtgewichtig (meist nur wenige MB)
- Starten in Sekundenbruchteilen
- Hohe Ressourceneffizienz

### Virtuelle Maschinen-Architektur (rechte Seite)

Die rechte Seite zeigt die Architektur von virtuellen Maschinen:

- **Virtual Machine mit App A, B und C**: Jede VM enthält eine komplette Betriebssysteminstanz
- **Guest Operating System**: Jede VM hat ihr eigenes vollständiges Betriebssystem
- **Hypervisor**: Software-Layer, der die Erstellung und Verwaltung von VMs ermöglicht
- **Host Operating System**: Das Betriebssystem des physischen Rechners
- **Infrastructure**: Die zugrunde liegende Hardware

#### Schlüsselmerkmale von virtuellen Maschinen:
- Jede VM enthält ein vollständiges Gastbetriebssystem
- Stärkere Isolation zwischen den VMs
- Höherer Ressourcenverbrauch (speicher- und CPU-intensiver)
- Längere Startzeit (oft Minuten)
- Größere Dateigröße (oft mehrere GB)

### Hauptunterschiede auf einen Blick

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
Ein Docker-Image ist eine unveränderliche Vorlage, die alles enthält, was zum Ausführen einer Anwendung notwendig ist. Images:
- Bestehen aus mehreren Schichten (Layers)
- Werden mit einem Dockerfile definiert
- Werden in Registries gespeichert

#### 2. Container
Ein Container ist eine laufende Instanz eines Images. Container:
- Haben ihre eigenen Dateisysteme
- Sind voneinander isoliert
- Können miteinander kommunizieren

#### 3. Dockerfile
Ein Dockerfile ist eine Textdatei mit Anweisungen zum Erstellen eines Docker-Images:

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

## Docker in der Praxis: Erste Schritte

### Grundlegende Docker-Befehle

#### Images verwalten:
```bash
# Image herunterladen
docker pull nginx

# Alle lokalen Images anzeigen
docker images

# Eigenes Image aus Dockerfile erstellen
docker build -t meine-app .
```

#### Container verwalten:
```bash
# Container starten
docker run -d -p 8080:80 nginx

# Laufende Container anzeigen
docker ps

# In einen laufenden Container einsteigen
docker exec -it mein-container bash

# Container stoppen
docker stop mein-container

# Container löschen
docker rm mein-container
```

### Ein einfaches Beispiel

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

## Zusammenfassung

Docker löst das Hauptproblem der Softwareentwicklung – die Umgebungsinkonsistenz – und bietet zahlreiche weitere Vorteile:

- Konsistente Umgebungen überall
- Isolierte Anwendungen
- Effiziente Ressourcennutzung
- Schneller Start und Betrieb

In den nächsten Abschnitten werden wir auf fortgeschrittene Themen eingehen und praktische Anwendungsfälle betrachten.
