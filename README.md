# TerraWatch ASM — BigQuery AI Geospatial Notebook (Kaggle Submission)

[![Kaggle](https://img.shields.io/badge/Platform-Kaggle-blue)](#) [![BigQuery](https://img.shields.io/badge/Google-BigQuery-informational)](#) [![GEE](https://img.shields.io/badge/Google%20Earth%20Engine-GEE-green)](#)

This repository contains **`kaggle_demo_BigQueryAI_FINAL.ipynb`**, an end‑to‑end geospatial analytics and AI workflow that:
- Builds an AOI‑specific satellite **time series** and persists analysis outputs in **BigQuery**.
- Uses **BigQuery ML / Gemini** for decision‑oriented **narratives** and long‑form **briefs** grounded in the data.
- Enables **semantic embeddings** and **vector search** over a knowledge corpus to add contextual explanations.
- Integrates (optional) **MISLAND / SDG 15.3.1**‑aligned degradation analytics for standards‑compliant reporting.
- Runs (optional) **AI.FORECAST / ML.FORECAST** to project near‑term indicator trajectories.
- Supports basic **anomaly detection** for indicators.

> **Python version:** The notebook was prepared and validated with **Python 3.12.10**. Please use this version (or match it closely) for reproducibility.

---

## Table of Contents
1. [Quick Start](#quick-start)
2. [Environment Setup](#environment-setup)
3. [Credentials](#credentials)
4. [Notebook Parameters (with Justifications & Examples)](#notebook-parameters-with-justifications--examples)
   - [A. Core / Global](#a-core--global)
   - [B. Area of Interest (AOI)](#b-area-of-interest-aoi)
   - [C. Time Series](#c-time-series)
   - [D. SDG 15.3.1 (MISLAND)](#d-sdg-1531-misland)
   - [E. Forecasting](#e-forecasting)
   - [F. Anomaly Detection](#f-anomaly-detection)
   - [G. RAG: Embeddings & Vector Search](#g-rag-embeddings--vector-search)
   - [H. Narrative Generation](#h-narrative-generation)
5. [End‑to‑End Execution Flow](#end-to-end-execution-flow)
6. [Outputs](#outputs)
7. [Troubleshooting](#troubleshooting)
8. [Reproducibility Notes](#reproducibility-notes)
9. [FAQ](#faq)
---

## Quick Start

> **Kaggle**: Open `kaggle_demo_BigQueryAI_FINAL.ipynb` → ensure Google Cloud is connected (*Add‑ons → Google Cloud*).  
> **Local**: Use a virtual environment pinned to Python 3.12.10.

```bash
# 1) Create & activate a venv (Windows PowerShell)
python -m venv .venv
. .venv/Scripts/Activate.ps1

# macOS/Linux
python3 -m venv .venv
source .venv/bin/activate

# 2) Install dependencies
pip install -U pip wheel
pip install -r requirements.txt
```

1) **Open** the notebook.  
2) **Run** the Setup cells (auth, imports, config checks).  
3) **Set parameters** in the “Parameters” cell(s) as described below.  
4) **Run cells sequentially** top→bottom.  
5) **Review outputs** (tables, narratives, vector search, MISLAND stats, forecasts).

---
## Requirements

- Google Cloud project with **BigQuery** enabled (and billing) and the below APIs enabled.
1) **Vertex AI API**.  
2) **BigQuery API**.  
3) **Gemini For Google Cloud API**.  
4) **Cloud Dataplex API**.  
5) **Google Earth Engine API**.
6) **Generative Language API**.

- BigQuery **ML** and **Model Garden** permissions (e.g., `ML.GENERATE_TEXT`, `ML.GENERATE_EMBEDDING`, VECTOR SEARCH). The code will automaticaly create a text generation model and a text embeddings model for your project. But before that, ensure to run the command below in you google cloud shell or your 

- Access to **AI.FORECAST / ML.FORECAST** (preview/region‑limited features may apply). If AI.FORECAST is not available in you region. In the forecast section, the code will create a temporary ARIMA PLUS model and  use ML.FORECAST to generate the forecast.
---
## Environment Setup

- **Python:** 3.12.10 (recommended).  
- **OS packages:** Ensure `gcloud`.  
- **Pip packages:** Installed via `requirements.txt`.
- **Jupyter:** Use Classic, JupyterLab, or Kaggle notebooks.

### BigQuery
The notebook uses `google.cloud.bigquery.Client`:
- **Kaggle:** *Add‑ons → Google Cloud* → select project → enable BigQuery.  
- **Local:** set env var `GOOGLE_APPLICATION_CREDENTIALS` to a service account JSON with BigQuery access:
```bash
export GOOGLE_APPLICATION_CREDENTIALS="/path/to/sa.json"
```

> **Dataset Region:** Some ML features (e.g., `ML.GENERATE_TEXT`, `AI.FORECAST`) are region‑specific. Create your dataset in a supported region and keep all tables/models there.

---

## Credentials

| Service   | How to Auth | Required Roles/Perms (example) |
|-----------|-------------|----------------------------------|
| BigQuery  | Kaggle Add‑on or `GOOGLE_APPLICATION_CREDENTIALS` | BigQuery Data Editor, BigQuery Job User, BigQuery ML User |
| Earth Engine (optional) | `ee.Authenticate(); ee.Initialize()` | GEE enabled on your account |
| GCS (optional) | Inherits from GCP auth | Storage Object Admin (for exports) |

---

## Notebook Parameters (with Justifications & Examples)

> Parameters can be provided by directly editing the parameter cell(s) or via environment variables (if the notebook reads `os.environ[...]`). Below is a **canonical list** to guide judges and users. If any parameter is absent in your runtime, skip it or keep the default shown in the notebook.

### A. Core / Global
- `PROJECT_ID` *(str, required)* — GCP project that owns your BigQuery dataset.  
  **Justification:** All queries/jobs bill and execute in this project.  
  **Example:** `"terrawatch-kaggle-demo"`

- `BQ_DATASET` *(str, required)* — BigQuery dataset for all outputs.  
  **Justification:** Central namespace for tables (time series, embeddings, forecasts, MISLAND).  
  **Example:** `"eo"`

- `BQ_REGION` *(str, recommended)* — Region where the dataset lives.  
  **Justification:** ML functions and vector indexes are region‑bound.  
  **Example:** `"europe-west4"`

- `GCS_BUCKET` *(str, optional)* — GCS bucket for exports (charts, CSVs).  
  **Justification:** Portable delivery of artifacts.  
  **Example:** `"gs://terrawatch-exports"`

### B. Area of Interest (AOI)
Choose **one** of the following approaches.

1) **Use a GeoJSON file path**  
   - `AOI_GEOJSON_PATH` *(str, required if not using config option)* — absolute path to a valid GeoJSON.  
     **Justification:** Ensures reproducible geometry and name.  
     **Example:** `"/home/judge/data/voi.geojson"`  
     **Naming:** The GeoJSON should contain a `FeatureCollection` with a clear `name` (e.g., `"voi"`). If no name is present, the notebook will derive one from the filename.

2) **Use a config‑provided AOI (Will be derived from your Geojson)**  
   - `AOI_NAME` *(str)* — e.g., `"voi"`.  
   **Justification:** Drives table naming and queries (e.g., joins on AOI).  
   **Example:** `AOI_TYPE="polygon"`, `AOI_NAME="voi"`.

### C. Time Series
- `TIMESERIES_START_DATE` *(YYYY-MM-DD, required)* — start of analysis window.  
  **Justification:** Controls the historical depth of indicators and downstream analytics.  
  **Example:** `"2018-01-01"`

- `TIMESERIES_END_DATE` *(YYYY-MM-DD, required)* — end of analysis window (≤ today).  
  **Example:** `"2025-09-01"`

- `INDICATORS` *(list[str], optional)* — which indicators to compute or read.  
  **Typical:** `["NDVI", "BSI", "LST_MIN", "LST_MAX"]`  
  **Justification:** Limits cost and focuses on relevant variables.

- `TEMPORAL_STEP_DAYS` *(int, optional)* — resampling step for the time series.  
  **Example:** `5` (i.e., 5‑day intervals)

### D. SDG 15.3.1 (MISLAND)
- `BASELINE_YEAR` *(int, required)* — e.g., 2015.  
  **Justification:** UNCCD baseline for SDG 15.3.1 comparisons.

- `TARGET_YEAR` *(int, required)* — after `BASELINE_YEAR`, ≤ current year.  
  **Example:** `2023`

- `ANALYSIS_MONTH` *(int, optional)* — focus month if the method requires seasonal focus.  
  **Example:** `8` (August)

- **Behavior:** The notebook checks if outputs exist for the AOI/years; if not, it triggers the computation pipeline saves results to bigquery and then reads the results for further insights.

- **Expected outputs:**  
  - `{BQ_DATASET}` (land cover productivity, land cover, soil organic carbon proxies)  

### E. Forecasting
- `FORECAST_INDICATOR` *(str, required)* — column name in `{BQ_DATASET}.time_series_analysis` to forecast.  
  **Example:** `"NDVI_mean"`

- `FORECAST_STEPS` *(int, default: `6`)* — number of 5‑day periods.  
  **Justification:** The notebook function inlines **`horizon_days = 5 * steps`** and **`steps_lit = steps`** for `AI.FORECAST`, projecting ≈30 days ahead when `steps=6`.

- **Outputs:**  
  - `{BQ_DATASET}.forecast_{FORECAST_INDICATOR}` with `date`, `yhat`, `yhat_lower`, `yhat_upper`.

> **Note:** If `AI.FORECAST` is unavailable in your region, the notebook falls back to `ML.FORECAST (ARIMA_PLUS)` with the same horizon logic.

### F. Anomaly Detection
- `ANOMALY_INDICATOR` *(str, required)* — column in the time series to analyze.  
  **Example:** `"NDVI_mean"`

- `ANOMALY_METHOD` *(str, optional)* — e.g., `"zscore"` or `"iqr"`.  
  **Justification:** Simple, interpretable anomaly flags for situational awareness.

- `ANOMALY_WINDOW` *(int, optional)* — window for rolling statistics (e.g., `30` days).  
- `ANOMALY_Z_THRESHOLD` *(float, optional)* — e.g., `2.5` for z‑score.  
- **Output:** `{BQ_DATASET}.anomalies_{ANOMALY_INDICATOR}` with timestamps and flags/scores.

### G. RAG: Embeddings & Vector Search
- `EMBEDDING_MODEL` *(str, required)* — BigQuery ML embedding model.  
  **Example:** `"text-embedding-004"`

- `EMBED_TABLE` *(str, required)* — destination table for embeddings.  
  **Example:** `"{BQ_DATASET}.documents_embeddings"`

- `VECTOR_INDEX` *(str, optional)* — vector index name for faster ANN queries.  
- **Output:** Embedding vectors and a vector‑enabled table/index used by `VECTOR_SEARCH` queries.

### H. Narrative Generation
- `GEN_TEXT_MODEL` *(str, required)* — e.g., `"gemini-2.0-flash-lite-001"`.  
  **Use:** `ML.GENERATE_TEXT` produces: short AOI **narratives** and a longer **brief** with data references.  

---

## End‑to‑End Execution Flow

1. **Initialization & Config**  
   Import libs → validate environment → connect BigQuery (and Earth Engine if used) → create/confirm `{BQ_DATASET}`.

2. **AOI & Time‑Series Preparation**  
   Load AOI from **GeoJSON path** or **config AOI** → build AOI‐specific **time series** (NDVI/BSI/LST…) → write to BigQuery.

3. **Narratives & Briefs (BigQuery ML)**  
   Use `ML.GENERATE_TEXT` on the AOI time series to produce a short **narrative** and a longer **brief** grounded in the data.

4. **Embeddings & Vector Search (RAG)**  
   Create embeddings via `ML.GENERATE_EMBEDDING` → store in vector table/index → run `VECTOR_SEARCH` for contextual Q&A with citations.

5. **(Optional) UNCCD / SDG 15.3.1**  
   Check for AOI/year outputs; if missing, trigger MISLAND pipeline → read latest stats → persist to BigQuery.

6. **(Optional) Forecasting**  
   Use `AI.FORECAST` (or fallback `ML.FORECAST`) with `horizon_days = 5 * steps` → write predictions to BigQuery → visualize/include in narrative.

7. **(Optional) Anomaly Detection**  
   Compute simple anomaly flags/scores on a chosen indicator → store results → visualize.

8. **Export & Sharing**  
   Save tables; optionally export CSVs/figures to GCS; publish dashboard panels powered by BigQuery tables.

---

## Outputs

- **BigQuery tables** for AOI **time series**, **aggregates**, **forecasts**, **anomalies**, **embeddings**, **narratives/briefs**.
- Optional **GCS exports** (charts/CSVs).
- Notebook cells provide sample SQL for inspection in the BigQuery UI.

---

## Troubleshooting

- **Auth errors (BigQuery):** Re‑authenticate; in Kaggle use *Add‑ons → Google Cloud*. Locally, set `GOOGLE_APPLICATION_CREDENTIALS` to a service account JSON with BigQuery access.
- **Region mismatch:** Ensure `{BQ_DATASET}` region supports the ML features you run; keep all assets co‑located.
- **Earth Engine init fails:** Run `ee.Authenticate()` (first time) → `ee.Initialize()`; ensure your account has GEE access.
- **Vector search not available:** Use a standard SQL cosine similarity over embedding arrays as a fallback.
- **Missing MISLAND outputs:** Ensure the AOI/years are available or re‑run the MISLAND step for the new AOI.
- **No forecast output:** Check that `{BQ_DATASET}.time_series_analysis` has sufficient historical rows and that `FORECAST_INDICATOR` exists.

---

## Reproducibility Notes

- Pin **Python 3.12.10** and `requirements.txt` versions.  
- Set parameters explicitly (avoid interactive prompts) for automation.  
- Keep a small **CHANGELOG** of table and schema changes.  
- Version any **prompt templates** used with `ML.GENERATE_TEXT`.

---

## FAQ

**Q: Can I run this without Earth Engine?**  
A: Yes, if your time series already exist in BigQuery. Earth Engine is only required when deriving indicators from imagery on the fly.

**Q: How do I change the AOI?**  
A: Re‑run from Initialization and input the path to you new Geojson file. 

**Q: What is the default forecast horizon?**  
A: `horizon_days = 5 * FORECAST_STEPS` (e.g., `steps=6` → ~30 days).

**Q: Where do the narratives come from?**  
A: From `ML.GENERATE_TEXT` using your AOI tables as grounding inputs; the outputs are stored in BigQuery for auditing.
