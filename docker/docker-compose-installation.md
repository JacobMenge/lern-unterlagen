# Docker Compose Installation für WSL

Diese Anleitung zeigt, wie Du Docker Compose schnell in Deiner WSL-Umgebung installieren kannst.

## Voraussetzungen
- WSL ist bereits eingerichtet
- Docker ist bereits installiert

## Installation

### 1. Paketlisten aktualisieren
```bash
sudo apt-get update
```
**Warum?** Dieser Befehl aktualisiert die Liste der verfügbaren Pakete. So stellst Du sicher, dass Du die neuesten Versionen installierst.

### 2. Voraussetzungen installieren
```bash
sudo apt-get install -y ca-certificates curl gnupg
```
**Warum?** Diese Tools werden benötigt:
- `ca-certificates`: Für sichere Verbindungen zum Internet
- `curl`: Zum Herunterladen von Dateien
- `gnupg`: Zum Überprüfen von digitalen Signaturen

### 3. Docker GPG-Schlüssel hinzufügen
```bash
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg
```
**Warum?** Der GPG-Schlüssel ist wie ein digitaler Fingerabdruck von Docker. Er stellt sicher, dass die Pakete wirklich von Docker stammen und nicht manipuliert wurden. Das ist wichtig für Deine Sicherheit, damit Du keine gefälschte Software installierst.

### 4. Docker Compose installieren

**Option A - über das Docker-Compose-Plugin (empfohlen):**
```bash
sudo apt-get install -y docker-compose-plugin
```
**Warum?** Diese Methode installiert Docker Compose als Plugin direkt in Docker. Das ist die empfohlene und modernere Methode.

**Option B - über pip (Python-Paketmanager):**
```bash
sudo apt-get install -y python3-pip
pip3 install docker-compose
```
**Warum?** Diese alternative Methode nutzt den Python-Paketmanager. Sie ist nützlich, wenn die erste Option nicht funktioniert oder Du spezifische Anforderungen hast.

### 5. Installation überprüfen
```bash
docker compose version
```
**Warum?** Mit diesem Befehl kannst Du prüfen, ob Docker Compose korrekt installiert wurde und welche Version Du hast.

## Hinweis zur Verwendung
- Bei Option A verwendest Du `docker compose` (mit Leerzeichen)
- Bei Option B verwendest Du `docker-compose` (mit Bindestrich)

## Fertig!
Du kannst jetzt Docker Compose in Deiner WSL-Umgebung nutzen, um mehrere Docker-Container gleichzeitig zu verwalten und zu starten.
