# Green AI — Flask + React

A sustainability dashboard tracking textile wastage in Faisalabad and global
airline fuel wastage, with company login and saved insights.

This is a real, runnable two-part app:

- `backend/` — Flask API: auth (JWT), saved "insights" per company (SQLite via SQLAlchemy),
  and stats endpoints that serve the cited research figures plus a server-computed
  live-counter baseline.
- `frontend/` — React (Vite) single-page app that talks to the API.

All waste/fuel figures are drawn from public research (cited in `backend/routes/stats.py`
and in the UI) and are modelled as live projections, not real sensor feeds.

## Quick start — option A: frontend + backend as two dev servers

### 1. Backend
```bash
cd backend
python -m venv venv
source venv/bin/activate        # Windows: venv\Scripts\activate
pip install -r requirements.txt
cp .env.example .env            # edit SECRET_KEY/JWT_SECRET_KEY for production
python app.py                   # runs on http://localhost:5000, creates green_ai.db
```

### 2. Frontend
```bash
cd frontend
npm install
cp .env.example .env            # VITE_API_URL=http://localhost:5000 for dev
npm run dev                     # runs on http://localhost:5173
```

Open http://localhost:5173 — the frontend calls the Flask API for auth, saved
insights, and stats baselines. Good for day-to-day development (hot reload).

## Quick start — option B: one Flask process serves everything

Flask can serve the built React app directly, so you only run (and deploy)
one process:

```bash
cd frontend
./build_into_backend.sh      # npm install + npm run build, copies dist/ -> ../backend/static

cd ../backend
pip install -r requirements.txt
python app.py                # now serves both the API and the frontend
```

Open http://localhost:5000 — the whole site, API included, is right there.
This is also what the free-host deploy in `DEPLOY.md` uses.

## Production notes
- Swap SQLite for Postgres by changing `SQLALCHEMY_DATABASE_URI` in `backend/config.py`.
- Set real `SECRET_KEY` / `JWT_SECRET_KEY` env vars — the defaults are dev-only.
- Run the backend with `gunicorn app:app` behind a reverse proxy; build the frontend
  with `npm run build` and serve the `dist/` folder as static assets (or host it
  separately and point `VITE_API_URL` at your API domain).
- Lock down CORS in `backend/app.py` to your real frontend origin before deploying.
