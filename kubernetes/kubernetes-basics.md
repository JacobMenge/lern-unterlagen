# Kubernetes-Grundlagen: Das Wesentliche auf einen Blick

## Was ist Kubernetes?

Kubernetes (oft als "K8s" abgekürzt) ist eine Open-Source-Plattform zur Automatisierung, Skalierung und Verwaltung von Container-Anwendungen. Es wurde ursprünglich von Google entwickelt und wird heute von der Cloud Native Computing Foundation (CNCF) betreut.

Stell dir vor, du hast viele Container-Anwendungen, die auf verschiedenen Servern laufen müssen. Kubernetes ist wie ein intelligenter Dirigent, der dafür sorgt, dass alle Container zur richtigen Zeit am richtigen Ort laufen und miteinander kommunizieren können.

## Warum Kubernetes?

- **Skalierbarkeit**: Kubernetes kann deine Anwendungen automatisch hoch- oder runterskalieren, basierend auf der aktuellen Last. Das bedeutet, dass du bei Lastspitzen schnell reagieren kannst, ohne manuell eingreifen zu müssen.

- **Selbstheilung**: Wenn ein Container oder Node ausfällt, erkennt Kubernetes das automatisch und startet neue Instanzen. So bleibt deine Anwendung immer verfügbar.

- **Service Discovery**: Dienste bekommen automatisch interne DNS-Namen und können sich so gegenseitig finden, ohne dass du IP-Adressen fest kodieren musst.

- **Load Balancing**: Eingehender Traffic wird automatisch auf alle verfügbaren Pods verteilt, was für eine gleichmäßige Auslastung sorgt.

- **Automatisierte Rollouts/Rollbacks**: Aktualisiere deine Anwendungen schrittweise ohne Ausfallzeiten. Bei Problemen kannst du mit einem Befehl zur vorherigen Version zurückkehren.

- **Infrastructure as Code**: Definiere deinen gewünschten Zustand deklarativ in YAML-Dateien, und Kubernetes kümmert sich um den Rest.

## Die Kubernetes-Architektur

![Kubernetes-Architektur](https://d33wubrfki0l68.cloudfront.net/2475489eaf20163ec0f54ddc1d92aa8d4c87c96b/e7c81/images/docs/components-of-kubernetes.svg)

### Control Plane (Master-Komponenten)

- **API-Server**: Die zentrale Schnittstelle für alle Kubernetes-Befehle. Jede Interaktion mit dem Cluster läuft über den API-Server.

- **etcd**: Eine verteilte, konsistente Key-Value-Datenbank, die den gesamten Zustand des Clusters speichert. Sie ist das "Gedächtnis" von Kubernetes.

- **Scheduler**: Analysiert neu erstellte Pods und weist ihnen basierend auf Ressourcenanforderungen, Hardware/Software-Einschränkungen und anderen Faktoren einen geeigneten Node zu.

- **Controller Manager**: Enthält verschiedene Controller, die den Zustand des Clusters überwachen und bei Abweichungen vom gewünschten Zustand Maßnahmen ergreifen.

- **Cloud Controller Manager**: Interagiert mit der zugrunde liegenden Cloud-Infrastruktur (bei Cloud-Installationen).

### Node-Komponenten (Worker)

- **Kubelet**: Der primäre "Node-Agent", der auf jedem Node läuft. Es stellt sicher, dass Container in einem Pod laufen und gesund sind.

- **Kube-Proxy**: Verwaltet die Netzwerkkommunikation zwischen Pods innerhalb des Clusters und nach außen. Implementiert die Kubernetes Service-Abstraktion.

- **Container Runtime**: Software, die die Container ausführt (z.B. Docker, containerd, CRI-O). Sie ist verantwortlich für das Herunterladen von Images und das Ausführen von Containern.

## Die wichtigsten Kubernetes-Objekte

### Workload-Ressourcen

1. **Pod**:
   - Die kleinste deploybare Einheit in Kubernetes
   - Enthält einen oder mehrere Container, die sich Netzwerk- und Speicherressourcen teilen
   - Hat eine einzigartige IP-Adresse im Cluster
   - Beispiel:
     ```yaml
     apiVersion: v1
     kind: Pod
     metadata:
       name: nginx-pod
     spec:
       containers:
       - name: nginx
         image: nginx:latest
         ports:
         - containerPort: 80
     ```

2. **Deployment**:
   - Verwaltet die Erstellung und Aktualisierung von Pods
   - Ermöglicht deklarative Updates und Rollbacks
   - Sorgt für die gewünschte Anzahl von Replikaten
   - Beispiel:
     ```yaml
     apiVersion: apps/v1
     kind: Deployment
     metadata:
       name: nginx-deployment
     spec:
       replicas: 3
       selector:
         matchLabels:
           app: nginx
       template:
         metadata:
           labels:
             app: nginx
         spec:
           containers:
           - name: nginx
             image: nginx:latest
             ports:
             - containerPort: 80
     ```

3. **StatefulSet**:
   - Ähnlich wie Deployments, aber für zustandsbehaftete Anwendungen
   - Bietet stabile Netzwerkidentitäten und persistenten Speicher
   - Ideal für Datenbanken und andere zustandsbehaftete Dienste

4. **DaemonSet**:
   - Stellt sicher, dass auf jedem Node (oder einer Teilmenge) ein bestimmter Pod läuft
   - Nützlich für Cluster-weite Dienste wie Monitoring oder Logging

### Service & Networking

5. **Service**:
   - Bietet eine stabile IP-Adresse und Port für den Zugriff auf eine Gruppe von Pods
   - Typen: ClusterIP (intern), NodePort (extern über Node-Ports), LoadBalancer (extern über Cloud-Provider)
   - Beispiel:
     ```yaml
     apiVersion: v1
     kind: Service
     metadata:
       name: nginx-service
     spec:
       selector:
         app: nginx
       ports:
       - port: 80
         targetPort: 80
       type: ClusterIP
     ```

6. **Ingress**:
   - Verwaltet externen Zugriff auf Dienste im Cluster, typischerweise HTTP/HTTPS
   - Bietet URL-Routing, SSL-Terminierung und mehr
   - Benötigt einen Ingress-Controller (z.B. Nginx, Traefik)

### Konfiguration & Speicher

7. **ConfigMap**:
   - Speichert nicht-sensible Konfigurationsdaten als Key-Value-Paare
   - Kann als Umgebungsvariablen oder Dateien in Pods eingebunden werden

8. **Secret**:
   - Ähnlich wie ConfigMaps, aber für sensible Informationen (Passwörter, Tokens)
   - Base64-kodiert gespeichert (aber nicht verschlüsselt, sofern nicht zusätzlich konfiguriert)

9. **PersistentVolume (PV) & PersistentVolumeClaim (PVC)**:
   - PV: Repräsentiert ein Stück physischen Speichers im Cluster
   - PVC: Anforderung eines Pods für Speicherplatz
   - Abstrahiert die zugrunde liegende Speichertechnologie

### Weitere wichtige Objekte

10. **Namespace**:
    - Virtuelle Cluster innerhalb eines physischen Clusters
    - Hilft bei der Organisation und Isolierung von Ressourcen

11. **HorizontalPodAutoscaler**:
    - Skaliert automatisch die Anzahl der Pods basierend auf CPU-Auslastung oder benutzerdefinierten Metriken

12. **NetworkPolicy**:
    - Definiert Regeln für die Kommunikation zwischen Pods
    - Implementiert Netzwerksicherheit auf Pod-Ebene

## Einfacher Workflow zur Bereitstellung einer Anwendung

1. **Anwendung containerisieren**:
   - Erstelle ein Dockerfile, das deine Anwendung beschreibt
   - Baue ein Docker-Image und pushe es in eine Registry
   ```bash
   docker build -t meine-app:v1 .
   docker push meine-registry/meine-app:v1
   ```

2. **Manifeste schreiben**:
   - Erstelle YAML-Dateien für deine Kubernetes-Objekte (Deployment, Service, etc.)
   - Organisiere sie in einer sinnvollen Verzeichnisstruktur

3. **Deployment ausrollen**:
   - Wende deine Manifeste auf den Cluster an
   ```bash
   kubectl apply -f deployment.yaml
   kubectl apply -f service.yaml
   # oder das ganze Verzeichnis
   kubectl apply -f ./kubernetes-manifests/
   ```

4. **Service erstellen und verfügbar machen**:
   - Expose deine Anwendung innerhalb oder außerhalb des Clusters
   ```bash
   kubectl expose deployment meine-app --type=LoadBalancer --port=80
   ```

5. **Überwachen und Skalieren**:
   - Beobachte die Performance deiner Anwendung
   - Skaliere manuell oder konfiguriere Auto-Scaling
   ```bash
   kubectl scale deployment/meine-app --replicas=5
   # oder
   kubectl apply -f horizontal-pod-autoscaler.yaml
   ```

## Häufig verwendete kubectl-Befehle

```bash
# Cluster-Überblick
kubectl get nodes                    # Zeigt alle Nodes im Cluster
kubectl get pods --all-namespaces    # Zeigt alle Pods in allen Namespaces
kubectl get pods -o wide             # Detaillierte Pod-Informationen inkl. IP und Node

# Deployment-Management
kubectl apply -f deployment.yaml     # Deployment erstellen/aktualisieren
kubectl delete -f deployment.yaml    # Deployment löschen
kubectl rollout status deployment/app-name  # Deployment-Status überwachen
kubectl rollout undo deployment/app-name    # Rollback zum vorherigen Zustand

# Debugging
kubectl logs pod-name                # Logs eines Pods anzeigen
kubectl logs -f pod-name             # Logs kontinuierlich anzeigen (follow)
kubectl describe pod pod-name        # Detaillierte Informationen zu einem Pod
kubectl exec -it pod-name -- /bin/bash  # In einen Pod einsteigen

# Skalierung
kubectl scale deployment/app-name --replicas=5  # Manuell skalieren

# Kontextinformationen
kubectl config get-contexts         # Verfügbare Kontexte anzeigen
kubectl config use-context my-context  # Kontext wechseln
```

## Kubernetes-Ökosystem: Wichtige Tools und Erweiterungen

### Paketmanagement & Deployment

- **Helm**: 
  - Der "Paketmanager" für Kubernetes
  - Erlaubt die Installation komplexer Anwendungen mit einem Befehl
  - Unterstützt Versionierung und Rollbacks
  - Beispiel: `helm install prometheus prometheus-community/prometheus`

- **Kustomize**:
  - Werkzeug zur Anpassung von Kubernetes-Manifesten ohne Templates
  - Integriert in kubectl: `kubectl apply -k ./`
  - Ideal für umgebungsspezifische Konfigurationen

- **ArgoCD**:
  - GitOps-Tool für kontinuierliche Bereitstellung
  - Synchronisiert Kubernetes-Cluster automatisch mit Git-Repositories
  - Bietet Visualisierung und Audit-Trails

### Monitoring & Logging

- **Prometheus & Grafana**:
  - Prometheus sammelt Metriken aus deinem Cluster
  - Grafana visualisiert diese Daten in anpassbaren Dashboards
  - De-facto-Standard für Kubernetes-Monitoring

- **ELK Stack / Loki**:
  - Sammelt und analysiert Logs aus deinem Cluster
  - Ermöglicht Suche und Visualisierung von Logdaten
  - Hilft bei der Fehlersuche und Performance-Analyse

### Service Mesh & Netzwerk

- **Istio**:
  - Vollständiges Service Mesh für komplexe Microservices-Architekturen
  - Bietet Traffic-Management, Sicherheit und Observability
  - Unterstützt fortgeschrittene Deployment-Strategien wie Canary-Releases

- **Cilium**:
  - Netzwerk-Plugin basierend auf eBPF
  - Bietet erweiterte Sicherheitsfunktionen und Performance
  - Unterstützt detaillierte Netzwerkrichtlinien

### Entwicklertools

- **Minikube / kind**:
  - Lokale Kubernetes-Cluster für die Entwicklung
  - Ermöglicht das Testen von Kubernetes-Anwendungen ohne Cloud-Ressourcen

- **Lens / K9s**:
  - Grafische Oberflächen für die Verwaltung von Kubernetes-Clustern
  - Vereinfachen die tägliche Arbeit mit Kubernetes

## Aktuelle Trends (2025)

- **GitOps**: 
  - Infrastruktur als Code mit automatischer Synchronisation vom Git-Repository
  - Tools wie Flux und ArgoCD gewinnen an Bedeutung
  - Verbessert Nachvollziehbarkeit und Reproduzierbarkeit

- **eBPF in Kubernetes**:
  - Erweiterte Netzwerk- und Sicherheitsfunktionen auf Kernel-Ebene
  - Verbesserte Performance und Sicherheit ohne Sidecar-Container
  - Tools wie Cilium setzen stark auf eBPF

- **WebAssembly (Wasm) in Kubernetes**:
  - Leichtgewichtige Alternative zu Containern
  - Schnellerer Start und geringerer Ressourcenverbrauch
  - Wachsendes Ökosystem von Tools und Projekten

- **Serverless Kubernetes**:
  - Kubernetes ohne die Verwaltung der Infrastruktur
  - Dienste wie AWS EKS Fargate, Google Cloud Run for Anthos
  - Fokus auf Anwendungen statt auf Infrastruktur

- **Platform Engineering**:
  - Interne Entwicklerplattformen basierend auf Kubernetes
  - Abstraktion der Kubernetes-Komplexität für Entwickler
  - Tools wie Backstage von Spotify gewinnen an Bedeutung

- **Multi-Cloud und Hybrid-Cloud Kubernetes**:
  - Einheitliche Verwaltung von Workloads über mehrere Clouds hinweg
  - Tools wie Anthos, Rancher und Crossplane

## Häufige Herausforderungen und Best Practices

### Ressourcenkontrolle

- **Immer Limits und Requests setzen**:
  ```yaml
  resources:
    requests:
      memory: "64Mi"
      cpu: "250m"
    limits:
      memory: "128Mi"
      cpu: "500m"
  ```
- **Ressourcen-Quotas für Namespaces definieren**, um Überverbrauch zu verhindern
- **Horizontal Pod Autoscaler** für automatische Skalierung nutzen

### Persistenter Speicher

- **StatefulSets für zustandsbehaftete Anwendungen** wie Datenbanken verwenden
- **Storage Classes** für verschiedene Anforderungen definieren (SSD vs. HDD)
- **Backups und Disaster Recovery** für persistente Daten planen

### Netzwerksicherheit

- **Network Policies implementieren** für Zero-Trust-Sicherheit:
  ```yaml
  apiVersion: networking.k8s.io/v1
  kind: NetworkPolicy
  metadata:
    name: default-deny
  spec:
    podSelector: {}
    policyTypes:
    - Ingress
  ```
- **Service Mesh** für komplexere Sicherheitsanforderungen nutzen
- **Ingress mit TLS** für sichere externe Verbindungen konfigurieren

### Komplexitätsmanagement

- **Helm oder Kustomize** zur Vereinfachung des Deployments nutzen
- **Operators** für komplexe Anwendungen mit spezifischen Betriebsanforderungen
- **Namespaces und Labels** für logische Organisation des Clusters

### Performance-Optimierung

- **Pod Affinity/Anti-Affinity** für optimale Platzierung
- **Topology Spread Constraints** für gleichmäßige Verteilung der Last
- **Node Taints und Tolerations** für spezielle Workloads

## Erste Schritte mit Kubernetes

1. **Lokale Entwicklungsumgebung einrichten**:
   - Installiere Minikube oder kind für einen lokalen Cluster
   - Installiere kubectl als Kommandozeilen-Tool
   - Optional: Installiere eine GUI wie Lens

2. **Erste Anwendung deployen**:
   - Beginne mit einem einfachen Deployment und Service
   - Experimentiere mit Skalierung und Updates
   - Lerne, Logs und Events zu untersuchen

3. **Fehlerdiagnose lernen**:
   - Nutze `kubectl describe` und `kubectl logs` für Debugging
   - Verstehe die häufigsten Pod-Statuses und ihre Bedeutung
   - Lerne, Ressourcenprobleme zu erkennen

4. **Fortgeschrittene Konzepte erkunden**:
   - Nutze Helm-Charts für komplexere Anwendungen
   - Implementiere Monitoring mit Prometheus und Grafana
   - Lerne über Continuous Deployment mit GitOps

---

## FAQ zu Kubernetes: Einfach erklärt für Anfänger

### Was ist der Unterschied zwischen Docker und Kubernetes?

**Docker** ist wie ein Paket für deine Anwendung. Es verpackt alles, was deine App zum Laufen braucht (Code, Bibliotheken, Einstellungen) in einen Container, sodass sie überall gleich läuft. Mit Docker kannst du einzelne Container auf einem Computer starten und stoppen.

**Kubernetes** ist wie ein Orchester-Dirigent für viele Docker-Container. Wenn du Dutzende oder Hunderte Container verwalten musst, hilft Kubernetes dabei:
- Sie auf verschiedenen Computern zu verteilen
- Sicherzustellen, dass immer genug Container laufen
- Container neu zu starten, wenn sie abstürzen
- Den Netzwerkverkehr zwischen ihnen zu verwalten

Stelle dir vor: Docker ist wie ein einzelner Musiker, der ein Instrument spielt, und Kubernetes ist der Dirigent, der dafür sorgt, dass alle Musiker zusammen ein Konzert geben.

### Was bedeutet eigentlich "Container"?

Ein **Container** ist wie ein leichtes, tragbares Zimmer für deine Anwendung. In diesem Zimmer ist alles enthalten, was die Anwendung braucht:
- Der Programmcode
- Alle Bibliotheken und Abhängigkeiten
- Konfigurationen und Einstellungen

Das Besondere: Container sind isoliert vom Rest des Systems. Das bedeutet:
- Sie können andere Programme nicht stören
- Sie funktionieren immer gleich, egal auf welchem Computer sie laufen
- Sie starten sehr schnell (in Sekunden)

Im Unterschied zu einer virtuellen Maschine teilen sich Container den Betriebssystemkern und sind daher viel leichter und effizienter.

### Warum brauche ich überhaupt Kubernetes?

Stell dir vor, du hast eine beliebte Webseite mit vielen Besuchern:

**Ohne Kubernetes:**
- Du musst manuell neue Server starten, wenn mehr Besucher kommen
- Wenn ein Server abstürzt, fällt deine Webseite aus
- Updates deiner Anwendung verursachen Ausfallzeiten
- Du musst komplizierte Skripte schreiben, um alles zu verwalten

**Mit Kubernetes:**
- Bei mehr Besuchern skaliert die Anwendung automatisch
- Wenn ein Server ausfällt, werden die Container automatisch woanders neu gestartet
- Updates können schrittweise ohne Ausfallzeit eingespielt werden
- Es gibt eine einheitliche Art, all diese Aufgaben zu erledigen

Kubernetes ist besonders nützlich, wenn deine Anwendung aus vielen kleinen Teilen (Microservices) besteht, die zusammenarbeiten müssen.

### Wie erkläre ich Kubernetes meinem Chef, der kein Technik-Experte ist?

**Ohne Fachjargon:**
"Kubernetes ist ein Automatisierungs-Tool, das uns Zeit und Geld spart, indem es unsere Server effizienter nutzt und weniger manuelle Eingriffe erfordert.

Es bietet drei Hauptvorteile:
1. **Ausfallsicherheit**: Wenn ein Server abstürzt, werden die Anwendungen automatisch auf anderen Servern neu gestartet.
2. **Skalierbarkeit**: Bei hohem Besucheraufkommen werden automatisch mehr Ressourcen bereitgestellt; bei weniger Besuchern werden Ressourcen freigegeben, um Kosten zu sparen.
3. **Konsistenz**: Unsere Entwicklungs-, Test- und Produktionsumgebungen sind identisch aufgebaut, was Fehler reduziert und die Produkteinführung beschleunigt."

### Was ist ein Pod und warum nicht einfach Container?

Ein **Pod** ist wie eine kleine Wohngemeinschaft für Container:

- In einem Pod leben ein oder mehrere eng verwandte Container zusammen
- Sie teilen sich dieselbe Netzwerkadresse und können über "localhost" miteinander sprechen
- Sie teilen sich auch Speicherplatz

**Warum Pods?** Manchmal braucht eine Anwendung Helfer-Container:
- Ein Hauptcontainer, der deine Anwendung ausführt
- Ein Helfer-Container, der Logs sammelt
- Ein weiterer Helfer, der Metriken erfasst

Durch Pods können diese Container als eine Einheit verwaltet werden: Sie werden zusammen gestartet, gestoppt und auf denselben Computer gesetzt.

### Was bedeuten all diese Kubernetes-Begriffe für Anfänger?

**Node**: Ein Computer (physisch oder virtuell), der Teil deines Kubernetes-Clusters ist. Nodes sind die Arbeiter, die deine Container ausführen.

**Cluster**: Eine Gruppe von Nodes, die zusammenarbeiten und von Kubernetes verwaltet werden.

**Deployment**: Ein Bauplan, der Kubernetes sagt: "Halte X Kopien dieser Anwendung am Laufen". Wenn ein Pod abstürzt, erstellt das Deployment automatisch einen neuen.

**Service**: Eine stabile "Adresse", unter der deine Anwendung erreichbar ist. Services sorgen dafür, dass Anfragen an die richtigen Pods weitergeleitet werden, auch wenn diese ständig kommen und gehen.

**ConfigMap**: Ein Ort, an dem du Konfigurationseinstellungen für deine Anwendung speichern kannst, ohne sie im Container-Image fest einzubauen.

**Secret**: Ähnlich wie ConfigMap, aber für sensible Daten wie Passwörter und API-Schlüssel.

**Namespace**: Ein virtueller "Raum" im Cluster, der Ressourcen voneinander trennt. Wie verschiedene Projekte in separaten Ordnern.

### Wie starte ich eine Anwendung in Kubernetes für absolute Anfänger?

Hier ist eine vereinfachte Schritt-für-Schritt-Anleitung:

1. **Installiere die nötigen Tools:**
   - Docker (zum Erstellen von Container-Images)
   - kubectl (das Kommandozeilen-Tool für Kubernetes)
   - minikube (für einen lokalen Kubernetes-Cluster zum Üben)

2. **Starte deinen lokalen Cluster:**
   ```bash
   minikube start
   ```

3. **Erstelle eine einfache YAML-Datei** (nennen wir sie `meine-app.yaml`):
   ```yaml
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: meine-app
   spec:
     replicas: 2
     selector:
       matchLabels:
         app: meine-app
     template:
       metadata:
         labels:
           app: meine-app
       spec:
         containers:
         - name: meine-app
           image: nginx
           ports:
           - containerPort: 80
   ---
   apiVersion: v1
   kind: Service
   metadata:
     name: meine-app-service
   spec:
     selector:
       app: meine-app
     ports:
     - port: 80
     type: NodePort
   ```

4. **Wende die Datei auf deinen Cluster an:**
   ```bash
   kubectl apply -f meine-app.yaml
   ```

5. **Prüfe, ob alles läuft:**
   ```bash
   kubectl get pods
   kubectl get services
   ```

6. **Öffne deine Anwendung im Browser:**
   ```bash
   minikube service meine-app-service
   ```

Das war's! Du hast gerade zwei Kopien eines Nginx-Webservers gestartet und sie über einen Service zugänglich gemacht.

### Wie funktioniert die Selbstheilung in Kubernetes?

Die Selbstheilung in Kubernetes ist wie ein automatischer Hausmeister:

1. **Ständige Überwachung:** Kubernetes prüft regelmäßig, ob deine Pods und Container noch funktionieren.

2. **Problemerkennung:** Es erkennt verschiedene Probleme:
   - Wenn ein Container abstürzt
   - Wenn ein Pod nicht mehr auf Health-Checks antwortet
   - Wenn ein Node komplett ausfällt

3. **Automatische Reparatur:**
   - Bei Abstürzen werden Container automatisch neu gestartet
   - Nicht reagierende Pods werden gelöscht und durch neue ersetzt
   - Wenn ein Node ausfällt, werden alle seine Pods auf anderen Nodes neu erstellt

Du kannst diese Checks anpassen mit:
- **Liveness Probes**: "Lebt die Anwendung noch?"
- **Readiness Probes**: "Ist die Anwendung bereit, Traffic zu empfangen?"
- **Startup Probes**: "Ist die Anwendung fertig mit dem Starten?"

Ein einfaches Beispiel für einen Pod mit Liveness-Check:
```yaml
containers:
- name: meine-app
  image: meine-app:1.0
  livenessProbe:
    httpGet:
      path: /health
      port: 8080
    initialDelaySeconds: 15
    periodSeconds: 10
```

### Brauche ich Kubernetes für mein kleines Projekt?

Wahrscheinlich nicht! Kubernetes ist wie ein großer Gabelstapler - sehr nützlich für große Lasten, aber übertrieben für einen Einkaufsbeutel.

**Überlege, ob folgendes auf dich zutrifft:**
- Du hast nur eine oder wenige Anwendungen
- Du hast nicht viel Traffic (wenige Nutzer gleichzeitig)
- Kurze Ausfallzeiten sind verkraftbar
- Du hast kein Team, das sich um Infrastruktur kümmert

**In diesen Fällen sind einfachere Alternativen besser:**
- **Docker Compose**: Für mehrere Container auf einem einzelnen Server
- **Platform as a Service (PaaS)**: Wie Heroku, Render oder Railway
- **Serverless-Dienste**: Wie AWS Lambda oder Google Cloud Functions

Diese Optionen benötigen viel weniger Einrichtung und Wartung. Du kannst später immer noch zu Kubernetes wechseln, wenn dein Projekt wächst!

### Wie viel kostet Kubernetes?

Kubernetes selbst ist **kostenlos und Open Source**. Die Kosten entstehen durch:

1. **Infrastruktur**: Die Server (Nodes), auf denen Kubernetes läuft
   - Bei Cloud-Anbietern: ca. 50-200€ pro Node pro Monat, je nach Größe
   - Minimum für Produktion: Mindestens 3 Nodes für Ausfallsicherheit

2. **Verwaltete Kubernetes-Dienste** (optional, aber empfohlen):
   - AWS EKS: ~70€ pro Monat pro Cluster + Kosten für Nodes
   - Google GKE: Kostenlos für den Control Plane + Kosten für Nodes
   - Azure AKS: Kostenlos für den Control Plane + Kosten für Nodes

3. **Personalkosten**: Kubernetes-Expertise ist nicht billig
   - Entweder: Eigene Mitarbeiter schulen
   - Oder: DevOps-Experten mit Kubernetes-Erfahrung einstellen

Für ein typisches kleines Produktions-Setup mit verwalteten Diensten solltest du mit mindestens 250-500€ monatlich rechnen, nur für die Infrastruktur.

### Was ist der Unterschied zwischen Kubernetes, k8s, k3s und minikube?

- **Kubernetes**: Der vollständige Name der Plattform zur Container-Orchestrierung

- **k8s**: Eine Abkürzung für Kubernetes (das "8" steht für die 8 Buchstaben zwischen K und s)

- **minikube**: Eine kleine Version von Kubernetes, die auf einem einzelnen Computer läuft. Ideal zum Lernen und Entwickeln.

- **k3s**: Eine leichtgewichtige Kubernetes-Distribution von Rancher Labs. Braucht weniger Ressourcen als Standard-Kubernetes und ist gut für Edge-Computing, IoT oder kleinere Umgebungen geeignet.

### Was sind Ingress, LoadBalancer und NodePort?

Alle drei sind Methoden, um deine Anwendungen in Kubernetes von außen zugänglich zu machen:

**NodePort**:
- Einfachste Methode
- Öffnet einen bestimmten Port auf allen Nodes
- Erreichbar über `<Node-IP>:<NodePort>`
- Ports sind auf 30000-32767 beschränkt
- Eher für Entwicklung und Tests gedacht

**LoadBalancer**:
- Erstellt einen externen Load Balancer in der Cloud
- Bekommt eine eigene externe IP-Adresse
- Leitet Traffic an deine Dienste weiter
- Einfach zu nutzen, aber jeder Service braucht einen eigenen Load Balancer (kann teuer werden)

**Ingress**:
- Fungiert als intelligenter Router/Reverse Proxy
- Ein Ingress kann viele Services bedienen
- Ermöglicht URL-basiertes Routing (z.B. `/app1` → Service1, `/app2` → Service2)
- Unterstützt SSL/TLS-Terminierung
- Benötigt einen Ingress Controller (wie Nginx oder Traefik)

**Was wählen?**
- Für Entwicklung: NodePort
- Für einfache Produktions-Setups: LoadBalancer
- Für mehrere Services mit einer IP: Ingress

---

## FAQ zu Kubernetes

### Was ist der Unterschied zwischen Docker und Kubernetes?

**Docker** ist eine Technologie zum Erstellen und Ausführen einzelner Container. Es abstrahiert die Anwendung von der zugrunde liegenden Infrastruktur.

**Kubernetes** hingegen orchestriert viele Container. Es ist ein System zur Verwaltung von containerisierten Anwendungen über mehrere Hosts hinweg und bietet Funktionen wie Skalierung, Selbstheilung und Service Discovery.

Der einfachste Vergleich: Docker verwaltet einzelne Container, Kubernetes verwaltet viele Container über viele Maschinen hinweg.

### Wann sollte ich Kubernetes einsetzen und wann nicht?

**Setze Kubernetes ein, wenn du:**
- Mehrere Microservices verwaltest
- Hohe Verfügbarkeit benötigst
- Automatische Skalierung brauchst
- Eine konsistente Umgebung über Entwicklung, Test und Produktion haben möchtest
- DevOps-Praktiken implementieren willst

**Verzichte auf Kubernetes, wenn:**
- Du nur eine einfache Anwendung mit einem oder wenigen Containern hast
- Du limitierte Ressourcen oder Know-how hast (der Overhead kann beträchtlich sein)
- Du nur eine kleine Entwicklungs- oder Testumgebung benötigst (hier könnte Docker Compose ausreichen)
- Deine Anwendung nicht containerisiert werden kann oder sollte

### Was ist der Unterschied zwischen einem Pod und einem Container?

Ein **Container** ist eine isolierte, ausführbare Software-Einheit, die alles enthält, was zum Ausführen einer Anwendung erforderlich ist.

Ein **Pod** ist die kleinste deploybare Einheit in Kubernetes und kann einen oder mehrere Container enthalten. Diese Container teilen sich Netzwerk- und Speicherressourcen und können über localhost miteinander kommunizieren. Pods sind temporär und können jederzeit neu erstellt werden.

### Was ist ein Kubernetes Operator?

Ein **Operator** ist ein anwendungsspezifischer Controller, der das Kubernetes-API erweitert, um komplexe zustandsbehaftete Anwendungen zu verwalten. Operatoren automatisieren Aufgaben, die normalerweise manuell durchgeführt werden müssten, wie:
- Installation und Upgrades
- Backup und Wiederherstellung
- Skalierung
- Failover

Bekannte Beispiele sind der Prometheus Operator, MySQL Operator oder Elasticsearch Operator.

### Wie funktioniert Networking in Kubernetes?

In Kubernetes:
- Jeder **Pod** erhält eine eigene IP-Adresse
- Alle Pods können ohne NAT miteinander kommunizieren
- **Services** abstrahieren den Zugriff auf Pods mit einer stabilen IP oder DNS-Namen
- **Ingress** verwaltet externen HTTP/HTTPS-Zugriff
- **Network Policies** definieren Regeln für die Pod-Kommunikation (ähnlich wie Firewalls)

Kubernetes selbst definiert nur, was das Netzwerk leisten muss, die Implementierung erfolgt durch verschiedene CNI-Plugins (Container Network Interface) wie Calico, Flannel oder Cilium.

### Was sind die Vorteile eines Service Mesh wie Istio?

Ein Service Mesh bietet:
- **Erweiterte Traffic-Kontrolle**: Canary Deployments, A/B-Tests, Circuit Breaking
- **Gegenseitige TLS-Authentifizierung** zwischen Services
- **Detaillierte Einblicke** in den Service-zu-Service-Verkehr
- **Richtlinien** für Zugriffssteuerung und Ratenlimitierung
- **Verteiltes Tracing** für die Fehlerbehebung

Es funktioniert typischerweise über Sidecar-Container, die neben deinen Anwendungs-Containern laufen und den gesamten Netzwerkverkehr verwalten.

### Wie kann ich Kubernetes-Ressourcen kostengünstig nutzen?

- **Richtige Ressourcenbemessung**: Starts mit kleinen Requests und überwache die tatsächliche Nutzung
- **Cluster-Autoscaler** aktivieren, um Nodes bei Nichtgebrauch herunterzufahren
- **Spot-Instanzen/Preemptible VMs** für nicht-kritische Workloads nutzen
- **Namespace Resource Quotas** implementieren, um Überallokation zu vermeiden
- **Pod Priority & Preemption** für wichtige Workloads konfigurieren
- **Kostenüberwachungstools** wie Kubecost einsetzen

### Was ist der Unterschied zwischen StatefulSets und Deployments?

**Deployments** sind für zustandslose Anwendungen gedacht, bei denen Pods austauschbar sind. Sie bieten:
- Identische, austauschbare Pods
- Keine stabile Netzwerk-Identität
- Kein geordnetes Deployment oder Scaling

**StatefulSets** sind für zustandsbehaftete Anwendungen wie Datenbanken konzipiert und bieten:
- Stabile, eindeutige Netzwerknamen (pod-0, pod-1, ...)
- Stabile, persistente Speicher, der erhalten bleibt, wenn Pods neu erstellt werden
- Geordnete, schrittweise Deployments und Skalierung

### Wie funktioniert Hochverfügbarkeit in Kubernetes?

Kubernetes bietet Hochverfügbarkeit auf mehreren Ebenen:
- **Control Plane HA**: Mehrere API-Server, Scheduler, etc. in verschiedenen Zonen
- **Anwendungs-HA**: Mehrere Replikate von Pods, verteilt über Nodes und Zonen
- **Pod-HA**: Automatisches Neustart von fehlgeschlagenen Containern oder Pods
- **Cluster-HA**: Multi-Cluster-Setup für regionsübergreifende Ausfallsicherheit

Best Practices:
- Mindestens 3 Control Plane Nodes und 3 Worker Nodes verwenden
- PodDisruptionBudgets definieren, um Verfügbarkeit während Wartungsarbeiten zu garantieren
- Anti-Affinity Rules nutzen, um Pods auf verschiedene Nodes zu verteilen

---

*Denk daran: Kubernetes ist mächtig, aber nicht immer notwendig. Für einfache Anwendungen kann es Overhead bedeuten. Wähle die richtige Technologie für deine Anforderungen.*
