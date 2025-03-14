# Docker Datenmanagement: Praktische Übung

In dieser praktischen Übung wirst du verschiedene Methoden zur Datenspeicherung in Docker kennenlernen und anwenden. Du wirst sowohl mit Volumes als auch mit Bind Mounts arbeiten und deren Unterschiede in der Praxis erleben.

## Voraussetzungen

- Docker ist installiert und läuft
- Grundlegende Kenntnisse von Docker-Befehlen
- Ein Terminal/Kommandozeile

## Übung 1: Datenbank mit persistentem Volume

In dieser Übung erstellen wir eine MongoDB-Datenbank, deren Daten in einem Docker Volume gespeichert werden.

### Schritt 1: Erstelle ein benanntes Volume

```bash
docker volume create mongodb-data
```

Dieser Befehl erstellt ein benanntes Volume namens "mongodb-data", das von Docker verwaltet wird.

### Schritt 2: Starte einen MongoDB-Container mit diesem Volume

```bash
docker run -d \
  --name mongodb \
  -v mongodb-data:/data/db \
  -p 27017:27017 \
  mongo:latest
```

**Erklärung:**
- `-d`: Startet den Container im Hintergrund
- `--name mongodb`: Gibt dem Container den Namen "mongodb"
- `-v mongodb-data:/data/db`: Verbindet das Volume "mongodb-data" mit dem Datenbankverzeichnis im Container
- `-p 27017:27017`: Leitet Port 27017 vom Host zum Container weiter
- `mongo:latest`: Verwendet das aktuelle MongoDB-Image

### Schritt 3: Füge Testdaten hinzu

Verbinde dich mit dem MongoDB-Container und erstelle einige Testdaten:

```bash
docker exec -it mongodb mongosh
```

Du bist jetzt in der MongoDB-Shell. Erstelle eine Datenbank und füge Daten hinzu:

```javascript
// Wechsle zur Datenbank "testdb"
use testdb

// Füge Testdaten hinzu
db.users.insertMany([
  { name: "Max Mustermann", email: "max@example.com", alter: 30 },
  { name: "Erika Musterfrau", email: "erika@example.com", alter: 28 }
])

// Zeige die eingefügten Daten an
db.users.find()

// Beende die MongoDB-Shell
exit
```

### Schritt 4: Container löschen und neu erstellen

Nun werden wir den Container löschen und neu erstellen, um zu sehen, ob die Daten erhalten bleiben:

```bash
# Container stoppen und löschen
docker stop mongodb
docker rm mongodb

# Container neu erstellen mit demselben Volume
docker run -d \
  --name mongodb-new \
  -v mongodb-data:/data/db \
  -p 27017:27017 \
  mongo:latest
```

### Schritt 5: Überprüfen, ob die Daten erhalten geblieben sind

```bash
# Mit dem neuen Container verbinden
docker exec -it mongodb-new mongosh

# In der MongoDB-Shell
use testdb
db.users.find()
exit
```

Du solltest die beiden Benutzer sehen, die du vorher eingefügt hast. Dies zeigt, dass die Daten im Volume "mongodb-data" erhalten geblieben sind, obwohl der ursprüngliche Container gelöscht wurde.

## Übung 2: Webserver mit Bind Mount für die Entwicklung

In dieser Übung erstellen wir einen Nginx-Webserver, dessen Inhalt über einen Bind Mount bereitgestellt wird.

### Schritt 1: Erstelle ein Projektverzeichnis

```bash
mkdir -p ~/nginx-project/html
cd ~/nginx-project
```

### Schritt 2: Erstelle eine HTML-Datei

```bash
echo '<!DOCTYPE html>
<html>
<head>
    <title>Docker Bind Mount Demo</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            margin: 40px;
            line-height: 1.6;
            color: #333;
        }
        h1 {
            color: #0066cc;
        }
        .container {
            max-width: 800px;
            margin: 0 auto;
            padding: 20px;
            background-color: #f9f9f9;
            border-radius: 5px;
            box-shadow: 0 2px 4px rgba(0,0,0,0.1);
        }
    </style>
</head>
<body>
    <div class="container">
        <h1>Hallo von Docker mit Bind Mount!</h1>
        <p>Diese Seite wird über einen Bind Mount von deinem Host-System in den Container geliefert.</p>
        <p>Bearbeite die Datei auf deinem Host und aktualisiere den Browser, um die Änderungen zu sehen.</p>
    </div>
</body>
</html>' > html/index.html
```

### Schritt 3: Starte einen Nginx-Container mit Bind Mount

```bash
docker run -d \
  --name nginx-server \
  -v $(pwd)/html:/usr/share/nginx/html \
  -p 8080:80 \
  nginx:alpine
```

**Erklärung:**
- `-v $(pwd)/html:/usr/share/nginx/html`: Bind Mount, der das lokale html-Verzeichnis mit dem Webserver-Verzeichnis im Container verbindet
- `-p 8080:80`: Leitet Port 8080 des Hosts an Port 80 des Containers weiter
- `nginx:alpine`: Verwendet das leichtgewichtige Alpine-basierte Nginx-Image

### Schritt 4: Überprüfe die Website

Öffne einen Browser und gehe zu http://localhost:8080. Du solltest die HTML-Seite sehen.

### Schritt 5: Ändere den Inhalt und prüfe die Live-Aktualisierung

Bearbeite die Datei `html/index.html` mit einem Texteditor deiner Wahl oder füge eine neue Zeile hinzu:

```bash
# Füge eine neue Zeile zur HTML-Datei hinzu
# Für Linux/macOS:
echo '<p style="color: red; font-weight: bold;">Diese Zeile wurde hinzugefügt, während der Container läuft!</p>' >> html/index.html

# Für Windows (PowerShell):
# Add-Content -Path html/index.html -Value '<p style="color: red; font-weight: bold;">Diese Zeile wurde hinzugefügt, während der Container läuft!</p>'
```

Aktualisiere nun die Browser-Seite (http://localhost:8080). Du solltest die hinzugefügte Zeile sofort sehen, ohne dass der Container neu gestartet werden musste.

## Übung 3: Anonymes Volume und tmpfs Mount

In dieser Übung vergleichen wir anonyme Volumes und tmpfs Mounts.

### Schritt 1: Starte einen Container mit anonymem Volume

```bash
docker run -d \
  --name volume-demo \
  -v /app/data \
  alpine \
  sh -c "while true; do date >> /app/data/dates.txt; sleep 5; done"
```

**Erklärung:**
- `-v /app/data`: Erstellt ein anonymes Volume für /app/data
- Der Container schreibt alle 5 Sekunden das aktuelle Datum in eine Datei

### Schritt 2: Überprüfe den Inhalt des anonymen Volumes

```bash
# Nach 10 Sekunden warten sollten einige Zeitstempel geschrieben sein
sleep 10

# Zeige den Inhalt der Datei im Volume an
docker exec volume-demo cat /app/data/dates.txt
```

### Schritt 3: Starte einen Container mit tmpfs Mount

```bash
docker run -d \
  --name tmpfs-demo \
  --tmpfs /app/data \
  alpine \
  sh -c "while true; do date >> /app/data/dates.txt; sleep 5; done"
```

**Erklärung:**
- `--tmpfs /app/data`: Erstellt einen tmpfs Mount für /app/data (im Arbeitsspeicher)
- Der Container führt den gleichen Befehl aus wie der vorherige

### Schritt 4: Überprüfe den Inhalt des tmpfs Mounts

```bash
# Nach 10 Sekunden warten
sleep 10

# Zeige den Inhalt der Datei im tmpfs Mount an
docker exec tmpfs-demo cat /app/data/dates.txt
```

Beide Container sollten ähnliche Ausgaben zeigen - eine Liste von Zeitstempeln.

### Schritt 5: Neustart der Container und Überprüfung der Persistenz

```bash
# Stoppe und starte den Container mit anonymem Volume neu
docker stop volume-demo
docker start volume-demo
sleep 5
echo "Inhalt des anonymen Volumes nach Neustart:"
docker exec volume-demo cat /app/data/dates.txt

# Stoppe und starte den Container mit tmpfs Mount neu
docker stop tmpfs-demo
docker start tmpfs-demo
sleep 5
echo "Inhalt des tmpfs Mounts nach Neustart:"
docker exec tmpfs-demo cat /app/data/dates.txt || echo "Datei nicht gefunden (Daten verloren)"
```

**Erwartetes Ergebnis:**
- Beim Container mit dem anonymen Volume bleiben die Daten erhalten
- Beim Container mit dem tmpfs Mount gehen die Daten beim Neustart verloren, da tmpfs nur im RAM speichert und nicht persistent ist

## Übung 4: Kombination von Bind Mount und anonymem Volume

Diese Übung zeigt, wie Bind Mounts und anonyme Volumes zusammen verwendet werden können. Wir erstellen eine Express-Anwendung für ein einfaches Notiz-System.

### Schritt 1: Erstelle ein Express-Projekt

```bash
mkdir -p ~/notes-app
cd ~/notes-app

# package.json erstellen
echo '{
  "name": "notes-app",
  "version": "1.0.0",
  "description": "Eine einfache Notiz-App mit Docker-Volumes",
  "main": "server.js",
  "scripts": {
    "start": "node server.js"
  },
  "dependencies": {
    "express": "^4.18.2",
    "body-parser": "^1.20.1"
  }
}' > package.json

# Server-Code erstellen
echo 'const express = require("express");
const bodyParser = require("body-parser");
const fs = require("fs");
const path = require("path");

const app = express();
const PORT = 3000;
const NOTES_FILE = path.join(__dirname, "data", "notes.json");

// Stelle sicher, dass das Datenverzeichnis existiert
const dataDir = path.join(__dirname, "data");
if (!fs.existsSync(dataDir)) {
  fs.mkdirSync(dataDir, { recursive: true });
}

// Wenn die Notiz-Datei nicht existiert, erstelle eine leere Liste
if (!fs.existsSync(NOTES_FILE)) {
  fs.writeFileSync(NOTES_FILE, "[]", "utf8");
}

// Middleware für JSON und URL-encoded Formulardaten
app.use(bodyParser.json());
app.use(bodyParser.urlencoded({ extended: true }));

// Statische Dateien aus dem "public"-Verzeichnis bereitstellen
app.use(express.static(path.join(__dirname, "public")));

// Alle Notizen abrufen
app.get("/api/notes", (req, res) => {
  const notes = JSON.parse(fs.readFileSync(NOTES_FILE, "utf8"));
  res.json(notes);
});

// Eine neue Notiz hinzufügen
app.post("/api/notes", (req, res) => {
  const notes = JSON.parse(fs.readFileSync(NOTES_FILE, "utf8"));
  
  const newNote = {
    id: Date.now(),
    title: req.body.title || "Unbenannt",
    content: req.body.content || "",
    createdAt: new Date().toISOString()
  };
  
  notes.push(newNote);
  fs.writeFileSync(NOTES_FILE, JSON.stringify(notes, null, 2), "utf8");
  
  res.status(201).json(newNote);
});

// Server starten
app.listen(PORT, () => {
  console.log(`Server läuft auf http://localhost:${PORT}`);
});' > server.js

# HTML/CSS/JS für die Frontend-Anwendung erstellen
mkdir -p public
echo '<!DOCTYPE html>
<html lang="de">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Notiz-App mit Docker Volumes</title>
  <style>
    body {
      font-family: Arial, sans-serif;
      line-height: 1.6;
      max-width: 800px;
      margin: 0 auto;
      padding: 20px;
    }
    h1 {
      color: #0066cc;
      border-bottom: 2px solid #eee;
      padding-bottom: 10px;
    }
    form {
      background: #f9f9f9;
      padding: 20px;
      border-radius: 5px;
      margin-bottom: 20px;
    }
    label {
      display: block;
      margin-bottom: 5px;
      font-weight: bold;
    }
    input, textarea {
      width: 100%;
      padding: 8px;
      margin-bottom: 15px;
      border: 1px solid #ddd;
      border-radius: 4px;
    }
    button {
      background: #0066cc;
      color: white;
      border: none;
      padding: 10px 15px;
      border-radius: 4px;
      cursor: pointer;
    }
    button:hover {
      background: #0055aa;
    }
    .note {
      background: white;
      border: 1px solid #ddd;
      border-radius: 4px;
      padding: 15px;
      margin-bottom: 15px;
      box-shadow: 0 2px 4px rgba(0,0,0,0.1);
    }
    .note h3 {
      margin-top: 0;
    }
    .date {
      color: #777;
      font-size: 0.8em;
      margin-bottom: 10px;
    }
    #notes-container {
      margin-top: 30px;
    }
  </style>
</head>
<body>
  <h1>Notiz-App mit Docker Volumes</h1>
  
  <form id="note-form">
    <div>
      <label for="title">Titel:</label>
      <input type="text" id="title" name="title" required>
    </div>
    <div>
      <label for="content">Inhalt:</label>
      <textarea id="content" name="content" rows="4" required></textarea>
    </div>
    <button type="submit">Notiz speichern</button>
  </form>
  
  <h2>Meine Notizen</h2>
  <div id="notes-container"></div>

  <script>
    // Notizen vom Server laden
    async function loadNotes() {
      try {
        const response = await fetch("/api/notes");
        const notes = await response.json();
        
        const container = document.getElementById("notes-container");
        container.innerHTML = "";
        
        if (notes.length === 0) {
          container.innerHTML = "<p>Noch keine Notizen vorhanden.</p>";
          return;
        }
        
        notes.reverse().forEach(note => {
          const noteElement = document.createElement("div");
          noteElement.className = "note";
          
          const date = new Date(note.createdAt).toLocaleString();
          
          noteElement.innerHTML = `
            <h3>${note.title}</h3>
            <div class="date">Erstellt am: ${date}</div>
            <p>${note.content}</p>
          `;
          
          container.appendChild(noteElement);
        });
      } catch (error) {
        console.error("Fehler beim Laden der Notizen:", error);
      }
    }
    
    // Beim Laden der Seite Notizen anzeigen
    document.addEventListener("DOMContentLoaded", loadNotes);
    
    // Notiz-Formular absenden
    document.getElementById("note-form").addEventListener("submit", async (e) => {
      e.preventDefault();
      
      const title = document.getElementById("title").value;
      const content = document.getElementById("content").value;
      
      try {
        const response = await fetch("/api/notes", {
          method: "POST",
          headers: {
            "Content-Type": "application/json"
          },
          body: JSON.stringify({ title, content })
        });
        
        if (response.ok) {
          // Formular zurücksetzen und Notizen neu laden
          document.getElementById("title").value = "";
          document.getElementById("content").value = "";
          loadNotes();
        }
      } catch (error) {
        console.error("Fehler beim Speichern der Notiz:", error);
      }
    });
  </script>
</body>
</html>' > public/index.html
```

### Schritt 2: Starte den Container mit Bind Mount für den Code und anonymem Volume für Daten

```bash
docker run -d \
  --name notes-app \
  -v $(pwd):/app \
  -v /app/node_modules \
  -v /app/data \
  -w /app \
  -p 3000:3000 \
  node:18 \
  sh -c "npm install && node server.js"
```

**Erklärung:**
- `-v $(pwd):/app`: Bind Mount für den Quellcode - erlaubt die Bearbeitung des Codes ohne Neustart
- `-v /app/node_modules`: Anonymes Volume für node_modules - verhindert Konflikte mit lokalen Modulen
- `-v /app/data`: Anonymes Volume für die Notizen-Daten - speichert Daten persistent im Container
- `-w /app`: Setzt das Arbeitsverzeichnis im Container
- `-p 3000:3000`: Leitet Port 3000 weiter
- `node:18`: Verwendet Node.js 18
- Der Container installiert die Abhängigkeiten und startet den Server

### Schritt 3: Teste die Anwendung

Öffne einen Browser und gehe zu http://localhost:3000. Du solltest die Notiz-App sehen:

1. Erstelle ein paar Notizen über das Formular
2. Die Notizen sollten unten auf der Seite erscheinen

### Schritt 4: Modifiziere den Quellcode und beobachte die Änderungen

Ändere den Titel in der Datei `public/index.html`:

```bash
# Für Linux/macOS:
sed -i 's/<h1>Notiz-App mit Docker Volumes<\/h1>/<h1>Meine Notizen-App mit Docker Volumes<\/h1>/' public/index.html

# Für Windows (PowerShell):
# (Get-Content public/index.html) -replace '<h1>Notiz-App mit Docker Volumes</h1>', '<h1>Meine Notizen-App mit Docker Volumes</h1>' | Set-Content public/index.html
```

Aktualisiere die Browser-Seite (http://localhost:3000). Du solltest den geänderten Titel sehen, ohne dass der Container neu gestartet werden musste.

### Schritt 5: Überprüfe die Datenpersistenz

Stoppe und starte den Container neu, um zu sehen, ob die Notizen erhalten bleiben:

```bash
docker stop notes-app
docker start notes-app
```

Öffne http://localhost:3000 erneut. Deine Notizen sollten immer noch vorhanden sein, da sie im anonymen Volume `/app/data` gespeichert werden.

## Aufräumen

Nach Abschluss der Übungen solltest du die erstellten Container und Volumes aufräumen:

```bash
# Container stoppen und entfernen
docker stop mongodb-new nginx-server volume-demo tmpfs-demo notes-app
docker rm mongodb-new nginx-server volume-demo tmpfs-demo notes-app

# Benannte Volumes auflisten
docker volume ls

# Benannte Volumes entfernen (Vorsicht: Daten gehen verloren!)
docker volume rm mongodb-data

# Alternativ: Alle nicht verwendeten Volumes entfernen
# docker volume prune
```

## Zusammenfassung

In dieser praktischen Übung hast du:

1. **Persistente Daten mit benannten Volumes** gespeichert (MongoDB-Datenbank)
2. **Bind Mounts für die Entwicklung** verwendet (Nginx-Webserver)
3. Den Unterschied zwischen **anonymen Volumes und tmpfs Mounts** kennengelernt
4. **Bind Mounts und anonyme Volumes kombiniert** für eine Notiz-Anwendung

Diese Übung hat dir gezeigt, wie du verschiedene Docker-Speichermethoden für unterschiedliche Anwendungsfälle einsetzen kannst. Du hast praktische Erfahrung mit allen wichtigen Datenspeicherkonzepten in Docker gesammelt.
