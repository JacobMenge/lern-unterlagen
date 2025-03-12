# Docker Datenmanagement Challenge: Notiz-Anwendung (Noch nicht fertig)

## Aufgabe

Erstelle eine Docker-Umgebung für eine einfache Notiz-Anwendung mit MongoDB. 

## Vorbereitung - Code
```
~/notiz-app-challenge/
└── app/
    ├── package.json        # Node.js Abhängigkeiten (express, mongodb, body-parser)
    ├── server.js           # Express.js Backend-Server (API-Endpunkte)
    └── public/             # Frontend-Dateien
        └── index.html      # HTML/CSS/JavaScript für die Benutzeroberfläche

Docker-Komponenten:
- MongoDB Container (Name: mongodb)
- MongoDB Volume (benanntes Volume für Datenpersistenz)
- Node.js Container (Bind Mount für den Code)
```

![image](https://github.com/user-attachments/assets/0bc4a734-1bf1-4209-ab05-1c5c600396f5)


Führe folgende Befehle aus, um die Anwendung vorzubereiten:

```bash
# Erstelle ein Projektverzeichnis
mkdir -p ~/notiz-app-challenge
cd ~/notiz-app-challenge

# Erstelle ein Unterverzeichnis für die Anwendung
mkdir -p app

# Erstelle die package.json
echo '{
  "name": "notiz-app",
  "version": "1.0.0",
  "description": "Einfache Notiz-App mit Docker-Volumes",
  "main": "server.js",
  "dependencies": {
    "express": "^4.18.2",
    "mongodb": "^4.12.0",
    "body-parser": "^1.20.1"
  }
}' > app/package.json

# Erstelle den Server-Code
echo 'const express = require("express");
const bodyParser = require("body-parser");
const { MongoClient } = require("mongodb");

const app = express();
const PORT = 3000;
const mongoUrl = "mongodb://mongodb:27017";
const dbName = "notizDB";

app.use(bodyParser.json());
app.use(bodyParser.urlencoded({ extended: true }));
app.use(express.static("public"));

// Notizen abrufen
app.get("/api/notizen", async (req, res) => {
  try {
    const client = await MongoClient.connect(mongoUrl);
    const db = client.db(dbName);
    const notizen = await db.collection("notizen").find({}).toArray();
    await client.close();
    res.json(notizen);
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
});

// Notiz hinzufügen
app.post("/api/notizen", async (req, res) => {
  try {
    const notiz = {
      titel: req.body.titel,
      inhalt: req.body.inhalt,
      erstellt: new Date()
    };
    
    const client = await MongoClient.connect(mongoUrl);
    const db = client.db(dbName);
    await db.collection("notizen").insertOne(notiz);
    await client.close();
    
    res.status(201).json(notiz);
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
});

app.listen(PORT, () => {
  console.log(`Server läuft auf http://localhost:${PORT}`);
});' > app/server.js

# Erstelle das HTML-Frontend
mkdir -p app/public
echo '<!DOCTYPE html>
<html lang="de">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Notiz-App</title>
  <style>
    body {
      font-family: Arial, sans-serif;
      max-width: 800px;
      margin: 0 auto;
      padding: 20px;
    }
    h1 {
      color: #0066cc;
    }
    form {
      background: #f5f5f5;
      padding: 15px;
      border-radius: 5px;
      margin-bottom: 20px;
    }
    label {
      display: block;
      margin-bottom: 5px;
    }
    input, textarea {
      width: 100%;
      padding: 8px;
      margin-bottom: 10px;
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
    .notiz {
      background: white;
      border: 1px solid #ddd;
      padding: 15px;
      margin-bottom: 15px;
      border-radius: 4px;
    }
    .datum {
      color: #777;
      font-size: 0.8em;
    }
  </style>
</head>
<body>
  <h1>Meine Notizen</h1>
  
  <form id="notiz-form">
    <div>
      <label for="titel">Titel:</label>
      <input type="text" id="titel" required>
    </div>
    <div>
      <label for="inhalt">Inhalt:</label>
      <textarea id="inhalt" rows="4" required></textarea>
    </div>
    <button type="submit">Notiz speichern</button>
  </form>
  
  <div id="notizen-container"></div>

  <script>
    // Notizen laden
    async function notizLaden() {
      try {
        const response = await fetch("/api/notizen");
        const notizen = await response.json();
        
        const container = document.getElementById("notizen-container");
        container.innerHTML = "";
        
        if (notizen.length === 0) {
          container.innerHTML = "<p>Keine Notizen vorhanden.</p>";
          return;
        }
        
        notizen.reverse().forEach(notiz => {
          const notizElement = document.createElement("div");
          notizElement.className = "notiz";
          
          const datum = new Date(notiz.erstellt).toLocaleString();
          
          notizElement.innerHTML = `
            <h3>${notiz.titel}</h3>
            <div class="datum">Erstellt am: ${datum}</div>
            <p>${notiz.inhalt}</p>
          `;
          
          container.appendChild(notizElement);
        });
      } catch (error) {
        console.error("Fehler beim Laden der Notizen:", error);
      }
    }
    
    // Beim Laden der Seite Notizen anzeigen
    document.addEventListener("DOMContentLoaded", notizLaden);
    
    // Notiz-Formular absenden
    document.getElementById("notiz-form").addEventListener("submit", async (e) => {
      e.preventDefault();
      
      const titel = document.getElementById("titel").value;
      const inhalt = document.getElementById("inhalt").value;
      
      try {
        const response = await fetch("/api/notizen", {
          method: "POST",
          headers: {
            "Content-Type": "application/json"
          },
          body: JSON.stringify({ titel, inhalt })
        });
        
        if (response.ok) {
          document.getElementById("titel").value = "";
          document.getElementById("inhalt").value = "";
          notizLaden();
        }
      } catch (error) {
        console.error("Fehler beim Speichern der Notiz:", error);
      }
    });
  </script>
</body>
</html>' > app/public/index.html
```

## Deine Aufgabe

Verwende Docker-Befehle, um eine Umgebung für diese Notiz-Anwendung zu erstellen:

### Teil 1: MongoDB mit benanntem Volume

1. Erstelle ein benanntes Volume für die MongoDB-Daten
2. Starte einen MongoDB-Container, der dieses Volume verwendet
3. Prüfe, ob das Volume erstellt wurde

### Teil 2: Notiz-Anwendung mit Bind Mount

1. Starte einen Node.js-Container, der:
   - Den Anwendungscode aus dem Verzeichnis `~/notiz-app-challenge/app` als Bind Mount einbindet
   - Mit dem MongoDB-Container kommunizieren kann
   - Die Abhängigkeiten installiert und den Server startet
   - Port 3000 nach außen freigibt

### Teil 3: Test der Persistenz

1. Öffne die Anwendung im Browser (http://localhost:3000)
2. Erstelle mehrere Notizen
3. Stoppe und lösche die Container
4. Starte die Container neu
5. Überprüfe, ob die Notizen noch vorhanden sind

### Teil 4: Bearbeite den Code (Live-Update)

1. Ändere während die Container laufen den Titel in der HTML-Datei (app/public/index.html)
2. Prüfe im Browser, ob die Änderung ohne Neustart des Containers übernommen wurde

## Hinweise

- Verwende für MongoDB das Image `mongo:latest`
- Verwende für Node.js das Image `node:16`
- Achte auf die richtigen Volume-Pfade:
  - MongoDB speichert Daten in `/data/db`
  - Der Anwendungscode soll in `/app` im Container gemountet werden
- Setze für den Node.js-Container `/app` als Arbeitsverzeichnis mit `-w /app`
- Der MongoDB-Container sollte unter dem Namen `mongodb` laufen
