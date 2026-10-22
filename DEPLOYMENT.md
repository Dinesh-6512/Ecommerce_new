# Deployment Guide: Local & Render Cloud

This document provides complete, step-by-step instructions for deploying our full-stack e-commerce application locally and to the **Render** cloud platform.

---

## 1. Cloud Architecture & Deployment Order

The cloud deployment consists of **three parts**, deployed in this exact sequence:

```mermaid
graph TD
    subgraph Step 1: Database
    A[(Neon PostgreSQL Cloud DB)]
    end

    subgraph Step 2: Backend
    B[Render Web Service: FastAPI] -->|DATABASE_URL via SSL| A
    end

    subgraph Step 3: Frontend
    C[Render Static Site: ReactJS] -->|VITE_API_URL| B
    end

    Browser([Customer Browser]) -->|HTTPS| C
```

1. **Step 1: PostgreSQL Database on Neon** (Already cloud-hosted; zero server maintenance).
2. **Step 2: FastAPI Backend on Render** (Deploys as a Python **Web Service**).
3. **Step 3: ReactJS Frontend on Render** (Deploys as a fast, global **Static Site**).

---

## 2. GitHub Integration & Security Verification

Before deploying to Render, the project is pushed to GitHub. Render automatically detects commits and builds the latest version.

### 1. Main `.gitignore` Protection
The project includes a root [.gitignore](file:///c:/Users/dines/Documents/Ecommerce_new/.gitignore) ensuring secrets and local artifacts are never committed:
* `.env`, `backend/.env`, `frontend/.env`, `*.env` (Hides all passwords and secret keys)
* `venv/`, `.venv/` (Hides local Python virtual environments)
* `__pycache__/`, `*.pyc` (Hides Python bytecode cache)
* `node_modules/` (Hides downloaded npm packages)
* `dist/` (Hides compiled production builds)

### 2. Pushing the Project to GitHub
1. Go to [github.com](https://github.com) and click **"New repository"**.
2. Name it (for example: `ecommerce-app`), select **Public** or **Private**, and click **"Create repository"** (do NOT initialize with README).
3. In your local terminal, link and push your code:
   ```powershell
   git remote add origin https://github.com/<your-username>/ecommerce-app.git
   git branch -M main
   git push -u origin main
   ```

### 3. Verify No Secrets on GitHub
Visit your GitHub repository in your browser:
* Confirm `backend/.env` is **NOT** present in the file tree.
* Confirm `frontend/.env` is **NOT** present in the file tree.
* Confirm no passwords, database links, or secret keys appear in any committed file.

---

## 3. Neon PostgreSQL Database Integration

### 1. Create Neon Project & Database
1. Go to [neon.tech](https://neon.tech) and log in.
2. Click **"Create Project"**:
   * **Project Name**: `ecommerce-db`
   * **Region**: Select the region closest to you (e.g., AWS US East, Singapore, or Frankfurt).
   * Click **"Create"**.

### 2. Copy the Connection String
1. On your Neon project dashboard, click the **Connect** button at the top.
2. Set **Connection pooling** to **ON** (recommended for scalable web apps).
3. Copy the pooled connection string.

### 3. Change Prefix for psycopg (v3) Driver
Neon provides a connection string starting with `postgresql://`. Because our [backend/requirements.txt](file:///c:/Users/dines/Documents/Ecommerce_new/backend/requirements.txt) uses the high-performance modern **`psycopg` (version 3)** driver, update the beginning of the URL:
* **Original Neon Link**:
  ```text
  postgresql://user:password@ep-xyz-pooler.region.aws.neon.tech/neondb?sslmode=require
  ```
* **Required SQLAlchemy Format**:
  ```text
  postgresql+psycopg://user:password@ep-xyz-pooler.region.aws.neon.tech/neondb?sslmode=require
  ```
*(Ensure `?sslmode=require` remains at the end).*

### 4. Local Testing with Neon
Add the connection string to `backend/.env`:
```env
DATABASE_URL=postgresql+psycopg://user:password@ep-xyz-pooler.region.aws.neon.tech/neondb?sslmode=require
```
When FastAPI starts, it connects over SSL, creates all 4 tables (`users`, `products`, `cart`, `cart_items`), and seeds sample products. The health check confirms this:
```powershell
Invoke-RestMethod -Uri http://127.0.0.1:8000/api/health
```

---

## 4. Render Deployment Settings

### Part 2: FastAPI Backend (Render Web Service)
In the Render dashboard, click **New +** $\rightarrow$ **Web Service**:

| Setting | Value | Why It Is Used |
| :--- | :--- | :--- |
| **Name** | `ecommerce-backend` | Name of your backend service. |
| **Language** | `Python` | Runtime environment. |
| **Root Directory** | `backend` | Tells Render to run commands inside the `backend/` folder. |
| **Build Command** | `pip install -r requirements.txt` | Installs FastAPI, SQLAlchemy, psycopg, pyjwt, bcrypt. |
| **Start Command** | `uvicorn main:app --host 0.0.0.0 --port $PORT` | Starts the production ASGI server on the port assigned by Render. |
| **Instance Type** | `Free` | Free tier web service. |

> [!NOTE]
> Once deployed, Render provides a public HTTPS URL (e.g., `https://ecommerce-backend.onrender.com`).  
> Verify it by visiting `https://ecommerce-backend.onrender.com/api/health`.

---

### Part 3: Render Frontend Deployment (Render Static Site)

Follow these exact steps to deploy the ReactJS frontend on Render and connect it to your live FastAPI backend:

#### RENDER FRONTEND STEPS:
1. In Render, click **New +** at the top right and choose **Static Site**.
2. Select the same GitHub repository (`Dinesh-6512/Ecommerce_new`) and click **Connect**.
3. Fill in the settings:
   * **Name**: `ecommerce-frontend`
   * **Branch**: `main`
   * **Root Directory**: `frontend` (the real frontend folder)
   * **Build Command**: `npm install && npm run build`
   * **Publish Directory**: `dist`
4. Scroll down to **Environment Variables** and click **Add Environment Variable**. Add:
   * **Key**: `VITE_API_URL`
   * **Value**: `<backend-url>/api`  
     *(Example: `https://ecommerce-backend.onrender.com/api` — the value must end with `/api` and must NOT have a trailing `/` after it).*
5. Click **Create Static Site** and wait until the status shows **Live**.
6. Open the Static Site dashboard, go to the **Redirects / Rewrites** tab and click **Add Rule**:
   * **Source**: `/*`
   * **Destination**: `/index.html`
   * **Action**: `Rewrite`
   * Click **Save Changes**. *(This prevents "Not Found" 404 errors whenever a subpage such as `/cart` or `/login` is refreshed).*
7. Copy your live frontend URL (example: `https://ecommerce-frontend.onrender.com`).
8. If `VITE_API_URL` is changed later, redeploy the frontend (**Manual Deploy** $\rightarrow$ **Clear build cache & deploy**), because Vite reads and embeds environment variables only during the build step.

> [!TIP]
> **Troubleshooting Node Version**: If the Render build fails with a Node version error, add the environment variable `NODE_VERSION = 22` under the **Environment** tab and redeploy.

---

## 5. Environment Variables Table

Set these in the **Environment** tab on Render. Only variable names are shown (never real secrets in documentation):

### Backend Environment Variables (Render Web Service)
| Variable Name | Required Value Description |
| :--- | :--- |
| `DATABASE_URL` | Your Neon connection string with `postgresql+psycopg://` and `?sslmode=require`. |
| `SECRET_KEY` | 32-byte secret hex key generated using `secrets.token_hex(32)`. |
| `ALGORITHM` | `HS256` |
| `ACCESS_TOKEN_EXPIRE_MINUTES` | `1440` (24 hours) |

### Frontend Environment Variables (Render Static Site)
| Variable Name | Required Value Description |
| :--- | :--- |
| `VITE_API_URL` | The live URL of your deployed Render backend (e.g., `https://ecommerce-backend.onrender.com`). |

---

## 6. How the First Admin User is Created

When the backend starts up on Render for the very first time:
1. FastAPI's lifespan startup handler automatically checks the `users` table in Neon.
2. If no admin exists, it automatically creates the default administrator account:
   * **Email**: `admin@store.com`
   * **Password**: `AdminPass123!`
   * **Role**: `is_admin = True`
3. The admin can immediately log in from the frontend at `/login` to access the protected **Add Product** screen and publish items.

---

## 7. Exact Deployment Steps in Order

When ready to deploy online:

1. **Step 1: Push Code to GitHub**
   * Stage and push all project files. Confirm `.env` files are not on GitHub.
2. **Step 2: Neon Cloud Database**
   * Confirm your Neon project exists and copy the pooled connection string.
3. **Step 3: Deploy Backend on Render**
   * Create **Web Service** pointing to your GitHub repository.
   * Set Root Directory to `backend`.
   * Set Build Command to `pip install -r requirements.txt`.
   * Set Start Command to `uvicorn main:app --host 0.0.0.0 --port $PORT`.
   * Add Environment Variables: `DATABASE_URL` and `SECRET_KEY`.
   * Wait for deployment to finish and copy your live backend URL (`https://...onrender.com`).
   * Test `https://<backend-url>/api/health` $\rightarrow$ should return `"status": "healthy"`.
4. **Step 4: Deploy Frontend on Render**
   * Create **Static Site** pointing to your GitHub repository.
   * Set Root Directory to `frontend`.
   * Set Build Command to `npm install && npm run build`.
   * Set Publish Directory to `dist`.
   * Add Environment Variable: `VITE_API_URL = https://<your-backend-url>`.
   * Add Rewrite Rule: `/*` $\rightarrow$ `/index.html` (Rewrite).
5. **Step 5: Verify Live Application**
   * Open the live frontend URL in your browser.
   * Test user flow: Register new customer $\rightarrow$ Login $\rightarrow$ View products $\rightarrow$ Log in as admin $\rightarrow$ Add product $\rightarrow$ Add to cart $\rightarrow$ View cart.
