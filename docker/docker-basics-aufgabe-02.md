# Praktische Übung: Web-Anwendung mit Docker erstellen und verwalten

## Aufgabe:

Erstelle eine einfache mehrseitige Website und betreibe sie in einem Docker-Container. Du sollst dann auch eine kleine Änderung an der Website vornehmen und den Container verwalten.

## Schritte:

### 1. Projektordner anlegen
Erstelle einen neuen Ordner für dein Projekt und wechsle in diesen:
```bash
mkdir docker-webprojekt
cd docker-webprojekt
```

### 2. Website-Dateien erstellen

a) Erstelle eine `index.html` Datei:
```bash
echo '<!DOCTYPE html>
<html>
<head>
    <title>Meine Docker-Website</title>
    <link rel="stylesheet" href="style.css">
</head>
<body>
    <header>
        <h1>Willkommen zu meiner Docker-Übung</h1>
        <nav>
            <a href="index.html">Startseite</a> |
            <a href="ueber.html">Über Docker</a>
        </nav>
    </header>
    <main>
        <p>Diese Seite wird über einen Docker-Container bereitgestellt.</p>
        <p>Besuche die "Über Docker"-Seite für mehr Informationen.</p>
    </main>
    <footer>
        <p>Docker-Übung 2025</p>
    </footer>
</body>
</html>' > index.html
```

b) Erstelle eine `ueber.html` Datei:
```bash
echo '<!DOCTYPE html>
<html>
<head>
    <title>Über Docker</title>
    <link rel="stylesheet" href="style.css">
</head>
<body>
    <header>
        <h1>Über Docker</h1>
        <nav>
            <a href="index.html">Startseite</a> |
            <a href="ueber.html">Über Docker</a>
        </nav>
    </header>
    <main>
        <h2>Was ist Docker?</h2>
        <p>Docker ist eine Open-Source-Plattform für die Entwicklung, den Versand und die Ausführung von Anwendungen.</p>
        <p>Docker ermöglicht es, Anwendungen von der Infrastruktur zu trennen, sodass Software schnell bereitgestellt werden kann.</p>
    </main>
    <footer>
        <p>Docker-Übung 2025</p>
    </footer>
</body>
</html>' > ueber.html
```

c) Erstelle eine `style.css` Datei:
```bash
echo 'body {
    font-family: Arial, sans-serif;
    margin: 0;
    padding: 20px;
    line-height: 1.6;
    color: #333;
}

header {
    background-color: #f4f4f4;
    padding: 10px;
    margin-bottom: 20px;
    border-radius: 5px;
}

nav {
    margin-top: 10px;
}

main {
    padding: 10px;
}

footer {
    margin-top: 20px;
    text-align: center;
    font-size: 0.8em;
    color: #777;
}

a {
    color: #0066cc;
    text-decoration: none;
}

a:hover {
    text-decoration: underline;
}' > style.css
```

### 3. Starte einen Container mit deiner Website
```bash
docker run -d -p 8080:80 \
  -v $(pwd)/index.html:/usr/share/nginx/html/index.html \
  -v $(pwd)/ueber.html:/usr/share/nginx/html/ueber.html \
  -v $(pwd)/style.css:/usr/share/nginx/html/style.css \
  --name meine-website nginx
```

### 4. Überprüfe deine Website
Öffne einen Browser und gehe zu:
- http://localhost:8080 (Startseite)
- http://localhost:8080/ueber.html (Über-Seite)


Aktualisiere die Seite in deinem Browser und beobachte die Änderung.

### 5. Untersuche den Container
```bash
# Container-Logs anzeigen
docker logs meine-website

# Verbindung zum Container herstellen
docker exec -it meine-website bash

# Im Container: Überprüfe, ob die Dateien korrekt gemountet wurden
ls -la /usr/share/nginx/html/

# Container verlassen
exit
```

### 6. Stoppe und entferne den Container
```bash
docker stop meine-website
docker rm meine-website
```

### 7. Bonus-Aufgabe: Erstelle ein eigenes Docker-Image

Erstelle eine Dockerfile-Datei:
```bash
echo 'FROM nginx:alpine
COPY index.html /usr/share/nginx/html/
COPY ueber.html /usr/share/nginx/html/
COPY style.css /usr/share/nginx/html/
EXPOSE 80' > Dockerfile
```

Baue dein eigenes Image:
```bash
docker build -t meine-eigene-website .
```

Starte einen Container von deinem eigenen Image:
```bash
docker run -d -p 8081:80 --name mein-eigenes-image meine-eigene-website
```

Überprüfe, ob die Website unter http://localhost:8081 funktioniert.

## Lösung überprüfen:

Deine Lösung ist korrekt, wenn:
1. Deine Website unter http://localhost:8080 aufrufbar ist
2. Du die Dateien im Container anzeigen kannst
3. Du den Container erfolgreich gestoppt und entfernt hast
4. (Bonus) Deine eigene Website über das selbst erstellte Image unter http://localhost:8081 läuft
