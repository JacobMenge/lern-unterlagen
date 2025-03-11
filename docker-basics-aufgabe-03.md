# Dockerfile Übung: Erste Schritte

In dieser praktischen Übung erstellen wir Schritt für Schritt unser erstes Dockerfile und lernen die Grundlagen kennen.

## Voraussetzungen

- Docker ist installiert
- Ein Terminal/Kommandozeile ist geöffnet
- Ein Texteditor steht zur Verfügung

## Übung: Erstelle deine erste Webseite in einem Docker-Container

### Schritt 1: Projekt-Ordner erstellen

Öffne dein Terminal und führe folgende Befehle aus:

```bash
mkdir meine-docker-webseite
cd meine-docker-webseite
```

> **Erklärung**: Wir erstellen einen neuen Ordner für unser Projekt und wechseln in diesen Ordner.

### Schritt 2: HTML-Datei erstellen

Erstelle eine Datei namens `index.html` mit folgendem Inhalt:

```bash
echo '<!DOCTYPE html>
<html>
<head>
    <title>Meine Docker-Webseite</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            margin: 0;
            padding: 30px;
            background-color: #f0f8ff;
        }
        .container {
            max-width: 800px;
            margin: 0 auto;
            background-color: white;
            padding: 20px;
            border-radius: 8px;
            box-shadow: 0 2px 4px rgba(0,0,0,0.1);
        }
        h1 {
            color: #0066cc;
        }
    </style>
</head>
<body>
    <div class="container">
        <h1>Hallo Docker!</h1>
        <p>Dies ist meine erste Webseite in einem Docker-Container.</p>
        <p>Heutiges Datum: <span id="datum"></span></p>
    </div>
    
    <script>
        document.getElementById("datum").textContent = new Date().toLocaleDateString();
    </script>
</body>
</html>' > index.html
```

> **Erklärung**: Diese HTML-Datei enthält eine einfache Webseite mit etwas CSS-Styling und einem JavaScript, das das aktuelle Datum anzeigt.

### Schritt 3: Dockerfile erstellen

Erstelle nun eine Datei namens `Dockerfile` (ohne Dateiendung) mit diesem Inhalt:

```bash
echo 'FROM nginx:alpine

COPY index.html /usr/share/nginx/html/

EXPOSE 80

CMD ["nginx", "-g", "daemon off;"]' > Dockerfile
```

> **Erklärung**: 
> - `FROM nginx:alpine`: Wir verwenden das Nginx-Webserver-Image auf Basis von Alpine Linux (sehr klein und effizient)
> - `COPY index.html /usr/share/nginx/html/`: Kopiert unsere HTML-Datei in den Standard-Webserver-Ordner von Nginx
> - `EXPOSE 80`: Dokumentiert, dass der Container auf Port 80 hört (Standard-HTTP-Port)
> - `CMD ["nginx", "-g", "daemon off;"]`: Startet den Nginx-Webserver im Vordergrund

### Schritt 4: Docker-Image erstellen

Baue dein Docker-Image mit dem folgenden Befehl:

```bash
docker build -t meine-webseite .
```

> **Erklärung**: 
> - `docker build`: Befehl zum Erstellen eines Docker-Images
> - `-t meine-webseite`: Gibt dem Image einen Namen (Tag)
> - `.`: Nutzt das aktuelle Verzeichnis als Build-Kontext

Du solltest eine Ausgabe sehen, die den Build-Prozess zeigt. Am Ende sollte dein Image erfolgreich erstellt worden sein.

### Schritt 5: Container starten

Starte einen Container aus deinem neuen Image:

```bash
docker run -d -p 8080:80 --name mein-webserver meine-webseite
```

> **Erklärung**: 
> - `docker run`: Befehl zum Starten eines Containers
> - `-d`: Führt den Container im Hintergrund aus (detached)
> - `-p 8080:80`: Leitet Traffic von Port 8080 auf deinem Computer an Port 80 im Container weiter
> - `--name mein-webserver`: Gibt dem Container einen Namen
> - `meine-webseite`: Der Name des Images, das wir verwenden

> **Hinweis**: Falls Port 8080 bereits belegt ist, kannst du einen anderen Port verwenden, z.B. `-p 8081:80` oder `-p 3000:80`.

### Schritt 6: Überprüfe deine Webseite

Öffne einen Webbrowser und gehe zu:
```
http://localhost:8080
```

Du solltest jetzt deine Webseite sehen!

> **Hinweis für WSL-Benutzer**: Falls du Docker unter WSL (Windows Subsystem for Linux) verwendest und Probleme mit localhost hast, versuche die IP-Adresse deiner WSL-Instanz zu verwenden. Du kannst diese mit `ip addr show` herausfinden.

### Schritt 7: Ändere deine Webseite

Lass uns die Webseite aktualisieren. Bearbeite die `index.html` Datei:

```bash
echo '<!DOCTYPE html>
<html>
<head>
    <title>Meine Docker-Webseite</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            margin: 0;
            padding: 30px;
            background-color: #f0f8ff;
        }
        .container {
            max-width: 800px;
            margin: 0 auto;
            background-color: white;
            padding: 20px;
            border-radius: 8px;
            box-shadow: 0 2px 4px rgba(0,0,0,0.1);
        }
        h1 {
            color: #cc0066;
        }
    </style>
</head>
<body>
    <div class="container">
        <h1>Hallo Docker! (Aktualisiert)</h1>
        <p>Dies ist meine erste Webseite in einem Docker-Container.</p>
        <p>Heutiges Datum: <span id="datum"></span></p>
        <p><strong>Diese Zeile wurde hinzugefügt!</strong></p>
    </div>
    
    <script>
        document.getElementById("datum").textContent = new Date().toLocaleDateString();
    </script>
</body>
</html>' > index.html
```

> **Wichtiger Hinweis**: Die Änderung der HTML-Datei auf deinem Computer hat keine Auswirkungen auf den laufenden Container! Die Datei wurde bereits beim Erstellen des Images in den Container kopiert und Container sind standardmäßig unveränderlich. Um die Änderungen zu sehen, müssen wir ein neues Image bauen und einen neuen Container starten.

### Schritt 8: Neues Image erstellen und neuen Container starten

```bash
# Stoppe und lösche den alten Container
docker stop mein-webserver
docker rm mein-webserver

# Erstelle ein neues Image (Version 2)
docker build -t meine-webseite:v2 .

# Starte einen neuen Container mit dem aktualisierten Image
docker run -d -p 8080:80 --name mein-webserver-v2 meine-webseite:v2
```

> **Erklärung**: 
> - Wir stoppen und entfernen zuerst den alten Container
> - Dann erstellen wir ein neues Image mit dem Tag "v2" (eine Versionsnummer)
> - Schließlich starten wir einen neuen Container mit dem aktualisierten Image
> - Dies zeigt ein wichtiges Konzept: In Docker werden Images erstellt, nicht verändert

### Schritt 9: Überprüfe die Änderungen

Öffne erneut den Browser und gehe zu:
```
http://localhost:8080
```

Du solltest jetzt die aktualisierte Version deiner Webseite sehen!

### Schritt 10: Container-Logs anzeigen

Um zu sehen, was im Container passiert:

```bash
docker logs mein-webserver-v2
```

> **Erklärung**: Dieser Befehl zeigt die Ausgaben des Containers an. Bei einem Webserver wie Nginx kannst du hier Zugriffe und mögliche Fehler sehen.

### Schritt 11: Verbinde dich mit dem Container

Du kannst eine Shell im laufenden Container öffnen:

```bash
docker exec -it mein-webserver-v2 sh
```

> **Erklärung**: 
> - `docker exec`: Führt einen Befehl im laufenden Container aus
> - `-it`: Ermöglicht eine interaktive Shell (`i` = interaktiv, `t` = Terminal)
> - `sh`: Startet die Shell im Container (Alpine verwendet sh statt bash)

Innerhalb des Containers kannst du:

```bash
# Inhalt des Webserver-Verzeichnisses anzeigen
ls -la /usr/share/nginx/html/

# Webserver-Konfiguration anzeigen
cat /etc/nginx/conf.d/default.conf

# Container verlassen
exit
```

### Schritt 12: Aufräumen

Wenn du fertig bist, kannst du aufräumen:

```bash
# Container stoppen
docker stop mein-webserver-v2

# Container löschen
docker rm mein-webserver-v2

# Images anzeigen
docker images

# (Optional) Images löschen
# docker rmi meine-webseite meine-webseite:v2
```

## Erweiterung: Erstelle ein dynamischeres Dockerfile

Wenn du noch weitermachen möchtest, erstelle ein neues Dockerfile mit mehr Funktionen:

```bash
# Erstelle ein Startskript
echo '#!/bin/sh
# Generiere dynamische Informationen
DATE=$(date)
HOSTNAME=$(hostname)

# Generiere ein temporäres HTML-Fragment
echo "<div class=\"dynamic-info\">
<p>Container-ID: $HOSTNAME</p>
<p>Container gestartet am: $DATE</p>
</div>" > /tmp/dynamic-content.html

# Füge das Fragment in die HTML-Datei ein - NACH dem Container-Tag, vor dem schließenden div
sed -i "s|<\/div>|$(cat /tmp/dynamic-content.html)\n</div>|" /usr/share/nginx/html/index.html

# Starte Nginx im Vordergrund
echo "Container gestartet am $DATE"
nginx -g "daemon off;"
' > start.sh

chmod +x start.sh

# Erstelle ein neues Dockerfile
echo 'FROM nginx:alpine

# Arbeitsverzeichnis setzen
WORKDIR /usr/share/nginx/html

# HTML-Datei kopieren
COPY index.html .

# Startskript kopieren
COPY start.sh /start.sh

# Umgebungsvariable setzen
ENV ERSTELLER="Docker-Kursteilnehmer"

# Diese Information zur HTML-Datei hinzufügen (beim Image-Build)
RUN echo "<p>Erstellt von: $ERSTELLER</p>" >> index.html

# Port dokumentieren
EXPOSE 80

# Container starten mit unserem Skript
CMD ["/start.sh"]' > Dockerfile.dynamic
```

> **Erklärung des verbesserten Skripts**: 
> - Das Skript erstellt die dynamischen Inhalte erst beim Start des Containers
> - Es verwendet sed, um den Inhalt an einer bestimmten Stelle einzufügen, statt ihn anzuhängen
> - Dies vermeidet doppelte Einträge bei Container-Neustarts

Baue und starte diesen dynamischen Container:

```bash
docker build -t dynamische-webseite -f Dockerfile.dynamic .
docker run -d -p 8081:80 --name dynamischer-container dynamische-webseite
```

> **Hinweis**: Wir verwenden hier Port 8081, damit du gleichzeitig beide Container laufen lassen kannst.

Besuche http://localhost:8081 und sieh dir die dynamisch generierte Webseite an!

## Zusammenfassung

In dieser Übung hast du:
1. Eine HTML-Datei für eine einfache Webseite erstellt
2. Ein Dockerfile geschrieben, um diese Webseite in einem Container bereitzustellen
3. Ein Docker-Image gebaut und einen Container gestartet
4. Die Webseite aktualisiert und ein neues Image erstellt
5. Gelernt, dass Container unveränderlich sind (immutable)
6. Grundlegende Docker-Befehle kennengelernt
7. Ein erweitertes Dockerfile mit mehr Funktionen erstellt

Herzlichen Glückwunsch zu deinem ersten Dockerfile!
