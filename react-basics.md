# React für Anfänger
## Ein vollständiger Praxisleitfaden mit kommentiertem Demoprojekt

## Inhaltsverzeichnis
1. [Was ist React und warum sollte ich es lernen?](#was-ist-react-und-warum-sollte-ich-es-lernen)
2. [Die Grundbausteine von React verstehen](#die-grundbausteine-von-react-verstehen)
3. [Werkzeuge und Umgebung einrichten](#werkzeuge-und-umgebung-einrichten)
4. [Unser Beispielprojekt: Eine Todo-App](#unser-beispielprojekt-eine-todo-app)
5. [Aufbau und Funktionsweise der Todo-App](#aufbau-und-funktionsweise-der-todo-app)
6. [Erweiterte Konzepte für die nächsten Schritte](#erweiterte-konzepte-für-die-nächsten-schritte)
7. [Häufige Fragen und Problemlösungen](#häufige-fragen-und-problemlösungen)

## Was ist React und warum sollte ich es lernen?

### Was genau ist React?

React ist eine JavaScript-Bibliothek, die von Facebook (jetzt Meta) entwickelt wurde, um interaktive Benutzeroberflächen zu erstellen. Stell dir React wie einen Baukasten vor, mit dem du Webseiten und Apps bauen kannst, indem du kleine, wiederverwendbare Teile zusammensetzt.

**Einfach erklärt:** Wenn du schon einmal mit LEGO gespielt hast, kennst du das Prinzip: Du hast verschiedene Bausteine, die du zu etwas Größerem zusammensetzen kannst. In React nennen wir diese Bausteine "Komponenten".

### Warum ist React so beliebt?

1. **Einfacheres Denken in Bausteinen**:  
   Statt eine ganze Webseite auf einmal zu programmieren, erstellst du kleine, überschaubare Teile und setzt sie zusammen.

2. **Effiziente Aktualisierungen**:  
   React aktualisiert nur die Teile der Webseite, die sich wirklich geändert haben. Das macht deine App schneller und flüssiger.

3. **Deklarativer Ansatz**:  
   Du musst nicht jeden kleinen Schritt programmieren. Stattdessen sagst du React, wie das Endergebnis aussehen soll, und React kümmert sich um den Rest.

   ```javascript
   // Ohne React: Viele Schritte
   const button = document.createElement('button');
   button.textContent = 'Klick mich';
   button.addEventListener('click', () => alert('Hallo!'));
   document.body.appendChild(button);

   // Mit React: Deklarativ und übersichtlich
   function Button() {
     return <button onClick={() => alert('Hallo!')}>Klick mich</button>;
   }
   ```

4. **Große Community und viel Unterstützung**:  
   Tausende Entwickler weltweit nutzen React. Das bedeutet viele Tutorials, Bibliotheken und Hilfe bei Problemen.

5. **Berufliche Möglichkeiten**:  
   React-Kenntnisse sind auf dem Arbeitsmarkt sehr gefragt.

6. **Mobile Apps mit derselben Grundlage**:  
   Mit React Native kannst du das Gelernte auch für die Entwicklung von mobilen Apps nutzen.

## Die Grundbausteine von React verstehen

Bevor wir in die Praxis einsteigen, sollten wir die wichtigsten Konzepte von React kennenlernen. Diese Grundbausteine werden in jedem React-Projekt verwendet.

### 1. Komponenten: Das Herzstück von React

**Was ist eine Komponente?**  
Eine Komponente ist wie ein LEGO-Baustein für deine Webseite. Sie kann so klein wie ein Button oder so groß wie eine ganze Seite sein. Jede Komponente hat ihre eigene Logik und Darstellung.

Es gibt zwei Arten, Komponenten zu erstellen:

#### Funktionskomponenten (Modern und empfohlen)

```jsx
// Eine einfache Begrüßungskomponente als Funktion
function Begruessung(props) {
  return <h1>Hallo, {props.name}!</h1>;
}

// Verwendung:
<Begruessung name="Max" />  // Zeigt "Hallo, Max!" an
```

#### Klassenkomponenten (Älterer Ansatz)

```jsx
// Eine Begrüßungskomponente als Klasse
import React from 'react';

class Begruessung extends React.Component {
  render() {
    return <h1>Hallo, {this.props.name}!</h1>;
  }
}
```

**Wichtig:** In modernen React-Projekten werden fast ausschließlich Funktionskomponenten verwendet. Klassenkomponenten zu kennen ist dennoch nützlich, da du sie in älteren Projekten antreffen wirst.

### 2. JSX: HTML und JavaScript vereint

JSX ist eine spezielle Syntax, die HTML direkt in JavaScript erlaubt. Es macht den Code viel lesbarer und intuitiver.

```jsx
// Das ist JSX - HTML-ähnliche Syntax in JavaScript
const element = <h1 className="titel">Willkommen zu React</h1>;

// Im Hintergrund wird es zu diesem JavaScript-Code umgewandelt
const element = React.createElement(
  'h1',
  { className: 'titel' },
  'Willkommen zu React'
);
```

**Die wichtigsten JSX-Regeln für Anfänger:**

1. **Ein Wurzelelement pro Komponente**:  
   Jede Komponente muss genau ein übergeordnetes Element zurückgeben.

   ```jsx
   // Falsch - mehrere Elemente auf oberster Ebene
   function Falsch() {
     return (
       <h1>Überschrift</h1>
       <p>Absatz</p>
     );
   }

   // Richtig - ein umschließendes Element
   function Richtig() {
     return (
       <div>
         <h1>Überschrift</h1>
         <p>Absatz</p>
       </div>
     );
   }

   // Auch richtig - mit React Fragment (kein zusätzliches DOM-Element)
   function AuchRichtig() {
     return (
       <>
         <h1>Überschrift</h1>
         <p>Absatz</p>
       </>
     );
   }
   ```

2. **Tags müssen geschlossen werden**:  
   Alle HTML-Tags müssen geschlossen werden, entweder mit einem schließenden Tag oder als selbstschließendes Tag.

   ```jsx
   // Richtig
   <div>Inhalt</div>
   <img src="bild.jpg" alt="Beschreibung" />
   ```

3. **JavaScript in geschweiften Klammern**:  
   JavaScript-Ausdrücke werden in geschweifte Klammern {} eingefasst.

   ```jsx
   const name = "Max";
   const element = <h1>Hallo, {name}!</h1>;
   ```

4. **HTML-Attribute werden camelCase geschrieben**:  
   In JSX werden HTML-Attribute in camelCase geschrieben, nicht in Kleinbuchstaben.

   ```jsx
   // HTML:    <div class="container" onclick="handleClick()">
   // JSX:
   <div className="container" onClick={handleClick}>
   ```

### 3. Props: Daten an Komponenten übergeben

Props (kurz für "properties") sind der Weg, wie Daten von einer Elternkomponente an eine Kindkomponente weitergegeben werden.

```jsx
// Elternkomponente übergibt Daten an Kindkomponente
function App() {
  return (
    <div>
      {/* name, alter und istAngemeldet sind Props */}
      <Begruessung name="Max" alter={28} istAngemeldet={true} />
    </div>
  );
}

// Kindkomponente empfängt Props
function Begruessung(props) {
  // Zugriff auf die übergebenen Daten
  return (
    <div>
      <h1>Hallo, {props.name}!</h1>
      <p>Du bist {props.alter} Jahre alt.</p>
      {/* Bedingtes Rendern - wird nur angezeigt, wenn istAngemeldet true ist */}
      {props.istAngemeldet && <p>Du bist angemeldet.</p>}
    </div>
  );
}
```

**Wichtig zu wissen:**

1. **Props sind schreibgeschützt:**  
   Eine Komponente darf ihre eigenen Props nie ändern. Sie werden von der Elternkomponente gesteuert.

2. **Props können verschiedene Datentypen haben:**  
   - Strings: `name="Max"`
   - Zahlen: `alter={28}` (beachte die geschweiften Klammern für nicht-String-Werte)
   - Booleans: `istAngemeldet={true}`
   - Arrays: `hobbys={['Lesen', 'Kochen']}`
   - Objekte: `person={{ name: 'Max', alter: 28 }}`
   - Funktionen: `onClick={handleClick}`

3. **Destrukturierung für bessere Lesbarkeit:**

   ```jsx
   // Statt props.name, props.alter, props.istAngemeldet
   function Begruessung({ name, alter, istAngemeldet }) {
     return (
       <div>
         <h1>Hallo, {name}!</h1>
         <p>Du bist {alter} Jahre alt.</p>
         {istAngemeldet && <p>Du bist angemeldet.</p>}
       </div>
     );
   }
   ```

### 4. State: Interaktive Komponenten erstellen

Der State (Zustand) ist ein Objekt, das Daten enthält, die sich im Laufe der Zeit ändern können. Wenn sich der State ändert, wird die Komponente automatisch neu gerendert.

In modernem React verwenden wir den `useState`-Hook:

```jsx
import React, { useState } from 'react';

function Zaehler() {
  // useState gibt uns zwei Dinge zurück:
  // 1. Die aktuelle State-Variable (zaehler)
  // 2. Eine Funktion zum Aktualisieren des States (setZaehler)
  const [zaehler, setZaehler] = useState(0); // 0 ist der Anfangswert
  
  return (
    <div>
      <p>Du hast {zaehler} mal geklickt</p>
      <button onClick={() => setZaehler(zaehler + 1)}>
        Klick mich
      </button>
    </div>
  );
}
```

**Die goldenen Regeln für State:**

1. **Ändere State nie direkt:**  
   Verwende immer die vom `useState`-Hook bereitgestellte Funktion.

   ```jsx
   // FALSCH - niemals direkter Zugriff
   zaehler = zaehler + 1;
   
   // RICHTIG - immer die Setter-Funktion verwenden
   setZaehler(zaehler + 1);
   ```

2. **State-Updates können asynchron sein:**  
   Wenn du den aktuellen State für das nächste Update benötigst, verwende die Funktionsform:

   ```jsx
   // Unsicher, wenn mehrere Updates hintereinander erfolgen
   setZaehler(zaehler + 1);
   
   // Sicher, weil es den aktuellen State als Parameter erhält
   setZaehler(aktuellerWert => aktuellerWert + 1);
   ```

3. **State ist komponenten-lokal:**  
   Jede Komponente hat ihren eigenen, unabhängigen State. Wenn du State zwischen Komponenten teilen möchtest, musst du ihn in eine gemeinsame Elternkomponente "hochheben".

### 5. React Hooks: Die modernen Werkzeuge

Hooks sind spezielle Funktionen, die es erlauben, in Funktionskomponenten Features wie State und Lifecycle-Methoden zu nutzen.

#### useState - für reaktive Daten

```jsx
const [wert, setWert] = useState(initialWert);
```

#### useEffect - für Nebeneffekte und Lifecycle

Mit `useEffect` kannst du Code ausführen, der "Nebeneffekte" hat, wie z.B. Daten laden, DOM-Elemente manipulieren oder Timer einrichten.

```jsx
import { useState, useEffect } from 'react';

function BeispielKomponente() {
  const [zaehler, setZaehler] = useState(0);
  
  // Effekt ohne Abhängigkeiten - läuft nach jedem Render
  useEffect(() => {
    console.log('Komponente wurde gerendert');
  });
  
  // Effekt mit leerer Abhängigkeitsliste - läuft nur beim ersten Render
  useEffect(() => {
    console.log('Komponente wurde montiert');
    
    // Aufräumfunktion - läuft, wenn die Komponente entfernt wird
    return () => {
      console.log('Komponente wird entfernt');
    };
  }, []);
  
  // Effekt mit Abhängigkeit - läuft nur, wenn sich zaehler ändert
  useEffect(() => {
    document.title = `Du hast ${zaehler} mal geklickt`;
    console.log(`Zähler aktualisiert: ${zaehler}`);
  }, [zaehler]);
  
  return (
    <div>
      <p>Zähler: {zaehler}</p>
      <button onClick={() => setZaehler(zaehler + 1)}>Erhöhen</button>
    </div>
  );
}
```

## Werkzeuge und Umgebung einrichten

Bevor wir mit der Programmierung beginnen, müssen wir die notwendigen Werkzeuge einrichten.

### 1. Voraussetzungen

Diese Grundwerkzeuge benötigst du für die React-Entwicklung:

- **Node.js und npm**: Die JavaScript-Laufzeitumgebung und der Paketmanager
  - Lade sie von [nodejs.org](https://nodejs.org/) herunter
  - Node.js ist eine Plattform, die es ermöglicht, JavaScript außerhalb des Browsers auszuführen
  - npm (Node Package Manager) wird verwendet, um JavaScript-Bibliotheken zu installieren

**Überprüfen der Installation:**
```bash
# Diese Befehle in der Kommandozeile/Terminal ausführen
node --version  # sollte die Version anzeigen, z.B. v16.14.0
npm --version   # sollte die Version anzeigen, z.B. 8.3.1
```

### 2. Ein React-Projekt erstellen

Es gibt zwei moderne Wege, ein neues React-Projekt zu starten:

#### Option 1: Create React App (traditionell)

Create React App ist ein offizielles Tool von Facebook, das alles für dich einrichtet.

```bash
# Erstellt ein neues React-Projekt
npx create-react-app meine-todo-app

# Wechselt in das Projektverzeichnis
cd meine-todo-app

# Startet den Entwicklungsserver
npm start
```

#### Option 2: Vite (moderner und schneller)

Vite ist ein neueres Build-Tool, das einen schnelleren Entwicklungsserver bietet.

```bash
# Erstellt ein neues React-Projekt mit Vite
npm create vite@latest meine-todo-app -- --template react

# Wechselt in das Projektverzeichnis
cd meine-todo-app

# Installiert Abhängigkeiten
npm install

# Startet den Entwicklungsserver
npm run dev
```

### 3. Ordnerstruktur verstehen

Nach dem Erstellen deines Projekts siehst du eine Ordnerstruktur ähnlich dieser:

```
meine-todo-app/
├── node_modules/        # Alle installierten Pakete (nicht bearbeiten)
├── public/              # Statische Dateien wie HTML und Bilder
├── src/                 # Unser Quellcode (hier werden wir arbeiten)
│   ├── App.jsx          # Hauptkomponente
│   ├── index.jsx        # Einstiegspunkt unserer Anwendung
│   └── ...
├── package.json         # Projekteinstellungen und Abhängigkeiten
├── README.md            # Projektdokumentation
└── ... (weitere Konfigurationsdateien)
```

Das Herzstück ist der `src`-Ordner, in dem wir unseren eigenen Code schreiben werden.

## Unser Beispielprojekt: Eine Todo-App

Lass uns nun eine einfache Todo-App erstellen, die alle grundlegenden React-Konzepte veranschaulicht. Wir werden Schritt für Schritt vorgehen und jeden Teil ausführlich erklären.

### Schritt 1: Projektstruktur planen

Für unsere Todo-App erstellen wir eine klare Struktur:

```
src/
├── components/          # Ordner für unsere Komponenten
│   ├── TodoForm.jsx     # Formular zum Hinzufügen neuer Todos
│   ├── TodoItem.jsx     # Einzelnes Todo-Element
│   └── TodoList.jsx     # Liste aller Todos
├── App.jsx              # Hauptkomponente, die alles zusammenhält
└── index.jsx            # Einstiegspunkt
```

Diese Struktur folgt dem Prinzip der Komponenten-Komposition, bei dem jede Komponente eine bestimmte Aufgabe hat:
- `TodoForm` kümmert sich um die Eingabe neuer Todos
- `TodoItem` zeigt ein einzelnes Todo an und bietet Interaktionsmöglichkeiten
- `TodoList` verwaltet die Anzeige aller Todos
- `App` koordiniert alle Komponenten und enthält den Haupt-State

### Schritt 2: Die Komponenten erstellen

Lass uns jetzt jede Komponente im Detail mit ausführlichen Kommentaren entwickeln.

**App.jsx** - Die Hauptkomponente unserer Anwendung

```jsx
/**
 * App.jsx - Die Hauptkomponente unserer Todo-Anwendung
 * 
 * Diese Komponente:
 * 1. Enthält den Hauptzustand (State) aller Todos
 * 2. Stellt Funktionen zum Hinzufügen, Umschalten und Löschen von Todos bereit
 * 3. Kombiniert alle untergeordneten Komponenten zu unserer kompletten Anwendung
 */

// Importiere React und den useState-Hook für die Zustandsverwaltung
import React, { useState } from 'react';

// Importiere unsere eigenen Komponenten
import TodoList from './components/TodoList';
import TodoForm from './components/TodoForm';

// Importiere die CSS-Datei für das Styling
import './App.css';

// Definiere die App-Komponente als Funktionskomponente
function App() {
  /**
   * State für unsere Todo-Liste
   * 
   * useState gibt uns:
   * - todos: Das aktuelle Array mit allen Todo-Objekten
   * - setTodos: Eine Funktion zum Aktualisieren des Arrays
   * 
   * Der Parameter von useState ist der Anfangswert (hier ein Array mit zwei Beispiel-Todos)
   */
  const [todos, setTodos] = useState([
    // Jedes Todo ist ein Objekt mit einer ID, einem Text und einem Status
    { id: 1, text: 'React lernen', completed: false },
    { id: 2, text: 'Einkaufen gehen', completed: true },
  ]);

  /**
   * Funktion zum Hinzufügen eines neuen Todos
   * 
   * Diese Funktion wird an die TodoForm-Komponente weitergegeben
   * @param {string} text - Der Text des neuen Todos
   */
  const addTodo = (text) => {
    // Erstelle ein neues Todo-Objekt
    const newTodo = {
      // Generiere eine einzigartige ID basierend auf dem aktuellen Zeitstempel
      id: Date.now(),
      // Der übergebene Text des Todos
      text: text,
      // Neue Todos sind standardmäßig nicht abgeschlossen
      completed: false
    };
    
    /**
     * Aktualisiere den State mit einem neuen Array
     * [...todos, newTodo] erstellt eine Kopie des alten Arrays und fügt das neue Todo am Ende hinzu
     * 
     * Wir erstellen immer ein neues Array, anstatt das bestehende zu verändern
     * Das ist wichtig für die Immutabilität in React (unveränderliche Daten)
     */
    setTodos([...todos, newTodo]);
  };

  /**
   * Funktion zum Umschalten des Status eines Todos (erledigt/nicht erledigt)
   * 
   * Diese Funktion wird über TodoList an TodoItem weitergegeben
   * @param {number} id - Die ID des Todos, dessen Status geändert werden soll
   */
  const toggleTodo = (id) => {
    // Aktualisiere den State mit einem neuen Array
    setTodos(
      // Durchlaufe jedes Todo im aktuellen Array mit map()
      todos.map(todo => {
        // Wenn die ID übereinstimmt...
        if (todo.id === id) {
          // ...erstelle ein neues Todo-Objekt mit allen Eigenschaften des alten, aber umgekehrtem Status
          return { ...todo, completed: !todo.completed };
        }
        // Sonst behalte das Todo unverändert
        return todo;
      })
    );
  };

  /**
   * Funktion zum Löschen eines Todos
   * 
   * Diese Funktion wird über TodoList an TodoItem weitergegeben
   * @param {number} id - Die ID des zu löschenden Todos
   */
  const deleteTodo = (id) => {
    // Aktualisiere den State mit einem gefilterten Array
    // filter() erstellt ein neues Array, das nur die Elemente enthält, die die Bedingung erfüllen
    setTodos(
      // Behalte nur die Todos, deren ID NICHT mit der übergebenen ID übereinstimmt
      todos.filter(todo => todo.id !== id)
    );
  };

  // Die Render-Methode definiert, was auf dem Bildschirm angezeigt wird
  return (
    // Ein Container für unsere gesamte App
    <div className="app">
      <h1>Meine Todo-Liste</h1>
      
      {/* 
        TodoForm-Komponente zum Hinzufügen neuer Todos
        Wir übergeben die addTodo-Funktion als Prop, damit TodoForm damit interagieren kann
      */}
      <TodoForm addTodo={addTodo} />
      
      {/* 
        TodoList-Komponente zum Anzeigen aller Todos
        Wir übergeben:
        - todos: das Array mit allen Todo-Objekten
        - toggleTodo: die Funktion zum Umschalten des Status
        - deleteTodo: die Funktion zum Löschen eines Todos
      */}
      <TodoList 
        todos={todos} 
        toggleTodo={toggleTodo} 
        deleteTodo={deleteTodo} 
      />
    </div>
  );
}

// Exportiere die App-Komponente, damit sie in index.jsx importiert werden kann
export default App;
```

**components/TodoForm.jsx** - Formular zum Hinzufügen neuer Todos

```jsx
/**
 * TodoForm.jsx - Komponente für die Eingabe neuer Todos
 * 
 * Diese Komponente:
 * 1. Zeigt ein Eingabefeld und einen Button an
 * 2. Verwaltet den Text-Input im lokalen State
 * 3. Ruft die addTodo-Funktion der Elternkomponente auf, wenn das Formular abgeschickt wird
 */

// Importiere React und den useState-Hook für die Zustandsverwaltung
import React, { useState } from 'react';

/**
 * TodoForm-Komponente
 * 
 * @param {Object} props - Die Properties, die von der Elternkomponente übergeben werden
 * @param {Function} props.addTodo - Funktion zum Hinzufügen eines neuen Todos
 */
function TodoForm({ addTodo }) {
  /**
   * State für den Eingabetext des Formulars
   * 
   * useState gibt uns:
   * - text: Der aktuelle Wert des Eingabefelds
   * - setText: Eine Funktion zum Aktualisieren des Werts
   * 
   * Der leere String '' ist der Anfangswert
   */
  const [text, setText] = useState('');

  /**
   * Event-Handler für das Absenden des Formulars
   * 
   * Diese Funktion wird aufgerufen, wenn das Formular abgeschickt wird (Enter oder Button-Klick)
   * @param {Event} e - Das Event-Objekt
   */
  const handleSubmit = (e) => {
    // Verhindere das standardmäßige Verhalten des Browsers (Seite neu laden)
    e.preventDefault();
    
    // Prüfe, ob der Text nicht leer ist (nach dem Entfernen von Leerzeichen)
    // Wenn text.trim() leer ist, wird die Funktion hier beendet
    if (!text.trim()) return;
    
    // Rufe die addTodo-Funktion mit dem aktuellen Text auf
    // Diese Funktion haben wir als Prop von der App-Komponente erhalten
    addTodo(text);
    
    // Setze das Eingabefeld zurück, indem wir den State auf einen leeren String setzen
    setText('');
  };

  // Die Render-Methode definiert, was auf dem Bildschirm angezeigt wird
  return (
    // Ein Formular-Element mit einem onSubmit-Event-Handler
    <form onSubmit={handleSubmit} className="todo-form">
      {/* 
        Eingabefeld für den Todo-Text
        - value ist an den State gebunden (text)
        - onChange aktualisiert den State bei jeder Änderung
        - placeholder zeigt einen Hinweis im leeren Feld an
      */}
      <input
        type="text"
        value={text}
        onChange={(e) => setText(e.target.value)}
        placeholder="Neues Todo hinzufügen..."
      />
      
      {/* 
        Button zum Absenden des Formulars
        Der type="submit" sorgt dafür, dass das Formular abgeschickt wird,
        wenn der Button geklickt wird
      */}
      <button type="submit">Hinzufügen</button>
    </form>
  );
}

// Exportiere die TodoForm-Komponente, damit sie in App.jsx importiert werden kann
export default TodoForm;
```

**components/TodoItem.jsx** - Einzelnes Todo-Element

```jsx
/**
 * TodoItem.jsx - Komponente für ein einzelnes Todo-Element
 * 
 * Diese Komponente:
 * 1. Zeigt ein einzelnes Todo mit Checkbox, Text und Löschen-Button an
 * 2. Ermöglicht das Umschalten des Status (erledigt/nicht erledigt)
 * 3. Ermöglicht das Löschen des Todos
 */

// Importiere React
import React from 'react';

/**
 * TodoItem-Komponente
 * 
 * @param {Object} props - Die Properties, die von der Elternkomponente übergeben werden
 * @param {Object} props.todo - Das Todo-Objekt mit id, text und completed
 * @param {Function} props.toggleTodo - Funktion zum Umschalten des Status
 * @param {Function} props.deleteTodo - Funktion zum Löschen des Todos
 */
function TodoItem({ todo, toggleTodo, deleteTodo }) {
  // Die Render-Methode definiert, was auf dem Bildschirm angezeigt wird
  return (
    // Container für das Todo-Element mit bedingter CSS-Klasse
    // Die Klasse 'completed' wird nur hinzugefügt, wenn todo.completed true ist
    <div className={`todo-item ${todo.completed ? 'completed' : ''}`}>
      {/* 
        Checkbox zum Markieren als erledigt/nicht erledigt
        - checked wird an den completed-Status des Todos gebunden
        - onChange ruft toggleTodo mit der Todo-ID auf, wenn die Checkbox angeklickt wird
      */}
      <input
        type="checkbox"
        checked={todo.completed}
        onChange={() => toggleTodo(todo.id)}
      />
      
      {/* Text des Todos */}
      <p>{todo.text}</p>
      
      {/* 
        Button zum Löschen des Todos
        onClick ruft deleteTodo mit der Todo-ID auf, wenn der Button geklickt wird
      */}
      <button onClick={() => deleteTodo(todo.id)}>Löschen</button>
    </div>
  );
}

// Exportiere die TodoItem-Komponente, damit sie in TodoList.jsx importiert werden kann
export default TodoItem;
```

**components/TodoList.jsx** - Liste aller Todos

```jsx
/**
 * TodoList.jsx - Komponente für die Liste aller Todos
 * 
 * Diese Komponente:
 * 1. Zeigt entweder eine Meldung an, wenn keine Todos vorhanden sind
 * 2. Oder rendert für jedes Todo eine TodoItem-Komponente
 */

// Importiere React
import React from 'react';

// Importiere die TodoItem-Komponente
import TodoItem from './TodoItem';

/**
 * TodoList-Komponente
 * 
 * @param {Object} props - Die Properties, die von der Elternkomponente übergeben werden
 * @param {Array} props.todos - Das Array mit allen Todo-Objekten
 * @param {Function} props.toggleTodo - Funktion zum Umschalten des Status
 * @param {Function} props.deleteTodo - Funktion zum Löschen eines Todos
 */
function TodoList({ todos, toggleTodo, deleteTodo }) {
  // Die Render-Methode definiert, was auf dem Bildschirm angezeigt wird
  return (
    // Container für die gesamte Todo-Liste
    <div className="todo-list">
      {/* 
        Bedingte Anzeige mit dem ternären Operator (condition ? trueValue : falseValue)
        - Wenn das todos-Array leer ist, zeigen wir eine Meldung an
        - Andernfalls zeigen wir die Liste der Todos
      */}
      {todos.length === 0 ? (
        // Meldung, wenn keine Todos vorhanden sind
        <p>Keine Todos vorhanden!</p>
      ) : (
        // Durchlaufe das todos-Array mit map() und erstelle für jedes Todo eine TodoItem-Komponente
        todos.map(todo => (
          // Jedes Element in einer Liste muss einen eindeutigen key haben
          // Wir verwenden die ID des Todos als key, damit React die Elemente effizient aktualisieren kann
          <TodoItem
            key={todo.id}
            todo={todo}
            toggleTodo={toggleTodo}
            deleteTodo={deleteTodo}
          />
        ))
      )}
    </div>
  );
}

// Exportiere die TodoList-Komponente, damit sie in App.jsx importiert werden kann
export default TodoList;

/**
 * App.css - Styling für unsere Todo-App
 * 
 * Dieses CSS sollte in einer separaten Datei App.css gespeichert werden
 */

/*
.app {
  max-width: 500px;           /* Begrenzt die Breite der App */
  margin: 0 auto;             /* Zentriert die App horizontal */
  padding: 20px;              /* Fügt inneren Abstand hinzu */
  font-family: Arial, sans-serif; /* Legt die Schriftart fest */
  box-shadow: 0 0 10px rgba(0, 0, 0, 0.1); /* Fügt einen subtilen Schatten hinzu */
  border-radius: 8px;         /* Rundet die Ecken ab */
  background-color: #fff;     /* Weißer Hintergrund */
}

/* Styling für die Überschrift */
.app h1 {
  text-align: center;         /* Zentriert den Text */
  color: #333;                /* Dunkelgraue Textfarbe */
  margin-bottom: 20px;        /* Abstand nach unten */
}

/* Styling für das Todo-Formular */
.todo-form {
  display: flex;              /* Flexbox-Layout für nebeneinander liegende Elemente */
  margin-bottom: 20px;        /* Abstand nach unten */
}

/* Styling für das Eingabefeld */
.todo-form input {
  flex-grow: 1;               /* Lässt das Input-Feld den verfügbaren Platz einnehmen */
  padding: 10px;              /* Innerer Abstand */
  border: 1px solid #ddd;     /* Hellgraue Rahmenfarbe */
  border-radius: 4px 0 0 4px; /* Rundet nur die linken Ecken ab */
  font-size: 16px;            /* Schriftgröße */
}

/* Styling für den Hinzufügen-Button */
.todo-form button {
  padding: 10px 15px;         /* Innerer Abstand */
  background-color: #0066ff;  /* Blaue Hintergrundfarbe */
  color: white;               /* Weiße Textfarbe */
  border: none;               /* Kein Rahmen */
  border-radius: 0 4px 4px 0; /* Rundet nur die rechten Ecken ab */
  cursor: pointer;            /* Hand-Cursor bei Hover */
  font-size: 16px;            /* Schriftgröße */
  font-weight: bold;          /* Fetter Text */
}

/* Hover-Effekt für den Button */
.todo-form button:hover {
  background-color: #0052cc;  /* Dunkleres Blau bei Hover */
}

/* Styling für ein Todo-Element */
.todo-item {
  display: flex;              /* Flexbox-Layout */
  align-items: center;        /* Vertikal zentrieren */
  padding: 10px;              /* Innerer Abstand */
  margin-bottom: 10px;        /* Abstand nach unten */
  background-color: #f9f9f9;  /* Hellgrauer Hintergrund */
  border-radius: 4px;         /* Rundet die Ecken ab */
  transition: background-color 0.3s; /* Sanfter Übergang bei Änderungen */
}

/* Hover-Effekt für Todo-Items */
.todo-item:hover {
  background-color: #f0f0f0;  /* Noch hellerer Hintergrund bei Hover */
}

/* Styling für den Text in einem Todo-Element */
.todo-item p {
  flex-grow: 1;               /* Lässt den Text den verfügbaren Platz einnehmen */
  margin: 0 10px;             /* Abstand links und rechts */
  word-break: break-word;     /* Bricht lange Wörter um */
}

/* Spezielles Styling für erledigte Todos */
.todo-item.completed p {
  text-decoration: line-through; /* Durchstreichen des Textes */
  color: #888;                /* Hellere Textfarbe */
}

/* Styling für die Checkbox */
.todo-item input[type="checkbox"] {
  margin-right: 10px;         /* Abstand nach rechts */
  cursor: pointer;            /* Hand-Cursor bei Hover */
}

/* Styling für den Löschen-Button */
.todo-item button {
  padding: 5px 10px;          /* Innerer Abstand */
  background-color: #ff4d4d;  /* Rote Hintergrundfarbe */
  color: white;               /* Weiße Textfarbe */
  border: none;               /* Kein Rahmen */
  border-radius: 4px;         /* Rundet die Ecken ab */
  cursor: pointer;            /* Hand-Cursor bei Hover */
  font-size: 14px;            /* Schriftgröße */
}

/* Hover-Effekt für den Löschen-Button */
.todo-item button:hover {
  background-color: #e60000; /* Dunkleres Rot bei Hover */
}

/* Styling für die "Keine Todos"-Meldung */
.todo-list p {
  text-align: center;         /* Zentriert den Text */
  color: #888;                /* Hellgraue Textfarbe */
  font-style: italic;         /* Kursiver Text */
  padding: 20px 0;            /* Abstand oben und unten */
}
*/

## Aufbau und Funktionsweise der Todo-App

Nachdem wir nun alle Komponenten erstellt haben, wollen wir verstehen, wie sie zusammenarbeiten und wie der Datenfluss funktioniert.

### Die Komponenten-Hierarchie

Unsere App ist wie folgt strukturiert:

```
App (Hauptkomponente)
├── TodoForm (Eingabe neuer Todos)
└── TodoList (Container für alle Todos)
    └── TodoItem (Einzelnes Todo, kann mehrfach vorkommen)
```

Diese Struktur folgt dem Prinzip der Komponenten-Komposition, bei dem komplexe Benutzeroberflächen aus einfacheren, wiederverwendbaren Teilen zusammengesetzt werden.

### Datenfluss in React

Ein wichtiges Konzept in React ist der unidirektionale (einwegige) Datenfluss:

1. **Daten fließen von oben nach unten (Eltern zu Kindern):**
   - Die `App`-Komponente enthält den Hauptzustand (todos-Array)
   - Dieser Zustand wird als Props an untergeordnete Komponenten weitergegeben
   - `TodoList` erhält das komplette todos-Array
   - Jedes `TodoItem` erhält ein einzelnes Todo aus diesem Array

2. **Aktionen fließen von unten nach oben (Kinder zu Eltern):**
   - Die Funktionen zum Ändern des Zustands (addTodo, toggleTodo, deleteTodo) werden in der `App`-Komponente definiert
   - Diese Funktionen werden als Props an untergeordnete Komponenten weitergegeben
   - Wenn ein Benutzer interagiert (z.B. auf einen Button klickt), rufen die untergeordneten Komponenten diese Funktionen auf
   - Die Funktionen aktualisieren den Zustand in der `App`-Komponente
   - React rendert automatisch alle betroffenen Komponenten neu

Dieser Muster nennt man auch "Lifting State Up" (Zustand nach oben heben) und ist ein Grundprinzip in React.

### Schritt-für-Schritt-Ablauf einer Benutzeraktion

Lassen wir uns anschauen, was passiert, wenn ein Benutzer mit unserer Todo-App interagiert:

#### Beispiel 1: Hinzufügen eines neuen Todos

1. Der Benutzer gibt Text in das Eingabefeld ein
   - Der `text`-State in `TodoForm` wird bei jeder Tastatureingabe aktualisiert

2. Der Benutzer klickt auf "Hinzufügen" oder drückt Enter
   - Die `handleSubmit`-Funktion in `TodoForm` wird ausgeführt
   - Sie prüft, ob der Text nicht leer ist
   - Sie ruft die `addTodo`-Funktion auf, die als Prop von `App` übergeben wurde

3. In der `App`-Komponente wird die `addTodo`-Funktion ausgeführt
   - Sie erstellt ein neues Todo-Objekt mit einer eindeutigen ID
   - Sie aktualisiert den `todos`-State, indem sie das neue Todo zum vorhandenen Array hinzufügt

4. React rendert die betroffenen Komponenten neu
   - `App` rendert aufgrund des geänderten `todos`-States neu
   - `TodoList` erhält das aktualisierte todos-Array und rendert neu
   - Ein neues `TodoItem` wird für das hinzugefügte Todo erstellt

#### Beispiel 2: Markieren eines Todos als erledigt

1. Der Benutzer klickt auf die Checkbox eines Todos
   - Die Checkbox in `TodoItem` löst ein onChange-Event aus
   - Sie ruft die `toggleTodo`-Funktion mit der ID des Todos auf

2. In der `App`-Komponente wird die `toggleTodo`-Funktion ausgeführt
   - Sie durchläuft das todos-Array und sucht das Todo mit der entsprechenden ID
   - Sie erstellt ein neues Array mit dem aktualisierten completed-Status des betroffenen Todos
   - Sie aktualisiert den `todos`-State mit diesem neuen Array

3. React rendert die betroffenen Komponenten neu
   - Das entsprechende `TodoItem` erhält den aktualisierten completed-Status und ändert seine Darstellung

#### Beispiel 3: Löschen eines Todos

1. Der Benutzer klickt auf den "Löschen"-Button eines Todos
   - Der Button in `TodoItem` löst ein onClick-Event aus
   - Er ruft die `deleteTodo`-Funktion mit der ID des Todos auf

2. In der `App`-Komponente wird die `deleteTodo`-Funktion ausgeführt
   - Sie filtert das todos-Array und entfernt das Todo mit der entsprechenden ID
   - Sie aktualisiert den `todos`-State mit diesem gefilterten Array

3. React rendert die betroffenen Komponenten neu
   - `TodoList` erhält das aktualisierte todos-Array ohne das gelöschte Todo
   - Das entsprechende `TodoItem` verschwindet aus der Anzeige

### Der virtuelle DOM und effizientes Rendering

Ein wichtiger Aspekt von React ist der virtuelle DOM, der effiziente Aktualisierungen ermöglicht:

1. Bei jeder State-Änderung erstellt React eine virtuelle Repräsentation des DOM (wie ein Entwurf)
2. React vergleicht diesen Entwurf mit dem vorherigen Zustand (Diffing)
3. Nur die tatsächlich geänderten Teile werden im echten DOM aktualisiert
4. Dadurch werden Aktualisierungen schneller und effizienter

In unserer Todo-App bedeutet das:
- Wenn ein Todo hinzugefügt wird, muss nur ein neues Element gerendert werden
- Wenn ein Todo als erledigt markiert wird, ändert sich nur die Darstellung dieses einen Elements
- Wenn ein Todo gelöscht wird, wird nur dieses eine Element entfernt

## Erweiterte Konzepte für die nächsten Schritte

Nachdem du die Grundlagen von React verstanden hast, kannst du dich mit fortgeschritteneren Konzepten befassen, um deine Fähigkeiten zu erweitern.

### 1. Globales State-Management

Für komplexere Anwendungen wird das Weiterreichen von Props durch viele Ebenen (Prop Drilling) umständlich. Es gibt mehrere Lösungen für globales State-Management:

#### React Context API

Die Context API ist in React eingebaut und ermöglicht den Zugriff auf Daten, ohne sie durch jede Ebene weiterzugeben:

```jsx
// 1. Erstelle einen Context
const TodoContext = React.createContext();

// 2. Erstelle einen Provider in deiner Hauptkomponente
function App() {
  const [todos, setTodos] = useState([]);
  
  // State und Funktionen, die du teilen möchtest
  const value = {
    todos,
    addTodo: (text) => { /* ... */ },
    toggleTodo: (id) => { /* ... */ },
    deleteTodo: (id) => { /* ... */ }
  };
  
  return (
    <TodoContext.Provider value={value}>
      {/* Alle Komponenten innerhalb des Providers haben Zugriff auf den Context */}
      <TodoForm />
      <TodoList />
    </TodoContext.Provider>
  );
}

// 3. Verwende den Context in einer tief verschachtelten Komponente
function TodoItem({ todo }) {
  // Greife auf den Context zu
  const { toggleTodo, deleteTodo } = useContext(TodoContext);
  
  return (
    <div>
      <input
        type="checkbox"
        checked={todo.completed}
        onChange={() => toggleTodo(todo.id)}
      />
      <p>{todo.text}</p>
      <button onClick={() => deleteTodo(todo.id)}>Löschen</button>
    </div>
  );
}
```

#### Externe State-Management-Bibliotheken

Für noch komplexere Anwendungen gibt es spezialisierte Bibliotheken:

- **Redux**: Eine umfassende Lösung mit strengen Regeln für Zustandsänderungen
- **Zustand**: Eine leichtgewichtige Alternative zu Redux
- **Recoil**: Von Facebook entwickelt, mit einem atombasierten Ansatz
- **MobX**: Verwendet beobachtbare Objekte für reaktives State-Management

### 2. Effekte und Daten-Fetching

In realen Anwendungen wirst du oft Daten von einem Server laden. Der `useEffect`-Hook ist perfekt dafür geeignet:

```jsx
function TodoApp() {
  const [todos, setTodos] = useState([]);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);
  
  // Lade Todos beim ersten Rendern
  useEffect(() => {
    async function fetchTodos() {
      try {
        setLoading(true);
        // Angenommen, wir haben eine API, die Todos zurückgibt
        const response = await fetch('https://api.example.com/todos');
        
        if (!response.ok) {
          throw new Error('Fehler beim Laden der Todos');
        }
        
        const data = await response.json();
        setTodos(data);
      } catch (err) {
        setError(err.message);
      } finally {
        setLoading(false);
      }
    }
    
    fetchTodos();
  }, []); // Leere Abhängigkeitsliste bedeutet: Nur beim ersten Rendern ausführen
  
  if (loading) return <p>Lade Todos...</p>;
  if (error) return <p>Fehler: {error}</p>;
  
  return (
    <div>
      {/* Deine Todo-App-Komponenten hier */}
    </div>
  );
}
```

### 3. Routing in React-Anwendungen

Für mehrere Seiten in deiner Anwendung brauchst du einen Router. Die beliebteste Lösung ist React Router:

```jsx
import { BrowserRouter, Routes, Route, Link } from 'react-router-dom';

function App() {
  return (
    <BrowserRouter>
      <nav>
        <Link to="/">Todos</Link>
        <Link to="/completed">Erledigte Todos</Link>
        <Link to="/about">Über uns</Link>
      </nav>

      <Routes>
        <Route path="/" element={<TodoApp />} />
        <Route path="/completed" element={<CompletedTodos />} />
        <Route path="/about" element={<AboutPage />} />
        {/* Fallback-Route für nicht gefundene Seiten */}
        <Route path="*" element={<NotFoundPage />} />
      </Routes>
    </BrowserRouter>
  );
}
```

### 4. Formular-Handling

Für komplexere Formulare gibt es spezialisierte Bibliotheken:

- **React Hook Form**: Performante Formularbibliothek mit wenig Code
- **Formik**: Umfassende Lösung für Formulare mit Validierung
- **Yup** oder **Zod**: Schemadefinition und Validierung

Beispiel mit React Hook Form:

```jsx
import { useForm } from "react-hook-form";

function TodoForm({ addTodo }) {
  const { register, handleSubmit, reset, formState: { errors } } = useForm();
  
  const onSubmit = (data) => {
    addTodo(data.todoText);
    reset(); // Formular zurücksetzen
  };
  
  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      <input
        {...register("todoText", { 
          required: "Text ist erforderlich",
          minLength: { value: 3, message: "Mindestens 3 Zeichen" }
        })}
        placeholder="Neues Todo hinzufügen..."
      />
      {errors.todoText && <p>{errors.todoText.message}</p>}
      <button type="submit">Hinzufügen</button>
    </form>
  );
}
```

## Häufige Fragen und Problemlösungen

### Wie kann ich meinen React-Code organisieren?

Eine bewährte Struktur für mittelgroße Projekte:

```
src/
├── components/       # Wiederverwendbare UI-Komponenten
│   ├── common/       # Häufig verwendete Basiskomponenten (Button, Input, etc.)
│   └── feature1/     # Komponenten für bestimmte Features
├── hooks/            # Eigene Hooks
├── context/          # Context-Definitionen
├── services/         # API-Zugriff und andere Dienste
├── utils/            # Hilfsfunktionen
├── pages/            # Komponenten für ganze Seiten
└── assets/           # Bilder, Schriftarten, etc.
```

### Wie kann ich die Leistung meiner React-App optimieren?

1. **Verwende `React.memo` für Komponenten, die sich nicht oft ändern:**
   ```jsx
   const MemoizedTodoItem = React.memo(TodoItem);
   ```

2. **Nutze `useCallback` für Funktionen, die als Props weitergegeben werden:**
   ```jsx
   const toggleTodo = useCallback((id) => {
     // Implementierung...
   }, [/* Abhängigkeiten */]);
   ```

3. **Verwende `useMemo` für teure Berechnungen:**
   ```jsx
   const completedCount = useMemo(() => {
     return todos.filter(todo => todo.completed).length;
   }, [todos]);
   ```

4. **Lazy Loading für Komponenten:**
   ```jsx
   const LazyComponent = React.lazy(() => import('./LazyComponent'));
   
   function App() {
     return (
       <Suspense fallback={<div>Lädt...</div>}>
         <LazyComponent />
       </Suspense>
     );
   }
   ```

### Wie kann ich meine React-App testen?

1. **Jest** für allgemeine Tests
2. **React Testing Library** für komponentenbasierte Tests

Beispiel für einen einfachen Test:

```jsx
// TodoItem.test.jsx
import { render, screen, fireEvent } from '@testing-library/react';
import TodoItem from './TodoItem';

test('rendert ein Todo-Item korrekt', () => {
  const mockTodo = { id: 1, text: 'Test Todo', completed: false };
  const mockToggle = jest.fn();
  const mockDelete = jest.fn();
  
  render(
    <TodoItem 
      todo={mockTodo} 
      toggleTodo={mockToggle} 
      deleteTodo={mockDelete} 
    />
  );
  
  // Prüfe, ob der Text angezeigt wird
  expect(screen.getByText('Test Todo')).toBeInTheDocument();
  
  // Prüfe, ob die Checkbox nicht angehakt ist
  const checkbox = screen.getByRole('checkbox');
  expect(checkbox).not.toBeChecked();
  
  // Simuliere Klick auf Checkbox
  fireEvent.click(checkbox);
  expect(mockToggle).toHaveBeenCalledWith(1);
  
  // Simuliere Klick auf Löschen-Button
  fireEvent.click(screen.getByText('Löschen'));
  expect(mockDelete).toHaveBeenCalledWith(1);
});
```

### Wie gehe ich mit Fehlern in React um?

Error Boundaries fangen JavaScript-Fehler in Kindkomponenten ab:

```jsx
class ErrorBoundary extends React.Component {
  constructor(props) {
    super(props);
    this.state = { hasError: false };
  }

  static getDerivedStateFromError(error) {
    return { hasError: true };
  }

  componentDidCatch(error, errorInfo) {
    console.error("Fehler in Komponente:", error, errorInfo);
  }

  render() {
    if (this.state.hasError) {
      return <h1>Etwas ist schiefgelaufen.</h1>;
    }

    return this.props.children;
  }
}

// Verwendung:
<ErrorBoundary>
  <MeineKomponente />
</ErrorBoundary>
```

## Schlussfolgerung und nächste Schritte

React bietet einen modernen, effizienten Ansatz zur Entwicklung von Benutzeroberflächen. Mit dem komponentenbasierten Ansatz, dem virtuellen DOM und dem unidirektionalen Datenfluss bietet es eine solide Grundlage für jede Webanwendung.

Als nächste Schritte könntest du:

1. **Diese Todo-App erweitern**:
   - Hinzufügen von Persistenz (LocalStorage oder eine Backend-API)
   - Implementieren von Filtern (alle/aktive/erledigte Todos)
   - Hinzufügen von Kategorien oder Prioritäten für Todos

2. **Fortgeschrittene React-Konzepte lernen**:
   - Context API und State Management
   - Custom Hooks erstellen
   - React Router für mehrere Seiten
   - Formulare mit Validierung

3. **Frameworks erkunden, die auf React aufbauen**:
   - Next.js für serverseitiges Rendering und statische Seiten
   - Gatsby für statische Websites
   - React Native für mobile Apps

Mit dem Wissen aus diesem Leitfaden hast du eine solide Grundlage, um diese nächsten Schritte zu meistern. Viel Erfolg bei deiner React-Reise!
