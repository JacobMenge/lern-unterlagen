# Continuous Deployment einer React-Anwendung mit GitHub Actions

In dieser Übung wirst du eine vollständige CI/CD-Pipeline für eine einfache React-Anwendung erstellen. Du wirst lernen, wie du deine Anwendung automatisch in verschiedene Umgebungen (Staging und Produktion) deployst und wie du das Deployment von Pull Requests abhängig machst.

> **Hinweis:** Continuous Deployment (CD) ist der Prozess, bei dem Code automatisch in Produktionsumgebungen bereitgestellt wird, nachdem er die CI-Pipeline erfolgreich durchlaufen hat. Dies ermöglicht schnellere Releases, reduziert manuelle Fehler und bietet ein besseres Feedback für dein Entwicklungsteam.

## Voraussetzungen

Stelle sicher, dass folgende Tools auf deiner Ubuntu WSL installiert sind:

```bash
# Überprüfe Git-Installation
git --version
# Falls nicht installiert:
# sudo apt update
# sudo apt install git

# Überprüfe Node.js & npm Installation
node --version  # Sollte 18.x oder neuer sein
npm --version   # Sollte 8.x oder neuer sein

# Falls Node.js aktualisiert werden muss:
curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -
sudo apt install -y nodejs
```

## 1. Eine einfache React-Anwendung erstellen

Zunächst erstellen wir eine einfache React-Anwendung, die wir später deployen werden.

### 1.1 React-Projekt initialisieren

```bash
# Erstelle ein neues React-Projekt
npx create-react-app react-deployment-demo
cd react-deployment-demo

# Initialisiere Git
git init
git add .
git commit -m "Initial commit"

# Verbinde mit GitHub
# Erstelle zuerst ein neues leeres Repository auf GitHub und führe dann aus:
git remote add origin https://github.com/dein-username/react-deployment-demo.git
git branch -M main  # Benenne den Branch in 'main' um, falls er noch 'master' heißt
git push -u origin main
```

**Erklärung:**
- `create-react-app` erstellt eine komplette React-Anwendung mit allen nötigen Konfigurationen
- `git init` initialisiert ein lokales Git-Repository
- `git remote add origin...` verbindet dein lokales Repository mit GitHub
- `git branch -M main` benennt deinen Hauptbranch in "main" um (falls er noch "master" heißt)
- `git push -u origin main` lädt deinen Code auf GitHub hoch

### 1.2 Anwendung anpassen

Öffne `src/App.js` und ersetze den Inhalt durch:

```jsx
import React from 'react';
import './App.css';

function App() {
  const environment = process.env.REACT_APP_ENVIRONMENT || 'development';
  const version = process.env.REACT_APP_VERSION || '1.0.0';
  const buildNumber = process.env.REACT_APP_BUILD_NUMBER || 'local';
  
  return (
    <div className="App">
      <header className="App-header">
        <h1>React Deployment Demo</h1>
        <p>
          Diese Anwendung demonstriert Continuous Deployment mit GitHub Actions.
        </p>
        <div className="environment-info">
          <p>Umgebung: <span className={`env-${environment}`}>{environment}</span></p>
          <p>Version: {version}</p>
          <p>Build: {buildNumber}</p>
        </div>
      </header>
    </div>
  );
}

export default App;
```

Ersetze den Inhalt von `src/App.css` durch:

```css
.App {
  text-align: center;
}

.App-header {
  background-color: #282c34;
  min-height: 100vh;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  font-size: calc(10px + 2vmin);
  color: white;
}

.environment-info {
  margin-top: 2rem;
  padding: 1rem;
  border: 1px solid #61dafb;
  border-radius: 8px;
  background-color: rgba(97, 218, 251, 0.1);
}

.env-development {
  color: yellow;
}

.env-staging {
  color: orange;
}

.env-production {
  color: #61dafb;
}
```

**Erklärung:**
- Wir passen die Standard-App an, um Umgebungsinformationen anzuzeigen
- Die CSS-Klassen `.env-development`, `.env-staging` und `.env-production` färben die Umgebungsnamen je nach Umgebung unterschiedlich

### 1.3 Umgebungsvariablen einrichten

Erstelle eine `.env.development`-Datei im Hauptverzeichnis:

```
REACT_APP_ENVIRONMENT=development
REACT_APP_VERSION=0.1.0
REACT_APP_BUILD_NUMBER=local
```

**Erklärung:**
- Diese Datei setzt Umgebungsvariablen für die lokale Entwicklung
- Create React App liest automatisch `.env`-Dateien und stellt die Variablen zur Verfügung
- Alle Variablen müssen mit `REACT_APP_` beginnen, um in der Anwendung verfügbar zu sein

### 1.4 Teste die Anwendung lokal

```bash
# Starte den Entwicklungsserver
npm start
```

Öffne deinen Browser und gehe zu http://localhost:3000. Du solltest deine angepasste React-Anwendung sehen.

> **Hinweis für WSL-Nutzer:** Falls der Browser nicht automatisch startet oder die Seite nicht erreichbar ist, versuche, die URL `http://localhost:3000` im Windows-Browser zu öffnen. Wenn das nicht funktioniert, könnte das an Port-Forwarding-Problemen liegen. Eine alternative Lösung ist, in WSL `BROWSER=none npm start` auszuführen und dann die lokale IP deiner WSL-Instanz zu verwenden (finde diese mit `ip addr show`).

### 1.5 Änderungen committen

```bash
git add .
git commit -m "Anpassung der React-App für Deployment-Demo"
git push origin main
```

**Erklärung:**
- `git add .` fügt alle Änderungen zur Staging-Area hinzu
- `git commit -m "..."` erstellt einen Commit mit einer beschreibenden Nachricht
- `git push origin main` lädt die Änderungen auf GitHub hoch

## 2. CI-Pipeline für die React-Anwendung einrichten

Als Nächstes richten wir eine CI-Pipeline ein, um unsere React-Anwendung zu testen und zu bauen.

### 2.1 Workflow-Verzeichnis erstellen

```bash
mkdir -p .github/workflows
```

**Erklärung:**
- `mkdir -p` erstellt Verzeichnisse und alle notwendigen übergeordneten Verzeichnisse
- `.github/workflows` ist der standardmäßige Speicherort für GitHub Actions Workflow-Dateien

### 2.2 CI-Workflow erstellen

Erstelle eine Datei `.github/workflows/ci.yml`:

```yaml
name: CI

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  build:
    name: Build
    runs-on: ubuntu-latest
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
      
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'
      
      - name: Install dependencies
        run: npm ci
      
      - name: Build
        run: npm run build
        env:
          REACT_APP_ENVIRONMENT: production
          REACT_APP_VERSION: ${{ github.ref_name }}
          REACT_APP_BUILD_NUMBER: ${{ github.run_number }}
      
      - name: Upload build artifacts
        uses: actions/upload-artifact@v4
        with:
          name: build
          path: build/
          retention-days: 7
```

**Erklärung:**
- `name: CI` definiert den Namen des Workflows
- `on:` legt fest, wann der Workflow ausgeführt wird (bei Pushes und Pull Requests auf main)
- `jobs:` definiert die auszuführenden Jobs
- `build:` ist unser Job zum Bauen der Anwendung
- `steps:` enthält die einzelnen Schritte des Jobs:
  - `checkout code` lädt den Code aus dem Repository
  - `setup Node.js` richtet Node.js ein
  - `install dependencies` installiert die Abhängigkeiten mit `npm ci` (schneller als `npm install`)
  - `build` baut die Anwendung mit Umgebungsvariablen
  - `upload build artifacts` speichert das Build-Ergebnis, damit andere Workflows darauf zugreifen können

### 2.3 Commit und Push der CI-Konfiguration

```bash
git add .github/workflows/ci.yml
git commit -m "Füge CI-Workflow hinzu"
git push origin main
```

### 2.4 CI-Workflow überprüfen

Gehe zu deinem GitHub-Repository und überprüfe den Tab "Actions". Du solltest sehen, dass der CI-Workflow ausgeführt wird.

**Erklärung:**
- Nach dem Push wird der Workflow automatisch ausgeführt
- Du kannst den Fortschritt und die Ergebnisse im "Actions"-Tab verfolgen
- Ein grünes Häkchen bedeutet, dass der Workflow erfolgreich war

## 3. Deployment auf GitHub Pages einrichten

Für unser erstes Deployment verwenden wir GitHub Pages, da es einfach einzurichten ist und keine externen Dienste erfordert.

### 3.1 gh-pages-Paket installieren

```bash
npm install --save-dev gh-pages
```

**Erklärung:**
- `gh-pages` ist ein npm-Paket, das das Deployment auf GitHub Pages vereinfacht
- `--save-dev` speichert es als Entwicklungsabhängigkeit, da es nur für den Build-Prozess benötigt wird

### 3.2 package.json aktualisieren

Öffne `package.json` und füge folgende Zeilen hinzu:

```json
"homepage": "https://dein-username.github.io/react-deployment-demo",
"scripts": {
  "start": "react-scripts start",
  "build": "react-scripts build",
  "test": "react-scripts test",
  "eject": "react-scripts eject",
  "predeploy": "npm run build",
  "deploy": "gh-pages -d build"
},
```

Ersetze `dein-username` durch deinen GitHub-Benutzernamen.

**Erklärung:**
- `homepage` teilt Create React App mit, wo die App gehostet wird
- `predeploy` wird automatisch vor `deploy` ausgeführt und baut die Anwendung
- `deploy` veröffentlicht den Build-Ordner auf GitHub Pages

### 3.3 CD-Workflow für GitHub Pages erstellen

Erstelle eine Datei `.github/workflows/deploy-gh-pages.yml`:

```yaml
name: Deploy to GitHub Pages

on:
  # Wird nur ausgeführt, wenn der CI-Workflow erfolgreich war
  workflow_run:
    workflows: ["CI"]
    branches: [main]
    types: [completed]

jobs:
  deploy:
    name: Deploy
    runs-on: ubuntu-latest
    # Nur ausführen, wenn der vorherige Workflow erfolgreich war
    if: ${{ github.event.workflow_run.conclusion == 'success' }}
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
      
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'
      
      - name: Install dependencies
        run: npm ci
      
      - name: Set environment variables
        run: |
          echo "REACT_APP_ENVIRONMENT=production" >> .env.production
          echo "REACT_APP_VERSION=${{ github.ref_name }}" >> .env.production
          echo "REACT_APP_BUILD_NUMBER=${{ github.run_number }}" >> .env.production
      
      - name: Build
        run: npm run build
      
      - name: Deploy to GitHub Pages
        uses: JamesIves/github-pages-deploy-action@v4
        with:
          folder: build
          branch: gh-pages
          clean: true
```

**Erklärung:**
- `on: workflow_run:` führt diesen Workflow aus, nachdem der CI-Workflow abgeschlossen ist
- `if: github.event.workflow_run.conclusion == 'success'` stellt sicher, dass der CI-Workflow erfolgreich war
- Die Build-Schritte sind ähnlich wie im CI-Workflow
- `Deploy to GitHub Pages` verwendet die JamesIves-Action, um die Anwendung zu deployen:
  - `folder: build` gibt den zu deployenden Ordner an
  - `branch: gh-pages` gibt den Branch an, in den deployt werden soll
  - `clean: true` löscht vorhandene Dateien im gh-pages Branch

### 3.4 Commit und Push der Deployment-Konfiguration

```bash
git add .github/workflows/deploy-gh-pages.yml package.json
git commit -m "Füge GitHub Pages Deployment hinzu"
git push origin main
```

### 3.5 GitHub Pages aktivieren

1. Gehe zu deinem Repository auf GitHub
2. Klicke auf "Settings"
3. Scrolle nach unten zu "Pages"
4. Unter "Source", wähle "Deploy from a branch" und dann den Branch "gh-pages" und klicke auf "Save"

**Erklärung:**
- GitHub muss wissen, von welchem Branch die Seite bereitgestellt werden soll
- Der gh-pages Branch wird von unserem Workflow erstellt und enthält den Build-Ordner

### 3.6 Überprüfe das Deployment

Nachdem der Workflow abgeschlossen ist, kannst du deine Anwendung unter `https://dein-username.github.io/react-deployment-demo` aufrufen. Ersetze `dein-username` durch deinen GitHub-Benutzernamen.

> **Hinweis:** Es kann einige Minuten dauern, bis die Seite verfügbar ist, nachdem der Workflow abgeschlossen ist.

## 4. Staging- und Produktions-Deployments mit AWS einrichten

Es gibt viele Anbieter wie Netlify oder Vercel, mit denen man React-Anwendungen einfach deployen kann. In dieser Anleitung konzentrieren wir uns jedoch auf AWS, da es eine flexible und kosteneffiziente Lösung bietet, die häufig in Unternehmen eingesetzt wird.

### 4.1 AWS Amplify für kostengünstiges React-Deployment

Wir verwenden AWS Amplify, da es eine einfache und kostengünstige Möglichkeit bietet, React-Anwendungen zu hosten, ohne Server verwalten zu müssen.

### 4.2 AWS Konto einrichten

1. Falls du noch kein AWS-Konto hast, erstelle eines unter [aws.amazon.com](https://aws.amazon.com/)
2. Melde dich bei der AWS Management Console an
3. Navigiere zur Amplify-Konsole (suche nach "Amplify" in der Suchleiste oben)

> **Wichtig:** AWS verlangt Kreditkarteninformationen, auch für das kostenlose Kontingent. Achte darauf, die Kostenlimits zu beachten und deine Nutzung zu überwachen.

### 4.3 Develop-Branch für Staging erstellen

Wir erstellen einen `develop`-Branch, der als Basis für unsere Staging-Umgebung dient:

```bash
# Wechsle zum main-Branch und hole die neuesten Änderungen
git checkout main
git pull

# Erstelle und wechsle zum develop-Branch
git checkout -b develop

# Pushe den develop-Branch zu GitHub
git push -u origin develop
```

**Erklärung:**
- Der `develop`-Branch dient als Zwischenschritt, bevor Änderungen in den `main`-Branch gelangen
- Dadurch können wir Änderungen in einer Staging-Umgebung testen, bevor sie in die Produktion gehen

### 4.4 Amplify für Produktion einrichten

1. Klicke in der Amplify-Konsole auf "Neue App erstellen"
2. Wähle "GitHub" als Repository-Quelle
3. Klicke auf "GitHub Connect" und folge den Anweisungen zur Authentifizierung
4. Wähle dein Repository aus der Liste aus
5. Wähle den Branch `main` für die Produktion
6. Konfiguriere die Build-Einstellungen (Amplify sollte sie automatisch erkennen)
   - Falls nicht, nutze folgende Build-Einstellungen:

```yaml
version: 1
frontend:
  phases:
    preBuild:
      commands:
        - npm ci
    build:
      commands:
        - npm run build
  artifacts:
    baseDirectory: build
    files:
      - '**/*'
  cache:
    paths:
      - node_modules/**/*
```

7. Füge folgende Umgebungsvariablen hinzu (unter "Erweiterte Einstellungen"):
   - `REACT_APP_ENVIRONMENT`: `production`
   - `REACT_APP_VERSION`: `1.0.0`

8. Klicke auf "Weiter" und dann auf "Speichern und bereitstellen"

**Erklärung:**
- Amplify verbindet sich mit deinem GitHub-Repository
- Die Build-Einstellungen sagen Amplify, wie die App gebaut werden soll
- Die Umgebungsvariablen werden während des Builds verwendet
- Amplify baut und deployt deine App automatisch

### 4.5 Amplify für Staging einrichten

1. Gehe zur Amplify-Konsole und wähle deine App aus
2. Klicke auf "Hosting-Umgebungen" in der linken Seitenleiste
3. Klicke auf "Branch hinzufügen"
4. Wähle den Branch `develop` aus
5. Behalte die gleichen Build-Einstellungen wie für die Produktion bei
6. Füge folgende Umgebungsvariablen hinzu:
   - `REACT_APP_ENVIRONMENT`: `staging`
   - `REACT_APP_VERSION`: `1.0.0`
7. Klicke auf "Weiter" und dann auf "Speichern und bereitstellen"

**Erklärung:**
- Wir erstellen eine separate Umgebung für den `develop`-Branch
- Diese Umgebung verwendet `staging` als Umgebungsvariable
- Änderungen im `develop`-Branch werden automatisch in dieser Umgebung deployt

### 4.6 Pull Request Vorschauen einrichten

1. Gehe zur Amplify-Konsole und wähle deine App aus
2. Klicke auf "Hosting-Umgebungen" in der linken Seitenleiste
3. Klicke auf "Vorschau-Einstellungen" in der oberen Leiste
4. Aktiviere "Pull-Request-Vorschauen"
5. Wähle "Automatisch PRs für alle Branches erstellen"
6. Klicke auf "Speichern"

**Erklärung:**
- Amplify erstellt automatisch temporäre Deployments für Pull Requests
- Dies ermöglicht die Überprüfung von Änderungen, bevor sie in `develop` oder `main` gemergt werden
- Die URL für diese Vorschau wird als Kommentar zum Pull Request hinzugefügt

## 5. Feature-Branch erstellen und Pull Request öffnen

### 5.1 Feature-Branch erstellen

```bash
# Stelle sicher, dass du auf dem develop-Branch bist
git checkout develop

# Erstelle und wechsle zum feature-Branch
git checkout -b feature/add-footer
```

**Erklärung:**
- Wir erstellen einen Feature-Branch basierend auf `develop`
- Der Name `feature/add-footer` gibt an, wofür der Branch ist
- Feature-Branches sollten immer von `develop` abzweigen und dorthin zurückgeführt werden

### 5.2 Footer zur App hinzufügen

Öffne `src/App.js` und füge einen Footer hinzu (nach dem `</header>`-Tag):

```jsx
<footer className="App-footer">
  <p>© {new Date().getFullYear()} React Deployment Demo</p>
  <p>
    <small>Deployment Environment: {environment}</small>
  </p>
</footer>
```

Füge auch Styling für den Footer in `src/App.css` hinzu:

```css
.App-footer {
  background-color: #20232a;
  padding: 1rem;
  color: #ffffff;
  position: fixed;
  bottom: 0;
  width: 100%;
  text-align: center;
}
```

**Erklärung:**
- Wir fügen einen Footer zur App hinzu
- `{new Date().getFullYear()}` zeigt das aktuelle Jahr dynamisch an
- Die CSS-Klasse `.App-footer` stylt den Footer

### 5.3 Committe und pushe die Änderungen

```bash
git add src/App.js src/App.css
git commit -m "Füge Footer hinzu"
git push -u origin feature/add-footer
```

**Erklärung:**
- `git add` fügt die geänderten Dateien zum Commit hinzu
- `git commit` erstellt einen Commit mit einer beschreibenden Nachricht
- `git push -u origin feature/add-footer` lädt den neuen Branch auf GitHub hoch

### 5.4 Pull Request erstellen

Gehe zu deinem GitHub-Repository und erstelle einen Pull Request für den `feature/add-footer`-Branch.

1. Du solltest eine Meldung sehen: "Compare & pull request" - klicke darauf
2. Stelle sicher, dass der Base-Branch `develop` ist (nicht `main`)
3. Gib einen Titel ein (z.B. "Add footer to the application")
4. Klicke auf "Create pull request"

**Erklärung:**
- Ein Pull Request (PR) ist eine Anfrage, Änderungen aus einem Branch in einen anderen zu mergen
- Der Base-Branch (`develop`) ist der Ziel-Branch, in den gemergt werden soll
- Der Compare-Branch (`feature/add-footer`) enthält die Änderungen
- GitHub zeigt automatisch die Unterschiede zwischen den Branches an

### 5.5 Überprüfe die Deployments

1. Der CI-Workflow wird automatisch für den Pull Request ausgeführt
2. AWS Amplify erstellt eine Vorschau für deinen Pull Request
3. Nach kurzer Zeit solltest du in deinem PR einen Kommentar von AWS Amplify mit der Vorschau-URL sehen
4. Öffne die URL und überprüfe, ob der Footer korrekt angezeigt wird

**Erklärung:**
- Der automatische Build und die Vorschau sind wichtige Teile des CI/CD-Prozesses
- Sie ermöglichen es, Änderungen zu sehen, bevor sie gemergt werden
- Dies hilft, Probleme früh zu erkennen und zu beheben

### 5.6 Pull Request mergen

Wenn alles in Ordnung ist:

1. Klicke auf "Merge pull request"
2. Wähle "Create a merge commit" als Merge-Methode
3. Klicke auf "Confirm merge"

**Erklärung:**
- Der Merge führt die Änderungen aus dem Feature-Branch in den `develop`-Branch zusammen
- "Create a merge commit" erstellt einen zusätzlichen Commit, der die Zusammenführung dokumentiert
- Nach dem Merge kann der Feature-Branch gelöscht werden (GitHub bietet dies an)

### 5.7 Überprüfe das Staging-Deployment

Nachdem der Pull Request gemergt wurde:

1. AWS Amplify baut und deployt automatisch den aktualisierten `develop`-Branch
2. Du kannst den Fortschritt in der Amplify-Konsole verfolgen
3. Wenn der Build abgeschlossen ist, öffne die Staging-URL und überprüfe, ob der Footer angezeigt wird

**Erklärung:**
- Änderungen im `develop`-Branch werden automatisch in die Staging-Umgebung deployt
- Dies ermöglicht Tests in einer produktionsähnlichen Umgebung
- Wenn alles in Ordnung ist, können die Änderungen später in den `main`-Branch gemergt werden

### 5.8 In Produktion deployen

Wenn die Änderungen im Staging erfolgreich getestet wurden:

```bash
# Wechsle zum main-Branch
git checkout main

# Hole die neuesten Änderungen
git pull

# Merge den develop-Branch in den main-Branch
git merge develop

# Pushe die Änderungen
git push origin main
```

Alternativ kannst du auch einen weiteren Pull Request von `develop` nach `main` erstellen.

**Erklärung:**
- Änderungen gehen erst in die Staging-Umgebung (`develop`) und dann in die Produktion (`main`)
- Dieser Workflow gewährleistet, dass nur getestete Änderungen in die Produktion gelangen
- Nach dem Push zu `main` wird die Anwendung automatisch in der Produktionsumgebung deployt

## 6. Feature-Flag-Funktionalität hinzufügen

Feature Flags ermöglichen es, Features selektiv in verschiedenen Umgebungen zu aktivieren oder zu deaktivieren.

### 6.1 Feature-Flag-Funktionalität implementieren

Erstelle einen neuen Feature-Branch:

```bash
# Stelle sicher, dass du auf dem develop-Branch bist
git checkout develop
git pull

# Erstelle und wechsle zum feature-Branch
git checkout -b feature/feature-flags
```

Erstelle eine Datei `src/featureFlags.js`:

```javascript
// Feature-Flags basierend auf der Umgebung
const environment = process.env.REACT_APP_ENVIRONMENT || 'development';

const featureFlags = {
  // Feature-Flags für verschiedene Umgebungen
  development: {
    showFooter: true,
    showContactForm: true,
  },
  staging: {
    showFooter: true,
    showContactForm: true,
  },
  production: {
    showFooter: true,
    showContactForm: false,
  }
};

// Exportiere die Feature-Flags für die aktuelle Umgebung
export default featureFlags[environment] || featureFlags.development;
```

**Erklärung:**
- Wir definieren ein Objekt mit Feature-Flags für jede Umgebung
- In `development` und `staging` sind beide Features aktiviert
- In `production` ist `showContactForm` deaktiviert
- Die Funktion gibt die Flags für die aktuelle Umgebung zurück

### 6.2 App.js mit Feature-Flags aktualisieren

Aktualisiere `src/App.js`:

```jsx
import React from 'react';
import './App.css';
import featureFlags from './featureFlags';

function App() {
  const environment = process.env.REACT_APP_ENVIRONMENT || 'development';
  const version = process.env.REACT_APP_VERSION || '1.0.0';
  const buildNumber = process.env.REACT_APP_BUILD_NUMBER || 'local';
  
  return (
    <div className="App">
      <header className="App-header">
        <h1>React Deployment Demo</h1>
        <p>
          Diese Anwendung demonstriert Continuous Deployment mit GitHub Actions.
        </p>
        <div className="environment-info">
          <p>Umgebung: <span className={`env-${environment}`}>{environment}</span></p>
          <p>Version: {version}</p>
          <p>Build: {buildNumber}</p>
        </div>
        
        {featureFlags.showContactForm && (
          <div className="contact-form">
            <h3>Kontaktformular</h3>
            <p>Dieses Feature ist nur in Development und Staging verfügbar.</p>
            <form onSubmit={(e) => e.preventDefault()}>
              <input type="text" placeholder="Name" className="form-input" />
              <input type="email" placeholder="E-Mail" className="form-input" />
              <button type="submit" className="# Continuous Deployment einer React-Anwendung mit GitHub Actions

In dieser Übung wirst du eine vollständige CI/CD-Pipeline für eine einfache React-Anwendung erstellen. Du wirst lernen, wie du deine Anwendung automatisch in verschiedene Umgebungen (Staging und Produktion) deployst und wie du das Deployment von Pull Requests abhängig machst.

> **Hinweis:** Continuous Deployment (CD) ist der Prozess, bei dem Code automatisch in Produktionsumgebungen bereitgestellt wird, nachdem er die CI-Pipeline erfolgreich durchlaufen hat. Dies ermöglicht schnellere Releases, reduziert manuelle Fehler und bietet ein besseres Feedback für dein Entwicklungsteam.

## 1. Eine einfache React-Anwendung erstellen

Zunächst erstellen wir eine einfache React-Anwendung, die wir später deployen werden.

### 1.1 React-Projekt initialisieren

```bash
# Erstelle ein neues React-Projekt
npx create-react-app react-deployment-demo
cd react-deployment-demo

# Initialisiere Git
git init
git add .
git commit -m "Initial commit"

# Verbinde mit GitHub
# Erstelle zuerst ein neues leeres Repository auf GitHub und führe dann aus:
git remote add origin https://github.com/dein-username/react-deployment-demo.git
git push -u origin main
```

**Erklärung:**
- `create-react-app` ist ein beliebtes Tool zum Erstellen von React-Anwendungen
- Es richtet eine komplette Entwicklungsumgebung mit Webpack, Babel und Jest ein
- Die erstellte Anwendung hat bereits eine Basisstruktur und einen Entwicklungsserver

### 1.2 Anwendung anpassen

Öffne `src/App.js` und ersetze den Inhalt durch:

```jsx
import React from 'react';
import './App.css';

function App() {
  const environment = process.env.REACT_APP_ENVIRONMENT || 'development';
  const version = process.env.REACT_APP_VERSION || '1.0.0';
  const buildNumber = process.env.REACT_APP_BUILD_NUMBER || 'local';
  
  return (
    <div className="App">
      <header className="App-header">
        <h1>React Deployment Demo</h1>
        <p>
          Diese Anwendung demonstriert Continuous Deployment mit GitHub Actions.
        </p>
        <div className="environment-info">
          <p>Umgebung: <span className={`env-${environment}`}>{environment}</span></p>
          <p>Version: {version}</p>
          <p>Build: {buildNumber}</p>
        </div>
      </header>
    </div>
  );
}

export default App;
```

Ersetze den Inhalt von `src/App.css` durch:

```css
.App {
  text-align: center;
}

.App-header {
  background-color: #282c34;
  min-height: 100vh;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  font-size: calc(10px + 2vmin);
  color: white;
}

.environment-info {
  margin-top: 2rem;
  padding: 1rem;
  border: 1px solid #61dafb;
  border-radius: 8px;
  background-color: rgba(97, 218, 251, 0.1);
}

.env-development {
  color: yellow;
}

.env-staging {
  color: orange;
}

.env-production {
  color: #61dafb;
}
```

### 1.3 Umgebungsvariablen einrichten

Erstelle eine `.env.development`-Datei im Hauptverzeichnis:

```
REACT_APP_ENVIRONMENT=development
REACT_APP_VERSION=0.1.0
REACT_APP_BUILD_NUMBER=local
```

### 1.4 Teste die Anwendung lokal

```bash
# Starte den Entwicklungsserver
npm start
```

Öffne deinen Browser und gehe zu http://localhost:3000. Du solltest deine angepasste React-Anwendung sehen.

### 1.5 Änderungen committen

```bash
git add .
git commit -m "Anpassung der React-App für Deployment-Demo"
git push origin main
```

## 2. CI-Pipeline für die React-Anwendung einrichten

Als Nächstes richten wir eine CI-Pipeline ein, um unsere React-Anwendung zu testen und zu bauen.

### 2.1 Workflow-Verzeichnis erstellen

```bash
mkdir -p .github/workflows
```

### 2.2 CI-Workflow erstellen

Erstelle eine Datei `.github/workflows/ci.yml`:

```yaml
name: CI

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  build:
    name: Build
    runs-on: ubuntu-latest
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
      
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '18'
          cache: 'npm'
      
      - name: Install dependencies
        run: npm ci
      
      - name: Build
        run: npm run build
        env:
          REACT_APP_ENVIRONMENT: production
          REACT_APP_VERSION: ${{ github.ref_name }}
          REACT_APP_BUILD_NUMBER: ${{ github.run_number }}
      
      - name: Upload build artifacts
        uses: actions/upload-artifact@v4
        with:
          name: build
          path: build/
          retention-days: 7
```

### 2.3 Commit und Push der CI-Konfiguration

```bash
git add .github/workflows/ci.yml
git commit -m "Füge CI-Workflow hinzu"
git push origin main
```

### 2.4 CI-Workflow überprüfen

Gehe zu deinem GitHub-Repository und überprüfe den Tab "Actions". Du solltest sehen, dass der CI-Workflow ausgeführt wird.

## 3. Deployment auf GitHub Pages einrichten

Für unser erstes Deployment verwenden wir GitHub Pages, da es einfach einzurichten ist und keine externen Dienste erfordert.

### 3.1 gh-pages-Paket installieren

```bash
npm install --save-dev gh-pages
```

### 3.2 package.json aktualisieren

Öffne `package.json` und füge folgende Zeilen hinzu:

```json
"homepage": "https://dein-username.github.io/react-deployment-demo",
"scripts": {
  "start": "react-scripts start",
  "build": "react-scripts build",
  "test": "react-scripts test",
  "eject": "react-scripts eject",
  "predeploy": "npm run build",
  "deploy": "gh-pages -d build"
},
```

Ersetze `dein-username` durch deinen GitHub-Benutzernamen.

### 3.3 CD-Workflow für GitHub Pages erstellen

Erstelle eine Datei `.github/workflows/deploy-gh-pages.yml`:

```yaml
name: Deploy to GitHub Pages

on:
  # Wird nur ausgeführt, wenn der CI-Workflow erfolgreich war
  workflow_run:
    workflows: ["CI"]
    branches: [main]
    types: [completed]

jobs:
  deploy:
    name: Deploy
    runs-on: ubuntu-latest
    # Nur ausführen, wenn der vorherige Workflow erfolgreich war
    if: ${{ github.event.workflow_run.conclusion == 'success' }}
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
      
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '18'
          cache: 'npm'
      
      - name: Install dependencies
        run: npm ci
      
      - name: Set environment variables
        run: |
          echo "REACT_APP_ENVIRONMENT=production" >> .env.production
          echo "REACT_APP_VERSION=${{ github.ref_name }}" >> .env.production
          echo "REACT_APP_BUILD_NUMBER=${{ github.run_number }}" >> .env.production
      
      - name: Build
        run: npm run build
      
      - name: Deploy to GitHub Pages
        uses: JamesIves/github-pages-deploy-action@v4
        with:
          folder: build
          branch: gh-pages
```

### 3.4 Commit und Push der Deployment-Konfiguration

```bash
git add .github/workflows/deploy-gh-pages.yml package.json
git commit -m "Füge GitHub Pages Deployment hinzu"
git push origin main
```

### 3.5 GitHub Pages aktivieren

1. Gehe zu deinem Repository auf GitHub
2. Klicke auf "Settings"
3. Scrolle nach unten zu "GitHub Pages"
4. Unter "Source", wähle den Branch "gh-pages" und klicke auf "Save"

### 3.6 Überprüfe das Deployment

Nachdem der Workflow abgeschlossen ist, kannst du deine Anwendung unter `https://dein-username.github.io/react-deployment-demo` aufrufen. Ersetze `dein-username` durch deinen GitHub-Benutzernamen.

## 4. Staging- und Produktions-Deployments mit AWS einrichten

Es gibt viele Anbieter wie Netlify oder Vercel, mit denen man React-Anwendungen einfach deployen kann. In dieser Anleitung konzentrieren wir uns jedoch auf AWS, da es eine flexible und kosteneffiziente Lösung bietet, die häufig in Unternehmen eingesetzt wird.

### 4.1 AWS Amplify für kostengünstiges React-Deployment

Wir verwenden AWS Amplify, da es eine einfache und kostengünstige Möglichkeit bietet, React-Anwendungen zu hosten, ohne Server verwalten zu müssen.

### 4.2 AWS Konto einrichten

1. Falls du noch kein AWS-Konto hast, erstelle eines unter [aws.amazon.com](https://aws.amazon.com/)
2. Melde dich bei der AWS Management Console an
3. Navigiere zur Amplify-Konsole

### 4.3 Amplify für Produktion einrichten

1. Klicke in der Amplify-Konsole auf "Neue App erstellen"
2. Wähle "GitHub" als Repository-Quelle
3. Authentifiziere dich bei GitHub und wähle dein Repository aus
4. Wähle den Branch `main` für die Produktion
5. Konfiguriere die Build-Einstellungen:

```yaml
version: 1
frontend:
  phases:
    preBuild:
      commands:
        - npm ci
    build:
      commands:
        - npm run build
  artifacts:
    baseDirectory: build
    files:
      - '**/*'
  cache:
    paths:
      - node_modules/**/*
```

6. Füge folgende Umgebungsvariablen hinzu:
   - `REACT_APP_ENVIRONMENT`: `production`
   - `REACT_APP_VERSION`: `1.0.0`

7. Klicke auf "Weiter" und dann auf "Speichern und bereitstellen"

### 4.4 Amplify für Staging einrichten

1. Gehe zur Amplify-Konsole und wähle deine App aus
2. Klicke auf "Hosting-Umgebungen" > "Branch hinzufügen"
3. Erstelle in GitHub einen Branch `develop` und wähle ihn hier aus
4. Verwende die gleichen Build-Einstellungen wie für die Produktion
5. Füge folgende Umgebungsvariablen hinzu:
   - `REACT_APP_ENVIRONMENT`: `staging`
   - `REACT_APP_VERSION`: `1.0.0`
6. Klicke auf "Weiter" und dann auf "Speichern und bereitstellen"

### 4.5 Pull Request Vorschauen einrichten

1. Gehe zur Amplify-Konsole und wähle deine App aus
2. Klicke auf "Hosting-Umgebungen" > "Vorschau-Einstellungen"
3. Aktiviere "Pull-Request-Vorschauen"
4. Wähle "Automatisch PRs für alle Branches erstellen"
5. Klicke auf "Speichern"

## 5. Feature-Branch erstellen und Pull Request öffnen

### 5.1 Feature-Branch erstellen

```bash
git checkout -b feature/add-footer
```

### 5.2 Footer zur App hinzufügen

Öffne `src/App.js` und füge einen Footer hinzu (nach dem `</header>`-Tag):

```jsx
<footer className="App-footer">
  <p>© {new Date().getFullYear()} React Deployment Demo</p>
  <p>
    <small>Deployment Environment: {environment}</small>
  </p>
</footer>
```

Füge auch Styling für den Footer in `src/App.css` hinzu:

```css
.App-footer {
  background-color: #20232a;
  padding: 1rem;
  color: #ffffff;
  position: fixed;
  bottom: 0;
  width: 100%;
  text-align: center;
}
```

### 5.3 Committe und pushe die Änderungen

```bash
git add src/App.js src/App.css
git commit -m "Füge Footer hinzu"
git push -u origin feature/add-footer
```

### 5.4 Pull Request erstellen

Gehe zu deinem GitHub-Repository und erstelle einen Pull Request für den `feature/add-footer`-Branch.

1. Klicke auf "Compare & pull request"
2. Gib einen Titel ein (z.B. "Add footer to the application")
3. Klicke auf "Create pull request"

### 5.5 Überprüfe die Deployments

1. Der CI-Workflow wird automatisch für den Pull Request ausgeführt
2. AWS Amplify erstellt eine Vorschau für deinen Pull Request
3. Überprüfe die Vorschau, um sicherzustellen, dass der Footer korrekt angezeigt wird

### 5.6 Pull Request mergen

Wenn alles in Ordnung ist:

1. Klicke auf "Merge pull request"
2. Bestätige den Merge

### 5.7 Überprüfe das Produktions-Deployment

Nachdem der Pull Request gemergt wurde:

1. Der CI-Workflow wird automatisch für den main-Branch ausgeführt
2. Der CD-Workflow für GitHub Pages wird ausgeführt
3. AWS Amplify aktualisiert das Produktions-Deployment
4. Überprüfe beide Deployments, um sicherzustellen, dass der Footer korrekt angezeigt wird

## 6. Feature-Flag-Funktionalität hinzufügen

Feature Flags ermöglichen es, Features selektiv in verschiedenen Umgebungen zu aktivieren oder zu deaktivieren.

### 6.1 Feature-Flag-Funktionalität implementieren

Erstelle eine Datei `src/featureFlags.js`:

```javascript
// Feature-Flags basierend auf der Umgebung
const environment = process.env.REACT_APP_ENVIRONMENT || 'development';

const featureFlags = {
  // Feature-Flags für verschiedene Umgebungen
  development: {
    showFooter: true,
    showContactForm: true,
  },
  staging: {
    showFooter: true,
    showContactForm: true,
  },
  production: {
    showFooter: true,
    showContactForm: false,
  }
};

// Exportiere die Feature-Flags für die aktuelle Umgebung
export default featureFlags[environment] || featureFlags.development;
```

### 6.2 App.js mit Feature-Flags aktualisieren

Erstelle einen neuen Branch:

```bash
git checkout main
git pull
git checkout -b feature/feature-flags
```

Aktualisiere `src/App.js`:

```jsx
import React from 'react';
import './App.css';
import featureFlags from './featureFlags';

function App() {
  const environment = process.env.REACT_APP_ENVIRONMENT || 'development';
  const version = process.env.REACT_APP_VERSION || '1.0.0';
  const buildNumber = process.env.REACT_APP_BUILD_NUMBER || 'local';
  
  return (
    <div className="App">
      <header className="App-header">
        <h1>React Deployment Demo</h1>
        <p>
          Diese Anwendung demonstriert Continuous Deployment mit GitHub Actions.
        </p>
        <div className="environment-info">
          <p>Umgebung: <span className={`env-${environment}`}>{environment}</span></p>
          <p>Version: {version}</p>
          <p>Build: {buildNumber}</p>
        </div>
        
        {featureFlags.showContactForm && (
          <div className="contact-form">
            <h3>Kontaktformular</h3>
            <p>Dieses Feature ist nur in Development und Staging verfügbar.</p>
            <form>
              <input type="text" placeholder="Name" className="form-input" />
              <input type="email" placeholder="E-Mail" className="form-input" />
              <button type="submit" className="form-button">Senden</button>
            </form>
          </div>
        )}
      </header>
      
      {featureFlags.showFooter && (
        <footer className="App-footer">
          <p>© {new Date().getFullYear()} React Deployment Demo</p>
          <p>
            <small>Deployment Environment: {environment}</small>
          </p>
        </footer>
      )}
    </div>
  );
}

export default App;
```

**Erklärung:**
- Wir importieren die Feature-Flags aus der neuen Datei
- `{featureFlags.showContactForm && ...}` rendert das Kontaktformular nur, wenn das Flag aktiviert ist
- `{featureFlags.showFooter && ...}` rendert den Footer nur, wenn das Flag aktiviert ist
- Diese bedingte Renderung ermöglicht es uns, Features in verschiedenen Umgebungen ein- oder auszuschalten

### 6.3 App.css aktualisieren

Füge folgende Stile zu `src/App.css` hinzu:

```css
.contact-form {
  margin-top: 20px;
  padding: 20px;
  border: 1px solid #61dafb;
  border-radius: 8px;
  background-color: rgba(97, 218, 251, 0.1);
  max-width: 80%;
}

.form-input {
  width: 100%;
  padding: 10px;
  margin: 8px 0;
  border: 1px solid #61dafb;
  border-radius: 4px;
  background-color: #282c34;
  color: white;
}

.form-button {
  padding: 10px 20px;
  margin-top: 10px;
  background-color: #61dafb;
  color: #282c34;
  border: none;
  border-radius: 4px;
  cursor: pointer;
}

.form-button:hover {
  background-color: #21a1cb;
}
```

**Erklärung:**
- Diese CSS-Klassen stylen das Kontaktformular und seine Elemente
- `.contact-form` definiert das Layout des Formular-Containers
- `.form-input` stylt die Eingabefelder
- `.form-button` stylt den Submit-Button
- `.form-button:hover` ändert das Aussehen des Buttons beim Darüberfahren mit der Maus

### 6.4 Commit und Push der Änderungen

```bash
git add src/featureFlags.js src/App.js src/App.css
git commit -m "Füge Feature-Flags hinzu"
git push -u origin feature/feature-flags
```

**Erklärung:**
- Wir fügen alle geänderten Dateien zum Commit hinzu
- Die Commit-Nachricht beschreibt klar, was geändert wurde
- Wir pushen den Branch zu GitHub und richten gleichzeitig das Upstream-Tracking ein

### 6.5 Pull Request erstellen und überprüfen

1. Gehe zu GitHub und erstelle einen Pull Request für den `feature/feature-flags`-Branch (ziele auf `develop`)
2. Warte auf den CI-Workflow und die Amplify-Vorschau
3. Überprüfe die Vorschau-URL und stelle sicher, dass:
   - Der Footer angezeigt wird
   - Das Kontaktformular angezeigt wird (weil wir in Staging/Preview sind)
4. Merge den Pull Request in den `develop`-Branch

**Erklärung:**
- Der Pull Request ermöglicht die Überprüfung der Änderungen
- Die automatische Vorschau zeigt, wie die Änderungen aussehen werden
- Da wir in einer Vorschau-/Staging-Umgebung sind, sollte das Kontaktformular sichtbar sein

### 6.6 In Produktion deployen

Nachdem die Änderungen im Staging getestet wurden:

```bash
# Wechsle zum main-Branch
git checkout main
git pull

# Merge den develop-Branch in den main-Branch
git merge develop

# Pushe die Änderungen
git push origin main
```

**Erklärung:**
- Wir bringen die getesteten Änderungen von `develop` nach `main`
- Nach dem Push zu `main` werden die Änderungen automatisch in der Produktionsumgebung deployt

### 6.7 Die verschiedenen Umgebungen vergleichen

Nach dem Deployment in allen Umgebungen kannst du die URLs vergleichen:

- **In der Produktionsumgebung** (main-Branch):
  - Der Footer wird angezeigt
  - Das Kontaktformular wird NICHT angezeigt (weil `showContactForm: false` in production)

- **In der Staging-Umgebung** (develop-Branch):
  - Der Footer wird angezeigt
  - Das Kontaktformular wird angezeigt

- **Lokal** (development):
  - Der Footer wird angezeigt
  - Das Kontaktformular wird angezeigt

**Erklärung:**
- Feature Flags ermöglichen unterschiedliches Verhalten in verschiedenen Umgebungen
- Dies ist nützlich für A/B-Tests, schrittweise Einführung neuer Features oder Funktionen, die noch nicht produktionsreif sind
- Mit diesem Ansatz können wir Code deployen, ohne alle Features sofort für alle Benutzer zu aktivieren

## 7. Zusammenfassung

In dieser Übung hast du gelernt:

1. Eine einfache React-Anwendung zu erstellen
2. Eine CI/CD-Pipeline mit GitHub Actions einzurichten
3. Die Anwendung auf GitHub Pages zu deployen
4. Staging- und Produktions-Deployments mit AWS Amplify zu konfigurieren
5. Mit Feature Branches und Pull Requests zu arbeiten
6. Feature-Flags zu implementieren, um Features selektiv zu aktivieren

### 7.1 Wichtige Konzepte

- **Continuous Integration (CI)**: Automatisiertes Testen und Bauen bei jedem Code-Push
- **Continuous Deployment (CD)**: Automatisiertes Deployment in verschiedene Umgebungen
- **Feature Branches**: Isolierte Entwicklung neuer Funktionen
- **Pull Requests**: Code-Review-Prozess vor der Integration
- **Feature Flags**: Selektive Aktivierung von Features je nach Umgebung

### 7.2 Nächste Schritte

- Integration von automatisierten Tests in die CI-Pipeline
- Einrichtung von Monitoring und Alerting
- Implementierung von A/B-Tests mit Feature Flags
- Konfiguration von Blue-Green-Deployments für Zero-Downtime-Updates

### 7.3 Fehlerbehebung

Hier sind einige häufige Probleme und ihre Lösungen:

- **GitHub Actions-Workflow schlägt fehl**:
  - Überprüfe die Logs im "Actions"-Tab für genaue Fehlerdetails
  - Stelle sicher, dass alle Abhängigkeiten korrekt installiert sind
  - Prüfe die Syntax deiner YAML-Dateien

- **AWS Amplify-Build schlägt fehl**:
  - Überprüfe die Build-Logs in der Amplify-Konsole
  - Stelle sicher, dass deine Build-Einstellungen korrekt sind
  - Prüfe, ob alle benötigten Umgebungsvariablen gesetzt sind

- **GitHub Pages zeigt 404**:
  - Stelle sicher, dass der gh-pages Branch existiert
  - Überprüfe, ob GitHub Pages in den Repository-Einstellungen aktiviert ist
  - Prüfe, ob die `homepage`-URL in package.json korrekt ist

- **WSL-spezifische Probleme**:
  - Bei Port-Forwarding-Problemen: Stelle sicher, dass der Windows-Firewall den Zugriff erlaubt
  - Bei Git-Authentifizierungsproblemen: Verwende ein personelles Zugriffstoken statt Passwort
  - Bei Node.js-Problemen: Verwende nvm zur Verwaltung verschiedener Node.js-Versionen
```

### 6.3 App.css aktualisieren

Füge folgende Stile zu `src/App.css` hinzu:

```css
.contact-form {
  margin-top: 20px;
  padding: 20px;
  border: 1px solid #61dafb;
  border-radius: 8px;
  background-color: rgba(97, 218, 251, 0.1);
  max-width: 80%;
}

.form-input {
  width: 100%;
  padding: 10px;
  margin: 8px 0;
  border: 1px solid #61dafb;
  border-radius: 4px;
  background-color: #282c34;
  color: white;
}

.form-button {
  padding: 10px 20px;
  margin-top: 10px;
  background-color: #61dafb;
  color: #282c34;
  border: none;
  border-radius: 4px;
  cursor: pointer;
}
```

### 6.4 Commit und Push der Änderungen

```bash
git add src/featureFlags.js src/App.js src/App.css
git commit -m "Füge Feature-Flags hinzu"
git push -u origin feature/feature-flags
```

### 6.5 Pull Request erstellen und überprüfen

1. Erstelle einen Pull Request für den `feature/feature-flags`-Branch
2. Überprüfe die Amplify-Vorschau und achte darauf, dass das Kontaktformular angezeigt wird
3. Merge den Pull Request

### 6.6 Die verschiedenen Umgebungen vergleichen

Nach dem Merge kannst du die verschiedenen Umgebungen vergleichen:

- **In Production**: 
  - Footer wird angezeigt
  - Kein Kontaktformular

- **In Staging**: 
  - Footer wird angezeigt
  - Kontaktformular wird angezeigt

- **Lokal in Development**: 
  - Footer wird angezeigt
  - Kontaktformular wird angezeigt

## 7. Zusammenfassung

In dieser Übung hast du gelernt:

1. Eine einfache React-Anwendung zu erstellen
2. Eine CI/CD-Pipeline mit GitHub Actions einzurichten
3. Die Anwendung auf GitHub Pages zu deployen
4. Staging- und Produktions-Deployments mit AWS Amplify zu konfigurieren
5. Mit Feature Branches und Pull Requests zu arbeiten
6. Feature-Flags zu implementieren, um Features selektiv zu aktivieren

### 7.1 Wichtige Konzepte

- **Continuous Integration (CI)**: Automatisiertes Testen und Bauen bei jedem Code-Push
- **Continuous Deployment (CD)**: Automatisiertes Deployment in verschiedene Umgebungen
- **Feature Branches**: Isolierte Entwicklung neuer Funktionen
- **Pull Requests**: Code-Review-Prozess vor der Integration
- **Feature Flags**: Selektive Aktivierung von Features je nach Umgebung

### 7.2 Nächste Schritte

- Integration von automatisierten Tests in die CI-Pipeline
- Einrichtung von Monitoring und Alerting
- Implementierung von A/B-Tests mit Feature Flags
- Konfiguration von Blue-Green-Deployments für Zero-Downtime-Updates
