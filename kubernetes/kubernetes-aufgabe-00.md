# Kubernetes HelloWorld-Übung: Erste Schritte in der Praxis

Willkommen zu dieser praktischen Einführung in Kubernetes. In dieser Übung führe ich dich Schritt für Schritt durch die Erstellung und Bereitstellung deiner ersten Kubernetes-Anwendung. Wir beginnen mit den grundlegenden Installationen und arbeiten uns bis zu einer laufenden Anwendung vor. Diese Anleitung ist bewusst ausführlich gestaltet, damit du nicht nur die Befehle ausführst, sondern auch verstehst, was im Hintergrund passiert. Solltest du Probleme oder Fragen haben, dann schreibe mir per E-mail, Slack oder sprich mich im Classroom an.

## Teil 1: Installation der Kubernetes-Ressourcen

Für deine Kubernetes-Umgebung benötigst du drei wesentliche Komponenten. Lass uns diese nacheinander installieren.

### 1.1 Docker installieren

Zuerst benötigen wir Docker – die Grundlage für unsere Containerisierung. Docker ermöglicht es, Anwendungen in portablen Containern zu verpacken. Stell dir Docker wie ein standardisiertes Transportsystem vor: Es sorgt dafür, dass deine Anwendung mit allen Abhängigkeiten in einem einheitlichen Format transportiert wird und überall gleich funktioniert – unabhängig von der Umgebung.

**Für Windows:**
1. Lade Docker Desktop für Windows von der [offiziellen Website](https://www.docker.com/products/docker-desktop) herunter
2. Führe die Installation aus und folge den Anweisungen
3. Starte deinen Computer neu
4. Starte Docker Desktop und prüfe, ob es läuft

**Für macOS:**
1. Lade Docker Desktop für Mac von der [offiziellen Website](https://www.docker.com/products/docker-desktop) herunter
2. Installiere die .dmg-Datei und folge den Anweisungen
3. Starte Docker Desktop und prüfe, ob es läuft

**Für Linux (Ubuntu):**
```bash
# Installiere benötigte Pakete
sudo apt-get update
sudo apt-get install -y apt-transport-https ca-certificates curl software-properties-common

# Füge Docker's offiziellen GPG-Schlüssel hinzu
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -

# Füge Docker-Repository hinzu
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"

# Installiere Docker
sudo apt-get update
sudo apt-get install -y docker-ce

# Füge deinen Benutzer zur Docker-Gruppe hinzu, um Docker ohne sudo ausführen zu können
sudo usermod -aG docker $USER
```

**Überprüfen der Installation:**
```bash
docker --version
docker run hello-world
```

### 1.2 kubectl installieren

Als nächstes installieren wir kubectl – das zentrale Steuerungswerkzeug für Kubernetes. Mit kubectl kommunizierst du mit deinem Kubernetes-Cluster, erstellst Ressourcen und überwachst den Zustand deiner Anwendungen. Es funktioniert wie eine Fernbedienung für deinen Cluster und erlaubt dir, Kommandos auszuführen und Informationen abzurufen.

**Für Windows:**
1. Lade die aktuelle Version von der [Kubernetes-Releases-Seite](https://storage.googleapis.com/kubernetes-release/release/stable.txt) herunter:
   ```
   curl -LO https://storage.googleapis.com/kubernetes-release/release/v1.28.4/bin/windows/amd64/kubectl.exe
   ```
2. Füge kubectl.exe zu deinem PATH hinzu oder verschiebe es in ein Verzeichnis, das bereits im PATH ist

**Für macOS:**
```bash
# Mit Homebrew
brew install kubectl

# Oder mit curl
curl -LO "https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/darwin/amd64/kubectl"
chmod +x ./kubectl
sudo mv ./kubectl /usr/local/bin/kubectl
```

**Für Linux:**
```bash
# Mit curl
curl -LO "https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl"
chmod +x ./kubectl
sudo mv ./kubectl /usr/local/bin/kubectl
```

**Überprüfen der Installation:**
```bash
kubectl version --client
```

### 1.3 Minikube installieren

Der dritte Baustein ist Minikube – eine Lösung, die einen vollständigen Kubernetes-Cluster auf deinem lokalen Rechner erzeugt. Minikube ist ideal für Lern- und Entwicklungszwecke, da es dir ermöglicht, mit Kubernetes zu experimentieren, ohne Cloud-Ressourcen zu benötigen. Es ist vergleichbar mit einer lokalen Testumgebung, in der du sicher Konfigurationen ausprobieren kannst, bevor du sie in einer Produktionsumgebung einsetzt.

**Für Windows:**
```
# Mit Chocolatey (vorher Chocolatey installieren)
choco install minikube

# Oder direkter Download
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-installer.exe
```

**Für macOS:**
```bash
# Mit Homebrew
brew install minikube

# Oder mit curl
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-darwin-amd64
chmod +x minikube-darwin-amd64
sudo mv minikube-darwin-amd64 /usr/local/bin/minikube
```

**Für Linux:**
```bash
# Mit curl
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
chmod +x minikube-linux-amd64
sudo mv minikube-linux-amd64 /usr/local/bin/minikube
```

**Überprüfen der Installation:**
```bash
minikube version
```

## Teil 2: Lokalen Kubernetes-Cluster einrichten

### 2.1 Minikube-Cluster starten

Nachdem alle Komponenten installiert sind, können wir unseren lokalen Kubernetes-Cluster starten. Mit einem einzigen Befehl erledigt Minikube mehrere komplexe Aufgaben für dich: Es richtet eine virtuelle Umgebung ein, installiert alle notwendigen Kubernetes-Komponenten und konfiguriert kubectl für die Kommunikation mit diesem Cluster. Dies ist vergleichbar mit dem Aufbau einer kompletten Serverinfrastruktur – nur dass es für dich automatisiert und auf deinem lokalen Rechner geschieht.

```bash
minikube start
```

**Was passiert hier?**
- Minikube lädt ein Image herunter, das alle notwendigen Kubernetes-Komponenten enthält
- Es startet eine virtuelle Umgebung (je nach System mit VirtualBox, HyperV oder direkt mit Docker)
- Es konfiguriert einen vollständigen Kubernetes-Cluster mit Master- und Worker-Komponenten
- Es konfiguriert kubectl so, dass es mit diesem Cluster kommuniziert

**Wichtige Informationen:**
- Der Startvorgang kann einige Minuten dauern, besonders beim ersten Mal
- Standardmäßig werden 2 CPUs und 2 GB RAM zugewiesen. Dies kannst du mit Parametern anpassen:
  ```bash
  minikube start --cpus=4 --memory=4096mb
  ```

### 2.2 Überprüfe den Cluster-Status

```bash
# Überprüfe den Minikube-Status
minikube status

# Zeige Node-Informationen an
kubectl get nodes

# Zeige Cluster-Informationen an
kubectl cluster-info
```

**Was du sehen solltest:**
- `minikube status` sollte "Running" für den Host, Kubelet und die API-Server anzeigen
- `kubectl get nodes` sollte einen Node namens "minikube" mit dem Status "Ready" anzeigen
- `kubectl cluster-info` sollte die URLs für den Kubernetes-Master und CoreDNS anzeigen

**Warum ist dies wichtig?**
Diese Befehle bestätigen, dass dein Cluster ordnungsgemäß läuft und bereit ist, Anwendungen zu hosten. Es ist wichtig, diesen Status zu überprüfen, bevor du versuchst, Anwendungen zu deployen.

## Teil 3: Erste Anwendung erstellen und bereitstellen

### 3.1 Hello World Deployment erstellen

Jetzt geht es an die eigentliche Anwendungsbereitstellung. In Kubernetes arbeiten wir mit "Deployments" – einer Art Blaupause für deine Anwendung. Ein Deployment definiert, welches Container-Image verwendet werden soll, wie viele Instanzen (Replicas) laufen sollen und wie Updates verwaltet werden. Du kannst dir ein Deployment wie einen Vertrag vorstellen: "Bitte stelle sicher, dass diese Anwendung mit diesen Spezifikationen läuft" – und Kubernetes kümmert sich darum, diesen Vertrag einzuhalten, selbst wenn einzelne Komponenten ausfallen.

**Erstelle eine Datei `helloworld.yaml` mit folgendem Inhalt:**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: helloworld
  labels:
    app: helloworld
spec:
  replicas: 1
  selector:
    matchLabels:
      app: helloworld
  template:
    metadata:
      labels:
        app: helloworld
    spec:
      containers:
      - name: helloworld
        image: paulbouwer/hello-kubernetes:1.10
        ports:
        - containerPort: 8080
        env:
        - name: MESSAGE
          value: "Hallo von meinem ersten Kubernetes-Deployment!"
```

**Was bedeutet dieser YAML-Code?**

- `apiVersion` und `kind`: Definieren, welche API-Version und welchen Ressourcentyp wir erstellen
- `metadata`: Enthält Namen und Labels für unser Deployment
- `spec`: Die Spezifikation des Deployments:
  - `replicas: 1`: Wir wollen eine Instanz unserer Anwendung laufen lassen
  - `selector`: Definiert, wie das Deployment die zu verwaltenden Pods findet
  - `template`: Die Pod-Vorlage:
    - `metadata`: Labels für die Pods
    - `spec`: Die Pod-Spezifikation:
      - `containers`: Liste der Container im Pod
        - `image`: Das Docker-Image (paulbouwer/hello-kubernetes:1.10)
        - `ports`: Der Container öffnet Port 8080
        - `env`: Umgebungsvariablen - hier setzen wir eine benutzerdefinierte Nachricht

**Wende das Deployment an:**

```bash
kubectl apply -f helloworld.yaml
```

**Was passiert hier?**
- kubectl sendet die YAML-Definition an den Kubernetes API-Server
- Der Deployment-Controller erstellt ein ReplicaSet
- Das ReplicaSet sorgt dafür, dass die gewünschte Anzahl an Pods (hier: 1) erstellt wird
- Der Scheduler weist den Pod einem Node zu
- Der Kubelet auf diesem Node startet den Container

### 3.2 Überprüfe das Deployment

```bash
# Überprüfe das Deployment
kubectl get deployments

# Überprüfe die Pods
kubectl get pods

# Zeige detaillierte Informationen zum Deployment an
kubectl describe deployment helloworld

# Zeige detaillierte Informationen zu einem Pod an (ersetze <pod-name>)
kubectl describe pod <pod-name>
```

**Was du sehen solltest:**
- `kubectl get deployments` zeigt dein "helloworld"-Deployment mit 1/1 READY
- `kubectl get pods` zeigt einen Pod mit dem Präfix "helloworld-" und dem Status "Running"
- `kubectl describe` zeigt detaillierte Informationen, einschließlich Events, die hilfreich für die Fehlersuche sind

**Warum ist dies wichtig?**
Diese Schritte helfen zu verstehen, wie Kubernetes Deployments und Pods verwaltet. Das describe-Kommando ist besonders nützlich, wenn etwas nicht funktioniert, da es Fehler und Ereignisse anzeigt.

### 3.3 Service erstellen

Nachdem unser Deployment läuft, müssen wir einen Zugriffspunkt dafür schaffen. Hier kommen Kubernetes Services ins Spiel. Ein entscheidendes Konzept in Kubernetes ist, dass Pods (die Container-Instanzen) vergänglich sind – sie können jederzeit neu erstellt werden und erhalten dabei neue IP-Adressen.

Ein Service fungiert wie eine stabile Fassade vor diesen sich ändernden Pods. Er bietet einen konstanten Endpunkt und kümmert sich um die Weiterleitung des Traffics an die aktuell verfügbaren Pods. Dies ist vergleichbar mit einer Empfangsstelle, die weiß, welche Mitarbeiter gerade verfügbar sind und Anfragen entsprechend weiterleitet – unabhängig davon, ob einzelne Mitarbeiter ihren Arbeitsplatz wechseln oder neue hinzukommen.

**Erstelle eine Datei `helloworld-service.yaml` mit folgendem Inhalt:**

```yaml
apiVersion: v1
kind: Service
metadata:
  name: helloworld-service
spec:
  selector:
    app: helloworld
  ports:
  - port: 80
    targetPort: 8080
  type: NodePort
```

**Was bedeutet dieser YAML-Code?**

- `apiVersion` und `kind`: Wir erstellen einen Service (API-Version v1)
- `metadata`: Name des Services
- `spec`: Die Spezifikation des Services:
  - `selector`: Der Service leitet Traffic an Pods mit dem Label `app: helloworld` weiter
  - `ports`: Traffic auf Port 80 des Services wird an Port 8080 der Pods weitergeleitet
  - `type: NodePort`: Der Service ist über einen Port auf dem Node erreichbar (notwendig für den Zugriff von außerhalb des Clusters)

**Wende den Service an:**

```bash
kubectl apply -f helloworld-service.yaml
```

**Was passiert hier?**
- Kubernetes erstellt einen Service mit dem Namen "helloworld-service"
- Der Service erhält eine Cluster-IP
- Da wir den Typ NodePort gewählt haben, wird auch ein zufälliger Port im Bereich 30000-32767 auf allen Nodes geöffnet
- Kubernetes erstellt automatisch Endpunkte, die auf die ausgewählten Pods verweisen

### 3.4 Überprüfe den Service und greife auf die Anwendung zu

```bash
# Überprüfe den Service
kubectl get services

# Hole dir den URL für den Zugriff auf den Service
minikube service helloworld-service --url

# Oder öffne direkt den Service im Browser
minikube service helloworld-service
```

**Was du sehen solltest:**
- `kubectl get services` zeigt deinen "helloworld-service" mit seiner Cluster-IP und dem NodePort
- Der letzte Befehl öffnet einen Browser, in dem du die Hello-Kubernetes-Anwendung mit deiner benutzerdefinierten Nachricht siehst

**Warum haben wir einen Service erstellt?**
- Services bieten eine stabile Netzwerkadresse für Pods
- Sie implementieren automatisch Load Balancing zwischen mehreren Pods
- Sie ermöglichen die Kommunikation zwischen verschiedenen Teilen der Anwendung
- Sie machen die Anwendung von außerhalb des Clusters erreichbar (mit bestimmten Service-Typen wie NodePort)

## Teil 4: Kubernetes-Features kennenlernen

### 4.1 Skalierung: Mehr Replicas erstellen

Einer der wesentlichen Vorteile von Kubernetes ist die einfache Skalierbarkeit. Mit einem einzigen Befehl kannst du die Anzahl der laufenden Instanzen deiner Anwendung erhöhen oder verringern – ohne Ausfallzeiten oder komplexe Umkonfigurationen.

Stell dir vor, du betreibst einen Kundenservice und musst auf erhöhtes Anfrageaufkommen reagieren. Anstatt manuell neue Server einzurichten, neue Software zu installieren und die Last zu verteilen, passt du einfach eine Zahl an – und Kubernetes erledigt den Rest. Es erstellt automatisch neue Instances, verteilt sie auf verfügbare Ressourcen und aktualisiert den Lastausgleich.

```bash
# Skaliere das Deployment auf 3 Replicas
kubectl scale deployment helloworld --replicas=3

# Überprüfe die Pods
kubectl get pods

# Überprüfe das Deployment
kubectl get deployment helloworld
```

**Was passiert hier?**
- Kubernetes erstellt zwei zusätzliche Pods basierend auf der gleichen Pod-Vorlage
- Der ReplicaSet-Controller überwacht ständig, ob die aktuelle Anzahl der Pods der gewünschten Anzahl entspricht
- Der Service leitet automatisch Anfragen an alle drei Pods weiter (Load Balancing)

**Warum ist Skalierung wichtig?**
- **Hochverfügbarkeit**: Wenn ein Pod ausfällt, können andere die Anfragen weiterhin bedienen
- **Lastverteilung**: Mehr Replicas können mehr Traffic bewältigen
- **Zero-Downtime Updates**: Mit mehreren Replicas können Updates schrittweise erfolgen

### 4.2 Selbstheilung testen

Eine der beeindruckendsten Eigenschaften von Kubernetes ist die automatische Selbstheilung. Kubernetes überwacht kontinuierlich den Zustand deiner Anwendungen und reagiert automatisch auf Ausfälle, indem es neue Instanzen startet, wenn bestehende nicht mehr funktionieren.

Dies ist vergleichbar mit einem Qualitätssicherungssystem in einer Produktion: Wenn ein Gerät ausfällt oder Fehlfunktionen zeigt, wird automatisch ein Ersatzgerät aktiviert, ohne dass der Produktionsprozess unterbrochen wird. In traditionellen Umgebungen würde ein Serverausfall oft manuelle Eingriffe erfordern – mit Kubernetes geschieht die Wiederherstellung automatisch und transparent.

```bash
# Liste alle Pods auf
kubectl get pods

# Lösche einen Pod (ersetze <pod-name> mit einem deiner Pod-Namen)
kubectl delete pod <pod-name>

# Beobachte, wie ein neuer Pod erstellt wird
kubectl get pods
```

**Was passiert hier?**
- Du löschst einen Pod manuell
- Das Deployment bemerkt, dass die aktuelle Anzahl von Pods (2) nicht der gewünschten Anzahl (3) entspricht
- Es erstellt automatisch einen neuen Pod, um wieder 3 Replicas zu haben
- Der neue Pod erhält einen neuen Namen, aber die gleiche Konfiguration

**Warum ist Selbstheilung wichtig?**
- **Zuverlässigkeit**: Kubernetes stellt sicher, dass deine Anwendung immer läuft
- **Reduzierter Betriebsaufwand**: Du musst ausgefallene Container nicht manuell neu starten
- **Robustheit**: Deine Anwendung kann Hardware- oder Softwarefehler überstehen

### 4.3 Rolling Updates durchführen

Ein weiteres Schlüsselmerkmal von Kubernetes sind Rolling Updates – die Fähigkeit, Anwendungen zu aktualisieren, ohne Ausfallzeiten zu verursachen. Bei herkömmlichen Deployments führt eine Aktualisierung oft zu einer kurzen Downtime, während der Service neu gestartet wird.

Kubernetes löst dieses Problem mit einem graduellen Ansatz: Es startet einige neue Pods mit der aktualisierten Version, wartet bis diese bereit sind, leitet dann Traffic zu ihnen um und entfernt schließlich die alten Pods. Dieser Prozess ist vergleichbar mit dem Austausch von Komponenten in einem laufenden System – wie wenn man einzelne Teile einer Brücke erneuert, ohne den Verkehr zu unterbrechen. Das Ergebnis: Deine Nutzer bemerken die Aktualisierung nicht einmal.

**Aktualisiere die `helloworld.yaml` Datei und ändere die Nachricht:**

```yaml
env:
- name: MESSAGE
  value: "Hallo von Version 2.0 meines Kubernetes-Deployments!"
```

**Wende das Update an:**

```bash
kubectl apply -f helloworld.yaml

# Beobachte den Rollout-Prozess
kubectl rollout status deployment/helloworld

# Schau dir die Pods an
kubectl get pods
```

**Was passiert hier?**
- Kubernetes erkennt die Änderung und startet ein Rolling Update
- Es erstellt einen neuen Pod mit der aktualisierten Konfiguration
- Nachdem der neue Pod bereit ist, beendet es einen alten Pod
- Dieser Prozess wird fortgesetzt, bis alle Pods aktualisiert sind
- Der Service leitet während des gesamten Vorgangs weiterhin Traffic an die verfügbaren Pods

**Warum sind Rolling Updates wichtig?**
- **Keine Ausfallzeiten**: Deine Anwendung bleibt während des Updates verfügbar
- **Schrittweise Einführung**: Neue Versionen werden schrittweise eingeführt
- **Automatisches Rollback**: Bei Problemen kann Kubernetes automatisch zur vorherigen Version zurückkehren

### 4.4 Rollback durchführen

Wenn ein Update Probleme verursacht, kannst du schnell zur vorherigen Version zurückkehren.

```bash
# Zeige den Rollout-Verlauf an
kubectl rollout history deployment/helloworld

# Führe ein Rollback zur vorherigen Version durch
kubectl rollout undo deployment/helloworld

# Überprüfe den Status
kubectl rollout status deployment/helloworld

# Überprüfe die Pods
kubectl get pods
```

**Was passiert hier?**
- Kubernetes startet einen ähnlichen Prozess wie bei einem Update, aber in umgekehrter Richtung
- Es erstellt neue Pods mit der vorherigen Konfiguration
- Es entfernt die Pods mit der aktuellen Konfiguration
- Der Service leitet Traffic während des gesamten Vorgangs weiter

**Warum ist Rollback wichtig?**
- **Schnelle Fehlerkorrektur**: Bei Problemen kannst du schnell zurück zur funktionierenden Version
- **Risikominimierung**: Du kannst neue Versionen mit weniger Risiko einführen
- **Zuverlässigkeit**: Deine Anwendung bleibt während des Rollbacks verfügbar

## Teil 5: Debugging und Interaktion mit Containern

### 5.1 Pod-Logs anzeigen

Logs sind entscheidend für das Debugging und das Verständnis, was in deinen Containern passiert.

```bash
# Zeige Logs eines Pods an (ersetze <pod-name>)
kubectl logs <pod-name>

# Zeige Logs kontinuierlich an (wie tail -f)
kubectl logs -f <pod-name>
```

**Was passiert hier?**
- kubectl holt die Logs direkt vom Container in dem Pod
- Diese Logs enthalten alles, was die Anwendung auf stdout oder stderr ausgibt
- Mit `-f` kannst du neue Logeinträge in Echtzeit sehen

**Warum sind Logs wichtig?**
- **Fehlersuche**: Logs zeigen dir, was in deiner Anwendung passiert
- **Überwachung**: Du kannst das Verhalten deiner Anwendung beobachten
- **Audit**: Logs sind wichtig für Sicherheits- und Compliance-Anforderungen

### 5.2 In einen Container einsteigen

Manchmal möchtest du direkt mit dem Container interagieren, um Probleme zu diagnostizieren oder Tests durchzuführen.

```bash
# Starte eine interaktive Shell im Container (ersetze <pod-name>)
kubectl exec -it <pod-name> -- /bin/sh

# Im Container kannst du nun Befehle ausführen
ls
env
ps
exit  # Zum Verlassen
```

**Was passiert hier?**
- kubectl öffnet eine Shell-Verbindung zum Container
- Du kannst Befehle im Container ausführen, als ob du direkt im Container wärst
- Dies ist nützlich für Debugging und Diagnose

**Warum ist der direkte Zugriff wichtig?**
- **Tiefes Debugging**: Du kannst die Umgebung des Containers untersuchen
- **Fehlersuche**: Du kannst Tests direkt im Container durchführen
- **Konfigurationsüberprüfung**: Du kannst sehen, ob Dateien und Umgebungsvariablen korrekt sind

### 5.3 Ressourcennutzung überwachen

```bash
# Aktiviere das Metrics-Server-Addon in Minikube
minikube addons enable metrics-server

# Warte einen Moment, bis Metriken gesammelt werden (ca. 1 Minute)

# Zeige die Ressourcennutzung der Pods an
kubectl top pods

# Zeige die Ressourcennutzung der Nodes an
kubectl top nodes
```

**Was passiert hier?**
- Der Metrics-Server sammelt CPU- und Speichernutzungsdaten von Pods und Nodes
- `kubectl top` zeigt diese Metriken an
- Du kannst sehen, wie viel CPU und Speicher deine Anwendung verbraucht

**Warum ist Monitoring wichtig?**
- **Ressourcenplanung**: Du kannst sehen, wie viel Ressourcen deine Anwendung benötigt
- **Performance-Optimierung**: Du kannst erkennen, wenn deine Anwendung zu viele Ressourcen verbraucht
- **Kapazitätsplanung**: Du kannst entscheiden, wann du skalieren musst

## Teil 6: Kubernetes Dashboard verwenden (optional)

Das Kubernetes Dashboard bietet eine grafische Oberfläche für die Verwaltung deines Clusters.

```bash
# Starte das Dashboard
minikube dashboard
```

**Was passiert hier?**
- Minikube aktiviert das Dashboard-Addon (falls noch nicht aktiviert)
- Es öffnet einen Browser mit dem Dashboard
- Du kannst nun grafisch mit deinem Cluster interagieren

**Was kannst du im Dashboard tun?**
- Deployments, Pods, Services und andere Ressourcen anzeigen und verwalten
- Ressourcen erstellen, bearbeiten oder löschen
- Logs anzeigen und in Container einsteigen
- Ressourcennutzung überwachen

**Warum ist das Dashboard nützlich?**
- **Einfachere Verwaltung**: Grafische Oberfläche statt Kommandozeile
- **Überblick**: Du siehst alle Ressourcen und ihren Status auf einen Blick
- **Schnelles Debugging**: Einfacher Zugriff auf Logs und Events

## Teil 7: Aufräumen

Nach Abschluss der Übung solltest du die erstellten Ressourcen löschen, um Speicherplatz zu sparen.

```bash
# Lösche den Service
kubectl delete service helloworld-service

# Lösche das Deployment
kubectl delete deployment helloworld

# Überprüfe, ob alles gelöscht wurde
kubectl get all

# Stoppe den Minikube-Cluster
minikube stop

# Wenn du den Cluster komplett löschen möchtest
# minikube delete
```

**Was passiert hier?**
- kubectl löscht die angegebenen Ressourcen aus dem Cluster
- `minikube stop` stoppt die virtuelle Maschine, behält aber den Cluster-Zustand
- `minikube delete` würde den Cluster vollständig löschen

**Warum ist Aufräumen wichtig?**
- **Ressourcenschonung**: Nicht benötigte Ressourcen verbrauchen Speicher und CPU
- **Klarheit**: Ein aufgeräumter Cluster ist einfacher zu verwalten
- **Vermeidung von Konflikten**: Alte Ressourcen können mit neuen in Konflikt geraten

## Zusammenfassung und Ausblick

Gratulation! Du hast soeben die grundlegenden Konzepte von Kubernetes in der Praxis kennengelernt. Lass uns zusammenfassen, was du erreicht hast:

1. **Einrichtung der Infrastruktur**: Installation von Docker, kubectl und Minikube
2. **Cluster-Erstellung**: Aufsetzen eines funktionierenden Kubernetes-Clusters
3. **Anwendungsbereitstellung**: Erstellung eines Deployments und eines Services
4. **Anwendungsmanagement**: Anwendung der Schlüsselkonzepte Kubernetes:
   - Skalierung durch Erhöhung der Replica-Anzahl
   - Erleben der automatischen Selbstheilung
   - Durchführung von Rolling Updates
   - Implementierung eines Rollbacks
5. **Debugging und Monitoring**: Zugriff auf Logs und Container, Überwachung der Ressourcennutzung
6. **Benutzeroberfläche**: Nutzung des Kubernetes Dashboards (optional)
7. **Ressourcenmanagement**: Ordnungsgemäßes Aufräumen und Freigeben von Ressourcen

## Nächste Schritte in deiner Kubernetes-Reise

Nach diesen Grundlagen gibt es viele Möglichkeiten, dein Wissen zu vertiefen:

1. **Komplexere Anwendungsarchitekturen**: Erstelle Deployments mit mehreren interagierenden Services
2. **Konfigurationsmanagement**: Implementiere ConfigMaps und Secrets für eine saubere Konfigurationsverwaltung
3. **Datenpersistenz**: Arbeite mit PersistentVolumes für zustandsbehaftete Anwendungen
4. **Netzwerkmanagement**: Richte Ingress-Controller für fortgeschrittenes HTTP-Routing ein
5. **Paketmanagement**: Lerne Helm für die effiziente Verwaltung komplexer Anwendungen
6. **Ressourcenoptimierung**: Implementiere präzise Ressourcenlimits und -anforderungen
7. **Automatische Skalierung**: Konfiguriere Auto-Scaling basierend auf Metriken wie CPU- oder Speicherauslastung

Diese erste praktische Erfahrung ist ein solides Fundament für deine weitere Arbeit mit Kubernetes – einem System, das die Art und Weise, wie wir Anwendungen in der Cloud bereitstellen und verwalten, grundlegend verändert hat.
