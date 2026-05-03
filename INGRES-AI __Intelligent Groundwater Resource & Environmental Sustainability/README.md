# INGRES-AI

### *Intelligent Groundwater Resource & Environmental Sustainability — AI-Powered Decision Support System*

> A full-stack, AI-augmented groundwater Decision Support System (DSS) for Andhra Pradesh, integrating real-time district-level hydrological data, machine learning-based stress prediction, RAG-powered conversational intelligence, and crop-water resource planning into a unified, role-aware web platform.

---

## Table of Contents

1. [Introduction](#introduction)
2. [Objectives](#objectives)
3. [Summary](#summary)
4. [Features](#features)
5. [Technologies Used](#technologies-used)
6. [Dataset](#dataset)
7. [System Architecture](#system-architecture)
8. [Model Workflow](#model-workflow)
9. [Implementation Details](#implementation-details)
10. [Results](#results)
11. [Installation](#installation)
12. [Usage](#usage)
13. [Future Enhancements](#future-enhancements)
14. [Contributing](#contributing)
15. [License](#license)
16. [Contact](#contact)

---

## Introduction

Groundwater depletion is one of the most critical environmental challenges facing Andhra Pradesh's agricultural economy. With over 60% of irrigation demand met by groundwater extraction, unsustainable pumping practices, erratic monsoon recharge cycles, and inadequate policy-level visibility have pushed numerous mandals into semi-critical and over-exploited categories under the Central Ground Water Board (CGWB) assessment framework.

**INGRES-AI** (Intelligent Groundwater Resource & Environmental Sustainability — AI) is a purpose-built Decision Support System that transforms raw district-level groundwater assessment data into actionable intelligence. The platform serves two distinct audiences: citizens and farmers who need accessible groundwater status information, and government officials who require advanced analytical tools including ML-based stress prediction, mandal-level risk mapping, intervention scenario modelling, and early warning systems.

The system integrates a Retrieval-Augmented Generation (RAG) chatbot powered by Google Gemini to deliver grounded, hallucination-resistant answers to groundwater queries. Statistical data is retrieved deterministically from the live database before any generative AI is invoked — ensuring that all numerical outputs are traceable to verified 2024 assessment records rather than model-generated approximations.

---

## Objectives

- Provide citizens, farmers, and government officials with accurate, district-wise groundwater availability and extraction data from Andhra Pradesh's 2024 CGWB assessment.
- Implement a machine learning pipeline to predict groundwater stress categories (Safe, Semi-Critical, Critical, Over-Exploited) at the mandal level from hydro-geological input parameters.
- Deploy a RAG-based AI assistant that retrieves structured data from the database before generating natural language explanations, preventing statistical hallucination.
- Build a Resource Planning Feasibility Analyzer that evaluates crop water demand against district-level groundwater availability for Kharif and Rabi seasons.
- Enable official government users to run intervention scenario simulations and visualise projected outcomes of recharge enhancement, extraction reduction, and conservation policy actions.
- Deliver a multi-language platform (English, Hindi, Telugu) accessible across devices, deployed via Docker on Hugging Face Spaces.
- Enforce a two-tier role model — Public and Official — with official users requiring administrative approval before accessing sensitive analytical dashboards.
- Maintain a full AI usage audit log for every AI-generated response, recording model name, data source, and whether AI generation was invoked.

---

## Summary

INGRES-AI is a Django 4.x web application organised across four primary apps — `accounts`, `portal`, `chatbot`, and `maps` — served via Gunicorn, containerised with Docker, and deployed on Hugging Face Spaces at port 7860.

The data layer is anchored by four district-level database tables sourced from official 2024 CGWB groundwater assessment reports: `AP_DISTRICT_GW_SUMMARY_2024` (extraction and recharge metrics per district), `AP_DISTRICT_CATEGORY_SUMMARY_2024` (unit-level safety classifications), `AP_CATEGORY_COMPARISON_2023_2024` (year-on-year trend comparisons), and `AP_MANDAL_CATEGORY_2024` (mandal-level stress classifications).

Two machine learning models are deployed: a groundwater stress category classifier (`groundwater_model.pkl`) trained on a 130-row hydro-geological dataset, and a crop-water resource planning model (`resource_planner_model.pkl`) that predicts irrigation feasibility from crop type, season, rainfall, and soil-type inputs. Both models are loaded via `joblib` with LRU-caching for inference efficiency.

The RAG chatbot architecture separates data retrieval from text generation — district statistics are fetched deterministically from the ORM before Gemini is invoked to generate only explanatory text, never numbers. This two-step RAG design is enforced in both the public and official chatbot variants.

---

## Features

### Public / Pre-Login

- Informational landing page describing the platform mission and data sources.
- Role-based registration (Public / Official) with email OTP verification (10-minute expiry window).
- Secure login with session management and CSRF protection.
- Password reset via tokenised email links.

### Public User Dashboard (Citizen / Farmer)

- **Groundwater Overview:** District-wise summary cards displaying annual recharge (MCM), total extraction (MCM), net availability (MCM), and stage-of-extraction percentage for all Andhra Pradesh districts from the 2024 CGWB assessment.
- **Risk Status Classification:** Automatic colour-coded categorisation of each district as Safe (< 70%), Semi-Critical (70–90%), Critical (90–100%), or Over-Exploited (> 100%) based on extraction stage.
- **Advanced Visualisation:** Interactive charts and comparative views of district-level extraction trends, recharge versus extraction ratios, and category distributions.
- **Groundwater Risk Trend Analysis:** Year-on-year comparison of mandal-level risk categories between 2023 and 2024, highlighting improving, stable, or deteriorating units with remark annotations.
- **Resource Planning Feasibility Analyzer:** Input crop type, season (Kharif/Rabi), rainfall category, soil type, and land size to receive a model-backed feasibility assessment, estimated water requirement, and advisory on groundwater stress compatibility.
- **Irrigation Recommender:** Crop-specific irrigation scheduling recommendations calibrated against district groundwater availability.
- **Irrigation Advisory Dashboard:** Public access portal for seasonal irrigation guidance based on district extraction stages.
- **Water Buddy AI Assistant (Public):** RAG-powered chatbot for groundwater queries. District-specific queries retrieve live database records first; generative AI produces only the interpretive explanation, not the statistics.
- **Interactive Map:** Leaflet.js-based geographic map displaying district markers with cached latitude/longitude coordinates for all 26 Andhra Pradesh districts, showing key groundwater parameters on selection.
- **Multi-Language Support:** Interface available in English, Hindi, and Telugu via Django's i18n framework with persistent language preference cookies.

### Official User Dashboard (Government / Policy)

- **Official Analytics Overview:** Aggregate platform metrics including total extraction volume, average stage-of-extraction, district count by category, and priority-scored district rankings.
- **Mandal-Level Stress View:** Granular mandal-wise stress classification table sourced from `AP_MANDAL_CATEGORY_2024`, filterable by district and category.
- **Groundwater Prediction Lab:** Input hydro-geological parameters to invoke the trained ML classifier and receive a predicted stress category with confidence indicators.
- **Early Warning System:** Automated identification of districts approaching critical extraction thresholds, ranked by priority score computed as a weighted function of stage-of-extraction and extraction-to-recharge ratio.
- **Intervention Scenario Lab:** Simulate outcomes of policy interventions — recharge enhancement, extraction cap enforcement, conservation measures — and project resulting changes in stage-of-extraction category.
- **Stress Prediction Module:** Dedicated stress prediction interface using the `groundwater_model.pkl` classifier with input parameter forms aligned to training feature schema.
- **Groundwater Control Panel:** Consolidated official management view aggregating stress indicators, priority rankings, and AI-generated policy advisory.
- **Official AI Assistant:** Gemini-powered chatbot for official users with access to extended analytical context including category comparisons, mandal summaries, and multi-district queries.
- **District Insight API:** JSON endpoint returning AI-generated district-level analytical summaries with dataset attribution and generation timestamps.

---

## Technologies Used

### Programming Languages
- Python 3.11
- JavaScript (ES6+)
- HTML5 / CSS3

### Libraries & Frameworks
- Django 4.x — Web framework and ORM
- Whitenoise — Static file serving (middleware-integrated)
- `python-dotenv` — Environment variable management
- `joblib` — ML model serialisation and deserialisation with LRU cache

### Machine Learning / AI
- Scikit-learn — Groundwater stress category classifier and resource planner model training
- NumPy — Numerical feature engineering
- Google Generative AI SDK (`google-genai`) — Gemini 2.5 Flash model integration
- RAG Architecture — Structured database retrieval before generative text production
- AI Usage Audit Logger — Per-request logging of model name, source, AI invocation status, and user ID

### Backend
- Django ORM — Relational data access across all four groundwater assessment tables
- `joblib.load` with `@lru_cache(maxsize=1)` — Efficient in-memory model caching
- Django i18n / LocaleMiddleware — Multi-language request routing (English, Hindi, Telugu)
- Gmail SMTP (TLS 587) — Email OTP and password reset delivery
- `functools.lru_cache` — Dataset option caching for resource planner UI

### Frontend
- Django Template Language (DTL) — Server-side HTML rendering
- Leaflet.js — Interactive geographic map with district markers
- Chart.js — Extraction trend and category distribution charts
- JavaScript Fetch API — Asynchronous AI assistant and prediction API calls

### Database
- SQLite 3 — Development and production (Hugging Face Spaces)

### Deployment & DevOps
- Docker — Containerised application packaging (`python:3.11-slim` base)
- Hugging Face Spaces (Docker SDK) — Cloud deployment at port 7860
- Gunicorn — Production WSGI server
- `entrypoint.sh` — Container startup script running `migrate`, `collectstatic`, and `gunicorn`
- Git LFS — Large file tracking for binary ML model artifacts (`.pkl` files)

### Security
- Django CSRF middleware with Hugging Face Spaces trusted origins
- Email OTP verification with 10-minute expiry (`accounts/models.py: is_expired()`)
- `is_official_approved` flag — Gated official dashboard access pending admin approval
- Django password validators (similarity, minimum length, common password, numeric)
- Role-based access guard (`_official_access_allowed`) on all official-tier views

---

## Dataset

### Primary Groundwater Assessment Data

**Source:** Central Ground Water Board (CGWB), Andhra Pradesh — Dynamic Ground Water Resources Assessment 2024.

The system ingests four structured assessment tables loaded into the SQLite database:

| Table | Description | Key Fields |
|---|---|---|
| `AP_DISTRICT_GW_SUMMARY_2024` | District-level groundwater budget | Recharge MCM, Extraction MCM (irrigation/industrial/domestic), Net Availability MCM, Stage of Extraction % |
| `AP_DISTRICT_CATEGORY_SUMMARY_2024` | Unit-level safety classification count per district | Total units, Safe, Semi-Critical, Critical, Over-Exploited, Saline |
| `AP_CATEGORY_COMPARISON_2023_2024` | Year-on-year mandal category change | Stage % 2023/2024, Category 2023/2024, Remark (Improved/Stable/Deteriorated) |
| `AP_MANDAL_CATEGORY_2024` | Mandal-level stress category | District, Assessment Unit Name, Category 2024 |

### ML Training Dataset

**File:** `final_groundwater_ml_training_dataset_130rows.csv`

A curated 130-record dataset of hydro-geological parameters used to train the groundwater stress category classifier. Features include extraction stage percentages, recharge volumes, and extraction type breakdowns. Target variable is the CGWB stress category (Safe / Semi-Critical / Critical / Over-Exploited).

**Preprocessing:** Label encoding for categorical features, min-max normalisation for continuous hydro-geological inputs, and train/test split for model evaluation. Encoder and target label mappings are persisted as `resource_planner_encoders.pkl` and `resource_planner_target.pkl` alongside the model artifact.

### Resource Planner Dataset

**File:** `resource_planner_dataset.csv`

Contains crop-season-rainfall-soil combinations with estimated water need values and irrigation feasibility labels. Used to train the resource planner model and dynamically populate UI dropdown options at runtime.

---

## System Architecture

```
┌──────────────────────────────────────────────────────────────────┐
│                       Presentation Layer                         │
│  Django Templates · Leaflet.js · Chart.js · JS Fetch API         │
│  Multi-language: English / Hindi / Telugu (LocaleMiddleware)      │
└──────────────────────────┬───────────────────────────────────────┘
                           │ HTTP / WSGI (Gunicorn:7860)
┌──────────────────────────▼───────────────────────────────────────┐
│                       Application Layer                          │
│                                                                  │
│  ┌──────────────┐  ┌───────────────────────────────────────────┐ │
│  │   accounts   │  │                  portal                   │ │
│  │  ──────────  │  │  ─────────────────────────────────────── │ │
│  │ Custom User  │  │  Public Dashboard · Official Dashboard   │ │
│  │ (Public /    │  │  Groundwater Overview · Mandal Stress    │ │
│  │  Official)   │  │  Prediction Lab · Early Warning System   │ │
│  │ Email OTP    │  │  Intervention Lab · Resource Planner     │ │
│  │ Notification │  │  Irrigation Recommender · Risk Trends    │ │
│  │ UserProfile  │  │  AI Assistant (Public + Official)        │ │
│  └──────────────┘  │  Control Panel · Insight APIs           │ │
│                    └───────────────────────────────────────────┘ │
│  ┌──────────────┐  ┌─────────────┐                               │
│  │   chatbot    │  │    maps     │                               │
│  │  ──────────  │  │  ─────────  │                               │
│  │ RAG Engine   │  │ MapMarker   │                               │
│  │ Gemini 2.5   │  │ Leaflet     │                               │
│  │ Public Bot   │  │ District    │                               │
│  │ Official Bot │  │ Coordinates │                               │
│  └──────────────┘  └─────────────┘                               │
└──────────────────────────┬───────────────────────────────────────┘
                           │ Django ORM
┌──────────────────────────▼───────────────────────────────────────┐
│                         Data Layer                               │
│  SQLite — AP_DISTRICT_GW_SUMMARY_2024                           │
│         — AP_DISTRICT_CATEGORY_SUMMARY_2024                     │
│         — AP_CATEGORY_COMPARISON_2023_2024                      │
│         — AP_MANDAL_CATEGORY_2024                               │
│  ML Artifacts: groundwater_model.pkl · resource_planner_*.pkl   │
└──────────────────────────┬───────────────────────────────────────┘
                           │
┌──────────────────────────▼───────────────────────────────────────┐
│                     External Services                            │
│  Google Gemini API (gemini-2.5-flash) · Gmail SMTP · Git LFS    │
│  Hugging Face Spaces (Docker) · Whitenoise Static Storage       │
└──────────────────────────────────────────────────────────────────┘
```

---

## Model Workflow

### Groundwater Stress Classifier Pipeline

```
Raw Input Parameters
(Stage of Extraction %, Recharge MCM,
 Extraction MCM by type, District)
        │
        ▼
  Feature Engineering
  └── Label encoding (district, category)
  └── Min-max normalisation (continuous features)
        │
        ▼
  Train / Test Split (130-row dataset)
        │
        ▼
  Scikit-learn Classifier Training
  (Random Forest / Gradient Boosting — groundwater_model.pkl)
        │
        ▼
  Model Serialisation → groundwater_model.pkl (joblib)
        │
  At Inference (Prediction Lab / Stress Prediction):
        │
        ▼
  _load_groundwater_model()  [LRU-cached, loads once per process]
        │
        ▼
  Input vector → model.predict() → Stress Category
  (Safe / Semi-Critical / Critical / Over-Exploited)
        │
        ▼
  JSON response with category and priority score
```

### RAG Chatbot Pipeline (Public & Official)

```
User Query (text)
        │
        ▼
  Step 1 — Structured Retrieval (Deterministic)
  ├── get_groundwater_context(user_msg)
  │     ├── District name detection (fuzzy alias matching)
  │     ├── ORM query: GroundwaterData.objects.filter(district_name__iexact=...)
  │     └── Returns: district, year, extraction %, recharge MCM, status category
        │
        ▼
  Step 2 — Response Construction
  ├── District FOUND:
  │     ├── Numerical block assembled from ORM record (no AI for numbers)
  │     └── AI Prompt: "Explain this status category. No numbers. No markdown."
  │               │
  │               └── Gemini 2.5 Flash → explanatory text only
  │
  └── District NOT FOUND (general query):
        └── Full prompt to Gemini 2.5 Flash with groundwater domain constraints
                  │
                  └── "No fabricated statistics. No markdown."
        │
        ▼
  Step 3 — AI Usage Audit Log
  _log_ai_usage(feature, ai_used, model_name, source, user_id)
        │
        ▼
  Step 4 — ChatConversation saved to database
  (user, message, response, bot_type, timestamp, is_anonymous)
        │
        ▼
  JSON Response → Frontend chat interface
```

### Resource Planner Prediction Pipeline

```
User Input: Crop, Season, Rainfall Category, Soil Type, Land Size (acres)
        │
        ▼
  Input Validation (all fields required)
        │
        ▼
  _load_resource_planner_artifacts()
  └── Loads: resource_planner_model.pkl
             resource_planner_encoders.pkl
             resource_planner_target.pkl
        │
        ▼
  Feature Encoding → model.predict()
        │
        ├──► Feasibility label (Feasible / Marginal / Not Feasible)
        │
        ├──► Water requirement estimate (calibrated from dataset)
        │
        └──► Gemini Advisory (if API key configured)
                  └── AI generates crop-season water management advisory
                      grounded in predicted feasibility and district context
```

---

## Implementation Details

**Two-Role Access Model:** The `accounts.User` model defines only two roles — `public` (Citizen/Farmer) and `official` (Government User). Official dashboard access is gated by a boolean `is_official_approved` field that must be set to `True` by an administrator after account review. The `_official_access_allowed` guard function is applied to all official-tier views, checking both role and approval status atomically.

**RAG Hallucination Prevention:** The chatbot enforces a strict separation between retrieval and generation. All numerical values in responses (extraction percentages, MCM figures, status categories) are drawn exclusively from the live ORM query result. The Gemini prompt explicitly instructs the model to produce no numbers and no markdown — only a professional explanatory paragraph. This design makes the chatbot's factual accuracy independent of Gemini's training data.

**Hardcoded District Coordinate Registry:** The `AP_DISTRICT_COORDINATES` dictionary in `portal/views.py` maps all 26 Andhra Pradesh district name variants (including post-reorganisation names like Alluri Sitharama Raju, Sri Potti Sriramulu Nellore, NTR District) to their geographic centroids. The `MapMarker` model caches these coordinates per `GroundwaterData` record to eliminate geocoding overhead on every map load.

**Priority Scoring Algorithm:** Each district in the official Early Warning System receives a priority score computed as `(stage_of_extraction_percent × 0.6) + (extraction_to_recharge_ratio × 100 × 0.4)`, clamped to the range [0, 100]. This weighted formula prioritises districts with both high extraction stages and poor recharge recovery, producing a more nuanced risk rank than extraction percentage alone.

**AI Audit Trail:** Every endpoint that invokes Gemini logs a structured warning-level message via Python's logging framework: `AI_USAGE feature=<name> ai_used=<yes/no> model=<gemini-2.5-flash> source=<gemini|rule-based-fallback> user_id=<id>`. This log provides a complete audit trail for verifying AI usage patterns in production without modifying the database schema.

**Containerised Deployment:** The `Dockerfile` uses `python:3.11-slim` as the base image. The `entrypoint.sh` script runs `python manage.py migrate`, `python manage.py collectstatic --noinput`, and `gunicorn core.wsgi:application --bind 0.0.0.0:7860` in sequence, making the container fully self-initialising on Hugging Face Spaces deployment.

**Multi-Language i18n:** The `LocaleMiddleware` is positioned in the middleware stack to intercept all requests before authentication, enabling language detection from the `django_language` cookie (1-year persistence) or the `Accept-Language` header. Translation catalogue paths (`locale/`) support English, Hindi, and Telugu, making the dashboard accessible to non-English-speaking farming communities.

---

## Results

| Metric | Value |
|---|---|
| Districts Covered | 26 (all Andhra Pradesh districts, 2024 CGWB data) |
| Assessment Tables Integrated | 4 (Summary, Category Summary, Comparison, Mandal) |
| ML Training Dataset Size | 130 records |
| ML Model Artifacts | 2 (Stress Classifier + Resource Planner) |
| AI Model | Google Gemini 2.5 Flash |
| RAG Design | Retrieval-first (statistics from ORM, generation for explanation only) |
| User Roles | 2 (Public / Official with approval gate) |
| Languages Supported | 3 (English, Hindi, Telugu) |
| Deployment Platform | Hugging Face Spaces (Docker, port 7860) |
| API Endpoints | 12+ (district insight, prediction, resource planner, AI assistant, map data) |
| Chat History Persistence | Per-user and anonymous, stored in `ChatConversation` model |

**Groundwater Coverage Findings (2024 Data):**
- Multiple Andhra Pradesh mandals have transitioned to Semi-Critical or Critical categories between 2023 and 2024, quantified by the `AP_CATEGORY_COMPARISON_2023_2024` table.
- The priority scoring system enables officials to rank districts by combined extraction and recharge stress, enabling targeted intervention planning.
- The resource planner model provides district-compatible crop feasibility assessments that directly reference extraction stage thresholds, supporting data-driven agricultural policy.

---

## Installation

### Prerequisites

- Python 3.11 or higher
- pip
- Git and Git LFS (`git lfs install`)
- A Google Gemini API key (for AI assistant features)
- A Gmail account with App Password (for email OTP)

### Step 1 — Clone the Repository

```bash
git clone https://github.com/yeswanth120-gif/ingrest-ai-groundwater.git
cd ingrest-ai-groundwater
git lfs pull
```

> Git LFS is required to download the binary ML model artifacts (`groundwater_model.pkl`, `resource_planner_model.pkl`).

### Step 2 — Create and Activate a Virtual Environment

```bash
python -m venv venv

# Windows
venv\Scripts\activate

# macOS / Linux
source venv/bin/activate
```

### Step 3 — Install Dependencies

```bash
pip install -r requirements.txt
```

### Step 4 — Configure Environment Variables

Create a `.env` file in the project root:

```env
SECRET_KEY=your-django-secret-key
DEBUG=True
ALLOWED_HOSTS=*
GEMINI_API_KEY=your-google-gemini-api-key
AI_API_KEY=your-google-gemini-api-key
EMAIL_HOST_USER=your-gmail-address@gmail.com
EMAIL_HOST_PASSWORD=your-gmail-app-password
```

> `GEMINI_API_KEY` powers the AI assistant and advisory features. `AI_API_KEY` is consumed by the chatbot module. Both should be set to the same Gemini API key value.

### Step 5 — Apply Migrations

```bash
python manage.py migrate
```

### Step 6 — Create a Superuser

```bash
python manage.py createsuperuser
```

### Step 7 — Load Groundwater Assessment Data

Load the 2024 CGWB district data into the database using Django's admin panel or a custom management command to populate the four assessment tables (`AP_DISTRICT_GW_SUMMARY_2024`, `AP_DISTRICT_CATEGORY_SUMMARY_2024`, `AP_CATEGORY_COMPARISON_2023_2024`, `AP_MANDAL_CATEGORY_2024`).

### Step 8 — Collect Static Files

```bash
python manage.py collectstatic
```

### Step 9 — Run the Development Server

```bash
python manage.py runserver
```

Navigate to `http://127.0.0.1:8000/` to access the platform.

### Docker Deployment (Hugging Face Spaces)

```bash
# Build and run locally
docker build -t ingres-ai .
docker run -p 7860:7860 --env-file .env ingres-ai
```

For Hugging Face Spaces deployment:

```bash
git init
git lfs install
git add .
git commit -m "Initial deployment"
git remote add origin https://huggingface.co/spaces/<username>/ingres-ai-groundwater
git push -u origin main
```

Set the following in Hugging Face Space Settings → Variables and Secrets: `SECRET_KEY`, `GEMINI_API_KEY`, `AI_API_KEY`, `EMAIL_HOST_USER`, `EMAIL_HOST_PASSWORD`, and `DEBUG=False`.

---

## Usage

### Platform URL

The live deployment is accessible at: `https://sky035-ingres-ai-groundwater.hf.space`

### Public User Workflow

1. Register with a Public role at `/accounts/signup/` and verify the emailed OTP.
2. Log in and navigate to **Groundwater Overview** (`/portal/groundwater-overview/`) to browse district extraction statistics.
3. Use **Groundwater Risk Trend** (`/portal/groundwater-risk-trend/`) to compare 2023 and 2024 mandal categories.
4. Access the **Resource Planning Analyzer** (`/portal/resource-planner/`) to assess crop feasibility against district groundwater availability.
5. Open the **Water Buddy AI Assistant** (`/portal/dashboard/public/ai-assistant/`) and query district-specific groundwater data in natural language.
6. View the **Interactive Map** for a geographic overview of district stress levels.

### Official User Workflow

1. Register with an Official role; await administrator approval of `is_official_approved`.
2. Log in to the **Official Dashboard** (`/portal/dashboard/official/`).
3. Navigate to the **Groundwater Prediction Lab** to run ML-based stress category predictions.
4. Use **Mandal Stress View** for granular mandal-level category analysis.
5. Access the **Early Warning System** to identify at-risk districts by priority score.
6. Run **Intervention Scenarios** to model the projected impact of policy actions on extraction categories.
7. Use the **Official AI Assistant** for comprehensive district comparisons, category trend summaries, and AI-generated policy advisories.

---

## Future Enhancements

- **Real-Time Sensor Integration:** Connect to IoT groundwater level sensors deployed in critical mandals for live extraction monitoring beyond annual assessment cycles.
- **Predictive Trend Modelling:** Train a time-series forecasting model on multi-year CGWB data to project district extraction stage trajectories over 5–10 year horizons.
- **Mobile PWA:** Develop a Progressive Web App version enabling offline map access and cached district data, targeting field use by agricultural extension officers in low-connectivity areas.
- **SMS Alert System:** Integrate an SMS notification gateway (Twilio / MSG91) to dispatch groundwater stress alerts to registered farmers in over-exploited mandals.
- **Satellite Recharge Estimation:** Incorporate ISRO Bhuvan or Copernicus satellite data to compute seasonal recharge estimates from vegetation indices (NDVI) and surface water extent.
- **Policy Impact Attribution:** Build a counterfactual analysis module allowing officials to attribute observed category changes to specific scheme interventions (e.g., APWREIS watershed programmes).
- **District Comparison Tool:** Add a side-by-side multi-district analytical dashboard allowing officials to benchmark extraction efficiency and recharge performance across peer districts.
- **Expanded Language Support:** Add Kannada and Tamil support to serve communities in border districts.

---

## Contributing

Contributions are welcome from environmental data scientists, Django developers, and domain experts in groundwater hydrology.

### Step 1 — Fork and Clone

```bash
git clone https://github.com/your-username/ingrest-ai-groundwater.git
cd ingrest-ai-groundwater
git lfs install
git lfs pull
```

### Step 2 — Create a Feature Branch

```bash
git checkout -b feature/satellite-recharge-integration
```

### Step 3 — Implement, Test, and Commit

```bash
git add .
git commit -m "feat: integrate Copernicus NDWI recharge index as model feature"
```

### Step 4 — Push and Open a Pull Request

```bash
git push origin feature/satellite-recharge-integration
```

Open a Pull Request against `main` on GitHub with a description of the change, the problem it addresses, and any relevant data sources referenced.

### Code Standards

- Follow PEP 8 for all Python source files.
- Isolate business logic in utility functions within `portal/views.py` or dedicated `utils.py` modules; keep view functions thin.
- All AI-invocating code paths must call `_log_ai_usage()` to maintain the audit trail.
- Never commit API keys, credentials, or `.env` files. Use Hugging Face Secrets for production values.
- Git LFS must track all `.pkl` binary model files via `.gitattributes`.

---

## License

```
MIT License

Copyright (c) 2026 INGRES-AI Project Team

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
```

---

## Contact

This project was developed as a final-year engineering capstone focused on AI-assisted environmental decision support for the Andhra Pradesh groundwater management ecosystem.

**GitHub Repository:** CONTACT US!!

**Live Deployment:** https://sky035-ingres-ai-groundwater.hf.space

---

&nbsp;

&nbsp;

## ***To acquire this project, please contact***

**GitHub:** https://github.com/harisaigithub

**Email:** [harisaiparasa@gmail.com](mailto:harisaiparasa@gmail.com)

For collaborations, customizations, or deployment support, feel free to reach out.