# EcoTrack Enterprise

A full-stack ML application that predicts product lifecycle carbon footprints using a FastAPI backend and a Streamlit dashboard frontend, all orchestrated with Docker Compose.

---

## Environment Setup

All files for this project live under a root folder called `eco-track-enterprise`. Start by creating that folder and placing all files inside it exactly as shown in the structure below.

### Step 1 — Create the project folder

```bash
mkdir eco-track-enterprise
cd eco-track-enterprise
```

### Step 2 — Recreate the folder structure

```bash
# Create all required directories
mkdir -p backend/app/api
mkdir -p backend/app/ml/artifacts
mkdir -p backend/data
mkdir -p frontend/data
```

### Step 3 — Place your files

Copy or move all project files into their correct locations:

```
eco-track-enterprise/
├── docker-compose.yml
├── backend/
│   ├── Dockerfile
│   ├── requirements.txt
│   ├── data/
│   │   ├── dpp_data.csv
│   │   ├── model.pkl
│   │   └── security_model.pkl
│   └── app/
│       ├── main.py
│       ├── config.py
│       ├── schemas.py
│       ├── dependencies.py
│       ├── logging_config.py
│       ├── api/
│       │   └── routes_model.py
│       └── ml/
│           ├── train.py
│           ├── predict.py
│           ├── preprocessing.py
│           ├── anomaly.py
│           └── artifacts/
└── frontend/
    ├── Dockerfile
    ├── requirements.txt
    ├── dashboard.py
    └── data/
```

### Step 4 — Verify your structure

```bash
# From inside eco-track-enterprise/
find . -type f | sort
```

You should see all files listed above before proceeding.

---

## Services

| Service    | Container           | Port   | Description                          |
|------------|---------------------|--------|--------------------------------------|
| `backend`  | `eco_backend_v4`    | `8000` | FastAPI REST API + ML inference      |
| `frontend` | `eco_frontend_v4`   | `8501` | Streamlit dashboard                  |

- The **frontend** depends on the **backend** and communicates via `http://backend:8000`.
- Both services share the `./backend/data` directory (CSV + model files).
- Both services use `restart: always` and have hot-reload volume mounts.

---

## Prerequisites

- [Docker](https://docs.docker.com/get-docker/) and [Docker Compose](https://docs.docker.com/compose/install/) installed.

---

## Run Commands

### 1. Build and Start All Services

```bash
docker-compose up --build
```

> Use `-d` to run in detached (background) mode:
```bash
docker-compose up --build -d
```

### 2. Stop All Services

```bash
docker-compose down
```

### 3. Train the ML Models (inside the backend container)

The models (`model.pkl` and `security_model.pkl`) are pre-built and included in `backend/data/`. If you need to retrain them from scratch:

```bash
docker-compose exec backend python app/ml/train.py
```

This will:
- Load `dpp_data.csv` from `/app/data/`
- Train a **Random Forest Regressor** (carbon footprint predictor)
- Train an **Isolation Forest** (anomaly/security detector)
- Save both models back to `/app/data/`

### 4. View Logs

```bash
# All services
docker-compose logs -f

# Backend only
docker-compose logs -f backend

# Frontend only
docker-compose logs -f frontend
```

### 5. Rebuild a Single Service

```bash
docker-compose up --build backend
docker-compose up --build frontend
```

---

## API Endpoints

Base URL: `http://localhost:8000`

| Method | Endpoint   | Description                                      |
|--------|------------|--------------------------------------------------|
| GET    | `/health`  | Returns API status and which models are loaded   |
| POST   | `/predict` | Predicts carbon footprint for a given input      |
| POST   | `/explain` | Returns SHAP values for a prediction             |
| POST   | `/drift`   | Detects data drift for a given input             |

### Health Check

```bash
curl http://localhost:8000/health
```

Response:
```json
{
  "status": "online",
  "models_loaded": ["regressor", "security"]
}
```

### Predict Carbon Footprint

```bash
curl -X POST http://localhost:8000/predict \
  -H "Content-Type: application/json" \
  -d '{
    "raw_material_energy": 120.5,
    "raw_material_emission_factor": 0.85,
    "raw_material_waste": 15.2,
    "manufacturing_energy": 200.0,
    "manufacturing_efficiency": 0.78,
    "manufacturing_water_usage": 500.0,
    "transport_distance_km": 1200.0,
    "transport_mode_factor": 0.12,
    "logistics_energy": 80.0,
    "usage_energy_consumption": 300.0,
    "usage_duration_hours": 8760.0,
    "grid_carbon_intensity": 0.45,
    "recycling_efficiency": 0.65,
    "disposal_emission_factor": 0.30,
    "recovered_material_value": 50.0,
    "state_complexity_index": 3.2,
    "policy_action_score": 7.5,
    "optimization_reward_signal": 0.9
  }'
```

Response:
```json
{
  "predicted_carbon_footprint": 423.17,
  "anomaly_detected": false,
  "model_version": "v4.0_Enterprise"
}
```

> **Security:** The `/predict` endpoint automatically runs anomaly detection on every input. If the input resembles an attack vector, `anomaly_detected` will be `true` and a warning is logged.

> **Rate Limiting:** The `/predict`, `/explain`, and `/drift` endpoints are limited to **20 requests/minute** per IP.

---

## Frontend Dashboard

Access the Streamlit dashboard at: **http://localhost:8501**

The dashboard connects to the backend at `http://backend:8000` (via Docker internal network).

---

## ML Models

| Model | Algorithm | Purpose |
|-------|-----------|---------|
| `model.pkl` | Random Forest Regressor | Predicts `total_lifecycle_carbon_footprint` |
| `security_model.pkl` | Isolation Forest | Detects anomalous/adversarial inputs (5% contamination threshold) |

### Input Features (18 total)

All features are `float` values:

| Feature | Description |
|---------|-------------|
| `raw_material_energy` | Energy used in raw material extraction |
| `raw_material_emission_factor` | Emission factor for raw materials |
| `raw_material_waste` | Waste generated during raw material stage |
| `manufacturing_energy` | Energy consumed during manufacturing |
| `manufacturing_efficiency` | Efficiency ratio of the manufacturing process |
| `manufacturing_water_usage` | Water used during manufacturing |
| `transport_distance_km` | Distance goods are transported (km) |
| `transport_mode_factor` | Emission factor for transport mode |
| `logistics_energy` | Energy used in logistics |
| `usage_energy_consumption` | Energy consumed during product use phase |
| `usage_duration_hours` | Total hours the product is in use |
| `grid_carbon_intensity` | Carbon intensity of the electricity grid |
| `recycling_efficiency` | Efficiency of end-of-life recycling |
| `disposal_emission_factor` | Emissions from disposal |
| `recovered_material_value` | Value of recovered materials |
| `state_complexity_index` | Complexity index for current product state |
| `policy_action_score` | Score reflecting policy/regulatory actions |
| `optimization_reward_signal` | RL-style optimization signal |

---

## Environment Variables

### Backend

| Variable | Default | Description |
|----------|---------|-------------|
| `DATA_PATH` | `/app/data/data_dpp.csv` | Path to training CSV |
| `MODEL_PATH` | `/app/data/model.pkl` | Path to regressor model |

### Frontend

| Variable | Default | Description |
|----------|---------|-------------|
| `BACKEND_URL` | `http://backend:8000` | URL of the backend API |

---

## Dependencies

**Backend** (`backend/requirements.txt`):
```
fastapi==0.109.0
uvicorn==0.27.0
pandas==2.2.0
scikit-learn==1.4.0
joblib==1.3.2
pydantic==2.6.0
python-multipart
```

**Frontend** (`frontend/requirements.txt`):
```
streamlit
plotly
pandas
scikit-learn==1.4.0
statsmodels
```

---

## Enterprise Benefits

**Scalability** — The microservices architecture (backend + frontend as separate containers) allows each service to be scaled independently based on load. As prediction demand grows, you can scale the backend without touching the dashboard, and vice versa.

**Security & Anomaly Detection** — Every prediction request is automatically screened by an Isolation Forest security model trained to detect adversarial or out-of-distribution inputs. Suspicious requests are flagged in real time with a RED security flag and logged for audit, giving enterprise teams visibility into potential data poisoning or API abuse attempts.

**Reproducibility** — All dependencies are pinned to exact versions in `requirements.txt` and containerized via Docker, ensuring the application runs identically across development, staging, and production environments with no "works on my machine" issues.

**Data Governance & Traceability** — The API returns a `model_version` field with every prediction, making it straightforward to trace which model version produced a given output — a key requirement for regulatory compliance and audit trails in enterprise settings.

**Rate Limiting** — Built-in rate limiting (20 requests/minute per IP) on all ML endpoints protects backend resources from abuse and ensures fair usage across multiple consumers of the API.

**Hot Reload & Developer Velocity** — Volume mounts map local source code directly into running containers, meaning code changes reflect instantly without rebuilding images. This dramatically reduces iteration time for data scientists and engineers working on the models or dashboard.

**Operational Resilience** — Both services are configured with `restart: always`, ensuring automatic recovery from crashes or host reboots without manual intervention — critical for production deployments.

**Extensibility** — The modular ML layer (`train.py`, `predict.py`, `preprocessing.py`, `anomaly.py`) is cleanly separated from the API layer, making it straightforward to swap in new models, add explainability endpoints (SHAP via `/explain`), or integrate drift monitoring (via `/drift`) as the project matures.
