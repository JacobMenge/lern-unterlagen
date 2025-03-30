# Aufgabenstellung: Microservices und Containerisierung (Nur für Fortgeschrittene)

## Einführung
In dieser Aufgabe werden wir die Flexibilität und Modularität einer Microservice-Architektur demonstrieren. Ihr habt bereits mit einer Anwendung gearbeitet, die aus drei Containern besteht:
- Frontend (React)
- Backend (Node.js)
- Datenbank (MongoDB)

Das Ziel dieser Aufgabe ist es, zu zeigen, wie einfach einzelne Komponenten in einer solchen Architektur ausgetauscht werden können, ohne das Gesamtsystem zu beeinträchtigen.

## Grundaufgabe: Backend-Austausch
Deine Hauptaufgabe besteht darin, das bestehende Node.js-Backend durch ein Python-Backend zu ersetzen, während sowohl das Frontend als auch die Datenbank unverändert bleiben.

### Python-Frameworks für das Backend
Für das neue Python-Backend könnt ihr eines dieser Frameworks verwenden:

#### FastAPI
**Was ist das?** Ein modernes, schnelles Web-Framework für Python, optimiert für APIs.
- Sehr performant (basiert auf Starlette und Pydantic)
- Automatische Dokumentation der API (OpenAPI/Swagger)
- Voll kompatibel mit asynchroner Programmierung
- Einfach zu lernen und zu implementieren

📚 **Dokumentation**: [https://fastapi.tiangolo.com/](https://fastapi.tiangolo.com/)

#### Flask
**Was ist das?** Ein leichtgewichtiges Web-Framework für Python, perfekt für kleine bis mittelgroße Anwendungen.
- Minimalistischer Aufbau, einfach zu verstehen
- Flexible Struktur, leicht erweiterbar
- Große Community und viele Erweiterungen verfügbar
- Gut geeignet für REST-APIs

📚 **Dokumentation**: [https://flask.palletsprojects.com/](https://flask.palletsprojects.com/)

### Anforderungen:
1. Erstelle einen neuen Docker-Container mit einem Python-Backend
   - Verwende FastAPI oder Flask als Framework
   - Implementiere die gleichen API-Endpunkte wie im ursprünglichen Node.js-Backend
   - Stelle sicher, dass die Kommunikation mit der MongoDB-Datenbank funktioniert

2. Aktualisiere die docker-compose.yml-Datei, um das neue Python-Backend zu integrieren

3. Dokumentiere die notwendigen Änderungen und erkläre, warum das Frontend trotz Backend-Wechsel weiterhin funktioniert

## Erweiterte Aufgaben (für noch weiter Fortgeschrittene / viel eigenrecherche!)

### Option 1: Redis-Integration
**Was ist Redis?** Ein In-Memory-Datenspeicher, der als Datenbank, Cache oder Message Broker verwendet werden kann.
- Extrem schnell, da die Daten im Arbeitsspeicher gehalten werden
- Unterstützt verschiedene Datenstrukturen (Strings, Hashes, Listen, Sets, etc.)
- Ideal für Caching, Session-Speicherung und Echtzeitanwendungen
- Verbessert die Anwendungsleistung durch Reduzierung von Datenbankabfragen

📚 **Dokumentation**: [https://redis.io/documentation](https://redis.io/documentation)

Füge einen vierten Container mit Redis hinzu, um:
- Caching für häufig abgefragte Daten zu implementieren
- Die Leistung der Anwendung zu verbessern
- Session-Management zu ermöglichen

### Option 2: PostgreSQL statt MongoDB
**Was ist PostgreSQL?** Ein leistungsstarkes, Open-Source-relationales Datenbanksystem.
- Im Gegensatz zu MongoDB, das dokumentenorientiert ist, speichert PostgreSQL Daten in Tabellen mit festen Schemas
- Unterstützt komplexe SQL-Abfragen und Transaktionen
- Exzellente Datenkonsistenz und Integrität
- Gut geeignet für strukturierte Daten und Beziehungen zwischen Datensätzen

📚 **Dokumentation**: [https://www.postgresql.org/docs/](https://www.postgresql.org/docs/)

Ersetze MongoDB durch PostgreSQL:
- Passe das Python-Backend an, um mit PostgreSQL zu kommunizieren
- Migriere das Datenbankschema und die Daten
- Erkläre die Unterschiede zwischen dokumentenorientierter und relationaler Datenspeicherung

### Option 3: Frontend-Alternativen
#### Vue.js
**Was ist Vue.js?** Ein progressives JavaScript-Framework zum Erstellen von Benutzeroberflächen.
- Einfach zu erlernen, besonders für Anfänger
- Komponentenbasierte Architektur, ähnlich wie React
- Flexibel und integriert sich gut mit anderen Bibliotheken
- Reaktives und komponierbare Datenbindung
- Vue 3 mit Composition API für bessere Code-Organisation und TypeScript-Integration

📚 **Dokumentation**: [https://vuejs.org/guide/introduction.html](https://vuejs.org/guide/introduction.html)

#### Angular
**Was ist Angular?** Ein von Google entwickeltes TypeScript-basiertes Framework für Webanwendungen.
- Vollständiges Frontend-Framework mit integrierten Features (Routing, Forms, HTTP-Client)
- Verwendet TypeScript für statische Typisierung
- Modulare Struktur mit wiederverwendbaren Komponenten
- Zwei-Wege-Datenbindung und Dependency Injection
- Aktuell in Version 17+ mit verbesserter Performance und Entwicklererfahrung

📚 **Dokumentation**: [https://angular.io/docs](https://angular.io/docs)

Erstelle eine alternative Frontend-Version:
- Verwende Vue.js oder Angular statt React
- Stelle sicher, dass es mit dem Python-Backend kommunizieren kann
- Implementiere ein Feature-Flag-System, um zwischen den Frontend-Versionen zu wechseln

## Hilfestellung
Die folgenden Code-Beispiele sind als Startpunkt gedacht. Du kannst und sollst sie an deine Bedürfnisse anpassen! Eigenrecherche und Experimentieren werden ausdrücklich empfohlen.

### Python-Backend mit FastAPI
Hier ist ein einfaches Python-Backend mit FastAPI, das mit einer MongoDB kommuniziert:

```python
from fastapi import FastAPI, HTTPException
from motor.motor_asyncio import AsyncIOMotorClient
from pydantic import BaseModel
import uvicorn

app = FastAPI()

# Datenbankverbindung
client = AsyncIOMotorClient("mongodb://mongodb:27017")
db = client.taskdb
collection = db.tasks

class Task(BaseModel):
    title: str
    description: str = None
    completed: bool = False

@app.get("/")
async def root():
    return {"message": "Task API is running"}

@app.get("/api/tasks")
async def get_tasks():
    tasks = []
    async for task in collection.find():
        task["_id"] = str(task["_id"])
        tasks.append(task)
    return tasks

@app.post("/api/tasks")
async def create_task(task: Task):
    new_task = task.model_dump()  # In neueren Pydantic-Versionen wird .dict() durch .model_dump() ersetzt
    result = await collection.insert_one(new_task)
    new_task["_id"] = str(result.inserted_id)
    return new_task

# Server starten
if __name__ == "__main__":
    uvicorn.run("main:app", host="0.0.0.0", port=5000, reload=True)
```

**Was macht der Code?** 
- Er definiert ein Datenmodell für Aufgaben (Tasks)
- Er stellt zwei Endpunkte bereit: einen zum Abrufen aller Aufgaben und einen zum Erstellen neuer Aufgaben
- Er nutzt MongoDB zur Datenspeicherung mit dem motor-Paket für asynchrone Kommunikation
- Der Server läuft auf Port 5000 und akzeptiert Verbindungen von allen Netzwerkschnittstellen

### Dockerfile für Python-Backend
So könnte ein Dockerfile für das Python-Backend aussehen:

```dockerfile
FROM python:3.9-slim

WORKDIR /app

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Beispiel requirements.txt:
# fastapi>=0.100.0
# uvicorn>=0.22.0
# motor>=3.2.0
# pydantic>=2.0.0

COPY . .

CMD ["python", "main.py"]
```

**Was macht das Dockerfile?**
- Es verwendet ein schlankes Python 3.9 Image als Basis
- Es installiert die benötigten Python-Pakete aus der requirements.txt
- Es kopiert den Code in den Container
- Beim Start des Containers wird die main.py ausgeführt

### Beispiel für docker-compose.yml
Hier ist ein grundlegendes docker-compose.yml für die Container-Orchestrierung:

```yaml
version: '3.8'  # Neuere Version von Docker Compose mit mehr Features

services:
  frontend:
    build: ./frontend
    ports:
      - "3000:3000"
    depends_on:
      - backend
  
  backend:
    build: ./python-backend  # Hier erfolgt der Austausch
    ports:
      - "5000:5000"
    depends_on:
      - mongodb
    environment:
      - MONGO_URI=mongodb://mongodb:27017/taskdb
  
  mongodb:
    image: mongo:latest
    ports:
      - "27017:27017"
    volumes:
      - mongo-data:/data/db

volumes:
  mongo-data:
```

**Was macht diese Konfiguration?**
- Sie definiert drei Dienste: frontend, backend und mongodb
- Der backend-Dienst wird aus dem Verzeichnis ./python-backend gebaut (wo dein neues Python-Backend liegt)
- Die Ports werden von den Containern auf den Host weitergeleitet
- Die Abhängigkeiten zwischen den Diensten werden festgelegt (frontend braucht backend, backend braucht mongodb)
- Ein Volume für die MongoDB-Daten sorgt dafür, dass diese erhalten bleiben, selbst wenn der Container neu gestartet wird

### Python + Redis Beispiel
Wenn du die erweiterte Aufgabe mit Redis angehst, könnte dein Code so aussehen:

```python
import redis
from fastapi import FastAPI, Depends
from fastapi.responses import JSONResponse

app = FastAPI()

# Verbindung zu Redis als Dependency für bessere Testbarkeit und saubereren Code
def get_redis_client():
    return redis.Redis(host='redis', port=6379, db=0, decode_responses=True)  # decode_responses=True für automatische UTF-8 Dekodierung

@app.get("/api/cached-data/{key}")
async def get_cached_data(key: str, redis_client: redis.Redis = Depends(get_redis_client)):
    # Zuerst im Cache nachsehen
    cached_value = redis_client.get(key)
    
    if cached_value:
        return {"source": "cache", "data": cached_value}  # Keine Dekodierung nötig mit decode_responses=True
    
    # Wenn nicht im Cache, aus der Datenbank holen (vereinfacht)
    # In einer echten Anwendung würde hier ein Datenbankzugriff stehen
    data = f"Daten für {key} aus der Datenbank"
    
    # Im Cache speichern für zukünftige Anfragen (60 Sekunden)
    redis_client.setex(key, 60, data)
    
    return {"source": "database", "data": data}
```

**Was macht dieser Code?**
- Er richtet eine Verbindung zu Redis als Cache-Speicher ein
- Er implementiert einen API-Endpunkt, der zuerst im Cache nachsieht, bevor er auf die Datenbank zugreift
- Wenn Daten im Cache gefunden werden, werden sie direkt zurückgegeben (schnell!)
- Wenn nicht, werden die Daten aus der Datenbank geholt und für zukünftige Anfragen im Cache gespeichert

Denk daran: Diese Beispiele sind nur ein Ausgangspunkt! Du sollst sie verstehen, anpassen und erweitern. Scheu dich nicht, die Dokumentation zu lesen und eigene Lösungen zu entwickeln.
