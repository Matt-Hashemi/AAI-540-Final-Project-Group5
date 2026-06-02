<div align="center">

# 💧 AAI-540 Final Project — Group 5

## Water Quality Risk Classification by ZIP Code Using Machine Learning

[![Course](https://img.shields.io/badge/Course-AAI--540-blue?style=flat-square)](https://www.sandiego.edu)
[![School](https://img.shields.io/badge/Shiley--Marcos%20School%20of%20Engineering-USD-navy?style=flat-square)](https://www.sandiego.edu/engineering)
[![Program](https://img.shields.io/badge/MAAI-Master%20of%20Applied%20AI-darkblue?style=flat-square)](https://www.sandiego.edu)
[![Python](https://img.shields.io/badge/Python-3.10%2B-3776AB?style=flat-square&logo=python&logoColor=white)](https://www.python.org)
[![AWS](https://img.shields.io/badge/AWS-SageMaker%20Studio-FF9900?style=flat-square&logo=amazonaws&logoColor=white)](https://aws.amazon.com/sagemaker)
[![Data](https://img.shields.io/badge/Data-EPA%20SDWA-green?style=flat-square)](https://echo.epa.gov/tools/data-downloads/sdwa-download-summary)

**Classifying U.S. ZIP codes as High Water Quality Risk or Normal/Lower Risk using EPA compliance data and machine learning.**

*Shiley-Marcos School of Engineering · University of San Diego · Master of Applied Artificial Intelligence · Summer 2026*

</div>

---

## 👥 Team Members

| Name | GitHub |
|------|--------|
| Puja Nandini | — |
| Gregory Bauer | — |
| Matt Hashemi | — |

---

## 📋 Table of Contents

| # | Section |
|---|---------|
| 1 | [Project Overview](#1--project-overview) |
| 2 | [Objectives](#2--objectives) |
| 3 | [Dataset](#3--dataset) |
| 4 | [Repository Structure](#4--repository-structure) |
| 5 | [S3 Bucket Structure](#5--s3-bucket-structure) |
| 6 | [SDWA Source Data Schema](#6--sdwa-source-data-schema) |
| 7 | [Setup: Connect to Shared S3 from SageMaker Studio](#7--setup-connect-to-shared-s3-from-sagemaker-studio) |
| 8 | [Reproducibility Workflow](#8--reproducibility-workflow) |
| 9 | [Team Conventions](#9--team-conventions) |
| 10 | [References](#10--references) |

---

## 1 · Project Overview

This project uses ZIP-code-level water quality, violation, enforcement, and environmental data from the **EPA Safe Drinking Water Act (SDWA)** national compliance database to classify U.S. ZIP codes as either **High Water Quality Risk** or **Normal/Lower Risk**.

The goal is to build an interpretable machine learning model that helps identify areas that may need further public health or infrastructure review.

> [!NOTE]
> This model is intended as a **decision-support tool** and does not replace official EPA, state, or local water authority reporting.

---

## 2 · Objectives

- 🧹 **Prepare** — Clean and prepare the water quality dataset at ZIP-code granularity
- ⚙️ **Engineer** — Build risk-related features such as violation count, lead risk score, and unresolved enforcement ratios
- 🤖 **Model** — Train and compare classifiers: Random Forest, XGBoost, and LightGBM
- 📊 **Evaluate** — Assess performance using precision, recall, F1-score, and confusion matrix
- 🔍 **Explain** — Identify the most important factors driving high-risk ZIP code predictions

---

## 3 · Dataset

The project uses the **US Water Quality Data by ZIP Code** dataset, which includes water quality indicators for more than **41,000 U.S. ZIP codes** aggregated from EPA SDWA national records.

### Key Features

| Feature | Description |
|---------|-------------|
| `violation_count` | Total SDWA violations recorded for the ZIP code |
| `contaminant_count` | Number of distinct contaminants detected |
| `lead_result` | Lead 90th-percentile sample measure (LCR) |
| `copper_result` | Copper 90th-percentile sample measure (LCR) |
| `enforcement_actions` | Count of enforcement actions issued |
| `boil_water_advisory` | Whether a boil water advisory has been issued |
| `population_served` | Estimated population served by the water system |
| `home_safety_score` | Composite home safety score |
| `unresolved_violations` | Count of violations with no return-to-compliance date |
| `health_based_violations` | Count of health-based violations (`IS_HEALTH_BASED_IND = Y`) |

### Target Variable

| Label | Meaning |
|-------|---------|
| `1` — **High Risk** | ZIP code exhibits elevated water quality risk indicators |
| `0` — **Normal/Lower Risk** | ZIP code within acceptable compliance bounds |

---

## 4 · Repository Structure

```text
aai540-group5/
│
├── README.md                     # This file
├── .gitignore
├── requirements.txt              # Python dependencies
│
├── data/
│   ├── raw/                      # Local copies of raw CSVs (gitignored)
│   └── processed/                # Cleaned/feature-engineered data (gitignored)
│
├── notebooks/
│   ├── 00_data_ingestion.ipynb   # Load data from S3, initial inspection
│   ├── 01_eda.ipynb              # Exploratory data analysis
│   ├── 02_preprocessing.ipynb    # Cleaning, encoding, feature engineering
│   ├── 03_modeling.ipynb         # Model training and evaluation
│   └── 04_explainability.ipynb   # SHAP / feature importance analysis
│
├── src/
│   ├── __init__.py
│   ├── data_loader.py            # S3 data loading utilities
│   ├── preprocessing.py          # Feature engineering pipeline
│   ├── model.py                  # Model definitions and training loops
│   └── evaluate.py               # Metrics, confusion matrix, ROC
│
├── models/
│   └── .gitkeep                  # Saved model artifacts (gitignored except placeholder)
│
└── reports/
    ├── figures/                  # Charts and visualizations
    └── final_report.pdf          # Final deliverable
```

---

## 5 · S3 Bucket Structure

> [!IMPORTANT]
> All raw SDWA data files are stored in the shared team S3 bucket. **Do not re-upload raw data** — always read directly from S3.

```text
s3://aai540-group5-public-866792937762-us-east-1-an/
│
├── raw/
│   ├── SDWA_SITES.csv
│   ├── SDWA_PUB_WATER_SYSTEMS.csv
│   ├── SDWA_VIOLATIONS_ENFORCEMENT.csv
│   ├── SDWA_FACILITIES.csv
│   ├── SDWA_SERVICE_AREAS.csv
│   ├── SDWA_LCR_SAMPLES.csv
│   ├── SDWA_GEOGRAPHIC_AREAS.csv
│   └── SDWA_PN_VIOLATION_ASSOC.csv
│
├── processed/
│   ├── zip_features.parquet      # ZIP-code aggregated feature table
│   ├── train.parquet
│   ├── val.parquet
│   └── test.parquet
│
├── models/
│   ├── random_forest_v1.pkl
│   ├── xgboost_v1.pkl
│   └── lightgbm_v1.pkl
│
└── reports/
    └── figures/
```

### Bucket Details

| Property | Value |
|----------|-------|
| **Bucket Name** | `aai540-group5-public-866792937762-us-east-1-an` |
| **ARN** | `arn:aws:s3:::aai540-group5-public-866792937762-us-east-1-an` |
| **Region** | US East (N. Virginia) — `us-east-1` |
| **Created** | May 24, 2026, 11:15:06 UTC−06:00 |
| **Access** | AWS Academy lab credentials (see [Section 7](#7--setup-connect-to-shared-s3-from-sagemaker-studio)) |

---

## 6 · SDWA Source Data Schema

All datasets originate from the [EPA SDWA national download](https://echo.epa.gov/tools/data-downloads/sdwa-download-summary). Column descriptions follow the official EPA data dictionary. Click any table name to expand its schema.

---

<details>
<summary><strong>6.1 &nbsp;SDWA_SITES</strong> — Geographic and ownership metadata for each regulated site</summary>

<br>

| Column | Type | Description |
|--------|------|-------------|
| `PWSID` | `str` | **Primary key** — Public Water System ID (e.g., `NM3510123`) |
| `SITE_ID` | `str` | Unique site identifier within the PWS |
| `SITE_NAME` | `str` | Name of the physical site |
| `SITE_TYPE_CODE` | `str` | Type of site (e.g., `CC` = Consecutive Connection) |
| `STATE_CODE` | `str` | Two-letter state abbreviation |
| `ZIP_CODE` | `str` | ZIP code of the site — **primary join key for ZIP-level aggregation** |
| `TRIBAL_CODE` | `str` | Tribal nation code if applicable *(nullable)* |
| `LATITUDE` | `float` | Site latitude (decimal degrees) |
| `LONGITUDE` | `float` | Site longitude (decimal degrees) |
| `CONGRESSIONAL_DISTRICT_NO` | `str` | Congressional district number |
| `COUNTY_SERVED` | `str` | County in which the site is located |
| `CITY_SERVED` | `str` | City in which the site is located |

</details>

---

<details>
<summary><strong>6.2 &nbsp;SDWA_PUB_WATER_SYSTEMS</strong> — Core registry of all public water systems (central entity)</summary>

<br>

| Column | Type | Description |
|--------|------|-------------|
| `PWSID` | `str` | **Primary key** — Public Water System ID |
| `PWS_NAME` | `str` | Name of the water system |
| `PRIMACY_AGENCY_CODE` | `str` | State/territory code of the primacy agency |
| `EPA_REGION` | `str` | EPA region number (`01`–`10`) |
| `PWS_TYPE_CODE` | `str` | `CWS`=Community · `NTNCWS`=Non-Transient Non-Community · `TNCWS`=Transient Non-Community |
| `OWNER_TYPE_CODE` | `str` | `F`=Federal · `S`=State · `L`=Local · `M`=Public/Private Mix · `P`=Private · `N`=Native American |
| `POPULATION_SERVED_COUNT` | `int` | Estimated number of people served |
| `SERVICE_CONNECTIONS_COUNT` | `int` | Number of service connections |
| `ACTIVITY_CODE` | `str` | `A`=Active · `I`=Inactive · `N`=Inactive (Non-NTNC) |
| `PWS_ACTIVITY_EFF_DATE` | `date` | Effective date of current activity status |
| `PRIMARY_SOURCE_CODE` | `str` | `GW`=Groundwater · `SW`=Surface Water · `GU`=Groundwater Under Influence |
| `GW_SW_CODE` | `str` | Simplified: `GW` or `SW` |
| `OUTSTANDING_PERFORMER` | `str` | `Y` if system has outstanding performance designation |
| `OUTSTANDING_PERF_BEGIN_DATE` | `date` | Date of outstanding performer designation |
| `ORG_NAME` | `str` | Organization/operator name |
| `ADMIN_NAME` | `str` | Administrator contact name |
| `EMAIL_ADDR` | `str` | Contact email *(may be redacted)* |
| `PHONE_NUMBER` | `str` | Contact phone |
| `ADDRESS_LINE1` | `str` | Mailing address |
| `CITY_NAME` | `str` | City |
| `STATE_CODE` | `str` | State |
| `ZIP_CODE` | `str` | ZIP code |
| `COUNTRY_CODE` | `str` | Country (typically `US`) |
| `CITIES_SERVED` | `str` | Pipe-delimited list of cities served |
| `COUNTIES_SERVED` | `str` | Pipe-delimited list of counties served |

</details>

---

<details>
<summary><strong>6.3 &nbsp;SDWA_VIOLATIONS_ENFORCEMENT</strong> — All recorded violations and enforcement actions &nbsp;🎯 <em>Primary feature + target source</em></summary>

<br>

| Column | Type | Description |
|--------|------|-------------|
| `PWSID` | `str` | **Foreign key** → `SDWA_PUB_WATER_SYSTEMS.PWSID` |
| `VIOLATION_ID` | `str` | Unique violation record ID |
| `FACILITY_ID` | `str` | Facility where violation occurred *(nullable)* |
| `POPULATION_SERVED_COUNT` | `int` | Population served at time of violation |
| `NPM_CANDIDATE` | `str` | `Y` if violation is a National Priority candidate |
| `PRIMACY_AGENCY_CODE` | `str` | State/territory of primacy |
| `EPA_REGION` | `str` | EPA region |
| `VIOLATION_CODE` | `str` | EPA violation code (see SDWA code table) |
| `VIOLATION_CATEGORY_CODE` | `str` | `MCL`=Max Contaminant Level · `MR`=Monitoring/Reporting · `TT`=Treatment Technique · `PN`=Public Notification · `OTHER` |
| `IS_HEALTH_BASED_IND` | `str` | ⭐ `Y` if health-based — **used to derive `health_based_violations` feature** |
| `CONTAMINANT_CODE` | `str` | EPA contaminant code |
| `CONTAMINANT_NAME` | `str` | Human-readable name (e.g., `Lead`, `Nitrate`, `Total Coliform`) |
| `COMPLIANCE_PERIOD_BEGIN_DATE` | `date` | Start of compliance monitoring period |
| `COMPLIANCE_PERIOD_END_DATE` | `date` | End of compliance monitoring period |
| `VIOLATION_BEGIN_DATE` | `date` | Date violation began |
| `VIOLATION_END_DATE` | `date` | Date violation ended *(nullable if ongoing)* |
| `RULE_CODE` | `str` | Applicable rule code (e.g., `110`=TCR · `400`=Lead & Copper) |
| `RULE_GROUP_CODE` | `str` | Broader rule grouping |
| `RULE_FAMILY_CODE` | `str` | Rule family (e.g., `100`=Microbials · `200`=Chemicals) |
| `SEVERITY_IND_CNT` | `int` | Count-based severity indicator |
| `PUBLIC_NOTIFICATION_TIER` | `int` | PN tier: `1`=most severe → `3`=least *(nullable)* |
| `CALCULATED_RTC_DATE` | `date` | Calculated return-to-compliance date |
| `RTC_DATE` | `date` | Actual return-to-compliance date *(nullable if unresolved — used for `unresolved_violations`)* |
| `ENFORCEMENT_ID` | `str` | Associated enforcement action ID *(nullable)* |
| `ENF_ACTION_TYPE_CODE` | `str` | Type of enforcement action taken |
| `ENF_ACTION_DATE` | `date` | Date of enforcement action |

</details>

---

<details>
<summary><strong>6.4 &nbsp;SDWA_FACILITIES</strong> — Physical infrastructure details for facilities within each water system</summary>

<br>

| Column | Type | Description |
|--------|------|-------------|
| `PWSID` | `str` | **Foreign key** → `SDWA_PUB_WATER_SYSTEMS.PWSID` |
| `FACILITY_ID` | `str` | Unique facility ID within the PWS |
| `FACILITY_NAME` | `str` | Name of the facility |
| `STATE_FACILITY_ID` | `str` | State-assigned facility identifier |
| `FACILITY_TYPE_CODE` | `str` | `CC`=Consecutive Connection · `DS`=Distribution System · `IN`=Intake · `WL`=Well · `TP`=Treatment Plant |
| `WATER_TYPE_CODE` | `str` | Type of water handled (`GW`, `SW`, `GU`, etc.) |
| `AVAILABILITY_CODE` | `str` | `E`=Emergency · `I`=Interim · `P`=Permanent · `S`=Seasonal |
| `SELLER_TREATMENT_CODE` | `str` | Treatment provided by seller (for purchased water) |
| `ACTIVITY_CODE` | `str` | `A`=Active · `I`=Inactive |
| `SUBMISSION_STATUS_CODE` | `str` | `A`=Approved · `P`=Pending |
| `IS_SOURCE_IND` | `str` | `Y` if this facility is a source |
| `IS_SOURCE_TREATED_IND` | `str` | `Y` if source water is treated |

</details>

---

<details>
<summary><strong>6.5 &nbsp;SDWA_SERVICE_AREAS</strong> — Geographic service area data for water systems</summary>

<br>

| Column | Type | Description |
|--------|------|-------------|
| `PWSID` | `str` | **Foreign key** → `SDWA_PUB_WATER_SYSTEMS.PWSID` |
| `SERVICE_AREA_TYPE_CODE` | `str` | Type of area served (e.g., `RETAIL`, `WHOLESALE`) |
| `IS_PRIMARY_SERVICE_AREA_CODE` | `str` | `Y` if this is the primary service area |
| `PRIMACY_AGENCY_CODE` | `str` | Primacy agency state code |

</details>

---

<details>
<summary><strong>6.6 &nbsp;SDWA_LCR_SAMPLES</strong> — Lead and Copper Rule sample results (critical for health-based features)</summary>

<br>

| Column | Type | Description |
|--------|------|-------------|
| `PWSID` | `str` | **Foreign key** → `SDWA_PUB_WATER_SYSTEMS.PWSID` |
| `SAMPLE_ID` | `str` | Unique sample identifier |
| `SAMPLING_START_DATE` | `date` | Date sample collection started |
| `SAMPLING_END_DATE` | `date` | Date sample collection ended |
| `RECONCILIATION_ID` | `str` | Reconciliation record ID |
| `PRIMACY_AGENCY_CODE` | `str` | State code |
| `SAR_ID` | `str` | Sample analyte result ID |
| `CONTAMINANT_CODE` | `str` | `PB90`=Lead 90th percentile · `CU90`=Copper 90th percentile |
| `RESULT_SIGN_CODE` | `str` | Result qualifier: `=`, `<`, or `>` |
| `SAMPLE_MEASURE` | `float` | ⭐ Measured concentration — **source for `lead_result` and `copper_result` features** |
| `UNIT_OF_MEASURE` | `str` | Unit (typically `mg/L` or `ppb`) |

</details>

---

<details>
<summary><strong>6.7 &nbsp;SDWA_GEOGRAPHIC_AREAS</strong> — Maps water systems to geographic/political boundaries</summary>

<br>

| Column | Type | Description |
|--------|------|-------------|
| `PWSID` | `str` | **Foreign key** → `SDWA_PUB_WATER_SYSTEMS.PWSID` |
| `GEO_ID` | `str` | Unique geographic area ID |
| `PRIMACY_AGENCY_CODE` | `str` | State code |
| `CITY_SERVED` | `str` | City served by the system |
| `COUNTY_SERVED` | `str` | County served |
| `STATE_SERVED` | `str` | State served |
| `ZIP_CODE_SERVED` | `str` | ⭐ ZIP code — **primary aggregation key for ZIP-level feature table** |
| `ANSI_ENTITY_CODE` | `str` | ANSI geographic entity code |
| `TRIBAL_CODE` | `str` | Tribal nation code *(nullable)* |

</details>

---

<details>
<summary><strong>6.8 &nbsp;SDWA_PN_VIOLATION_ASSOC</strong> — Associates public notification records with originating violations</summary>

<br>

| Column | Type | Description |
|--------|------|-------------|
| `PWSID` | `str` | **Foreign key** → `SDWA_PUB_WATER_SYSTEMS.PWSID` |
| `PN_VIOLATION_ID` | `str` | Public notification violation ID |
| `RELATED_VIOLATION_ID` | `str` | Original underlying violation ID |
| `PRIMACY_AGENCY_CODE` | `str` | State code |

</details>

---

## 7 · Setup: Connect to Shared S3 from SageMaker Studio

### 7.1 Prerequisites

- ✅ Active AWS Academy Learner Lab session (lab must be **Started** — green dot)
- ✅ Lab credentials refreshed — they expire at the end of every session

---

### 7.2 Configure AWS Credentials

> [!WARNING]
> AWS Academy **does not support persistent credential storage.** You must set credentials at the start of every lab session. **Never commit credentials to Git.**

**Option A — Environment Variables** *(recommended for notebooks)*

Open a terminal in SageMaker Studio via `File → New → Terminal`, then run:

```bash
export AWS_ACCESS_KEY_ID="your_access_key_id"
export AWS_SECRET_ACCESS_KEY="your_secret_access_key"
export AWS_SESSION_TOKEN="your_session_token"
export AWS_DEFAULT_REGION="us-east-1"
```

**Option B — AWS CLI**

```bash
aws configure
# Enter your Academy key, secret, session token, and region: us-east-1
```

**Option C — Credentials File**

Paste directly into `~/.aws/credentials`:

```ini
[default]
aws_access_key_id     = YOUR_KEY
aws_secret_access_key = YOUR_SECRET
aws_session_token     = YOUR_TOKEN
```

---

### 7.3 Verify Bucket Access

```bash
aws s3 ls s3://aai540-group5-public-866792937762-us-east-1-an/
```

Expected output lists the `raw/`, `processed/`, `models/`, and `reports/` prefixes.

---

### 7.4 Load Data in a Notebook

Install dependencies (first time only):

```bash
pip install -r requirements.txt
```

**Load a single SDWA table from S3:**

```python
import boto3
import pandas as pd
from io import StringIO

BUCKET = "aai540-group5-public-866792937762-us-east-1-an"
REGION = "us-east-1"

s3 = boto3.client("s3", region_name=REGION)

def load_sdwa_table(table_name: str) -> pd.DataFrame:
    """Load a raw SDWA CSV table from the shared S3 bucket."""
    key = f"raw/{table_name}.csv"
    obj = s3.get_object(Bucket=BUCKET, Key=key)
    df  = pd.read_csv(StringIO(obj["Body"].read().decode("utf-8")), low_memory=False)
    print(f"Loaded {table_name}: {df.shape[0]:,} rows × {df.shape[1]} columns")
    return df

# Example usage
pws   = load_sdwa_table("SDWA_PUB_WATER_SYSTEMS")
viols = load_sdwa_table("SDWA_VIOLATIONS_ENFORCEMENT")
geo   = load_sdwa_table("SDWA_GEOGRAPHIC_AREAS")
lcr   = load_sdwa_table("SDWA_LCR_SAMPLES")
```

**Load the ZIP-level processed feature table** *(preferred for modeling):*

```python
import pandas as pd

s3_path = "s3://aai540-group5-public-866792937762-us-east-1-an/processed/zip_features.parquet"
df = pd.read_parquet(s3_path, storage_options={"anon": False})
```

> [!NOTE]
> `pyarrow` and `s3fs` are required for Parquet reads — both included in `requirements.txt`.

---

## 8 · Reproducibility Workflow

### 8.1 Execution Order

Run notebooks **strictly in numerical order:**

```text
00_data_ingestion.ipynb   →  Confirms S3 access, profiles raw SDWA tables
01_eda.ipynb              →  ZIP-level distribution analysis, missing value review
02_preprocessing.ipynb    →  Cleaning, ZIP aggregation, feature engineering
03_modeling.ipynb         →  Train/val/test split, RF / XGBoost / LightGBM training
04_explainability.ipynb   →  SHAP values, feature importance, error analysis
```

---

### 8.2 Random Seeds

Set at the **top of every notebook** that involves randomness:

```python
RANDOM_SEED = 42

import numpy as np
import random

np.random.seed(RANDOM_SEED)
random.seed(RANDOM_SEED)
```

> [!TIP]
> For scikit-learn, XGBoost, and LightGBM, always pass `random_state=RANDOM_SEED`
> (or `seed=RANDOM_SEED`) to every estimator.

---

### 8.3 Data Versioning

> [!CAUTION]
> Raw files in `raw/` are **read-only.** Never overwrite them.

When making breaking changes to processed outputs, version by date prefix:

```text
s3://aai540-group5-public-866792937762-us-east-1-an/processed/2026-05-25/zip_features.parquet
```

The `processed/` root always points to the latest stable version used in `03_modeling.ipynb`.

---

### 8.4 Model Tracking

Copy this cell into every modeling run:

```python
import json

experiment = {
    "model":       "XGBoostClassifier",
    "version":     "v1",
    "date":        "2026-05-25",
    "author":      "your_name",
    "target":      "high_risk_label",
    "features":    list(X_train.columns),
    "hyperparams": model.get_params(),
    "val_roc_auc": val_roc_auc,
    "val_f1":      val_f1,
    "notes":       "Baseline ZIP-level classifier, no class weighting"
}

print(json.dumps(experiment, indent=2))
```

Save trained models to S3:

```python
import pickle

model_key = "models/xgboost_v1.pkl"
s3.put_object(Bucket=BUCKET, Key=model_key, Body=pickle.dumps(model))
print(f"Model saved → s3://{BUCKET}/{model_key}")
```

---

### 8.5 Git Hygiene

```bash
# Before every commit — confirm no credentials or data files are staged
git status

git add notebooks/ src/
git commit -m "feat: add ZIP-level feature aggregation pipeline"
git push origin main
```

**`.gitignore` must include:**

```gitignore
# Data (never commit raw or processed data)
data/raw/
data/processed/
*.csv
*.parquet

# Models
models/
*.pkl

# Secrets & credentials
.env
.aws/

# Python artifacts
__pycache__/
.ipynb_checkpoints/
```

---

## 9 · Team Conventions

| Convention | Rule |
|------------|------|
| **Branch naming** | `feat/<short-description>` · `fix/<short-description>` · `exp/<experiment-name>` |
| **Commit style** | Conventional Commits: `feat:` · `fix:` · `docs:` · `exp:` · `refactor:` |
| **Notebook cells** | Clear all outputs before committing — `Cell → All Output → Clear` |
| **Column names** | Always uppercase to match EPA schema (e.g., `PWSID`, not `pwsid`) |
| **Date fields** | Parse with `pd.to_datetime(..., errors='coerce')` — many fields contain nulls |
| **S3 writes** | All writes go under `processed/` or `models/` — never overwrite `raw/` |
| **Secrets** | Environment variables only — never hardcode credentials |

---

## 10 · References

| Resource | Link |
|----------|------|
| EPA SDWA Datasets Download | [echo.epa.gov](https://echo.epa.gov/tools/data-downloads/sdwa-download-summary) |
| EPA SDWA Data Dictionary (PDF) | [SDWA_FileDesc_2024.pdf](https://echo.epa.gov/system/files/SDWA_FileDesc_2024.pdf) |
| Safe Drinking Water Act — 40 CFR 141–143 | [ecfr.gov](https://www.ecfr.gov/current/title-40/chapter-I/subchapter-D) |
| EPA Lead and Copper Rule | [epa.gov](https://www.epa.gov/ground-water-and-drinking-water/lead-and-copper-rule) |
| AWS boto3 S3 Documentation | [docs.aws.amazon.com](https://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/s3.html) |
| SageMaker Studio User Guide | [docs.aws.amazon.com](https://docs.aws.amazon.com/sagemaker/latest/dg/studio.html) |

---

<div align="center">

**AAI-540 · Group 5 · Shiley-Marcos School of Engineering · University of San Diego · Spring 2026**

*Built with EPA open data and AWS Academy infrastructure*

> ⚠️ This model is a decision-support tool and does not replace official EPA, state, or local water authority reporting.

</div>
