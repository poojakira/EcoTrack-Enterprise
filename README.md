🌍 EcoTrack-Enterprise



**High-Fidelity Carbon Lifecycle Analytics & Security Shield**

EcoTrack-Enterprise is an industrial-grade ML microservice architecture designed to quantify product carbon footprints. It leverages FastAPI’s asynchronous execution, Random Forest regression for predictive precision, and an Isolation Forest layer for real-time anomaly detection.

**Data Source: Extracted from Kaggle: Product Lifecycle Carbon Footprint and processed as dpp_data.csv.**


## 🚀 Validated Performance

Predictive Precision: Training benchmarks consistently yield an $R^2$ score of 0.9952, explaining 99.5% of carbon variance.

Operational Scalability: Successfully validated with 1,000 concurrent requests achieving a P90 latency of 281ms.

Zero-Trust Security: Real-time anomaly detection flags out-of-distribution inputs in <5ms.

<img width="501" height="137" alt="image" src="https://github.com/user-attachments/assets/98b3ab6b-d121-4f5b-9c6e-87de8961d8e6" />



## 🔧 Project Layout

**Extract the zip file**

The repository has a multi‑service structure with separate backend and
frontend components, plus shared data at the top level:

```
Docker-compose.yml          # orchestrates backend + frontend containers

backend/                    # FastAPI server, ML training and APIs
  ├── Dockerfile            # container build instructions
  ├── requirements.txt      # backend Python dependencies
  ├── app/                  # application source code
  │   ├── config.py         # configuration settings and paths
  │   ├── dependencies.py   # DI helpers (if any)
  │   ├── logging_config.py # logging setup
  │   ├── main.py           # FastAPI entrypoint & endpoints
  │   ├── schemas.py        # Pydantic models for requests/responses
  │   ├── api/              # route definitions (routes_model.py)
  │   └── ml/               # machine‑learning logic
  │       ├── anomaly.py
  │       ├── predict.py
  │       ├── preprocessing.py
  │       ├── train.py
  │       └── artifacts/    # optional output directory
  ├── data/                 # dataset CSV + serialized models (.pkl)
  └── stress_test.py        # concurrency / load‑testing client

frontend/                   # simple dashboard application
  ├── dashboard.py
  ├── Dockerfile            # container build
  ├── requirements.txt
  └── data/                 # frontend-specific data if any

data/                       # top-level dataset copy (dpp_data.csv)
```

The backend `app/config.py` automatically points to either `/app/data`
inside Docker or `backend/data` during local development, so the same
artifacts can be shared.

---

## ✅ Quick Start (full command list)

Follow these commands in sequence to get the system up and running.

```bash
# 1. prepare environment (run once)
cd backend                           # always operate from backend/
python -m venv venv                 # create virtualenv
source venv/bin/activate            # .\venv\Scripts\activate on Windows
pip install -r requirements.txt     # install Python dependencies

# 2. train models (writes model.pkl and security_model.pkl)
python app/ml/train.py

# 3. start FastAPI server
uvicorn app.main:app --host 0.0.0.0 --port 8000
# leave this process running in its own terminal

# 4. check health (new terminal)
curl http://127.0.0.1:8000/health

# 5. issue a test prediction (bash example)
curl -X POST http://127.0.0.1:8000/predict \
  -H "Content-Type: application/json" \
  -d '{"raw_material_energy": 123.4, ...}'

# 5b. PowerShell variant
curl.exe -X POST http://127.0.0.1:8000/predict `
  -H "Content-Type: application/json" `
  -d '{"raw_material_energy":123.4, ...}'
# or use Invoke-RestMethod with full JSON body per README

# 6. run concurrency/stress test (optional)
python backend/stress_test.py --concurrency 20 --requests 1000

# 7. start frontend dashboard (optional)
#   - open a new terminal and switch to the workspace root
cd frontend
# install frontend dependencies (after first checkout)
pip install -r requirements.txt

# launch using Streamlit (recommended):
streamlit run dashboard.py

# the app will start a local server (usually http://127.0.0.1:8050).
# you can also execute the script directly with `python dashboard.py`.
# in that case the script detects the missing Streamlit runtime, prints a
# short reminder, and exits cleanly – no UI will appear.  The CLI is still
# preferred because it initializes the browser URL and session state.

```

> **Note:** the `...` above is shorthand; real payloads must include all 18
> fields enclosed in double quotes (see earlier examples below the command
> in the README for a complete JSON object).

---

### 📋 Commands Cheat Sheet

| Purpose                    | Command                                         |
|---------------------------|-------------------------------------------------|
| create virtualenv         | `python -m venv venv`                           |
| activate venv             | `source venv/bin/activate` (Windows: `.\venv\Scripts\activate`)
| install deps              | `pip install -r requirements.txt`              |
| train models              | `python app/ml/train.py`                       |
| start API                 | `uvicorn app.main:app --port 8000`            |
| health check              | `curl http://127.0.0.1:8000/health`           |
| prediction (bash)         | see command in step 5 above                    |
| prediction (PowerShell)   | `Invoke-RestMethod -Method POST -Uri ...`      |
| stress test               | `python backend/stress_test.py --concurrency 20 --requests <N>` |
| start dashboard           | `cd frontend && python dashboard.py`          |
| install frontend deps     | `cd frontend && pip install -r requirements.txt` |

---

## 🔁 Lifecycle & Architecture

1. **Training** – `train.py` loads `dpp_data.csv`, selects 18 features,
   fits a `RandomForestRegressor` and an `IsolationForest`, computes an
   R² score, and saves both models.
2. **Startup** – `main.py` loads the serialized models into a global dict.
3. **Prediction** – client POSTs JSON to `/predict`; input is validated,
   run through the anomaly detector, and scored by the regressor.  Response
   includes the predicted footprint and `anomaly_detected` boolean.
4. **Benchmarking** – `stress_test.py` picks random dataset rows and hits
   `/predict` concurrently, reporting latency and error rates.

All requests are stateless, making horizontal scaling trivial via multiple
Uvicorn workers or containers.

---

## 🧪 Performance & Security Notes

* Training output generally yields R² ≈ 0.99 on this dataset.
* Anomaly detector flags irregular inputs and logs them at warning level.
* Configuration auto‑switches between local and Docker paths.
* Rate limiting is **not** built‑in; add middleware if deploying publicly.

---

![dashboard_main](https://github.com/user-attachments/assets/18b5aeb5-1695-41cd-b8e9-504b998397c1)

