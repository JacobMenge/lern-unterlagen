# **API-Design mit Postman**

## **Einführung**
APIs (Application Programming Interfaces) ermöglichen die Kommunikation zwischen verschiedenen Systemen und sind die Grundlage moderner Softwareentwicklung. 

---

## **Teil 1: Grundlagen von APIs**

### **Was ist eine API?**
Stell dir vor, du bestellst in einem Restaurant Essen. Du sagst dem Kellner, was du möchtest, und dieser bringt dein Essen aus der Küche zu dir. Du musst nicht wissen, wie das Essen zubereitet wird – du gibst einfach deine Bestellung auf und erhältst das gewünschte Gericht. Genau so funktioniert eine API.

📌 **Definition:** Eine API ist eine Schnittstelle, über die zwei Systeme miteinander kommunizieren können. Sie ermöglicht es, Daten anzufordern, zu senden und zu verarbeiten.

📌 Warum sind APIs wichtig?
- Sie verbinden unterschiedliche Anwendungen miteinander.
- Sie automatisieren Prozesse und reduzieren den manuellen Aufwand.
- Sie ermöglichen eine flexible und skalierbare Softwareentwicklung.

### **Anwendungsfälle für APIs**
APIs sind überall in unserem digitalen Alltag zu finden. Hier einige Beispiele:

- **Zahlungs-APIs:** 
Wenn du in einem Online-Shop mit PayPal oder Klarna bezahlst, kommuniziert der Shop mit der API des Zahlungsdienstleisters.
- **Wetter-APIs:**
Apps wie Wettervorhersagen greifen auf APIs zu, um aktuelle Daten von Wetterdiensten abzurufen.
- **Social Media APIs:** 
Instagram oder Twitter-APIs erlauben es anderen Apps, Inhalte zu teilen oder auf Nutzerinformationen zuzugreifen.
- **Kartendienste:** 
Dienste wie Google Maps bieten APIs an, damit Entwickler Kartendaten in ihre Anwendungen integrieren können.

### **Arten von APIs**
Es gibt verschiedene API-Typen, die sich in ihrer Funktionsweise und Struktur unterscheiden:

- **REST** (Representational State Transfer) → Die häufigste Art von Web-APIs, nutzt standardisierte HTTP-Methoden.
- **SOAP** (Simple Object Access Protocol) → Älter und komplexer als REST, basiert auf XML.
- **GraphQL** → Flexibler als REST, ermöglicht es, genau die Daten abzufragen, die benötigt werden.

### **HTTP-Methoden & CRUD**
APIs nutzen bestimmte Methoden, um mit Daten zu arbeiten. Diese Methoden folgen dem CRUD-Prinzip:
| **Methode**  | **Bedeutung**  | **Beispiel**  |
|-------------|--------------|----------------|
| **GET**     | Daten abrufen | `GET /users` |
| **POST**    | Neue Daten erstellen | `POST /users` |
| **PUT**     | Daten aktualisieren | `PUT /users/1` |
| **DELETE**  | Daten löschen | `DELETE /users/1` |

### **HTTP-Statuscodes**
Jede API-Antwort enthält einen Statuscode, der beschreibt, ob die Anfrage erfolgreich war oder nicht.
- **200 OK** – Anfrage erfolgreich
- **201 Created** – Neues Objekt erstellt
- **400 Bad Request** – Fehlerhafte Anfrage
- **401 Unauthorized** – Keine Berechtigung
- **404 Not Found** – Ressource nicht gefunden
- **500 Internal Server Error** – Serverproblem


---

## **Teil 2: API-Design – Best Practices**

### **Was macht eine gute API aus?**
✅ **Klar & konsistent** – Einheitliche Namensgebung und Struktur  
✅ **Einfach & intuitiv** – Verständliche Endpunkte  
✅ **Gut dokumentiert** – Klare API-Dokumentation  
✅ **Erweiterbar & stabil** – Zukünftige Änderungen bedenken  

### **RESTful API-Design Prinzipien**
- **Ressourcen-orientiert:** Jede Entität (z. B. User, Produkt) ist eine eigene Ressource.
- **Stateless:** Jede Anfrage enthält alle benötigten Informationen, keine Sessions.
- **Klare Endpunkte:** `/users`, `/orders`, `/products`
- **Richtige HTTP-Methoden nutzen** (GET, POST, PUT, DELETE)

### **API-Routing – Beispiel für Endpunkte**
| **Endpunkt**          | **Methode** | **Aktion** |
|---------------------|-----------|----------|
| `/products`         | GET       | Alle Produkte abrufen |
| `/products/:id`     | GET       | Ein bestimmtes Produkt abrufen |
| `/products`         | POST      | Ein neues Produkt erstellen |
| `/products/:id`     | PUT       | Ein Produkt aktualisieren |
| `/products/:id`     | DELETE    | Ein Produkt löschen |

### **API-Versionierung – Warum und Wie?**

APIs entwickeln sich mit der Zeit weiter. Neue Funktionen werden hinzugefügt, Fehler werden behoben und alte Features werden möglicherweise entfernt. Um sicherzustellen, dass bestehende Anwendungen weiterhin mit der API funktionieren, ohne dass Änderungen sie unerwartet brechen, wird **API-Versionierung** eingesetzt.

---

### **Warum ist API-Versionierung wichtig?**
- 🔄 **Vermeidung von Breaking Changes**: Neue Funktionen können hinzugefügt werden, ohne bestehende Clients zu stören.
- ⚙️ **Flexibilität**: Entwickler können verschiedene Versionen einer API gleichzeitig betreiben.
- 🛠 **Langfristige Wartbarkeit**: Unterstützt eine schrittweise Migration auf neuere API-Versionen.

---

### **Arten der API-Versionierung**

Es gibt verschiedene Methoden, eine API-Version zu definieren:

#### **1. URL-basierte Versionierung (empfohlen für große Änderungen)**
Die API-Version wird direkt in der URL angegeben.

**Beispiel:**
```
https://api.example.com/v1/products
https://api.example.com/v2/products
```
📌 **Vorteile:** Einfach zu implementieren, gut sichtbar für Clients.  
📌 **Nachteile:** Erfordert neue Routen für jede API-Version.

---

#### **2. Header-basierte Versionierung**
Die API-Version wird über HTTP-Header übermittelt.

**Beispiel:**
```http
GET /products HTTP/1.1
Host: api.example.com
Accept: application/vnd.example.v2+json
```
📌 **Vorteile:** URL bleibt sauber, verschiedene Versionen können parallel existieren.  
📌 **Nachteile:** Erfordert zusätzliche Header-Konfiguration, weniger transparent für Nutzer.

---

#### **3. Parameter-basierte Versionierung**
Die Version wird als Query-Parameter in der URL übergeben.

**Beispiel:**
```
https://api.example.com/products?version=2
```
📌 **Vorteile:** Einfach umzusetzen, keine Änderungen an Endpunkten erforderlich.  
📌 **Nachteile:** Nicht standardisiert und potenziell unsicher.

---

### **Best Practices für API-Versionierung**
✅ Nutze **URL-basierte Versionierung** für größere Updates.  
✅ Verwende **Header-basierte Versionierung**, wenn APIs von vielen verschiedenen Clients genutzt werden.  
✅ Dokumentiere **klar**, welche Versionen aktuell unterstützt werden und wann ältere Versionen abgeschaltet werden.  
✅ Biete eine **Übergangsphase** an, wenn eine neue Version eingeführt wird.  

---


### **API-Dokumentation**

#### **Warum ist eine API-Dokumentation wichtig?**
Eine gute API-Dokumentation ist essenziell, um Entwicklern das Arbeiten mit einer API zu erleichtern. Ohne eine klare Dokumentation müssen Entwickler erraten, wie sie die API nutzen können, was zu Fehlern, Frustration und erhöhtem Support-Aufwand führt.

✅ **Verständliche API-Dokumentation hilft dabei:**
- Den Zweck und die Funktion der API schnell zu erfassen.
- Die verfügbaren Endpunkte und Methoden zu verstehen.
- Parameter, Header und Antwortformate richtig zu nutzen.
- Typische Fehler zu vermeiden und effizient zu debuggen.

---

### **Bestandteile einer guten API-Dokumentation**
Eine vollständige API-Dokumentation sollte folgende Elemente enthalten:

1. **Übersicht & Einführung**  
   - Kurze Erklärung, was die API macht und wofür sie genutzt werden kann.
   - Beispielanwendungen oder Anwendungsfälle.

2. **Authentifizierung & Autorisierung**  
   - Welche Methoden zur Authentifizierung gibt es? (z. B. API-Key, OAuth, JWT)
   - Beispiel für einen Authentifizierungs-Request:
   
   ```http
   GET /user/profile HTTP/1.1
   Host: api.example.com
   Authorization: Bearer <token>
   ```

3. **Verfügbare Endpunkte & HTTP-Methoden**  
   - Eine Liste aller Endpunkte mit Beschreibung, z. B.:

   | Methode | Endpunkt        | Beschreibung                    |
   |---------|----------------|--------------------------------|
   | **GET** | `/users`        | Alle Benutzer abrufen          |
   | **POST**| `/users`        | Einen neuen Benutzer erstellen |
   | **PUT** | `/users/{id}`   | Benutzer aktualisieren         |
   | **DELETE** | `/users/{id}` | Benutzer löschen               |

4. **Beispielanfragen und -antworten**  
   - Zeigen, wie eine API-Anfrage aussieht und welche Antwort zurückkommt:

   **Beispiel für eine GET-Anfrage:**
   ```http
   GET /users/1 HTTP/1.1
   Host: api.example.com
   Accept: application/json
   ```
   
   **Antwort:**
   ```json
   {
     "id": 1,
     "name": "John Doe",
     "email": "john.doe@example.com"
   }
   ```

5. **Fehlermeldungen & Statuscodes**  
   - Übersicht über mögliche Fehlercodes und deren Bedeutung:

   | Statuscode | Bedeutung               | Beschreibung                       |
   |-----------|-----------------------|----------------------------------|
   | **200 OK** | Erfolg                 | Anfrage wurde erfolgreich verarbeitet |
   | **400 Bad Request** | Ungültige Anfrage | Der Client hat fehlerhafte Daten gesendet |
   | **401 Unauthorized** | Keine Berechtigung | Fehlende oder falsche Authentifizierung |
   | **404 Not Found** | Ressource nicht gefunden | Die angeforderte Ressource existiert nicht |
   | **500 Internal Server Error** | Serverfehler | Ein unerwarteter Fehler ist aufgetreten |

6. **Rate Limits & Nutzungsbeschränkungen**  
   - Gibt es Begrenzungen für API-Anfragen pro Minute/Stunde?
   - Beispiel: **Maximal 100 Anfragen pro Minute pro API-Key.**

7. **Changelog & Versionierung**  
   - Notizen zu neuen Versionen und Änderungen an der API.
   - Beispiel:
   ```
   v2.1 (15.02.2025) - Neuer Endpunkt `/orders` hinzugefügt.
   v2.0 (10.01.2025) - API-Version 2 veröffentlicht, v1 bleibt bis 31.12.2025 aktiv.
   ```

---

### **Tools zur Automatisierung der API-Dokumentation**
Viele Unternehmen nutzen spezielle Tools zur Generierung der API-Dokumentation. Die bekanntesten sind:

- **Swagger / OpenAPI** – Automatische Generierung von API-Dokumentationen.
- **Postman** – Erlaubt das Dokumentieren von API-Requests und -Tests.
- **Redoc** – Stilvolle OpenAPI-Dokumentation.

📌 **Beispiel einer Swagger-Dokumentation:**
```yaml
openapi: 3.0.0
title: Beispiel API
version: 1.0.0
paths:
  /users:
    get:
      summary: Alle Benutzer abrufen
      responses:
        '200':
          description: Erfolgreiche Antwort
          content:
            application/json:
              schema:
                type: array
                items:
                  type: object
                  properties:
                    id:
                      type: integer
                    name:
                      type: string
                    email:
                      type: string
```

---
