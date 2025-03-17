# Docker Compose Installation für WSL

Diese Anleitung zeigt, wie Sie Docker Compose schnell in Ihrer WSL-Umgebung installieren können.

## Voraussetzungen
- WSL ist bereits eingerichtet
- Docker ist bereits installiert

## Installation

### 1. Paketlisten aktualisieren

```bash
sudo apt-get update
```

### 2. Voraussetzungen installieren

```bash
sudo apt-get install -y ca-certificates curl gnupg
```

### 3. Docker GPG-Schlüssel hinzufügen (falls noch nicht vorhanden)

```bash
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg
```

### 4. Docker Compose installieren

**Option A - über das Docker-Compose-Plugin (empfohlen):**

```bash
sudo apt-get install -y docker-compose-plugin
```

**Option B - über pip (Python-Paketmanager):**

```bash
sudo apt-get install -y python3-pip
pip3 install docker-compose
```

### 5. Installation überprüfen

```bash
docker compose version
```

## Hinweis zur Verwendung

- Bei Option A verwenden Sie `docker compose` (mit Leerzeichen)
- Bei Option B verwenden Sie `docker-compose` (mit Bindestrich)

## Fertig!

Sie können jetzt Docker Compose in Ihrer WSL-Umgebung nutzen.
