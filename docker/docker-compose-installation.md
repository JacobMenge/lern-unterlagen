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

### 2. Docker-Repository einrichten (falls noch nicht geschehen)
```bash
# Installiere benötigte Pakete
sudo apt-get install -y ca-certificates curl gnupg lsb-release

# Füge Docker's offiziellen GPG-Schlüssel hinzu
sudo mkdir -p /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg

# Füge das Repository zur Apt-Quellenliste hinzu
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# Aktualisiere die Paketlisten mit dem neuen Repository
sudo apt-get update
```
**Warum?** Diese Schritte fügen das offizielle Docker-Repository zu Deinem System hinzu. Der GPG-Schlüssel ist wie ein digitaler Fingerabdruck von Docker. Er stellt sicher, dass die Pakete wirklich von Docker stammen und nicht manipuliert wurden. Das ist wichtig für Deine Sicherheit.

### 3. Docker Compose installieren

**Option A - Docker Compose als Plugin installieren:**
```bash
sudo apt-get install -y docker-compose-plugin
```
**Warum?** Diese Methode installiert Docker Compose als Plugin direkt in Docker. Nach der Einrichtung des Repositories sollte dieses Paket verfügbar sein.

**Falls Option A nicht funktioniert, versuche Option B:**

**Option B - Docker Compose über den offiziellen Installer installieren:**
```bash
# Aktuelle stabile Version von Docker Compose herunterladen
sudo curl -L "https://github.com/docker/compose/releases/download/v2.24.1/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose

# Ausführbare Berechtigungen hinzufügen
sudo chmod +x /usr/local/bin/docker-compose

# Symbolischen Link erstellen (optional)
sudo ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose
```
**Warum?** Diese Methode lädt die Docker Compose Binärdatei direkt von GitHub herunter. Der Vorteil ist, dass du genau die Version bekommst, die du angibst, und diese Methode funktioniert auf fast allen Linux-Systemen.

**Option C - über pip (Python-Paketmanager):**
```bash
sudo apt-get install -y python3-pip
pip3 install docker-compose
```
**Warum?** Diese alternative Methode nutzt den Python-Paketmanager. Sie ist nützlich, wenn die anderen Optionen nicht funktionieren.

### 4. Installation überprüfen
```bash
# Bei Option A
docker compose version

# Bei Option B oder C
docker-compose version
```
**Warum?** Mit diesem Befehl kannst Du prüfen, ob Docker Compose korrekt installiert wurde und welche Version Du hast.

## Hinweis zur Verwendung
- Bei Option A verwendest Du `docker compose` (mit Leerzeichen)
- Bei Option B oder C verwendest Du `docker-compose` (mit Bindestrich)

## Fertig!
Du kannst jetzt Docker Compose in Deiner WSL-Umgebung nutzen, um mehrere Docker-Container gleichzeitig zu verwalten und zu starten.
