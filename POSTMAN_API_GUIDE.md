# 📬 classMentor AI — Postman API Guide

> **Base URL:** `http://localhost:5000`
> **Content-Type:** `application/json` (for all JSON body requests)

---

## 🔐 Authentication Note

Most routes require a **Bearer Token** in the Header.
After **Register** or **Login**, you will receive a `token` in the response. Copy that token and use it in every protected request like this:

| Key | Value |
|---|---|
| `Authorization` | `Bearer <paste_your_token_here>` |

---

## ✅ 1. Health Check

| Field | Value |
|---|---|
| **Method** | `GET` |
| **URL** | `http://localhost:5000/api/health` |
| **Auth Required** | ❌ No |
| **Body** | None |

**Expected Response:**
```json
{
  "status": "ok",
  "message": "classMentor AI API is running"
}
```

---

## 👤 AUTH ROUTES — `/api/auth`

---

### 1.1 Register

| Field | Value |
|---|---|
| **Method** | `POST` |
| **URL** | `http://localhost:5000/api/auth/register` |
| **Auth Required** | ❌ No |

**Headers:**
| Key | Value |
|---|---|
| `Content-Type` | `application/json` |

**Body → raw → JSON:**
```json
{
  "name": "John Doe",
  "email": "john@example.com",
  "password": "password123"
}
```

**Expected Response (201):**
```json
{
  "_id": "65f1234abcd...",
  "name": "John Doe",
  "email": "john@example.com",
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
}
```

**Error Responses:**
```json
{ "message": "User already exists with this email" }   // 400
{ "message": "Name, email and password are required" }  // 400
```

---

### 1.2 Login

| Field | Value |
|---|---|
| **Method** | `POST` |
| **URL** | `http://localhost:5000/api/auth/login` |
| **Auth Required** | ❌ No |

**Headers:**
| Key | Value |
|---|---|
| `Content-Type` | `application/json` |

**Body → raw → JSON:**
```json
{
  "email": "john@example.com",
  "password": "password123"
}
```

**Expected Response (200):**
```json
{
  "_id": "65f1234abcd...",
  "name": "John Doe",
  "email": "john@example.com",
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
}
```

**Error Responses:**
```json
{ "message": "Invalid email or password" }  // 401
```

> 💡 **Copy the `token` from this response — you'll need it for all protected routes below.**

---

### 1.3 Get My Profile

| Field | Value |
|---|---|
| **Method** | `GET` |
| **URL** | `http://localhost:5000/api/auth/me` |
| **Auth Required** | ✅ Yes |

**Headers:**
| Key | Value |
|---|---|
| `Authorization` | `Bearer <your_token>` |

**Body:** None

**Expected Response (200):**
```json
{
  "_id": "65f1234abcd...",
  "name": "John Doe",
  "email": "john@example.com",
  "createdAt": "2024-03-20T10:00:00.000Z"
}
```

---

### 1.4 Update Profile

| Field | Value |
|---|---|
| **Method** | `PUT` |
| **URL** | `http://localhost:5000/api/auth/profile` |
| **Auth Required** | ✅ Yes |

**Headers:**
| Key | Value |
|---|---|
| `Authorization` | `Bearer <your_token>` |
| `Content-Type` | `application/json` |

**Body → raw → JSON:**
```json
{
  "name": "John Updated"
}
```

**Expected Response (200):**
```json
{
  "_id": "65f1234abcd...",
  "name": "John Updated",
  "email": "john@example.com"
}
```

---

### 1.5 Delete Account

| Field | Value |
|---|---|
| **Method** | `DELETE` |
| **URL** | `http://localhost:5000/api/auth/me` |
| **Auth Required** | ✅ Yes |

**Headers:**
| Key | Value |
|---|---|
| `Authorization` | `Bearer <your_token>` |

**Body:** None

**Expected Response (200):**
```json
{
  "message": "Account deleted successfully"
}
```

> ⚠️ This also **deletes all chats** associated with the account!

---

### 1.6 Forgot Password

| Field | Value |
|---|---|
| **Method** | `POST` |
| **URL** | `http://localhost:5000/api/auth/forgot-password` |
| **Auth Required** | ❌ No |

**Headers:**
| Key | Value |
|---|---|
| `Content-Type` | `application/json` |

**Body → raw → JSON:**
```json
{
  "email": "john@example.com"
}
```

**Expected Response (200):**
```json
{
  "message": "If an account exists with this email, you will receive a reset link."
}
```

> 💡 A password reset email will be sent to the address. The email contains a **reset token** (JWT). Copy that token for the next step.

---

### 1.7 Reset Password

| Field | Value |
|---|---|
| **Method** | `POST` |
| **URL** | `http://localhost:5000/api/auth/reset-password` |
| **Auth Required** | ❌ No |

**Headers:**
| Key | Value |
|---|---|
| `Content-Type` | `application/json` |

**Body → raw → JSON:**
```json
{
  "token": "<reset_token_from_email>",
  "password": "newPassword123"
}
```

**Expected Response (200):**
```json
{
  "message": "Password reset successful. You can now sign in."
}
```

**Error Responses:**
```json
{ "message": "Invalid or expired reset link" }  // 400 — token expired or invalid
```

---

## 💬 CHAT ROUTES — `/api/chats`

> ⚠️ **All chat routes require Authorization header.**

---

### 2.1 Create New Chat

| Field | Value |
|---|---|
| **Method** | `POST` |
| **URL** | `http://localhost:5000/api/chats` |
| **Auth Required** | ✅ Yes |

**Headers:**
| Key | Value |
|---|---|
| `Authorization` | `Bearer <your_token>` |

**Body:** None (no body needed)

**Expected Response (201):**
```json
{
  "_id": "65f9876xyz...",
  "user": "65f1234abcd...",
  "title": "New Chat",
  "messages": [],
  "createdAt": "2024-03-20T10:00:00.000Z",
  "updatedAt": "2024-03-20T10:00:00.000Z"
}
```

> 💡 **Save the `_id` from this response** — you'll need it as `<chat_id>` in the routes below.

---

### 2.2 Get All Chats

| Field | Value |
|---|---|
| **Method** | `GET` |
| **URL** | `http://localhost:5000/api/chats` |
| **Auth Required** | ✅ Yes |

**Headers:**
| Key | Value |
|---|---|
| `Authorization` | `Bearer <your_token>` |

**Optional Query Param (search):**
```
http://localhost:5000/api/chats?search=javascript
```

**Body:** None

**Expected Response (200):**
```json
[
  {
    "_id": "65f9876xyz...",
    "title": "What is JavaScript?",
    "updatedAt": "2024-03-20T10:05:00.000Z"
  },
  {
    "_id": "65f9877abc...",
    "title": "New Chat",
    "updatedAt": "2024-03-20T09:50:00.000Z"
  }
]
```

---

### 2.3 Get Single Chat (with all messages)

| Field | Value |
|---|---|
| **Method** | `GET` |
| **URL** | `http://localhost:5000/api/chats/<chat_id>` |
| **Auth Required** | ✅ Yes |

**Example URL:**
```
http://localhost:5000/api/chats/65f9876xyz
```

**Headers:**
| Key | Value |
|---|---|
| `Authorization` | `Bearer <your_token>` |

**Body:** None

**Expected Response (200):**
```json
{
  "_id": "65f9876xyz...",
  "title": "What is JavaScript?",
  "messages": [
    { "role": "user", "content": "What is JavaScript?" },
    { "role": "assistant", "content": "JavaScript is a programming language..." }
  ]
}
```

---

### 2.4 Send a Message (Chat with AI)

| Field | Value |
|---|---|
| **Method** | `POST` |
| **URL** | `http://localhost:5000/api/chats/<chat_id>/messages` |
| **Auth Required** | ✅ Yes |

**Example URL:**
```
http://localhost:5000/api/chats/65f9876xyz/messages
```

**Headers:**
| Key | Value |
|---|---|
| `Authorization` | `Bearer <your_token>` |
| `Content-Type` | `application/json` |

**Body → raw → JSON:**
```json
{
  "content": "What is JavaScript?"
}
```

**Expected Response (200):**
```json
{
  "userMessage": {
    "role": "user",
    "content": "What is JavaScript?"
  },
  "assistantMessage": {
    "role": "assistant",
    "content": "JavaScript is a lightweight, interpreted programming language..."
  }
}
```

> 💡 The chat title will auto-update to the **first message** you send (max 50 characters).

---

### 2.5 Delete a Chat

| Field | Value |
|---|---|
| **Method** | `DELETE` |
| **URL** | `http://localhost:5000/api/chats/<chat_id>` |
| **Auth Required** | ✅ Yes |

**Example URL:**
```
http://localhost:5000/api/chats/65f9876xyz
```

**Headers:**
| Key | Value |
|---|---|
| `Authorization` | `Bearer <your_token>` |

**Body:** None

**Expected Response (200):**
```json
{
  "message": "Chat deleted"
}
```

---

## 🎙️ VOICE ROUTES — `/api/voice`

> ⚠️ **Requires Authorization. Uses `form-data`, NOT JSON.**

---

### 3.1 Transcribe Audio

| Field | Value |
|---|---|
| **Method** | `POST` |
| **URL** | `http://localhost:5000/api/voice/transcribe` |
| **Auth Required** | ✅ Yes |

**Headers:**
| Key | Value |
|---|---|
| `Authorization` | `Bearer <your_token>` |

> ⚠️ Do **NOT** manually set `Content-Type` here — Postman sets it automatically when using form-data.

**Body → form-data:**
| Key | Type | Value |
|---|---|---|
| `audio` | **File** | Select an audio file from your computer |

**Supported Audio Formats:**
- `.webm`, `.mp3`, `.mpeg`, `.mp4`, `.m4a`, `.ogg`, `.wav`, `.flac`
- **Max size:** 25 MB

**How to set it up in Postman:**
1. Go to **Body** tab
2. Select **form-data**
3. In the Key column, type `audio`
4. Click the **dropdown** next to the key (it shows "Text") and change it to **File**
5. In the Value column, click **Select Files** and choose your audio file
6. Hit **Send**

**Expected Response (200):**
```json
{
  "transcription": "Hello, can you explain what React hooks are?"
}
```

**Error Responses:**
```json
{ "message": "Invalid audio type. Use webm, mp3, wav, etc." }  // 400
{ "message": "Invalid audio upload" }                           // 400
```

---

## 🔁 Recommended Test Flow in Postman

Follow this order to test all APIs end-to-end:

```
1. GET  /api/health                    → Verify server is running
2. POST /api/auth/register             → Create account, copy token
3. POST /api/auth/login                → Login, copy token
4. GET  /api/auth/me                   → View your profile
5. PUT  /api/auth/profile              → Update your name
6. POST /api/chats                     → Create a new chat, copy chat _id
7. POST /api/chats/:id/messages        → Send a message, get AI response
8. GET  /api/chats                     → See all your chats
9. GET  /api/chats/:id                 → See full chat with messages
10. POST /api/voice/transcribe         → Upload audio, get text
11. DELETE /api/chats/:id              → Delete a chat
12. POST /api/auth/forgot-password     → Trigger password reset email
13. POST /api/auth/reset-password      → Reset password using token from email
14. DELETE /api/auth/me                → Delete account (last step!)
```

---

## ⚡ Setting Up Environment in Postman (Recommended)

Create a **Postman Environment** to avoid copy-pasting the token every time:

1. Click **Environments** → **Add**
2. Name it `classMentor Local`
3. Add these variables:

| Variable | Initial Value |
|---|---|
| `base_url` | `http://localhost:5000` |
| `token` | *(leave empty — paste after login)* |
| `chat_id` | *(leave empty — paste after creating chat)* |

4. In your requests use `{{base_url}}`, `{{token}}`, `{{chat_id}}` as placeholders
5. Example URL: `{{base_url}}/api/chats/{{chat_id}}/messages`
6. Example Header: `Authorization: Bearer {{token}}`
