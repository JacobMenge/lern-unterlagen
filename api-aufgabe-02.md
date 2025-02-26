## **Aufgabe: API mit Datenbank erstellen und in Azure bereitstellen**

#### **Ziel**
Deine Aufgabe ist es, eine RESTful API mit Node.js und Express zu erstellen, diese mit einer einfachen Datenbank zu verbinden, zu dokumentieren und anschließend auf der **Azure Sandbox von Techstarter** bereitzustellen.

#### **Aufgabenstellung**

1. **Wiederhole kurz**, wie du gestern eine API mit Node.js und Express erstellt und deployed hast.

2. **Erweitere deine API um eine Datenbank (SQLite)**:
   - Installiere SQLite:
     ```sh
     npm install sqlite3
     ```
   - Erstelle eine Datenbank-Datei (`database.db`) und integriere diese mit deiner API.
   - Implementiere mindestens drei Endpunkte mit CRUD-Funktionen (`GET`, `POST`, `DELETE`).

3. **Deploye deine aktualisierte API erneut in Azure (Techstarter Sandbox)**:
   - Nutze das gestern gezeigte ZIP-Deployment für Azure App Service.

4. **Erstelle eine API-Dokumentation in Markdown**:
   - Dokumentiere jeden Endpunkt, beschreibe Methoden, Parameter und Rückgaben klar und übersichtlich.
   - https://www.markdowntoolbox.com/de/blog/markdown-api-dokumentationsvorlage/

5. **Teste deine API umfassend mit Postman oder Curl**:
   - Führe verschiedene API-Requests durch (GET, POST, DELETE).

#### **Ergebnis**
- Deine API läuft öffentlich auf Azure App Service in der Techstarter Sandbox.
- Du hast eine klare API-Dokumentation in Markdown erstellt.
- Du kannst deine API mit Postman testen.

---

## **Zusatzaufgabe:**
Implementiere zusätzlich eine API-Versionierung (`/v1/users`) und/oder erweitere deine API um JWT-basierte Authentifizierung.
