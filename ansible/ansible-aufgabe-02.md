# Freiwillige Aufgabe: MongoDB lokal mit Ansible installieren und konfigurieren

## Ziel
In dieser freiwilligen Aufgabe erstellst du ein Ansible-Setup, mit dem du eine lokale MongoDB-Instanz automatisiert installieren und konfigurieren kannst. Dabei geht es nicht nur darum, die Datenbank zum Laufen zu bringen, sondern auch um ein grundlegendes Verständnis für Automatisierung mit Ansible und den Umgang mit Systemdiensten.

## Was du tun sollst

### 1. Projekt vorbereiten
* Lege ein neues Verzeichnis für dein Ansible-Projekt an, zum Beispiel `mongodb-ansible`
* Erstelle ein Inventar, in dem `localhost` als Zielhost definiert ist
* Sorge dafür, dass du Playbooks auf deinem lokalen System ausführen kannst (z. B. mit `connection: local`)

### 2. MongoDB installieren
* Schreibe ein Ansible-Playbook, das:
  * die nötigen Systempakete installiert (z. B. `gnupg`, `curl` etc.)
  * das MongoDB-Repository einrichtet (z. B. über `apt` unter Ubuntu oder `yum` bei CentOS)
  * MongoDB installiert
  * den MongoDB-Dienst startet und so konfiguriert, dass er beim Systemstart automatisch mitgestartet wird

### 3. MongoDB konfigurieren
* Passe die Konfigurationsdatei so an, dass:
  * nur Verbindungen von `localhost` erlaubt sind
  * der Standardport 27017 verwendet wird
  * optional: ein Admin-Benutzer mit Passwort eingerichtet wird und die Authentifizierung aktiviert ist

### 4. Test und Verifikation
* Teste deine Installation, zum Beispiel über einen lokalen Verbindungsversuch mit der `mongo`-Shell
* Du kannst auch einen einfachen Check als zusätzlichen Ansible-Task integrieren, der prüft, ob der Dienst läuft

## Strukturvorschlag
Wenn du mit Rollen arbeitest, könnte dein Projekt so aussehen:

```
mongodb-ansible/
├── inventory
├── playbook.yml
├── roles/
│   └── mongodb/
│       ├── tasks/
│       │   └── main.yml
│       └── templates/
│           └── mongod.conf.j2
```
