# Docker Node.js Challenge

## Aufgabe

Erstelle einen Docker-Container für eine einfache Node.js-Anwendung, die eine REST-API bereitstellt. Die Anwendung soll Informationen über Bücher speichern und abrufen können.

## Anforderungen

1. Die Anwendung soll auf Node.js basieren
2. Die API soll mindestens folgende Endpunkte haben:
   - GET `/books` - Liste aller Bücher zurückgeben
   - GET `/books/:id` - Details zu einem spezifischen Buch
   - POST `/books` - Neues Buch hinzufügen
3. Die Daten können im Arbeitsspeicher (als JavaScript-Array) gespeichert werden
4. Die Anwendung soll in einem Docker-Container laufen
5. Der Container soll auf Port 3000 erreichbar sein

## Hinweise

- Du musst ein geeignetes Basis-Image für Node.js auswählen
- Ein Express.js-Server eignet sich gut für diese Aufgabe
- Dein Dockerfile sollte die App korrekt containerisieren
- Die API sollte mit einem Tool wie Postman oder curl testbar sein

## Beispiel für die Datenstruktur

```json
[
  {
    "id": 1,
    "title": "Docker für Einsteiger",
    "author": "Jacob Menge",
    "year": 2023
  },
  {
    "id": 2,
    "title": "Node.js und Container",
    "author": "Jacobs Katze",
    "year": 2022
  }
]
```

## Möglicher Projektaufbau

```
node-docker-app/
├── package.json
├── server.js
├── Dockerfile
└── README.md
```

## Bewertungskriterien

- Die Anwendung funktioniert wie erwartet
- Das Dockerfile folgt den Best Practices
- Der Container kann gebaut und gestartet werden
- Die API ist über den Container zugänglich

## Tipps

- Beginne mit der Erstellung der Node.js-Anwendung und teste sie lokal
- Erstelle dann ein Dockerfile, um die Anwendung zu containerisieren
- Überprüfe, ob du alle notwendigen Dateien in den Container kopierst
- Vergiss nicht, den richtigen Port freizugeben und weiterzuleiten
- Für die Entwicklung kann das Einbinden des Quellcode-Verzeichnisses nützlich sein

Viel Erfolg bei der Challenge!
