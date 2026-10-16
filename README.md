# LightsOut

F1 season hub and race forecasts. One repo, two independently deployed apps
that talk over HTTP:

| Folder | What | Deploys to |
| --- | --- | --- |
| [`api/`](api/) | FastAPI backend: LightGBM models, Monte Carlo race forecasts, calendar and standings | Render (Docker, `render.yaml`) |
| [`web/`](web/) | Next.js 16 frontend | Vercel (Root Directory `web`) |

The web app reads the API at `NEXT_PUBLIC_API_URL`
(`https://lightsout-api.onrender.com/api` by default), so neither app imports
the other's code. See each folder's README for setup.

## Local development

```bash
# API — http://localhost:8000
cd api
python -m venv .venv && .venv\Scripts\activate
pip install -r requirements.txt
uvicorn app.main:app --reload

# Web — http://localhost:3000 (point .env.local at the local API)
cd web
cp .env.local.example .env.local
npm install
npm run dev
```

## Automation

Both workflows live in `.github/workflows/` and commit straight to `main`:

- `retrain.yml` — Tuesdays 03:00 UTC: rebuild the dataset and retrain models
  in `api/`, commit artifacts. Render redeploys (its build filter only watches
  `api/**`).
- `refresh-base-data.yml` — Tuesdays 03:30 UTC: refresh the web app's offline
  calendar/standings snapshot. Vercel redeploys; `web/vercel.json` skips builds
  for commits that don't touch `web/`.
