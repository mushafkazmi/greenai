# Deploying Green AI for free

## Recommended: one Flask service, free tier

Flask now serves the built React app itself, so the whole site — frontend
and API — can live on a **single free web service** (Render, Railway, or
Fly.io all work the same way). This has been built and tested locally end
to end (SPA routes, static assets, and the JSON API all verified working
from one Flask process).

1. Push this repo to GitHub (these hosts deploy from a git repo).
2. On Render.com → New → Web Service → connect the repo, root directory
   `backend/`.
3. Build command:
   ```
   pip install -r requirements.txt && cd ../frontend && npm install && npm run build && rm -rf ../backend/static && cp -r dist ../backend/static
   ```
   (This runs `frontend/build_into_backend.sh`'s steps inline so Render's
   single build command does both halves. If your host lets you run a
   custom script instead, point it at `frontend/build_into_backend.sh`.)
4. Start command: `gunicorn app:app` (already in `backend/Procfile`, so
   Render should pick it up automatically if you leave this blank).
5. Environment variables, in the Render dashboard:
   - `SECRET_KEY` — any long random string
   - `JWT_SECRET_KEY` — a different long random string
   - `DATABASE_URL` — see the SQLite warning below
   - You do **not** need `FRONTEND_ORIGIN` or `VITE_API_URL` for this setup
     — same origin, no CORS to configure.
6. Deploy. You'll get one URL, e.g. `https://green-ai.onrender.com`, and
   that's the whole site.

**SQLite warning:** the app defaults to a local SQLite file. That's fine for
trying things out, but most free hosts (Render's free tier included) use an
*ephemeral* filesystem — the database resets whenever the service redeploys
or spins back up from sleep. For anything you want to keep:
- Render offers a free Postgres instance (90-day limit on the free tier,
  then it needs recreating or upgrading) — create one and paste its
  "External Database URL" into `DATABASE_URL`.
- Alternatives with a free Postgres tier: [Neon](https://neon.tech) or
  [Supabase](https://supabase.com) — copy their connection string the
  same way.
- The code already normalizes `postgres://` → `postgresql://`, so no
  further changes are needed once `DATABASE_URL` is set.

## Alternative: frontend and backend as two separate free services

Still supported if you'd rather have the frontend on a CDN (Netlify/Vercel)
and the API elsewhere (Render/Railway) — useful if the frontend will get far
more traffic than the API, or you want independent deploy cycles.

**Backend (Render):** same as above, but skip the frontend build step in
the build command (`pip install -r requirements.txt` only), and set
`FRONTEND_ORIGIN` to the frontend's URL (comma-separate multiple origins if
needed).

**Frontend (Netlify):** New site from Git → base directory `frontend/` →
build command `npm run build` → publish directory `dist` (`netlify.toml`
already sets these). Add env var `VITE_API_URL` = your backend's URL
**before** the first build — Vite bakes it in at build time, so changing it
later needs a redeploy. `vercel.json` is included if you use Vercel instead.

Go back to the backend's env vars afterward and set `FRONTEND_ORIGIN` to the
frontend's exact deployed URL, then redeploy the backend so CORS allows it.

## Things that are fine for a demo but worth knowing

- **Free-tier sleep:** Render's free web services spin down after inactivity
  and take ~30–60s to wake on the next request — the first load after a
  quiet period will feel slow. Not a code problem, just how free compute works.
- **JWT expiry:** access tokens default to 15 minutes
  (flask-jwt-extended's default). Fine for testing; for a real demo you may
  want to extend `JWT_ACCESS_TOKEN_EXPIRES` in `backend/config.py` or add a
  refresh flow.
- **Secrets:** the `.env.example` files are placeholders — never commit a
  real `.env` with production secrets to a public repo.
- **Rebuild on frontend changes:** with the single-service setup, any
  frontend edit needs `frontend/build_into_backend.sh` re-run (or your
  host's build command re-triggered) before it shows up — Flask is serving
  a static snapshot in `backend/static`, not live-compiling React.
