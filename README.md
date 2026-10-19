# Heart Disease Risk Check: Railway Deployment

**Student:** Jayaprakash · **BITS ID:** 2025AE05611 · BITS Pilani, MLOps (AIMLCZG523), Assignment 01

This is the public cloud deployment of the heart disease classifier from the main assignment repository ([heart-disease-mlops](https://github.com/jpailabs/heart-disease-mlops)). It is the same FastAPI service, model and preprocessing pipeline, packaged for [Railway](https://railway.com). It adds a **web UI** where you can enter a new patient's details and get a risk prediction straight away.

| | |
|---|---|
| Live app | `https://<your-app>.up.railway.app` (add after the first deploy) |
| Model | Logistic Regression (full sklearn pipeline): CV ROC-AUC 0.908, test ROC-AUC 0.959 |
| Stack | FastAPI · scikit-learn · Docker · Railway · GitHub Actions |

## What's included

```
├── api/
│   ├── app.py            FastAPI service: UI, /predict, /predict/batch, /health, /ready, /model-info, /metrics
│   └── static/index.html web UI (single page, no external dependencies)
├── src/heart/            data cleaning, feature pipeline, training and inference code
├── models/               heart_model.joblib + model_metadata.json (trained model)
├── data/processed/       cleaned UCI Cleveland dataset (for retraining)
├── tests/                pytest unit tests (data, features, model, API and UI)
├── Dockerfile            listens on $PORT (Railway) and falls back to 8000
├── railway.json          Railway build/deploy config (Dockerfile builder, /ready healthcheck)
└── .github/workflows/ci.yml   lint + tests + Docker smoke test with a Railway-style PORT
```

## Web UI

Open the app root (`/`). Enter the patient's details:

- demographics
- symptoms and vitals
- ECG and exercise test results
- imaging results, which are optional: choose **Unknown** if they aren't available, and the model fills them in

Then press **Predict risk**. The page shows:

* the estimated probability of disease, on a low / moderate / high risk meter
* the predicted class and the model's confidence
* a table of the predictions made in this session

The **Fill high-risk example** and **Fill low-risk example** buttons load sample patients. The browser checks your input first, and the API checks it again, so out-of-range values are flagged.

## Deploy on Railway

**Option A: from GitHub (recommended).** Railway then redeploys automatically on every push to `main`.

1. Open [railway.com/new](https://railway.com/new) and choose **Deploy from GitHub repo**.
2. Select this repository. Railway reads `railway.json` and builds the `Dockerfile`.
3. In the service, open **Settings → Networking → Generate Domain**.
4. Open the generated URL. You should see the web UI. The API docs are at `/docs`.

**Option B: from the Railway CLI**

```bash
brew install railway        # or: npm i -g @railway/cli
railway login
railway init                # create a new project
railway up                  # build and deploy from this folder
railway domain              # create the public URL
```

There are no required environment variables. These are optional:

| Variable | Default | Purpose |
|---|---|---|
| `DECISION_THRESHOLD` | `0.5` | probability cut-off for the disease class |
| `LOG_LEVEL` | `INFO` | log level for the JSON request logs (shown in Railway's log viewer) |

## Run locally

```bash
python3.11 -m venv .venv && source .venv/bin/activate
pip install -r requirements.txt && pip install -e .
pytest && flake8 src api tests scripts
uvicorn api.app:app --port 8000      # then open http://localhost:8000
```

With Docker:

```bash
docker build -t heart-railway .
docker run -e PORT=9000 -p 9000:9000 heart-railway     # http://localhost:9000
```

## API

| Method | Path | Description |
|---|---|---|
| GET | `/` | web UI |
| POST | `/predict` | one patient (JSON) → `prediction`, `probability_disease`, `confidence`, `risk_band` |
| POST | `/predict/batch` | `{"patients": [...]}` (up to 500 records) |
| GET | `/health`, `/ready` | liveness / readiness checks |
| GET | `/model-info` | model parameters and metrics |
| GET | `/metrics` | Prometheus metrics |
| GET | `/docs` | Swagger UI |

```bash
curl -X POST https://<your-app>.up.railway.app/predict \
     -H 'content-type: application/json' -d @sample_request.json
```

> For educational use only. This tool does not provide a medical diagnosis.
