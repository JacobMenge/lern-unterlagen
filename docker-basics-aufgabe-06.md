# 🐳 Docker Volume Challenge: Container-Monitoring-Dashboard

## Szenario
Entwickle eine Anwendung, die Systemressourcen und Container-Statistiken in Echtzeit visualisiert.

### Architektur
```
Log-Collector   [Volume]   Web-Frontend
    |               |           |
    | Writes -----> |           |
    |               |           |
    |               | <--- Reads|
    |               |           |
```

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

## Aufgabe: Container-Ressourcen-Monitor

### Teil 1: Log-Collector (Bash-Skript)
Erstelle `log-collector/collect-stats.sh`:
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

### Teil 2: Web-Frontend

#### HTML (web-frontend/index.html)
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

#### JavaScript (web-frontend/script.js)
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

### Dockerfile für Log-Collector
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

### Dockerfile für Web-Frontend
```dockerfile
FROM nginx:alpine

# HTML und JavaScript kopieren
COPY index.html /usr/share/nginx/html/
COPY script.js /usr/share/nginx/html/

# Logs-Volume mounten (für CSV-Zugriff)
VOLUME /logs

# Nginx läuft standardmäßig
```

## Befehle zum Ausführen

```bash
# Volume erstellen
docker volume create monitoring-logs

# Log-Collector starten
docker run -d \
  --name log-collector \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -v monitoring-logs:/logs \
  log-collector-image

# Web-Frontend starten
docker run -d \
  --name web-frontend \
  -p 8080:80 \
  -v monitoring-logs:/logs \
  web-frontend-image
```

## Herausforderungen

1. Baue die Docker-Images
2. Starte beide Container
3. Öffne http://localhost:8080
4. Beobachte Echtzeit-Container-Statistiken

## Lernziele
- Docker Volumes im Praxiseinsatz
- Container-übergreifende Datenfreigabe
- Bash-Skripte für Systemmonitoring
- Einfache Webvisualisierung
