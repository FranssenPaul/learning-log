# Classroom Pong - Minimal Setup Deep Dive

**Date:** 2026-02-01  
**Topic:** Understanding the Three-Interface Approval System Architecture

---

## 📋 Table of Contents

1. [Project Overview](#project-overview)
2. [React Router Setup](#react-router-setup)
3. [How HTML Gets to Your Phone (SPA Explained)](#how-html-gets-to-your-phone-spa-explained)
4. [The Three Interfaces](#the-three-interfaces)
5. [WebSocket Communication](#websocket-communication)
6. [Backend Logic (Under the Hood)](#backend-logic-under-the-hood)
7. [Complete Data Flow](#complete-data-flow)
8. [File Structure](#file-structure)

---

## Project Overview

### What We Built

A **minimal classroom gaming system** with three distinct interfaces:

1. **Teacher Dashboard** (`/admin`) - Control center for approving/rejecting students
2. **Display Screen** (`/display`) - Large screen showing student messages (projector)
3. **Student Phone** (`/player`) - Phone interface for joining and sending messages

### Core Concept

Students cannot freely join - they must request access and wait for teacher approval. Once approved, they can send messages that appear on the big display screen in real-time.

### Technology Stack

- **Frontend:** React + Vite + React Router
- **Backend:** FastAPI + Uvicorn
- **Communication:** WebSocket (real-time bidirectional)
- **Network:** Local WiFi (no internet required)

---

## React Router Setup

### What is React Router?

React Router enables **client-side routing** - changing URLs without page reloads. Instead of requesting new HTML from the server, it swaps React components in the browser.

### Our Configuration

**File:** `frontend/src/App.jsx`

```jsx
import { BrowserRouter as Router, Routes, Route, Link } from 'react-router-dom';
import Home from './pages/Home';
import TeacherDashboard from './pages/TeacherDashboard';
import SimpleDisplay from './pages/SimpleDisplay';
import SimplePlayer from './pages/SimplePlayer';

function App() {
  return (
    <Router>
      <div className="app">
        <nav>
          <Link to="/">🏠 Home</Link>
          <Link to="/admin">🎓 Dashboard</Link>
          <Link to="/display">📺 Display</Link>
        </nav>
        
        <Routes>
          <Route path="/" element={<Home />} />
          <Route path="/admin" element={<TeacherDashboard />} />
          <Route path="/display" element={<SimpleDisplay />} />
          <Route path="/player" element={<SimplePlayer />} />
        </Routes>
      </div>
    </Router>
  );
}
```

### How It Works

#### 1. **BrowserRouter** (aliased as `Router`)
- Wraps the entire app
- Listens to URL changes
- Keeps UI in sync with URL

#### 2. **Routes** and **Route**
- `<Routes>` container holds all route definitions
- `<Route path="/admin" element={<TeacherDashboard />} />` means:
  - When URL is `/admin`, render `<TeacherDashboard />`
  - Path matching is exact by default

#### 3. **Link**
- `<Link to="/admin">` creates an `<a>` tag
- **BUT:** Prevents default browser navigation
- Instead: Updates URL and tells Router to swap components
- **Result:** Instant, no page reload!

### Why Client-Side Routing?

**Traditional (Server-Side):**
```
User clicks link → Browser requests /admin → Server sends HTML → Full page reload
```

**Client-Side (React Router):**
```
User clicks link → React Router updates URL → Swaps component → Instant!
```

**Benefits:**
- Faster (no network request)
- Preserves app state
- Smoother user experience
- Still works with browser back/forward buttons

---

## How HTML Gets to Your Phone (SPA Explained)

### The Question

When your phone requests `http://192.168.0.100:5173/player`, how does it get the right HTML?

### The Answer: It Doesn't!

**Every URL gets the SAME HTML file.** Then JavaScript decides what to show.

### Step-by-Step Flow

#### 1. Phone Makes HTTP Request

```
Phone Browser → "GET /player HTTP/1.1" → Vite Server (192.168.0.100:5173)
```

#### 2. Vite Responds with index.html

Vite **doesn't** look for a `/player.html` file. It **always** sends:

**File:** `frontend/index.html`
```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Classroom Pong</title>
  </head>
  <body>
    <div id="root"></div>
    <script type="module" src="/src/main.jsx"></script>
  </body>
</html>
```

**Key points:**
- Empty `<div id="root"></div>` - React will fill this
- `<script src="/src/main.jsx">` - Loads the React app

#### 3. Browser Downloads JavaScript

The phone's browser sees the script tag and requests:
```
GET /src/main.jsx
GET /src/App.jsx
GET /src/pages/SimplePlayer.jsx
GET /node_modules/react/... (React library)
GET /node_modules/react-router-dom/... (React Router library)
```

Vite bundles and sends all these JavaScript files.

#### 4. React Starts

**File:** `frontend/src/main.jsx`
```jsx
import React from 'react'
import ReactDOM from 'react-dom/client'
import App from './App.jsx'

ReactDOM.createRoot(document.getElementById('root')).render(
  <React.StrictMode>
    <App />
  </React.StrictMode>
)
```

**What this does:**
1. `document.getElementById('root')` - Finds the empty div
2. `ReactDOM.createRoot()` - Tells React to control this div
3. `.render(<App />)` - Renders the App component inside the div

#### 5. React Router Reads the URL

**File:** `frontend/src/App.jsx`
```jsx
function App() {
  return (
    <Router>
      <Routes>
        <Route path="/" element={<Home />} />
        <Route path="/admin" element={<TeacherDashboard />} />
        <Route path="/display" element={<SimpleDisplay />} />
        <Route path="/player" element={<SimplePlayer />} />
      </Routes>
    </Router>
  );
}
```

**React Router's logic:**
```javascript
const currentURL = window.location.pathname;  // "/player"

if (currentURL === "/player") {
  renderComponent(<SimplePlayer />);
}
```

#### 6. SimplePlayer Component Renders

**File:** `frontend/src/pages/SimplePlayer.jsx`
```jsx
function SimplePlayer() {
  return (
    <div className="player-container join">
      <h1>📱 Join Game</h1>
      <input placeholder="Enter your name" />
      <button>Join</button>
    </div>
  );
}
```

**React converts this to:**
```html
<div class="player-container join">
  <h1>📱 Join Game</h1>
  <input placeholder="Enter your name" />
  <button>Join</button>
</div>
```

And inserts it into `<div id="root">`.

#### 7. Final HTML in Phone Browser

```html
<!DOCTYPE html>
<html>
  <head>
    <title>Classroom Pong</title>
    <link rel="stylesheet" href="/src/pages/SimplePlayer.css">
  </head>
  <body>
    <div id="root">
      <!-- React rendered this: -->
      <div class="player-container join">
        <h1>📱 Join Game</h1>
        <input placeholder="Enter your name" />
        <button>Join</button>
      </div>
    </div>
    <script src="...react bundle..."></script>
  </body>
</html>
```

### The SPA Pattern

**Single Page Application (SPA)** means:

**Traditional Multi-Page App:**
```
Browser requests /player → Server sends player.html
Browser requests /admin → Server sends admin.html
Browser requests /display → Server sends display.html
```

**Single Page App (React):**
```
Browser requests /player → Server sends index.html
Browser requests /admin → Server sends index.html (same file!)
Browser requests /display → Server sends index.html (same file!)

Then JavaScript looks at URL and shows different content!
```

### Why SPA?

**Advantages:**
1. **Faster navigation** - No full page reload when switching routes
2. **Smoother UX** - Instant transitions, app stays running
3. **Shared state** - Data persists as you navigate
4. **Code reuse** - Components can be shared across routes

**How it feels:**
- Traditional: Click link → White screen → New page loads (slow)
- SPA: Click link → Component swaps instantly (fast!)

### URL Changes Without Requests

When you click a React Router `<Link>`:

```jsx
<Link to="/display">Display</Link>
```

**What happens:**
1. Link intercepts the click (prevents default browser navigation)
2. Uses browser's History API: `window.history.pushState(null, '', '/display')`
3. URL changes to `/display` BUT no HTTP request!
4. React Router sees URL changed, renders `<SimpleDisplay />`

**Result:** URL changes, content changes, but no network request! ⚡

### Vite's Role in Development

**Vite (dev server) is smart:**

```
ANY request for a path → Return index.html
```

Examples:
- `GET /` → index.html
- `GET /player` → index.html
- `GET /admin` → index.html
- `GET /some/random/path` → index.html

Then React Router handles the routing **in the browser**.

### Production Build

In production (after `npm run build`):

```bash
npm run build
# Creates: frontend/dist/index.html
```

Your backend server (FastAPI) needs to do the same thing:

```python
from fastapi import FastAPI
from fastapi.staticfiles import StaticFiles
from starlette.responses import FileResponse

app = FastAPI()

# Serve static files
app.mount("/assets", StaticFiles(directory="dist/assets"), name="assets")

# Catch-all route: return index.html for any path
@app.get("/{full_path:path}")
async def serve_spa(full_path: str):
    return FileResponse('dist/index.html')
```

**Why catch-all?**
- User might refresh page on `/player`
- Server needs to return index.html
- React Router takes over and shows the right component

### Summary: The Journey

```
1. Phone types: http://192.168.0.100:5173/player
2. HTTP GET request → Vite server
3. Vite responds: index.html
4. Browser downloads JavaScript bundles
5. React starts, renders App component
6. React Router sees URL is "/player"
7. React Router renders <SimplePlayer />
8. Component appears on screen
9. WebSocket connects to backend
10. Student can join the game!
```

All of this happens in **under 1 second**! 🚀

---

## The Three Interfaces

### 1. Teacher Dashboard (`/admin`)

**File:** `frontend/src/pages/TeacherDashboard.jsx`

#### Purpose
Teacher control center - approve/reject student connections and monitor activity.

#### Key Features

**Pending Approvals Section:**
```jsx
{pending.map(name => (
  <div key={name}>
    <strong>{name} wants to join</strong>
    <button onClick={() => approvePlayer(name)}>✅ Approve</button>
    <button onClick={() => rejectPlayer(name)}>❌ Reject</button>
  </div>
))}
```

**Connected Players List:**
```jsx
{players.map(name => (
  <li key={name}>🟢 {name}</li>
))}
```

**Messages Log:**
```jsx
{messages.map((msg, idx) => (
  <div key={idx}>
    <strong>{msg.player}</strong> ({msg.time})
    <div>{msg.message}</div>
  </div>
))}
```

#### State Management
```jsx
const [players, setPlayers] = useState([]);      // Approved players
const [pending, setPending] = useState([]);      // Waiting for approval
const [messages, setMessages] = useState([]);    // All messages
const wsRef = useRef(null);                      // WebSocket connection
```

**Why `useRef` for WebSocket?**
- WebSocket is NOT React state - it's an external connection
- `useRef` persists across re-renders without triggering them
- Perfect for storing connection objects, timers, DOM references

---

### 2. Display Screen (`/display`)

**File:** `frontend/src/pages/SimpleDisplay.jsx`

#### Purpose
Large screen (projector) showing student messages in real-time.

#### Key Features

**Full-Screen Design:**
```css
.display-container {
  min-height: 100vh;
  background-color: #1a1a1a;
  color: white;
  font-size: 3rem;  /* Large for visibility */
}
```

**Message Animation:**
```jsx
messages.slice(-5).reverse().map((msg, idx) => (
  <div key={messages.length - idx} className="message-card">
    <div className="message-player">{msg.player}</div>
    <div className="message-text">{msg.message}</div>
    <div className="message-timestamp">{msg.time}</div>
  </div>
))
```

- `slice(-5)` → Show only last 5 messages
- `reverse()` → Newest first
- CSS animation on `.message-card` for fade-in effect

#### Passive Interface
- Display screen only **receives** data
- No user interaction needed
- WebSocket connection stays open, listening for messages

---

### 3. Student Phone (`/player`)

**File:** `frontend/src/pages/SimplePlayer.jsx`

#### Purpose
Phone interface for students to join and send messages.

#### State Machine Design

The player interface has **4 states**:

```jsx
const [status, setStatus] = useState('disconnected');
// Possible values: 'connected', 'waiting', 'approved', 'rejected'
```

**State Flow:**
```
disconnected → connected → waiting → approved (or rejected)
```

#### State 1: Connected (Join Screen)
```jsx
if (status === 'connected') {
  return (
    <form onSubmit={handleJoin}>
      <input 
        value={name}
        onChange={(e) => setName(e.target.value)}
        placeholder="Enter your name"
      />
      <button>Join</button>
    </form>
  );
}
```

**What happens on submit:**
```jsx
const handleJoin = (e) => {
  e.preventDefault();
  wsRef.current.send(JSON.stringify({
    type: 'join',
    name: name.trim()
  }));
};
```

#### State 2: Waiting (Approval Pending)
```jsx
if (status === 'waiting') {
  return (
    <div>
      <h1>⏳ Waiting for Approval</h1>
      <p>Teacher needs to approve your connection...</p>
    </div>
  );
}
```

#### State 3: Approved (Active)
```jsx
if (status === 'approved') {
  return (
    <form onSubmit={handleSendMessage}>
      <textarea 
        value={message}
        onChange={(e) => setMessage(e.target.value)}
        placeholder="Type your message..."
      />
      <button>📤 Send to Display</button>
    </form>
  );
}
```

**Sending messages:**
```jsx
const handleSendMessage = (e) => {
  e.preventDefault();
  wsRef.current.send(JSON.stringify({
    type: 'message',
    message: message.trim()
  }));
  setMessage('');  // Clear input
};
```

#### State 4: Rejected
```jsx
if (status === 'rejected') {
  return (
    <div>
      <h1>❌ Connection Rejected</h1>
      <p>Please ask your teacher for permission.</p>
    </div>
  );
}
```

---

## WebSocket Communication

### What is WebSocket?

**HTTP (Traditional):**
- Client sends request → Server responds → Connection closes
- For updates, client must keep asking (polling)

**WebSocket:**
- Client and server open a **persistent connection**
- **Bidirectional:** Both can send messages anytime
- Perfect for real-time apps (chat, games, live updates)

### Our WebSocket Setup

#### Backend Endpoints

**File:** `backend/main.py`

```python
@app.websocket("/ws/admin")
async def admin_websocket(websocket: WebSocket):
    await websocket.accept()
    state.admin_ws = websocket
    # Connection stays open, listening for messages

@app.websocket("/ws/display")
async def display_websocket(websocket: WebSocket):
    await websocket.accept()
    state.display_ws = websocket
    # Connection stays open

@app.websocket("/ws/player")
async def player_websocket(websocket: WebSocket):
    await websocket.accept()
    # Each player gets their own connection
```

#### Frontend Connection Pattern

**Every interface follows this pattern:**

```jsx
useEffect(() => {
  // 1. Build WebSocket URL
  const wsProtocol = window.location.protocol === 'https:' ? 'wss:' : 'ws:';
  const wsHost = window.location.hostname;
  const websocket = new WebSocket(`${wsProtocol}//${wsHost}:8000/ws/admin`);
  
  // 2. Handle connection opened
  websocket.onopen = () => {
    console.log('Connected');
  };

  // 3. Handle incoming messages
  websocket.onmessage = (event) => {
    const data = JSON.parse(event.data);
    
    if (data.type === 'join_request') {
      setPending(prev => [...prev, data.playerName]);
    }
  };

  // 4. Store reference
  wsRef.current = websocket;

  // 5. Cleanup on unmount
  return () => websocket.close();
}, []);  // Empty array = run once on mount
```

**What does the `return` do here?**  
In `useEffect`, the value you return is a **cleanup function**, not a normal return value. React will call it when the component **unmounts** (or before the effect re-runs). With an empty dependency array `[]`, this cleanup runs **only on unmount**, so the WebSocket closes when the component is removed — not immediately.

### Why Dynamic URL Construction?

```jsx
const wsHost = window.location.hostname;
const websocket = new WebSocket(`ws://${wsHost}:8000/ws/admin`);
```

**Problem:** Hardcoding `ws://192.168.0.100:8000` won't work on different devices!

**Solution:**
- On PC: `window.location.hostname` = `localhost`
- On Phone: `window.location.hostname` = `192.168.0.100`
- WebSocket uses the **same IP** the page loaded from

**Result:** Code works on all devices automatically! 🎯

### Clarifying Host, wsHost, and Sockets

**What is "host" here?**  
`window.location.hostname` comes from the **URL in the browser**, so it points to the **server address** that served the page (not the client device).

**Can `wsHost` change?**  
Yes. It follows how the page was opened:
- `http://localhost:5173` → `wsHost = "localhost"`
- `http://192.168.0.100:5173` → `wsHost = "192.168.0.100"`
- `https://myapp.com` → `wsHost = "myapp.com"`

**Why does the WebSocket need that address?**  
`new WebSocket("ws://...")` must know **which server to connect to**. If the host is wrong, the connection fails.

**What is a socket (and is it a port)?**  
A **port** is just a number (like `8000`).  
A **socket** is the **full endpoint**: **IP/host + port + protocol**.  
So a socket **uses** a port, but it's more than just the port.

**What is a "service"?**  
A **service** is a program running on a machine that listens for requests and does a specific job (e.g., web server, WebSocket server, database, SSH).  
A **port** tells the machine **which service** should receive the incoming connection.

**Example in this project:**  
- `http://192.168.0.100:5173` → Vite dev server serving the React app (web server service).  
- `ws://192.168.0.100:8000/ws/admin` → FastAPI WebSocket server handling real-time messages (WebSocket service).  
Same machine, different **services**, separated by **ports**.

### Message Format (JSON)

All WebSocket messages use JSON structure:

```javascript
// Outgoing (Client → Server)
{
  "type": "join",
  "name": "Alice"
}

{
  "type": "message",
  "message": "Hello class!"
}

// Incoming (Server → Client)
{
  "type": "join_request",
  "playerName": "Alice"
}

{
  "type": "approved"
}

{
  "type": "player_message",
  "playerName": "Alice",
  "message": "Hello class!"
}
```

---

## Backend Logic (Under the Hood)

### State Management

**File:** `backend/main.py`

```python
class AppState:
    def __init__(self):
        self.admin_ws: Optional[WebSocket] = None
        self.display_ws: Optional[WebSocket] = None
        self.players: Dict[str, dict] = {}
        self.pending_players: Dict[str, WebSocket] = {}

state = AppState()
```

**Why a global state object?**
- WebSocket handlers are **async functions** - they run independently
- Need shared memory to coordinate between:
  - Admin approving a player
  - Player waiting for approval
  - Display showing messages
- Python dict is thread-safe for basic operations in asyncio

### The Approval Flow (Step-by-Step)

#### Step 1: Student Joins

**Phone sends:**
```json
{"type": "join", "name": "Alice"}
```

**Backend receives in `/ws/player`:**
```python
data = await websocket.receive_json()

if data["type"] == "join":
    player_name = data["name"]
    state.pending_players[player_name] = websocket
    
    # Notify admin
    if state.admin_ws:
        await state.admin_ws.send_json({
            "type": "join_request",
            "playerName": player_name
        })
    
    # Tell player to wait
    await websocket.send_json({"type": "waiting_approval"})
```

**Result:**
- Alice's WebSocket stored in `pending_players`
- Admin dashboard receives notification
- Alice's phone shows "Waiting..." screen

#### Step 2: Teacher Approves

**Dashboard sends:**
```json
{"type": "approve", "playerName": "Alice"}
```

**Backend receives in `/ws/admin`:**
```python
if data["type"] == "approve":
    player_name = data["playerName"]
    
    # Move from pending to active
    if player_name in state.pending_players:
        player_ws = state.pending_players.pop(player_name)
        state.players[player_name] = {
            "ws": player_ws,
            "approved": True,
            "messages": []
        }
        
        # Tell player they're approved
        await player_ws.send_json({"type": "approved"})
        
        # Update admin UI
        await websocket.send_json({
            "type": "player_approved",
            "playerName": player_name
        })
```

**Result:**
- Alice moved from `pending_players` to `players`
- Alice's phone receives approval → screen turns green
- Dashboard updates to show Alice in connected list

#### Step 3: Student Sends Message

**Phone sends:**
```json
{"type": "message", "message": "Hello class!"}
```

**Backend receives in `/ws/player`:**
```python
if data["type"] == "message":
    if player_name and player_name in state.players:
        message = data["message"]
        
        # Store message
        state.players[player_name]["messages"].append(message)
        
        # Send to display
        if state.display_ws:
            await state.display_ws.send_json({
                "type": "player_message",
                "playerName": player_name,
                "message": message
            })
        
        # Notify admin
        if state.admin_ws:
            await state.admin_ws.send_json({
                "type": "player_message",
                "playerName": player_name,
                "message": message
            })
```

**Result:**
- Message stored in backend
- Display screen shows message instantly
- Dashboard logs the message

### Connection Cleanup

**When WebSocket disconnects:**
```python
except WebSocketDisconnect:
    if player_name:
        state.pending_players.pop(player_name, None)
        state.players.pop(player_name, None)
```

**Why cleanup matters:**
- Prevents memory leaks
- Removes disconnected players from lists
- Teacher dashboard shows accurate connected count

---

## Complete Data Flow

### Scenario: Alice Joins and Sends "Hello!"

```
┌─────────┐         ┌─────────┐         ┌─────────┐
│  Alice  │         │ Backend │         │ Teacher │
│  Phone  │         │ Server  │         │Dashboard│
└────┬────┘         └────┬────┘         └────┬────┘
     │                   │                   │
     │  1. {join, Alice} │                   │
     ├──────────────────>│                   │
     │                   │                   │
     │                   │  2. {join_request}│
     │                   ├──────────────────>│
     │                   │                   │
     │ 3. {waiting}      │                   │
     │<──────────────────┤                   │
     │                   │                   │
     │                   │  4. {approve}     │
     │                   │<──────────────────┤
     │                   │                   │
     │ 5. {approved}     │                   │
     │<──────────────────┤                   │
     │                   │                   │
     │ 6. {msg, "Hello"} │                   │
     ├──────────────────>│                   │
     │                   │                   │
     │                   ├─────────┐         │
     │                   │ 7. Broadcast      │
     │                   │<────────┘         │
     │                   │                   │
     │                   │ 8. {player_msg}   │
     │                   ├──────────────────>│
     │                   │                   │
     │                   │ 9. {player_msg}   │
     │                   ├──────────────────>│ Display
     │                   │                   │
```

### Flow Breakdown

1. **Alice joins** - Phone sends join request with name
2. **Backend notifies teacher** - Admin dashboard shows pending approval
3. **Alice waits** - Phone shows "Waiting for approval" screen
4. **Teacher approves** - Dashboard sends approval to backend
5. **Backend confirms** - Sends approval to Alice's phone
6. **Alice sends message** - Phone sends message content
7. **Backend broadcasts** - Prepares to send to all listeners
8. **Admin receives** - Dashboard logs the message
9. **Display receives** - Big screen shows the message

### Key Insight: The Backend is the Hub

```
     Dashboard
        ↑ ↓
        │ │
    Backend  ←──→  Display
        │ │
        ↑ ↓
      Player
```

- All communication goes **through** the backend
- Players never talk directly to Display
- Backend controls **who** can send **what** to **whom**

---

## File Structure

### Backend

```
backend/
├── main.py              # FastAPI app with WebSocket endpoints
└── pyproject.toml       # Dependencies (fastapi, uvicorn)
```

**Key backend components:**
```python
# main.py
from fastapi import FastAPI, WebSocket, WebSocketDisconnect
from fastapi.middleware.cors import CORSMiddleware

app = FastAPI()

# CORS allows frontend on port 5173 to connect to backend on port 8000
app.add_middleware(CORSMiddleware, allow_origins=["*"])

# State management
class AppState:
    admin_ws: Optional[WebSocket]      # One admin
    display_ws: Optional[WebSocket]    # One display
    players: Dict[str, dict]           # Multiple players
    pending_players: Dict[str, WebSocket]

# WebSocket endpoints
@app.websocket("/ws/admin")
@app.websocket("/ws/display")
@app.websocket("/ws/player")
```

### Frontend

```
frontend/
├── src/
│   ├── App.jsx                      # React Router setup
│   ├── pages/
│   │   ├── Home.jsx                 # Landing page
│   │   ├── Home.css
│   │   ├── TeacherDashboard.jsx     # Admin interface
│   │   ├── TeacherDashboard.css
│   │   ├── SimpleDisplay.jsx        # Display screen
│   │   ├── SimpleDisplay.css
│   │   ├── SimplePlayer.jsx         # Phone interface
│   │   └── SimplePlayer.css
│   └── App.css
└── package.json
```

**Design pattern:**
- Each page = one `.jsx` file + one `.css` file
- Separation of concerns (logic vs styles)
- Easy to find and modify specific interfaces

---

## Development vs Production

### Development (Current Setup)

**Two servers running:**

```bash
# Terminal 1: Backend (port 8000)
cd backend
uv run uvicorn main:app --reload --host 0.0.0.0

# Terminal 2: Frontend (port 5173)
cd frontend
npm run dev -- --host 0.0.0.0
```

**Why two servers?**
- Vite (frontend) has hot reload - instant updates when you edit code
- Uvicorn (backend) also has `--reload` for instant backend updates
- Faster development experience

**URLs in development:**
- Frontend: `http://localhost:5173`
- Backend: `http://localhost:8000`
- WebSocket: `ws://localhost:8000/ws/...`

### Production (Future)

**One server:**

```bash
# Build React app
cd frontend
npm run build
# Creates: frontend/dist/ folder with optimized static files

# Backend serves everything
cd backend
uv run uvicorn main:app --host 0.0.0.0
```

**FastAPI serves static files:**
```python
from fastapi.staticfiles import StaticFiles

app.mount("/", StaticFiles(directory="../frontend/dist", html=True))
```

**Result:**
- Single port (8000)
- Simpler deployment
- Backend serves React files + handles WebSockets

---

## Key Concepts Summary

### 1. Client-Side Routing (React Router)
- URL changes without page reloads
- Components swap instantly
- Better user experience

### 2. WebSocket Communication
- Persistent bidirectional connection
- Real-time updates without polling
- Perfect for multiplayer/collaborative apps

### 3. State Machine Design (Player Interface)
- Clear states: connected → waiting → approved/rejected
- Each state has specific UI
- Prevents invalid actions (can't send message before approval)

### 4. Centralized State (Backend)
- Backend holds all connections
- Coordinates communication between interfaces
- Single source of truth

### 5. Dynamic Configuration
- `window.location.hostname` for cross-device compatibility
- Same code works on PC and phone
- No hardcoded IPs

---

## What's Next?

Now that you understand the minimal setup, you can:

1. **Add games** - Replace message sending with game controls
2. **Add features:**
   - Kick players
   - Chat between students
   - Game selection menu
   - Score tracking
3. **Improve UI** - Better styling, animations, sounds
4. **Production deployment** - Single-server setup, maybe on Raspberry Pi

The core architecture (3 interfaces, WebSocket, approval system) stays the same! 🚀

---

## Testing Checklist

- [ ] Backend running on port 8000
- [ ] Frontend running on port 5173
- [ ] Dashboard shows pending approvals
- [ ] Phone can join and wait
- [ ] Teacher can approve/reject
- [ ] Approved phone can send messages
- [ ] Display shows messages in real-time
- [ ] Dashboard logs all activity

---

**End of Document** 📚
