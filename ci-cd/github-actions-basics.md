# CI/CD Pipeline mit GitHub Actions

Diese Unterlagen sind deine umfassende Einführung in das Thema **CI/CD Pipeline mit GitHub Actions**. Hier findest du alle wichtigen Informationen und Erklärungen, um die Konzepte hinter Continuous Integration und Deployment zu verstehen, eigene Workflows zu erstellen und dich optimal auf Übung 1 („Hello World“-Workflow) vorzubereiten.

> **Hinweis:** CI/CD steht für Continuous Integration und Continuous Deployment. Mit automatisierten Pipelines sorgst du dafür, dass dein Code bei jedem Push oder Pull Request getestet, gebaut und – bei Bedarf – automatisch ausgeliefert wird. GitHub Actions ist ein integriertes Tool in GitHub, das dir genau diese Automatisierung ermöglicht.

---

## 1. Was sind GitHub Actions?

GitHub Actions ist ein leistungsfähiges Automatisierungstool, das direkt in GitHub integriert ist. Anstelle von allgemeiner CI/CD-Beschreibung wollen wir uns hier darauf konzentrieren, was GitHub Actions für dich leistet und wie es deinen Entwicklungsalltag erleichtert.

Mit GitHub Actions kannst du wiederkehrende Aufgaben—wie das Ausführen von Tests, das Bauen von Artefakten oder das Ausrollen in Umgebungen—vollständig in deinem Repository definieren. Workflows werden in YAML-Dateien abgelegt und dort versioniert, sodass dein Automatisierungsprozess genauso nachvollziehbar und überprüfbar ist wie dein Quellcode. Jede Änderung an der Pipeline wird somit historisch dokumentiert und kann über Pull Requests begutachtet werden.

Ein entscheidender Vorteil ist die enge Verzahnung mit allen GitHub-Funktionen: Du kannst auf Events wie Push, Pull Request, Issues oder Releases reagieren und in der gleichen Infrastruktur Aktionen ausführen, die sonst über externe CI-Services gelöst werden müssten. Zusätzlich bietet GitHub Actions ein großes Ökosystem an vorgefertigten Aktionen („Actions“), die du direkt in deine Workflows einbinden kannst. Ob Code auschecken, Node.js-Umgebung einrichten oder Docker-Container starten – viele gängige Schritte sind bereits als Actions verfügbar.

Durch die Verwendung von selbstgehosteten Runnern kannst du deine Pipeline sogar auf deiner eigenen Hardware betreiben, wenn du spezielle Anforderungen hast. Gleichzeitig stellt GitHub kostenlose und kostenpflichtige gehostete Runner bereit, die du ohne zusätzliche Einrichtung nutzen kannst. Alles, was du brauchst, ist ein Repository und du legst im Verzeichnis `.github/workflows` los.

---

## 2. Deine erste Workflow-Datei verstehen

Ein Workflow in GitHub Actions besteht im Kern aus mehreren Bestandteilen, die du in einer YAML-Datei zusammenfügst. Zunächst gibst du unter `name` deinem Workflow einen aussagekräftigen Titel und definierst unter `on`, welche Ereignisse (z.B. Push, Pull Request oder manuelle Auslösung) die Pipeline starten. Mit dem Block `env` legst du global verfügbare Umgebungsvariablen fest, damit du häufig verwendete Werte nur einmal pflegen musst.

Der zentrale Abschnitt ist `jobs`, in dem du einzelne Phasen deines Ablaufs beschreibst. Jeder Job läuft auf einem Runner, den du über `runs-on` festlegst (etwa `ubuntu-latest` für Linux oder `windows-latest` für Windows). Innerhalb eines Jobs definierst du unter `steps` die genauen Arbeitsschritte. Ein Schritt kann entweder eine vordefinierte Action mit `uses` einbinden oder mit `run` eigene Shell-Befehle ausführen. Wenn du für eine Action zusätzliche Parameter benötigst, übergibst du sie im Unterabschnitt `with`.

Um die Ausführung mehrerer Jobs zu steuern, verwendest du das Keyword `needs`, damit ein Job erst startet, wenn ein anderer erfolgreich abgeschlossen ist. Über die `if`-Expression kannst du außerdem Bedingungen formulieren, zum Beispiel, dass ein Deployment nur auf dem `main`-Branch oder bei bestimmten Tags erfolgen soll.

Mit einer Matrix-Strategie lässt sich dein Code zeitgleich auf unterschiedlichen Betriebssystemen, Node.js-Versionen oder anderen Kombinationen testen. Du definierst die Parameter in einer Matrix, und GitHub Actions erstellt automatisch die einzelnen Jobs für jede Kombination. So erhältst du umfassende Tests, ohne deine Konfiguration mehrfach kopieren zu müssen.

---

## 3. Architektur einer CI/CD-Pipeline mit GitHub Actions

### 3.0 Visuelles Beispiel
```text
                 +-------------+
      Push       |  Trigger    |        Workflow startet bei Push, Pull Request oder Zeitplan
    ───────────► |  (on)       |                      |
                 +-------------+                      |
                        |                             |
                        v                             v
               +---------------+               +---------------+
               | Build & Test  |               |   Schedule    |
               |   Job         |◄──────────────| (cron-Timer)  |
               +---------------+               +---------------+
                        |                             |
                        v                             v
               +---------------+               +---------------+
               |  Deploy Job   |               | Manuelle Run  |
               |               |               | (workflow_    |
               +---------------+               | dispatch)     |
                        |                       +---------------+
                        v
                  +------------+
                  | Artifacts  |
                  | (Upload)   |
                  +------------+
```

Diese schematische Darstellung zeigt, wie ein einfacher Workflow abläuft: Ein Push oder Pull Request (oder ein Zeitplan bzw. manuelle Ausführung) löst den Workflow aus, der zuerst den Build-&-Test-Job startet und danach das Deployment durchführt. Am Ende werden Artefakte hochgeladen, die später heruntergeladen werden können.

### 3.1 Trigger definieren
```yaml
# Workflow-Trigger Konfiguration
on:                            # Definiert, welche Events den Workflow starten
  push:                        # Auslöser: Push-Events
    branches:                  # Auf den folgenden Branches
      - main                   # Haupt-Branch
  pull_request:                # Auslöser: Pull Requests auf main
  schedule:                    # Zeitgesteuerte Ausführung
    - cron: '0 2 * * *'        # Täglich um 02:00 Uhr
```

### 3.2 Build & Test Job
```yaml
# Build- und Test-Job Definition
jobs:
  build:                       # Job-Name: build
    runs-on: ubuntu-latest     # Runner-Umgebung: Ubuntu (neuste Version)
    steps:                     # Schritte in diesem Job
      - uses: actions/checkout@v4       # Holt den Code aus dem Repository
      - name: Dependencies installieren  # Beschreibung des Schritts
        run: npm ci                    # Installiert Node.js-Abhängigkeiten
      - name: Tests ausführen          # Beschreibung des Schritts
        run: npm test                  # Führt Unit-Tests aus
```

### 3.3 Deployment-Job
```yaml
# Deployment-Job Definition
  deploy:                      # Job-Name: deploy
    needs: build               # Wartet auf erfolgreichen Abschluss des build-Jobs
    runs-on: ubuntu-latest     # Runner-Umgebung: Ubuntu (neuste Version)
    steps:                     # Schritte in diesem Job
      - uses: actions/checkout@v4       # Holt den Code erneut aus dem Repository
      - name: Deployment-Skript         # Beschreibung des Schritts
        run: ./deploy.sh                # Führt das Deploy-Skript aus
```

---

## 4. Einrichtung & Voraussetzungen

Bevor du loslegst, überprüfe folgende Punkte:

1. **GitHub-Account** – kostenlos oder kostenpflichtig, je nach Projektanforderung.
2. **Git lokal installiert:**
   ```bash
   git --version
   ```
3. **Repository** – neu erstellt oder bestehendes Projekt.
4. **Zugriffstoken (PAT)** – bei privaten Repositories mit `repo`-Rechten.
5. **IDE/Editor** – z. B. VS Code, IntelliJ, Sublime.

Nutze SSH oder HTTPS, um dein Repository lokal zu klonen:

```bash
# Klonen via HTTPS
git clone https://github.com/dein-username/dein-repo.git
# oder via SSH
git clone git@github.com:dein-username/dein-repo.git
```

Wechsle in das Verzeichnis:

```bash
cd dein-repo
```

---

## 5. Wichtige Konzepte & Beispiele

### 5.1 Workflow-Datei

- **Pfad:** `.github/workflows/<name>.yml`
- **Name:** Wird in der GitHub Actions UI angezeigt.

```yaml
name: "Hello World Workflow"
on:
  push:
    branches: [ main ]
jobs:
  hello:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Hallo sagen
        run: echo "Hello, World!"
```

**Erklärung:**

- `uses: actions/checkout@v4` lädt deinen Code in die Laufumgebung.
- `run:` führt Shell-Befehle aus.

### 5.2 Umgebungsvariablen & Secrets

Umgebungsvariablen vereinfachen Wiederverwendbarkeit:

```yaml
env:
  NODE_ENV: production
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - run: echo "Environment: ${{ env.NODE_ENV }}"
```

Secrets hinterlegst du im GitHub-Repo unter **Settings → Secrets & variables**. Sie werden nie im Klartext in Logs ausgegeben:

```yaml
- name: API-Anfrage mit Secret
  run: |
    curl -H "Authorization: token ${{ secrets.MY_TOKEN }}" \
         https://api.github.com/repos/${{ github.repository }}
```

### 5.3 Artefakte

Artefakte sind nützlich, um Build‑Ergebnisse oder Logs zu speichern:

- Upload:
  ```yaml
  - name: Artefakt speichern
    uses: actions/upload-artifact@v4
    with:
      name: build-output
      path: dist/
  ```
- Download:
  ```yaml
  - name: Artefakt herunterladen
    uses: actions/download-artifact@v4
    with:
      name: build-output
  ```

---

## 6. Syntax & Aufbau einer GitHub Actions Workflow-Datei

In einer GitHub Actions Workflow-Datei beschreibst du in YAML-Notation, wie dein Automatisierungsprozess ablaufen soll. Die Datei beginnt stets mit einer `name`-Angabe, die als Titel deines Workflows in der GitHub-Oberfläche erscheint. Direkt darunter folgt der Abschnitt `on`, der festlegt, welche Ereignisse—sei es ein Push auf einen bestimmten Branch, ein Pull Request oder eine manuelle Auslösung via `workflow_dispatch`—die Pipeline starten.

Unter `env` kannst du globale Umgebungsvariablen definieren, die in allen nachfolgenden Jobs und Schritten automatisch zur Verfügung stehen. Dies eignet sich beispielsweise, um eine Umgebungsbezeichnung wie `production` oder `staging` einmal zentral festzulegen.

Der Kern deines Workflows befindet sich im Abschnitt `jobs`. Jeder Job steht für eine Phase deines CI/CD-Prozesses und läuft auf einem sogenannten Runner, den du mit `runs-on` spezifizierst. Üblicherweise wählst du hier images wie `ubuntu-latest`, `windows-latest` oder `macos-latest`, um deine Anwendung in unterschiedlichen Umgebungen zu prüfen.

Innerhalb eines Jobs definierst du unter `steps` die einzelnen Arbeitsschritte. Ein Schritt kann entweder eine externe Aktion („Action“) einbinden, indem du `uses: owner/action@version` angibst, oder einen eigenen Shell-Befehl ausführen, indem du `run: dein-shell-befehl` nutzt. Wenn eine Aktion zu konfigurieren ist, schreibst du unter `with` entsprechende Parameter, etwa Pfade oder Zugangstoken.

Um Abläufe von Jobs zu verketten, verwendest du das Schlüsselwort `needs`. Damit wartest du darauf, dass ein anderer Job erfolgreich beendet ist, bevor dein aktueller gestartet wird. Häufig tritt dies bei Deployment-Jobs auf, die erst nach einem erfolgreichen Build und Test ausgeführt werden.

Für spezielle Bedingungen steht dir die `if`-Expression zur Verfügung. Damit kannst du etwa festlegen, dass bestimmte Schritte nur für Tags oder nur auf dem `main`-Branch laufen. Die Syntax lautet in geschweiften Klammern `${{ }}`, innerhalb derer du Funktionen wie `startsWith` oder Vergleiche wie `github.ref == 'refs/heads/main'` verwenden kannst.

Wenn du deine Tests oder Builds auf mehreren Umgebungen und Versionen parallel ausführen möchtest, hilft dir die Matrix-Strategie. Du legst eine Kombination von Parametern fest, etwa verschiedene Betriebssysteme und Node.js-Versionen und GitHub Actions erzeugt daraus automatisch einzelne Jobs für jede Kombination. Auf diese Weise erhältst du umfassende Testergebnisse, ohne deine Workflow-Datei mehrfach kopieren zu müssen.

Im Zusammenspiel dieser Elemente entsteht eine übersichtliche, modular aufgebaute Pipeline, die Transparenz schafft und dir erlaubt, jeden Schritt deines Entwicklungsprozesses zu automatisieren und nachzuvollziehen.

---

## 7. Best Practices
Im Folgenden findest du einige Empfehlungen, um deine Workflows stabil, performant und sicher zu gestalten:

- **Modularität:** Baue deine Pipeline in überschaubaren, wiederverwendbaren Dateien auf. So behältst du den Überblick und kannst Teile unabhängig testen und anpassen.
- **Caching:** Nutze `actions/cache`, um wiederkehrende Abhängigkeitsinstallationen zu beschleunigen und Laufzeiten zu verkürzen.
- **Least Privilege:** Erstelle und verwende nur die unbedingt notwendigen Secrets und Tokens. Vermeide weitreichende Zugriffsrechte.
- **Versionierung:** Referenziere Actions mit festen Versionen (z. B. `@v4`). So bleibt dein Workflow stabil, selbst wenn sich der Haupt-Branch einer Action ändert.
- **Concurreny-Control:** Setze `concurrency`, um gleichzeitige Workflow-Runs zu steuern und Konflikte beim Zugriff auf dieselben Ressourcen zu vermeiden.
- **Logs & Dokumentation:** Vergib sprechende Namen für Steps und kommentiere komplexe Abläufe. Eine gute Dokumentation hilft dir und deinem Team, die Pipeline schnell zu verstehen.

---

## 8. Beispiel: Vollständige Workflow-Datei mit Kommentaren

Im Folgenden siehst du eine komplette `hello-world.yml`-Datei, Zeile für Zeile kommentiert, damit du verstehst, was jede Zeile bewirkt:

```yaml
# 1. Name des Workflows, angezeigt in der GitHub Actions UI
name: "Hello World Workflow"

# 2. Trigger-Ereignisse, die den Workflow starten
on:
  push:                          # Bei jedem Push in den definierten Branches
    branches:
      - main                     # Nur auf dem 'main'-Branch
  pull_request:                  # Bei Pull Requests gegen 'main'
  workflow_dispatch:             # Ermöglicht manuelles Auslösen in der UI

# 3. Globale Umgebungsvariablen für alle Jobs und Schritte
env:
  GREETING: "Hello, GitHub Actions!"

jobs:
  # 4. Definiert den 'hello'-Job
  hello:
    runs-on: ubuntu-latest       # 5. Läuft auf Ubuntu-Linux

    steps:
      # 6. Holt den Code aus dem Repository
      - name: Checkout code
        uses: actions/checkout@v4

      # 7. Gibt die Begrüßung aus der Umgebungsvariable aus
      - name: Begrüßung ausgeben
        run: echo "${{ env.GREETING }}"

      # 8. Zeigt Details zum GitHub-Kontext an
      - name: GitHub-Kontext zeigen
        run: |
          echo "Repository: ${{ github.repository }}"
          echo "Event: ${{ github.event_name }}"
          echo "Branch: ${{ github.ref }}"

      # 9. Erstellt eine Datei im Arbeitsverzeichnis und schreibt ein Datum hinein
      - name: Datei generieren
        run: |
          echo "Diese Datei wurde durch GitHub Actions erstellt." > generated-file.txt
          echo "Zeitstempel: $(date)" >> generated-file.txt

      # 10. Lädt das generierte Artefakt hoch
      - name: Artefakt hochladen
        uses: actions/upload-artifact@v4
        with:
          name: generated-file
          path: generated-file.txt
```

### Erklärung der Syntax anhand des Beispiels

Die obige Workflow-Datei nutzt die zentrale YAML-Struktur von GitHub Actions: Zunächst wird über `name` und `on` festgelegt, wie der Workflow heißt und wann er ausgeführt wird. Unter `env` definierst du wiederverwendbare Variablen wie `GREETING`, die du in den Schritten über `${{ env.VARIABLE }}` aufrufst.

Im Abschnitt `jobs` folgen die einzelnen Jobs. Jeder Job beginnt mit seinem Namen (`hello`) und der Angabe `runs-on`, die bestimmt, auf welchem Betriebssystem (hier `ubuntu-latest`) er läuft. Die `steps` gliedern den Job weiter, indem sie Aktionen (`uses`) oder Shell-Befehle (`run`) ausführen. Durch das Einbinden der Action `actions/checkout@v4` stellst du sicher, dass dein Repository-Code in der Laufumgebung vorliegt.

Mehrzeilige Shell-Befehle realisierst du mit der Pipe-Syntax `|`, wie beim Schreiben in `generated-file.txt`. Um Dateien oder Ergebnisse zwischen Jobs zu übertragen, nutzt du `actions/upload-artifact@v4`, das Artefakte für spätere Schritte oder Downloads speichert.

Mit dieser kommentierten Komplettansicht und der begleitenden Erklärung hast du nun ein vollständiges Beispiel für einen einfachen „Hello World“-Workflow sowie das Verständnis dafür, wie jede YAML-Zeile ihre Wirkung entfaltet. Viel Spaß beim Ausprobieren und Anpassen!

