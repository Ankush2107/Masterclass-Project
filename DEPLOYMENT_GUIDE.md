# 🚀 Deployment Guide — Vercel (Frontend) + Render (Backend)

> **Project:** classMentor AI
> **Frontend:** React + Vite → Deploy on **Vercel**
> **Backend:** Node.js + Express → Deploy on **Render**

---

## ⚠️ Before You Start — Code Changes Already Made

Two files have been updated to make the project production-ready:

1. **`client/src/services/api.js`** — now reads the backend URL from `VITE_API_URL` env variable instead of hardcoded `/api` (which only works locally via Vite proxy)
2. **`client/vercel.json`** — added so that React Router works correctly on Vercel (page refresh won't show 404)

Commit and push these changes first:

```bash
git add .
git commit -m "fix: production api url + vercel config"
git push origin main
```

---

## 🖥️ PART 1 — Deploy Backend on Render

### Step 1: Create a Render Account
1. Go to **[render.com](https://render.com)**
2. Click **Get Started for Free**
3. Sign up with your **GitHub account** (recommended — easier to connect your repo)

---

### Step 2: Create a New Web Service
1. After logging in, click **New +** in the top right
2. Select **Web Service**
3. Click **Connect account** and authorize Render to access your GitHub
4. Find your repo `Masterclass-Project` in the list and click **Connect**

---

### Step 3: Configure the Web Service

Fill in these settings on the configuration page:

| Setting | Value |
|---|---|
| **Name** | `classmentor-ai-backend` |
| **Region** | Choose closest to you (e.g., Singapore for India) |
| **Branch** | `main` |
| **Root Directory** | `server` |
| **Runtime** | `Node` |
| **Build Command** | `npm install` |
| **Start Command** | `npm start` |
| **Instance Type** | `Free` |

---

### Step 4: Add Environment Variables
Scroll down to the **Environment Variables** section. Add each one:

| Key | Value |
|---|---|
| `MONGODB_URI` | Your MongoDB Atlas connection string |
| `JWT_SECRET` | Any long random string (e.g., `mysupersecret123abc!`) |
| `GROQ_API_KEY` | Your Groq API key from console.groq.com |
| `USE_MOCK_AI` | `false` |
| `SMTP_HOST` | `smtp.gmail.com` |
| `SMTP_PORT` | `587` |
| `SMTP_SECURE` | `false` |
| `SMTP_USER` | Your Gmail address |
| `SMTP_PASS` | Your Gmail App Password (16-char) |
| `EMAIL_FROM` | `classMentor AI <your-email@gmail.com>` |
| `APP_URL` | *(leave blank for now — fill after Vercel deploy)* |

> 💡 Do NOT add `PORT` — Render automatically assigns and manages the port.

---

### Step 5: Deploy the Backend
1. Click **Create Web Service**
2. Render will start building — you can watch the logs
3. Wait for status to show ✅ **Live**
4. You will get a URL like:
   ```
   https://classmentor-ai-backend.onrender.com
   ```
5. **Copy this URL** — you need it for the Vercel setup

---

### Step 6: Test the Backend
Open your browser or Postman and hit:
```
GET https://classmentor-ai-backend.onrender.com/api/health
```
Expected response:
```json
{ "status": "ok", "message": "classMentor AI API is running" }
```

> ⚠️ **Free Render services spin down after 15 min of inactivity.** First request after sleep takes ~30 seconds. This is normal on the free plan.

---

## 🌐 PART 2 — Deploy Frontend on Vercel

### Step 1: Create a Vercel Account
1. Go to **[vercel.com](https://vercel.com)**
2. Click **Sign Up**
3. Sign up with your **GitHub account**

---

### Step 2: Import Your Project
1. After login, click **Add New…** → **Project**
2. Find `Masterclass-Project` in the list and click **Import**

---

### Step 3: Configure the Project

| Setting | Value |
|---|---|
| **Framework Preset** | `Vite` (Vercel auto-detects this) |
| **Root Directory** | `client` ← **Very important! Click Edit and type `client`** |
| **Build Command** | `npm run build` |
| **Output Directory** | `dist` |
| **Install Command** | `npm install` |

---

### Step 4: Add Environment Variables
Click **Environment Variables** and add:

| Key | Value |
|---|---|
| `VITE_API_URL` | `https://classmentor-ai-backend.onrender.com/api` |

> ⚠️ The value must include `/api` at the end!

---

### Step 5: Deploy the Frontend
1. Click **Deploy**
2. Wait for the build to complete (usually ~1-2 minutes)
3. You will get a URL like:
   ```
   https://classmentor-ai.vercel.app
   ```
4. **Copy this URL** — go back to Render now

---

### Step 6: Update APP_URL on Render
1. Go back to your **Render dashboard**
2. Click your `classmentor-ai-backend` service
3. Go to **Environment** tab
4. Find `APP_URL` and set it to your Vercel URL:
   ```
   https://classmentor-ai.vercel.app
   ```
5. Click **Save Changes** — Render will auto-redeploy

> This is important because `APP_URL` is used in password reset emails to generate the correct link.

---

## 🧪 PART 3 — Final Testing After Deployment

Open your Vercel URL and test end-to-end:

- [ ] Register a new account
- [ ] Login and verify JWT token is working
- [ ] Create a new chat and send a message (tests AI + MongoDB)
- [ ] Test voice transcription (upload a short audio)
- [ ] Test forgot password (check that email arrives with correct reset link)
- [ ] Reset password using the link in the email

---

## 🔁 How Future Updates Work

You don't need to do anything special after the initial deploy. Both Vercel and Render are connected to your GitHub — any time you push to `main`, they auto-redeploy.

```bash
# Just push your code changes as usual
git add .
git commit -m "your message"
git push origin main
# ✅ Vercel and Render redeploy automatically!
```

---

## ❗ Common Issues & Fixes

| Problem | Fix |
|---|---|
| Render shows build error | Check that **Root Directory** is set to `server` |
| Vercel shows 404 on page refresh | Make sure `client/vercel.json` is committed |
| API calls fail from Vercel | Double-check `VITE_API_URL` ends with `/api`, no trailing slash |
| "Cannot connect to DB" on Render | Check your `MONGODB_URI` — it must be the Atlas URI (not localhost) |
| Password reset link goes to localhost | Update `APP_URL` on Render to your Vercel URL |
| Render takes 30s to respond | Normal on free plan — service was sleeping, try again |
| Vercel shows blank page | Check browser console — likely `VITE_API_URL` is missing or wrong |
