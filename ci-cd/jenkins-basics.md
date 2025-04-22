# CI/CD Pipeline mit Jenkins

Diese Unterlagen sind deine umfassende Einführung in das Thema **CI/CD Pipeline mit Jenkins**. Hier findest du alle wichtigen Informationen und Erklärungen, um die Konzepte hinter Continuous Integration und Deployment mit Jenkins zu verstehen, eigene Pipelines zu erstellen und einen Vergleich zu GitHub Actions zu ziehen.

> **Hinweis:** CI/CD steht für Continuous Integration und Continuous Deployment. Mit automatisierten Pipelines sorgst du dafür, dass dein Code bei jedem Push oder Pull Request getestet, gebaut und – bei Bedarf – automatisch ausgeliefert wird. Jenkins ist eines der bekanntesten und am weitesten verbreiteten Open-Source-Tools für CI/CD-Automatisierung.

---

## 1. Was ist Jenkins?

Jenkins ist ein führendes Open-Source-Automatisierungstool, das sich auf Continuous Integration und Continuous Delivery spezialisiert hat. Im Gegensatz zu GitHub Actions, das direkt in GitHub integriert ist, ist Jenkins eine eigenständige Anwendung, die auf einem Server installiert und betrieben werden muss.

Mit Jenkins kannst du komplexe Automatisierungsaufgaben wie Tests, Builds und Deployments in einer Pipeline zusammenfassen. Während GitHub Actions seine Workflows in YAML-Dateien im Repository definiert, bietet Jenkins verschiedene Möglichkeiten zur Definition von Pipelines – von der webbasierten UI über die Jenkinsfile (eine Groovy-basierte DSL) bis hin zu deklarativen Pipelines in einer Jenkinsfile im Repository.

Ein Schlüsselvorteil von Jenkins ist seine Flexibilität und Erweiterbarkeit. Mit über 1.800 Plugins kannst du Jenkins an fast jeden Workflow und jede technische Infrastruktur anpassen. Dies ermöglicht die Integration mit nahezu jeder Entwicklungsumgebung, Build-Tool oder Deployment-Plattform. Während GitHub Actions ein Ökosystem von vorgefertigten Actions bietet, geht Jenkins einen Schritt weiter mit einem noch umfassenderen Plugin-System.

Jenkins unterstützt auch die Verteilung von Jobs auf mehrere "Agents" (vergleichbar mit Runnern bei GitHub Actions), was besonders nützlich ist, wenn du verschiedene Betriebssysteme oder Hardwareanforderungen hast. Im Gegensatz zu GitHub Actions, wo du zwischen gehosteten und selbstgehosteten Runnern wählen kannst, betreibst du bei Jenkins in der Regel deine eigene Infrastruktur, hast aber dadurch volle Kontrolle über die Umgebung.

Um mit Jenkins zu starten, musst du eine Instanz einrichten, konfigurieren und verwalten – ein deutlicher Unterschied zu GitHub Actions, wo du sofort loslegen kannst, sobald du ein Repository hast.

---

## 2. Deine erste Jenkinsfile verstehen

Eine Jenkinsfile in Jenkins erfüllt einen ähnlichen Zweck wie die Workflow-Datei in GitHub Actions, jedoch mit einer anderen Syntax und Struktur. In der deklarativen Pipeline-Syntax definierst du deine Pipeline in einer `Jenkinsfile` an der Wurzel deines Repositorys. Diese Datei wird in Groovy geschrieben und enthält verschiedene Abschnitte.

Der `pipeline`-Block umschließt die gesamte Definition. Im Abschnitt `agent` legst du fest, wo deine Pipeline ausgeführt wird – beispielsweise auf einem beliebigen verfügbaren Agent mit `agent any` oder einem spezifischen Label mit `agent { label 'my-label' }`. Dies entspricht dem `runs-on`-Parameter bei GitHub Actions.

Mit `environment` definierst du globale Umgebungsvariablen für die gesamte Pipeline, ähnlich dem `env`-Block in GitHub Actions. Die `stages`-Sektion ist das Herzstück deiner Pipeline und enthält individuelle `stage`-Blöcke, die den einzelnen Jobs in GitHub Actions ähneln. Jede Stage umfasst `steps`, die die eigentlichen Aktionen ausführen, sei es durch Shell-Befehle mit `sh` oder durch Nutzung von Jenkins-Plugins.

Für die Fehlerbehandlung und Aufräumarbeiten bietet Jenkins den `post`-Abschnitt, in dem du Aktionen für verschiedene Ergebnisse deiner Pipeline (erfolgreich, fehlgeschlagen, immer) definieren kannst. Dies entspricht den if-Conditions in GitHub Actions, ist aber strukturierter eingebunden.

Die Bedingungslogik wird in Jenkins mit `when`-Blöcken realisiert, die du innerhalb einer Stage verwenden kannst, um festzulegen, ob sie ausgeführt werden soll – ähnlich wie `if`-Ausdrücke in GitHub Actions. Auch hier kannst du auf Branch-Namen, Tags oder andere Kontext-Variablen zugreifen.

Für Parallelisierung bietet Jenkins das `parallel`-Keyword, das die gleichzeitige Ausführung mehrerer Stages ermöglicht – vergleichbar mit der Matrix-Strategie in GitHub Actions, aber mit einer anderen Struktur und Syntax.

---

## 3. Architektur einer CI/CD-Pipeline mit Jenkins

### 3.0 Visuelles Beispiel
```text
                 +-------------+
    SCM Event    |  Trigger    |        Pipeline startet bei SCM-Events oder Zeitplan
    ───────────► |  (webhook)  |                      |
                 +-------------+                      |
                        |                             |
                        v                             v
               +---------------+               +---------------+
               | Build & Test  |               |   Schedule    |
               |   Stage       |◄──────────────| (cron-Timer)  |
               +---------------+               +---------------+
                        |                             |
                        v                             v
               +---------------+               +---------------+
               |  Deploy Stage |               | Manuelle      |
               |               |               | Ausführung    |
               +---------------+               +---------------+
                        |
                        v
                  +------------+
                  | Artifacts  |
                  | (Archiv)   |
                  +------------+
```

Diese schematische Darstellung zeigt, wie eine einfache Jenkins-Pipeline abläuft: Ein SCM-Event (z.B. Push) oder ein Zeitplan löst die Pipeline aus, die zuerst die Build-&-Test-Stage startet und danach das Deployment durchführt. Am Ende werden Artefakte archiviert, die später abgerufen werden können.

### 3.1 Trigger definieren

Jenkins bietet verschiedene Trigger-Mechanismen, um Pipeline-Ausführungen automatisch zu starten. Im Gegensatz zu GitHub Actions, wo Trigger direkt über das `on`-Schlüsselwort an Events wie Push oder Pull Request gebunden werden, verwendet Jenkins einen speziellen `triggers`-Block innerhalb der Pipeline-Definition.

```groovy
// Pipeline-Trigger Konfiguration
pipeline {
    agent any
    
    triggers {
        // Polling des Repositorys (ähnlich zu GitHub Actions push/pull_request)
        pollSCM('H/15 * * * *')  // Alle 15 Min prüfen
        // Zeitgesteuerte Ausführung (ähnlich zu GitHub Actions schedule)
        cron('0 2 * * *')         // Täglich um 02:00 Uhr
    }
    
    // Rest der Pipeline...
}
```

**Detaillierte Erklärung:**

- `pollSCM('H/15 * * * *')`: Dieser Trigger fragt das SCM (Source Code Management, z.B. Git) regelmäßig nach Änderungen ab. Die Syntax `'H/15 * * * *'` ist ein Cron-Ausdruck, der Jenkins anweist, alle 15 Minuten zu prüfen, ob es neue Commits gibt. Das `H` steht für "Hash", was bedeutet, dass Jenkins die genaue Minute innerhalb des 15-Minuten-Intervalls selbst bestimmt, um die Serverlast zu verteilen. Dieser Ansatz unterscheidet sich von GitHub Actions, wo das Repository-Hosting und die CI-Plattform direkt integriert sind und Ereignisse sofort per Webhook gemeldet werden.

- `cron('0 2 * * *')`: Ähnlich wie der `schedule`-Trigger in GitHub Actions, startet dieser die Pipeline jeden Tag um 2 Uhr morgens, unabhängig davon, ob Code-Änderungen vorliegen. Dies ist nützlich für nächtliche Builds, regelmäßige Tests oder geplante Deployments.

Neben diesen Standardtriggern unterstützt Jenkins auch andere Auslöser:

- `upstream`: Startet deine Pipeline, wenn ein anderer Job erfolgreich abgeschlossen wurde
- `githubPush`: Ein spezieller Trigger (erfordert GitHub-Plugin), der auf GitHub-Webhooks reagiert
- Manuelle Trigger über die Jenkins-Oberfläche oder via API

Während GitHub Actions eng mit dem GitHub-Event-System integriert ist und direkt auf Repository-Ereignisse reagiert, erfordert Jenkins oft zusätzliche Konfiguration für ähnliche Integrationen. Die Stärke von Jenkins liegt jedoch in der Flexibilität, mit der du benutzerdefinierte Trigger und komplexe Abhängigkeiten zwischen Jobs definieren kannst.

### 3.2 Build & Test Stage

Die Build- und Test-Stage ist typischerweise der erste Hauptabschnitt einer Jenkins-Pipeline. Hier wird der Code ausgecheckt, Abhängigkeiten installiert und Tests ausgeführt – ähnlich wie in GitHub Actions, aber mit einer anderen Syntax und Struktur.

```groovy
// Build- und Test-Stage Definition
stages {
    stage('Build') {
        steps {
            // Code auschecken (ähnlich zu actions/checkout@v4)
            checkout scm
            // Dependencies installieren
            sh 'npm ci'
            // Tests ausführen
            sh 'npm test'
        }
    }
}
```

**Detaillierte Erklärung:**

- `stage('Build')`: Definiert eine benannte Phase in der Pipeline. Dies verbessert die Übersichtlichkeit in der Jenkins-Oberfläche und ermöglicht es, den Fortschritt der Pipeline visuell zu verfolgen. In GitHub Actions würde dies einem einzelnen `job` entsprechen.

- `steps`: Dieser Block enthält die konkreten Aktionen, die innerhalb der Stage ausgeführt werden. In GitHub Actions würde dies dem `steps`-Array innerhalb eines Jobs entsprechen.

- `checkout scm`: Lädt den Code aus dem konfigurierten Source Code Management-System (wie Git). Dies entspricht der `actions/checkout@v4`-Action in GitHub Actions, ist aber in Jenkins kürzer, da das SCM bereits auf Job-Ebene konfiguriert ist. Der `scm`-Parameter verwendet automatisch die im Job konfigurierten Repository-Einstellungen.

- `sh 'npm ci'` und `sh 'npm test'`: Führt Shell-Befehle aus, ähnlich wie `run: npm ci` und `run: npm test` in GitHub Actions. Der Befehl `sh` ist für Unix/Linux-basierte Systeme; für Windows würde man `bat` verwenden.

Jenkins bietet zusätzliche Möglichkeiten für diese Stage:

- Tools-Deklaration: Mit dem `tools`-Block kannst du spezifische Versionen von Build-Tools wie JDK, Maven oder Gradle verwenden.
- Workspace-Handling: Jenkins bietet feinere Kontrolle über den Workspace, in dem deine Pipeline ausgeführt wird.
- Build-Wrapper: Du kannst spezielle Umgebungen oder Einstellungen für den Build-Prozess definieren.

Im Vergleich zu GitHub Actions ist die Jenkins-Syntax für diese Schritte expliziter und spiegelt die längere Geschichte von Jenkins als Build-Tool wider. GitHub Actions ist etwas kompakter, während Jenkins mehr Flexibilität für komplexe Build-Szenarien bietet.

### 3.3 Deployment-Stage

Die Deployment-Stage ist ein kritischer Teil der CI/CD-Pipeline, bei dem erfolgreich getestete Code-Änderungen in eine Zielumgebung ausgerollt werden. Jenkins bietet mächtige Möglichkeiten, um bedingte Deployments zu steuern.

```groovy
// Deployment-Stage Definition
stage('Deploy') {
    // Nur ausführen, wenn Build erfolgreich war (ähnlich zu needs: build)
    when {
        branch 'main'  // Nur auf dem main-Branch
    }
    steps {
        // Deployment-Skript ausführen
        sh './deploy.sh'
    }
}
```

**Detaillierte Erklärung:**

- `stage('Deploy')`: Definiert die Deployment-Phase in der Pipeline. Diese Stage folgt typischerweise nach der Build- und Test-Stage und wird nur ausgeführt, wenn die vorherigen Stages erfolgreich waren. In GitHub Actions würde dies einem separaten Job entsprechen, der mit `needs: [build]` konfiguriert ist.

- `when { branch 'main' }`: Diese Bedingung stellt sicher, dass die Deployment-Stage nur ausgeführt wird, wenn die Pipeline auf dem `main`-Branch läuft. Dies entspricht der `if: github.ref == 'refs/heads/main'`-Bedingung in GitHub Actions. Jenkins bietet eine Vielzahl von Bedingungen, die du im `when`-Block kombinieren kannst, z.B.:
  - `expression { return params.DEPLOY == true }`: Prüft einen benutzerdefinierten Parameter
  - `environment name: 'PRODUCTION'`: Prüft die Umgebung
  - `anyOf { branch 'main'; branch 'release-*' }`: Mehrere Branch-Muster
  - `not { branch 'dev' }`: Negative Bedingung

- `sh './deploy.sh'`: Führt ein Deployment-Skript aus, ähnlich wie in GitHub Actions. Die Stärke von Jenkins liegt hier in der Integration mit verschiedenen Deployment-Tools und -Plattformen über Plugins, wie z.B.:
  - Kubernetes Deployment über das Kubernetes-Plugin
  - AWS Deployments über das AWS-Plugin
  - Docker-Container-Deployment über Docker-Plugins

Jenkins bietet zudem erweiterte Deployment-Muster:

- **Blue/Green Deployment**: Mit Parametern kannst du zwischen verschiedenen Umgebungen wechseln
- **Canary Releases**: Schrittweise Einführung neuer Funktionen für einen Teil der Nutzer
- **Rollback-Strategien**: Automatisierte Wiederherstellung bei fehlgeschlagenem Deployment

Im Vergleich zu GitHub Actions bietet Jenkins mehr Built-in-Funktionen für komplexe Deployment-Strategien, erfordert aber auch mehr Konfigurationsaufwand. GitHub Actions ist einfacher einzurichten, besonders für GitHub-basierte Projekte, während Jenkins eine umfassendere Kontrolle über den gesamten Deployment-Prozess ermöglicht.

---

## 4. Einrichtung & Voraussetzungen

Bevor du mit Jenkins loslegst, überprüfe folgende Punkte:

1. **Server oder VM** – für die Jenkins-Installation.
2. **Java installiert** – Jenkins benötigt Java zum Laufen:
   ```bash
   java -version
   ```
3. **Git installiert:**
   ```bash
   git --version
   ```
4. **Zugriff auf dein Repository** – über HTTPS oder SSH.
5. **Administrative Rechte** – für die Installation und Konfiguration.

Um Jenkins zu installieren (Linux-Beispiel):

```bash
# Jenkins-Repository hinzufügen
wget -q -O - https://pkg.jenkins.io/debian-stable/jenkins.io.key | sudo apt-key add -
sudo sh -c 'echo deb https://pkg.jenkins.io/debian-stable binary/ > /etc/apt/sources.list.d/jenkins.list'
sudo apt-get update
sudo apt-get install jenkins
```

Starte den Jenkins-Service:

```bash
sudo systemctl start jenkins
```

Öffne http://localhost:8080 im Browser und folge den Einrichtungsschritten.

---

## 5. Wichtige Konzepte & Beispiele

### 5.1 Jenkinsfile

- **Pfad:** In der Regel im Stammverzeichnis deines Repositorys.
- **Syntax:** Deklarativ (empfohlen) oder Skript-basiert.

```groovy
pipeline {
    agent any
    
    stages {
        stage('Hello') {
            steps {
                echo 'Hello, Jenkins!'
            }
        }
    }
}
```

**Erklärung:**

- `agent any` führt die Pipeline auf jedem verfügbaren Agent aus.
- `echo` gibt Text in der Konsole aus.

### 5.2 Umgebungsvariablen & Credentials

Umgebungsvariablen werden im `environment`-Block definiert:

```groovy
environment {
    NODE_ENV = 'production'
}

stages {
    stage('Build') {
        steps {
            echo "Environment: ${NODE_ENV}"
        }
    }
}
```

Für Credentials verwendest du den Credentials-Manager in Jenkins und bindest sie ein:

```groovy
environment {
    // Credentials als Umgebungsvariable verfügbar machen
    MY_TOKEN = credentials('my-token-id')
}

steps {
    sh 'curl -H "Authorization: token ${MY_TOKEN}" https://api.github.com/repos/user/repo'
}
```

### 5.3 Artefakte

Artefakte werden in Jenkins archiviert und gespeichert:

- Archivieren:
  ```groovy
  steps {
      // Build-Schritte...
      
      // Artefakte archivieren
      archiveArtifacts artifacts: 'dist/**', fingerprint: true
  }
  ```
- In einer anderen Stage verwenden:
  ```groovy
  stage('Deploy') {
      steps {
          // Vorherigen Build-Output verwenden
          copyArtifacts(projectName: '${JOB_NAME}', selector: specific('${BUILD_NUMBER}'))
      }
  }
  ```

---

## 6. Syntax & Aufbau einer Jenkins-Pipeline

In einer Jenkins-Pipeline definierst du in Groovy-Syntax, wie dein Automatisierungsprozess ablaufen soll. Im Gegensatz zur YAML-Notation von GitHub Actions folgt Jenkins einer programmatischen Struktur. Eine deklarative Pipeline beginnt stets mit dem Block `pipeline`, der den gesamten Prozess umschließt.

Im `agent`-Block legst du fest, wo deine Pipeline ausgeführt wird. Du kannst `any` für einen beliebigen verfügbaren Agent verwenden oder spezifische Labels, Docker-Container oder sogar Kubernetes-Pods definieren. Dies entspricht der `runs-on`-Direktive in GitHub Actions, bietet aber mehr Auswahlmöglichkeiten und Kontrolle.

Unter `environment` definierst du globale Umgebungsvariablen, die in allen nachfolgenden Stages und Steps verfügbar sind. Dies entspricht dem `env`-Block in GitHub Actions, funktioniert aber auch für komplexere Szenarien wie das Laden von Credentials direkt aus dem Jenkins Credentials Store.

Der Kern deiner Pipeline liegt im `stages`-Block, der einzelne `stage`-Blöcke enthält – vergleichbar mit den `jobs` in GitHub Actions. Jede Stage repräsentiert eine Phase deines CI/CD-Prozesses und kann durch den `when`-Block konditionell ausgeführt werden, ähnlich wie `if`-Conditions in GitHub Actions.

Innerhalb jeder Stage definierst du unter `steps` die konkreten Schritte. Anstatt zwischen `uses` und `run` zu unterscheiden wie in GitHub Actions, verwendest du in Jenkins Groovy-Methoden wie `sh` für Shell-Befehle, `bat` für Windows-Befehle oder Plugin-spezifische Schritte wie `junit` zum Verarbeiten von Testergebnissen.

Für die Ausführung paralleler Prozesse bietet Jenkins den `parallel`-Block innerhalb einer Stage, der mehrere Unterprozesse gleichzeitig starten kann. Dies ist ähnlich zur Matrix-Strategie in GitHub Actions, aber flexibler in der Umsetzung.

Ein besonderes Merkmal von Jenkins ist der `post`-Block, in dem du Aktionen definieren kannst, die nach Abschluss der Pipeline ausgeführt werden – sei es bei Erfolg, Misserfolg oder in jedem Fall. Dies ermöglicht eine elegante Aufräumlogik, die in GitHub Actions oft durch zusätzliche Steps mit Bedingungslogik realisiert wird.

Für parameterisierte Pipelines bietet Jenkins den `parameters`-Block, mit dem du Benutzereingaben definieren kannst – von einfachen Strings über Boolesche Werte bis hin zu Auswahlmenüs. In GitHub Actions wird Ähnliches durch `workflow_dispatch`-Inputs erreicht, aber mit weniger Auswahlmöglichkeiten.

Mit dieser Struktur und Syntax ermöglicht Jenkins einen programmatischen Ansatz zur Definition von CI/CD-Pipelines, der zwar eine steilere Lernkurve als GitHub Actions haben kann, aber dafür mehr Flexibilität und Kontrolle bietet.

---

## 7. Best Practices
Im Folgenden findest du einige Empfehlungen, um deine Jenkins-Pipelines stabil, performant und sicher zu gestalten:

- **Pipeline as Code:** Halte deine Pipeline-Definitionen im selben Repository wie deinen Code (Jenkinsfile), um Versionskontrolle und Nachvollziehbarkeit zu gewährleisten.
- **Shared Libraries:** Erstelle wiederverwendbare Funktionen in Jenkins Shared Libraries, um Code-Duplikation zu vermeiden und Wartbarkeit zu verbessern.
- **Caching:** Nutze das `stash`/`unstash`-Feature von Jenkins, um Dateien zwischen Stages zu teilen und Wiederholungen zu vermeiden.
- **Credentials Management:** Verwende den Jenkins Credential Store für sichere Speicherung von Tokens und Passwörtern. Vermeide es, sensible Informationen im Klartext in der Pipeline zu speichern.
- **Agents Management:** Organisiere deine Agents mit sinnvollen Labels, um Jobs gezielt auf geeigneten Maschinen laufen zu lassen.
- **Timeout & Retry:** Setze Timeouts für deine Stages und verwende `retry`-Mechanismen für instabile Schritte, um Robustheit zu erhöhen.
- **Regelmäßige Wartung:** Aktualisiere Jenkins und Plugins regelmäßig, um Sicherheitslücken zu schließen und von neuen Features zu profitieren.

---

## 8. Beispiel: Vollständige Jenkinsfile mit Kommentaren

Im Folgenden siehst du eine komplette `Jenkinsfile`, Zeile für Zeile kommentiert, damit du verstehst, was jede Zeile bewirkt:

```groovy
// 1. Die Pipeline-Definition beginnt hier
pipeline {
    // 2. Festlegen, wo die Pipeline ausgeführt wird
    agent any
    
    // 3. Globale Optionen für die Pipeline
    options {
        // Artefakte für maximal 5 Builds behalten
        buildDiscarder(logRotator(numToKeepStr: '5'))
        // Zeitlimit für den gesamten Pipeline-Lauf
        timeout(time: 60, unit: 'MINUTES')
    }
    
    // 4. Pipeline-Trigger definieren
    triggers {
        // Alle 15 Minuten nach Änderungen im SCM prüfen
        pollSCM('H/15 * * * *')
    }
    
    // 5. Globale Umgebungsvariablen
    environment {
        GREETING = "Hello, Jenkins!"
        // Credentials als Umgebungsvariable laden
        GITHUB_TOKEN = credentials('github-token')
    }
    
    // 6. Pipeline-Parameter für manuelle Ausführungen
    parameters {
        // Dropdown-Menü
        choice(name: 'ENVIRONMENT', choices: ['dev', 'staging', 'prod'], description: 'Deployment-Umgebung')
        // Textfeld
        string(name: 'VERSION', defaultValue: '1.0.0', description: 'Version für das Deployment')
    }
    
    // 7. Einzelne Phasen der Pipeline
    stages {
        // 8. Phase: Code auschecken
        stage('Checkout') {
            steps {
                // Git-Repository auschecken
                checkout scm
            }
        }
        
        // 9. Phase: Build und Test
        stage('Build & Test') {
            steps {
                // Die Begrüßung ausgeben
                echo "${env.GREETING}"
                
                // Shell-Befehle ausführen
                sh '''
                    echo "Repository: ${JOB_NAME}"
                    echo "Branch: ${GIT_BRANCH}"
                    npm ci
                    npm test
                '''
                
                // Test-Ergebnisse sammeln
                junit 'test-results/*.xml'
            }
        }
        
        // 10. Phase: Deployment (bedingt)
        stage('Deploy') {
            // Nur ausführen, wenn auf main/master und prod ausgewählt
            when {
                expression { 
                    return (env.GIT_BRANCH == 'origin/main' || env.GIT_BRANCH == 'origin/master') &&
                           params.ENVIRONMENT == 'prod'
                }
            }
            steps {
                // Eine Datei erstellen
                sh '''
                    echo "Diese Datei wurde durch Jenkins erstellt." > generated-file.txt
                    echo "Zeitstempel: $(date)" >> generated-file.txt
                    echo "Version: ${params.VERSION}" >> generated-file.txt
                '''
                
                // Deployment ausführen
                sh './deploy.sh ${params.ENVIRONMENT}'
            }
        }
    }
    
    // 11. Nach-Pipeline-Aktionen
    post {
        // Immer ausführen
        always {
            // Artefakte archivieren
            archiveArtifacts artifacts: 'generated-file.txt', allowEmptyArchive: true
        }
        // Nur bei Erfolg
        success {
            echo "Pipeline erfolgreich abgeschlossen!"
        }
        // Nur bei Fehler
        failure {
            // Benachrichtigung senden (erfordert Email-Plugin)
            emailext (
                subject: "Pipeline fehlgeschlagen: ${currentBuild.fullDisplayName}",
                body: "Die Pipeline ist fehlgeschlagen. Siehe ${env.BUILD_URL}",
                to: 'team@example.com'
            )
        }
    }
}
```

### Erklärung der Syntax anhand des Beispiels

Die obige Jenkinsfile nutzt die deklarative Pipeline-Syntax von Jenkins: Der gesamte Ablauf wird in einem `pipeline`-Block definiert. Im `agent`-Block wird festgelegt, dass die Pipeline auf jedem verfügbaren Agent ausgeführt werden kann. Unter `options` werden allgemeine Pipeline-Einstellungen wie Zeitlimits und die Anzahl zu behaltender Build-Logs definiert.

Mit `triggers` wird das automatische Starten der Pipeline konfiguriert, ähnlich dem `on`-Block in GitHub Actions, aber mit einer anderen Syntax und Logik. Der `environment`-Block entspricht dem gleichnamigen Konzept in GitHub Actions und erlaubt die Definition globaler Variablen.

Eine Besonderheit von Jenkins sind die `parameters`, die eine interaktive Konfiguration der Pipeline vor dem Start ermöglichen – vergleichbar mit `workflow_dispatch.inputs` in GitHub Actions, aber mit mehr Optionen und besserer Integration in die UI.

Im `stages`-Block werden die einzelnen Phasen der Pipeline definiert, vergleichbar mit `jobs` in GitHub Actions. Jede `stage` enthält `steps` mit konkreten Aktionen. Anstelle von `uses` und `run` in GitHub Actions verwendest du Groovy-Methoden wie `sh` für Shell-Befehle.

Die bedingte Ausführung wird in Jenkins mit `when`-Blöcken realisiert, die eine ähnliche Funktionalität wie `if`-Ausdrücke in GitHub Actions bieten, aber mit der vollen Mächtigkeit der Groovy-Sprache.

Ein einzigartiges Feature von Jenkins ist der `post`-Block, in dem du Aktionen definieren kannst, die nach Abschluss der Pipeline ausgeführt werden – sei es bei Erfolg (`success`), Misserfolg (`failure`) oder in jedem Fall (`always`). Dies erleichtert das Handling von Benachrichtigungen und Aufräumarbeiten im Vergleich zu GitHub Actions.

Mit dieser kommentierten Komplettansicht und der begleitenden Erklärung hast du nun ein vollständiges Beispiel für eine Jenkins-Pipeline sowie das Verständnis dafür, wie jede Zeile ihre Wirkung entfaltet. Viel Spaß beim Ausprobieren und Anpassen!

---

## 9. Jenkins vs. GitHub Actions: Ein Vergleich

Um die Unterschiede und Gemeinsamkeiten von Jenkins und GitHub Actions besser zu verstehen, hier eine Gegenüberstellung wichtiger Aspekte:

### Hosting & Setup
- **Jenkins:** 
  - Selbstgehostet auf eigener Infrastruktur, erfordert Installation und Wartung
  - Mehr Kontrolle über die Infrastruktur und Sicherheit
  - Höherer initialer Aufwand für Setup und Konfiguration
  - Regelmäßige Wartung und Updates notwendig

- **GitHub Actions:** 
  - Vollständig in GitHub integriert, keine separate Installation nötig
  - Sofort einsatzbereit für GitHub-Repositories
  - Automatische Updates und Wartung durch GitHub
  - Begrenzte Kontrolle über die Runner-Umgebung (außer bei selbstgehosteten Runnern)

### Syntax & Definition
- **Jenkins:** 
  - Groovy-basierte Jenkinsfile (deklarativ oder skriptbasiert)
  - Höhere Lernkurve durch programmatischen Ansatz
  - Mächtigere Ausdrucksmöglichkeiten durch vollständige Programmiersprache
  - Mehrere Definitionsmöglichkeiten: Jenkinsfile im Repository, über UI, über Job DSL

- **GitHub Actions:** 
  - YAML-basierte Workflow-Dateien im `.github/workflows`-Verzeichnis
  - Einfachere Syntax, schneller zu erlernen
  - Stärker strukturiert und weniger flexibel
  - Eindeutige Definition als Code im Repository

### Ausführungsumgebung
- **Jenkins:** 
  - Selbstverwaltete Agents auf eigener Hardware oder Cloud
  - Volle Kontrolle über die Ausführungsumgebung und Software-Installation
  - Unbegrenzte Ausführungszeit möglich
  - Keine direkten Nutzungsbeschränkungen

- **GitHub Actions:** 
  - GitHub-gehostete Runner (Linux, macOS, Windows) mit vorinstallierter Software
  - Selbstgehostete Runner für spezielle Anforderungen
  - Zeitliche Begrenzung der Workflow-Ausführung (6 Stunden für gehostete Runner)
  - Kontingente für Build-Minuten, abhängig vom GitHub-Plan
