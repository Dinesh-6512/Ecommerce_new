# Local Deployment Guide

This document describes how to deploy, run, and test the full-stack e-commerce application on your local machine.

---

## 1. System Overview & Architecture

To run the application locally, you only need to run two local servers:
1. **FastAPI Backend**: Python REST API service running on port `8000`.
2. **ReactJS Frontend**: Single-page application running on port `3000`.
3. **Database (Neon PostgreSQL)**: Cloud-hosted on [neon.tech](https://neon.tech).  
   *No local PostgreSQL database server needs to be installed or run on your machine.*

```mermaid
graph LR
    Browser([Browser]) -->|http://localhost:3000| Frontend[ReactJS Frontend]
    Frontend -->|http://127.0.0.1:8000/api| Backend[FastAPI Backend]
    Backend -->|SSL / Cloud Connection| Neon[(Neon PostgreSQL Cloud DB)]
```

---

## 2. Required Environment Variables

All configuration is externalized in `.env` files. Credentials are never written directly in code.

### Backend (`backend/.env`)
| Variable | Description | Example Value |
| :--- | :--- | :--- |
| `DATABASE_URL` | Neon PostgreSQL connection string with `postgresql+psycopg://` dialect and SSL. | `postgresql+psycopg://user:pass@ep-xyz.region.aws.neon.tech/neondb?sslmode=require` |
| `SECRET_KEY` | 32-byte secret hexadecimal key for signing and verifying JWT tokens. | `03e92f96768449873ddd53d5752c722c922cba39893ab8d6e35fe69ab897ab2d` |
| `ALGORITHM` | Algorithm used for JWT encoding. | `HS256` |
| `ACCESS_TOKEN_EXPIRE_MINUTES` | Token expiration duration in minutes. | `1440` (24 hours) |

### Frontend (`frontend/.env`)
| Variable | Description | Example Value |
| :--- | :--- | :--- |
| `VITE_API_URL` | The base URL of the FastAPI backend. Used by React's API client. | `http://127.0.0.1:8000` |

---

## 3. Files That Must NEVER Be Uploaded to GitHub

The following files and folders contain sensitive credentials or build artifacts and are protected by [.gitignore](file:///c:/Users/dines/Documents/Ecommerce_new/.gitignore):
* `backend/.env` (Contains your database password and JWT secret key)
* `frontend/.env` (Contains local environment variables)
* `backend/.venv/` (Local Python virtual environment)
* `frontend/node_modules/` (Downloaded npm packages)
* `frontend/dist/` (Production build output)
* `__pycache__/` and `*.pyc` (Python bytecode files)

---

## 4. Backend Setup & Run Instructions

From the project root directory (`Ecommerce_new/`):

### Install Dependencies:
```powershell
# 1. Create Python virtual environment inside backend folder
python -m venv backend\.venv

# 2. Install required backend packages
.\backend\.venv\Scripts\python -m pip install -r backend\requirements.txt
```

### Start the Backend Server:
```powershell
.\backend\.venv\Scripts\uvicorn backend.main:app --port 8000 --reload
```
* **Local Backend URL**: `http://127.0.0.1:8000`
* **Swagger API Documentation**: `http://127.0.0.1:8000/docs`
* **Health Check URL**: `http://127.0.0.1:8000/api/health`

### Restart the Backend:
Press `CTRL + C` in the backend terminal window to stop the server, then run the start command again.

---

## 5. Frontend Setup & Run Instructions

From the project root directory (`Ecommerce_new/`):

### Install Dependencies:
```powershell
cd frontend
npm install
cd ..
```

### Start the Development Server:
```powershell
cd frontend
npm run dev
```
* **Local Frontend URL**: `http://localhost:3000`

### Build Production Bundle:
To test or verify the production bundle:
```powershell
cd frontend
npm run build
```
*(Produces minified and bundled static assets in `frontend/dist/`)*.

---

## 6. How to Verify Everything is Working

1. **Verify Backend Health**:
   Open a browser or run PowerShell:
   ```powershell
   Invoke-RestMethod -Uri http://127.0.0.1:8000/api/health
   ```
   **Expected Response**:
   ```json
   {
     "status": "healthy",
     "database": "connected to Neon PostgreSQL",
     "tables": ["users", "products", "cart", "cart_items"]
   }
   ```

2. **Verify API Documentation**:
   Visit `http://127.0.0.1:8000/docs` in your browser. All authentication, product, and cart endpoints will be visible.

3. **Verify Frontend Storefront**:
   Visit `http://localhost:3000` in your browser. The live product catalog fetched from Neon PostgreSQL will appear on screen.

---

## 7. End-to-End User Verification Workflow

Test the complete flow in the browser or via API:
1. **Register**: Go to `/register` and create an account with email and password.
2. **Login**: Go to `/login` and sign in. Confirm your email appears in the header.
3. **View Products**: On `/`, browse the live items.
4. **Add Product (as Admin)**: Log in with `admin@store.com` / `AdminPass123!`, go to `/add-product`, and submit a new product. Verify it appears on the home page.
5. **Add to Cart**: Click "+ Add to Cart" on any product card.
6. **View Cart**: Go to `/cart`. Verify the selected items, quantities, and calculated total match.
