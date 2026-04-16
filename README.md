# The Project — Notes App (Django REST + React/Vite)

A full‑stack notes app:

- **Backend**: Django + Django REST Framework + SimpleJWT (auth) + Postgres (via `DATABASE_URL`)
- **Frontend**: React + Vite + Axios + React Router

## Features

- **User registration**
- **JWT login** (access/refresh)
- **Create / list / delete notes** (per-user; authenticated)
- **Health check** endpoint for uptime monitoring

## Django REST Framework & serializers

This backend is built with **Django REST Framework (DRF)**:

- **API views** are DRF generic views (for example `ListCreateAPIView`, `DestroyAPIView`, `CreateAPIView`) that handle common REST patterns.
- **Serializers** (in `backend/api/serializers.py`) define how model/user data is:
  - **validated** when the frontend sends JSON (request body)
  - **serialized** into JSON when the backend returns responses

In this project, `NoteSerializer` is used for the notes endpoints, and `UserSerializer` is used for registration.

## Project structure

```text
backend/                     Django project (API)
  manage.py                  Django entry point (runserver/migrations)
  backend/                   Django settings/urls/wsgi package
  api/                       Notes API app (models, serializers, views, urls)

frontend/                    React app (Vite)
  src/
    api.js                   Axios client + JWT auth header interceptor
    pages/                   Page-level components (e.g. `Home.jsx`)
    components/              Reusable UI components (e.g. `Note`)

render.yaml                  Render deployment config (backend)
```

## API endpoints

Base URL: your backend host (local: `http://127.0.0.1:8000`)

- **Health**: `GET /` → `{ "status": "ok" }`
- **Register**: `POST /api/user/register/`
- **Token**: `POST /api/token/`
- **Refresh**: `POST /api/token/refresh/`
- **Notes**
  - `GET /api/notes/` (list current user notes)
  - `POST /api/notes/` (create note: `{ "title": "...", "content": "..." }`)
  - `DELETE /api/notes/delete/<id>/`

## How the frontend communicates with the backend

- **HTTP client**: the frontend uses an Axios instance in `frontend/src/api.js`.
- **Base URL**: Axios `baseURL` is set to `import.meta.env.VITE_API_URL` (if provided) and otherwise defaults to `https://the-project-kdip.onrender.com/`.
- **Auth header**: on every request, an interceptor reads the access token from `localStorage` (key: `ACCESS_TOKEN`) and sets:
  - `Authorization: Bearer <token>`
- **API calls**: pages (e.g. `frontend/src/pages/Home.jsx`) call backend routes like `GET /api/notes/`, `POST /api/notes/`, and `DELETE /api/notes/delete/<id>/`.

## Local development

### Backend (Django)

Prereqs: **Python 3.x**, (optional) **Postgres** if you don’t want to use a hosted DB.

From `backend/`:

```bash
python -m venv .venv
# Windows (PowerShell)
.\.venv\Scripts\Activate.ps1

pip install -r requirements.txt

# set env vars (see "Environment variables" below), then:
python manage.py makemigrations
python manage.py migrate
python manage.py runserver
```

Backend runs at `http://127.0.0.1:8000`.

### Frontend (React/Vite)

Prereqs: **Node.js 18+** (recommended).

From `frontend/`:

```bash
npm install

# optional: point to local backend
# (PowerShell) $env:VITE_API_URL="http://127.0.0.1:8000"

npm run dev
```

Frontend runs at `http://localhost:5173`.

## Environment variables

### Backend

The backend reads these variables (see `backend/backend/settings.py` and `render.yaml`):

- **`SECRET_KEY`**: required
- **`DEBUG`**: `true` / `false` (defaults to `false`)
- **`DATABASE_URL`**: required (Postgres URL, e.g. `postgres://user:pass@host:5432/dbname`)

### Frontend

- **`VITE_API_URL`**: optional; API base URL (defaults to `https://the-project-kdip.onrender.com/`)

## Notes on CORS (local dev)

The backend currently allows CORS for the deployed frontend origin (`https://the-project-frontend.onrender.com`).
If you run the frontend locally, you may need to add `http://localhost:5173` to `CORS_ALLOWED_ORIGINS` in `backend/backend/settings.py`.

## Deployment (Render)

This repo includes `render.yaml` to deploy the **backend** to Render using Gunicorn and to run migrations during build.
Set the Render secrets/variables referenced in `render.yaml`:

- `RENDER_SECRET_KEY` → `SECRET_KEY`
- `RENDER_DEBUG` → `DEBUG`
- `RENDER_DB_URL` → `DATABASE_URL`

