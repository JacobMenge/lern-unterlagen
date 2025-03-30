# Docker Compose Grundlagenübung

Diese Übung konzentriert sich ausschließlich auf die grundlegenden Funktionen von Docker Compose.

## Voraussetzungen

- Docker und Docker Compose installiert

## Übung 1: Erste Docker Compose Datei

### Beschreibung

Erstelle deine erste `docker-compose.yml` Datei und starte einen einfachen Nginx-Webserver.

### Vorgefertigter Code

#### `index.html`
```html
<!DOCTYPE html>
<html>
<head>
    <title>Docker Compose Test</title>
</head>
<body>
    <h1>Hallo Docker Compose!</h1>
    <p>Wenn du diese Seite siehst, funktioniert Docker Compose korrekt.</p>
</body>
</html>
```

### Aufgabe

1. Erstelle einen neuen Ordner für dein Projekt
2. Erstelle den Unterordner `html` und speichere die `index.html` darin
3. Erstelle eine `docker-compose.yml` mit folgendem Inhalt:

```yaml
version: '3'

services:
  webserver:
    image: nginx:alpine
    ports:
      - "80:80"
    volumes:
      - ./html:/usr/share/nginx/html
```

4. Starte den Container mit:
```bash
docker-compose up
```

5. Öffne einen Browser und gehe zu `http://localhost`
6. Beende den Container mit Strg+C oder in einem neuen Terminal mit:
```bash
docker-compose down
```

## Übung 2: Docker Compose im Hintergrund

### Beschreibung

Lerne, wie du Docker Compose im Hintergrund (detached mode) ausführst.

### Aufgabe

1. Starte den Container im Hintergrund:
```bash
docker-compose up -d
```

2. Überprüfe den Status der Container:
```bash
docker-compose ps
```

3. Sieh dir die Logs an:
```bash
docker-compose logs
```

4. Stoppe die Container:
```bash
docker-compose down
```

## Übung 3: Mehrere Services

### Beschreibung

Füge einen zweiten Service hinzu, um zu lernen, wie Docker Compose mehrere Container verwaltet.

### Vorgefertigter Code

#### `html2/index.html`
```html
<!DOCTYPE html>
<html>
<head>
    <title>Zweiter Service</title>
</head>
<body>
    <h1>Zweiter Service</h1>
    <p>Dies ist der zweite Service in deiner Docker Compose Konfiguration.</p>
</body>
</html>
```

### Aufgabe

1. Erstelle einen neuen Ordner `html2` und speichere die zweite `index.html` darin
2. Erweitere die `docker-compose.yml`:

```yaml
version: '3'

services:
  webserver1:
    image: nginx:alpine
    ports:
      - "80:80"
    volumes:
      - ./html:/usr/share/nginx/html

  webserver2:
    image: nginx:alpine
    ports:
      - "8080:80"
    volumes:
      - ./html2:/usr/share/nginx/html
```

3. Starte beide Container:
```bash
docker-compose up -d
```

4. Überprüfe, ob beide Webseiten unter `http://localhost` und `http://localhost:8080` erreichbar sind
5. Stoppe einen einzelnen Service:
```bash
docker-compose stop webserver2
```

6. Starte ihn wieder:
```bash
docker-compose start webserver2
```

7. Beende alle Container:
```bash
docker-compose down
```

## Übung 4: Einfache Netzwerke

### Beschreibung

Lerne, wie Container in einem Netzwerk miteinander kommunizieren können.

### Aufgabe

1. Aktualisiere die `docker-compose.yml`:

```yaml
version: '3'

services:
  webserver1:
    image: nginx:alpine
    ports:
      - "80:80"
    volumes:
      - ./html:/usr/share/nginx/html
    networks:
      - webnet

  webserver2:
    image: nginx:alpine
    ports:
      - "8080:80"
    volumes:
      - ./html2:/usr/share/nginx/html
    networks:
      - webnet

networks:
  webnet:
```

2. Starte die Container:
```bash
docker-compose up -d
```

3. Führe eine Bash-Shell im ersten Container aus:
```bash
docker-compose exec webserver1 sh
```

4. Ping den zweiten Container vom ersten aus an:
```bash
ping webserver2
```

5. Verlasse die Shell mit `exit`
6. Beende alle Container:
```bash
docker-compose down
```

## Übung 5: Umgebungsvariablen

### Beschreibung

Lerne, wie du Umgebungsvariablen in Docker Compose verwendest.

### Aufgabe

1. Erstelle eine Datei `.env` im Projektordner:
```
WEB_PORT=8888
```

2. Aktualisiere die `docker-compose.yml`, um die Umgebungsvariable zu verwenden:

```yaml
version: '3'

services:
  webserver:
    image: nginx:alpine
    ports:
      - "${WEB_PORT}:80"
    volumes:
      - ./html:/usr/share/nginx/html
```

3. Starte den Container:
```bash
docker-compose up -d
```

4. Überprüfe, ob die Webseite unter `http://localhost:8888` erreichbar ist
5. Beende den Container:
```bash
docker-compose down
```

## Fazit

In diesen einfachen Übungen hast du die Grundlagen von Docker Compose gelernt:

1. Wie man eine einfache Docker Compose-Konfiguration erstellt
2. Wie man Container im Hintergrund startet und überwacht
3. Wie man mehrere Services definiert und verwaltet
4. Wie man Netzwerke für die Kommunikation zwischen Containern einrichtet
5. Wie man Umgebungsvariablen zur Konfiguration verwendet
