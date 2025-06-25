# Windows Firewall - Grundlagen und praktische Übungen

Heute lernt ihr, wie Windows Firewalls funktionieren und wie ihr euer System sicher analysieren könnt - ohne etwas zu riskieren! Wir schauen nur, ändern nichts und  lernen dabei eine Menge über Cybersecurity.

## Was ihr heute lernen werdet

In diesem Workshop entwickelt ihr ein fundiertes Verständnis für Windows Firewalls und deren praktische Anwendung. Ihr lernt zunächst die theoretischen Grundlagen kennen: Was ist eine Firewall überhaupt, warum braucht ihr sie und  wie funktioniert das System in Windows? Dabei erkläre ich euch das Konzept von Ports, Firewall-Regeln und den verschiedenen Sicherheitsprofilen.

Anschließend werdet ihr diese Theorie in die Praxis umsetzen. Ihr lernt wichtige Windows-Befehle kennen, mit denen ihr euer eigenes System analysieren könnt. Dabei findet ihr heraus, welche Programme gerade Netzwerkverbindungen nutzen, welche Ports offen sind und  wie ihr Firewall-Logs richtig interpretiert. Am Ende könnt ihr selbstständig einschätzen, ob euer System sicher konfiguriert ist.

Das Besondere: Ihr macht das alles komplett sicher, ohne euer System zu gefährden. Wir nutzen nur die Tools, die Windows bereits mitbringt und  ändern nie etwas an eurer Konfiguration. Ihr lernt durch Beobachten und Analysieren - das ist oft viel lehrreicher als blindes Konfigurieren.

---

# TEIL 1: THEORIE - ALLES WAS IHR WISSEN MÜSST

## Was ist eine Firewall und warum brauche ich sie?

Eine Firewall ist im Grunde ein intelligenter Wächter für euren Computer. Stellt euch vor, euer Computer wäre ein Haus mit vielen verschiedenen Türen. Jede dieser Türen wird in der Computerwelt als "Port" bezeichnet und  es gibt insgesamt 65.535 solcher Türen. Eine Firewall funktioniert wie ein sehr aufmerksamer Türsteher, der bei jeder einzelnen Tür steht und genau kontrolliert, wer hinein- und herausgehen darf.

Ohne eine Firewall wären alle diese Türen unkontrolliert - jeder könnte einfach hineispazieren und sich nehmen, was er möchte. Das wäre etwa so, als würdet ihr euer Haus verlassen und alle Türen sperrangelweit offen stehen lassen. Hacker, Viren und andere schädliche Programme könnten ungehindert auf euren Computer zugreifen, eure Daten stehlen, euch ausspionieren oder euren Computer sogar für weitere Angriffe missbrauchen.

**Beispiel aus der Praxis:** Stellt euch vor, ihr habt einen Webserver auf eurem Computer laufen (Port 80), aber keine Firewall. Jeder im Internet könnte auf diesen Server zugreifen, eure Dateien durchstöbern oder sogar schädlichen Code hochladen.

Die Windows Firewall überwacht dabei zwei Richtungen des Datenverkehrs:
- **Eingehender Traffic:** Alle Verbindungsversuche von außen zu eurem Computer
- **Ausgehender Traffic:** Alle Verbindungen, die Programme auf eurem Computer zu anderen Computern aufbauen wollen

## Wie funktioniert die Windows Firewall?

Die Windows Firewall arbeitet nach einem sehr logischen System, das man sich wie eine intelligente Ampel vorstellen kann. Für jede Netzwerkverbindung entscheidet sie: Rot bedeutet stoppen und blockieren, Gelb bedeutet nachfragen und den Benutzer entscheiden lassen und  Grün bedeutet durchlassen und erlauben.

Das Besondere an der Windows Firewall ist, dass sie nicht nur stupide Regeln befolgt, sondern intelligent mitdenkt. Sie merkt sich zum Beispiel, wenn ihr eine Webseite aufruft und  erlaubt dann automatisch die Antwort vom Webserver. Das nennt man "Stateful Inspection" - die Firewall versteht den Zusammenhang zwischen eurer Anfrage und der Antwort.

**Beispiel für Stateful Inspection:**
```
1. Ihr ruft google.de auf
2. Firewall merkt: Ausgehende Anfrage zu Google-Server erlaubt
3. Google antwortet zurück
4. Firewall denkt: "Das ist die Antwort auf die Anfrage" → Erlaubt
```

Windows nutzt dabei drei verschiedene Sicherheitsprofile, die automatisch je nach Netzwerkumgebung aktiviert werden:

**Private Netzwerke (Zuhause):**
- Vertraut anderen Geräten im Netzwerk
- Datei- und Druckerfreigabe funktioniert
- Weniger restriktive Regeln

**Domänen-Netzwerke (Firma):**
- IT-Administrator gibt Sicherheitsrichtlinien vor
- Zentral verwaltete Firewall-Regeln
- Mittlere Sicherheitsstufe

**Öffentliche Netzwerke (Café, Hotel):**
- Strengste Sicherheitseinstellungen
- Blockiert fast alle eingehenden Verbindungen
- Keine Freigaben an andere Geräte

## Was sind Ports und warum sind sie wichtig?

Ports sind ein fundamentales Konzept, das ihr unbedingt verstehen müsst. Stellt euch vor, euer Computer wäre ein riesiges Bürogebäude mit 65.535 verschiedenen Zimmern. Jedes Zimmer hat eine eindeutige Nummer - das ist die Port-Nummer. Verschiedene Programme und Dienste "wohnen" in verschiedenen Zimmern und haben feste Adressen.

**Beispiel einer typischen Webseiten-Anfrage:**
```
Euer Browser → Port 443 (HTTPS) → google.de
google.de → Port 443 → Euer Browser (Antwort)
```

### Die wichtigsten Standard-Ports:

**Web-Traffic:**
```
Port 80   = HTTP (unverschlüsselte Webseiten)
Port 443  = HTTPS (verschlüsselte Webseiten)
```

**E-Mail:**
```
Port 25   = SMTP (E-Mails versenden)
Port 110  = POP3 (E-Mails abholen, alte Methode)
Port 143  = IMAP (E-Mails abholen, moderne Methode)
Port 993  = IMAPS (verschlüsseltes IMAP)
Port 995  = POP3S (verschlüsseltes POP3)
```

**System-Dienste:**
```
Port 53   = DNS (Adress-Auflösung)
Port 22   = SSH (sichere Fernwartung)
Port 3389 = RDP (Windows Remote Desktop)
Port 445  = SMB (Windows Dateifreigabe)
```

Diese Standardisierung ist extrem wichtig, weil sich alle Computer im Internet daran halten. Wenn ihr www.google.de aufruft, weiß euer Browser automatisch, dass er an Port 443 anklopfen muss. Der Google-Server weiß, dass er in Zimmer 443 auf Anfragen warten muss.

**Praktisches Beispiel - Was passiert beim Surfen:**
```
1. Ihr tippt "https://github.com" in den Browser
2. Browser kontaktiert DNS-Server über Port 53: "Wie ist die IP von github.com?"
3. DNS antwortet: "140.82.121.4"
4. Browser verbindet sich zu 140.82.121.4:443
5. Webseite wird geladen
```

### Gefährliche Ports - Aufpassen bei:

```
Port 21   = FTP (unverschlüsselte Dateiübertragung)
Port 23   = Telnet (unverschlüsselte Fernwartung)
Port 135  = RPC (oft von Malware missbraucht)
Port 1337 = Häufig von Hackern genutzt
Port 4444 = Typischer Malware-Port
Port 31337 = "Elite" Hacker-Port
```

**Warum sind diese gefährlich?** Manche Hacker nutzen ganz bewusst ungewöhnliche Port-Nummern wie 1337, 4444 oder 31337, weil sie hoffen, dass diese weniger auffallen. Wenn ihr diese Ports auf eurem System findet, solltet ihr genauer hinschauen.

## Firewall-Regeln verstehen

Firewall-Regeln sind das Herzstück des gesamten Systems und  ihr müsst unbedingt verstehen, wie sie funktionieren. Das wichtigste Prinzip ist: **Die erste passende Regel gewinnt immer**. Windows arbeitet die Regelliste von oben nach unten ab und wendet die erste Regel an, die auf eine Verbindung passt.

**Beispiel für Regel-Reihenfolge:**
```
Regel 1: BLOCKIERE alle Verbindungen von 192.168.1.5
Regel 2: ERLAUBE alle Verbindungen von 192.168.1.0/24
Regel 3: BLOCKIERE alles andere

Ergebnis: 192.168.1.5 wird blockiert (Regel 1 gewinnt)
         192.168.1.10 wird erlaubt (Regel 2 greift)
         203.0.113.5 wird blockiert (Regel 3 greift)
```

### Anatomie einer Firewall-Regel:

Jede Firewall-Regel besteht aus mehreren Komponenten:

```
RICHTUNG:   Eingehend / Ausgehend
AKTION:     Zulassen / Blockieren  
PROGRAMM:   firefox.exe / Alle Programme / Bestimmte .exe
PROTOKOLL:  TCP / UDP / Beide
PORT:       443 / 8000-8080 / Alle
IP-ADRESSE: 192.168.1.100 / 192.168.1.0/24 / Alle
```

**Beispiel einer vollständigen Regel:**
```
Richtung: Ausgehend
Aktion: Zulassen
Programm: firefox.exe
Protokoll: TCP
Port: 443
IP-Adresse: Alle
```
Diese Regel erlaubt Firefox, HTTPS-Verbindungen (Port 443) zu allen IP-Adressen aufzubauen.

### Die Default Deny Philosophie:

Moderne Firewalls arbeiten nach dem Prinzip "Default Deny":

```
SCHLECHT (Default Allow):
1. Erlaube alles
2. Blockiere Bekannte Gefahren
→ Problem: Neue Gefahren kommen durch

GUT (Default Deny):
1. Blockiere alles
2. Erlaube nur Nötiges
→ Vorteil: Unbekannte Gefahren bleiben draußen
```

## Windows Firewall Tools - Diese Befehle werden wir nutzen

Um mit der Windows Firewall zu arbeiten und euer System zu analysieren, nutzen wir verschiedene eingebaute Tools. Diese Tools sind mächtig, kostenlos und bereits in Windows integriert.

### 1. netstat - Der Netzwerk-Klassiker

Der `netstat`-Befehl ist das wichtigste Tool für Netzwerk-Diagnose. Er zeigt euch alle aktiven Verbindungen eures Computers an.

**Grundsyntax:**
```cmd
netstat [Optionen]
```

**Die wichtigsten Optionen:**
- `-a` = Zeigt ALLE Verbindungen (auch wartende)
- `-n` = Zeigt Zahlen statt Namen (schneller und eindeutiger)  
- `-b` = Zeigt welches Programm die Verbindung nutzt

**Beispiel-Befehl:**
```cmd
netstat -an
```

**Beispiel-Ausgabe:**
```
TCP    192.168.1.100:3389    192.168.1.50:12345    ESTABLISHED
TCP    0.0.0.0:135          0.0.0.0:0             LISTENING
UDP    0.0.0.0:53           *:*                   
```

**Was bedeutet das?**
- Zeile 1: Aktive RDP-Verbindung zwischen zwei Computern
- Zeile 2: RPC-Dienst wartet auf Verbindungen von überall
- Zeile 3: DNS-Dienst ist aktiv (UDP)

### 2. Verbindungszustände verstehen

```
LISTENING    = Port ist offen, wartet auf Verbindungen
ESTABLISHED  = Aktive Verbindung läuft gerade
TIME_WAIT    = Verbindung wird gerade beendet
CLOSE_WAIT   = Verbindung wurde vom anderen Ende geschlossen
SYN_SENT     = Verbindungsaufbau läuft noch
```

**Praktisches Beispiel:**
```cmd
netstat -an | findstr :443
```
Dieser Befehl zeigt nur HTTPS-Verbindungen (Port 443). Das `findstr` ist Windows' Version von grep und filtert Zeilen nach bestimmten Texten.

### 3. PowerShell - Die moderne Alternative

PowerShell bietet modernere und flexiblere Netzwerk-Befehle:

```powershell
Get-NetTCPConnection
```

**Beispiel - Nur wartende Verbindungen:**
```powershell
Get-NetTCPConnection | Where-Object {$_.State -eq "Listen"}
```

**Beispiel - Nur externe Verbindungen:**
```powershell
Get-NetTCPConnection | Where-Object {$_.RemoteAddress -ne "127.0.0.1"}
```

### 4. netsh - Windows Netzwerk-Konfiguration

`netsh` ist das Schweizer Taschenmesser für Windows-Netzwerkkonfiguration:

**Firewall-Status prüfen:**
```cmd
netsh advfirewall show allprofiles
```

**Alle Firewall-Regeln anzeigen:**
```cmd
netsh advfirewall firewall show rule name=all
```

**Beispiel-Ausgabe des Firewall-Status:**
```
Domänenprofil-Einstellungen:
Status                         Ein
Firewall-Richtlinie           BlockInbound,AllowOutbound

Privates Profil-Einstellungen:
Status                         Ein  
Firewall-Richtlinie           BlockInbound,AllowOutbound

Öffentliches Profil-Einstellungen:
Status                         Ein
Firewall-Richtlinie           BlockInbound,AllowOutbound
```

## Event Viewer - Windows Protokolle verstehen

Der Event Viewer ist Windows' zentrale Anlaufstelle für alle Systemprotokolle. Hier protokolliert Windows alle wichtigen Ereignisse, einschließlich Firewall-Aktivitäten.

**Event Viewer öffnen:**
```
Windows + R → "eventvwr.msc" → Enter
```

### Wichtige Firewall Event-IDs:

```
Event ID 5156 = Verbindung wurde ERLAUBT
Event ID 5157 = Verbindung wurde BLOCKIERT  
Event ID 2004 = Firewall-Regel wurde hinzugefügt
Event ID 2006 = Firewall-Regel wurde gelöscht
```

**Beispiel eines Event 5157 (blockiert):**
```
Quelle: 203.0.113.5
Ziel: 192.168.1.100
Port: 3389
Protokoll: TCP
Grund: Keine passende Regel gefunden
```

Das würde bedeuten: Jemand von außen (203.0.113.5) hat versucht, sich über Remote Desktop (Port 3389) an euren Computer zu verbinden, aber die Firewall hat es blockiert.

## Typische Windows-Dienste und ihre Ports

Windows bringt viele Netzwerkdienste mit. Hier ist eine Übersicht, welche normal und welche potenziell problematisch sind:

### Normale Windows-Dienste:

```
svchost.exe (Port 135)  = RPC-Dienst (normal)
System (Port 445)       = Windows-Dateifreigabe
lsass.exe (variabel)    = Windows-Authentifizierung
spoolsv.exe (variabel)  = Druckdienst
```

### Browser-Traffic (normal):

```
chrome.exe (Port 80,443)    = Web-Browsing
firefox.exe (Port 80,443)   = Web-Browsing
msedge.exe (Port 80,443)    = Web-Browsing
```

### Verdächtige Muster:

```
unknown.exe (Port 1337)     = Potentielle Malware
win32.exe (Port 4444)       = Fake Windows-Name
system32.exe (beliebig)     = Tarnung als System-Datei
```

**Warum sind diese verdächtig?** Echte Windows-Systemdateien haben sehr spezifische Namen und Speicherorte. Programme, die versuchen, sich als Windows-Komponenten zu tarnen, sind oft Malware.

---

# TEIL 2: PRAKTISCHE ÜBUNGEN

## Aufgabe 1: "Mein System kennenlernen" (15 Min)

In dieser ersten Aufgabe werdet ihr eine grundlegende Bestandsaufnahme eures Systems machen. Das Ziel ist es, herauszufinden, welche Netzwerkverbindungen gerade aktiv sind und welche Ports euer Computer überwacht. Das ist der erste Schritt jeder Sicherheitsanalyse - man muss verstehen, was das System gerade tut, bevor man beurteilen kann, ob es sicher ist.

### Schritt 1: Kommandozeile öffnen

```
1. Windows-Taste drücken
2. "cmd" tippen
3. Enter drücken
```

### Schritt 2: Erste Netzwerk-Analyse

Führt diesen Befehl aus:

```cmd
netstat -an
```

**Was passiert jetzt?** Windows zeigt euch alle aktiven Netzwerkverbindungen an. Die Ausgabe kann auf den ersten Blick überwältigend wirken, aber lasst euch nicht entmutigen.

**Beispiel der Ausgabe:**
```
Aktive Verbindungen

  Proto  Lokale Adresse         Remoteadresse          Status
  TCP    0.0.0.0:135           0.0.0.0:0              LISTENING
  TCP    0.0.0.0:445           0.0.0.0:0              LISTENING
  TCP    192.168.1.100:443     172.217.16.68:80      ESTABLISHED
  TCP    192.168.1.100:3389    0.0.0.0:0              LISTENING
```

### Eure Detective-Aufgaben:

**1. Findet 3 Zeilen mit "LISTENING"**

LISTENING bedeutet: "Mein Computer hat eine Tür offen und wartet auf Besucher."

**Beispiel für eure Notizen:**
```
Port 135: _____ (RPC-Dienst - normal für Windows)
Port 445: _____ (Dateifreigabe - normal im Heimnetzwerk)
Port ___: _____ (Was ist das?)
```

**2. Findet 3 Zeilen mit "ESTABLISHED"**

ESTABLISHED bedeutet: "Mein Computer führt gerade ein Gespräch mit einem anderen Computer."

**Beispiel-Analyse:**
```
Verbindung 1:
Mein Port: _____ → Remote IP: _____ → Was macht das?
(Tipp: Port 443 = HTTPS, Port 80 = HTTP)

Verbindung 2:
Mein Port: _____ → Remote IP: _____ → Was macht das?

Verbindung 3:  
Mein Port: _____ → Remote IP: _____ → Was macht das?
```

**3. Spezial-Analyse: HTTPS-Traffic zählen**

```cmd
netstat -an | findstr :443
```

**Was zeigt dieser Befehl?** Nur Verbindungen über Port 443 (sicherer Web-Traffic).

**Eure Aufgabe:** Zählt die Zeilen. Jede Zeile = eine sichere Webverbindung.

Anzahl: _____ HTTPS-Verbindungen

### Was ist normal? Was ist verdächtig?

**Normal:**
- Port 135 (RPC): Standard Windows-Dienst
- Port 445 (SMB): Dateifreigabe, normal im Heimnetzwerk
- Port 80/443: Web-Traffic von Browsern
- Mehrere ESTABLISHED zu verschiedenen IPs: Ihr surft gerade

**Aufmerksam werden bei:**
- Unbekannte hohe Port-Nummern (>10000)
- Ports 21, 23: Unsichere Protokolle
- Port 3389 (RDP): Nur wenn ihr Remote Desktop nutzt
- Viele Verbindungen zu derselben fremden IP

## Aufgabe 2: "Firewall-Status checken" (10 Min)

Jetzt prüfen wir, ob eure Firewall richtig arbeitet. Eine falsch konfigurierte Firewall ist wie ein Türsteher, der schläft - sieht sicher aus, schützt aber nicht.

### Schritt 1: PowerShell als Administrator öffnen

```
1. Windows-Taste drücken
2. "PowerShell" tippen  
3. Rechtsklick auf "Windows PowerShell"
4. "Als Administrator ausführen"
5. Eventuelle UAC-Meldung mit "Ja" bestätigen
```

**Warum Administrator-Rechte?** Firewall-Informationen sind sicherheitskritisch, deshalb braucht ihr erweiterte Berechtigungen.

### Schritt 2: Firewall-Status abfragen

```cmd
netsh advfirewall show allprofiles
```

**Was macht dieser Befehl?** Er zeigt den detaillierten Status aller drei Firewall-Profile an.

### Eure Analyse-Aufgabe:

**Beispiel-Ausgabe und was ihr dokumentieren sollt:**

```
Domänenprofil-Einstellungen:
----------------------------------------------------------------------
Status                         Ein
Firewall-Richtlinie            BlockInbound,AllowOutbound

Privates Profil-Einstellungen:
----------------------------------------------------------------------  
Status                         Ein
Firewall-Richtlinie            BlockInbound,AllowOutbound

Öffentliches Profil-Einstellungen:
----------------------------------------------------------------------
Status                         Ein
Firewall-Richtlinie            BlockInbound,AllowOutbound
```

**Füllt diese Tabelle aus:**

| Profil | Status (Ein/Aus) | Eingehend | Ausgehend |
|--------|------------------|-----------|-----------|
| Domäne | _____ | _____ | _____ |
| Privat | _____ | _____ | _____ |  
| Öffentlich | _____ | _____ | _____ |

### Sicherheits-Bewertung:

**Optimal (10 Punkte):**
- Alle Profile: Status = Ein
- Eingehend: Block oder NotConfigured
- Ausgehend: Allow oder NotConfigured

**Problematisch (0 Punkte):**
- Ein oder mehrere Profile: Status = Aus
- Eingehend: Allow (zu permissiv)

**Eure Bewertung:** _____ / 10 Punkte

## Aufgabe 3: "Port-Detektiv spielen" (20 Min)

Zeit für echte Detektivarbeit! Ihr findet heraus, welche Programme auf eurem System welche Netzwerkports nutzen. Das ist eine der wichtigsten Fähigkeiten in der IT-Sicherheit.

### Schritt 1: Programme und ihre Ports identifizieren

```cmd
netstat -anb
```

**Achtung:** Dieser Befehl braucht 30-60 Sekunden! Er muss für jede Verbindung herausfinden, welches Programm dahintersteckt.

**Beispiel der Ausgabe:**
```
TCP    0.0.0.0:135    0.0.0.0:0    LISTENING
 RpcSs
 [svchost.exe]

TCP    192.168.1.100:443    172.217.16.68:80    ESTABLISHED  
 [chrome.exe]

TCP    0.0.0.0:3389   0.0.0.0:0    LISTENING
 TermService
 [svchost.exe]
```

### Schritt 2: Moderne PowerShell-Alternative

```powershell
Get-NetTCPConnection | Where-Object {$_.State -eq "Listen"} | Select LocalAddress,LocalPort,OwningProcess
```

**Was macht dieser Befehl?**
- `Get-NetTCPConnection`: Holt alle TCP-Verbindungen
- `Where-Object {...}`: Filtert nur wartende Verbindungen
- `Select ...`: Zeigt nur interessante Spalten

### Detective-Missionen:

**Mission 1: Webserver-Fahndung**
```cmd
netstat -an | findstr :80
```

**Eure Analyse:**
- Seht ihr eine Zeile mit ":80" und "LISTENING"? Ja/Nein: _____
- Falls ja, IP-Adresse: _____ (0.0.0.0 = von überall erreichbar!)
- **Sicherheitsbewertung:** Webserver auf Desktop = verdächtig

**Mission 2: Remote Desktop-Check**  
```cmd
netstat -an | findstr :3389
```

**Eure Analyse:**
- Remote Desktop aktiv? Ja/Nein: _____
- **Sicherheitsfrage:** Nutzt ihr Remote Desktop bewusst?
- **Risiko:** RDP ist beliebtes Angriffsziel

**Mission 3: Dateifreigabe-Ermittlung**
```cmd
netstat -an | findstr :445
```

**Beispiel-Ausgabe:**
```
TCP    0.0.0.0:445         0.0.0.0:0              LISTENING
TCP    192.168.1.100:445   0.0.0.0:0              LISTENING
```

**Eure Analyse:**
- SMB-Freigabe aktiv? Ja/Nein: _____
- IP-Adresse: _____ 
- **Sicherheitstipp:** 0.0.0.0 = potentiell von außen erreichbar

**Mission 4: DNS-Dienst-Suche**
```cmd
netstat -an | findstr :53
```

**Normale Desktop-Computer sollten KEINEN DNS-Server betreiben!**

### Programm-Analyse mit Beispielen:

**Normale Programme:**
```
chrome.exe (Port 443) → Web-Browsing = OK
svchost.exe (Port 135) → Windows RPC = OK  
System (Port 445) → Windows-Freigabe = OK (nur lokal)
```

**Verdächtige Programme:**
```
unknown.exe (Port 1337) → Unbekannt = VERDÄCHTIG
win32.exe (Port 4444) → Fake Windows-Name = VERDÄCHTIG
server.exe (Port 80) → Warum Webserver? = PRÜFEN
```

### Sicherheits-Score:

**Bewertungskriterien:**
- Nur bekannte Windows-Dienste (Port 135, 445): +3 Punkte
- Nur normale Programme (Browser, bekannte Software): +3 Punkte
- Kein RDP ohne bewusste Nutzung: +2 Punkte
- Kein Webserver/DNS auf Desktop: +2 Punkte

**Euer Score:** _____ / 10 Punkte

## Aufgabe 4: "Firewall-Logs analysieren" (15 Min)

Jetzt werdet ihr zu digitalen Forensikern! Firewall-Logs sind wie ein Tagebuch eurer Netzwerkaktivitäten und können euch helfen, Angriffe oder ungewöhnliche Aktivitäten zu erkennen.

### Schritt 1: Event Viewer öffnen

```
Windows + R → "eventvwr.msc" → Enter
```

### Schritt 2: Zu Firewall-Logs navigieren

**Navigation im Event Viewer:**
```
1. Links: "Windows-Protokolle" erweitern
2. "Sicherheit" anklicken  
3. Rechts: "Aktuelles Protokoll filtern..."
4. Ereignis-IDs: "5156,5157" eingeben
5. OK klicken
```

**Was passiert?** Windows zeigt nur noch Firewall-Ereignisse an:
- **Event 5156:** Verbindung wurde erlaubt
- **Event 5157:** Verbindung wurde blockiert

### Eure Forensik-Analyse:

**1. Grundlegende Statistik:**

```
Gesamte Ereignisse: _____ Stück
Zeitraum: Von _____ bis _____
Event 5156 (erlaubt): _____ Stück  
Event 5157 (blockiert): _____ Stück
```

**2. Detailanalyse eines Events:**

Doppelklickt auf ein beliebiges Ereignis. Ihr seht Details wie:

**Beispiel Event 5156 (erlaubt):**
```
Zeitstempel: 2025-06-25 14:23:15
Quell-IP: 192.168.1.100 (euer Computer)
Ziel-IP: 172.217.16.68 (Google-Server)
Port: 443 (HTTPS)
Protokoll: TCP
Programm: chrome.exe
```

**Eure Dokumentation:**
```
Event-ID: _____
Quelle: _____  
Ziel: _____
Port: _____
Programm: _____
Normal/Verdächtig? _____
```

### Was bedeuten die Logs?

**Normale Event 5156-Beispiele:**
```
chrome.exe → google.de:443 = Web-Browsing ✓
outlook.exe → mail.microsoft.com:993 = E-Mail ✓  
svchost.exe → microsoft.com:80 = Windows Update ✓
```

**Verdächtige Muster in Event 5157:**
```
Viele Versuche von derselben IP = Angriff?
Ungewöhnliche Ports (1337, 4444) = Malware?
Unbekannte Programme = Untersuchen!
```

**Beispiel eines Port-Scan-Angriffs:**
```
Event 5157: 203.0.113.5 → Euer PC:22 (SSH)
Event 5157: 203.0.113.5 → Euer PC:23 (Telnet)  
Event 5157: 203.0.113.5 → Euer PC:80 (HTTP)
Event 5157: 203.0.113.5 → Euer PC:443 (HTTPS)
... (alle in kurzer Zeit)
```

**Das ist ein Port-Scan!** Jemand testet systematisch, welche Dienste auf eurem Computer laufen.

### Praktische Log-Bewertung:

**Gute Zeichen:**
- Viele Event 5157 (Firewall blockiert Angriffe)
- Event 5156 nur von bekannten Programmen
- Blockierte Verbindungen zu verdächtigen Ports

**Schlechte Zeichen:**
- Keine Event 5157 (Firewall blockiert nichts?)
- Event 5156 von unbekannten Programmen
- Erfolgreiche Verbindungen zu Malware-Ports

## Aufgabe 5: "Browser-Traffic verstehen" (10 Min)

Diese Aufgabe zeigt euch die Realität des modernen Internets: Eine einzige Webseite kann dutzende von Netzwerkverbindungen auslösen!

### Schritt 1: Baseline erstellen

```cmd
netstat -an | findstr :443 > vorher.txt
```

**Was macht das?**
- `netstat -an`: Alle Verbindungen anzeigen
- `findstr :443`: Nur HTTPS-Verbindungen filtern  
- `> vorher.txt`: Ergebnis in Datei speichern

**Beispiel der Baseline:**
```
TCP    192.168.1.100:443    172.217.16.68:54321    ESTABLISHED
TCP    192.168.1.100:443    151.101.1.140:54322    ESTABLISHED
```

### Schritt 2: Webseite besuchen

**Eure Mission:**
1. Browser öffnen
2. Neue, große Webseite besuchen (z.B. github.com, stackoverflow.com)
3. 10 Sekunden warten (wichtig!)

**Warum warten?** Moderne Webseiten laden Inhalte asynchron nach - auch nach dem ersten Laden können weitere Verbindungen entstehen.

### Schritt 3: Neue Verbindungen erfassen

```cmd
netstat -an | findstr :443 > nachher.txt
```

### Schritt 4: Vergleich in PowerShell

```powershell
$vorher = Get-Content vorher.txt
$nachher = Get-Content nachher.txt  
Compare-Object $vorher $nachher
```

**Beispiel-Ausgabe:**
```
InputObject                                          SideIndicator
-----------                                          -------------
TCP    192.168.1.100:443    140.82.121.4:443       =>
TCP    192.168.1.100:443    185.199.108.153:443    =>  
TCP    192.168.1.100:443    151.101.193.194:443    =>
```

**Was bedeutet "=>"?** Das sind neue Verbindungen durch euren Webseitenbesuch!

### Eure Analyse-Aufgaben:

**1. Verbindungs-Explosion:**
```
Neue HTTPS-Verbindungen: _____ Stück
(Zählt die Zeilen mit "=>")
```

**2. IP-Detektiv-Arbeit:**

Notiert 3 verschiedene IP-Adressen:
```
IP 1: _____ 
IP 2: _____
IP 3: _____
```

**3. Wem gehören diese IPs?**

```cmd
nslookup 140.82.121.4
```

**Beispiel-Ausgabe:**
```
Server:  dns.google
Address:  8.8.8.8

Name:    github.com
Address:  140.82.121.4
```

**Eure Ergebnisse:**
```
IP 1 gehört zu: _____
IP 2 gehört zu: _____  
IP 3 gehört zu: _____
```

### Was ihr entdeckt:

**Typische Verbindungen einer modernen Webseite:**
```
github.com (140.82.121.4) → Hauptwebseite
github.githubassets.com → CSS/JavaScript-Dateien
avatars.githubusercontent.com → Benutzer-Bilder
collector.github.com → Analytics/Tracking
```

**Warum so viele Verbindungen?**
- **CDNs:** Inhalte von geografisch nahen Servern
- **Analytics:** Google Analytics, etc.
- **Werbung:** Werbenetzwerke
- **Social Media:** Twitter/Facebook-Buttons
- **Security:** Zertifikat-Prüfungen

**Real-World-Beispiel GitHub.com:**
```
Hauptseite: 1 Verbindung
CSS/JS-Dateien: 3-5 Verbindungen  
Bilder/Avatare: 2-4 Verbindungen
Analytics: 1-2 Verbindungen
APIs: 1-3 Verbindungen
Total: 8-15 Verbindungen für EINE Webseite!
```

### Lerneffekt:

Das Internet ist nicht mehr "ein Computer spricht mit einem Server". Moderne Webseiten sind verteilte Anwendungen, die Inhalte von dutzenden verschiedenen Servern weltweit zusammenfügen. Das ist normal und sicher - aber ihr solltet verstehen, was dabei passiert.

## Aufgabe 6: "Sicherheits-Checkup" (15 Min)

In dieser finalen Mission führt ihr eine systematische Sicherheitsüberprüfung durch. Ihr kombiniert alle gelernten Techniken zu einem umfassenden Security-Audit eures Systems.

### Test 1: Externe Angriffsfläche minimieren

```powershell
Get-NetTCPConnection | Where-Object {$_.State -eq "Listen" -and $_.LocalAddress -eq "0.0.0.0"} | Select LocalPort,OwningProcess
```

**Was macht dieser Befehl?**
- Filtert nur Dienste, die auf **allen** Netzwerk-Interfaces lauschen
- `0.0.0.0` bedeutet: "Von überall erreichbar"
- Das ist eure potentielle Angriffsfläche!

**Beispiel-Ausgabe:**
```
LocalPort OwningProcess
--------- ------------
135       1028
445       4
3389      1234
```

### Gefährliche Ports-Checkliste:

**Sofort problematisch:**
```
□ Port 21 (FTP) → Unverschlüsselt, oft unsicher
□ Port 23 (Telnet) → Komplett unverschlüsselt  
□ Port 5900 (VNC) → Oft schwach geschützt
```

**Prüfen erforderlich:**
```
□ Port 135 (RPC) → Normal, aber Angriffsziel
□ Port 3389 (RDP) → Nur wenn bewusst aktiviert
□ Port 445 (SMB) → Sollte nur im LAN erreichbar sein
```

**Euer System:**
```
Gefährliche Ports gefunden: _____
Erklärbare Ports: _____  
Unbekannte Ports: _____
```

### Test 2: Programm-Legitimität prüfen

```cmd
netstat -anb | findstr /C:"[" | findstr /C:"]"
```

**Was zeigt das?** Alle Programme, die gerade Netzwerkverbindungen nutzen.

**Beispiel-Ausgabe:**
```
[chrome.exe]
[svchost.exe]  
[System]
[firefox.exe]
[unknown.exe]
```

### Programm-Bewertung:

**Vertrauenswürdige Programme:**
```
□ Browser (chrome.exe, firefox.exe, msedge.exe)
□ Windows-System (svchost.exe, System, lsass.exe)
□ Bekannte Software (steam.exe, discord.exe, etc.)
```

**Verdächtige Muster:**
```
□ Unbekannte .exe-Dateien
□ Programme mit Windows-ähnlichen Namen (win32.exe, system32.exe)
□ Programme in ungewöhnlichen Verzeichnissen
```

**Eure Programme-Liste:**
```
Bekannte Programme: _____
Unbekannte Programme: _____
Verdächtige Programme: _____
```

### Test 3: Aktive Bedrohungen erkennen

```powershell
Get-NetTCPConnection | Where-Object {$_.RemoteAddress -ne "127.0.0.1" -and $_.RemoteAddress -ne "0.0.0.0" -and $_.State -eq "Established"} | Select LocalPort,RemoteAddress,OwningProcess
```

**Was zeigt das?** Alle aktiven Verbindungen zu externen Servern.

**Beispiel-Analyse:**
```
LocalPort RemoteAddress    OwningProcess
--------- -------------    -------------
443       172.217.16.68    chrome.exe     → Google (normal)
443       140.82.121.4     chrome.exe     → GitHub (normal)  
1337      203.0.113.5      unknown.exe    → VERDÄCHTIG!
```

### Final-Score Berechnung:

**Netzwerk-Sicherheit (max. 4 Punkte):**
```
□ Keine gefährlichen Ports offen (+1)
□ Keine unbekannten Ports offen (+1)  
□ RDP nur wenn benötigt (+1)
□ SMB nicht extern erreichbar (+1)
```

**Programm-Sicherheit (max. 3 Punkte):**
```
□ Alle Programme bekannt (+1)
□ Keine verdächtige Programme (+1)
□ Nur vertrauenswürdige Netzwerk-Programme (+1)
```

**Firewall-Konfiguration (max. 3 Punkte):**
```
□ Alle Profile aktiviert (+1)
□ Default Deny konfiguriert (+1)  
□ Logs zeigen blockierte Angriffe (+1)
```

### Euer Sicherheits-Zeugnis:

```
Netzwerk-Sicherheit: _____ / 4 Punkte
Programm-Sicherheit: _____ / 3 Punkte  
Firewall-Konfiguration: _____ / 3 Punkte

GESAMT: _____ / 10 Punkte
```

**Bewertung:**
- **9-10 Punkte:** Exzellente Sicherheit! Weiter so!
- **7-8 Punkte:** Gute Grundsicherheit, kleine Optimierungen möglich
- **5-6 Punkte:** Solide Basis, einige Sicherheitslücken schließen
- **0-4 Punkte:** Erhöhtes Risiko, dringend nachbessern!

### Konkrete Verbesserungsempfehlungen:

**Bei niedrigem Score:**
```
□ Firewall in allen Profilen aktivieren
□ Ungenutzte Dienste deaktivieren (RDP, Telnet, FTP)
□ Unbekannte Programme untersuchen/entfernen
□ Windows Updates installieren
□ Regelmäßige Sicherheits-Checkups (monatlich)
```

**Profi-Tipps:**
```
□ Router-Firewall als zweite Schutzebene aktivieren
□ Regelmäßige Log-Analyse etablieren
□ Network Monitoring Tools installieren
□ Incident Response Plan erstellen
```

**Das ist euer Startpunkt für bessere Cybersecurity!** Diese Techniken könnt ihr regelmäßig anwenden, um euer System sicher zu halten.
