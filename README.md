# TerraWatch ASM — BigQuery AI Geospatial Notebook (Kaggle Submission)

[![Kaggle](https://img.shields.io/badge/Platform-Kaggle-blue)](#) [![BigQuery](https://img.shields.io/badge/Google-BigQuery-informational)](#) [![GEE](https://img.shields.io/badge/Google%20Earth%20Engine-GEE-green)](#)


This repository contains the **kaggle_demo_BigQueryAI_FINAL.ipynb** notebook, an end‑to‑end geospatial analytics and AI workflow that:
- Builds an AOI‑specific satellite **time series** and persists analysis outputs in **BigQuery**.
- Uses **BigQuery ML / Gemini** for decision‑oriented **narratives** and long‑form **briefs** grounded in the data.
- Enables **semantic embeddings** and **vector search** over a knowledge corpus to add contextual explanations.
- (Optional) Integrates **MISLAND / SDG 15.3.1**‑aligned degradation analytics for standards‑compliant reporting.
- (Optional) Runs **AI.FORECAST / ML.FORECAST** to project near‑term indicator trajectories.

## Table of Contents
1. [Quick Start](#quick-start)
2. [Requirements](#requirements)
3. [Credentials & Environment](#credentials--environment)
4. [Parameters](#parameters)
5. [End‑to‑End Execution Flow](#end-to-end-execution-flow)
6. [Outputs](#outputs)
7. [Troubleshooting](#troubleshooting)
8. [Reproducibility Notes](#reproducibility-notes)
9. [FAQ](#faq)

## Quick Start

- To run/test the code, create a folder on you local machine, upon cretion of the folder, open command prompt in that folder and run this command to create a virtual environment

```python
# ========================================
# 1. ENVIRONMENT SETUP
# ========================================
python -m venv BigqueryAI

# ========================================
# 1. Requirements setup
# ========================================
pip install -r requirements.txt
```

1. **Open** `kaggle_demo_BigQueryAI_FINAL.ipynb` in Kaggle (or a GCP‑enabled Jupyter environment).
2. **Run the Setup cells** (auth, imports, config). Ensure BigQuery and, if used, Earth Engine are authenticated.
3. **Set parameters** for your AOI and time window (see [Parameters](#parameters)).
4. **Execute cells sequentially** (top → bottom). Each section reads from/writes to BigQuery tables as needed.
5. **Review outputs** (tables, narratives, vector search results, MISLAND stats) and export if required.

## Requirements

- Google Cloud project with **BigQuery** enabled (and billing).
- **Google Earth Engine (GEE)** access and initialization capability.
- BigQuery **ML** and **Model Garden** permissions (e.g., `ML.GENERATE_TEXT`, `ML.GENERATE_EMBEDDING`, VECTOR SEARCH).
- Access to **AI.FORECAST / ML.FORECAST** (preview/region‑limited features may apply).
- Access to MISLAND/SDG 15.3.1 data sources or pipeline code referenced by the notebook.

## Credentials & Environment

- **BigQuery:** The notebook uses `google.cloud.bigquery.Client`. Authenticate in Kaggle via *Add-ons → Google Cloud* or locally with `GOOGLE_APPLICATION_CREDENTIALS`.
- **GEE (if enabled):** The notebook calls `ee.Initialize()`; run the interactive auth flow once in your environment.
- **Kaggle runtime:** Ensure Internet is enabled (if needed for GCP auth) and that your account has access to the target GCP project.
- **Location / Region:** Some ML features require specific BigQuery regions—match your dataset region to model availability.

## Parameters

The notebook exposes the following parameters (heuristically detected from code cells):

| Parameter   | Value/Expression   |
|:------------|:-------------------|
| TABLE       | =========          |

> **Tip:** If a value is expressed via `os.environ.get(...)`, set it in your environment (Kaggle secrets or local shell) before running.

## End‑to‑End Execution Flow

1. **Initialization & Config**
   - Import libraries, validate environment, authenticate BigQuery (and Earth Engine if used).
   - Create / validate BigQuery **dataset** and working tables.

2. **AOI & Time‑Series Preparation**
   - Load AOI (geometry/GeoJSON) and select analysis window.
   - Build AOI‑specific **time series** of satellite indicators; write to BigQuery.

3. **Narratives & Briefs (BigQuery ML)**
   - Use `ML.GENERATE_TEXT` on AOI tables to produce short **narratives** and a longer **brief** grounded in the data.

4. **Embeddings & Vector Search (RAG)**
   - Create embeddings via `ML.GENERATE_EMBEDDING`, store in a vector‑enabled table/index.
   - Run **VECTOR_SEARCH** to answer free‑text questions with citations to your corpus.

5. **(Optional) MISLAND / SDG 15.3.1**
   - Trigger MISLAND pipeline if results are not present; otherwise read latest stats.
   - Summarize degradation status and transitions; persist to BigQuery and include in the brief.

6. **(Optional) Forecasting**
   - Use `AI.FORECAST / ML.FORECAST` to project near‑term values of selected indicators.
   - Store predictions (yhat, intervals) and visualize or include in the narrative.

7. **Export & Sharing**
   - Save tables, export CSVs or charts as needed.
   - Optionally publish dashboards connected to the BigQuery tables.

## Outputs

- **BigQuery tables** with AOI time‑series indicators and aggregates.
- **Narrative summaries** and **briefs** generated by BigQuery ML (text).
- **Embedding tables** and **vector indexes** for semantic retrieval.
- **(Optional) MISLAND / SDG 15.3.1** degradation metrics per AOI.
- **(Optional) Forecast tables** with predicted trajectories.

## Troubleshooting

- **Auth errors (BigQuery):** Re‑authenticate; in Kaggle use *Add-ons → Google Cloud*. Locally, set `GOOGLE_APPLICATION_CREDENTIALS` to a service account JSON with BigQuery access.
- **Region mismatch errors:** Ensure your BigQuery **dataset** location matches the supported regions for ML functions used.
- **Earth Engine initialization fails:** Run `ee.Authenticate()` (first time) and `ee.Initialize()`. Verify your account has GEE access.
- **Insufficient permissions:** Ask your admin for `BigQuery Data Editor` and `BigQuery Job User` on the dataset; add ML permissions if using Model Garden.
- **Vector search not available:** Enable the **Preview**/feature flags if required, or fall back to cosine similarity in SQL over embedding arrays.
- **Missing MISLAND outputs:** Run the MISLAND step, or point the notebook to the correct BigQuery dataset where MISLAND results are stored.

## Reproducibility Notes

- Set parameters explicitly for **AOI**, **dates**, and **dataset names**. Avoid interactive prompts when running in automation.
- Keep a **CHANGELOG** of schema and table names in case you re‑run with different datasets.
- Version your **prompt templates** used with `ML.GENERATE_TEXT` for consistent narratives across runs.

## FAQ

**Q: Can I run this without Earth Engine?**  
A: Yes, if your time‑series source is already in BigQuery. Earth Engine is required only when the notebook constructs imagery‑derived indicators directly.

**Q: Do I need special quotas?**  
A: Embeddings, text generation, and forecasting consume BigQuery ML quota. Review your project quotas and region availability before large runs.

**Q: How do I change the AOI?**  
A: Update the **AOI_TYPE/AOI_NAME** (or the geometry file path) in the parameter cell, then re‑run from Initialization.
