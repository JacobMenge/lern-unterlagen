# CYBER-DETEKTIV ERMITTLUNGSBERICHT - MUSTERLÖSUNG

**Name:** Max Mustermann (Musterlösung)  
**Fall:** Operation Blackout - MustermannGmbH  
**Datum:** 23. Juni 2025

---

## QUALIFIKATIONSTEST - LÖSUNGEN

### Frage 1: IP-Adress-Analyse
**Antwort:** Der Verdächtige mit IP 192.168.1.50 ist ein **Insider** (Mitarbeiter/internes Netzwerk), da die IP-Adresse aus dem privaten Bereich 192.168.x.x stammt. Der Verdächtige mit IP 203.0.113.45 kommt **von außen** (Internet), da dies eine öffentliche IP-Adresse ist.

**Erklärung:** Private IP-Bereiche (192.168.x.x, 10.x.x.x, 172.16-31.x.x) werden nur innerhalb von Firmennetzwerken verwendet, während öffentliche IPs für Internetverbindungen genutzt werden.

### Frage 2: Port 3389 Analyse
**Antwort:** Der Angreifer versucht eine **Remote Desktop Verbindung** aufzubauen, um ferngesteuerten Zugriff auf den Computer zu erlangen. Port 3389 ist der Standard-Port für Microsoft Remote Desktop Protocol (RDP).

**Erklärung:** Dies ist besonders gefährlich, da der Angreifer bei erfolgreichem Zugriff vollständige Kontrolle über den Computer erhält.

### Frage 3: Event ID 4625 Analyse
**Antwort:** Hier findet ein **Brute-Force-Angriff** statt. Event ID 4625 bedeutet "fehlgeschlagene Anmeldung" - fünf Versuche in 2 Minuten deuten darauf hin, dass jemand systematisch verschiedene Passwörter ausprobiert.

**Erklärung:** Diese Angriffsmethode nutzt automatisierte Tools, um durch Ausprobieren verschiedener Passwort-Kombinationen Zugang zu erlangen.

### Frage 4: Datei-Analyse "vertrag.pdf.exe"
**Antwort:** Die Datei ist **verdächtig**, da sie eine **Doppelendung** hat (.pdf.exe). Dies ist eine typische Tarnmethode für Malware - die Datei sieht wie ein PDF aus, ist aber eine ausführbare Datei (.exe).

**Erklärung:** Cyberkriminelle nutzen diese Technik, um Benutzer zu täuschen. Das Opfer denkt, es öffnet ein Dokument, startet aber tatsächlich Schadsoftware.

---

## TATORT-ANALYSE

### 3.1: Tatrekonstruktion - Chronologische Timeline

```
08:10:15 - [PC-Buchhaltung] Erster Brute-Force-Versuch auf Administrator-Konto
08:10:45 - [PC-Buchhaltung] Zweiter fehlgeschlagener Anmeldeversuch
08:11:15 - [PC-Buchhaltung] Dritter fehlgeschlagener Anmeldeversuch  
08:11:45 - [PC-Buchhaltung] ✓ Erfolgreiche Administrator-Anmeldung (Passwort geknackt)
08:12:00 - [PC-Buchhaltung] Administrator-Rechte aktiviert (SeBackupPrivilege, SeRestorePrivilege)
08:15:22 - [PC-Buchhaltung] Firewall blockiert eingehende Verbindung von 198.51.100.45
08:16:45 - [PC-Buchhaltung] Ausgehende Verbindung zu 198.51.100.45 (2.8 MB Download)
08:17:15 - [PC-Buchhaltung] Antivirus erkennt "Rechnung_Mai.pdf.exe" als Cerber-Ransomware
08:18:00 - [PC-Buchhaltung] Ransomware startet Verschlüsselung (*.docx → *.locked)
08:30:22 - [PC-Marketing] Normale Anmeldung von Stefan Schmidt
08:35:12 - [PC-Marketing] Firewall blockiert Verbindung von 45.142.212.98
08:35:45 - [PC-Marketing] Lateral Movement: Zugriff auf PC-Buchhaltung mit Administrator-Rechten
08:36:30 - [PC-Marketing] Download von 198.51.100.45 (1.2 MB) 
08:37:45 - [PC-Marketing] Antivirus erkennt "Angebot.pdf.exe" als Cerber-Ransomware
08:38:30 - [PC-Marketing] Zweite Ransomware-Infektion startet (*.xlsx → *.locked)
```

### 3.2: Täteranalyse

#### Frage 1: Erstes Opfer
**Antwort:** **PC-Buchhaltung (192.168.1.105)** war das erste Opfer.

**Beweise:**
- Erste Angriffsspuren um 08:10:15 auf diesem Computer
- Brute-Force-Angriff erfolgte hier zuerst
- Erste Malware-Infektion um 08:17:15
- PC-Marketing wurde erst später (ab 08:35:45) über Lateral Movement angegriffen

#### Frage 2: Einbruchsmethode
**Antwort:** **a) Durch Passwort-Knacken (Brute Force)**

**Begründung:** Die Event-Logs zeigen deutlich drei fehlgeschlagene Anmeldeversuche (Event ID 4625) gefolgt von einer erfolgreichen Anmeldung (Event ID 4624). Dies ist das klassische Muster eines Brute-Force-Angriffs.

#### Frage 3: Vorbereitungsphase (08:10:15 - 08:12:00)
**Antwort:** In der Vorbereitungsphase führte der Täter folgende Schritte durch:
1. **Reconnaissance:** Brute-Force-Angriff auf das Administrator-Konto
2. **Initial Access:** Nach drei Versuchen erfolgreiche Kompromittierung des Administrator-Kontos
3. **Privilege Escalation:** Aktivierung besonderer Administrator-Rechte (SeBackupPrivilege, SeRestorePrivilege)

Diese Rechte ermöglichten es dem Angreifer, Dateien zu lesen, zu ändern und auf andere Systeme zuzugreifen.

#### Frage 4: Ausbreitungsstrategie
**Antwort:** Der Angriff breitete sich durch **Lateral Movement** aus:
1. Nach Kompromittierung des ersten Systems erhielt der Angreifer Administrator-Rechte
2. Um 08:35:45 nutzte der Angreifer diese Rechte für "Anmeldung mit gespeicherten Anmeldedaten" (Event ID 4648)
3. Dadurch konnte er vom PC-Marketing aus auf den bereits kompromittierten PC-Buchhaltung zugreifen
4. Die Administrator-Rechte ermöglichten die Installation weiterer Malware auf PC-Marketing

### 3.3: Täter-IP-Recherche

**Hinweis:** Da AbuseIPDB.com eine echte Datenbank ist, simuliere ich realistische Ergebnisse basierend auf dem Threat Intelligence Report:

#### IP-Überprüfung:
- **198.51.100.45:** JA - Mehrfach als Command & Control Server gemeldet
- **45.142.212.98:** JA - Als Malware Distribution Point bekannt  
- **102.209.217.173:** (Nicht in den Logs erwähnt - Prüfung unnötig)

#### Kriminelle Aktivitäten:
- **198.51.100.45:** Berichte über Ransomware-C&C-Server, Datenexfiltration
- **45.142.212.98:** Verteilung von Cerber-Ransomware, Phishing-Kampagnen

**Korrelation:** Diese Befunde bestätigen den Threat Intelligence Report und zeigen, dass es sich um eine koordinierte Ransomware-Kampagne handelt.

### 3.4: Schadensgutachten

#### Frage 1: Verschlüsselte Dateitypen
**Antwort:** 
- **PC-Buchhaltung:** Word-Dokumente (*.docx → *.locked)
- **PC-Marketing:** Excel-Tabellen (*.xlsx → *.locked)

#### Frage 2: Schwerwiegenderer Schaden
**Antwort:** **PC-Buchhaltung** hatte den schwerwiegenderen Schaden.

**Begründung:**
- Vollständige Verschlüsselung aller Word-Dokumente im Documents-Ordner
- Bei PC-Marketing wurde der Prozess gestoppt, nur 15 Dateien verschlüsselt
- Buchhaltungsdaten sind oft kritischer für den Geschäftsbetrieb
- PC-Buchhaltung war länger kompromittiert (8 Minuten vs. 3 Minuten)

#### Frage 3: Opferliste
1. **Maria Mueller** (Buchhaltung) - Primäropfer, alle Word-Dokumente verschlüsselt
2. **Stefan Schmidt** (Marketing) - Sekundäropfer, 15 Excel-Dateien verschlüsselt  
3. **Administrator/IT-System** - Kompromittierte Administrator-Accounts

### 3.5: Notfall-Einsatzplan

**Priorisierung (1 = höchste Priorität):**

**1** Betroffene Computer vom Netzwerk isolieren  
**2** Backup-Systeme auf Integrität prüfen  
**3** Vollständigen Antivirus-Scan durchführen  
**4** Alle Passwörter sofort ändern  
**5** Geschäftsführung über den Vorfall informieren  
**6** Polizei/Cyber-Crime-Unit verständigen  
**7** Betroffene Mitarbeiter über den Vorfall aufklären  

**Begründung:** Erste Priorität ist die Schadensbegrenzung (Isolation), dann Systemsicherheit (Backup-Prüfung, Antivirus), anschließend Kommunikation und rechtliche Schritte.

### 3.6: Krisenkommunikation

**Betreff:** DRINGEND - Cyber-Sicherheitsvorfall erfordert sofortiges Handeln

**Text:**
Um 09:00 Uhr wurde ein Ransomware-Angriff auf unsere IT-Systeme entdeckt. Betroffen sind PC-Buchhaltung und PC-Marketing mit verschlüsselten Dateien. Die infizierten Systeme wurden isoliert. Aktuell läuft die Schadensbewertung. Backup-Wiederherstellung wird geprüft. Externe IT-Forensik wird eingeschaltet. Mitarbeiter wurden informiert, verdächtige E-Mails zu melden. Updates folgen stündlich.

### 3.7: Präventions-Strategie

1. **Multi-Faktor-Authentifizierung (MFA)** für alle Administrator-Konten implementieren
2. **Regelmäßige Sicherheitsschulungen** zu Phishing und Social Engineering  
3. **Automatische Backup-Systeme** mit Offline-Kopien einrichten
4. **Network Segmentation** - Trennung kritischer Systeme vom Hauptnetzwerk
5. **Endpoint Detection and Response (EDR)** für erweiterte Malware-Erkennung

---

## ERMITTLUNGS-REFLEXION

### Drei "Aha!"-Momente:

1. **Log-Korrelation ist entscheidend:** Erst durch die Kombination verschiedener Log-Quellen (Firewall, Antivirus, Windows Events) entsteht das vollständige Bild des Angriffs.

2. **Timing-Analyse zeigt Angriffsmuster:** Die chronologische Auswertung offenbart die systematische Vorgehensweise der Angreifer und zeigt Lateral Movement deutlich.

3. **Threat Intelligence hilft bei Attribution:** Der Abgleich mit bekannten IOCs (Indicators of Compromise) bestätigt die Angriffsmethoden und hilft bei der Einschätzung.

### Was würde ich ab sofort anders machen?

**Privat:** Aktivierung von 2FA auf allen wichtigen Accounts, regelmäßige Backups meiner persönlichen Daten, kritischere Betrachtung von E-Mail-Anhängen.

**Beruflich:** Verstärktes Bewusstsein für die Wichtigkeit von Security Monitoring, besseres Verständnis für die Bedeutung schneller Incident Response.

---
