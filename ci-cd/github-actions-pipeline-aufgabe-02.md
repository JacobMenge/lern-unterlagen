# CI/CD Pipeline mit Tests - Automatisierte Tests in GitHub Actions

In dieser Übung wirst du eine CI/CD-Pipeline für eine einfache JavaScript-Anwendung erstellen. Du lernst, wie du automatisierte Tests in deine Pipeline integrierst und wie die kontinuierliche Integration deinen Entwicklungsprozess verbessern kann. Diese Übung baut auf den Grundlagen auf, die du in Level 1 gelernt hast.

> **Hinweis:** Automatisierte Tests sind ein wesentlicher Bestandteil moderner Softwareentwicklung. Sie helfen dir, Fehler früh zu erkennen und stellen sicher, dass deine Anwendung wie erwartet funktioniert. Durch die Integration von Tests in deine CI/CD-Pipeline kannst du sicherstellen, dass nur Code, der alle Tests besteht, in deine Hauptbranches gelangt.

## Umgebungsvoraussetzungen

Bevor du mit der Übung beginnst, stelle sicher, dass deine Entwicklungsumgebung richtig eingerichtet ist:

### Für Windows-Nutzer mit WSL (Windows Subsystem für Linux)

Wenn du WSL verwendest, führe diese Schritte aus, um eine saubere Node.js-Umgebung einzurichten:

```bash
# Installiere curl (falls nicht vorhanden)
sudo apt update
sudo apt install -y curl

# Installiere nvm (Node Version Manager)
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.7/install.sh | bash

# Lade nvm in die aktuelle Shell
export NVM_DIR="$HOME/.nvm"
[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"

# Stelle sicher, dass nvm in zukünftigen Shell-Sitzungen verfügbar ist
echo 'export NVM_DIR="$HOME/.nvm"' >> ~/.bashrc
echo '[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"' >> ~/.bashrc

# Installiere Node.js LTS
nvm install --lts

# Verwende diese Version als Standard
nvm use --lts

# Überprüfe die Installation
node -v
npm -v

# Konfiguriere Git für Linux-Zeilenumbrüche
git config --global core.autocrlf input
```

> **Wichtig für WSL-Nutzer:** Speichere deine Projektdateien im Linux-Dateisystem (`/home/...`), nicht im Windows-Dateisystem (`/mnt/c/...`), um Leistungs- und Kompatibilitätsprobleme zu vermeiden.

### Für Windows-Nutzer ohne WSL

1. Installiere Node.js und npm von [https://nodejs.org/](https://nodejs.org/) (LTS-Version empfohlen)
2. Überprüfe die Installation:
   ```
   node -v
   npm -v
   ```

### Für macOS-Nutzer

Mit Homebrew:
```bash
brew install node
```

Oder mit nvm (empfohlen):
```bash
# Installiere nvm
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.7/install.sh | bash

# Lade nvm
export NVM_DIR="$HOME/.nvm"
[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"

# Installiere Node.js LTS
nvm install --lts
```

### Für Linux-Nutzer

Mit nvm (empfohlen):
```bash
# Installiere nvm
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.7/install.sh | bash

# Lade nvm
export NVM_DIR="$HOME/.nvm"
[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"

# Installiere Node.js LTS
nvm install --lts
```

## 1. JavaScript-Projekt einrichten

In diesem Abschnitt erstellen wir eine einfache JavaScript-Anwendung, die wir später testen werden.

### 1.1 GitHub-Repository erstellen und Projekt initialisieren

1. **Erstelle ein neues Repository auf GitHub:**
   - Gehe zu GitHub (https://github.com) und melde dich an
   - Klicke auf das '+' Symbol in der oberen rechten Ecke und wähle "New repository"
   - Gib "js-testing-pipeline" als Repository-Namen ein
   - Füge optional eine Beschreibung hinzu, z.B. "Eine einfache JavaScript-Anwendung mit Tests für GitHub Actions"
   - Wähle "Public" als Sichtbarkeit (oder "Private", falls gewünscht)
   - **Wichtig:** Aktiviere KEINE der Optionen zum Initialisieren des Repositories (kein README, keine .gitignore, keine Lizenz)
   - Klicke auf "Create repository"

2. **Erstelle ein neues Verzeichnis für dein lokales Projekt:**

```bash
# Erstelle ein neues Projekt
mkdir js-testing-pipeline
cd js-testing-pipeline

# Initialisiere Git
git init

# Prüfe den aktuellen Branch-Namen
git branch

# Bei neueren Git-Versionen ist der Standard-Branch bereits "main"
# Bei älteren Git-Versionen musst du den Branch "master" in "main" umbenennen
git branch -M main

# Verbinde dein lokales Repository mit dem GitHub-Repository
git remote add origin https://github.com/dein-username/js-testing-pipeline.git
```

> **Hinweis:** Der Befehl `git branch -M main` benennt deinen aktuellen Branch in "main" um, unabhängig davon, wie er vorher hieß. Dies ist wichtig, da GitHub standardmäßig "main" als Namen für den Hauptbranch verwendet.
> 
> **Wichtig:** Ersetze "dein-username" mit deinem tatsächlichen GitHub-Benutzernamen. Die URL findest du auch auf der GitHub-Seite deines neuen Repositories.

### 1.2 Projektdateien erstellen

Zunächst erstellen wir eine einfache `package.json`-Datei:

```bash
# Initialisiere npm mit Standardwerten
npm init -y
```

Öffne die generierte `package.json`-Datei und aktualisiere sie mit folgendem Inhalt:

```json
{
  "name": "js-testing-pipeline",
  "version": "1.0.0",
  "description": "Eine einfache JavaScript-Anwendung mit Tests für GitHub Actions",
  "main": "src/calculator.js",
  "scripts": {
    "test": "jest",
    "lint": "eslint src/**/*.js"
  },
  "keywords": [
    "github-actions",
    "testing"
  ],
  "author": "Dein Name",
  "license": "MIT",
  "devDependencies": {
    "eslint": "^8.57.0",
    "jest": "^29.7.0"
  }
}
```

**Was ist in dieser Datei wichtig?**
- `"scripts"`: Definiert die Befehle, die wir mit `npm run` ausführen können
- `"test"`: Führt Jest aus, unser Test-Framework
- `"lint"`: Führt ESLint aus, um unseren Code auf Qualität zu prüfen
- `"devDependencies"`: Die Entwicklungswerkzeuge, die wir für Tests und Linting benötigen

### 1.3 Source-Code erstellen

Jetzt erstellen wir eine einfache Taschenrechner-Funktion, die wir später testen werden:

```bash
# Erstelle src-Verzeichnis
mkdir -p src

# Erstelle calculator.js
touch src/calculator.js
```

Füge folgenden Code in `src/calculator.js` ein:

```javascript
/**
 * Einfacher Taschenrechner mit grundlegenden mathematischen Operationen
 */
class Calculator {
  /**
   * Addiert zwei Zahlen
   * @param {number} a - Erste Zahl
   * @param {number} b - Zweite Zahl
   * @returns {number} Summe der beiden Zahlen
   */
  add(a, b) {
    return a + b;
  }

  /**
   * Subtrahiert b von a
   * @param {number} a - Erste Zahl
   * @param {number} b - Zweite Zahl
   * @returns {number} Differenz der beiden Zahlen
   */
  subtract(a, b) {
    return a - b;
  }

  /**
   * Multipliziert zwei Zahlen
   * @param {number} a - Erste Zahl
   * @param {number} b - Zweite Zahl
   * @returns {number} Produkt der beiden Zahlen
   */
  multiply(a, b) {
    return a * b;
  }

  /**
   * Dividiert a durch b
   * @param {number} a - Dividend
   * @param {number} b - Divisor
   * @returns {number} Quotient der Division
   * @throws {Error} Wenn der Divisor 0 ist
   */
  divide(a, b) {
    if (b === 0) {
      throw new Error('Division durch Null ist nicht erlaubt.');
    }
    return a / b;
  }
}

module.exports = Calculator;
```

**Erklärung:**
- Wir erstellen eine einfache `Calculator`-Klasse mit vier grundlegenden mathematischen Operationen
- Jede Methode hat JSDoc-Kommentare, die ihre Funktionalität und Parameter dokumentieren
- Die `divide`-Methode wirft einen Fehler, wenn versucht wird, durch Null zu dividieren
- Wir exportieren die Klasse, damit sie in anderen Dateien verwendet werden kann

### 1.4 Test-Dateien erstellen

Jetzt erstellen wir Tests für unsere Calculator-Klasse:

```bash
# Erstelle test-Verzeichnis
mkdir -p test

# Erstelle calculator.test.js
touch test/calculator.test.js
```

Füge folgenden Code in `test/calculator.test.js` ein:

```javascript
const Calculator = require('../src/calculator');

describe('Calculator', () => {
  let calculator;

  // Vor jedem Test eine neue Instanz erstellen
  beforeEach(() => {
    calculator = new Calculator();
  });

  describe('add method', () => {
    test('should add two positive numbers correctly', () => {
      expect(calculator.add(2, 3)).toBe(5);
    });

    test('should handle negative numbers', () => {
      expect(calculator.add(-1, -3)).toBe(-4);
      expect(calculator.add(-1, 5)).toBe(4);
    });

    test('should handle zero', () => {
      expect(calculator.add(0, 0)).toBe(0);
      expect(calculator.add(5, 0)).toBe(5);
    });
  });

  describe('subtract method', () => {
    test('should subtract two numbers correctly', () => {
      expect(calculator.subtract(5, 3)).toBe(2);
    });

    test('should handle negative results', () => {
      expect(calculator.subtract(2, 5)).toBe(-3);
    });

    test('should handle zero', () => {
      expect(calculator.subtract(5, 0)).toBe(5);
      expect(calculator.subtract(0, 5)).toBe(-5);
    });
  });

  describe('multiply method', () => {
    test('should multiply two numbers correctly', () => {
      expect(calculator.multiply(2, 3)).toBe(6);
    });

    test('should handle negative numbers', () => {
      expect(calculator.multiply(-2, 3)).toBe(-6);
      expect(calculator.multiply(-2, -3)).toBe(6);
    });

    test('should handle zero', () => {
      expect(calculator.multiply(5, 0)).toBe(0);
    });
  });

  describe('divide method', () => {
    test('should divide two numbers correctly', () => {
      expect(calculator.divide(6, 3)).toBe(2);
      expect(calculator.divide(7, 2)).toBe(3.5);
    });

    test('should handle dividing by negative numbers', () => {
      expect(calculator.divide(6, -3)).toBe(-2);
      expect(calculator.divide(-6, -3)).toBe(2);
    });

    test('should throw an error when dividing by zero', () => {
      expect(() => calculator.divide(5, 0)).toThrow('Division durch Null ist nicht erlaubt.');
    });
  });
});
```

**Erklärung:**
- Wir importieren unsere `Calculator`-Klasse
- Mit `describe` gruppieren wir zusammenhängende Tests
- `beforeEach` erstellt vor jedem Test eine neue `Calculator`-Instanz
- Wir testen jede Methode mit verschiedenen Eingabewerten
- Für die `divide`-Methode testen wir auch den Fehlerfall (Division durch Null)

### 1.5 ESLint-Konfiguration erstellen

Füge eine ESLint-Konfiguration hinzu, um Codequalitätsregeln zu definieren:

```bash
# Erstelle .eslintrc.json
touch .eslintrc.json
```

Füge folgenden Inhalt in `.eslintrc.json` ein:

```json
{
  "env": {
    "commonjs": true,
    "es2021": true,
    "node": true,
    "jest": true
  },
  "extends": "eslint:recommended",
  "parserOptions": {
    "ecmaVersion": 2023
  },
  "rules": {
    "indent": ["error", 2],
    "linebreak-style": ["error", "unix"],
    "quotes": ["error", "single"],
    "semi": ["error", "always"],
    "no-unused-vars": "warn"
  }
}
```

**Erklärung:**
- `env`: Definiert die Umgebungen, in denen unser Code ausgeführt wird
- `extends`: Verwendet die empfohlenen ESLint-Regeln als Basis
- `rules`: Definiert spezifische Regeln für unseren Code
  - Einrückung mit 2 Leerzeichen
  - UNIX-Zeilenumbrüche
  - Einfache Anführungszeichen für Strings
  - Immer Semikolons verwenden
  - Warnung bei unbenutzten Variablen

### 1.6 Jest-Konfiguration erstellen

Erstelle eine Konfigurationsdatei für Jest:

```bash
# Erstelle jest.config.js
touch jest.config.js
```

Füge folgenden Inhalt in `jest.config.js` ein:

```javascript
module.exports = {
  verbose: true,
  testEnvironment: 'node',
  collectCoverage: true,
  collectCoverageFrom: ['src/**/*.js'],
  coverageThreshold: {
    global: {
      branches: 80,
      functions: 80,
      lines: 80,
      statements: 80
    }
  },
  coverageReporters: ['text', 'lcov'],
  // Hilft bei der korrekten Auflösung von Pfaden in verschiedenen Umgebungen
  roots: ['<rootDir>/src/', '<rootDir>/test/'],
  testMatch: ['**/__tests__/**/*.js?(x)', '**/?(*.)+(spec|test).js?(x)'],
  moduleFileExtensions: ['js', 'json', 'jsx', 'node']
};
```

**Erklärung:**
- `verbose`: Gibt detaillierte Testberichte aus
- `testEnvironment`: Legt fest, dass die Tests in einer Node.js-Umgebung ausgeführt werden
- `collectCoverage`: Aktiviert die Erfassung von Testabdeckungsberichten
- `collectCoverageFrom`: Legt fest, für welche Dateien die Testabdeckung gemessen wird
- `coverageThreshold`: Definiert Mindestwerte für die Testabdeckung
- `coverageReporters`: Legt die Formate der Testabdeckungsberichte fest
- Die zusätzlichen Einstellungen verbessern die Kompatibilität mit verschiedenen Umgebungen, einschließlich WSL

### 1.7 .gitignore erstellen

Erstelle eine `.gitignore`-Datei, um bestimmte Dateien und Verzeichnisse von der Versionskontrolle auszuschließen:

```bash
# Erstelle .gitignore
touch .gitignore
```

Füge folgenden Inhalt in `.gitignore` ein:

```
# Dependencies
node_modules/

# Coverage reports
coverage/

# IDE und Editor-spezifische Dateien
.vscode/
.idea/
*.swp
*.swo

# Betriebssystem-spezifische Dateien
.DS_Store
Thumbs.db
```

### 1.8 README.md mit Status-Badge erstellen

Erstelle eine README.md-Datei mit einem GitHub Actions Status-Badge:

```bash
# Erstelle README.md
touch README.md
```

Füge folgenden Inhalt in `README.md` ein:

```markdown
# JavaScript Testing Pipeline

Eine einfache JavaScript-Anwendung mit Tests für GitHub Actions.

## GitHub Actions Status
![Test Workflow](https://github.com/dein-username/js-testing-pipeline/actions/workflows/test.yml/badge.svg)

## Beschreibung

Dieses Projekt demonstriert, wie man eine CI/CD-Pipeline mit GitHub Actions für eine JavaScript-Anwendung einrichtet.
```

> **Hinweis:** Ersetze "dein-username" durch deinen tatsächlichen GitHub-Benutzernamen.

### 1.9 Abhängigkeiten installieren

Installiere die benötigten Abhängigkeiten:

```bash
# Installiere die Abhängigkeiten
npm install
```

Dies dauert einen Moment, während npm Jest, ESLint und andere Abhängigkeiten herunterlädt und installiert.

### 1.10 Lokale Testausführung

Führe die Tests lokal aus, um sicherzustellen, dass alles funktioniert:

```bash
# Führe die Tests aus
npm test
```

Du solltest eine Ausgabe wie diese sehen:

```
PASS  test/calculator.test.js
  Calculator
    add method
      ✓ should add two positive numbers correctly (3 ms)
      ✓ should handle negative numbers (1 ms)
      ✓ should handle zero
    subtract method
      ✓ should subtract two numbers correctly
      ✓ should handle negative results
    multiply method
      ✓ should multiply two numbers correctly
      ✓ should handle negative numbers
      ✓ should handle zero
    divide method
      ✓ should divide two numbers correctly
      ✓ should handle dividing by negative numbers
      ✓ should throw an error when dividing by zero (2 ms)

Test Suites: 1 passed, 1 total
Tests:       11 passed, 11 total
Snapshots:   0 total
Time:        0.389 s, estimated 1 s
```

Zusätzlich wird ein Testabdeckungsbericht generiert, der anzeigt, wie viele Codeteile durch die Tests abgedeckt werden.

Falls du ESLint separat ausführen möchtest:

```bash
# Führe die Lint-Prüfung aus
npm run lint
```

Dies überprüft deinen Code auf Stilfehler und andere Qualitätsprobleme.

## 2. GitHub Actions Workflow für Tests einrichten

Jetzt richten wir einen GitHub Actions Workflow ein, der unsere Tests automatisch ausführt, wenn Code gepusht wird.

### Wichtige GitHub Actions-Konzepte:

- **Workflow**: Eine automatisierte Prozedur, die aus einem oder mehreren Jobs besteht
- **Event**: Ein Auslöser, der einen Workflow startet (z.B. Push, Pull Request)
- **Job**: Eine Gruppe von Schritten, die auf demselben Runner ausgeführt werden
- **Step**: Eine einzelne Aufgabe innerhalb eines Jobs
- **Action**: Eine wiederverwendbare Einheit, die spezifische Aufgaben ausführt
- **Runner**: Ein Server, auf dem die Workflows ausgeführt werden

### 2.1 Workflow-Verzeichnis erstellen

```bash
# Erstelle das Verzeichnis für Workflows
mkdir -p .github/workflows
```

### 2.2 Test-Workflow erstellen

Erstelle eine neue Datei namens `test.yml` im Verzeichnis `.github/workflows/`:

```bash
# Erstelle die Workflow-Datei
touch .github/workflows/test.yml
```

Füge folgenden Inhalt in `.github/workflows/test.yml` ein:

```yaml
name: Test Workflow

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]
  workflow_dispatch:

jobs:
  test:
    name: Tests und Linting
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
      
      - name: Run linting
        run: npm run lint
      
      - name: Run tests
        run: npm test
      
      - name: Upload coverage report
        uses: actions/upload-artifact@v4
        with:
          name: coverage-report
          path: coverage/
```

**Erklärung:**
- Wir führen den Workflow aus, wenn Code in den main-Branch gepusht wird oder ein Pull Request erstellt wird
- Der Job `test` wird auf der neuesten Ubuntu-Version ausgeführt
- Wir checken den Code aus und richten Node.js ein
- `npm ci` ist ähnlich wie `npm install`, aber speziell für CI-Umgebungen optimiert
- Wir führen Linting und Tests aus
- Am Ende laden wir den Testabdeckungsbericht als Artefakt hoch

### 2.4 Änderungen committen und auf GitHub hochladen

Nachdem du alle Dateien erstellt und konfiguriert hast, ist es Zeit, deine Änderungen zu committen und auf GitHub hochzuladen:

```bash
# Füge alle Dateien hinzu
git add .

# Commit erstellen
git commit -m "Füge Taschenrechner-Implementierung und Tests hinzu"

# Stelle sicher, dass du auf dem "main"-Branch bist
git branch

# Falls nötig, benenne deinen aktuellen Branch in "main" um
git branch -M main

# Haupt-Branch erstellen und Änderungen auf GitHub pushen
# Der -u Parameter erstellt den Branch auf GitHub und setzt das Upstream
git push -u origin main
```

Falls der Push-Befehl fehlschlägt, überprüfe Folgendes:

1. **Stelle sicher, dass du mindestens einen Commit hast:**
   ```bash
   git log
   ```
   
2. **Überprüfe, ob die Remote-URL korrekt ist:**
   ```bash
   git remote -v
   ```
   
3. **Falls nötig, setze die Remote-URL neu:**
   ```bash
   git remote set-url origin https://github.com/dein-username/js-testing-pipeline.git
   ```

4. **Dann versuche erneut zu pushen:**
   ```bash
   git push -u origin main
   ```

### 2.5 Workflow-Ausführung überprüfen

Gehe zu deinem GitHub-Repository und überprüfe den ausgeführten Workflow:

1. Klicke auf den Tab "Actions"
2. Du solltest deinen "Test Workflow" sehen, der gerade ausgeführt wird oder bereits abgeschlossen ist
3. Klicke auf den Workflow-Lauf, um die Details anzuzeigen
4. Überprüfe die Logs für die verschiedenen Schritte
5. Nachdem der Workflow abgeschlossen ist, kannst du den Testabdeckungsbericht als Artefakt herunterladen

## 3. Code-Änderungen und Pull Requests

Jetzt werden wir den CI-Prozess in Aktion sehen, indem wir eine Änderung über einen Pull Request einführen.

### 3.1 Erstelle einen neuen Branch

```bash
# Erstelle einen neuen Branch
git checkout -b feature/power-function
```

**Hinweis zu Branch-Namen**: In der Praxis werden oft Konventionen wie `feature/`, `bugfix/`, `hotfix/` etc. als Präfixe für Branch-Namen verwendet. Dies hilft dabei, den Zweck eines Branches schnell zu erkennen.

Du siehst eine Ausgabe wie: `Switched to a new branch 'feature/power-function'`

### 3.2 Füge eine neue Funktion hinzu

Öffne `src/calculator.js` und füge eine neue Methode zur `Calculator`-Klasse hinzu:

```javascript
/**
 * Berechnet a hoch b
 * @param {number} a - Basis
 * @param {number} b - Exponent
 * @returns {number} a hoch b
 */
power(a, b) {
  return Math.pow(a, b);
}
```

Die vollständige Datei sollte jetzt so aussehen:

```javascript
/**
 * Einfacher Taschenrechner mit grundlegenden mathematischen Operationen
 */
class Calculator {
  /**
   * Addiert zwei Zahlen
   * @param {number} a - Erste Zahl
   * @param {number} b - Zweite Zahl
   * @returns {number} Summe der beiden Zahlen
   */
  add(a, b) {
    return a + b;
  }

  /**
   * Subtrahiert b von a
   * @param {number} a - Erste Zahl
   * @param {number} b - Zweite Zahl
   * @returns {number} Differenz der beiden Zahlen
   */
  subtract(a, b) {
    return a - b;
  }

  /**
   * Multipliziert zwei Zahlen
   * @param {number} a - Erste Zahl
   * @param {number} b - Zweite Zahl
   * @returns {number} Produkt der beiden Zahlen
   */
  multiply(a, b) {
    return a * b;
  }

  /**
   * Dividiert a durch b
   * @param {number} a - Dividend
   * @param {number} b - Divisor
   * @returns {number} Quotient der Division
   * @throws {Error} Wenn der Divisor 0 ist
   */
  divide(a, b) {
    if (b === 0) {
      throw new Error('Division durch Null ist nicht erlaubt.');
    }
    return a / b;
  }

  /**
   * Berechnet a hoch b
   * @param {number} a - Basis
   * @param {number} b - Exponent
   * @returns {number} a hoch b
   */
  power(a, b) {
    return Math.pow(a, b);
  }
}

module.exports = Calculator;
```

### 3.3 Committe die Änderung

```bash
# Füge die Änderung hinzu
git add src/calculator.js

# Commit erstellen
git commit -m "Füge power-Methode zum Taschenrechner hinzu"
```

Du solltest eine Bestätigung sehen, dass die Änderung committet wurde.

### 3.4 Push den Branch und erstelle einen Pull Request

```bash
# Push den Branch auf GitHub
git push -u origin feature/power-function
```

Nach dem Push wirst du eine Ausgabe sehen, die anzeigt, dass der Branch auf GitHub hochgeladen wurde, mit einem Link zum Erstellen eines Pull Requests.

Gehe zu deinem GitHub-Repository und erstelle einen Pull Request:
1. Du solltest eine Nachricht sehen, die vorschlägt, einen Pull Request für deinen neuen Branch zu erstellen
2. Klicke auf "Compare & pull request"
3. Gib einen Titel (z.B. "Füge Power-Methode hinzu") und eine Beschreibung (z.B. "Implementiert die Potenzierungsfunktion") für deinen Pull Request ein
4. Klicke auf "Create pull request"

### 3.5 Workflow-Ausführung für den Pull Request überprüfen

Nach dem Erstellen des Pull Requests wird der Test-Workflow automatisch ausgeführt. Du kannst den Fortschritt im "Actions"-Tab verfolgen.

Beachte jedoch, dass die Testabdeckung sinken wird, da wir eine neue Methode ohne Tests hinzugefügt haben. Der Pull Request könnte aufgrund fehlender Tests fehlschlagen, wenn wir strikte Testabdeckungsanforderungen haben.

### 3.6 Tests für die neue Funktion hinzufügen

Kehre zurück zu deinem lokalen Repository und öffne `test/calculator.test.js`. Füge einen neuen Testblock für die `power`-Methode hinzu:

```javascript
describe('power method', () => {
  test('should calculate power correctly for positive numbers', () => {
    expect(calculator.power(2, 3)).toBe(8);
    expect(calculator.power(3, 2)).toBe(9);
  });

  test('should handle zero exponent', () => {
    expect(calculator.power(5, 0)).toBe(1);
  });

  test('should handle negative exponent', () => {
    expect(calculator.power(2, -2)).toBe(0.25);
  });

  test('should handle zero base', () => {
    expect(calculator.power(0, 5)).toBe(0);
  });
});
```

Die Tests sollten nach dem Block für die `divide`-Methode eingefügt werden.

### 3.7 Committe und pushe die Tests

```bash
# Füge die Änderung hinzu
git add test/calculator.test.js

# Commit erstellen
git commit -m "Füge Tests für power-Methode hinzu"

# Push den Branch auf GitHub
git push origin feature/power-function
```

### 3.8 Überprüfe den aktualisierten Pull Request

Kehre zu deinem Pull Request auf GitHub zurück und überprüfe die neue Workflow-Ausführung. Jetzt sollte die Testabdeckung wieder in Ordnung sein.

## 4. Branch-Schutzregeln einrichten

Um sicherzustellen, dass nur Code, der alle Tests besteht, in den main-Branch gelangt, können wir Branch-Schutzregeln einrichten.

### 4.1 Branch-Schutzregeln konfigurieren

1. Gehe zu deinem Repository auf GitHub
2. Klicke auf "Settings"
3. Klicke auf "Branches" in der linken Seitenleiste
4. Unter "Branch protection rules" klicke auf "Add rule"
5. Gib "main" als Branch-Namen ein
6. Aktiviere "Require status checks to pass before merging"
7. Suche nach dem "test" Job und wähle ihn aus
8. Optional: Aktiviere "Require pull request reviews before merging"
9. Klicke auf "Create" oder "Save changes"

**Warum Branch-Schutzregeln wichtig sind:**
- Sie verhindern direktes Pushen in den main-Branch
- Sie stellen sicher, dass alle Tests bestanden werden müssen, bevor Code gemergt werden kann
- Sie ermöglichen Code-Reviews, wenn gewünscht
- Sie verbessern die Codequalität und Stabilität

### 4.2 Pull Request mergen

Nachdem alle Tests bestanden wurden und die Branch-Schutzregeln erfüllt sind, kannst du deinen Pull Request mergen:

1. Gehe zu deinem Pull Request
2. Klicke auf "Merge pull request"
3. Klicke auf "Confirm merge"

## 5. Erweiterte CI-Funktionen

Lass uns nun einige fortgeschrittenere CI-Funktionen zu unserem Workflow hinzufügen.

### 5.1 Testreport-Plugin hinzufügen

Installiere das Jest-Reporter-Plugin für GitHub Actions:

```bash
# Kehre zum main-Branch zurück
git checkout main
git pull

# Erstelle einen neuen Branch
git checkout -b feature/test-reports

# Installiere das Plugin
npm install --save-dev jest-github-actions-reporter
```

Diese Änderung wird auch die `package.json` und `package-lock.json` aktualisieren, um die neue Abhängigkeit zu berücksichtigen.

**Erklärung:**
- Das `jest-github-actions-reporter`-Plugin zeigt Testergebnisse als Annotationen direkt in der GitHub-Oberfläche an
- Dies macht es einfacher, fehlgeschlagene Tests zu erkennen und zu beheben

Aktualisiere die `jest.config.js`-Datei:

```javascript
module.exports = {
  verbose: true,
  testEnvironment: 'node',
  collectCoverage: true,
  collectCoverageFrom: ['src/**/*.js'],
  coverageThreshold: {
    global: {
      branches: 80,
      functions: 80,
      lines: 80,
      statements: 80
    }
  },
  coverageReporters: ['text', 'lcov'],
  // Hilft bei der korrekten Auflösung von Pfaden in verschiedenen Umgebungen
  roots: ['<rootDir>/src/', '<rootDir>/test/'],
  testMatch: ['**/__tests__/**/*.js?(x)', '**/?(*.)+(spec|test).js?(x)'],
  moduleFileExtensions: ['js', 'json', 'jsx', 'node'],
  // Hinzufügen des GitHub Actions Reporters
  reporters: process.env.GITHUB_ACTIONS
    ? ['default', 'jest-github-actions-reporter']
    : ['default'],
  testLocationInResults: true
};
```

### 5.2 Workflow-Datei aktualisieren

Aktualisiere die `.github/workflows/test.yml`-Datei:

```yaml
name: Test Workflow

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]
  workflow_dispatch:

jobs:
  test:
    name: Tests und Linting
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
      
      - name: Run linting
        run: npm run lint
      
      - name: Run tests
        run: npm test
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
      
      - name: Upload coverage report
        uses: actions/upload-artifact@v4
        with:
          name: coverage-report
          path: coverage/
      
      - name: Check test coverage
        run: |
          COVERAGE=$(node -e "const fs=require('fs'); const coverage=JSON.parse(fs.readFileSync('./coverage/coverage-summary.json')); console.log(coverage.total.lines.pct);")
          echo "Test coverage: $COVERAGE%"
          node -e "if (parseFloat(process.env.COVERAGE) < 80) { console.error('Test coverage is below 80%'); process.exit(1); }"
        env:
          COVERAGE: $COVERAGE
```

**Erklärung der neuen Elemente:**
- Wir fügen die Umgebungsvariable `GITHUB_TOKEN` hinzu, die für Annotationen benötigt wird
- Wir fügen einen Schritt hinzu, der die Testabdeckung überprüft und fehlschlägt, wenn sie unter 80% liegt
- Wir verwenden Node.js für den Coverage-Check, um Plattformunabhängigkeit zu gewährleisten

### 5.3 Committe und pushe die Änderungen

```bash
# Füge die Änderungen hinzu
git add jest.config.js .github/workflows/test.yml package.json package-lock.json

# Commit erstellen
git commit -m "Füge verbesserte Testreports und Testabdeckungsprüfung hinzu"

# Push den Branch auf GitHub
git push -u origin feature/test-reports
```

### 5.4 Erstelle einen Pull Request und merge ihn

Erstelle einen Pull Request für deinen `feature/test-reports`-Branch und merge ihn, sobald die Tests bestanden wurden.

## 6. Matrix-Builds für verschiedene Node.js-Versionen

Lass uns unseren Workflow erweitern, um die Anwendung auf verschiedenen Node.js-Versionen zu testen.

### 6.1 Erstelle einen neuen Branch

```bash
# Kehre zum main-Branch zurück
git checkout main
git pull

# Erstelle einen neuen Branch
git checkout -b feature/matrix-builds
```

### 6.2 Aktualisiere die Workflow-Datei

Aktualisiere die `.github/workflows/test.yml`-Datei:

```yaml
name: Test Workflow

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]
  workflow_dispatch:

jobs:
  test:
    name: Node ${{ matrix.node-version }} auf ${{ matrix.os }}
    runs-on: ${{ matrix.os }}
    strategy:
      matrix:
        node-version: [18.x, 20.x, 21.x]
        os: [ubuntu-latest, windows-latest]
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
      
      - name: Setup Node.js ${{ matrix.node-version }}
        uses: actions/setup-node@v4
        with:
          node-version: ${{ matrix.node-version }}
          cache: 'npm'
      
      - name: Install dependencies
        run: npm ci
      
      - name: Run linting
        run: npm run lint
      
      - name: Run tests
        run: npm test
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
      
      - name: Upload coverage report
        uses: actions/upload-artifact@v4
        with:
          name: coverage-report-${{ matrix.node-version }}-${{ matrix.os }}
          path: coverage/
          if-no-files-found: ignore
          
      - name: Check test coverage
        if: matrix.os == 'ubuntu-latest' # Führe den Coverage-Check nur auf einem Betriebssystem aus
        run: |
          COVERAGE=$(node -e "const fs=require('fs'); const coverage=JSON.parse(fs.readFileSync('./coverage/coverage-summary.json')); console.log(coverage.total.lines.pct);")
          echo "Test coverage: $COVERAGE%"
          node -e "if (parseFloat(process.env.COVERAGE) < 80) { console.error('Test coverage is below 80%'); process.exit(1); }"
        env:
          COVERAGE: $COVERAGE
```

**Erklärung der Matrix-Strategie:**
- Wir definieren eine Matrix mit verschiedenen Node.js-Versionen (alle aktuell unterstützt) und Betriebssystemen
- GitHub Actions erstellt für jede Kombination einen separaten Job
- Wir geben den Node.js-Versionen und Betriebssystemen im Job-Namen an
- Wir benennen die Artefakte eindeutig für jede Kombination
- Wir führen den Coverage-Check nur auf Ubuntu aus, um redundante Checks zu vermeiden

### 6.3 Committe und pushe die Änderungen

```bash
# Füge die Änderung hinzu
git add .github/workflows/test.yml

# Commit erstellen
git commit -m "Füge Matrix-Builds für verschiedene Node.js-Versionen und Betriebssysteme hinzu"

# Push den Branch auf GitHub
git push -u origin feature/matrix-builds
```

### 6.4 Erstelle einen Pull Request und merge ihn

Erstelle einen Pull Request für deinen `feature/matrix-builds`-Branch und merge ihn, sobald die Tests bestanden wurden.

## 7. Zusammenfassung

In dieser Übung hast du:

- Eine einfache JavaScript-Anwendung mit Tests erstellt
- Einen GitHub Actions Workflow für automatisierte Tests eingerichtet
- Den CI-Prozess mit Pull Requests und Branch-Schutzregeln durchlaufen
- Fortgeschrittene CI-Funktionen wie Test-Reports und Testabdeckungsprüfungen hinzugefügt
- Matrix-Builds für verschiedene Node.js-Versionen und Betriebssysteme konfiguriert

### 7.1 Konzepte und Best Practices

- **Kontinuierliche Integration (CI)**: Automatisiertes Testen bei jedem Push oder Pull Request
- **Test-Driven Development (TDD)**: Schreiben von Tests vor oder gleichzeitig mit dem Code
- **Testabdeckung**: Messung des Anteils des Codes, der durch Tests abgedeckt wird
- **Branch-Schutzregeln**: Verhindern, dass nicht getesteter Code in den Hauptbranch gelangt
- **Matrix-Builds**: Testen auf verschiedenen Plattformen und Umgebungen

### 7.2 Nächste Schritte

- Erweitere deine Anwendung um weitere Funktionen
- Füge End-to-End-Tests hinzu
- Integriere Code-Qualitätstools wie SonarCloud
- Richte eine kontinuierliche Deployment-Pipeline ein

Im nächsten Level werden wir uns mit dem automatischen Deployment einer JavaScript-Anwendung befassen!

### 7.3 Fehlerbehebung

Hier sind einige häufige Probleme und ihre Lösungen:

#### Allgemeine Probleme

- **Tests schlagen fehl**: Überprüfe die Testlogs und behebe die Fehler in deinem Code
- **Linting-Fehler**: Befolge die ESLint-Regeln oder passe sie nach Bedarf an
- **Niedrige Testabdeckung**: Schreibe zusätzliche Tests, um alle Codepfade abzudecken
- **Abhängigkeitsprobleme**: Stelle sicher, dass `package.json` und `package-lock.json` konsistent sind
- **npm ci schlägt fehl**: Dies kann passieren, wenn package.json und package-lock.json nicht synchron sind. Führe lokal `npm install` aus und committe die aktualisierte package-lock.json.

#### WSL-spezifische Probleme

- **Node.js-Pfadprobleme**: Falls du auf Probleme mit Node.js-Pfaden stößt, überprüfe mit `which node` und `which npm`, ob du die Linux-Version verwendest. Falls nicht, installiere Node.js mit nvm neu, wie zu Beginn der Übung beschrieben.

- **Zeilenumbruch-Probleme**: Wenn ESLint Fehler wegen Zeilenumbrüchen meldet, führe den folgenden Befehl aus:
  ```bash
  git config --global core.autocrlf input
  ```

- **Langsame Dateizugriffe**: Wenn die Tests in WSL sehr langsam laufen, überprüfe, ob du im Linux-Dateisystem (`/home/...`) und nicht im Windows-Dateisystem (`/mnt/c/...`) arbeitest.

- **Jest kann Konfiguration nicht finden**: Stelle sicher, dass die jest.config.js-Datei im richtigen Verzeichnis liegt und mit den richtigen Einstellungen konfiguriert ist, wie in dieser Übung gezeigt.

- **Pfadkonflikte in Fehlermeldungen**: Wenn Fehlermeldungen Windows-Pfade anzeigen, obwohl du in WSL arbeitest, verwende die nvm-Installation von Node.js, wie in der Übung beschrieben.

Viel Erfolg mit deiner CI/CD-Reise!
