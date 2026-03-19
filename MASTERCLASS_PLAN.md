# classMentor AI - Masterclass Plan

A ChatGPT-like full-stack application built with **MERN Stack** (MongoDB, Express, React, Node.js) using **JavaScript only**.

**4 Phases:** Component Design & UI → RESTful APIs & Groq → NoSQL & MongoDB → Deployment (Vercel & Render).

---

## Git Workflow for Each Masterclass

After completing each masterclass, push your code with the masterclass tag:

```bash
git add .
git commit -m "Masterclass X complete"
git tag masterclass-X
git push origin main
git push origin masterclass-X
```

Or if using branches:
```bash
git checkout -b masterclass-X
git add .
git commit -m "Masterclass X complete"
git push origin masterclass-X
```

Use **masterclass-1** through **masterclass-4** for the four phases below.

---

## Masterclass 1: Component Design & UI Logic with React (~2 hours)

**Focus:** The Face — building the React UI, reusable components, and client-side logic.

**What you'll build:**
- React app with Vite
- React Router (routes for login, register, forgot/reset password, home, chat, profile)
- Auth context (global auth state, login, register, logout)
- Theme context (light/dark, persisted in localStorage)
- Login, Register, Forgot Password, Reset Password pages
- Home page (welcome, New Chat, suggestion cards)
- Chat page (sidebar + message list + input with voice)
- Profile page (edit name, delete account with confirmation)
- Protected route wrapper (redirect to login if not authenticated)
- Reusable components: PasswordInput, LogoutButton (with confirmation modal), UserAvatar, ThemeToggle, ChatSidebar, MessageList, MessageInput (with mic), ChatBubble
- API service layer (axios with interceptors: token, 401 handling, FormData for voice)

**Files to create / cover:**
- `client/` — Vite + React app
- `client/src/main.jsx`, `client/src/App.jsx`, `client/src/index.css`
- `client/src/context/AuthContext.jsx`, `client/src/context/ThemeContext.jsx`
- `client/src/pages/Login.jsx`, `Register.jsx`, `ForgotPassword.jsx`, `ResetPassword.jsx`, `Home.jsx`, `Chat.jsx`, `Profile.jsx`
- `client/src/components/ProtectedRoute.jsx`, `PasswordInput.jsx`, `LogoutButton.jsx`, `UserAvatar.jsx`, `ThemeToggle.jsx`, `ChatSidebar.jsx`, `MessageList.jsx`, `MessageInput.jsx`, `ChatBubble.jsx`
- `client/src/services/api.js`

**Key concepts taught:**
- React Context API
- React Router (Routes, Route, Navigate, useNavigate)
- Component composition and props
- Form handling and controlled inputs
- Axios interceptors (attach token, handle 401)
- MediaRecorder for voice input (optional)

**Push as:** `masterclass-1`

---

## Masterclass 2: RESTful APIs & LLM Integration (Groq) (~2 hours)

**Focus:** The Brain — Express server, REST endpoints, auth, and Groq (chat + speech-to-text).

**What you'll build:**
- Express server setup (CORS, JSON body, health check)
- Auth routes: register, login, getMe, updateProfile, deleteAccount, forgot-password, reset-password
- JWT authentication and protect middleware
- Password hashing with bcrypt
- Chat routes: create chat, list chats, get chat, send message, delete chat
- Voice route: POST /transcribe (multer + Groq Whisper)
- Groq integration: chat completions (e.g. llama-3.1-8b-instant) and audio transcriptions (Whisper)
- Optional: email service (Nodemailer) for welcome email and password reset
- Request validation (express-validator) and error handling

**Files to create / cover:**
- `server/server.js`
- `server/routes/auth.js`, `server/routes/chat.js`, `server/routes/voice.js`
- `server/controllers/authController.js`, `server/controllers/chatController.js`, `server/controllers/voiceController.js`
- `server/middleware/auth.js`, `server/middleware/validate.js`
- `server/services/aiService.js` (Groq chat + transcribe)
- `server/services/emailService.js`, `server/config/email.js` (optional)
- `server/validators/authValidators.js`, `server/validators/chatValidators.js`

**Key concepts taught:**
- RESTful API design (HTTP methods, status codes)
- JWT flow and middleware pattern
- External API integration (Groq OpenAI-compatible API)
- Multer for multipart uploads (audio)
- Async/await and error handling

**Push as:** `masterclass-2`

---

## Masterclass 3: NoSQL Modeling & Schema Design with MongoDB (~2 hours)

**Focus:** The Memory — data models, Mongoose schemas, and persistence.

**What you'll build:**
- MongoDB connection (Mongoose, connection string in .env)
- User model: name, email (unique), password (hashed), timestamps
- Chat model: reference to user, title, embedded messages (role, content, timestamps)
- How auth uses User (register, login, getMe, updateProfile, deleteAccount)
- How chat uses Chat (create, list by user, get one, append messages, delete)
- Indexes and constraints (e.g. unique email)
- Environment variables for database (MONGODB_URI)

**Files to create / cover:**
- `server/config/db.js`
- `server/models/User.js`
- `server/models/Chat.js` (with embedded message subdocuments)
- `server/.env.example` (MONGODB_URI, JWT_SECRET, etc.)
- Integration with controllers (authController, chatController)

**Key concepts taught:**
- NoSQL document model vs relational
- Mongoose schemas, types, and options
- References (ref to User) vs embedded documents (messages in Chat)
- Connection and error handling
- When to embed vs reference

**Push as:** `masterclass-3`

---

## Masterclass 4: Deployment — Vercel & Render Cloud Configuration (~2 hours)

**Focus:** The Launch — putting the app on the cloud.

**What you'll build:**
- Frontend deployment on **Vercel** (Vite build, env variables, preview/production)
- Backend deployment on **Render** (Node/Express, start command, env variables)
- Environment configuration for production (MONGODB_URI, JWT_SECRET, GROQ_API_KEY, CORS origin, APP_URL for emails)
- Optional: SMTP and APP_URL for password reset in production
- Health check and debugging in production
- Notes on free tiers, cold starts, and scaling

**Steps to cover:**
- Build client: `npm run build` in `client/`
- Deploy client to Vercel (connect repo, set root to `client`, build command, output dir `dist`)
- Deploy server to Render (Web Service, connect repo, root `server`, build `npm install`, start `npm start` or `node server.js`)
- Set production env vars on both platforms
- Configure CORS on server to allow Vercel frontend URL
- Update frontend API base URL if needed (e.g. proxy or full backend URL)

**Key concepts taught:**
- Separation of frontend and backend in deployment
- Environment variables in the cloud
- CORS and security in production
- Using Vercel and Render free tiers

**Push as:** `masterclass-4`

---

## Tech Stack Summary

| Layer        | Technology                                  |
|-------------|---------------------------------------------|
| Frontend    | React 18, Vite, React Router, Axios         |
| Backend     | Node.js, Express                           |
| Database    | MongoDB, Mongoose                          |
| Auth        | JWT, bcrypt                                |
| AI / Voice  | Groq (chat: Llama, speech-to-text: Whisper) |
| Deployment  | Vercel (client), Render (server)           |

---

## Prerequisites for Students

- Node.js 18+
- MongoDB (local or [Atlas](https://www.mongodb.com/cloud/atlas) free tier)
- [Groq](https://console.groq.com) API key (free tier for chat and Whisper)
- Git
- Accounts: [Vercel](https://vercel.com), [Render](https://render.com)

**Optional:** Gmail App Password for forgot-password emails (see `server/.env.example`).

---

## Running the Project Locally

**Backend:**
```bash
cd server
npm install
cp .env.example .env
# Edit .env: MONGODB_URI, JWT_SECRET, GROQ_API_KEY, etc.
npm run dev
```

**Frontend:**
```bash
cd client
npm install
npm run dev
```

---

*Happy Teaching! 🎓*
