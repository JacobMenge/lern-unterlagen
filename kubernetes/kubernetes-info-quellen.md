# Kubernetes: Grundlagenwissen für die Übungen

Hier findest du das notwendige Wissen für die drei Kubernetes-Übungen und passende Quellen zum Vertiefen. Die Inhalte sind nach Schwierigkeitsgrad und Themengebiet strukturiert.

## Grundlagen (für die erste Übung)

### Kubernetes-Architektur

**Was du wissen solltest:**
- Kubernetes besteht aus Control Plane (Master) und Worker-Nodes
- Der API-Server ist der zentrale Zugangspunkt für alle Anfragen
- Kubelet ist für die Ausführung von Containern auf den Nodes verantwortlich

**Weiterführende Quellen:**
- [Kubernetes-Komponenten](https://kubernetes.io/de/docs/concepts/overview/components/) - Offizielle Dokumentation
- [Kubernetes-Architektur erklärt](https://spot.io/resources/kubernetes-architecture-101/) - Visueller Leitfaden

### Pods verstehen

**Was du wissen solltest:**
- Pods sind die kleinste deploybare Einheit in Kubernetes
- Ein Pod kann einen oder mehrere Container enthalten
- Container im selben Pod teilen sich Netzwerk und Speicher

**Weiterführende Quellen:**
- [Pod-Übersicht](https://kubernetes.io/docs/concepts/workloads/pods/) - Offizielle Dokumentation
- [Pod-Lifecycle](https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/) - Zustände und Phasen

### Deployments

**Was du wissen solltest:**
- Deployments verwalten ReplicaSets, die wiederum Pods verwalten
- Sie ermöglichen deklarative Updates und Rollbacks
- Die Replica-Anzahl bestimmt, wie viele identische Pods laufen sollen

**Weiterführende Quellen:**
- [Deployments](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/) - Offizielle Dokumentation
- [Kubernetes Deployments erklärt](https://www.containiq.com/post/kubernetes-deployments) - Praxisnahe Erklärung

### Services

**Was du wissen solltest:**
- Services bieten eine stabile Netzwerkadresse für Pods
- Sie ermöglichen Load Balancing zwischen Pods
- Service-Typen: ClusterIP, NodePort, LoadBalancer, ExternalName

**Weiterführende Quellen:**
- [Services](https://kubernetes.io/docs/concepts/services-networking/service/) - Offizielle Dokumentation
- [Kubernetes-Services für Anfänger](https://matthewpalmer.net/kubernetes-app-developer/articles/service-kubernetes-example-tutorial.html) - Einfache Erklärung mit Beispielen

### ConfigMaps

**Was du wissen solltest:**
- ConfigMaps speichern nicht-sensible Konfigurationsdaten
- Diese können als Umgebungsvariablen oder Dateien in Pods eingebunden werden
- Sie erlauben die Trennung von Code und Konfiguration

**Weiterführende Quellen:**
- [ConfigMaps](https://kubernetes.io/docs/concepts/configuration/configmap/) - Offizielle Dokumentation
- [ConfigMaps und Secrets](https://kubernetes.io/docs/tutorials/configuration/configure-redis-using-configmap/) - Tutorial

## Mittelstufe (für die zweite Übung)

### Persistenter Speicher

**Was du wissen solltest:**
- PersistentVolumes (PV) sind Speicherressourcen im Cluster
- PersistentVolumeClaims (PVC) sind Anforderungen an diesen Speicher
- StorageClasses definieren dynamisch bereitgestellte Speicherressourcen

**Weiterführende Quellen:**
- [Persistente Volumes](https://kubernetes.io/docs/concepts/storage/persistent-volumes/) - Offizielle Dokumentation
- [Kubernetes Storage erklärt](https://www.youtube.com/watch?v=0swOh5C3OVM) - Video-Tutorial

### Secrets

**Was du wissen solltest:**
- Secrets speichern sensible Daten wie Passwörter und Tokens
- Sie werden Base64-codiert (aber nicht verschlüsselt!)
- Können als Umgebungsvariablen oder Dateien eingebunden werden

**Weiterführende Quellen:**
- [Secrets](https://kubernetes.io/docs/concepts/configuration/secret/) - Offizielle Dokumentation
- [Kubernetes Secrets besser verwalten](https://learnk8s.io/kubernetes-secrets-in-git) - Best Practices

### Ingress

**Was du wissen solltest:**
- Ingress steuert den externen Zugriff auf Services im Cluster
- Ermöglicht HTTP/HTTPS-Routing, URL-Pfadweiterleitung und mehr
- Benötigt einen Ingress-Controller (z.B. NGINX, Traefik)

**Weiterführende Quellen:**
- [Ingress](https://kubernetes.io/docs/concepts/services-networking/ingress/) - Offizielle Dokumentation
- [Ingress-Controller einrichten](https://kubernetes.github.io/ingress-nginx/deploy/) - Praktische Anleitung

### Horizontale Pod-Autoskalierung

**Was du wissen solltest:**
- HorizontalPodAutoscaler (HPA) skaliert Pods basierend auf Ressourcennutzung
- Benötigt den Metrics Server für CPU/Memory-Metriken
- Kann mit benutzerdefinierten Metriken erweitert werden

**Weiterführende Quellen:**
- [Horizontale Autoskalierung](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale/) - Offizielle Dokumentation
- [HPA Walkthrough](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale-walkthrough/) - Schritt-für-Schritt-Anleitung

## Fortgeschritten (für die dritte Übung)

### StatefulSets

**Was du wissen solltest:**
- StatefulSets sind für zustandsbehaftete Anwendungen wie Datenbanken
- Bieten stabile Netzwerkidentitäten und persistenten Speicher
- Pods werden in geordneter Reihenfolge erstellt und gelöscht

**Weiterführende Quellen:**
- [StatefulSets](https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/) - Offizielle Dokumentation
- [StatefulSets erklärt](https://cloud.google.com/kubernetes-engine/docs/concepts/statefulset) - Google Cloud-Dokumentation

### Multi-Tier-Anwendungen

**Was du wissen solltest:**
- Aufteilung in Frontend-, Backend- und Datenbankschicht
- Jede Schicht kann unabhängig skaliert werden
- Kommunikation zwischen den Schichten über Services

**Weiterführende Quellen:**
- [Beispiel: Mehrstufige Anwendung](https://kubernetes.io/docs/tutorials/stateless-application/guestbook/) - Gästebuch-Tutorial
- [Multi-Tier-Architektur mit Kubernetes](https://www.weave.works/blog/kubernetes-patterns-multi-tier-applications) - Weave Works Blog

### Monitoring mit Prometheus und Grafana

**Was du wissen solltest:**
- Prometheus sammelt Metriken durch "Scraping" von Endpunkten
- Grafana visualisiert diese Metriken in Dashboards
- Der Prometheus Operator vereinfacht die Einrichtung

**Weiterführende Quellen:**
- [Prometheus-Operator](https://github.com/prometheus-operator/prometheus-operator) - GitHub-Repository
- [Kubernetes-Monitoring mit Prometheus](https://sysdig.com/blog/kubernetes-monitoring-prometheus/) - Umfassender Guide
- [Grafana-Dashboard für Kubernetes](https://grafana.com/grafana/dashboards/6417) - Vorgefertigtes Dashboard

### Ressourcenverwaltung

**Was du wissen solltest:**
- Requests definieren garantierte Ressourcen für einen Container
- Limits definieren maximale Ressourcen, die ein Container nutzen darf
- QoS-Klassen (Quality of Service) bestimmen Priorität bei Ressourcenknappheit

**Weiterführende Quellen:**
- [Ressourcenverwaltung](https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/) - Offizielle Dokumentation
- [Best Practices für Ressourcenlimits](https://cloud.google.com/blog/products/containers-kubernetes/kubernetes-best-practices-resource-requests-and-limits) - Google Cloud Blog

### Headless Services

**Was du wissen solltest:**
- Headless Services (clusterIP: None) bieten keine Load-Balancing
- Ermöglichen direkten Zugriff auf einzelne Pods über DNS
- Werden häufig mit StatefulSets verwendet

**Weiterführende Quellen:**
- [Headless Services](https://kubernetes.io/docs/concepts/services-networking/service/#headless-services) - Offizielle Dokumentation
- [DNS für Services und Pods](https://kubernetes.io/docs/concepts/services-networking/dns-pod-service/) - Interne DNS-Namensauflösung

## Praktische Tools und Cheat Sheets

### kubectl

**Die wichtigsten Befehle:**
```bash
# Ressourcen anzeigen
kubectl get pods/deployments/services

# Details anzeigen
kubectl describe pod POD_NAME

# Logs anzeigen
kubectl logs POD_NAME

# In einen Pod einsteigen
kubectl exec -it POD_NAME -- /bin/bash

# Konfiguration anwenden
kubectl apply -f datei.yaml

# Ressource löschen
kubectl delete -f datei.yaml
```

**Weiterführende Quellen:**
- [kubectl Cheat Sheet](https://kubernetes.io/docs/reference/kubectl/cheatsheet/) - Offizielle Dokumentation
- [kubectl Befehls-Referenz](https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands) - Vollständige Referenz

### Minikube

**Grundlegende Befehle:**
```bash
# Starten
minikube start

# Dashboard öffnen
minikube dashboard

# Tunnel für LoadBalancer-Services
minikube tunnel

# Add-ons aktivieren
minikube addons enable ingress

# IP-Adresse anzeigen
minikube ip
```

**Weiterführende Quellen:**
- [Minikube Dokumentation](https://minikube.sigs.k8s.io/docs/) - Offizielle Dokumentation
- [Minikube-Tutorial](https://kubernetes.io/docs/tutorials/hello-minikube/) - Erste Schritte

## Lernpfade und Zertifizierungen

Wenn du tiefer in Kubernetes einsteigen möchtest:

1. **Certified Kubernetes Administrator (CKA)**
   - [Offizielle Informationen](https://www.cncf.io/certification/cka/)
   - [Prüfungsinhalte](https://github.com/cncf/curriculum)

2. **Certified Kubernetes Application Developer (CKAD)**
   - [Offizielle Informationen](https://www.cncf.io/certification/ckad/)
   - [Vorbereitungskurs](https://www.udemy.com/course/certified-kubernetes-application-developer/)

3. **Kostenlose Kubernetes-Kurse**
   - [Introduction to Kubernetes](https://www.edx.org/course/introduction-to-kubernetes) - edX/Linux Foundation
   - [Kubernetes Fundamentals](https://www.katacoda.com/courses/kubernetes) - Interaktive Labs

## Troubleshooting-Tipps

### Häufige Probleme und Lösungen

**Pod startet nicht:**
1. Überprüfe den Pod-Status: `kubectl describe pod POD_NAME`
2. Prüfe auf Image-Pull-Probleme oder Ressourcenmangel
3. Schau in die Logs: `kubectl logs POD_NAME`

**Service ist nicht erreichbar:**
1. Überprüfe, ob Selector und Pod-Labels übereinstimmen
2. Überprüfe, ob die Ports korrekt konfiguriert sind
3. Teste den Service innerhalb des Clusters: `kubectl run -it --rm debug --image=busybox -- wget -O- SERVICE_NAME:PORT`

**PersistentVolumeClaim bleibt im Pending-Status:**
1. Überprüfe, ob ein passendes PersistentVolume verfügbar ist
2. Prüfe, ob StorageClass, AccessModes und Kapazität übereinstimmen

**Weiterführende Quellen:**
- [Kubernetes Troubleshooting](https://kubernetes.io/docs/tasks/debug-application-cluster/troubleshooting/) - Offizielle Dokumentation
- [Kubernetes Debugging](https://kubernetes.io/docs/tasks/debug-application-cluster/debug-application/) - Anwendungsfehler beheben

---

Dieses Dokument bietet dir das notwendige Grundlagenwissen für die drei Kubernetes-Übungen und weiterführende Quellen zum Vertiefen. Bei spezifischen Fragen oder Problemen konsultiere die verlinkte Dokumentation oder frage im Classroom nach.
