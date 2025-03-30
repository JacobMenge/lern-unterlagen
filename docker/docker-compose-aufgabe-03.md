# Docker und Docker Compose Grundlagen - Übungen

## Übung 1: Docker Basics
### Aufgabe 1.1: Container starten und verwalten
- Starte einen Nginx-Container mit dem Namen "mein-webserver" im Hintergrund
- Überprüfe, ob der Container läuft
- Zeige die Logs des Containers an
- Stoppe den Container
- Starte den Container erneut
- Lösche den Container

### Aufgabe 1.2: Container-Interaktion
- Starte einen Ubuntu-Container im interaktiven Modus
- Führe den Befehl `ls -la` aus
- Installiere das Paket `curl` im Container
- Verlasse den Container, ohne ihn zu beenden
- Verbinde dich erneut mit dem laufenden Container
- Beende und lösche den Container

## Übung 2: Dockerfile verstehen
### Aufgabe 2.1: Dockerfile Analyse
Gegeben ist folgendes Dockerfile für eine Node.js-Anwendung:

```dockerfile
FROM node:14
WORKDIR /app
COPY package.json .
RUN npm install
COPY . .
EXPOSE 3000
CMD ["npm", "start"]
```

Beantworte folgende Fragen:
- Was ist die Basis des Images?
- Welcher Befehl wird bei Container-Start ausgeführt?
- In welchem Verzeichnis werden die Befehle ausgeführt?
- Welcher Port wird freigegeben?

### Aufgabe 2.2: Einfaches Dockerfile erstellen
Erstelle ein Dockerfile für eine einfache HTML/CSS-Website:
- Verwende nginx:alpine als Basis-Image
- Kopiere den Inhalt eines "html"-Ordners in den nginx-Webserver-Pfad (/usr/share/nginx/html)
- Exponiere Port 80
- Baue das Image und starte einen Container

## Übung 3: Docker Compose Grundlagen
### Aufgabe 3.1: Docker Compose File Analyse
Gegeben ist folgendes Docker Compose File:

```yaml
services:
  frontend:
    image: nginx
    ports:
      - "8080:80"
    volumes:
      - ./frontend:/usr/share/nginx/html
    
  backend:
    build: ./backend
    ports:
      - "3000:3000"
    environment:
      - MONGO_URL=mongodb://db:27017/myapp
    depends_on:
      - db
      
  db:
    image: mongo:4.4
    volumes:
      - mongo_data:/data/db

volumes:
  mongo_data:
```

Beantworte folgende Fragen:
- Welche Services werden definiert?
- Auf welchem Host-Port ist der Frontend-Service erreichbar?
- Welcher Dienst wird aus einem lokalen Dockerfile gebaut?
- Wie kann das Backend auf die Datenbank zugreifen?
- Wo werden die MongoDB-Daten gespeichert?

### Aufgabe 3.2: Einfache Webserver-Umgebung
Erstelle ein Docker Compose File für einen einfachen Webserver:
- Verwende nginx als Webserver
- Mappe den Port 80 des Containers auf Port 8080 des Hosts
- Mounte ein lokales Verzeichnis mit HTML/CSS-Dateien als Webroot
- Starte die Umgebung und teste den Zugriff

## Übung 4: Fehlersuche und Debugging
### Aufgabe 4.1: Fehler im Docker Compose File finden
Gegeben ist folgendes fehlerhaftes Docker Compose File:

```yaml
services:
  frontend:
    image: nginx
    ports:
      - 8080:80
    links:
      - api
    
  api:
    build: ./backend
    ports:
      - 3000:3000
    links:
      - db
    environment:
      - DB_HOST=mongodb://db:27017
    
  db:
    image: mongo
    ports:
      - 27017
```

Identifiziere und korrigiere mindestens 3 Probleme in diesem File.

## Aufgabe 5: Der Infrastruktur-Spaziergang

Nach so viel Zeit mit Docker und Containern ist es wichtig, auch die eigene "Infrastruktur" zu warten. Nimm dir 20 Minuten Zeit für einen Spaziergang an der frischen Luft.

### Warum ist das gesund?

Als Informatiker im Homeoffice verbringen wir oft 8+ Stunden täglich sitzend vor dem Bildschirm, was mehrere negative Auswirkungen haben kann:

1. **Körperliche Gesundheit**: Langes Sitzen erhöht das Risiko für Rückenprobleme, Herz-Kreislauf-Erkrankungen und Stoffwechselstörungen.

2. **Kognitive Leistung**: Studien zeigen, dass regelmäßige Bewegung die Gehirndurchblutung fördert, was zu besserer Konzentration und Problemlösungsfähigkeit führt - genau das, was wir beim Debuggen komplexer Docker-Compose-Setups brauchen.

3. **Debug-Effekt**: Wie beim Rubber-Duck-Debugging hilft das Weggehen vom Problem oft dabei, neue Lösungsansätze zu finden. Viele knifflige Container-Netzwerkprobleme lösen sich gedanklich während eines Spaziergangs.

   > **Was ist Rubber-Duck-Debugging?** Diese Technik beschreibt, wie man durch das laute Erklären eines Problems oft selbst auf die Lösung kommt. Statt einer Gummiente kann das auch ein Kollege sein, der einfach nur zuhört, oder sogar ein leerer Stuhl. Das Wesentliche ist der Perspektivwechsel: Indem du versuchst, das Problem für jemand anderen verständlich zu machen, musst du es anders strukturieren und durchdenken. Bei diesem Prozess fällt dir häufig auf: "Moment mal, habe ich eigentlich geprüft, ob alle Container im selben Netzwerk sind?" oder "Warte, ich habe ja gar nicht verifiziert, ob die Umgebungsvariable richtig gesetzt wurde!" Ein Spaziergang funktioniert ähnlich - du trittst zurück vom Bildschirm, bekommst Abstand und siehst plötzlich den Wald vor lauter Bäumen wieder.

4. **Augenentlastung**: Die 20-20-20-Regel empfiehlt, alle 20 Minuten für 20 Sekunden 20 Fuß (etwa 6 Meter) weit zu schauen, um Augenbelastung zu reduzieren. Ein Spaziergang bietet diese Entlastung automatisch.

5. **Container für Gedanken**: Wie Docker Container isolierte Umgebungen bieten, schafft ein Spaziergang einen "Container" für deine Gedanken - abgekapselt von Slack-Benachrichtigungen und E-Mails.

### Bonus-Aufgabe

Überlege während des Spaziergangs, welche Parallelen es zwischen einem gut orchestrierten Docker-System und einem gesunden Lebensstil gibt.


# Docker und Docker Compose Grundlagen - Lösungen

## Übung 1: Docker Basics

### Aufgabe 1.1: Container starten und verwalten

```bash
# Nginx-Container im Hintergrund starten
docker run -d --name mein-webserver nginx

# Container-Status überprüfen
docker ps

# Logs anzeigen
docker logs mein-webserver

# Container stoppen
docker stop mein-webserver

# Container wieder starten
docker start mein-webserver

# Container löschen
docker rm -f mein-webserver
```

Bei `docker run` bedeutet `-d`, dass der Container im Hintergrund läuft, und mit `--name` kannst du ihm einen Namen geben. Mit `docker ps` siehst du alle laufenden Container. Wenn du den Container löschen willst, aber er noch läuft, kannst du `-f` benutzen, um das Löschen zu erzwingen.

### Aufgabe 1.2: Container-Interaktion

```bash
# Ubuntu-Container interaktiv starten
docker run -it --name ubuntu-container ubuntu bash

# Im Container: Dateien auflisten
ls -la

# Im Container: curl installieren
apt-get update
apt-get install -y curl

# Container verlassen ohne zu beenden
# Drücke CTRL+P, dann CTRL+Q

# Wieder mit dem Container verbinden
docker exec -it ubuntu-container bash

# Aufräumen
docker stop ubuntu-container
docker rm ubuntu-container
```

Die Option `-it` ist wichtig, wenn du mit dem Container interagieren willst - `-i` hält STDIN offen und `-t` gibt dir ein Terminal. Mit `docker exec` kannst du Befehle in einem laufenden Container ausführen.

## Übung 2: Dockerfile verstehen

### Aufgabe 2.1: Dockerfile Analyse

Für dieses Dockerfile:
```dockerfile
FROM node:14
WORKDIR /app
COPY package.json .
RUN npm install
COPY . .
EXPOSE 3000
CMD ["npm", "start"]
```

Die Antworten sind:
- Basis des Images: **node:14**
- Befehl bei Container-Start: **npm start**
- Ausführungsverzeichnis: **/app**
- Freigegebener Port: **3000**

Die Befehle kopieren zuerst nur die package.json, installieren die Abhängigkeiten und kopieren dann erst den Rest der Dateien. Das ist ein Trick, um den Docker-Cache besser zu nutzen - wenn sich nur der Code ändert, aber nicht die Abhängigkeiten, muss `npm install` nicht jedes Mal neu ausgeführt werden.

### Aufgabe 2.2: Einfaches Dockerfile erstellen

```dockerfile
FROM nginx:alpine
COPY html/ /usr/share/nginx/html/
EXPOSE 80
```

Um das Image zu bauen und zu starten:

```bash
# Image bauen
docker build -t meine-website .

# Container starten
docker run -d -p 80:80 --name website-container meine-website
```

Ich hab hier kein CMD hinzugefügt, weil das nginx-Image bereits ein Standardkommando hat, das den Webserver startet. `-p 80:80` verbindet Port 80 des Containers mit Port 80 auf deinem Computer.

## Übung 3: Docker Compose Grundlagen

### Aufgabe 3.1: Docker Compose File Analyse

Für dieses Docker Compose File:
```yaml
services:
  frontend:
    image: nginx
    ports:
      - "8080:80"
    volumes:
      - ./frontend:/usr/share/nginx/html
    
  backend:
    build: ./backend
    ports:
      - "3000:3000"
    environment:
      - MONGO_URL=mongodb://db:27017/myapp
    depends_on:
      - db
      
  db:
    image: mongo:4.4
    volumes:
      - mongo_data:/data/db
volumes:
  mongo_data:
```

Die Antworten:
- Definierte Services: **frontend, backend, db**
- Host-Port des Frontend-Services: **8080**
- Dienst aus lokalem Dockerfile: **backend**
- Backend-Zugriff auf die Datenbank: **Über die Umgebungsvariable MONGO_URL mit dem Hostnamen "db"**
- Speicherort der MongoDB-Daten: **Im benannten Volume "mongo_data"**

Docker Compose erstellt automatisch ein Netzwerk zwischen den Containern, sodass sie sich über ihre Service-Namen erreichen können. Deshalb kann der Backend-Service die Datenbank über den Namen "db" ansprechen.

### Aufgabe 3.2: Einfache Webserver-Umgebung

```yaml
version: '3'

services:
  webserver:
    image: nginx
    ports:
      - "8080:80"
    volumes:
      - ./website:/usr/share/nginx/html
```

So kannst du es testen:

```bash
# Testdatei erstellen
mkdir -p website
echo "<h1>Hallo Docker!</h1>" > website/index.html

# Starten
docker-compose up -d

# Im Browser aufrufen: http://localhost:8080
```

Das lokale Verzeichnis "website" wird in den Container gemountet, sodass du die Webseiten bearbeiten kannst, ohne den Container neu starten zu müssen.

## Übung 4: Fehlersuche und Debugging

### Aufgabe 4.1: Fehler im Docker Compose File finden

Hier ist das korrigierte File:

```yaml
services:
  frontend:
    image: nginx
    ports:
      - "8080:80"          # Anführungszeichen hinzugefügt
    depends_on:            # "links" ist veraltet, besser "depends_on" nutzen
      - api
    
  api:
    build: ./backend
    ports:
      - "3000:3000"        # Anführungszeichen hinzugefügt
    depends_on:            # "links" ist veraltet
      - db
    environment:
      - DB_HOST=mongodb://db:27017
    
  db:
    image: mongo
    volumes:               # Volume für Datenpersistenz hinzugefügt
      - mongo_data:/data/db
    ports:
      - "27017:27017"      # Port-Format korrigiert

volumes:                   # Fehlenden Volumes-Abschnitt hinzugefügt
  mongo_data:
```

Die wichtigsten Fehler waren:
1. Fehlende Anführungszeichen bei den Ports
2. Die veraltete "links"-Option statt "depends_on"
3. Keine Datenpersistenz für die Datenbank
4. Falsche Port-Schreibweise bei der Datenbank
5. Fehlender "volumes"-Abschnitt unten

Ohne das Volume würden alle Daten in der Datenbank verloren gehen, sobald der Container neu gestartet wird!
