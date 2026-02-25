# Git Workflow for Masterclasses

Use this workflow to push your code after each masterclass so you can reference it while teaching.

## Option 1: Using Tags (Recommended)

After completing each masterclass:

```bash
git add .
git commit -m "Masterclass X complete: [brief description]"
git tag masterclass-X
git push origin main
git push origin masterclass-X
```

To view code at a specific masterclass:
```bash
git checkout masterclass-3
```

To return to latest:
```bash
git checkout main
```

## Option 2: Using Branches

```bash
git checkout -b masterclass-1
git add .
git commit -m "Masterclass 1: Project Setup & Backend Foundation"
git push origin masterclass-1

# For masterclass 2, branch from masterclass 1:
git checkout masterclass-1
git checkout -b masterclass-2
# ... make changes ...
git add .
git commit -m "Masterclass 2: Authentication"
git push origin masterclass-2

# And so on...
```

## Tag/Branch Reference

| Tag/Branch     | Content |
|----------------|---------|
| masterclass-1  | Project setup, Express server, MongoDB, User model |
| masterclass-2  | + JWT auth, register, login, protected routes |
| masterclass-3  | + React app, Auth UI, Login/Register pages |
| masterclass-4  | + Chat model, OpenAI integration, chat API |
| masterclass-5  | + Full chat UI, sidebar, messages, polish |

## Initial Setup (First Time Only)

```bash
cd "e:\Masterclass Project"
git init
git add .
git commit -m "Initial commit - classMentor AI"
git branch -M main
git remote add origin <your-repo-url>
git push -u origin main
```
