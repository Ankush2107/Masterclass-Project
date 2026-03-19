# Masterclass 4 — The Launch
## Deployment: Vercel & Render Cloud Configuration

---

## Slide 1 — Welcome Back

**What we've built across all masterclasses:**
- 🎨 **Masterclass 1 (Face)** — Complete React frontend with routing, context, components, voice input
- 🧠 **Masterclass 2 (Brain)** — Express REST API with JWT auth, Groq AI, Whisper, email service
- 🗄️ **Masterclass 3 (Memory)** — MongoDB Atlas with Mongoose schemas for persistent data

**The final challenge:**
Right now the app only works on your laptop because it runs on `localhost`. If you share your IP address with a friend, it won't work — their request can't reach your machine. We need to put the app on a real server on the internet with a proper domain.

**Today's focus → THE LAUNCH**
We'll deploy the React frontend to **Vercel** and the Express backend to **Render** — two cloud platforms that are completely free to start with. By the end of this masterclass, anyone in the world can use your app through a real URL.

---

## Slide 2 — The 4 Masterclass Journey

| # | Name | What We Build |
|---|------|---------------|
| 1 | The Face | React UI — pages, components, routing |
| 2 | The Brain | Express server, REST APIs, Groq AI |
| 3 | The Memory | MongoDB, Mongoose schemas |
| **4** | **The Launch** ← *You are here* | Vercel + Render deployment, production config |

---

## Slide 3 — Production Architecture: What Goes Where

In development, both frontend and backend run on your machine. In production, they live on separate cloud platforms:

```
┌───────────────────────────────────────────────────────────────┐
│                         INTERNET                               │
└─────────────────────┬────────────────────────┬────────────────┘
                      │                        │
           ┌──────────▼──────────┐  ┌──────────▼──────────┐
           │       VERCEL        │  │       RENDER         │
           │   (Frontend CDN)    │  │   (Backend Server)   │
           │                     │  │                      │
           │  classmentor.vercel │  │  classmentor-api     │
           │  .app               │  │  .onrender.com       │
           │                     │  │                      │
           │  Static files:      │  │  Node.js process:    │
           │  HTML + JS + CSS    │  │  Express API server  │
           │  (built by Vite)    │◄►│  (your server.js)    │
           └─────────────────────┘  └──────────┬───────────┘
                                               │
                                    ┌──────────▼──────────┐
                                    │    MongoDB Atlas     │
                                    │   (Cloud Database)   │
                                    │   cluster.mongodb   │
                                    │   .net              │
                                    └─────────────────────┘
```

| Platform | What it hosts | Cost | Speciality |
|----------|--------------|------|------------|
| **Vercel** | React frontend (static HTML/JS/CSS) | Free | Global CDN, auto-deploy from Git, instant preview URLs |
| **Render** | Express backend (Node.js process) | Free | Auto-deploy from Git, managed HTTPS, environment variables |
| **MongoDB Atlas** | Database | Free (M0) | Managed cloud MongoDB, automatic backups |

---

## Slide 4 — Step 1: Build the Frontend for Production

Before deploying the React app, we need to compile it into plain HTML, CSS, and JavaScript files that any web server can serve. React/JSX is a development format — browsers can't read it directly.

**Run the build command:**
```bash
cd client
npm run build
# What this actually runs: vite build
```

**What Vite does during the build:**
1. Reads your JSX files and converts them to plain JavaScript
2. Bundles all your components, pages, and libraries into a few optimised files
3. Minifies the code (removes whitespace and shortens variable names) to reduce file size
4. Generates unique content hashes in filenames (e.g. `index-a3f2c1.js`) for browser caching

**The output — `client/dist/` folder:**
```
client/dist/
├── index.html              ← The entry HTML page (very short — just loads the JS)
└── assets/
    ├── index-a3f2c1.js     ← All React code, all components, all logic — minified
    └── index-b8d4e2.css    ← All styles — minified
```

The `dist/` folder is what actually gets deployed. Vercel will run `npm run build` automatically, so you never need to build manually before pushing.

> 💡 **Why content-hash filenames?** When you deploy a new version, the filename changes (e.g. `index-a3f2c1.js` → `index-z9p1k7.js`). This forces browsers to download the new version instead of using their cached old version. This is called "cache busting."

---

## Slide 5 — Deploying the Frontend to Vercel

Vercel specialises in hosting JavaScript frontends. It connects to your GitHub repository and automatically re-deploys whenever you push new code.

**Step-by-step deployment:**

1. Go to [vercel.com](https://vercel.com) and click "Sign Up" — use your GitHub account for easiest setup

2. On the dashboard, click **"Add New" → "Project"**

3. Find your GitHub repository in the list and click **"Import"**

4. Configure the project settings:

| Setting | Value | Why |
|---------|-------|-----|
| **Framework Preset** | `Vite` | Tells Vercel how to build and serve the project |
| **Root Directory** | `client` | Our React code is in the `client/` folder, not the repo root |
| **Build Command** | `npm run build` | Vite build command that creates the `dist/` folder |
| **Output Directory** | `dist` | The folder Vercel should serve as the website |

5. Click **"Deploy"** — Vercel runs the build and makes your app live in about 30 seconds

**What Vercel gives you:**
- A live URL like `https://classmentor-ai-ankush.vercel.app`
- An SSL certificate (HTTPS) — automatically managed, always up to date
- A global CDN — your files are copied to servers worldwide; users get the closest one
- A new preview URL for every Git branch — great for testing before merging

---

## Slide 6 — Connecting Frontend to Backend (API URL Problem)

**In development**, the Vite dev server runs at `localhost:3000` and the Express server at `localhost:5000`. Our `api.js` uses `/api` as the base URL — but how does the frontend know where the backend is in production?

There are two approaches:

**Option A — Vite Dev Proxy (development only)**

This is configured in `vite.config.js` and only affects the dev server. When the browser makes a request to `/api`, Vite's proxy forwards it to `localhost:5000`:

```js
// client/vite.config.js
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';

export default defineConfig({
  plugins: [react()],
  server: {
    proxy: {
      '/api': {
        target: 'http://localhost:5000', // Forward /api requests to Express
        changeOrigin: true,              // Change the request origin header to the target
      },
    },
  },
});
```

This works great in development but doesn't exist in production (there's no Vite dev server).

**Option B — Environment Variable (for production)**

Add a `VITE_API_URL` environment variable in Vercel's dashboard:

```js
// client/src/services/api.js — update to use the env var
const API_URL = import.meta.env.VITE_API_URL || '/api';
// In production: VITE_API_URL = 'https://classmentor-api.onrender.com/api'
// In development: falls back to '/api' (proxied by Vite)

const api = axios.create({ baseURL: API_URL });
```

Add this environment variable in Vercel: Project → Settings → Environment Variables:
- Key: `VITE_API_URL`
- Value: `https://classmentor-api.onrender.com/api`

> 💡 **Why must Vercel env vars start with `VITE_`?** Vite only exposes environment variables prefixed with `VITE_` to the browser at build time. All other env vars are available server-side only and would be empty in the browser. This is a security feature — you don't want secret keys exposed in your JavaScript bundle.

---

## Slide 7 — Deploying the Backend to Render

Render is a cloud platform for hosting backend services. It supports Node.js natively, connects to GitHub, and auto-deploys on every push.

**Step-by-step deployment:**

1. Go to [render.com](https://render.com) and sign up with your GitHub account

2. Click **"New +"** in the top right corner

3. Select **"Web Service"** — this is the service type for a constantly-running process like an Express server

4. Connect your GitHub repository — search for it in the list

5. Configure the service:

| Setting | Value | Why |
|---------|-------|-----|
| **Name** | `classmentor-api` | The identifier for this service on Render |
| **Root Directory** | `server` | Our Express code is in the `server/` folder |
| **Runtime** | `Node` | We're running a Node.js application |
| **Build Command** | `npm install` | Install all packages listed in `package.json` |
| **Start Command** | `node server.js` | The command to start the server after building |

6. Click **"Create Web Service"**

Render assigns you a URL like: `https://classmentor-api.onrender.com`

> 💡 **Why `node server.js` instead of `npm run dev`?** The `dev` script uses `nodemon`, which is a development tool that restarts on file changes. In production there are no file changes — you want a stable, efficient process. `node server.js` runs the server directly without any development overhead.

---

## Slide 8 — Setting Production Environment Variables on Render

Environment variables are the mechanism that makes the same code work in different environments (development, staging, production) without any code changes. In production, they're set in the Render dashboard instead of a `.env` file.

Go to your Render Web Service → **Environment** tab → click "Add Environment Variable":

| Variable | Value to set | Why it's needed |
|----------|-------------|-----------------|
| `MONGODB_URI` | Your Atlas connection string | Connects the server to your cloud database |
| `JWT_SECRET` | A long random string (32+ characters) | Signs and verifies JWT tokens; must be secret |
| `PORT` | Render sets this automatically | Render tells your app which port to listen on |
| `GROQ_API_KEY` | From console.groq.com | Enables real AI chat and voice transcription |
| `USE_MOCK_AI` | `false` | Disable mock mode; use real Groq AI in production |
| `SMTP_HOST` | `smtp.gmail.com` | Gmail's SMTP server address |
| `SMTP_PORT` | `465` | SSL port for Gmail (or 587 for TLS) |
| `SMTP_USER` | `your@gmail.com` | Your Gmail address |
| `SMTP_PASS` | Your Gmail App Password | The 16-character token, NOT your Gmail password |
| `EMAIL_FROM` | `classMentor <your@gmail.com>` | The "From" name and address in emails |
| `APP_URL` | `https://classmentor-ai.vercel.app` | Your Vercel frontend URL, used in reset email links |
| `CLIENT_URL` | `https://classmentor-ai.vercel.app` | Used to configure CORS to allow only this origin |

**How to generate a strong JWT_SECRET:**
In your terminal, run: `node -e "console.log(require('crypto').randomBytes(32).toString('hex'))"`
This generates a 64-character random hex string — impossible to guess or brute-force.

> ⚠️ **Never set these in your code or commit them to Git.** Environment variables are meant to be secret. If `JWT_SECRET` is exposed, anyone can forge tokens and access any user's account.

---

## Slide 9 — Configuring CORS for Production

**CORS (Cross-Origin Resource Sharing)** is a browser security feature that blocks requests between different origins by default. An "origin" is the combination of protocol + domain + port.

**The problem without CORS configured:**
```
Browser at: https://classmentor-ai.vercel.app (origin A)
Makes a request to: https://classmentor-api.onrender.com/api/auth/login (origin B)

Browser first sends a "preflight" OPTIONS request to check if this cross-origin call is allowed.
Server with open cors() returns: "Access-Control-Allow-Origin: *" (allow all)
Browser proceeds with the actual POST request.
```

In development, `app.use(cors())` allows all origins — which is fine for local testing. In production, only your Vercel frontend should be allowed to call your backend. Everything else should be blocked.

**Configuring CORS for production:**

```js
// server/server.js — update the CORS configuration

const allowedOrigins = [
  'http://localhost:3000',                     // Development frontend
  'http://localhost:5173',                     // Vite default port (alternative)
  process.env.CLIENT_URL,                      // Production frontend URL from env var
].filter(Boolean); // Remove undefined values if CLIENT_URL isn't set

app.use(cors({
  origin: (requestOrigin, callback) => {
    // Allow requests with no origin (mobile apps, Postman, curl, server-to-server)
    if (!requestOrigin) return callback(null, true);

    if (allowedOrigins.includes(requestOrigin)) {
      // This origin is allowed — respond with the origin in the header
      callback(null, requestOrigin);
    } else {
      // This origin is not on our list — reject the request
      callback(new Error(`CORS: Origin ${requestOrigin} not allowed`));
    }
  },
  credentials: true, // Allow cookies and Authorization headers to be sent cross-origin
}));
```

**What happens without CORS configured correctly in production:**
The browser will deny the request before it even leaves the browser, and the user sees a cryptic "Network Error" with no useful information — one of the most common deployment debugging issues.

---

## Slide 10 — What is CORS? (The Full Explanation)

Understanding CORS is essential because it's the source of many deployment headaches. Here's exactly what happens under the hood:

**Step 1: The browser sends a "preflight" request first**
Before making the actual login request, the browser sends an HTTP OPTIONS request to check if the server allows this:

```
Browser sends:
OPTIONS /api/auth/login
Origin: https://classmentor-ai.vercel.app
Access-Control-Request-Method: POST
Access-Control-Request-Headers: authorization, content-type
```

**Step 2: Your server (with CORS middleware) responds:**
```
Allow: GET,HEAD,PUT,PATCH,POST,DELETE
Access-Control-Allow-Origin: https://classmentor-ai.vercel.app
Access-Control-Allow-Headers: authorization, content-type
Access-Control-Allow-Credentials: true
```

**Step 3: The browser sees the "okay" and sends the real request:**
```
POST /api/auth/login
Origin: https://classmentor-ai.vercel.app
Authorization: Bearer eyJhbG...
Content-Type: application/json
{ "email": "...", "password": "..." }
```

**If the server doesn't include `Access-Control-Allow-Origin`:**
The browser sees the permission was not granted and blocks the response from reaching your JavaScript code. The request technically reached the server, but the browser hides the response — causing your Axios promise to reject with "Network Error" instead of a useful message.

> 💡 The `cors` npm package handles ALL of this automatically when configured correctly. You just need to tell it which origins are allowed.

---

## Slide 11 — Render Cold Starts: What to Expect

Render's free tier is generous but has one important limitation you must understand before your first demo:

**What is a cold start?**

When no requests have been sent to your Render service for **15 minutes**, Render automatically "spins down" the server process to save computing resources. When the next request arrives, Render has to wake it up — which takes 30 to 60 seconds.

**The user experience during a cold start:**
The user clicks "Login" → the spinner appears → nothing happens for 30-60 seconds → then the login works normally. This is jarring if you don't expect it.

**How to handle it:**

**Option 1 — Show a friendly loading message:**
```jsx
// In your Login.jsx, if the request takes longer than 5 seconds,
// show a message explaining what's happening
const [isSlowLoad, setIsSlowLoad] = useState(false);

useEffect(() => {
  const timer = setTimeout(() => {
    if (loading) setIsSlowLoad(true);
  }, 5000);
  return () => clearTimeout(timer);
}, [loading]);

// In the JSX:
{isSlowLoad && (
  <p style={{ color: 'orange', fontSize: '12px' }}>
    Server is waking up — this may take up to 60 seconds on first load...
  </p>
)}
```

**Option 2 — Ping the health check every 14 minutes:**
You can set up a free cron job service (like cron-job.org) to call `GET /api/health` every 14 minutes. This keeps the server awake and prevents cold starts. It's a common workaround for Render's free tier.

**Option 3 — Upgrade to Render's paid plan:**
The Starter plan ($7/month) keeps the server always on — no cold starts.

---

## Slide 12 — MongoDB Atlas Network Access for Production

MongoDB Atlas has a security feature called **Network Access** that controls which IP addresses are allowed to connect to your database.

**The problem in production:**
- In development, you added your home IP to the Atlas allowlist
- Render (the cloud server) has **dynamic IP addresses** — they change every time the server restarts
- This means your Render server might not be able to connect to Atlas because its current IP was never allowlisted

**The solution:**
In MongoDB Atlas → Security → Network Access → Add IP Address:
- Click **"Allow Access From Anywhere"** → This adds `0.0.0.0/0`
- This means any IP can attempt a connection — but they still need valid username/password credentials

> ⚠️ **Is this safe?** Yes, for most applications. The connection requires a valid database username and password in the URI. The IP allowlist is an additional layer of security, but for apps on dynamic-IP hosts (Render free tier), allowing all IPs is standard practice. Your database credentials are the real security layer.

**If you're on Render's paid plan**, you can get a static IP and add only that to Atlas — which is more secure but not worth the complexity for a student project.

---

## Slide 13 — Gmail App Password for SMTP

If you want the password-reset email feature to work in production, you need to configure your Gmail account to allow SMTP access from your Render server. Regular Gmail accounts block direct password logins from non-Google apps for security.

**Why you can't use your real Gmail password:**
Google calls this "Less Secure App Access" and it's disabled by default. Even if you enable it, Google may block it intermittently. The official solution is to use an "App Password" — a special 16-character token that works only for SMTP.

**How to generate a Gmail App Password:**
1. Go to your Google Account at [myaccount.google.com](https://myaccount.google.com)
2. Click **"Security"** in the left sidebar
3. Under "How you sign in to Google" → enable **"2-Step Verification"** if not already on
4. After enabling 2-Step Verification, search for **"App Passwords"** in the search bar
5. Click "App Passwords" → Select app: "Mail" → Select device: "Other" → type "classmentor"
6. Click "Generate" → you get a **16-character password** like `xqjk mplr abcd efgh`
7. Copy this (it's shown only once) and paste it as `SMTP_PASS` in Render's environment variables

```bash
# Render environment variables for email:
SMTP_HOST=smtp.gmail.com
SMTP_PORT=465        # Use 465 for SSL — more reliable than 587 for Gmail
SMTP_USER=youraddress@gmail.com
SMTP_PASS=xqjk mplr abcd efgh    # The App Password (spaces are fine — Gmail ignores them)
EMAIL_FROM=classMentor AI <youraddress@gmail.com>
APP_URL=https://classmentor-ai.vercel.app  # Used to build the reset link in the email body
```

> 💡 **What if I skip this?** Our email service has a graceful fallback — if SMTP is not configured, the server logs `[Email mock]` to the console and returns success to the client. The reset password flow will appear to work on the frontend but no actual email will be sent. This is fine for a demo.

---

## Slide 14 — Vercel and Render Auto-Deploy Flow

One of the best features of both platforms is **automatic deployment** — every time you push code to GitHub, both Vercel and Render detect the change and automatically deploy the new version. You never need to manually upload files:

**The automated deployment pipeline:**
```
You make code changes → save files
  ↓
git add . && git commit -m "Fix bug in chat sidebar"
  ↓
git push origin main
  ↓
GitHub receives the new commit
  ↓
GitHub notifies Vercel via webhook  AND  GitHub notifies Render via webhook
  ↓                                       ↓
Vercel pulls the code               Render pulls the code
Runs: npm install (in client/)      Runs: npm install (in server/)
Runs: npm run build                 Starts: node server.js
Uploads dist/ to CDN                New server process is live
New URL is live in ~30 seconds      New version deployed in ~2 minutes
```

**Vercel also creates Preview Deployments:**
Every Git branch or pull request gets its own unique preview URL (e.g. `https://classmentor-ai-git-feature-xyz.vercel.app`). This lets you test a new feature in a live-like environment before merging it to `main`.

---

## Slide 15 — Deployment Checklist

Before calling your deployment done, go through this checklist to make sure everything is properly configured:

**✅ Backend on Render:**
- [ ] Service type: Web Service (not Background Worker or Cron Job)
- [ ] Root directory correctly set to `server`
- [ ] Build command: `npm install`
- [ ] Start command: `node server.js`
- [ ] All environment variables set (MONGODB_URI, JWT_SECRET, GROQ_API_KEY, SMTP_*, APP_URL)
- [ ] `NODE_ENV=production` is set
- [ ] `USE_MOCK_AI=false` is set
- [ ] Test the health check: `curl https://your-api.onrender.com/api/health`

**✅ Database on MongoDB Atlas:**
- [ ] M0 free cluster created
- [ ] Database user created with read/write access
- [ ] Network Access → `0.0.0.0/0` added (allow all IPs for Render)
- [ ] Connection string tested and working

**✅ Frontend on Vercel:**
- [ ] Root directory: `client`
- [ ] Framework preset: Vite
- [ ] Build command: `npm run build`
- [ ] Output directory: `dist`
- [ ] `VITE_API_URL` environment variable set to Render backend URL
- [ ] Re-deploy triggered after setting environment variables

**✅ Final end-to-end test:**
- [ ] Register a new account
- [ ] Log in with those credentials
- [ ] Create a chat and send a message — get back an AI response
- [ ] Test voice input (if Groq key is active)
- [ ] Test the forgot/reset password flow (if SMTP is configured)
- [ ] Log out and confirm redirect to login

---

## Slide 16 — Security Best Practices for Production

The basic app works, but a production app should have some additional security hardening. Here are the most important improvements to add after your initial deploy:

**1. Add HTTP security headers with `helmet`:**
```bash
npm install helmet
```
```js
import helmet from 'helmet';
// Add this BEFORE all other middleware in server.js
app.use(helmet());
// Helmet automatically sets headers like:
// X-XSS-Protection: prevents basic XSS attacks
// X-Content-Type-Options: prevents MIME sniffing
// X-Frame-Options: prevents clickjacking
// Content-Security-Policy: controls what resources can be loaded
```

**2. Add rate limiting to prevent brute force attacks:**
```bash
npm install express-rate-limit
```
```js
import rateLimit from 'express-rate-limit';

// Apply strict limits to auth routes — prevent password guessing
const authLimiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15-minute window
  max: 20,                   // Maximum 20 attempts per IP per window
  message: 'Too many requests from this IP, please try again in 15 minutes.',
  standardHeaders: true,     // Include rate limit info in response headers
});

app.use('/api/auth', authLimiter, authRoutes); // Apply only to auth routes

// Less strict limit for other API routes
const apiLimiter = rateLimit({ windowMs: 15 * 60 * 1000, max: 200 });
app.use('/api', apiLimiter);
```

**3. Other production security practices:**
- ✅ Never log JWT tokens or passwords — even in error logs
- ✅ Keep all npm packages updated — `npm audit` checks for known vulnerabilities
- ✅ Use HTTPS only — both Vercel and Render enforce this automatically
- ✅ Set `SameSite` cookie attributes if you ever use cookies instead of localStorage

---

## Slide 17 — Git Tagging: Your Project History

One of the most valuable teaching tools in this project is the Git tag system. It lets students (and you) travel back to any exact point in the masterclass series to see what the project looked like at that stage.

```bash
# After completing and testing each masterclass:

# Tag Masterclass 4 (current masterclass)
git add .
git commit -m "Masterclass 4 complete - deployed to Vercel and Render"
git tag masterclass-4               # Create a local tag pointing to this commit
git push origin main                # Push the code changes
git push origin masterclass-4       # Push the tag itself (separate command needed!)
```

**The complete tag history when all 4 masterclasses are done:**
```
masterclass-1  ← UI only, no backend, mock API calls
masterclass-2  ← Working backend, AI integration, no database persistence
masterclass-3  ← Full stack with MongoDB, data persists
masterclass-4  ← Deployed to cloud, live URL, production config
```

**How students can check out any specific point:**
```bash
# See the app as it was after Masterclass 1 (UI only)
git checkout masterclass-1

# Return to the latest version
git checkout main

# See what files changed between Masterclass 2 and 3
git diff masterclass-2 masterclass-3 --stat
```

---

## Slide 18 — Common Deployment Errors and How to Fix Them

Deployment debugging is a skill in itself. Here are the most common issues students run into:

| Error | What it means | How to fix it |
|-------|--------------|--------------|
| `CORS error in browser` | Server isn't allowing your Vercel origin | Set `CLIENT_URL` env var on Render and update your CORS config |
| `Cannot connect to MongoDB` | Atlas Network Access is blocking Render's IP | Add `0.0.0.0/0` to Atlas Network Access allowlist |
| `Invalid or missing API key` | GROQ_API_KEY not set in Render env vars | Add `GROQ_API_KEY` in Render → Environment tab |
| `Application error on Render` | Server crashed on startup (check logs) | Go to Render → Logs tab to see the actual error message |
| `Blank page on Vercel` | Build failed or wrong output directory | Check Vercel → Deployments → Build Logs for the error |
| `404 on page refresh` | Single-page app routing not configured | In Vercel, add a rewrite rule: source `/.*` → destination `/index.html` |
| `slow first request` | Cold start on Render free tier | Expected behaviour — explain to users or use a keepalive ping |
| `Email not received` | SMTP not configured or wrong App Password | Check `SMTP_PASS` is the 16-char App Password, not your Gmail password |

**The golden rule of debugging deployment:**
Always check the **server logs first** (Render's Logs tab). 90% of deployment issues show a clear error message in the logs — the browser error is often too generic to be helpful.

---

## Slide 19 — Key Concepts Covered Today

**✅ Vercel for React deployment**
Vite builds the React app into static files (HTML/JS/CSS). Vercel hosts them on a global CDN with automatic deploys on every Git push. HTTPS is automatic. Preview URLs for every branch.

**✅ Render for Node.js deployment**
Render runs your Express server as a permanent process. Auto-deploys from Git. Environment variables are set in the dashboard. Free tier has cold starts after 15 minutes of inactivity.

**✅ Separating frontend and backend deployment**
Frontend and backend are separate Git deployments on separate platforms. They communicate via HTTP as they do in development — the only difference is the URLs are real domains.

**✅ Production environment variables**
Secrets never live in code. They're set in platform dashboards and injected at runtime (Render) or build time (Vercel with `VITE_*` prefix). Each environment (dev, staging, prod) has its own values.

**✅ CORS in production**
Only your Vercel origin should be able to call your Render API. The `cors` package handles the required HTTP headers. Misconfigured CORS is one of the most common deployment bugs.

**✅ MongoDB Atlas for production**
Network Access must allow `0.0.0.0/0` for Render's dynamic IPs. Always use a separate production database user with a strong password.

**✅ Gmail App Password for SMTP**
Real Gmail password won't work for SMTP. Generate a 16-character App Password in Google Account settings. The app works gracefully even without email configured.

**✅ Security additions**
`helmet` for HTTP security headers. `express-rate-limit` to prevent brute-force attacks. These are production essentials that can be added quickly after the initial deployment.

---

## Slide 20 — You Built This! — Full Recap

Let's take a moment to appreciate everything you've built across all 4 masterclasses:

**Masterclass 1 — The Face:**
A complete React application with Vite, 7 full pages (login, register, forgot/reset password, home, chat, profile), 9 reusable components, two global context providers (Auth and Theme), a working voice input using the browser's MediaRecorder API, and an Axios API service layer with token injection and 401 auto-logout.

**Masterclass 2 — The Brain:**
A Node.js/Express REST API with 13 endpoints across 3 namespaces (auth, chats, voice). JWT-based stateless authentication. Groq AI integration for both chat completions (Llama 3.1) and speech-to-text (Whisper). Multer for binary audio file uploads. express-validator for input validation. Nodemailer for password-reset emails. A mock AI mode for teaching without a live API key.

**Masterclass 3 — The Memory:**
MongoDB Atlas cluster with two Mongoose models. User model with automatic password hashing via a pre-save hook, unique email index, and hidden password field. Chat model with a referenced user, a default title, and an embedded messages array with role validation. Full CRUD operations, regex-based search across chat history, and cascading deletes.

**Masterclass 4 — The Launch:**
Deployed to the real internet. React frontend on Vercel's global CDN with automatic deploys. Express backend on Render with all secrets safely stored as environment variables. CORS properly configured. MongoDB Atlas network access configured for cloud IPs. Gmail App Password for production email. Security improvements with `helmet` and rate limiting.

**Your app is live. Anyone with the URL can use it.** 🌍

---

## Slide 21 — What to Learn Next

Now that you've completed the full MERN stack journey, here are the natural next steps to level up your skills:

| Topic | What to learn | Why it matters |
|-------|--------------|----------------|
| **Streaming AI responses** | Server-Sent Events (SSE) | Show AI responses word-by-word like ChatGPT does, instead of waiting for the full response |
| **Testing** | Jest + React Testing Library + Supertest | Catch bugs before users do; write tests for controllers, components, and API routes |
| **Real-time features** | WebSockets with Socket.io | Build features like "AI is typing..." indicators or live chat between users |
| **Authentication upgrades** | Refresh tokens, OAuth (Google login) | Shorter-lived access tokens with refresh tokens; let users "Continue with Google" |
| **Payments** | Stripe API | Add a premium subscription tier with usage limits |
| **Performance** | Redis caching | Cache frequent database queries; reduce API response time |
| **Containerisation** | Docker + Docker Compose | Package the app so it runs identically on any machine — development, staging, production |
| **CI/CD** | GitHub Actions | Automatically run tests and deploy only if tests pass |

---

## Slide 22 — Q&A / Final Recap

> What did we deploy today?

1. **React frontend live on Vercel** — Global CDN, HTTPS, instant deploys on any `git push`
2. **Express backend live on Render** — Node.js process, all env vars securely configured
3. **MongoDB Atlas connected** — Persistent cloud database, network access for Render
4. **CORS properly configured** — Only the Vercel frontend origin can call the backend API
5. **All secrets in environment variables** — Nothing hardcoded, nothing in Git
6. **Security hardening applied** — `helmet` for HTTP headers, `express-rate-limit` for auth routes
7. **Full test** — Register, login, chat with AI, voice input, password reset

**Congratulations! You designed, built, and shipped a full-stack AI web application. That's a genuinely impressive thing.** 🚀

**Questions?** 🎓
