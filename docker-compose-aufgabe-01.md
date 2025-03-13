# 🚀 Docker Compose Hello World Projekt

## 🎯 Was wir in dieser Übung machen werden

### Projektübersicht
In dieser Übung werden wir eine einfache Webanwendung mit Docker Compose erstellen. Unser Ziel ist es, die Grundlagen der Containerisierung zu verstehen und eine praktische Anwendung zu entwickeln, die:
- Eine Node.js-Anwendung enthält
- Mit Docker Compose containerisiert wird
- Umgebungsvariablen nutzt
- Einfache Routen bereitstellt

### Warum Docker Compose?
Docker Compose ist ein Tool, das dir hilft:
- Mehrere Container gleichzeitig zu verwalten
- Anwendungen konsistent zu entwickeln und zu deployen
- Komplexe Anwendungsumgebungen einfach zu konfigurieren

**Vergleich:** Stelle dir Docker Compose wie einen Dirigenten vor, der verschiedene Musiker (Container) koordiniert, sodass sie perfekt zusammenspielen.

### Was du lernen wirst
- Docker Compose Grundlagen
- Containerisierung einer Node.js-Anwendung
- Konfiguration von Containern
- Umgang mit Umgebungsvariablen
- Grundlegende Webanwendungs-Architektur

## 🔧 Docker Compose Installation und Überprüfung

### Installation überprüfen
```bash
# Warum prüfen? 
# Um sicherzustellen, dass Docker Compose installiert und funktionsfähig ist
docker compose version
```

### Installation (falls nicht vorhanden)

#### Für Linux
```bash
# Warum diese Befehle?
# Sie aktualisieren die Paketliste und installieren das Docker Compose Plugin
sudo apt-get update
sudo apt-get install docker-compose-plugin
```

#### Für Windows
1. Docker Desktop herunterladen und installieren
   - Warum? Docker Desktop bringt alle notwendigen Tools mit
   - Offizielle Docker Website: https://www.docker.com/products/docker-desktop

#### Für macOS
1. Docker Desktop herunterladen und installieren
   - Gleicher Grund wie bei Windows
   - Offizielle Docker Website: https://www.docker.com/products/docker-desktop

### Zusätzliche Überprüfung
```bash
# Prüft die Versionen von Docker und Docker Compose
# Hilft bei der Fehlersuche und Systemkonfiguration
docker --version
docker compose version
```

## 📁 Projektstruktur Erklärung
```
hello-world-docker/
│
├── src/                   # Quellcode-Verzeichnis
│   └── app.js             # Hauptanwendungsdatei
│
├── Dockerfile             # Anweisungen zum Bauen des Container-Images
├── docker-compose.yml     # Konfiguration für Docker Compose
├── package.json           # Node.js Projektmetadaten und Abhängigkeiten
└── .env                   # Umgebungsvariablen
```

**Warum diese Struktur?**
- Klare Trennung von Konfiguration und Code
- Einfache Wartbarkeit
- Standardisierte Projektorganisation

## 🚶‍♂️ Schritt-für-Schritt-Anleitung

### Schritt 1: Projekt initialisieren
```bash
# Projektordner erstellen
mkdir hello-world-docker
cd hello-world-docker

# Node.js-Projekt initialisieren
# Erstellt package.json mit Standardeinstellungen
npm init -y
```

**Warum diese Schritte?**
- Einen neuen Projektordner anlegen
- `npm init -y` generiert eine Basis-Projektkonfiguration
- Vorbereitung für Node.js-Anwendung

### Schritt 2: Abhängigkeiten installieren
```bash
# Notwendige Pakete installieren
# express: Webserver-Framework
# dotenv: Verwaltung von Umgebungsvariablen
npm install express dotenv
```

**Warum diese Pakete?**
- `express`: Vereinfacht das Erstellen von Webservern
- `dotenv`: Ermöglicht einfache Konfiguration durch Umgebungsvariablen

### Schritt 3: Hauptanwendung erstellen
Datei `src/app.js`:
```javascript
const express = require('express');
const dotenv = require('dotenv');

// Lädt Umgebungsvariablen aus .env-Datei
dotenv.config();

// Express-Anwendung erstellen
const app = express();

// Port und Begrüßungsnachricht aus Umgebungsvariablen
const PORT = process.env.PORT || 3000;
const GREETING = process.env.GREETING || 'Hallo Welt';

// Hauptroute: Gibt JSON-Informationen zurück
app.get('/', (req, res) => {
  res.json({
    message: GREETING,
    environment: process.env.NODE_ENV || 'development',
    timestamp: new Date().toISOString()
  });
});

// Gesundheits-Endpunkt für Systemüberwachung
app.get('/health', (req, res) => {
  res.status(200).json({
    status: 'healthy',
    uptime: process.uptime()
  });
});

// Server starten
app.listen(PORT, () => {
  console.log(`Server läuft auf Port ${PORT}`);
  console.log(`Aktuelle Begrüßung: ${GREETING}`);
});
```

**Warum diese Anwendungsstruktur?**
- Flexible Konfiguration durch Umgebungsvariablen
- Zwei Endpunkte: Hauptroute und Healthcheck
- Einfache Konfigurationsmöglichkeiten

### Schritt 4: Dockerfile erstellen
```dockerfile
# Basisimage: Offizielle Node.js Alpine-Version
# Alpine ist sehr klein und effizient
FROM node:18-alpine

# Arbeitsverzeichnis im Container festlegen
WORKDIR /app

# package.json und Abhängigkeiten kopieren
COPY package*.json ./

# Produktionsabhängigkeiten installieren
# 'npm ci' ist schneller und sicherer für CI/CD
RUN npm ci --only=production

# Quellcode in den Container kopieren
COPY src ./src

# Port freigeben, auf dem die App läuft
EXPOSE 3000

# Startbefehl für die Anwendung
CMD ["node", "src/app.js"]
```

**Warum diese Dockerfile-Konfiguration?**
- Alpine-Image reduziert Containergröße
- Trennung von Abhängigkeiten und Code
- Optimiert für Produktionsumgebungen

### Schritt 5: Umgebungsvariablen definieren
Datei `.env`:
```env
# Grundlegende Anwendungskonfiguration
PORT=3000                                 # Webserver-Port
NODE_ENV=development                      # Entwicklungsumgebung
GREETING=Hallo aus dem Docker Compose Container!  # Individuelle Begrüßung

# Zusätzliche Konfigurationsoptionen
LOG_LEVEL=info                            # Logging-Einstellung
```

**Vorteile von Umgebungsvariablen:**
- Sichere Konfiguration
- Einfache Anpassung zwischen Umgebungen
- Keine hartcodierten Werte im Code

### Schritt 6: Docker Compose Konfiguration
Datei `docker-compose.yml`:
```yaml
version: '3.8'  # Aktuelle Compose-Datei-Version

services:
  # Definiert unseren Webservice
  web:
    # Wie das Image gebaut wird
    build: 
      context: .             # Aktueller Ordner als Basis
      dockerfile: Dockerfile # Zu verwendende Dockerfile

    # Port-Mapping: Host:Container
    ports:
      - "3000:3000"          # Erreichbar unter localhost:3000

    # Umgebungsvariablen laden
    env_file:
      - .env

    # Entwicklungs-Volume: Code-Änderungen sofort sichtbar
    volumes:
      - ./src:/app/src

    # Neustart-Verhalten
    restart: unless-stopped  # Startet bei Absturz neu
```

**Warum diese Compose-Konfiguration?**
- Einfache Service-Definition
- Automatisches Build des Images
- Flexibles Port-Mapping
- Volumes für Entwicklung

### Anwendung starten und testen
```bash
# Container erstellen und starten
docker compose up --build

# Im Hintergrund laufen lassen
docker compose up -d --build

# Status überprüfen
docker compose ps

# Logs anzeigen
docker compose logs web

# Testen
curl http://localhost:3000
```

**Was passiert hier?**
- Container werden gebaut und gestartet
- Anwendung wird über localhost zugänglich
- Logs helfen bei Fehlersuche und Überprüfung

> **💡 Lernziel erreicht!** Du verstehst nun die Grundlagen von Docker Compose und Containerisierung.
> Du hast jetzt eine vollständig containerisierte Node.js-Anwendung erstellt, die:
> - Einfach zu starten ist
> - Umgebungsvariablen nutzt
> - Mit Docker Compose verwaltet wird
