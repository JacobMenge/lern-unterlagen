# 🐳 Docker Container Monitoring Dashboard: Vollständige Erklärung

## Projektübersicht

Dieses Projekt erstellt ein Echtzeit-Monitoring-Dashboard für Docker-Container. Es besteht aus zwei Hauptkomponenten:

1. **Log-Collector**: Sammelt Statistiken von allen laufenden Docker-Containern
2. **Web-Frontend**: Zeigt diese Statistiken in einer benutzerfreundlichen Weboberfläche an

Die beiden Komponenten kommunizieren über ein gemeinsam genutztes Docker Volume, wodurch der Datenaustausch ohne direkte Netzwerkkommunikation ermöglicht wird.

## Architektur

```
Log-Collector   [Volume]   Web-Frontend
    |               |           |
    | Writes -----> |           |
    |               |           |
    |               | <--- Reads|
    |               |           |
```

Diese Architektur demonstriert ein wichtiges Konzept in Docker: die Verwendung von Volumes zur gemeinsamen Nutzung von Daten zwischen Containern.

## Projektstruktur

```
~/container-monitor/
├── log-collector/
│   ├── Dockerfile
│   └── collect-stats.sh
└── web-frontend/
    ├── Dockerfile
    ├── index.html
    └── script.js
```

## Komponentenerklärung

### 1. Log-Collector

#### collect-stats.sh

```bash
#!/bin/bash

# Zielverzeichnis für Logs
LOGS_DIR="/logs"
mkdir -p $LOGS_DIR

# Funktion zum Sammeln von Container-Statistiken
collect_container_stats() {
    while true; do
        # Sammle Docker-Container-Statistiken
        docker stats --no-stream --format "{{.Name}},{{.CPUPerc}},{{.MemUsage}},{{.NetIO}},{{.BlockIO}}" | 
        while IFS=',' read -r name cpu mem net block; do
            # Zeitstempel und Statistiken in Datei schreiben
            echo "$(date '+%Y-%m-%d %H:%M:%S'),$name,$cpu,$mem,$net,$block" >> "$LOGS_DIR/container_stats.csv"
        done
        
        # Warte 5 Sekunden zwischen den Sammlungen
        sleep 5
    done
}

# CSV-Header generieren
echo "Timestamp,ContainerName,CPU,Memory,Network,BlockIO" > "$LOGS_DIR/container_stats.csv"

# Statistiken sammeln
collect_container_stats
```

**Erklärung:**

1. Das Skript definiert ein Verzeichnis `/logs`, in dem die Statistiken gespeichert werden.
2. Die Funktion `collect_container_stats()` sammelt kontinuierlich Statistiken von allen laufenden Docker-Containern:
   - `docker stats --no-stream` liefert eine einmalige Ausgabe der aktuellen Container-Statistiken
   - `--format "{{.Name}},{{.CPUPerc}},...` formatiert die Ausgabe als CSV-Daten
   - Mit einer while-Schleife wird jede Zeile verarbeitet und mit Zeitstempel in die CSV-Datei geschrieben
   - `sleep 5` sorgt für ein 5-Sekunden-Intervall zwischen den Messungen
3. Vor dem Datensammeln wird eine Kopfzeile in die CSV-Datei geschrieben
4. Schließlich wird die Funktion aufgerufen, um das kontinuierliche Sammeln zu starten

#### Dockerfile für Log-Collector

```dockerfile
FROM docker:20.10

# Docker CLI installieren
RUN apk add --no-cache bash

# Skript kopieren und ausführbar machen
COPY collect-stats.sh /collect-stats.sh
RUN chmod +x /collect-stats.sh

# Docker-Socket mounten
VOLUME /var/run/docker.sock

# Logs-Volume mounten
VOLUME /logs

CMD ["/collect-stats.sh"]
```

**Erklärung:**

1. Basis-Image ist `docker:20.10`, das bereits die Docker-CLI enthält
2. Bash wird installiert, da es für das Skript benötigt wird
3. Das Sammelskript wird in den Container kopiert und ausführbar gemacht
4. Zwei VOLUME-Anweisungen definieren:
   - `/var/run/docker.sock`: Ermöglicht dem Container, mit dem Docker-Daemon des Hosts zu kommunizieren
   - `/logs`: Das Verzeichnis, in dem die CSV-Datei gespeichert wird
5. Der Container startet automatisch das Skript

### 2. Web-Frontend

#### index.html

```html
<!DOCTYPE html>
<html lang="de">
<head>
    <meta charset="UTF-8">
    <title>Container Monitor</title>
    <style>
        body { 
            font-family: Arial, sans-serif; 
            max-width: 800px; 
            margin: 0 auto; 
            padding: 20px; 
        }
        table { 
            width: 100%; 
            border-collapse: collapse; 
        }
        th, td { 
            border: 1px solid #ddd; 
            padding: 8px; 
            text-align: left; 
        }
        th { 
            background-color: #f2f2f2; 
        }
    </style>
</head>
<body>
    <h1>🖥️ Container Monitoring</h1>
    <table id="statsTable">
        <thead>
            <tr>
                <th>Zeitstempel</th>
                <th>Container</th>
                <th>CPU</th>
                <th>Speicher</th>
                <th>Netzwerk</th>
                <th>Block-IO</th>
            </tr>
        </thead>
        <tbody id="statsBody"></tbody>
    </table>
    <script src="script.js"></script>
</body>
</html>
```

**Erklärung:**

1. Eine einfache HTML-Seite mit einem Tabellenlayout
2. Die Tabelle hat Spalten für alle gesammelten Statistikwerte
3. Der Tabellenkörper (`<tbody id="statsBody">`) wird dynamisch mit JavaScript gefüllt
4. Ein externes JavaScript-File `script.js` wird eingebunden, um die Daten zu laden und anzuzeigen

#### script.js

```javascript
function updateStats() {
    fetch('/logs/container_stats.csv')
        .then(response => response.text())
        .then(data => {
            const rows = data.trim().split('\n');
            const statsBody = document.getElementById('statsBody');
            
            // Letzten 10 Einträge anzeigen (Header überspringen)
            const displayRows = rows.slice(1).slice(-10);
            
            statsBody.innerHTML = displayRows.map(row => {
                const [timestamp, name, cpu, mem, net, block] = row.split(',');
                return `
                    <tr>
                        <td>${timestamp}</td>
                        <td>${name}</td>
                        <td>${cpu}</td>
                        <td>${mem}</td>
                        <td>${net}</td>
                        <td>${block}</td>
                    </tr>
                `;
            }).join('');
        })
        .catch(error => console.error('Fehler:', error));
}

// Aktualisiere Stats alle 5 Sekunden
setInterval(updateStats, 5000);
updateStats(); // Initialer Aufruf
```

**Erklärung:**

1. Die `updateStats()`-Funktion ruft die CSV-Datei über HTTP ab
2. Die Datei wird als Text geladen und in Zeilen aufgeteilt
3. Nur die letzten 10 Einträge werden zur Anzeige ausgewählt (der Header wird übersprungen)
4. Für jede Zeile werden die Werte in HTML-Tabellenzeilen umgewandelt und in die Tabelle eingefügt
5. Ein `setInterval()` ruft diese Funktion alle 5 Sekunden auf, um die Anzeige zu aktualisieren
6. Der erste Aufruf erfolgt sofort nach dem Laden der Seite

#### Dockerfile für Web-Frontend

```dockerfile
FROM nginx:alpine

# HTML und JavaScript kopieren
COPY index.html /usr/share/nginx/html/
COPY script.js /usr/share/nginx/html/

# Logs-Volume mounten (für CSV-Zugriff)
VOLUME /logs

# Nginx läuft standardmäßig
```

**Erklärung:**

1. Basis-Image ist `nginx:alpine`, ein schlanker Webserver
2. Die HTML- und JavaScript-Dateien werden in das Standardverzeichnis von Nginx kopiert
3. Ein VOLUME für `/logs` wird definiert, damit der Container auf die CSV-Datei zugreifen kann
4. Der Nginx-Webserver startet automatisch und serviert die Dateien

## Einrichtung und Ausführung

### Schritt 1: Projektstruktur erstellen

```bash
mkdir -p ~/container-monitor/log-collector ~/container-monitor/web-frontend
```

### Schritt 2: Dateien erstellen

Speichern Sie jede Datei an ihrem entsprechenden Ort:
- `collect-stats.sh` im Verzeichnis `log-collector`
- `index.html` und `script.js` im Verzeichnis `web-frontend`
- Die entsprechenden Dockerfiles in jedem Verzeichnis

Machen Sie das Skript ausführbar:
```bash
chmod +x ~/container-monitor/log-collector/collect-stats.sh
```

### Schritt 3: Docker-Images bauen

```bash
# Log-Collector-Image bauen
cd ~/container-monitor/log-collector
docker build -t log-collector-image .

# Web-Frontend-Image bauen
cd ~/container-monitor/web-frontend
docker build -t web-frontend-image .
```

### Schritt 4: Docker-Volume erstellen

```bash
docker volume create monitoring-logs
```

### Schritt 5: Container starten

```bash
# Log-Collector-Container starten
docker run -d \
  --name log-collector \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -v monitoring-logs:/logs \
  log-collector-image

# Web-Frontend-Container starten
docker run -d \
  --name web-frontend \
  -p 8080:80 \
  -v monitoring-logs:/logs \
  web-frontend-image
```

### Schritt 6: Dashboard aufrufen

Öffnen Sie Ihren Webbrowser und navigieren Sie zu:
```
http://localhost:8080
```

## Funktionsweise im Detail

1. **Datensammlung:**
   - Der Log-Collector-Container nutzt den Docker-Socket, um auf die Docker-API zuzugreifen
   - Über die Docker-API werden Statistiken aller laufenden Container abgefragt
   - Diese Statistiken werden mit Zeitstempel in eine CSV-Datei im gemeinsamen Volume geschrieben

2. **Datenaustausch:**
   - Das Docker-Volume `monitoring-logs` wird von beiden Containern gemountet
   - Der Log-Collector schreibt in das Volume, das Web-Frontend liest daraus
   - Diese Methode ermöglicht Datenaustausch ohne direkte Container-zu-Container-Kommunikation

3. **Datenvisualisierung:**
   - Der Nginx-Webserver im Web-Frontend serviert die HTML- und JavaScript-Dateien
   - Das JavaScript ruft regelmäßig die CSV-Datei ab und extrahiert die neuesten Daten
   - Die Daten werden in einer Tabelle dargestellt und alle 5 Sekunden aktualisiert

## Fehlerbehebung

### Wenn der Log-Collector keine Daten sammelt:
Überprüfen Sie, ob der Container Zugriff auf den Docker-Socket hat:
```bash
docker logs log-collector
```

### Wenn das Web-Frontend keine Daten anzeigt:
Stellen Sie sicher, dass beide Container Zugriff auf das gemeinsame Volume haben:
```bash
docker inspect monitoring-logs
docker exec -it web-frontend ls -la /logs
```

### Um die rohen Container-Statistikdaten anzusehen:
```bash
docker exec -it web-frontend cat /logs/container_stats.csv
```

## Lernziele und Konzepte

1. **Docker Volumes:**
   - Volumes sind der bevorzugte Mechanismus für die Persistenz von Daten in Docker
   - Sie ermöglichen die Dateifreigabe zwischen Host und Containern oder zwischen Containern
   - Anders als Bind Mounts werden Volumes von Docker verwaltet und sind unabhängig vom Host-Dateisystem

2. **Container-übergreifende Kommunikation:**
   - Dieses Projekt zeigt eine Alternative zu Netzwerk-basierter Kommunikation
   - Dateisystem-basierter Datenaustausch ist einfach zu implementieren und zu verstehen
   - Für bestimmte Anwendungsfälle kann dies effizienter sein als API-Aufrufe

3. **Systemüberwachung:**
   - Der Zugriff auf den Docker-Socket ermöglicht tiefe Einblicke in das Containersystem
   - Die Docker-CLI bietet leistungsstarke Tools zur Systemüberwachung
   - Realtime-Monitoring ist wichtig für die Systemwartung und das Debugging

4. **Webvisualisierung:**
   - Einfache HTML/JavaScript-Anwendungen können mächtige Dashboards erstellen
   - Die Fetch-API ermöglicht einfachen Datenabruf ohne komplexe Frameworks
   - Periodische Aktualisierungen durch `setInterval()` erzeugen einen Echtzeit-Eindruck
