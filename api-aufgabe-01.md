### **Aufgabe: API erstellen und in Azure bereitstellen**

#### **Ziel**
Erstelle eine eigene RESTful API mit Node.js und Express, die grundlegende CRUD-Funktionalitäten bietet. Die API soll mit der **Azure Sandbox von Techstarter** unter [https://sandboxes.techstarter.de/](https://sandboxes.techstarter.de/) bereitgestellt werden.

### **Aufgabenstellung**
1. **Entwickle eine API** mit mindestens drei Endpunkten (`GET`, `POST`, `DELETE`).
2. **Teste die API mit Postman** und führe mehrere API-Requests durch.
3. **Stelle die API als Azure App Service bereit**

---

# Bereitstellung einer RESTful API in Azure

## **Welche Methode ist die beste?**
| Methode                | Einfachheit | Kosten | Skalierbarkeit | Wartung |
|------------------------|------------|--------|---------------|---------|
| **Azure App Service**  | ⭐⭐⭐⭐⭐     | 💲💲    | ⭐⭐⭐⭐         | ⭐⭐⭐⭐    |
| **Azure Functions**    | ⭐⭐⭐⭐      | 💲      | ⭐⭐⭐⭐⭐        | ⭐⭐⭐⭐⭐   |
| **Azure VM**          | ⭐         | 💲💲💲  | ⭐⭐⭐⭐⭐        | ⭐       |

### **Empfehlung:**
- **Einsteiger / Einfach & schnell:** **Azure App Service**
- **Kosten sparen & event-driven:** **Azure Functions**
- **Maximale Kontrolle & Power:** **Azure Virtual Machine (VM)**

Falls du nicht weißt, wo du anfangen sollst: **Azure App Service ist die beste Wahl für eine einfache REST API.** 🚀

## 1. Azure App Service (Empfohlen für Einfachheit)
### Schritte zur Bereitstellung:
- Erstelle eine Azure App Service Instanz
  - Gehe zum Azure Portal in der Techstarter-Sandbox ([portal.azure.com](https://sandboxes.techstarter.de/)).
  - Erstelle einen neuen App Service.
    ![image](https://github.com/user-attachments/assets/4047c8ec-c36c-4462-915c-22efedb302e3)
  - Wähle die Optionen wie in dem Bild gezeigt:
    ![image](https://github.com/user-attachments/assets/498fd471-0d7f-4fee-90c2-ff093401fd1f)

2. **Code vorbereiten (Falls noch nicht geschehen)**
   - Erstelle eine `server.js` Datei:
     ```javascript
     const express = require("express");
     const app = express();
     const PORT = process.env.PORT || 3000;

     app.use(express.json());

     let items = [{ id: 1, name: "Item 1" }];

     app.get("/api/items", (req, res) => res.json(items));
     app.post("/api/items", (req, res) => {
       const newItem = { id: items.length + 1, name: req.body.name };
       items.push(newItem);
       res.status(201).json(newItem);
     });
     app.delete("/api/items/:id", (req, res) => {
       items = items.filter((item) => item.id !== parseInt(req.params.id));
       res.status(204).send();
     });

     app.listen(PORT, () => console.log(`Server läuft auf Port ${PORT}`));
     ```

   - Füge eine `package.json` Datei hinzu:
     ```json
     {
       "name": "my-api",
       "version": "1.0.0",
       "scripts": {
         "start": "node server.js"
       },
       "dependencies": {
         "express": "^4.18.2"
       }
     }
     ```

3. **Code in GitHub hochladen** (Falls noch nicht geschehen)
   ```sh
   git init
   git add .
   git commit -m "Deploy to Azure"
   git branch -M main
   git remote add origin <Dein-GitHub-Repo-URL>
   git push -u origin main
   ```

4. **SCM aktivieren**
![image](https://github.com/user-attachments/assets/a19d4803-feb7-403c-baea-53b576903190)
- Anschließend speichern!

5. **Azure mit GitHub verknüpfen und Bereitstellung starten**
   - Gehe in **Azure App Service** → "Bereitstellungscenter"
   - Wähle **GitHub** als Quelle
   - Wähle dein Repository & Branch (z. B. `main`)
   - Klicke auf "Bereitstellen"

    ![image](https://github.com/user-attachments/assets/9d6620e8-7c74-4fad-990c-91df3c1162d4)
    - Anschließend speichern!


6. **App Service starten und testen**
   - Die URL zum testen findet ihr unter Übersicht:
   - ![image](https://github.com/user-attachments/assets/c3568c29-3998-459b-b745-592f8b9e1807)

   ```sh
   curl -X GET https://dein-app-service.azurewebsites.net/api/items
   ```

---

## 2. Azure Functions (Serverless, Pay-per-Use)
(Kommt noch)
---

## 3. Azure Virtual Machines (VMs mit Node.js installieren)
(Kommt noch)
---




➡ **Ergebnis:** Deine API soll öffentlich über die Techstarter Azure Sandbox gehostet sein und in Postman getestet oder curl werden können.
---

## **Zusatzaufgabe: API-Dokumentation und Deployment in Azure mit der Techstarter Sandbox**

### **Warum ist eine API-Dokumentation wichtig?**
Eine gute API-Dokumentation ist essenziell, um Entwicklern das Arbeiten mit einer API zu erleichtern. Ohne eine klare Dokumentation müssen Entwickler erraten, wie sie die API nutzen können, was zu Fehlern, Frustration und erhöhtem Support-Aufwand führt.

✅ **Verständliche API-Dokumentation hilft dabei:**
- Den Zweck und die Funktion der API schnell zu erfassen.
- Die verfügbaren Endpunkte und Methoden zu verstehen.
- Parameter, Header und Antwortformate richtig zu nutzen.
- Typische Fehler zu vermeiden und effizient zu debuggen.

---
