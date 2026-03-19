# Masterclass 1 — The Face
## Component Design & UI Logic with React

---

## Slide 1 — Welcome & What We're Building

**classMentor AI** is the full application we are going to build together across 4 masterclasses.

It is a **ChatGPT-like AI chat application** — meaning users can log in, have conversations with an AI, switch between multiple chats, and even speak to it using a microphone. Let's understand what it does at a high level before we write a single line of code:

- 🧠 **Real AI responses** powered by Groq (runs the Llama 3.1 model) — the AI actually understands your questions and responds intelligently
- 🎙️ **Voice input** with Whisper speech-to-text — you can speak into the mic and your words get converted to text automatically
- 🌗 **Light/Dark theme** that persists — when you switch to dark mode and refresh the page, it remembers your choice
- 🔐 **Full auth system** — users can register, log in, reset their password via email, and delete their account
- 💬 **Multi-chat sidebar** like ChatGPT — you can have many separate conversations, search through them, and delete old ones

**Today's focus → THE FACE**
> We are building everything the user sees and touches — the React frontend with all its pages, reusable components, and global state management. No backend today, we'll stub that in later.

---

## Slide 2 — The 4 Masterclass Journey

Think of the full application as having 4 distinct layers. We build one layer per masterclass:

| # | Name | What We Build |
|---|------|---------------|
| **1** | **The Face** ← *You are here* | React pages, components, global state, routing |
| 2 | The Brain | Express server, REST APIs, Groq AI integration |
| 3 | The Memory | MongoDB database, Mongoose models, data persistence |
| 4 | The Launch | Cloud deployment on Vercel and Render |

After completing each masterclass, we tag the code in Git so students can travel back to any specific point in time.

---

## Slide 3 — Tech Stack for Today

Before we start coding, let's understand each tool we'll be using and exactly why we're using it:

| Tool | Role | Why we chose it |
|------|------|-----------------|
| **React 18** | UI framework | Industry standard for building component-based UIs; large ecosystem and community |
| **Vite 5** | Dev server + build tool | Extremely fast Hot Module Replacement (HMR) — changes appear in browser instantly without full reload |
| **React Router v6** | Client-side navigation | Handles URL changes without full page refreshes; supports protected/nested routes cleanly |
| **Axios** | HTTP requests | A promise-based HTTP client with powerful interceptor support; cleaner than `fetch` for complex auth scenarios |
| **react-markdown** | Markdown rendering | The AI responds with markdown (bold text, bullet lists, code blocks) — this library renders it properly in the browser |

**How to set up the project:**
```bash
npm create vite@latest client -- --template react
cd client
npm install react-router-dom axios react-markdown
npm run dev
```
> After running `npm run dev`, Vite starts a local development server at `http://localhost:3000`. Every file change you save auto-refreshes in the browser — this is HMR in action.

---

## Slide 4 — Project Folder Structure (Client)

Understanding the folder structure before writing code saves a lot of confusion. Here is the complete structure of our frontend project and what each folder/file is responsible for:

```
client/src/
├── main.jsx          ← Entry point — mounts the React app into the HTML page
├── App.jsx           ← Defines all the URL routes (which page to show for which URL)
├── index.css         ← Global CSS — base font, body reset, layout defaults
│
├── context/          ← Global state that any component anywhere can access
│   ├── AuthContext.jsx   ← Knows who is logged in; provides login/logout/register functions
│   └── ThemeContext.jsx  ← Knows if dark/light mode is active; provides toggle function
│
├── pages/            ← Full-screen views — one file = one page
│   ├── Login.jsx         ← The sign-in screen with email + password form
│   ├── Register.jsx      ← The sign-up screen with name + email + password form
│   ├── ForgotPassword.jsx ← User enters email to request a password reset link
│   ├── ResetPassword.jsx  ← User enters new password (token comes from the email link)
│   ├── Home.jsx          ← Landing page after login: welcome message + suggestion cards
│   ├── Chat.jsx          ← The main chat interface (sidebar + message feed + input)
│   └── Profile.jsx       ← Edit display name, change settings, delete account
│
├── components/       ← Reusable building blocks used across multiple pages
│   ├── ProtectedRoute.jsx  ← Wrapper that blocks unauthenticated users from accessing a page
│   ├── PasswordInput.jsx   ← A text input with a show/hide password toggle button
│   ├── LogoutButton.jsx    ← A button that shows a "Are you sure?" modal before logging out
│   ├── UserAvatar.jsx      ← A circle with the user's initials (e.g. "AK" for Ankush Kumar)
│   ├── ThemeToggle.jsx     ← Sun/Moon icon button that switches between light and dark mode
│   ├── ChatSidebar.jsx     ← Left panel showing all conversations with search and delete
│   ├── MessageList.jsx     ← The scrollable area showing all messages in a conversation
│   ├── MessageInput.jsx    ← Bottom input area with text field, mic button, and send button
│   └── ChatBubble.jsx      ← A single message — green bubble on right (user), grey on left (AI)
│
└── services/
    └── api.js            ← Central place for all API calls; has Axios interceptors for auth
```

---

## Slide 5 — Entry Point: main.jsx

`main.jsx` is the **very first JavaScript file** that runs when someone opens the app. Think of it as the root of everything. Its job is to:

1. Find the `<div id="root">` element inside `index.html`
2. Inject the entire React application into that div
3. Set up the global providers that all child components will depend on

```jsx
// client/src/main.jsx
import React from 'react';
import ReactDOM from 'react-dom/client';
import { BrowserRouter } from 'react-router-dom';
import { AuthProvider } from './context/AuthContext';
import { ThemeProvider } from './context/ThemeContext';
import App from './App';
import './index.css';

ReactDOM.createRoot(document.getElementById('root')).render(
  <BrowserRouter>
    <AuthProvider>
      <ThemeProvider>
        <App />
      ThemeProvider>
    </AuthProvider>
  </BrowserRouter>
);
```

**Breaking down the nesting (from outside to inside):**
- `<BrowserRouter>` — This enables React Router, so the app can react to URL changes (e.g. going from `/login` to `/chat`) without a full page reload
- `<AuthProvider>` — This wraps the entire app with the authentication context. Any component anywhere in the tree can now call `useAuth()` to know if someone is logged in
- `<ThemeProvider>` — This wraps the app with the theme context. Any component can call `useTheme()` to get the current colour palette
- `<App />` — This is where the routes are defined. It sits inside all providers so it can access both auth and theme

> 💡 **Key concept — Provider pattern:** Providers are like "signal towers" — they broadcast data to all components below them in the tree. Components "tune in" using a matching custom hook. This avoids passing props through many levels of components.

---

## Slide 6 — Routing: App.jsx

`App.jsx` is the **traffic director** of the application. Its only job is to say: "When the URL is X, show component Y." It uses React Router v6's `<Routes>` and `<Route>` system.

```jsx
// client/src/App.jsx
import { Routes, Route, Navigate } from 'react-router-dom';
import ProtectedRoute from './components/ProtectedRoute';
import Login from './pages/Login';
import Register from './pages/Register';
import ForgotPassword from './pages/ForgotPassword';
import ResetPassword from './pages/ResetPassword';
import Home from './pages/Home';
import Chat from './pages/Chat';
import Profile from './pages/Profile';

function App() {
  return (
    <Routes>
      {/* Public routes — no login needed */}
      <Route path="/login"           element={<Login />} />
      <Route path="/register"        element={<Register />} />
      <Route path="/forgot-password" element={<ForgotPassword />} />
      <Route path="/reset-password"  element={<ResetPassword />} />

      {/* Protected routes — must be logged in */}
      <Route path="/"        element={<ProtectedRoute><Home /></ProtectedRoute>} />
      <Route path="/chat"    element={<ProtectedRoute><Chat /></ProtectedRoute>} />
      <Route path="/profile" element={<ProtectedRoute><Profile /></ProtectedRoute>} />

      {/* Catch-all: any unknown URL → redirect to home */}
      <Route path="*" element={<Navigate to="/" replace />} />
    </Routes>
  );
}
```

**Understanding each part:**
- **Public routes** (login, register, forgot/reset password) — anyone can visit these, even without an account
- **Protected routes** — wrapped with `<ProtectedRoute>`. If the user is not logged in, they get redirected to `/login` before the page even loads
- **`<Navigate to="/" replace />`** — if someone types a made-up URL like `/banana`, they get sent back to the home page instead of seeing a broken page

> 💡 **Concept — Declarative routing:** In React Router, you don't manually switch pages with JavaScript logic. You just declare "at path X, render component Y" and React Router handles everything automatically.

---

## Slide 7 — Context API: AuthContext

`AuthContext` is the **global authentication state manager**. Without it, every component would need to receive user data through props — which would mean passing it through dozens of layers. The Context API eliminates that problem.

**What it keeps track of:**
- `user` — the currently logged-in user object (`{ _id, name, email }`) or `null` if not logged in
- `loading` — `true` while checking if a saved token is still valid on app startup

**Functions it provides to the rest of the app:**
- `login(email, password)` — calls the server, gets a JWT token, stores it, updates `user`
- `register(name, email, password)` — creates account, auto-logs in, stores token
- `logout()` — clears the token and user data from localStorage and memory
- `updateProfile(name)` — updates the user's display name on the server and in state
- `deleteAccount()` — permanently deletes the account and all chats

```jsx
// AuthContext.jsx (simplified to focus on key concepts)
const AuthContext = createContext(null);

export const AuthProvider = ({ children }) => {
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(true);

  // On app startup: check if we have a saved token and if it's still valid
  useEffect(() => {
    const token = localStorage.getItem('token');
    const savedUser = localStorage.getItem('user');

    if (token && savedUser) {
      // First: immediately show the saved user so the UI doesn't flash
      setUser(JSON.parse(savedUser));

      // Then: verify the token is still valid with the server
      api.get('/auth/me')
        .then((res) => {
          setUser(res.data);                             // Update with fresh data
          localStorage.setItem('user', JSON.stringify(res.data));
        })
        .catch(() => {
          // Token is expired or invalid — force logout
          localStorage.removeItem('token');
          localStorage.removeItem('user');
          setUser(null);
        })
        .finally(() => setLoading(false)); // Done checking; allow app to render
    } else {
      setLoading(false); // No token at all — not logged in, render immediately
    }
  }, []);

  const login = async (email, password) => {
    const res = await api.post('/auth/login', { email, password });
    const { token, ...userData } = res.data; // Separate token from user info
    localStorage.setItem('token', token);
    localStorage.setItem('user', JSON.stringify(userData));
    setUser(userData);
    return res.data;
  };

  const logout = () => {
    localStorage.removeItem('token');
    localStorage.removeItem('user');
    setUser(null); // React re-renders automatically, ProtectedRoutes kick in
  };

  return (
    <AuthContext.Provider value={{ user, loading, login, register, logout, updateProfile, deleteAccount }}>
      {children}
    </AuthContext.Provider>
  );
};

// Custom hook for easy access — instead of useContext(AuthContext) everywhere
export const useAuth = () => useContext(AuthContext);
```

> 💡 **Why `loading` state matters:** If the app renders before it has finished checking localStorage + server, protected routes might briefly redirect to `/login` even for valid sessions. The `loading` flag prevents rendering until we know for certain.

---

## Slide 8 — Context API: ThemeContext

`ThemeContext` manages the **visual theme** of the entire app. Instead of using CSS variables, we store all colour values as a JavaScript object — this makes them available directly in component styles without any string lookups.

**The theme object contains all colour tokens:**
```jsx
const themes = {
  dark: {
    bg: '#0f0f0f',          // Main background — very dark grey (not pure black)
    surface: '#1a1a1a',     // Cards, panels, sidebar background
    surfaceHover: '#1f1f1f',// Hover state for surface elements
    text: '#e4e4e4',        // Primary text — off-white (easier on the eyes than pure white)
    textMuted: '#888888',   // Secondary text — timestamps, labels, hints
    border: '#333333',      // Borders and dividers
    inputBg: '#0f0f0f',     // Background inside text inputs and textareas
    accent: '#238636',      // Primary action colour — GitHub's green (buttons, active states)
    accentHover: '#2ea043', // Accent colour on hover — slightly brighter
    error: '#f85149',       // Error messages and destructive actions
    link: '#58a6ff',        // Hyperlinks and clickable text
    userBubble: '#238636',  // User's chat bubble background — green
    assistantBubble: '#1a1a1a', // AI's chat bubble background — dark surface
    overlay: 'rgba(0,0,0,0.6)', // Semi-transparent background behind modals
    modalBg: '#1a1a1a',    // Modal dialog background
  },
  light: {
    bg: '#f6f8fa',          // GitHub's light grey — soft, not glaring white
    surface: '#ffffff',
    text: '#1f2328',
    accent: '#238636',      // Same green — consistent brand colour across themes
    // ... etc
  }
};

// Using useLayoutEffect instead of useEffect prevents a visible flash of wrong theme
useLayoutEffect(() => {
  localStorage.setItem(STORAGE_KEY, themeMode); // Persist the choice
  document.documentElement.setAttribute('data-theme', themeMode); // For CSS if needed
}, [themeMode]);

const toggleTheme = () =>
  setThemeMode(prev => prev === 'dark' ? 'light' : 'dark');
```

**How components use the theme:**
```jsx
const { theme } = useTheme();
// Then use in inline styles:
<div style={{ background: theme.surface, color: theme.text }}>...</div>
```

> 💡 **Why `useLayoutEffect` instead of `useEffect`?** `useLayoutEffect` runs synchronously before the browser paints. This means the theme is applied before the user sees anything — preventing the annoying flash of wrong colours on page load.

---

## Slide 9 — ProtectedRoute Component

`ProtectedRoute` is a **security wrapper**. Any page that requires a logged-in user is wrapped with it. If someone navigates directly to `/chat` or `/profile` without being logged in, they get redirected to `/login` automatically — no matter how they try to access it.

```jsx
// components/ProtectedRoute.jsx
import { Navigate } from 'react-router-dom';
import { useAuth } from '../context/AuthContext';

const ProtectedRoute = ({ children }) => {
  const { user, loading } = useAuth();

  // Still checking if the user is logged in — don't decide yet
  if (loading) return <div>Loading...</div>;

  // No user found after checking — redirect to login
  if (!user) return <Navigate to="/login" replace />;

  // User is authenticated — render the actual page
  return children;
};
```

**The three states it handles:**
1. **Loading** — The app is still checking localStorage + server. We show a loading indicator rather than incorrectly redirecting
2. **Not logged in** — Redirect to `/login`. The `replace` prop means the user can't press "Back" to get to the protected page
3. **Logged in** — Render whatever component was passed as `children`

> 💡 **The `replace` prop on `<Navigate>`:** Without `replace`, the browser would add `/login` to the history stack. With `replace`, it replaces the current entry, so pressing "Back" after logging in goes to the page before the protected attempt — much better UX.

---

## Slide 10 — Pages Overview

All pages live in `client/src/pages/`. Each one is a complete screen that the user can navigate to. Here's what each one does and what makes it interesting:

| Page | Route | Key Features & Behaviour |
|------|-------|--------------------------|
| `Login.jsx` | `/login` | Email + password form with validation; shows error messages inline; has "Forgot password?" link; on success stores JWT and navigates to `/` |
| `Register.jsx` | `/register` | Name + email + password; calls register API; auto-logs the user in on success; navigates to home |
| `ForgotPassword.jsx` | `/forgot-password` | Single email field; on submit calls forgot-password API; shows a success message regardless (to avoid revealing if an email exists) |
| `ResetPassword.jsx` | `/reset-password?token=xxx` | Reads the JWT reset token from the URL query string; shows new password form; calls reset-password API; redirects to login on success |
| `Home.jsx` | `/` | Landing page after login; shows a greeting with the user's name; has a "+ New Chat" button; shows suggestion cards like "Explain React hooks" |
| `Chat.jsx` | `/chat` | The core of the app — three-column layout with sidebar, message feed, and input area; handles creating/selecting/deleting chats and sending messages |
| `Profile.jsx` | `/profile` | Edit display name; "Delete Account" button with a confirmation modal that asks the user to type "DELETE" to confirm |

---

## Slide 11 — Chat Page Architecture

`Chat.jsx` is the most complex page. It acts as an **orchestrator** — it manages all the state and passes data + functions down to three child components.

```
Chat.jsx — the parent, manages all state
│
│  State it manages:
│  - chats[] — list of all user's conversations (shown in sidebar)
│  - activeChat — the currently selected chat object
│  - loading — true while the AI is generating a response
│  - sidebarOpen — whether the sidebar is visible on mobile
│
├── functions it handles:
│   ├── fetchChats() — loads all chats from the server on mount
│   ├── fetchChat(id) — loads a specific chat's messages when selected
│   ├── createChat() — creates a new empty chat and selects it
│   ├── sendMessage(content) — sends a user message and appends the AI reply
│   └── deleteChat(id) — removes a chat from the list
│
├── ChatSidebar (receives: chats, activeChat, onSelect, onCreate, onDelete)
│   ├── Displays the list of all conversations
│   ├── Has a search input that filters chats client-side
│   ├── Each chat item has an × button with a "Are you sure?" confirmation modal
│   └── "+ New Chat" button at the top creates a fresh conversation
│
├── MessageList (receives: messages, loading)
│   ├── Maps over the messages array and renders one ChatBubble per message
│   └── Shows an animated "AI is thinking..." indicator when loading is true
│
└── MessageInput (receives: onSend, loading)
    ├── A textarea that grows with content (not a fixed-height input)
    ├── Enter key sends the message; Shift+Enter adds a new line
    ├── 🎙️ Mic button toggles voice recording
    └── Send button triggers the onSend callback
```

---

## Slide 12 — MessageInput + Voice Recording

The **mic button** is one of the most impressive features of the app. Here is the step-by-step flow of how voice input works, using the browser's built-in `MediaRecorder` API — no external library needed:

```jsx
// Step 1: Request microphone access from the browser
// The browser shows a permission popup the first time
const stream = await navigator.mediaDevices.getUserMedia({ audio: true });

// Step 2: Create a MediaRecorder that will capture the audio stream
const recorder = new MediaRecorder(stream);
const audioChunks = []; // We'll collect pieces of audio data here

// Step 3: When audio data is available, store it
recorder.ondataavailable = (event) => {
  audioChunks.push(event.data); // Each chunk is a small piece of audio
};

// Step 4: When recording stops, assemble the chunks into one file
recorder.onstop = async () => {
  // Combine all chunks into a single binary Blob (a file in memory)
  const audioBlob = new Blob(audioChunks, { type: 'audio/webm' });

  // Wrap it in FormData so we can send it as a file upload
  const formData = new FormData();
  formData.append('audio', audioBlob, 'recording.webm');

  // Send to our Express server, which forwards it to Groq Whisper
  const response = await voiceAPI.transcribe(formData);

  // Insert the returned text into the input field
  setInputText(response.data.text);
};

// Step 5: Start recording when mic button is pressed
recorder.start();

// Step 6: Stop after user presses mic button again
recorder.stop();
stream.getTracks().forEach(track => track.stop()); // Release mic access
```

**Why does the Axios interceptor delete `Content-Type` for FormData?**

Normally, Axios sets `Content-Type: application/json`. But when uploading a file, the browser needs to set `Content-Type: multipart/form-data; boundary=abc123...` — the `boundary` value is auto-generated and critical for the server to parse the file correctly. If we manually set any Content-Type, the boundary gets lost and the upload fails.

> 💡 **User experience tip:** Show a visual indicator (e.g., pulsing red dot) while the mic is active — users need clear feedback that the app is recording.

---

## Slide 13 — ChatBubble Component

`ChatBubble` renders **one single message**. Every message in a conversation is either from the `user` (the human) or the `assistant` (the AI). The rendering is different depending on who sent it:

**User messages** (role = `"user"`):
- Aligned to the **right** side of the screen
- Has a **green background** (`theme.userBubble` = `#238636`)
- Text is white — high contrast on green
- Plain text, no markdown rendering (users type plain text)

**AI messages** (role = `"assistant"`):
- Aligned to the **left** side of the screen  
- Has a **grey background** (`theme.assistantBubble`)
- Rendered with `<ReactMarkdown>` — the AI often responds with:
  - `**bold text**` for emphasis
  - `` `code snippets` `` for examples
  - ` ```code blocks``` ` for multi-line code
  - `- bullet lists` for structured explanations
  - `## Headers` for long responses

```jsx
const ChatBubble = ({ message }) => {
  const { theme } = useTheme();
  const isUser = message.role === 'user';

  return (
    <div style={{
      display: 'flex',
      justifyContent: isUser ? 'flex-end' : 'flex-start', // Align direction
      marginBottom: '12px',
    }}>
      <div style={{
        maxWidth: '70%',                  // Bubbles don't stretch full width
        padding: '10px 14px',
        borderRadius: isUser ? '18px 18px 4px 18px' : '18px 18px 18px 4px',
        background: isUser ? theme.userBubble : theme.assistantBubble,
        color: isUser ? '#ffffff' : theme.text,
      }}>
        {isUser
          ? message.content                       // Plain text for user messages
          : <ReactMarkdown>{message.content}</ReactMarkdown> // Markdown for AI
        }
      </div>
    </div>
  );
};
```

> 💡 **Why different border-radius values?** The "tail" of a speech bubble points toward the speaker. For user messages (right), the bottom-right corner is flat. For AI messages (left), the bottom-left corner is flat. This is a classic chat UI convention.

---

## Slide 14 — API Service Layer (api.js)

All communication with the backend goes through a **single Axios instance** defined in `services/api.js`. This gives us one central place to handle repeated concerns like authentication headers and error handling — instead of duplicating them in every component.

```js
// client/src/services/api.js
import axios from 'axios';

// Create an Axios instance with default settings
const api = axios.create({
  baseURL: '/api',     // All requests are prefixed with /api (e.g., /api/auth/login)
  timeout: 60000,      // If no response in 60 seconds, treat as an error
  headers: {
    'Content-Type': 'application/json', // Default: tell the server we're sending JSON
  },
});

// ─── REQUEST INTERCEPTOR ─────────────────────────────────────
// This runs BEFORE every request is sent
api.interceptors.request.use((config) => {
  const token = localStorage.getItem('token');

  if (token) {
    // Attach the JWT so the server knows who is making the request
    config.headers.Authorization = `Bearer ${token}`;
  }

  // Special case: if we're sending a file (FormData), don't set Content-Type
  // The browser will set it automatically with the correct multipart boundary
  if (config.data instanceof FormData) {
    delete config.headers['Content-Type'];
  }

  return config; // Must return config otherwise the request is blocked
});

// ─── RESPONSE INTERCEPTOR ─────────────────────────────────────
// This runs AFTER every response is received
api.interceptors.response.use(
  (response) => response, // Success — just pass through unchanged

  (error) => {
    if (error.response?.status === 401) {
      // 401 = Unauthorized — our token is expired or invalid
      // Force a full logout: clear stored data and redirect to login
      localStorage.removeItem('token');
      localStorage.removeItem('user');
      window.location.href = '/login'; // Hard redirect forces a full re-render
    }
    return Promise.reject(error); // Re-throw so components can catch specific errors
  }
);

// ─── API NAMESPACES ─────────────────────────────────────────
// These are the actual functions components call to make API requests

export const authAPI = {
  login: (email, password) => api.post('/auth/login', { email, password }),
  register: (name, email, password) => api.post('/auth/register', { name, email, password }),
  getMe: () => api.get('/auth/me'),                        // get current user profile
  updateProfile: (name) => api.put('/auth/profile', { name }),
  deleteAccount: () => api.delete('/auth/me'),
  forgotPassword: (email) => api.post('/auth/forgot-password', { email }),
  resetPassword: (token, password) => api.post('/auth/reset-password', { token, password }),
};

export const chatAPI = {
  createChat: () => api.post('/chats'),                    // create empty conversation
  getChats: (search) => api.get('/chats', { params: search ? { search } : {} }), // list + search
  getChat: (id) => api.get(`/chats/${id}`),                // get single chat with messages
  sendMessage: (chatId, content) => api.post(`/chats/${chatId}/messages`, { content }),
  deleteChat: (id) => api.delete(`/chats/${id}`),
};

export const voiceAPI = {
  transcribe: (formData) => api.post('/voice/transcribe', formData, {
    timeout: 30000, // Voice uploads need a shorter timeout than the default 60s
  }),
};
```

---

## Slide 15 — Reusable Components Deep Dive

Each component has a single, well-defined responsibility. Here is a more detailed look at what each one does internally:

| Component | What it renders | Key internal behaviour |
|-----------|-----------------|------------------------|
| `PasswordInput` | A text input with a 👁️ toggle button | Manages `showPassword` state (boolean); switches `type` between `"password"` and `"text"` |
| `LogoutButton` | A button that opens a confirmation modal | Manages `modalOpen` state; only calls `logout()` from AuthContext after user confirms |
| `UserAvatar` | A coloured circle with initials | Extracts first letter of first and last name from `user.name`; applies accent colour background |
| `ThemeToggle` | A sun ☀️ or moon 🌙 icon button | Calls `toggleTheme()` from ThemeContext; displays the icon for the current mode |
| `ChatSidebar` | The left navigation panel | Manages its own `searchQuery` state; filters chat list locally without any extra API calls |
| `MessageList` | The scrollable message feed | Uses `useEffect` + `useRef` to auto-scroll to the bottom whenever a new message is added |
| `MessageInput` | The bottom input bar | Manages `inputText`, `isRecording`, and `mediaRecorder` state; handles both Enter key and button click |
| `ChatBubble` | One message bubble | Renders markdown for AI messages using `<ReactMarkdown>`; applies theme-aware colours |

> 💡 **Principle — Single Responsibility:** Each component should do exactly one thing well. When debugging, you know exactly which file to look in. When reusing, you just import and drop it in.

---

## Slide 16 — Key Concepts Covered Today

Let's recap every major concept we applied in this masterclass:

**✅ React Context API**
Global state without prop-drilling. We created two contexts: Auth (who is logged in) and Theme (what colour scheme is active). Any component can access these with a single hook call.

**✅ React Router v6**
Declarative client-side routing using `<Routes>`, `<Route>`, `<Navigate>`, and `useNavigate`. URLs change without page reloads.

**✅ Protected Routes**
A wrapper component that checks authentication before rendering a page. Unauthenticated users are redirected to `/login` before they see anything.

**✅ Controlled Inputs**
Form inputs where React controls the value — `value={state}` + `onChange={setState}`. This gives us full control over user input for validation.

**✅ Axios Interceptors**
Middleware functions that run on every HTTP request or response. We use them to attach the auth token automatically and handle 401 errors globally.

**✅ MediaRecorder API**
The browser's built-in API for recording microphone audio. We collect chunks, assemble them into a Blob, and upload to our server for transcription.

**✅ Component Composition**
Building pages by assembling small, focused components. Chat.jsx is composed of ChatSidebar + MessageList + MessageInput — each independent and testable.

**✅ localStorage Persistence**
Storing the JWT token, user object, and theme preference in the browser so they survive page refreshes and browser restarts.

---

## Slide 17 — What's Next?

Today you built **The Face** — a complete, beautiful React frontend. But right now, all the API calls fail because there is no backend to receive them. That changes in the next masterclass.

**Next → Masterclass 2: The Brain**

We will build the Express server that powers everything:
- Registration and login endpoints that issue real JWTs
- All chat endpoints that store and retrieve conversations
- Groq AI integration that generates intelligent responses
- Voice transcription using Whisper
- Request validation so bad inputs are rejected cleanly

```bash
# Tag and push your Masterclass 1 work before the next session
git add .
git commit -m "Masterclass 1 complete"
git tag masterclass-1
git push origin main
git push origin masterclass-1  # push the tag as well
```

---

## Slide 18 — Q&A / Recap

> What exactly did we build in this masterclass?

1. **A full Vite + React app** with 7 pages and client-side routing using React Router v6
2. **AuthContext** — manages who is logged in, persists session across refreshes
3. **ThemeContext** — manages light/dark theme with a full colour token system
4. **9 reusable components** — including a working voice input with MediaRecorder
5. **A centralised API layer** — Axios instance with request and response interceptors
6. **Protected routes** — automatic redirect to login for unauthenticated access
7. **react-markdown** rendering — AI responses are rendered with proper formatting

**Questions?** 🚀
