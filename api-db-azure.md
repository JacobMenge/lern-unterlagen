## Anleitung zur Aufgabe: API mit Datenbank erstellen und in Azure bereitstellen

Folge diesen Schritten genau, um eine RESTful API mit Node.js und Express zu erstellen, eine SQLite-Datenbank zu integrieren, die API zu dokumentieren und auf der **Azure Sandbox von Techstarter** bereitzustellen.

---

### 1. Vorbereitung & Wiederholung
- Erstelle zunächst ein neues Projektverzeichnis für deine API und initialisiere es:
  ```sh
  mkdir meine-api && cd meine-api
  npm init -y
  npm install express
  ```
- Erstelle eine einfache `server.js`:
  ```javascript
  const express = require('express');
  const app = express();
  app.use(express.json());

  app.get('/', (req, res) => res.send('Hallo API!'));

  const PORT = process.env.PORT || 3000;
  app.listen(PORT, () => console.log(`Server läuft auf Port ${PORT}`));
  ```

---

### 2. Datenbank (SQLite) einbinden
- Installiere SQLite:
  ```sh
  npm install sqlite3
  ```
- Erstelle eine Datenbank (`database.db`) und eine Tabelle:
  ```javascript
  const sqlite3 = require('sqlite3').verbose();
  const db = new sqlite3.Database('./database.db');

  db.serialize(() => {
    db.run("CREATE TABLE IF NOT EXISTS users (id INTEGER PRIMARY KEY, name TEXT)");
  });
  ```
- Implementiere CRUD-Endpunkte:
  ```javascript
  // User hinzufügen
  app.post('/users', (req, res) => {
    const { name } = req.body;
    db.run("INSERT INTO users (name) VALUES (?)", [name], function(err) {
      if (err) return res.status(500).send(err);
      res.status(201).send({ id: this.lastID });
    });
  });

  // Alle User abrufen
  app.get('/users', (req, res) => {
    db.all("SELECT * FROM users", [], (err, rows) => {
      if (err) return res.status(500).send(err);
      res.json(rows);
    });
  });

  // User löschen
  app.delete('/users/:id', (req, res) => {
    const id = req.params.id;
    db.run("DELETE FROM users WHERE id=?", id, (err) => {
      if (err) return res.status(500).send(err);
      res.status(204).send();
    });
  });
  ```

---

### 3. ZIP-Deployment in Azure App Service

- **ZIP-Tool installieren (falls nicht vorhanden)**:

  **Windows:** ZIP ist standardmäßig vorhanden. Alternativ kannst du 7-Zip nutzen:
  [7-Zip Download](https://www.7-zip.org/)

  **Linux (Debian/Ubuntu):**
  ```sh
  sudo apt install zip
  ```

  **macOS (mit Homebrew):**
  ```sh
  brew install zip
  ```

- Erstelle ein ZIP-Archiv deiner API (ohne node_modules):
  ```sh
  zip -r api.zip . -x "node_modules/*"
  ```
- Melde dich in der Azure Cloud-Shell an:

- Erstelle eine neue Azure App Service Instanz (falls nicht bereits vorhanden):
  ```sh
  az webapp up --name meine-api-app --resource-group MeineAPI-Gruppe --runtime "NODE|18-lts" --sku F1
  ```
- Führe das ZIP-Deployment aus:
  ```sh
  az webapp deployment source config-zip --resource-group MeineAPI-Gruppe --name meine-api-app --src api.zip
  ```
- Überprüfe, ob das Deployment erfolgreich war:
  ```sh
  az webapp show --name meine-api-app --resource-group MeineAPI-Gruppe --query "defaultHostName"
  ```

---

### 4. API-Dokumentation in Markdown erstellen
Erstelle eine Markdown-Datei (`API_Dokumentation.md`) und dokumentiere:
- **Endpunkte** (z.B. `/users`)
- **HTTP-Methoden** (GET, POST, DELETE)
- **Request- und Response-Beispiele**

Beispiel:
```markdown
### GET `/users`
Antwort:
```json
[
  {"id": 1, "name": "Max"}
]
```

### POST `/users`
Request:
```json
{"name": "Anna"}
```
Antwort:
```json
{"id": 2}
```

---

### 5. API ausführlich testen
Nutze Postman oder Curl, um verschiedene API-Requests zu testen:
- GET, POST und DELETE auf die Endpunkte testen und die Ergebnisse dokumentieren.

---

### Zusatzaufgabe
Implementiere zusätzlich eine API-Versionierung (`/v1/users`) und/oder erweitere deine API um JWT-basierte Authentifizierung:
- JWT installieren:
  ```sh
  npm install jsonwebtoken
  ```
- JWT in API integrieren und dokumentieren.
