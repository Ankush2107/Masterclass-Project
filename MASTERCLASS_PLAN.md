# classMentor AI - Masterclass Plan

A ChatGPT-like full-stack application built with **MERN Stack** (MongoDB, Express, React, Node.js) using **JavaScript only**.

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

---

## Masterclass 1: Project Setup & Backend Foundation (~2 hours)

**What you'll build:**
- Project folder structure (monorepo: `client/` + `server/`)
- Node.js + Express server setup
- MongoDB connection with Mongoose
- User model & schema
- Basic health check API
- Environment variables (.env)

**Files to create:**
- `server/package.json`
- `server/server.js`
- `server/config/db.js`
- `server/models/User.js`
- `server/.env.example`
- `server/.gitignore`

**Key concepts taught:**
- REST API basics
- Mongoose schemas
- Environment configuration
- Project architecture

**Push as:** `masterclass-1`

---

## Masterclass 2: Authentication & User Management (~2 hours)

**What you'll build:**
- JWT (JSON Web Token) authentication
- User registration API (`POST /api/auth/register`)
- User login API (`POST /api/auth/login`)
- Password hashing with bcrypt
- Auth middleware for protected routes
- Get current user endpoint (`GET /api/auth/me`)

**Files to create:**
- `server/routes/auth.js`
- `server/controllers/authController.js`
- `server/middleware/auth.js`
- Update `server/server.js` with auth routes

**Key concepts taught:**
- JWT flow
- bcrypt password hashing
- Middleware pattern
- Protected routes

**Push as:** `masterclass-2`

---

## Masterclass 3: Frontend Foundation & Auth UI (~2 hours)

**What you'll build:**
- React app with Vite
- React Router setup
- Auth context (global auth state)
- Login page
- Register page
- Protected route wrapper
- API service layer (axios)
- Basic layout/navigation

**Files to create:**
- `client/` - Full React app
- `client/src/context/AuthContext.jsx`
- `client/src/pages/Login.jsx`
- `client/src/pages/Register.jsx`
- `client/src/pages/Home.jsx`
- `client/src/components/ProtectedRoute.jsx`
- `client/src/services/api.js`

**Key concepts taught:**
- React Context API
- React Router
- Axios interceptors
- Form handling

**Push as:** `masterclass-3`

---

## Masterclass 4: AI Chat Backend (~2 hours)

**What you'll build:**
- Chat model (conversations with messages)
- Create new chat API (`POST /api/chats`)
- Send message & get AI response (`POST /api/chats/:id/messages`)
- Get user's chat history (`GET /api/chats`)
- Google AI Studio (Gemini) integration (or mock for teaching)
- Message streaming (optional)

**Files to create:**
- `server/models/Chat.js`
- `server/models/Message.js`
- `server/routes/chat.js`
- `server/controllers/chatController.js`
- `server/services/googleAiService.js` (or mock)

**Key concepts taught:**
- Nested Mongoose schemas
- External API integration
- Async/await patterns
- RESTful resource design

**Push as:** `masterclass-4`

---

## Masterclass 5: Chat UI & Polish (~2 hours)

**What you'll build:**
- ChatGPT-like chat interface
- Sidebar with chat history
- Message bubbles (user vs AI)
- New chat button
- Loading states & typing indicator
- Error handling & toast notifications
- Responsive design
- Final polish & deployment notes

**Files to create:**
- `client/src/pages/Chat.jsx`
- `client/src/components/ChatSidebar.jsx`
- `client/src/components/MessageList.jsx`
- `client/src/components/MessageInput.jsx`
- `client/src/components/ChatBubble.jsx`
- Update routing and navigation

**Key concepts taught:**
- Component composition
- State management for chat
- UX best practices
- Real-time feel (loading states)

**Push as:** `masterclass-5`

---

## Tech Stack Summary

| Layer | Technology |
|-------|------------|
| Frontend | React 18, Vite, React Router, Axios |
| Backend | Node.js, Express |
| Database | MongoDB, Mongoose |
| Auth | JWT, bcrypt |
| AI | Google AI Studio (Gemini) - free tier |

---

## Prerequisites for Students

- Node.js 18+
- MongoDB (local or Atlas)
- Google AI Studio API key (free at aistudio.google.com) - or use `googleAiService.mock.js` for demo
- Git

### Without OpenAI API Key

Use the mock service for demos: in `chatController.js`, import from `openaiService.mock.js` instead of `openaiService.js`.

---

## Running the Project

**Backend:**
```bash
cd server
npm install
cp .env.example .env
# Edit .env with your MongoDB URI and OpenAI key
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
