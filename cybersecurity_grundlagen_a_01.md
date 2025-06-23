# Cybersecurity Detektiv: Der Fall MustermannGmbH

**Ihre Mission:** Sie sind Cyber-Ermittler und müssen einen mysteriösen Sicherheitsvorfall aufklären!  
**Abgabe:** Textdokument mit Ihrem kompletten Ermittlungsbericht, bsp. seht ihr am Ende.

---

## TEIL 1: IHR ERMITTLER-BRIEFING (Lesen & Verstehen)

### 1.1 Was ist Cybersecurity?

**Cybersecurity** = Schutz von Computersystemen, Netzwerken und Daten vor digitalen Angriffen.

**Die drei Grundprinzipien (CIA-Triade):**
- **Confidentiality (Vertraulichkeit):** Nur berechtigte Personen haben Zugriff
- **Integrity (Integrität):** Daten sind korrekt und unverändert  
- **Availability (Verfügbarkeit):** Systeme funktionieren wenn benötigt

### 1.2 Die häufigsten Cyber-Verbrechen (Ihr Fachwissen)

**Ransomware (Erpresser-Software):**
- Verschlüsselt Dateien auf dem Computer
- Fordert Lösegeld für Entschlüsselung
- **Erkennungszeichen:** Dateien werden umbenannt (z.B. .locked, .encrypted)
- **Typische Familien:** Cerber, Locky, WannaCry

**Phishing (Köder-E-Mails):**
- Gefälschte E-Mails mit schädlichen Anhängen
- Ziel: Passwörter stehlen oder Malware installieren
- **Tarnmethode:** Anhänge oft als .pdf.exe getarnt

**Brute Force (Passwort-Knacken):**
- Systematisches Ausprobieren von Passwörtern
- **Erkennbar durch:** Viele fehlgeschlagene Login-Versuche hintereinander

### 1.3 Digitale Spuren verstehen: IP-Adressen

**IP-Adresse = eindeutige "Hausnummer" für jeden Computer im Netzwerk**

**Wichtige Bereiche:**
- **Private IPs (Firmen-intern):** 192.168.x.x, 10.x.x.x, 172.16-31.x.x
- **Öffentliche IPs (Internet):** Alle anderen (z.B. 203.0.113.50, 8.8.8.8)
- **Localhost:** 127.0.0.1 (eigener Computer)

**Detektiv-Tipp:** 
- 192.168.1.100 → Verdächtiger ist im Firmennetzwerk
- 45.142.212.98 → Verdächtiger kommt von außen (Internet)

### 1.4 Digitale "Türen": Netzwerk-Ports

**Port = "Türnummer" für verschiedene Dienste**
- **Port 80:** Normale Websites (HTTP)
- **Port 443:** Verschlüsselte Websites (HTTPS)  
- **Port 3389:** Remote Desktop (Fernzugriff auf Computer) **WARNUNG: Beliebtes Angriffsziel!**
- **Port 22:** SSH (sichere Fernverbindung)
- **Port 445:** SMB (Datei-Freigaben in Windows)

### 1.5 Digitale Beweismittel: Log-Dateien

**Log-Datei = automatisches "Tatort-Protokoll" des Computers**

**Typischer Aufbau:**
```
[Datum Zeit] [Typ] [Quelle] [Was ist passiert]
2024-06-23 14:30:15 ERROR Firewall Verbindung blockiert
```

**Ihre wichtigsten Beweisquellen:**
- **Firewall-Logs:** Welche Verbindungen erlaubt/blockiert wurden
- **Antivirus-Logs:** Gefundene Schadsoftware
- **Windows Event-Logs:** Anmeldungen und Systemereignisse

**Windows Event-IDs (Ihr Detektiv-Code-Buch):**
- **4624:** Erfolgreiche Anmeldung
- **4625:** Fehlgeschlagene Anmeldung **VERDÄCHTIG bei mehrmaligem Auftreten!**
- **4672:** Administrator-Rechte wurden verwendet **HOHES RISIKO!**
- **4648:** Anmeldung mit gespeicherten Anmeldedaten

**Anmeldungsarten (Logon Types):**
- **Type 2:** Lokale Anmeldung (direkt am Computer)
- **Type 3:** Netzwerk-Anmeldung (z.B. Datei-Zugriff)
- **Type 10:** Remote Desktop Anmeldung

### 1.6 Ihre Ermittlungs-Werkzeuge

**Für IP-Recherche:**
- **AbuseIPDB.com** → Prüft ob IP-Adresse bereits bei Verbrechen gesichtet wurde
- **VirusTotal.com** → Prüft Dateien und IPs auf bekannte Malware

---

## AKTUELLER THREAT INTELLIGENCE REPORT

**SICHERHEITSWARNUNG - Aktuelle Bedrohungslage**

*Unsere Cyber-Geheimdienste melden folgende verdächtige Aktivitäten:*

### Bekannte Angreifer-Infrastruktur
```
VERDÄCHTIGE IP-ADRESSEN (Letzte 48h):
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
KRITISCH: 198.51.100.45    → Command & Control Server
KRITISCH: 45.142.212.98    → Malware Distribution Point
KRITISCH: 203.0.113.22     → Data Exfiltration Hub
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

### Aktive Malware-Kampagnen
```
RANSOMWARE-FAMILIE: Cerber-Variante
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Tarnmethoden:
   • Fake-PDFs: "rechnung.pdf.exe"
   • Fake-Angebote: "angebot.pdf.exe"
   • Fake-Dokumente: "wichtig.pdf.exe"

Verschlüsselungsmuster:
   • Word-Dateien (.docx) → .locked
   • Excel-Dateien (.xlsx) → .locked
   • Prozess: svchost.exe (Tarnung)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

### Bekannte Angriffsmuster
```
PHISHING-WELLE: "Rechnungs-E-Mails"
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Betreff oft: "Ihre Rechnung Mai 2024"
Anhang: PDF-Dateien mit .exe-Endung
Ziel: Credential Theft + Ransomware
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

**Ihre Aufgabe als Ermittler:** Halten Sie diese Informationen bereit - Sie werden sie für Ihre Analyse brauchen!

---

## TEIL 2: ERMITTLER-QUALIFIKATIONSTEST

**Aufgabe 2.1:** Beweisen Sie Ihr Grundwissen (in ganzen Sätzen):

**Frage 1:** Ein Verdächtiger nutzt IP 192.168.1.50, ein anderer 203.0.113.45. Welcher Täter ist ein "Insider" und welcher kommt von außen?

**Frage 2:** In den Überwachungsprotokollen sehen Sie Verbindungsversuche zu Port 3389. Was versucht der Angreifer zu tun?

**Frage 3:** Sie entdecken Event ID 4625 fünfmal hintereinander in 2 Minuten. Was ist hier los?

**Frage 4:** Ein Benutzer hat eine Datei namens "vertrag.pdf.exe" heruntergeladen. Warum klingeln bei Ihnen die Alarmglocken?

---

## TEIL 3: DER GROSSE FALL - "OPERATION BLACKOUT"

### Der mysteriöse Fall

**Tatort:** MustermannGmbH (50 Mitarbeiter, Softwareentwicklung)  
**Tatzeitpunkt:** Montag, 9:00 Uhr  
**Erste Meldungen:**

> *"Hilfe! Meine Word-Dateien sind alle kaputt!"* - Maria Mueller, Buchhaltung
> 
> *"Auf meinem Bildschirm steht eine komische Nachricht mit Bitcoin!"* - Stefan Schmidt, Marketing
> 
> *"Das Internet funktioniert nicht mehr richtig!"* - Diverse Mitarbeiter

**Ihre Mission:** Rekonstruieren Sie den Tathergang anhand der digitalen Spuren!

### 3.1 Beweismittel A: Firewall-Überwachungsprotokolle

**Firmen-Netzwerk:** 192.168.1.x (alle Verdächtigen und Opfer)

```
Zeit: 2024-06-23 08:15:22
Regel: EXTERNAL_BLOCK
Von: 198.51.100.45:80 → Nach: 192.168.1.105:49152
Aktion: BLOCKIERT

Zeit: 2024-06-23 08:16:45  
Regel: OUTBOUND_ALLOW
Von: 192.168.1.105:49153 → Nach: 198.51.100.45:443
Aktion: ERLAUBT
Übertragung: 2.8 MB

Zeit: 2024-06-23 08:35:12
Regel: EXTERNAL_BLOCK  
Von: 45.142.212.98:443 → Nach: 192.168.1.106:49154
Aktion: BLOCKIERT

Zeit: 2024-06-23 08:36:30
Regel: OUTBOUND_ALLOW
Von: 192.168.1.106:49155 → Nach: 198.51.100.45:443  
Aktion: ERLAUBT
Übertragung: 1.2 MB
```

### 3.2 Beweismittel B: Antivirus-Detektionsprotokolle

```
Zeit: 2024-06-23 08:17:15
Computer: PC-Buchhaltung (192.168.1.105)  
Datei: C:\Users\mmueller\Downloads\Rechnung_Mai.pdf.exe
Bedrohung: Trojan.Ransomware.Cerber
Aktion: Datei gelöscht

Zeit: 2024-06-23 08:18:00
Computer: PC-Buchhaltung (192.168.1.105)
Prozess: svchost.exe
Aktivität: Massenverschlüsselung erkannt
Betroffene Dateien: C:\Users\mmueller\Documents\*.docx → *.locked
Aktion: Prozess beendet, aber Schaden bereits entstanden

Zeit: 2024-06-23 08:37:45
Computer: PC-Marketing (192.168.1.106)
Datei: C:\Users\sschmidt\Desktop\Angebot.pdf.exe  
Bedrohung: Trojan.Ransomware.Cerber
Aktion: Datei gelöscht

Zeit: 2024-06-23 08:38:30
Computer: PC-Marketing (192.168.1.106)
Aktivität: Verschlüsselungsversuch erkannt
Betroffene Dateien: C:\Users\sschmidt\Documents\*.xlsx → *.locked
Aktion: Prozess gestoppt, 15 Dateien bereits verschlüsselt
```

### 3.3 Beweismittel C: Windows Anmeldungsprotokolle

**Computer: PC-Buchhaltung (192.168.1.105)**
```
Event ID: 4625 - Anmeldefehler
Zeit: 2024-06-23 08:10:15
Benutzer: Administrator  
Quelle: 192.168.1.105
Grund: Falsches Passwort

Event ID: 4625 - Anmeldefehler
Zeit: 2024-06-23 08:10:45
Benutzer: Administrator
Quelle: 192.168.1.105  
Grund: Falsches Passwort

Event ID: 4625 - Anmeldefehler
Zeit: 2024-06-23 08:11:15
Benutzer: Administrator
Quelle: 192.168.1.105
Grund: Falsches Passwort

Event ID: 4624 - Erfolgreiche Anmeldung
Zeit: 2024-06-23 08:11:45
Benutzer: Administrator
Quelle: 192.168.1.105
Anmeldungstyp: 2 (Lokal)

Event ID: 4672 - Besondere Rechte zugewiesen
Zeit: 2024-06-23 08:12:00
Benutzer: Administrator
Rechte: SeBackupPrivilege, SeRestorePrivilege
```

**Computer: PC-Marketing (192.168.1.106)**
```
Event ID: 4624 - Erfolgreiche Anmeldung
Zeit: 2024-06-23 08:30:22
Benutzer: sschmidt
Quelle: 192.168.1.106
Anmeldungstyp: 2 (Lokal)

Event ID: 4648 - Anmeldung mit gespeicherten Anmeldedaten
Zeit: 2024-06-23 08:35:45
Benutzer: sschmidt
Ziel-Server: 192.168.1.105
Ziel-Benutzer: Administrator
```

---

## IHRE ERMITTLUNGSAUFGABEN

### Aufgabe 3.1: Tatrekonstruktion - Timeline erstellen

**Detektiv-Aufgabe:** Sortieren Sie ALLE Spuren chronologisch und rekonstruieren Sie den Tathergang!

**Format:**
```
08:10:15 - [Ort] [Was geschah genau]
08:10:45 - [Ort] [Was geschah genau]
...
```

**Ermittler-Tipp:** Achten Sie auf die Uhrzeiten - jede Minute kann entscheidend sein!

### Aufgabe 3.2: Täteranalyse 

**Frage 1:** Welcher Computer war das erste Opfer? Präsentieren Sie Ihre Beweise!

**Frage 2:** Wie ist der Täter wahrscheinlich eingedrungen? (Ihre Theorie)
a) Durch Passwort-Knacken (Brute Force)
b) Durch eine Phishing-E-Mail  
c) Durch eine Software-Sicherheitslücke
d) Durch einen infizierten USB-Stick

**Frage 3:** Beschreiben Sie die "Vorbereitungsphase" des Angriffs (08:10:15 bis 08:12:00) auf PC-Buchhaltung. Was hat der Täter getan?

**Frage 4:** Erklären Sie die "Ausbreitungsstrategie": Wie hat sich der Angriff auf PC-Marketing übertragen?

### Aufgabe 3.3: Täter-IP-Recherche 

**Cyberpol-Recherche:** Nutzen Sie **AbuseIPDB.com** und überprüfen Sie diese verdächtigen IPs:
- 198.51.100.45
- 45.142.212.98
- 102.209.217.173

**Frage 1:** Sind diese IP-Adressen bereits bei anderen Cyber-Verbrechen aufgefallen? (Ja/Nein für jede)

**Frage 2:** Falls ja, welche kriminellen Aktivitäten werden in den Berichten erwähnt?

**Ermittler-Hinweis:** Vergleichen Sie Ihre Funde mit dem Threat Intelligence Report!

### Aufgabe 3.4: Schadensgutachten

**Betroffene Tatorte:** 
- PC-Buchhaltung (192.168.1.105) - Maria Mueller
- PC-Marketing (192.168.1.106) - Stefan Schmidt

**Frage 1:** Welche Dateitypen wurden durch die Ransomware verschlüsselt?

**Frage 2:** Wo war der Schaden schwerwiegender? Begründen Sie Ihr Gutachten.

**Frage 3:** Erstellen Sie eine Opferliste (3 betroffene Personen):
1. _____________
2. _____________  
3. _____________

### Aufgabe 3.5: Notfall-Einsatzplan

**Situation:** Es ist jetzt 9:00 Uhr Montag. Sie sind der erste Ermittler vor Ort. Was ist zu tun?

**Priorisieren Sie Ihre Sofortmaßnahmen (1 = höchste Priorität):**

___ Betroffene Computer vom Netzwerk isolieren
___ Geschäftsführung über den Vorfall informieren  
___ Alle Passwörter sofort ändern
___ Backup-Systeme auf Integrität prüfen
___ Polizei/Cyber-Crime-Unit verständigen
___ Vollständigen Antivirus-Scan durchführen
___ Betroffene Mitarbeiter über den Vorfall aufklären

**Ihre Begründung:** Erklären Sie Ihre Strategie (2-3 Sätze).

### Aufgabe 3.6: Krisenkommunikation 

**Verfassen Sie einen Notfall-Bericht an die Geschäftsführung (max. 100 Wörter):**

**Betreff:** ________________

**Text:**
_________________________________
_________________________________
_________________________________

**Profi-Tipp:** Bleiben Sie sachlich, aber dringend!

### Aufgabe 3.7: Präventions-Strategie

**Blick in die Zukunft:** Welche 5 Schutzmaßnahmen hätten diesen Cyber-Angriff verhindert?

1. ________________________________
2. ________________________________  
3. ________________________________
4. ________________________________
5. ________________________________

---

## TEIL 4: ERMITTLUNGS-REFLEXION

### Aufgabe 4.1: Lessons Learned - Was haben Sie gelernt? 

**Ihre wichtigsten Erkenntnisse als Cyber-Detektiv:**

**Drei "Aha!"-Momente:**
1. ________________________________
2. ________________________________
3. ________________________________

**Was würden Sie ab sofort in Ihrem Leben anders machen?**
_________________________________
_________________________________

---

## ABGABE IHRES ERMITTLUNGSBERICHTS

**Dateiname:** `Nachname_Vorname_CyberDetektiv_Bericht.txt`

**Struktur Ihres Berichts:**
```
CYBER-DETEKTIV ERMITTLUNGSBERICHT
Name: [Ihr Name]
Fall: Operation Blackout - MustermannGmbH
Datum: [Datum der Bearbeitung]

==========================================
QUALIFIKATIONSTEST
==========================================

Frage 1: [Ihre Analyse]
Frage 2: [Ihre Analyse]
Frage 3: [Ihre Analyse]  
Frage 4: [Ihre Analyse]

==========================================
TATORT-ANALYSE
==========================================

Fall 3.1: Tatrekonstruktion
[Ihre chronologische Timeline]

Fall 3.2: Täteranalyse
Frage 1: [Ihre Beweise]
Frage 2: [a/b/c/d]
Frage 3: [Ihre Analyse]
Frage 4: [Ihre Theorie]

[...Fortsetzung für alle Ermittlungen...]

==========================================
ERMITTLUNGS-REFLEXION
==========================================

[Ihre Lessons Learned]

==========================================
ERMITTLER-SIGNATUR
==========================================
Dieser Bericht wurde erstellt von: [Ihr Name]
Cyber-Detektiv in Ausbildung
```
