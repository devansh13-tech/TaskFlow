# 🚀 Complete Deployment Guide: Modules 24, 25 & 26

This guide provides end-to-end instructions for deploying your Flask & SQLite backend to production.

---

## 📋 Overview of Deployment Flow

```
[Local Code in Complete_Project]
            │
            ▼  (Module 24)
      [Git & GitHub]
            │
            ├───► [Render Web Service] (Module 25 - Mandatory / Recommended)
            │         ↳ Runs Gunicorn WSGI 24/7 with SQLite persistence
            │
            └───► [Vercel Serverless] (Module 26 - Optional Demo)
                      ↳ Runs as ephemeral serverless Python functions
```

---

## 📌 PART 1: Git & GitHub Setup (Module 24)

### Step 1: Initialize Git inside the project directory
Open your terminal inside the `Complete_Project` folder:
```bash
cd Task_App/Complete_Project
git init
```

### Step 2: Verify `.gitignore`
Make sure `.gitignore` excludes temporary files and local database files:
```bash
# Check status — venv, *.db, and __pycache__ must NOT be listed
git status
```

### Step 3: Stage and Commit All Files
```bash
git add .
git commit -m "feat: complete TaskFlow backend with SQLite, REST API, and deployment configs"
```

### Step 4: Create a New GitHub Repository
1. Go to [https://github.com](https://github.com) and log in.
2. Click the **+** icon in the top right &rarr; **New repository**.
3. Repository name: `taskflow-backend` (or your preferred name).
4. Set visibility to **Public** (or Private).
5. **Do NOT** check "Add a README file" (we already created one).
6. Click **Create repository**.

### Step 5: Link Local Repo to GitHub & Push
```bash
git branch -M main
git remote add origin https://github.com/YOUR_USERNAME/taskflow-backend.git
git push -u origin main
```

---

## 🌐 PART 2: Deploying to Render (Module 25 — Mandatory)

Render is a modern cloud hosting platform ideal for Flask applications because it runs a persistent server process (Gunicorn) that keeps your SQLite database alive.

### How Render Works with TaskFlow
- **Build Command**: `pip install -r requirements.txt` (Installs Flask, Flask-WTF, WTForms, and Gunicorn).
- **Start Command**: `gunicorn app:app` (Defined in `Procfile` to run a multi-worker production server).

### Step-by-Step Render Deployment:
1. Go to [https://render.com](https://render.com) and create an account (you can sign in with GitHub).
2. On your Render Dashboard, click **New +** &rarr; select **Web Service**.
3. Select **Build and deploy from a Git repository** &rarr; click **Next**.
4. Connect your GitHub account and select your `taskflow-backend` repository.
5. Fill in the deployment details:
   - **Name**: `taskflow-live` (or your custom name)
   - **Region**: Choose the region closest to you (e.g., Singapore, Frankfurt, Oregon)
   - **Branch**: `main`
   - **Root Directory**: (Leave blank if repo root contains `app.py`, or enter `Task_App/Complete_Project` if you pushed the whole workspace)
   - **Runtime**: `Python 3`
   - **Build Command**: `pip install -r requirements.txt`
   - **Start Command**: `gunicorn app:app`
   - **Instance Type**: `Free`
6. Configure Environment Variables (Module 23):
   - Scroll down to **Environment Variables** &rarr; click **Add Environment Variable**.
   - Key: `SECRET_KEY`
     - Value: Generate a random string:
       ```bash
       python -c "import secrets; print(secrets.token_hex(32))"
       ```
   - Key: `FLASK_DEBUG`
     - Value: `False`
7. Click **Create Web Service**.
8. Render will now clone your repository, install dependencies, initialize SQLite, and launch Gunicorn!
9. Once the build completes, Render will provide your public live URL:
   `https://taskflow-live.onrender.com`

---

## ⚡ PART 3: Deploying to Vercel (Module 26 — Optional / Demo)

Vercel is primarily a serverless platform designed for frontend frameworks, but supports Python WSGI applications via `@vercel/python`.

### Understanding Serverless with Flask & SQLite
- On Render, the server runs continuously 24/7.
- On Vercel, requests trigger an **ephemeral serverless container** that spins down after seconds of inactivity.
- **Important Limitation (Module 26)**: Local SQLite database files (`tasks.db`) stored on Vercel's filesystem are **ephemeral** and reset on each cold restart. For true persistence on Vercel, you would connect to a remote cloud database (such as PostgreSQL on Neon or Supabase).

### Configuration in `vercel.json`
We have included `vercel.json`:
```json
{
  "builds": [
    {
      "src": "app.py",
      "use": "@vercel/python"
    }
  ],
  "routes": [
    {
      "src": "/(.*)",
      "dest": "app.py"
    }
  ]
}
```

### Steps to Deploy to Vercel:
1. Go to [https://vercel.com](https://vercel.com) and sign in with GitHub.
2. Click **Add New...** &rarr; **Project**.
3. Import your `taskflow-backend` repository.
4. Set **Environment Variables**:
   - `SECRET_KEY`: your random production key
   - `FLASK_DEBUG`: `False`
5. Click **Deploy**.
6. Vercel will build and provide a live URL: `https://your-project.vercel.app`.

---

## ⚖️ Render vs Vercel: Platform Comparison

| Feature | Render (Web Service) | Vercel (Serverless) |
| :--- | :--- | :--- |
| **Architecture** | Persistent container / VM | On-demand serverless functions |
| **WSGI Server** | Gunicorn (`Procfile`) | `@vercel/python` serverless wrapper |
| **SQLite Support** | ✅ Works with local file persistence | ⚠️ Ephemeral: file resets on cold start |
| **Cold Starts** | Free tier sleeps after 15 min inactivity | Fast cold start (< 1 second) |
| **Best Used For** | Full-stack Flask apps, REST APIs, SQLite | Static sites, Next.js, APIs with cloud DB |
| **Role in Course** | **Mandatory & Recommended** | **Optional / Conceptual Demo** |

---

## 🧪 PART 4: Testing Your Live Backend

Once your app is deployed to Render, test both the Web UI and the REST API endpoints:

### 1. Test the Web Interface
Open your live URL in your browser:
```
https://YOUR-APP.onrender.com
```
- Verify the Dashboard loads with task metric cards.
- Click **+ New Task** and submit a task to confirm WTForms & SQLite write operations.
- Edit and delete a task to confirm full CRUD.

### 2. Test the Live REST APIs with cURL or Postman

#### A. Create a User (POST /api/users)
```bash
curl -X POST https://YOUR-APP.onrender.com/api/users \
  -H "Content-Type: application/json" \
  -d '{"name": "Sarah Connor", "email": "sarah@skynet.com"}'
```
Expected response:
```json
{
  "status": "success",
  "message": "User created successfully.",
  "user": {
    "id": 1,
    "name": "Sarah Connor",
    "email": "sarah@skynet.com",
    "created_at": "..."
  }
}
```

#### B. Create a Task Linked to User (POST /api/tasks)
```bash
curl -X POST https://YOUR-APP.onrender.com/api/tasks \
  -H "Content-Type: application/json" \
  -d '{"title": "Setup CI/CD pipeline", "description": "Automate tests on commit", "status": "pending", "user_id": 1}'
```
Expected response:
```json
{
  "status": "success",
  "message": "Task created successfully.",
  "task": {
    "id": 1,
    "user_id": 1,
    "title": "Setup CI/CD pipeline",
    "description": "Automate tests on commit",
    "status": "pending",
    "user_name": "Sarah Connor",
    "created_at": "..."
  }
}
```

#### C. List All Tasks (GET /api/tasks)
```bash
curl -X GET https://YOUR-APP.onrender.com/api/tasks
```

#### D. Filter Tasks by Status (GET /api/tasks?status=pending)
```bash
curl -X GET "https://YOUR-APP.onrender.com/api/tasks?status=pending"
```

#### E. Test Error Validation (POST /api/tasks with missing title)
```bash
curl -X POST https://YOUR-APP.onrender.com/api/tasks \
  -H "Content-Type: application/json" \
  -d '{"status": "pending"}'
```
Expected response:
```json
{
  "status": "error",
  "error": "Task title is required."
}
```
HTTP Status: `400 Bad Request`.
