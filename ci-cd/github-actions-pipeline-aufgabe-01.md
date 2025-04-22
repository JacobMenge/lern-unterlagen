# GitHub Actions "Hello World" - Deine erste CI/CD Pipeline

Diese Übung führt dich Schritt für Schritt durch deine ersten Erfahrungen mit GitHub Actions. Du wirst lernen, wie du ein Repository erstellst, eine einfache Workflow-Datei anlegst und deine erste Pipeline ausführst.  Solltest du während der Übung Probleme haben oder Fragen aufkommen, melde dich gerne jederzeit per Slack oder im CR bei mir. Ich helfe dir gerne weiter!

> **Hinweis:** CI/CD (Continuous Integration/Continuous Deployment) ist ein moderner Entwicklungsprozess, der Automatisierung nutzt, um Code schneller und zuverlässiger zu testen und bereitzustellen. GitHub Actions ist ein CI/CD-Tool, das direkt in GitHub integriert ist und es dir ermöglicht, automatisierte Workflows für deine Projekte zu erstellen.

## 0. GitHub-Authentifizierung einrichten

Bevor du mit den eigentlichen Schritten beginnst, musst du sicherstellen, dass du dich bei GitHub authentifizieren kannst. Seit August 2021 unterstützt GitHub keine Passwort-Authentifizierung mehr für Git-Operationen. Stattdessen musst du entweder SSH oder einen persönlichen Zugriffstoken (PAT) verwenden.

### 0.1 SSH-Authentifizierung einrichten (empfohlen)

1. **Überprüfe, ob du bereits SSH-Keys hast:**
   ```bash
   ls -la ~/.ssh
   ```
   Wenn du Dateien wie `id_rsa`, `id_ed25519` oder ähnliche siehst, hast du bereits SSH-Keys.

2. **Falls keine SSH-Keys vorhanden sind, erstelle welche:**
   ```bash
   ssh-keygen -t ed25519 -C "deine-email@example.com"
   ```
   Drücke mehrmals Enter, um die Standardeinstellungen zu akzeptieren. Es werden zwei Dateien erstellt: 
   - `~/.ssh/id_ed25519` (privater Schlüssel - halte diesen geheim!)
   - `~/.ssh/id_ed25519.pub` (öffentlicher Schlüssel - diesen teilst du mit GitHub)

3. **Kopiere den öffentlichen Schlüssel:**
   ```bash
   cat ~/.ssh/id_ed25519.pub
   ```
   Kopiere die gesamte Ausgabe.

4. **Füge den Schlüssel zu deinem GitHub-Konto hinzu:**
   - Gehe zu GitHub → Settings → SSH and GPG keys
   - Klicke auf "New SSH key"
   - Gib einen Titel ein (z.B. "Mein Laptop")
   - Füge den kopierten Schlüssel in das Feld "Key" ein
   - Klicke auf "Add SSH key"

5. **Teste die Verbindung:**
   ```bash
   ssh -T git@github.com
   ```
   Du solltest eine Meldung sehen: "Hi username! You've successfully authenticated..."

### 0.2 Persönlichen Zugriffstoken (PAT) erstellen (Alternative)

Wenn du lieber mit HTTPS arbeiten möchtest:

1. **Erstelle einen PAT auf GitHub:**
   - Gehe zu GitHub → Settings → Developer settings → Personal access tokens → Tokens (classic)
   - Klicke auf "Generate new token"
   - Gib einen Namen für den Token ein
   - Wähle mindestens den Bereich "repo" aus
   - Klicke auf "Generate token"
   - **Wichtig:** Kopiere den erzeugten Token sofort! Er wird nur einmal angezeigt.

2. **Speichere den Token für die weitere Verwendung:**
   - Du kannst ihn in einer Passwort-Manager-App speichern
   - Oder Git konfigurieren, den Token zu speichern:
     ```bash
     git config --global credential.helper store
     ```
     (Beachte, dass dies den Token im Klartext auf deinem Computer speichert)

3. **Bei der nächsten Git-Operation:**
   - Verwende deinen GitHub-Benutzernamen
   - Verwende den Token als Passwort

## 1. GitHub Repository erstellen

### 1.1 Neues Repository anlegen

Besuche GitHub und erstelle ein neues Repository:

1. Logge dich in deinen GitHub-Account ein
2. Klicke auf das "+" Symbol in der oberen rechten Ecke und wähle "New repository"
3. Gib deinem Repository einen Namen (z.B. "github-actions-demo")
4. Wähle "Public" als Sichtbarkeit
5. Aktiviere die Option "Add a README file"
6. Klicke auf "Create repository"

**Warum ein Repository?**
- GitHub Actions wird direkt in dein Repository integriert
- Alle Workflow-Dateien werden in deinem Repository gespeichert
- Es ermöglicht die Versionskontrolle deiner Pipeline-Konfiguration

### 1.2 Repository klonen

Klone das Repository auf deinen lokalen Computer. Je nach deiner bevorzugten Authentifizierungsmethode:

**Mit SSH (empfohlen, wenn du den SSH-Schlüssel eingerichtet hast):**
```bash
# Repository mit SSH klonen
git clone git@github.com:dein-username/github-actions-demo.git

# In das Verzeichnis wechseln
cd github-actions-demo
```

**Mit HTTPS und Personal Access Token:**
```bash
# Repository mit HTTPS klonen
git clone https://github.com/dein-username/github-actions-demo.git
# Du wirst nach Benutzername und Passwort gefragt - verwende deinen Token als Passwort

# Oder direkt mit Token im URL (nicht empfohlen für öffentliche Skripts)
git clone https://dein-username:dein-token@github.com/dein-username/github-actions-demo.git

# In das Verzeichnis wechseln
cd github-actions-demo
```

**Warum?** Lokales Arbeiten ermöglicht dir, Änderungen zu testen, bevor du sie auf GitHub hochlädst, und gibt dir mehr Flexibilität bei der Bearbeitung der Dateien.

## 2. Eine einfache GitHub Actions Workflow-Datei erstellen

### 2.1 Workflow-Verzeichnis anlegen

GitHub Actions sucht nach Workflow-Dateien im Verzeichnis `.github/workflows/`. Erstelle dieses Verzeichnis:

```bash
# Erstelle das Verzeichnis für Workflows
mkdir -p .github/workflows
```

**Warum?** GitHub Actions verwendet diese spezifische Verzeichnisstruktur, um Workflow-Dateien zu finden und auszuführen.

### 2.2 Erstelle deine erste Workflow-Datei

Erstelle eine neue Datei namens `hello-world.yml` im Verzeichnis `.github/workflows/`:

```bash
# Erstelle und öffne die Workflow-Datei
touch .github/workflows/hello-world.yml
```

Öffne diese Datei in deinem bevorzugten Texteditor und füge folgenden Inhalt hinzu:

```yaml
name: Hello World Workflow

# Definiert, wann der Workflow ausgeführt werden soll
on:
  # Wird ausgelöst, wenn ein Push auf den main-Branch erfolgt
  push:
    branches: [ main ]
  # Wird auch ausgelöst, wenn ein Pull Request zum main-Branch erstellt wird
  pull_request:
    branches: [ main ]
  # Ermöglicht manuelle Ausführung des Workflows über die GitHub-Oberfläche
  workflow_dispatch:

# Ein Workflow besteht aus einem oder mehreren Jobs
jobs:
  # Dieser Job heißt "hello"
  hello:
    # Definiert, auf welchem Betriebssystem der Job ausgeführt werden soll
    runs-on: ubuntu-latest
    
    # Ein Job besteht aus einer Reihe von Schritten
    steps:
      # Schritt 1: Repository auschecken, damit der Workflow darauf zugreifen kann
      - name: Checkout repository
        uses: actions/checkout@v4
      
      # Schritt 2: Gib "Hello, World!" aus
      - name: Sage Hallo
        run: echo "Hello, World! Willkommen bei GitHub Actions!"
      
      # Schritt 3: Zeige Datum und Uhrzeit an
      - name: Zeige Datum
        run: echo "Aktuelles Datum ist $(date)"
      
      # Schritt 4: Zeige Repository-Informationen
      - name: Zeige Repository-Info
        run: |
          echo "Repository: ${{ github.repository }}"
          echo "Branch: ${{ github.ref }}"
```

**Erklärung der YAML-Datei:**
- `name`: Der Name deines Workflows, der in der GitHub-Oberfläche angezeigt wird
- `on`: Definiert die Ereignisse, die den Workflow auslösen
- `jobs`: Enthält die verschiedenen Jobs, die in diesem Workflow ausgeführt werden
- `runs-on`: Definiert die Umgebung, in der der Job ausgeführt wird
- `steps`: Die einzelnen Schritte, die der Job ausführt
- `uses`: Verwendet eine vordefinierte Aktion (hier: `actions/checkout@v4` zum Auschecken des Repositories)
- `run`: Führt einen Shell-Befehl aus

> **Tipp:** YAML ist eine Auszeichnungssprache, die für ihre Lesbarkeit bekannt ist. Achte auf die korrekte Einrückung, da YAML darauf basiert.

## 3. Workflow hochladen und ausführen

### 3.1 Änderungen committen und pushen

Speichere deine Änderungen und lade sie auf GitHub hoch:

```bash
# Status überprüfen
git status

# Änderungen hinzufügen
git add .github/workflows/hello-world.yml

# Commit erstellen
git commit -m "Füge ersten GitHub Actions Workflow hinzu"

# Änderungen auf GitHub pushen
git push origin main
```

Nachdem du diesen Befehl ausgeführt hast, werden deine Änderungen auf GitHub hochgeladen, und der Workflow wird automatisch gestartet.

**Hinweis:** Wenn du Authentifizierungsprobleme beim Push hast, überprüfe nochmals deinen SSH-Key oder Personal Access Token.

### 3.2 Workflow-Ausführung überprüfen

Nachdem du deine Änderungen gepusht hast, wird der Workflow automatisch ausgelöst:

1. Gehe in deinem Browser zu deinem GitHub-Repository
2. Klicke auf den Tab "Actions" in der oberen Leiste
3. Du solltest deinen "Hello World Workflow" sehen, der gerade ausgeführt wird oder bereits abgeschlossen ist (mit grünem Häkchen)
4. Klicke auf den Workflow-Lauf, um die Details anzuzeigen
5. Klicke auf den Job "hello", um die einzelnen Schritte und ihre Ausgaben zu sehen

Du solltest die Ausgaben der echo-Befehle in den Protokollen sehen können, zum Beispiel:
- "Hello, World! Willkommen bei GitHub Actions!"
- Das aktuelle Datum
- Repository-Informationen

**Was passiert hier?**
- Dein Push auf den main-Branch hat den Workflow automatisch ausgelöst
- GitHub Actions stellt eine virtuelle Maschine bereit (in diesem Fall Ubuntu)
- Der Workflow wird ausgeführt und führt jeden Schritt der Reihe nach aus
- Die Ausgaben jedes Schritts werden protokolliert und sind in der GitHub-Oberfläche sichtbar

## 4. Workflow-Datei erweitern

Lass uns nun den Workflow erweitern, um mehr über GitHub Actions zu lernen:

### 4.1 Öffne die Workflow-Datei

```bash
# Öffne die Workflow-Datei erneut
nano .github/workflows/hello-world.yml  # Oder verwende deinen bevorzugten Editor
```

### 4.2 Erweitere den Workflow

Ersetze den Inhalt mit folgendem erweiterten Workflow:

```yaml
name: Erweiterter Hello World Workflow

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]
  workflow_dispatch:

# Definiere Umgebungsvariablen, die im gesamten Workflow verfügbar sind
env:
  GREETING: "Willkommen bei deinem ersten GitHub Actions Workflow!"

jobs:
  hello:
    runs-on: ubuntu-latest
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
      
      - name: Sage Hallo mit Umgebungsvariable
        run: echo "${{ env.GREETING }}"
      
      - name: Zeige GitHub-Kontext
        run: |
          echo "Event name: ${{ github.event_name }}"
          echo "Repository: ${{ github.repository }}"
          echo "Runner OS: ${{ runner.os }}"
      
      - name: Zeige Verzeichnisinhalt
        run: ls -la
      
      - name: Erstelle eine Datei
        run: |
          echo "Diese Datei wurde durch GitHub Actions erstellt." > generated-file.txt
          echo "Zeitstempel: $(date)" >> generated-file.txt
      
      - name: Zeige Dateiinhalt
        run: cat generated-file.txt
      
      - name: Speichere die Datei als Artefakt
        uses: actions/upload-artifact@v4
        with:
          name: generated-file
          path: generated-file.txt
```

**Erklärung der neuen Elemente:**
- `env`: Definiert Umgebungsvariablen für den gesamten Workflow
- `${{ env.GREETING }}`: Greift auf eine Umgebungsvariable zu
- `${{ github.event_name }}`, `${{ github.repository }}`, `${{ runner.os }}`: Kontextvariablen, die Informationen über den aktuellen Workflow-Lauf bereitstellen
- `actions/upload-artifact@v4`: Eine vordefinierte Aktion, die Dateien als Artefakte speichert, die später heruntergeladen werden können

### 4.3 Änderungen hochladen und überprüfen

Speichere deine Änderungen und lade sie auf GitHub hoch:

```bash
git add .github/workflows/hello-world.yml
git commit -m "Erweitere GitHub Actions Workflow"
git push origin main
```

Überprüfe die Ausführung auf der GitHub-Oberfläche, wie zuvor beschrieben.

### 4.4 Manuelles Ausführen des Workflows

Du kannst den Workflow auch manuell ausführen:

1. Gehe zu deinem Repository auf GitHub
2. Klicke auf den Tab "Actions"
3. Wähle deinen "Erweiterter Hello World Workflow" aus der linken Seitenleiste
4. Klicke auf den Button "Run workflow" auf der rechten Seite
5. Klicke nochmals auf "Run workflow" im Popup-Dialog

Der Workflow wird ausgeführt, und du kannst die Ergebnisse wie zuvor beschrieben überwachen. Diese Funktion ist dank des `workflow_dispatch`-Triggers verfügbar, den wir in der Workflow-Datei definiert haben.

### 4.5 Artefakt herunterladen

Nachdem der Workflow abgeschlossen ist:

1. Gehe zu deinem Workflow-Lauf
2. Scrolle nach unten zum Abschnitt "Artifacts"
3. Klicke auf "generated-file", um das Artefakt herunterzuladen
4. Öffne die heruntergeladene ZIP-Datei, um den Inhalt der generierten Datei zu sehen

Das Artefakt sollte den Text enthalten, den wir im Workflow definiert haben, einschließlich des Zeitstempels.

## 5. Mehrere Jobs hinzufügen

Lass uns einen weiteren Job hinzufügen, um zu zeigen, wie Jobs zusammenarbeiten können:

### 5.1 Öffne die Workflow-Datei

```bash
# Öffne die Workflow-Datei erneut
nano .github/workflows/hello-world.yml
```

### 5.2 Füge einen zweiten Job hinzu

Ersetze den Inhalt mit folgendem erweiterten Workflow:

```yaml
name: Multi-Job Workflow

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]
  workflow_dispatch:

env:
  GREETING: "Willkommen bei deinem ersten GitHub Actions Workflow!"

jobs:
  ersteller:
    name: Datei-Ersteller
    runs-on: ubuntu-latest
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
      
      - name: Erstelle eine Datei
        run: |
          echo "Diese Datei wurde durch GitHub Actions erstellt." > generated-file.txt
          echo "Zeitstempel: $(date)" >> generated-file.txt
          echo "GREETING: ${{ env.GREETING }}" >> generated-file.txt
      
      - name: Speichere die Datei als Artefakt
        uses: actions/upload-artifact@v4
        with:
          name: generated-file
          path: generated-file.txt
  
  verarbeiter:
    name: Datei-Verarbeiter
    needs: ersteller  # Dieser Job wartet, bis der "ersteller"-Job abgeschlossen ist
    runs-on: ubuntu-latest
    
    steps:
      - name: Lade Artefakt herunter
        uses: actions/download-artifact@v4
        with:
          name: generated-file
      
      - name: Zeige Dateiinhalt
        run: cat generated-file.txt
      
      - name: Verarbeite Datei
        run: |
          echo "Die Datei wurde verarbeitet." >> generated-file.txt
          echo "Verarbeitungszeitstempel: $(date)" >> generated-file.txt
      
      - name: Zeige bearbeitete Datei
        run: cat generated-file.txt
      
      - name: Speichere die verarbeitete Datei als neues Artefakt
        uses: actions/upload-artifact@v4
        with:
          name: processed-file
          path: generated-file.txt
```

**Erklärung der neuen Elemente:**
- Wir haben nun zwei Jobs: `ersteller` und `verarbeiter`
- `needs: ersteller`: Gibt an, dass der `verarbeiter`-Job erst gestartet wird, wenn der `ersteller`-Job erfolgreich abgeschlossen wurde
- `actions/download-artifact@v4`: Lädt ein Artefakt herunter, das von einem vorherigen Job erstellt wurde

### 5.3 Änderungen hochladen und überprüfen

Speichere deine Änderungen und lade sie auf GitHub hoch:

```bash
git add .github/workflows/hello-world.yml
git commit -m "Füge Multi-Job-Workflow hinzu"
git push origin main
```

Überprüfe die Ausführung auf der GitHub-Oberfläche:
1. Beachte, wie die beiden Jobs nacheinander ausgeführt werden
2. Der `verarbeiter`-Job wartet, bis der `ersteller`-Job abgeschlossen ist
3. Am Ende solltest du zwei Artefakte sehen: `generated-file` und `processed-file`

## 6. Weiterführende Übungen

Du hast jetzt erfolgreich:
1. Ein GitHub-Repository erstellt
2. Eine einfache GitHub Actions Workflow-Datei erstellt
3. Deinen ersten Workflow ausgeführt und überwacht
4. Artefakte erstellt und heruntergeladen
5. Einen Workflow mit mehreren Jobs erstellt

Hier sind einige Ideen für nächste Schritte:

### 6.1 Manuelle Ausführung mit Parametern

Erweitere deinen Workflow, um Parameter bei der manuellen Ausführung zu akzeptieren:

```yaml
name: Parametrisierter Workflow

on:
  workflow_dispatch:
    inputs:
      name:
        description: 'Dein Name'
        required: true
        default: 'GitHub-Benutzer'
      message:
        description: 'Begrüßungsnachricht'
        required: false
        default: 'Hallo von GitHub Actions!'

jobs:
  greet:
    runs-on: ubuntu-latest
    steps:
      - name: Begrüße den Benutzer
        run: |
          echo "Hallo, ${{ github.event.inputs.name }}!"
          echo "Deine Nachricht: ${{ github.event.inputs.message }}"
```

Teste die manuelle Ausführung:
1. Gehe zu deinem Repository auf GitHub
2. Klicke auf "Actions"
3. Wähle "Parametrisierter Workflow" aus der linken Seitenleiste
4. Klicke auf "Run workflow"
5. Gib deinen Namen und eine Nachricht ein
6. Klicke auf "Run workflow" und überprüfe die Ergebnisse

### 6.2 Job-Matrix verwenden

Erweitere deinen Workflow, um einen Job auf verschiedenen Betriebssystemen auszuführen:

```yaml
name: Matrix-Workflow

on:
  workflow_dispatch:

jobs:
  test:
    name: Test auf ${{ matrix.os }}
    runs-on: ${{ matrix.os }}
    strategy:
      matrix:
        os: [ubuntu-latest, windows-latest, macos-latest]
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
      
      - name: Zeige Betriebssystem
        run: |
          echo "Betriebssystem: ${{ matrix.os }}"
          echo "Systeminformationen:"
          if [ "${{ matrix.os }}" == "ubuntu-latest" ]; then
            uname -a
          elif [ "${{ matrix.os }}" == "windows-latest" ]; then
            systeminfo | findstr /B /C:"OS Name" /C:"OS Version"
          elif [ "${{ matrix.os }}" == "macos-latest" ]; then
            sw_vers
          fi
        shell: bash
```

**Hinweis:** Die `if`-Bedingungen verwenden die Bash-Shell, die auf allen Plattformen verfügbar ist.

## 7. Zusammenfassung

In dieser Übung hast du:
- Die Authentifizierung mit GitHub über SSH oder Personal Access Token eingerichtet
- Ein GitHub-Repository erstellt
- Eine GitHub Actions Workflow-Datei erstellt und ausgeführt
- Grundlegende Konzepte wie Jobs, Steps, Artefakte und Job-Abhängigkeiten kennengelernt
- Einen Workflow mit mehreren Jobs erstellt
- Gelernt, wie man Parameter für manuelle Ausführungen hinzufügt
- Eine Job-Matrix für verschiedene Betriebssysteme erstellt

### 7.1 Wichtige Konzepte

- **Workflow**: Eine automatisierte Prozedur, die aus einem oder mehreren Jobs besteht
- **Job**: Eine Gruppe von Steps, die auf demselben Runner ausgeführt werden
- **Step**: Eine einzelne Aufgabe, die Befehle ausführen oder Aktionen verwenden kann
- **Action**: Eine wiederverwendbare Einheit, die häufige Aufgaben vereinfacht
- **Artifact**: Eine Datei oder Sammlung von Dateien, die während eines Workflows produziert werden
- **Runner**: Ein Server, der Jobs ausführt
- **Event**: Ein bestimmtes Ereignis, das einen Workflow auslöst

### 7.2 Fehlerbehebung

Hier sind einige häufige Probleme und ihre Lösungen:

- **Authentifizierungsprobleme**: Überprüfe deine SSH-Keys oder dein Personal Access Token
- **SSH-Key wird abgelehnt**: Stelle sicher, dass der öffentliche Schlüssel in deinem GitHub-Account hinzugefügt wurde
- **Syntax-Fehler**: Überprüfe die Einrückung und Struktur deiner YAML-Datei
- **Workflow wird nicht ausgelöst**: Stelle sicher, dass der Trigger (`on`) korrekt konfiguriert ist
- **Job schlägt fehl**: Überprüfe die Protokolle für Fehlerdetails
- **Artefakt nicht verfügbar**: Stelle sicher, dass der Upload-Schritt erfolgreich war

> **Tipp:** Im nächsten Level werden wir eine echte Anwendung bauen und automatisierte Tests hinzufügen.
