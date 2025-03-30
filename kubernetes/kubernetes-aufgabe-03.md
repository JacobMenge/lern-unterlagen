# Fortgeschrittene Kubernetes-Übung: Multi-Tier-Anwendung

In dieser Übung baust du eine komplette Multi-Tier-Anwendung mit Frontend, Backend und Datenbank in Kubernetes. Ich führe dich Schritt für Schritt durch die wichtigsten Konzepte und gebe dir praktische Tipps zur Fehlerbehebung.

## Warum eine Multi-Tier-Anwendung?

Die Multi-Tier-Architektur (auch mehrschichtige Architektur genannt) ist die Grundlage moderner Anwendungsentwicklung. Du teilst dabei eine Anwendung in funktional getrennte Schichten auf:

1. **Präsentationsschicht (Frontend)**: Kümmert sich um die Benutzeroberfläche 
2. **Logikschicht (Backend)**: Enthält die Geschäftslogik und verarbeitet Anfragen
3. **Datenschicht (Datenbank)**: Speichert und verwaltet deine Daten

Der große Vorteil? Jede Schicht kann unabhängig entwickelt, gewartet und skaliert werden!

Warum das cool ist:
- Du kannst bei Bedarf nur das Frontend skalieren, wenn viele Nutzer zugreifen
- Entwicklerteams können an verschiedenen Schichten arbeiten, ohne sich gegenseitig zu stören
- Wenn eine Komponente ausfällt, funktionieren die anderen oft weiter
- Du kannst für jede Schicht die beste Technologie wählen

Kubernetes ist perfekt für diese Architektur:
- Es gibt dir Deployments für zustandslose Dienste (Frontend, Backend)
- StatefulSets für Datenbanken
- Services für die Kommunikation zwischen den Schichten
- Ingress-Controller für den externen Zugriff

Diese Übung zeigt dir nicht nur Kubernetes-Konzepte, sondern wie du sie in einer echten Anwendung einsetzt - vom einfachen Webdienst bis hin zu komplexen Microservices. Das Wissen kannst du direkt für eigene Projekte nutzen.

---

## Inhaltsverzeichnis

1. [Vorbereitung](#vorbereitung)
2. [Datenbank-Tier (MongoDB mit StatefulSet)](#teil-1-datenbank-aufsetzen)
3. [Backend-Tier (Node.js/Express)](#teil-2-das-backend)
4. [Frontend-Tier (React)](#teil-3-das-frontend)
5. [Ingress und Routing](#teil-4-ingress-controller-und-routing)
6. [Monitoring mit Prometheus und Grafana](#teil-5-monitoring-mit-prometheus-und-grafana)
7. [Automatische Skalierung](#teil-6-horizontale-pod-autoskalierung)
8. [Zusammenfassung und nächste Schritte](#geschafft)

---

## Vorbereitung

**Was du brauchst:**

- Eine laufende Kubernetes-Umgebung (Minikube oder Kind)
- Grundlagen von Kubernetes (Pods, Deployments, Services)
- Helm (der Kubernetes-Paketmanager)

> **Tipp:** Nutze die Kubernetes-Doku und die Materialien im Classroom. Die meisten Fragen kannst du damit selbst lösen.

### Hilfreiche Links:

| Thema | Link |
|-------|------|
| Kubernetes Konzepte | [kubernetes.io/de/docs/concepts](https://kubernetes.io/de/docs/concepts/) |
| StatefulSets | [kubernetes.io/docs/concepts/workloads/controllers/statefulset](https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/) |
| Persistent Volumes | [kubernetes.io/docs/concepts/storage/persistent-volumes](https://kubernetes.io/docs/concepts/storage/persistent-volumes/) |
| Services & Networking | [kubernetes.io/docs/concepts/services-networking](https://kubernetes.io/docs/concepts/services-networking/) |
| Ingress | [kubernetes.io/docs/concepts/services-networking/ingress](https://kubernetes.io/docs/concepts/services-networking/ingress/) |
| Horizontal Pod Autoscaling | [kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale/) |

---

## Teil 1: Datenbank aufsetzen

**Was wir erreichen wollen:** Eine MongoDB-Datenbank, deren Daten auch nach Neustarts erhalten bleiben

### 1.1 Namespace erstellen

```bash
kubectl create namespace meine-fullstack-app
```

**Was passiert hier?** 
Namespaces sind wie virtuelle Cluster innerhalb deines Kubernetes-Clusters. Sie helfen dir, deine Sachen zu sortieren und zu isolieren. Es ist wie verschiedene Projekte in verschiedenen Ordnern zu haben.

### 1.2 PersistentVolume erstellen

Speichere als `mongo-pv.yaml`:

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: mongo-pv
spec:
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  storageClassName: standard
  hostPath:
    path: /data/mongo
```

**Was passiert hier?** 
Ein PersistentVolume ist wie eine Festplatte für deine Anwendungen. Die Daten bleiben erhalten, auch wenn Pods neu starten. Wir erstellen hier einen 1GB großen Speicherplatz für MongoDB.

**Für Minikube-Nutzer:** 
Du musst eventuell das Verzeichnis `/data/mongo` auf dem Host erstellen:

```bash
minikube ssh "sudo mkdir -p /data/mongo && sudo chmod 777 /data/mongo"
```

Jetzt das PersistentVolume anlegen:

```bash
kubectl apply -f mongo-pv.yaml
```

**Kurze Kontrolle:**
```bash
kubectl get pv
```
Status sollte `Available` sein.

### 1.3 Secret für MongoDB-Zugangsdaten anlegen

```bash
kubectl create secret generic mongo-secret \
  --namespace meine-fullstack-app \
  --from-literal=username=admin \
  --from-literal=password=geheimespasswort
```

**Was passiert hier?** 
Secrets speichern sensible Daten wie Passwörter. In einer echten Umgebung würdest du natürlich stärkere Passwörter nutzen!

### 1.4 StatefulSet für MongoDB erstellen

Speichere als `mongo-statefulset.yaml`:

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: mongo
  namespace: meine-fullstack-app
spec:
  serviceName: "mongo"
  replicas: 1
  selector:
    matchLabels:
      app: mongo
  template:
    metadata:
      labels:
        app: mongo
    spec:
      containers:
      - name: mongo
        image: mongo:4.4
        ports:
        - containerPort: 27017
        volumeMounts:
        - name: mongo-data
          mountPath: /data/db
        env:
        - name: MONGO_INITDB_ROOT_USERNAME
          valueFrom:
            secretKeyRef:
              name: mongo-secret
              key: username
        - name: MONGO_INITDB_ROOT_PASSWORD
          valueFrom:
            secretKeyRef:
              name: mongo-secret
              key: password
  volumeClaimTemplates:
  - metadata:
      name: mongo-data
    spec:
      accessModes: [ "ReadWriteOnce" ]
      storageClassName: "standard"
      resources:
        requests:
          storage: 1Gi
```

**Was passiert hier?** 
Ein StatefulSet ist perfekt für Datenbanken. Anders als normale Deployments sorgt es für:
- Feste Hostnames
- Immer gleiche Speicherzuordnung
- Geordnete Updates

Das `volumeClaimTemplates` sorgt dafür, dass automatisch ein PersistentVolumeClaim erstellt wird.

**Häufiger Fehler:** 
Stelle sicher, dass du das Secret erstellt hast, bevor du das StatefulSet anlegst!

Los geht's:

```bash
kubectl apply -f mongo-statefulset.yaml
```

**Wenn's nicht klappt:**
Schau nach dem Problem mit:

```bash
kubectl describe pod mongo-0 -n meine-fullstack-app
```

Typische Probleme:
- Secret-Name falsch
- PVC kann nicht gebunden werden
- Nicht genug Ressourcen

### 1.5 Service für MongoDB erstellen

Speichere als `mongo-service.yaml`:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: mongo
  namespace: meine-fullstack-app
  labels:
    app: mongo
spec:
  ports:
  - port: 27017
    targetPort: 27017
  clusterIP: None
  selector:
    app: mongo
```

**Was passiert hier?** 
Dies ist ein Headless-Service (`clusterIP: None`), perfekt für StatefulSets. Er erstellt DNS-Einträge, mit denen das Backend später auf `mongo:27017` zugreifen kann.

Service anlegen:

```bash
kubectl apply -f mongo-service.yaml
```

**Kurze Kontrolle:**
```bash
kubectl get svc -n meine-fullstack-app
```
Der `mongo`-Service sollte `None` als ClusterIP haben.

### 1.6 Überprüfen, ob alles läuft

```bash
kubectl get statefulsets -n meine-fullstack-app
kubectl get pods -n meine-fullstack-app
kubectl get pvc -n meine-fullstack-app
```

**Wichtig:** 
MongoDB braucht einen Moment zum Starten. Warte, bis der Pod-Status `Running` zeigt!

### 1.7 Mit der MongoDB verbinden (falls du möchtest)

```bash
kubectl exec -it mongo-0 -n meine-fullstack-app -- mongo -u admin -p geheimespasswort
```

**Ein paar Testbefehle:**
```
use testdb
db.testCollection.insertOne({ name: "test", value: 42 })
db.testCollection.find()
```

---

## Teil 2: Das Backend

**Ziel:** Ein Node.js-Backend erstellen, das mit der MongoDB kommuniziert

### 2.1 ConfigMap für die Backend-Konfiguration erstellen

Speichere als `backend-config.yaml`:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: backend-config
  namespace: meine-fullstack-app
data:
  config.js: |
    module.exports = {
      mongoUri: 'mongodb://admin:geheimespasswort@mongo:27017/admin',
      port: 3000
    };
```

**Erklärung:** 
ConfigMaps speichern Konfigurationsdaten im Key-Value-Format. Im Gegensatz zu Secrets sind sie für nicht-sensible Daten gedacht. Hier speichern wir eine JavaScript-Konfigurationsdatei, die später ins Backend-Pod eingebunden wird.

**Sicherheitshinweis:** 
In einer Produktionsumgebung würdest du das Passwort nicht in der ConfigMap speichern! Stattdessen sollte das Backend es aus dem Secret beziehen oder einen Dienst wie Vault verwenden.

Wende die ConfigMap an:

```bash
kubectl apply -f backend-config.yaml
```

**Überprüfung:**
```bash
kubectl get configmap backend-config -n meine-fullstack-app -o yaml
```

### 2.2 Deployment für das Backend erstellen

Speichere als `backend-deployment.yaml`:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: backend
  namespace: meine-fullstack-app
spec:
  replicas: 2
  selector:
    matchLabels:
      app: backend
  template:
    metadata:
      labels:
        app: backend
    spec:
      containers:
      - name: backend
        image: node:14-alpine
        ports:
        - containerPort: 3000
        volumeMounts:
        - name: config-volume
          mountPath: /app/config
        - name: app-volume
          mountPath: /app
        command: ["/bin/sh", "-c"]
        args:
        - |
          cat > /app/server.js << 'EOF'
          const express = require('express');
          const mongoose = require('mongoose');
          const cors = require('cors');
          const config = require('./config/config');
          
          const app = express();
          app.use(cors());
          app.use(express.json());
          
          mongoose.connect(config.mongoUri, { 
            useNewUrlParser: true, 
            useUnifiedTopology: true 
          })
          .then(() => console.log('MongoDB Connected'))
          .catch(err => console.log(err));
          
          // Simple Item model
          const Item = mongoose.model('Item', new mongoose.Schema({
            name: String,
            description: String,
            created: { type: Date, default: Date.now }
          }));
          
          // Routes
          app.get('/api/items', async (req, res) => {
            try {
              const items = await Item.find();
              res.json(items);
            } catch (err) {
              res.status(500).json({ error: err.message });
            }
          });
          
          app.post('/api/items', async (req, res) => {
            try {
              const newItem = new Item(req.body);
              const savedItem = await newItem.save();
              res.json(savedItem);
            } catch (err) {
              res.status(500).json({ error: err.message });
            }
          });
          
          app.get('/api/health', (req, res) => {
            res.json({ status: 'UP' });
          });
          
          const PORT = config.port || 3000;
          app.listen(PORT, () => console.log(`Server running on port ${PORT}`));
          EOF
          
          cd /app && npm init -y && 
          npm install express mongoose cors &&
          node server.js
        resources:
          requests:
            memory: "128Mi"
            cpu: "100m"
          limits:
            memory: "256Mi"
            cpu: "200m"
        readinessProbe:
          httpGet:
            path: /api/health
            port: 3000
          initialDelaySeconds: 15
          periodSeconds: 10
        livenessProbe:
          httpGet:
            path: /api/health
            port: 3000
          initialDelaySeconds: 30
          periodSeconds: 20
      volumes:
      - name: config-volume
        configMap:
          name: backend-config
      - name: app-volume
        emptyDir: {}
```

**Erklärung:** 
Dieses Deployment erstellt Node.js-Pods mit einem einfachen Express-Server. Besonderheiten:
- Der Code wird direkt im Container erstellt (für Produktionen würde man ein Image bauen)
- `readinessProbe` und `livenessProbe` prüfen die Gesundheit der Anwendung
- Die ConfigMap wird als Volume eingebunden
- Das `emptyDir`-Volume ist temporär und wird bei Pod-Neustarts gelöscht

**Hinweise zur Fehlersuche:**
- Die initialDelaySeconds in den Probes geben der Anwendung Zeit zum Starten, bevor Kubernetes prüft, ob sie gesund ist
- Falls die Anwendung langsamer startet, könntest du diese Werte erhöhen müssen
- Ein häufiger Fehler ist die falsche MongoDB-URI - achte auf die korrekte Syntax

Wende das Deployment an:

```bash
kubectl apply -f backend-deployment.yaml
```

**Wichtig:** 
Die erste Bereitstellung kann einige Minuten dauern, da Node.js-Abhängigkeiten installiert werden müssen.

**Fehlersuche:** 
Wenn die Pods nicht starten:

```bash
kubectl logs -l app=backend -n meine-fullstack-app
```

Häufige Probleme:
- MongoDB ist nicht erreichbar (überprüfe die MongoDB-Pods)
- NPM-Pakete können nicht installiert werden (Netzwerkprobleme)
- Der Code enthält Syntaxfehler

### 2.3 Service für das Backend erstellen

Speichere als `backend-service.yaml`:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: backend
  namespace: meine-fullstack-app
spec:
  selector:
    app: backend
  ports:
  - port: 80
    targetPort: 3000
```

**Erklärung:** 
Dieser Service macht das Backend innerhalb des Clusters unter dem Namen `backend` zugänglich. Anfragen an Port 80 des Services werden an Port 3000 der Backend-Pods weitergeleitet.

**Tipp:** 
Wir verwenden Port 80 nach außen, um den Standard-HTTP-Port zu verwenden, aber intern bleibt die Anwendung auf Port 3000.

Wende den Service an:

```bash
kubectl apply -f backend-service.yaml
```

### 2.4 Überprüfe das Backend

```bash
kubectl get deployments -n meine-fullstack-app
kubectl get pods -n meine-fullstack-app
kubectl logs -l app=backend -n meine-fullstack-app
```

**Wichtig:** 
Warte, bis alle Pods den Status `Running` haben und die Readiness-Probe erfolgreich ist.

### 2.5 Testen des Backends (optional)

Port-Forwarding zum Backend:

```bash
kubectl port-forward service/backend 3000:80 -n meine-fullstack-app
```

In einem anderen Terminal, sende Anfragen:

```bash
# Element erstellen
curl -X POST -H "Content-Type: application/json" -d '{"name":"Test Item","description":"This is a test"}' http://localhost:3000/api/items

# Alle Elemente abrufen
curl http://localhost:3000/api/items
```

**Hinweis für Windows-Nutzer:** 
Falls curl nicht verfügbar ist, kannst du einen Browser öffnen und http://localhost:3000/api/items besuchen, oder Tools wie Postman verwenden.

---

## Teil 3: Das Frontend

**Ziel:** Ein React-Frontend erstellen, das mit dem Backend kommuniziert

### 3.1 Deployment für das Frontend erstellen

Speichere als `frontend-deployment.yaml`:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: frontend
  namespace: meine-fullstack-app
spec:
  replicas: 2
  selector:
    matchLabels:
      app: frontend
  template:
    metadata:
      labels:
        app: frontend
    spec:
      containers:
      - name: frontend
        image: node:14-alpine
        ports:
        - containerPort: 3000
        volumeMounts:
        - name: app-volume
          mountPath: /app
        command: ["/bin/sh", "-c"]
        args:
        - |
          cd /app && npx create-react-app frontend
          cd frontend

          # App.js ersetzen
          cat > src/App.js << 'EOF'
          import React, { useState, useEffect } from 'react';
          import './App.css';
          
          function App() {
            const [items, setItems] = useState([]);
            const [newItem, setNewItem] = useState({ name: '', description: '' });
            const [isLoading, setIsLoading] = useState(false);
            const [error, setError] = useState(null);
            
            const backendUrl = process.env.REACT_APP_BACKEND_URL || 'http://localhost:3000';
            
            useEffect(() => {
              fetchItems();
            }, []);
            
            const fetchItems = async () => {
              setIsLoading(true);
              try {
                const response = await fetch(`${backendUrl}/api/items`);
                const data = await response.json();
                setItems(data);
                setError(null);
              } catch (err) {
                setError('Fehler beim Laden der Daten');
                console.error(err);
              } finally {
                setIsLoading(false);
              }
            };
            
            const handleSubmit = async (e) => {
              e.preventDefault();
              try {
                const response = await fetch(`${backendUrl}/api/items`, {
                  method: 'POST',
                  headers: {
                    'Content-Type': 'application/json',
                  },
                  body: JSON.stringify(newItem),
                });
                
                const data = await response.json();
                setItems([...items, data]);
                setNewItem({ name: '', description: '' });
              } catch (err) {
                setError('Fehler beim Speichern');
                console.error(err);
              }
            };
            
            return (
              <div className="App">
                <header className="App-header">
                  <h1>Kubernetes Full-Stack Demo</h1>
                </header>
                <main>
                  <section>
                    <h2>Neuen Eintrag erstellen</h2>
                    <form onSubmit={handleSubmit}>
                      <div>
                        <label htmlFor="name">Name:</label>
                        <input
                          type="text"
                          id="name"
                          value={newItem.name}
                          onChange={(e) => setNewItem({ ...newItem, name: e.target.value })}
                          required
                        />
                      </div>
                      <div>
                        <label htmlFor="description">Beschreibung:</label>
                        <textarea
                          id="description"
                          value={newItem.description}
                          onChange={(e) => setNewItem({ ...newItem, description: e.target.value })}
                          required
                        />
                      </div>
                      <button type="submit">Speichern</button>
                    </form>
                  </section>
                  
                  <section>
                    <h2>Alle Einträge</h2>
                    {isLoading && <p>Lade Daten...</p>}
                    {error && <p>Error: {error}</p>}
                    <ul>
                      {items.map(item => (
                        <li key={item._id}>
                          <h3>{item.name}</h3>
                          <p>{item.description}</p>
                          <small>Erstellt am: {new Date(item.created).toLocaleString()}</small>
                        </li>
                      ))}
                    </ul>
                  </section>
                </main>
              </div>
            );
          }
          
          export default App;
          EOF
          
          # CSS anpassen
          cat > src/App.css << 'EOF'
          .App {
            max-width: 800px;
            margin: 0 auto;
            padding: 20px;
          }
          
          .App-header {
            background-color: #282c34;
            padding: 20px;
            color: white;
            border-radius: 5px;
            margin-bottom: 20px;
          }
          
          form {
            display: flex;
            flex-direction: column;
            gap: 15px;
            margin-bottom: 30px;
            background-color: #f5f5f5;
            padding: 20px;
            border-radius: 5px;
          }
          
          form div {
            display: flex;
            flex-direction: column;
          }
          
          input, textarea {
            padding: 8px;
            border: 1px solid #ddd;
            border-radius: 4px;
            margin-top: 5px;
          }
          
          button {
            background-color: #4CAF50;
            color: white;
            padding: 10px 15px;
            border: none;
            border-radius: 4px;
            cursor: pointer;
          }
          
          button:hover {
            background-color: #45a049;
          }
          
          ul {
            list-style: none;
            padding: 0;
          }
          
          li {
            border: 1px solid #ddd;
            border-radius: 4px;
            padding: 15px;
            margin-bottom: 10px;
          }
          
          h2 {
            border-bottom: 2px solid #282c34;
            padding-bottom: 10px;
          }
          EOF
          
          # .env Datei erstellen
          echo "REACT_APP_BACKEND_URL=http://backend" > .env
          
          # Starte den Development-Server
          npm start
        resources:
          requests:
            memory: "256Mi"
            cpu: "100m"
          limits:
            memory: "512Mi"
            cpu: "300m"
      volumes:
      - name: app-volume
        emptyDir: {}
```

**Erklärung:** 
Dieses Deployment erstellt eine React-Anwendung. Ähnlich wie beim Backend wird der Code direkt im Container erstellt. Die Anwendung zeigt ein einfaches Formular zum Hinzufügen und Auflisten von Items.

**Wichtiger Hinweis:** 
Diese Methode (Code im Container zu erstellen) eignet sich nur für Lernzwecke. In einer echten Umgebung würdest du:
1. Ein eigenes Docker-Image mit der Anwendung erstellen
2. Dieses Image in einer Registry speichern
3. Das Image im Deployment referenzieren

**Mögliche Probleme:**
- Create-React-App benötigt viel Speicher - achte auf ausreichende Ressourcen
- Die erste Installation dauert lange (10+ Minuten)
- In Minikube mit begrenzten Ressourcen kann dies zum Timeout führen

Wende das Deployment an:

```bash
kubectl apply -f frontend-deployment.yaml
```

**Wichtiger Hinweis:** 
Die Erstellung der React-App nimmt viel Zeit in Anspruch! Dies kann 10-15 Minuten dauern, je nach Ressourcen deines Clusters.

**Fehlersuche:** 
Falls die Pods abstürzen oder neu starten:

```bash
kubectl describe pod -l app=frontend -n meine-fullstack-app
```

Häufige Probleme:
- OOMKilled: Nicht genug Speicher (versuche, die Ressourcenlimits zu erhöhen)
- NPM-Fehler (Netzwerkprobleme oder zu wenig Ressourcen)

### 3.2 Service für das Frontend erstellen

Speichere als `frontend-service.yaml`:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: frontend
  namespace: meine-fullstack-app
spec:
  selector:
    app: frontend
  ports:
  - port: 80
    targetPort: 3000
```

**Erklärung:** 
Ähnlich wie beim Backend-Service wird hier Port 80 auf Port 3000 der Frontend-Pods gemappt, was später für den Ingress wichtig ist.

Wende den Service an:

```bash
kubectl apply -f frontend-service.yaml
```

**Überprüfung:**
```bash
kubectl get svc -n meine-fullstack-app
```

---

## Teil 4: Ingress-Controller und Routing

**Ziel:** Ein Ingress-Controller einrichten, um die Anwendung von außen zugänglich zu machen

### 4.1 Ingress-Controller aktivieren (Minikube)

```bash
minikube addons enable ingress
```

**Erklärung:** 
Der Ingress-Controller ist ein Reverse-Proxy, der HTTP-Anfragen an die richtigen Services weiterleitet. In Minikube ist er als Add-on verfügbar und basiert auf NGINX.

**Hinweis für andere Umgebungen:** 
- Für Kind: [NGINX Ingress Controller Installation](https://kind.sigs.k8s.io/docs/user/ingress/)
- Für Cloud-Anbieter: Viele bieten eigene Ingress-Controller an (z.B. ALB in AWS)

**Fehlersuche:** 
Prüfe, ob der Ingress-Controller läuft:

```bash
kubectl get pods -n ingress-nginx
```

Es sollte mindestens einen laufenden Pod mit "controller" im Namen geben.

### 4.2 Ingress-Ressource erstellen

Speichere als `app-ingress.yaml`:

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: app-ingress
  namespace: meine-fullstack-app
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /$1
spec:
  rules:
  - host: fullstack-app.local
    http:
      paths:
      - path: /api(/|$)(.*)
        pathType: Prefix
        backend:
          service:
            name: backend
            port:
              number: 80
      - path: /(.*)
        pathType: Prefix
        backend:
          service:
            name: frontend
            port:
              number: 80
```

**Erklärung:** 
- Diese Ingress-Ressource definiert Routing-Regeln für unsere Anwendung
- Anfragen an `/api/*` werden an den Backend-Service weitergeleitet
- Alle anderen Anfragen (`/*`) gehen an den Frontend-Service
- Die `rewrite-target`-Annotation entfernt das Präfix, bevor die Anfrage weitergeleitet wird

**Wichtiger Hinweis:** 
Die Pfad-Reihenfolge ist wichtig! Spezifischere Pfade (wie `/api`) müssen vor allgemeineren Pfaden (wie `/`) kommen.

Wende die Ingress-Ressource an:

```bash
kubectl apply -f app-ingress.yaml
```

**Fehlersuche:** 
Überprüfe den Status des Ingress:

```bash
kubectl get ingress -n meine-fullstack-app
kubectl describe ingress app-ingress -n meine-fullstack-app
```

Es kann einige Minuten dauern, bis der Ingress aktiv wird.

### 4.3 Hosts-Datei anpassen

Füge folgende Zeile zu deiner Hosts-Datei hinzu:

```
127.0.0.1 fullstack-app.local
```

- Unter Linux/macOS: `/etc/hosts`
- Unter Windows: `C:\Windows\System32\drivers\etc\hosts`

**Hinweis für Windows-Nutzer:** 
Du musst die Hosts-Datei als Administrator bearbeiten.

**Alternative:** 
Du kannst auch die IP-Adresse direkt verwenden, ohne die Hosts-Datei zu ändern. Finde die IP mit `kubectl get ingress -n meine-fullstack-app`.

### 4.4 Port-Forwarding zum Ingress-Controller

```bash
kubectl port-forward -n ingress-nginx service/ingress-nginx-controller 8080:80
```

Jetzt kannst du deine Anwendung im Browser unter http://fullstack-app.local:8080 erreichen!

---

## Teil 5: Monitoring mit Prometheus und Grafana

**Ziel:** Ein Monitoring-Setup für unsere Anwendung einrichten

### 5.1 Helm-Repository hinzufügen

```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
```

**Erklärung:** 
Helm ist ein Paketmanager für Kubernetes, ähnlich wie apt für Ubuntu oder npm für JavaScript. Mit Helm können wir komplexe Anwendungen wie Prometheus und Grafana einfach installieren.

### 5.2 Namespace für Monitoring erstellen

```bash
kubectl create namespace monitoring
```

### 5.3 Prometheus und Grafana installieren

```bash
helm install prometheus prometheus-community/kube-prometheus-stack \
  --namespace monitoring \
  --set grafana.enabled=true \
  --set prometheus.enabled=true
```

**Erklärung:**
- Prometheus sammelt Metriken von deinen Anwendungen und der Kubernetes-Infrastruktur
- Grafana visualisiert diese Daten in Dashboards
- Der kube-prometheus-stack installiert beide Komponenten und konfiguriert sie für den sofortigen Einsatz

**Häufige Probleme:**
- Wenn du begrenzte Ressourcen hast, kann die Installation fehlschlagen. Versuche in diesem Fall, andere Ressourcen freizugeben.
- Bei Berechtigungsproblemen kannst du `--set rbac.create=true` hinzufügen.

### 5.4 Überprüfe die Installation

```bash
kubectl get pods -n monitoring
```

Die Ausgabe sollte verschiedene Pods für Prometheus, Grafana und Alert Manager zeigen.

### 5.5 Zugriff auf Grafana

```bash
kubectl port-forward svc/prometheus-grafana 3000:80 -n monitoring
```

Öffne in deinem Browser: http://localhost:3000
- Benutzername: admin
- Passwort: prom-operator

**Tipp:** 
In Grafana kannst du unter "Dashboards" > "Browse" vorkonfigurierte Dashboards finden. Besonders interessant sind:
- Kubernetes Cluster Overview
- Kubernetes Deployments
- Node Exporter Full

---

## Teil 6: Horizontale Pod-Autoskalierung

**Ziel:** Automatische Skalierung für unsere Anwendung basierend auf der CPU-Auslastung

### 6.1 Metrics Server aktivieren (für Minikube)

```bash
minikube addons enable metrics-server
```

**Erklärung:**
Der Metrics Server sammelt Ressourcennutzungsdaten (CPU, Speicher) von Nodes und Pods. Dies ist eine Voraussetzung für die Autoskalierung in Kubernetes.

**Wichtig:** Nach der Aktivierung kann es einige Minuten dauern, bis der Metrics Server Daten sammelt.

### 6.2 HorizontalPodAutoscaler erstellen

Speichere als `backend-hpa.yaml`:

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: backend-hpa
  namespace: meine-fullstack-app
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: backend
  minReplicas: 2
  maxReplicas: 5
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 50
```

**Erklärung:**
- Dieser HPA überwacht das Backend-Deployment
- Er skaliert zwischen 2 und 5 Replicas
- Die Skalierung erfolgt, wenn die durchschnittliche CPU-Auslastung 50% überschreitet
- Bei hoher Auslastung werden neue Pods erstellt, bei niedriger werden sie entfernt

Wende den HPA an:

```bash
kubectl apply -f backend-hpa.yaml
```

### 6.3 HPA überwachen

```bash
kubectl get hpa -n meine-fullstack-app -w
```

Der Parameter `-w` lässt den Befehl laufen und zeigt Änderungen in Echtzeit an.

### 6.4 Last erzeugen (optional)

In einem neuen Terminal:

```bash
kubectl run -i --tty load-generator --rm --image=busybox:1.28 --restart=Never -n meine-fullstack-app -- /bin/sh -c "while sleep 0.01; do wget -q -O- http://backend/api/items; done"
```

**Erklärung:**
Dieser Befehl erstellt einen temporären Pod, der kontinuierlich Anfragen an das Backend sendet, um Last zu erzeugen. Im ersten Terminal solltest du beobachten können, wie der HPA die Anzahl der Replicas erhöht.

**Tipp:**
Um den Lasttest zu beenden, drücke CTRL+C im Terminal, in dem der load-generator läuft.

---

## Geschafft!

Herzlichen Glückwunsch! Du hast erfolgreich eine komplette Multi-Tier-Anwendung in Kubernetes aufgebaut, mit:

1. Einer persistenten MongoDB-Datenbank (StatefulSet)
2. Einem skalierbaren Node.js-Backend (Deployment mit HPA)
3. Einem React-Frontend (Deployment)
4. Routing mit Ingress
5. Monitoring mit Prometheus und Grafana
6. Automatischer Skalierung

Du hast damit die wichtigsten Konzepte für den Betrieb moderner Anwendungen in Kubernetes kennengelernt.

### Nächste Schritte

Wenn du mehr lernen möchtest:

- **GitOps**: Automatisiere Deployments mit [Argo CD](https://argo-cd.readthedocs.io/en/stable/) oder [Flux](https://fluxcd.io/docs/)
- **CI/CD-Pipeline**: Integriere deine Anwendung in eine CI/CD-Pipeline mit GitHub Actions oder Jenkins
- **Service Mesh**: Füge ein [Istio](https://istio.io/latest/docs/) oder [Linkerd](https://linkerd.io/2.12/overview/) Service Mesh hinzu
- **Zero-Downtime Deployments**: Implementiere [Rolling Updates](https://kubernetes.io/docs/tutorials/kubernetes-basics/update/update-intro/) und [Blue/Green Deployments](https://kubernetes.io/blog/2018/04/30/zero-downtime-deployment-kubernetes-jenkins/)
- **Datensicherung**: Implementiere [Backup und Wiederherstellung](https://velero.io/docs/) für deine Datenbank