# 📝 Gruppenaufgabe: Planung eines Webprojekts

## 🎯 Ziel der Aufgabe

In dieser Aufgabe plant ihr in Gruppen die Struktur eines Webprojekts. Dabei geht es darum, eine sinnvolle **Technologiewahl**, eine **Architektur** und einen groben **Entwicklungsplan** zu erstellen.

Die Aufgabe simuliert den realen Prozess der Webentwicklung: Ihr setzt euch mit **Anforderungen, Zielgruppen, Technologie-Stack und den Phasen der Entwicklung** auseinander.

---

## 🔍 Aufgabenstellung

Ihr seid ein Entwicklerteam und sollt für einen Kunden eine **Webanwendung** planen. Dabei geht ihr folgendermaßen vor:

1️⃣ **Projektanforderungen analysieren**:  
   - Wer ist die Zielgruppe?  
   - Welche Kernfunktionen soll die Anwendung bieten?  
   - Gibt es besondere Anforderungen (z. B. Datenschutz, Performance)?  

2️⃣ **Technologie-Stack bestimmen**:  
   - Welche Technologien nutzt ihr für Frontend, Backend und Datenbank?  
   - Welche Hosting-Lösung ist für das Projekt am besten geeignet?  
   - Welche Tools nutzt ihr für Testing & CI/CD?  

3️⃣ **Architektur und Entwicklung planen**:  
   - Soll das Projekt monolithisch oder als Microservices aufgebaut sein?  
   - Wie sieht die Datenbankstruktur aus?  
   - Wie wird der Entwicklungsprozess organisiert? (z. B. Scrum, Kanban)  

4️⃣ **Ergebnisse dokumentieren & präsentieren**:  
   - Erstellt eine kurze Skizze oder ein Diagramm der Architektur  
   - Stellt die wichtigsten Entscheidungen in Stichpunkten zusammen  
   - Präsentiert eure Lösung in 5 Minuten  

---

## 📌 Beispiel-Projekt: Planung einer **To-Do-App**

### **1. Anforderungen & Zielgruppe**
✅ Zielgruppe: Berufstätige, Studierende  
✅ Funktionen:
   - Nutzer können sich registrieren und anmelden  
   - Aufgaben hinzufügen, bearbeiten, löschen  
   - Filterung nach Kategorien (z. B. Arbeit, Privat)  
✅ Besondere Anforderungen:
   - Mobile-freundlich (Responsive Design)  
   - Offline-Funktionalität für unterwegs  
   - Daten sollen sicher gespeichert werden  

### **2. Technologie-Stack**
✅ **Frontend:** React + TailwindCSS  
✅ **Backend:** Node.js mit Express  
✅ **Datenbank:** MongoDB (NoSQL)  
✅ **Hosting:** Vercel (Frontend) + MongoDB Atlas (Datenbank)  
✅ **CI/CD & Testing:** GitHub Actions + Cypress für UI-Tests  

### **3. Architektur & Entwicklungsplan**
✅ **Architektur:**  
   - Frontend kommuniziert mit Backend via REST API  
   - Backend verarbeitet Logik und speichert Daten in MongoDB  
   - Authentifizierung mit JWT (JSON Web Token)  

✅ **Datenbankstruktur (Vereinfachtes Schema)**:  
```plaintext
User:
- id (String, Unique)
- username (String, Unique)
- password (String, Hashed)

Task:
- id (String, Unique)
- userId (String, References User)
- title (String)
- description (String)
- status (Enum: "Open", "In Progress", "Done")
- category (String)
```
✅ **Entwicklungsmethodik:**  
   - Agiles Vorgehen mit Scrum  
   - Tägliche Abstimmung & zweiwöchentliche Sprints  
   - Erste Version innerhalb von 3 Wochen als MVP  

---

## ⚠️ Wichtige Punkte bei der Planung

📌 **Zielgruppe immer im Blick behalten**:  
   - Welche Probleme löst die Anwendung für die Nutzer?  
   - Was sind die wichtigsten Funktionen?  

📌 **Technologie-Stack mit Bedacht wählen**:  
   - Passt die Technologie zu den Anforderungen?  
   - Ist das Team mit der Technologie vertraut?  

📌 **Saubere Architektur planen**:  
   - Monolith vs. Microservices?  
   - API-Schnittstellen und Datenflüsse klar definieren  

📌 **Deployment & Wartung berücksichtigen**:  
   - Wo wird das Projekt gehostet?  
   - Wie werden Updates und Fehlerbehebungen gehandhabt?  

---

## 🏆 Aufgabe für euch: Plant euer eigenes Webprojekt!

**💡 Szenario:**  
Euer Team plant eine **Event-Planungsplattform**, mit der Nutzer Veranstaltungen erstellen, verwalten und mit anderen teilen können. Die Anwendung soll folgende Funktionen haben:

✅ Nutzer können sich anmelden und Events erstellen  
✅ Events können mit Freunden geteilt und kommentiert werden  
✅ Automatische Benachrichtigungen für Teilnehmer  
✅ Integration eines Kalender-Features  
✅ Mobile-optimiertes Design  

**Eure Aufgabe:**  
1️⃣ Erstellt einen **Plan für das Projekt** mit den oben genannten Schritten  
2️⃣ Dokumentiert eure Entscheidungen in einer kurzen Zusammenfassung  
3️⃣ Erstellt eine Skizze der Architektur (z. B. auf einem Whiteboard oder als Diagramm)  
4️⃣ Präsentiert eure Ergebnisse in 5 Minuten  

**🎤 Viel Erfolg! Wir freuen uns auf eure kreativen Lösungen!** 🚀
