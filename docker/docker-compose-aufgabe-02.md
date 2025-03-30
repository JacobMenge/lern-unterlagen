# ToDo-App mit Docker Compose

## Aufgabenbeschreibung

In dieser Übung wirst du eine einfache ToDo-Anwendung mit Docker Compose zum Laufen bringen. Der gesamte Code ist bereits vorgegeben. Die Anwendung besteht aus drei Komponenten: einem React-Frontend, einem Node.js/Express-Backend und einer MongoDB-Datenbank.

Deine Aufgabe ist es, die Projektstruktur zu erstellen, die vorgegebenen Dateien zu platzieren und die Anwendung mit Docker Compose zu starten.

## Vorgabe: docker-compose.yml

```yaml
version: '3.8'

services:
  # Frontend mit React
  frontend:
    build: ./frontend
    ports:
      - "3000:3000"
    volumes:
      - ./frontend:/app
      - /app/node_modules
    environment:
      - REACT_APP_API_URL=http://localhost:4000
    depends_on:
      - backend
    networks:
      - frontend-network

  # Backend mit Node.js und Express
  backend:
    build: ./backend
    ports:
      - "4000:4000"
    volumes:
      - ./backend:/app
      - /app/node_modules
    environment:
      - MONGODB_URI=mongodb://db:27017/todoapp
      - PORT=4000
      - NODE_ENV=development
    depends_on:
      - db
    networks:
      - frontend-network
      - backend-network
    command: npm run dev

  # MongoDB Datenbank
  db:
    image: mongo:latest
    ports:
      - "27017:27017"
    volumes:
      - mongo-data:/data/db
    networks:
      - backend-network
    restart: always

networks:
  frontend-network:
  backend-network:

volumes:
  mongo-data:
```

## Schritt-für-Schritt-Anleitung

### Schritt 1: Projektstruktur erstellen

Erstelle folgende Ordnerstruktur:

```
todo-app/
├── docker-compose.yml
├── frontend/
│   ├── Dockerfile
│   ├── package.json
│   ├── public/
│   │   ├── index.html
│   │   ├── favicon.ico
│   │   └── manifest.json
│   └── src/
│       ├── App.css
│       ├── App.js
│       ├── index.css
│       └── index.js
├── backend/
    ├── Dockerfile
    ├── package.json
    ├── server.js
    ├── routes/
    │   └── todos.js
    └── models/
        └── Todo.js
```

### Schritt 2: Frontendcode implementieren

#### frontend/Dockerfile
```dockerfile
FROM node:18-alpine

WORKDIR /app

COPY package*.json ./

RUN npm install

COPY . .

EXPOSE 3000

CMD ["npm", "start"]
```

#### frontend/package.json
```json
{
  "name": "todo-frontend",
  "version": "0.1.0",
  "private": true,
  "dependencies": {
    "@testing-library/jest-dom": "^5.16.5",
    "@testing-library/react": "^13.4.0",
    "@testing-library/user-event": "^13.5.0",
    "axios": "^1.3.4",
    "react": "^18.2.0",
    "react-dom": "^18.2.0",
    "react-scripts": "5.0.1",
    "web-vitals": "^2.1.4"
  },
  "scripts": {
    "start": "react-scripts start",
    "build": "react-scripts build",
    "test": "react-scripts test",
    "eject": "react-scripts eject"
  },
  "eslintConfig": {
    "extends": [
      "react-app",
      "react-app/jest"
    ]
  },
  "browserslist": {
    "production": [
      ">0.2%",
      "not dead",
      "not op_mini all"
    ],
    "development": [
      "last 1 chrome version",
      "last 1 firefox version",
      "last 1 safari version"
    ]
  }
}
```

#### frontend/public/index.html
```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="utf-8" />
    <link rel="icon" href="%PUBLIC_URL%/favicon.ico" />
    <meta name="viewport" content="width=device-width, initial-scale=1" />
    <meta name="theme-color" content="#000000" />
    <meta
      name="description"
      content="ToDo App - Eine einfache ToDo-Anwendung"
    />
    <link rel="manifest" href="%PUBLIC_URL%/manifest.json" />
    <title>ToDo App</title>
  </head>
  <body>
    <noscript>You need to enable JavaScript to run this app.</noscript>
    <div id="root"></div>
  </body>
</html>
```

#### frontend/public/manifest.json
```json
{
  "short_name": "ToDo App",
  "name": "ToDo App - Eine einfache ToDo-Anwendung",
  "icons": [
    {
      "src": "favicon.ico",
      "sizes": "64x64 32x32 24x24 16x16",
      "type": "image/x-icon"
    }
  ],
  "start_url": ".",
  "display": "standalone",
  "theme_color": "#000000",
  "background_color": "#ffffff"
}
```

#### frontend/src/index.js
```javascript
import React from 'react';
import ReactDOM from 'react-dom/client';
import './index.css';
import App from './App';

const root = ReactDOM.createRoot(document.getElementById('root'));
root.render(
  <React.StrictMode>
    <App />
  </React.StrictMode>
);
```

#### frontend/src/index.css
```css
body {
  margin: 0;
  font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', 'Roboto', 'Oxygen',
    'Ubuntu', 'Cantarell', 'Fira Sans', 'Droid Sans', 'Helvetica Neue',
    sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  background-color: #f5f5f5;
}

code {
  font-family: source-code-pro, Menlo, Monaco, Consolas, 'Courier New',
    monospace;
}
```

#### frontend/src/App.css
```css
.container {
  max-width: 600px;
  margin: 0 auto;
  padding: 20px;
}

h1 {
  text-align: center;
  color: #333;
}

.todo-form {
  display: flex;
  margin-bottom: 20px;
}

.todo-input {
  flex: 1;
  padding: 10px;
  font-size: 16px;
  border: 1px solid #ddd;
  border-radius: 4px 0 0 4px;
}

.todo-button {
  padding: 10px 20px;
  background-color: #4caf50;
  color: white;
  font-size: 16px;
  border: none;
  border-radius: 0 4px 4px 0;
  cursor: pointer;
}

.todo-button:hover {
  background-color: #45a049;
}

.todo-list {
  list-style-type: none;
  padding: 0;
}

.todo-item {
  display: flex;
  align-items: center;
  padding: 10px;
  margin-bottom: 10px;
  background-color: white;
  border-radius: 4px;
  box-shadow: 0 1px 3px rgba(0, 0, 0, 0.1);
}

.todo-checkbox {
  margin-right: 10px;
}

.todo-text {
  flex: 1;
  font-size: 16px;
}

.todo-text.completed {
  text-decoration: line-through;
  color: #888;
}

.todo-delete {
  padding: 5px 10px;
  background-color: #f44336;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
}

.todo-delete:hover {
  background-color: #d32f2f;
}
```

#### frontend/src/App.js
```javascript
import React, { useState, useEffect } from 'react';
import axios from 'axios';
import './App.css';

function App() {
  const [todos, setTodos] = useState([]);
  const [newTodo, setNewTodo] = useState('');
  const API_URL = process.env.REACT_APP_API_URL || 'http://localhost:4000';

  useEffect(() => {
    fetchTodos();
  }, []);

  const fetchTodos = async () => {
    try {
      const response = await axios.get(`${API_URL}/api/todos`);
      setTodos(response.data);
    } catch (error) {
      console.error('Error fetching todos:', error);
    }
  };

  const addTodo = async (e) => {
    e.preventDefault();
    if (!newTodo.trim()) return;

    try {
      const response = await axios.post(`${API_URL}/api/todos`, {
        text: newTodo,
        completed: false
      });
      setTodos([...todos, response.data]);
      setNewTodo('');
    } catch (error) {
      console.error('Error adding todo:', error);
    }
  };

  const toggleComplete = async (id, completed) => {
    try {
      const response = await axios.put(`${API_URL}/api/todos/${id}`, {
        completed: !completed
      });
      
      setTodos(todos.map(todo => 
        todo._id === id ? { ...todo, completed: !todo.completed } : todo
      ));
    } catch (error) {
      console.error('Error updating todo:', error);
    }
  };

  const deleteTodo = async (id) => {
    try {
      await axios.delete(`${API_URL}/api/todos/${id}`);
      setTodos(todos.filter(todo => todo._id !== id));
    } catch (error) {
      console.error('Error deleting todo:', error);
    }
  };

  return (
    <div className="container">
      <h1>ToDo App</h1>
      
      <form className="todo-form" onSubmit={addTodo}>
        <input
          type="text"
          className="todo-input"
          value={newTodo}
          onChange={(e) => setNewTodo(e.target.value)}
          placeholder="Was muss erledigt werden?"
        />
        <button type="submit" className="todo-button">Hinzufügen</button>
      </form>
      
      <ul className="todo-list">
        {todos.map((todo) => (
          <li key={todo._id} className="todo-item">
            <input
              type="checkbox"
              className="todo-checkbox"
              checked={todo.completed}
              onChange={() => toggleComplete(todo._id, todo.completed)}
            />
            <span className={`todo-text ${todo.completed ? 'completed' : ''}`}>
              {todo.text}
            </span>
            <button
              className="todo-delete"
              onClick={() => deleteTodo(todo._id)}
            >
              Löschen
            </button>
          </li>
        ))}
      </ul>
    </div>
  );
}

export default App;
```

### Schritt 3: Backendcode implementieren

#### backend/Dockerfile
```dockerfile
FROM node:18-alpine

WORKDIR /app

COPY package*.json ./

RUN npm install

COPY . .

EXPOSE 4000

CMD ["npm", "start"]
```

#### backend/package.json
```json
{
  "name": "todo-backend",
  "version": "1.0.0",
  "description": "Backend für die ToDo App",
  "main": "server.js",
  "scripts": {
    "start": "node server.js",
    "dev": "nodemon server.js"
  },
  "dependencies": {
    "cors": "^2.8.5",
    "express": "^4.18.2",
    "mongoose": "^7.0.3"
  },
  "devDependencies": {
    "nodemon": "^2.0.22"
  }
}
```

#### backend/server.js
```javascript
const express = require('express');
const mongoose = require('mongoose');
const cors = require('cors');
const todosRoutes = require('./routes/todos');

// Erstelle Express-App
const app = express();
const PORT = process.env.PORT || 4000;
const MONGODB_URI = process.env.MONGODB_URI || 'mongodb://localhost:27017/todoapp';

// Middleware
app.use(cors());
app.use(express.json());

// Verbinde mit MongoDB
mongoose.connect(MONGODB_URI)
  .then(() => console.log('MongoDB verbunden'))
  .catch(err => console.error('MongoDB Verbindungsfehler:', err));

// Routen
app.use('/api/todos', todosRoutes);

// Startseite
app.get('/', (req, res) => {
  res.send('ToDo-API ist aktiv');
});

// Starte Server
app.listen(PORT, () => {
  console.log(`Server läuft auf Port ${PORT}`);
});
```

#### backend/models/Todo.js
```javascript
const mongoose = require('mongoose');

const TodoSchema = new mongoose.Schema({
  text: {
    type: String,
    required: true,
    trim: true
  },
  completed: {
    type: Boolean,
    default: false
  },
  createdAt: {
    type: Date,
    default: Date.now
  }
});

module.exports = mongoose.model('Todo', TodoSchema);
```

#### backend/routes/todos.js
```javascript
const express = require('express');
const router = express.Router();
const Todo = require('../models/Todo');

// GET /api/todos - Alle ToDos abrufen
router.get('/', async (req, res) => {
  try {
    const todos = await Todo.find().sort({ createdAt: -1 });
    res.json(todos);
  } catch (err) {
    res.status(500).json({ message: err.message });
  }
});

// POST /api/todos - Neuen ToDo erstellen
router.post('/', async (req, res) => {
  const todo = new Todo({
    text: req.body.text,
    completed: req.body.completed || false
  });

  try {
    const newTodo = await todo.save();
    res.status(201).json(newTodo);
  } catch (err) {
    res.status(400).json({ message: err.message });
  }
});

// PUT /api/todos/:id - ToDo aktualisieren
router.put('/:id', async (req, res) => {
  try {
    const todo = await Todo.findById(req.params.id);
    
    if (!todo) {
      return res.status(404).json({ message: 'ToDo nicht gefunden' });
    }
    
    if (req.body.text !== undefined) {
      todo.text = req.body.text;
    }
    
    if (req.body.completed !== undefined) {
      todo.completed = req.body.completed;
    }
    
    const updatedTodo = await todo.save();
    res.json(updatedTodo);
  } catch (err) {
    res.status(500).json({ message: err.message });
  }
});

// DELETE /api/todos/:id - ToDo löschen
router.delete('/:id', async (req, res) => {
  try {
    const todo = await Todo.findById(req.params.id);
    
    if (!todo) {
      return res.status(404).json({ message: 'ToDo nicht gefunden' });
    }
    
    await todo.deleteOne();
    res.json({ message: 'ToDo gelöscht' });
  } catch (err) {
    res.status(500).json({ message: err.message });
  }
});

module.exports = router;
```

### Schritt 4: Docker Compose starten

1. Kopiere die `docker-compose.yml` in das Hauptverzeichnis.
2. Führe folgenden Befehl aus, um die Anwendung zu starten:

```bash
docker-compose up --build
```

Die Anwendung sollte nun unter folgenden URLs verfügbar sein:
- Frontend: http://localhost:3000
- Backend-API: http://localhost:4000
- MongoDB: mongodb://localhost:27017/todoapp

## Aufgabe

1. Erstelle die Projektstruktur mit allen oben genannten Dateien.
2. Starte die Anwendung mit Docker Compose.
3. Teste die ToDo-App, indem du:
   - Neue ToDo-Einträge hinzufügst
   - ToDo-Einträge als erledigt markierst
   - ToDo-Einträge löschst

## Erklärung: Wie funktioniert die Anwendung?

### 1. Docker Compose Orchestrierung

Die `docker-compose.yml` Datei orchestriert die drei Container:

- **Frontend**: Ein React-Container, der auf Port 3000 läuft
- **Backend**: Ein Node.js/Express-Container, der auf Port 4000 läuft
- **Datenbank**: Ein MongoDB-Container, der auf Port 27017 läuft

Die Container sind über Docker-Netzwerke miteinander verbunden. Das Frontend kann nur mit dem Backend kommunizieren, während das Backend sowohl mit dem Frontend als auch mit der Datenbank kommunizieren kann.

### 2. Volume-Mapping

Für eine bessere Entwicklungserfahrung werden die lokalen Verzeichnisse in die Container gemounted:

- Der gesamte `frontend`-Ordner wird in den Frontend-Container gemounted
- Der gesamte `backend`-Ordner wird in den Backend-Container gemounted
- Für die Datenbank wird ein persistentes Volume verwendet, um Daten dauerhaft zu speichern

### 3. Umgebungsvariablen

Umgebungsvariablen werden für die Container-Konfiguration verwendet:

- Das Frontend benutzt `REACT_APP_API_URL`, um zu wissen, wo das Backend zu finden ist
- Das Backend benutzt `MONGODB_URI`, um zu wissen, wo die Datenbank zu finden ist

### 4. Technologie-Stack

- **Frontend**: React mit Axios für HTTP-Anfragen
- **Backend**: Node.js, Express und Mongoose für MongoDB-Integration
- **Datenbank**: MongoDB

## Warum Docker Compose?

Docker Compose bietet folgende Vorteile für diese Anwendung:

1. **Isolation**: Jede Komponente läuft in ihrem eigenen Container.
2. **Portabilität**: Die Anwendung kann auf jedem System mit Docker ausgeführt werden.
3. **Konsistente Umgebung**: Alle Entwickler arbeiten mit der gleichen Umgebung.
4. **Einfache Verwaltung**: Alle Dienste können mit einem einzigen Befehl gestartet und gestoppt werden.
5. **Keine lokale Installation erforderlich**: MongoDB, Node.js und npm müssen nicht lokal installiert sein.

## Mögliche Erweiterungen

Nach erfolgreicher Implementierung könntest du folgende Erweiterungen vornehmen:

1. Benutzerauthentifizierung hinzufügen (z.B. mit JWT)
2. ToDo-Kategorien implementieren
3. Eine Suchfunktion hinzufügen
4. Fälligkeitsdaten für ToDos hinzufügen
5. Einen Nginx-Container als Reverse-Proxy einrichten
