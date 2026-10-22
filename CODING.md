# Coding Guidelines & Backend Structure

This document outlines the coding patterns and project organization used in this e-commerce application.

---

## 1. Directory Structure

```text
Ecommerce_new/
├── ARCHITECTURE.md          # System architecture and flow
├── BRD.md                   # Business requirements & scope
├── CODING.md                # Code structure and patterns (this file)
├── UI_UX.md                 # Screen layouts and user journeys
├── .gitignore               # Protects .env, .venv, and build files
│
└── backend/
    ├── .env                 # Environment secrets (DATABASE_URL, SECRET_KEY)
    ├── .venv/               # Python virtual environment
    ├── requirements.txt     # Python dependencies
    ├── database.py          # SQLAlchemy connection to Neon PostgreSQL
    ├── models.py            # SQLAlchemy database tables (User, Product, Cart, CartItem)
    ├── schemas.py           # Pydantic data validation models
    ├── auth.py              # Password hashing, JWT token creation & authentication
    └── main.py              # FastAPI app setup, routes, and error handling
```

---

## 2. Component Responsibilities

1. **`database.py`**:
   * Reads `DATABASE_URL` strictly from `backend/.env`.
   * Manages connection pool to Neon PostgreSQL.
   * Provides `get_db` generator for route dependencies.

2. **`models.py`**:
   * Defines database tables using SQLAlchemy Declarative Base.
   * Tables: `users`, `products`, `cart`, `cart_items`.

3. **`schemas.py`**:
   * Defines Pydantic models for request bodies and API responses.
   * Ensures data sent by client is validated before processing.

4. **`auth.py`**:
   * Hashes and verifies passwords using `bcrypt`.
   * Encodes and decodes JWT access tokens using `pyjwt`.
   * Reads `SECRET_KEY` strictly from `backend/.env`.
   * Provides `get_current_user` dependency for protected routes.

5. **`main.py`**:
   * Initializes FastAPI application and CORS middleware.
   * Hosts REST API endpoints (`/api/health`, `/api/auth/...`, `/api/products...`, `/api/cart...`).

---

## 3. Core Coding Rules

* **Secrets Management**: Credentials (`DATABASE_URL`, `SECRET_KEY`) must **never** be hardcoded. Always load from `backend/.env`.
* **Single Backend**: FastAPI is the only backend. No Node.js or duplicate servers.
* **Simple & Reusable**: Prefer readable, standard Python over complex abstractions.
* **Error Handling**: Use standard `HTTPException` with clear status codes (`400 Bad Request`, `401 Unauthorized`, `404 Not Found`).
* **Authentication**: Use Bearer JWT tokens in the `Authorization: Bearer <token>` header for protected endpoints.
