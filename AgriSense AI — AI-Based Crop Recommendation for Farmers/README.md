# AgriSense AI — AI-Based Crop Recommendation for Farmers

> **An Intelligent, Multilingual Agricultural Intelligence Platform Powered by Deep Learning, Google Gemini, and Real-Time Weather Analytics**

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

**AgriSense AI** is a comprehensive, AI-powered agricultural decision support system built for Indian farmers. The platform integrates deep learning-based crop recommendation, yield prediction, crop disease diagnosis, multilingual AI-assisted chat support, live weather intelligence, and government scheme awareness into a single, unified Django web application.

Agriculture in India is inherently data-rich but decision-poor. Farmers — particularly smallholders — lack access to timely, precise recommendations for crop selection, yield forecasting, and disease management. Decisions made on intuition or outdated guidance lead to poor soil utilization, suboptimal yields, and significant financial losses. AgriSense AI addresses this gap by combining a trained Artificial Neural Network (ANN) for crop classification, a separate ANN regressor for yield prediction, a Google Gemini 2.5 Flash-powered visual disease diagnosis engine (AgriLens), and a real-time weather dashboard — all within a secure, OTP-authenticated, role-based web portal available in English, Hindi, and Telugu.

---

## Objectives

- Train a deep learning ANN classifier on the Kaggle Crop Recommendation Dataset to recommend the optimal crop from 22 classes based on seven soil and climatic parameters, achieving validation accuracy exceeding 98%.
- Train a separate ANN regression model on the Crop Production in India dataset to predict expected crop yield in tonnes based on crop type, cultivation season, and area under cultivation.
- Deploy both trained models within a Django web application that persists each inference as a timestamped, user-linked log entry for historical tracking and admin analytics.
- Integrate Google Gemini 2.5 Flash multimodal API for AI-powered crop disease diagnosis from uploaded leaf or crop photographs, returning structured diagnoses covering crop identification, issue classification, root cause, remedy steps, and expert consultation advice.
- Provide a multilingual AgriExpert AI chatbot capable of understanding and responding in English, Hindi, and Telugu, automatically detecting the user's language from Unicode character ranges.
- Deliver a live weather dashboard using the OpenWeatherMap API supporting both GPS-based and manual city-based weather lookups, with a temperature heatmap intensity indicator and historical weather logging.
- Implement a Government Schemes module presenting farmers with categorized scheme listings fully translated into Hindi and Telugu.
- Build a role-segregated portal with separate Farmer and Admin Command Center dashboards, including user directory management, AI monitor health, chat log auditing, and analytics.

---

## Summary

AgriSense AI is a seven-application Django project deployed on a standard WSGI/ASGI server with SQLite as the persistence layer. The system is organized into the following Django applications: `accounts` (custom email-based authentication with OTP), `ml` (crop recommendation and yield prediction views and logs), `chatbot` (Gemini-powered multilingual chatbot), `weather` (OpenWeatherMap integration), `schemes` (government scheme management with i18n translations), and `portal` (landing page, farmer dashboard, admin command center, AgriLens AI disease diagnosis).

The ML inference layer (`ml/utils.py`) loads two pre-trained Keras ANN models and their corresponding joblib-serialized scalers and label encoder at application startup. The crop recommendation model takes a seven-feature input vector (N, P, K, temperature, humidity, pH, rainfall), scales it via a StandardScaler, and runs it through a four-layer ANN (256 → 128 → 64 → 22 softmax units) to produce a crop label and confidence percentage. The yield prediction model accepts a 131-element one-hot-encoded feature vector encoding crop type, season, and area, and passes it through a four-layer ANN regression head (256 → 128 → 64 → 1 linear unit) to output expected yield in tonnes.

The entire UI is available in English, Hindi, and Telugu via Django's LocaleMiddleware with compiled `.po`/`.mo` translation files maintained under the `locale/` directory.

---

## Features

### Before Login (Public Access)

- **Landing Page** — Branded public page introducing AgriSense AI with feature highlights, navigation links, and login/registration call-to-action buttons. Authenticated users are automatically redirected to their respective dashboards.
- **Farmer Registration** — Email-based sign-up flow. New accounts are created as inactive; a six-digit OTP is generated, emailed via Gmail SMTP, and verified before the account is activated.
- **OTP Verification** — Time-limited (10-minute) OTP confirmation screen. Expired or incorrect OTP inputs return a descriptive error message.
- **Login** — Email/password authentication with session-based state management. Login count is incremented on each successful authentication.
- **AgriExpert AI Chatbot (Anonymous Mode)** — Anonymous users can access the chatbot for basic agricultural queries. Conversations are saved with `is_anonymous=True` for moderation review.

### After Login — Farmer Portal

**Farmer Dashboard**
- Summary of total crop recommendations made and total yield predictions saved.
- Recent Crop Recommendations table (last 5 entries) with localized crop name display in Hindi or Telugu based on the active language.
- Recent Yield Prediction logs with crop name, season, area, and predicted yield.

**AI Crop Recommendation**
- Seven-parameter input form: Nitrogen (N), Phosphorus (P), Potassium (K), Temperature, Humidity, Soil pH, and Rainfall.
- ANN inference returning the recommended crop name and prediction confidence percentage.
- Contextual cultivation guidance for key crops (rice, maize, wheat, cotton).
- Each recommendation is persisted to `CropRecommendationLog` linked to the authenticated farmer.

**Yield Prediction**
- Input form accepting crop type, cultivation season, and area in hectares.
- Prediction via a 131-feature one-hot ANN regression model returning expected yield in tonnes.
- Each prediction is persisted to `YieldPredictionLog` with full metadata.

**AgriLens — AI Crop Disease Diagnosis**
- Upload interface for leaf or crop photographs (JPEG/PNG).
- Gemini 2.5 Flash multimodal API performs structured diagnosis returning: Crop Name, Issues Identified, Cause, Remedy (with farmer-friendly steps and medicine recommendations), and Expert Advice.
- Plain-text output with markdown stripped via `clean_markdown()` for clean display.

**AgriExpert AI Chatbot (Authenticated Mode)**
- Persistent, session-aware conversational AI powered by Gemini 2.5 Flash.
- Automatic language detection from message Unicode ranges: Telugu (U+0C00–U+0C7F), Hindi (U+0900–U+097F), English (default).
- Language locked for the session and injected into system prompts as explicit language-lock instructions.
- Supports text queries and image uploads for visual crop analysis.
- Conversations persisted to `ChatConversation` with user attribution.

**Live Weather Dashboard**
- Priority 1: GPS-based geolocation auto-populates weather data.
- Priority 2: Manual city name search fallback.
- Displays: city, temperature (°C), humidity (%), wind speed (m/s), weather description, rainfall (mm/hr), and condition icon.
- Dynamic heatmap intensity bar normalizing temperature on a 0–100 scale.
- Each lookup persisted to `WeatherLog` for historical tracking.

**Government Schemes**
- Categorized listing of six major schemes: PM-KISAN Samman Nidhi, PMFBY, PMKSY, Kisan Credit Card (KCC), Soil Health Card Scheme, and PKVY.
- Full in-application translation of scheme title, description, eligibility, and benefits into Hindi and Telugu via server-side Python dictionaries.
- Category labels also translated for a fully localized experience.

**Profile and Language Settings**
- Email update form with validation.
- Language switcher (English / Hindi / Telugu) persisted in the session with a one-year cookie lifetime.
- System role display: `ROOT_ADMIN` or `VERIFIED_FARMER`.

### After Login — Admin Command Center

**Admin Dashboard** — Real-time KPI cards: total registered farmers, total inferences, total yield predictions, recent activity (last 7 days). Latest 5 crop recommendation entries with user and timestamp.

**Analytics** — Crop recommendation distribution chart data (labels and counts) from database aggregation, and total user count.

**User Directory** — Full table of non-superuser farmer accounts ordered by registration date, with per-user activate/suspend toggle for node management.

**AI Monitor** — Gemini API status panel showing operational status, latency, model name, and active capability list.

**Chat Logs** — Last 50 conversations from all users for quality control and moderation.

**Government Scheme Bulk Upload** — JSON file upload interface for batch-importing scheme records.

---

## Technologies Used

### Programming Languages

- **Python 3.10** — all backend logic, ML inference, API integrations.
- **HTML5 / CSS3 / JavaScript** — frontend templates rendered via Django's template engine.

### Libraries & Frameworks

- **Django 4.x** — web framework for URL routing, ORM, templates, sessions, i18n (LocaleMiddleware), email backend, and CSRF protection.
- **python-dotenv** — environment variable management for secrets isolation.
- **requests** — HTTP client for OpenWeatherMap API calls.
- **Pillow** — image processing for the AI Lens upload pipeline.

### Machine Learning / AI

- **TensorFlow / Keras** — deep learning framework for both ANN models (`crop_recommendation_model.h5`, `yield_prediction_model.h5`), loaded at startup with `compile=False` for inference efficiency.
- **scikit-learn** — `StandardScaler` for feature normalization; `LabelEncoder` for crop class encoding/decoding; one-hot feature construction for yield prediction.
- **NumPy** — array construction and manipulation for model input preparation.
- **joblib** — serialization/deserialization of scalers and label encoder.
- **Google Gemini 2.5 Flash (`google-genai`)** — multimodal AI API for AgriLens disease diagnosis and AgriExpert chatbot, invoked via `genai.Client` with image bytes as `types.Part.from_bytes()`.

### Backend

- **Django Views** — function-based views with `@login_required` and `@user_passes_test` decorators for role-based access control.
- **Django Sessions** — server-side session storage for auth state and language preference.
- **Django Mail** — Gmail SMTP backend for OTP delivery.
- **Django i18n** — `LocaleMiddleware`, `i18n_patterns`, and compiled `.mo` files for English, Hindi, and Telugu.

### Database

- **SQLite 3** (`db.sqlite3`) — Django ORM models: `User`, `EmailOTP`, `Notification` (`accounts`); `CropRecommendationLog`, `YieldPredictionLog` (`ml`); `ChatConversation` (`chatbot`); `WeatherLog` (`weather`); `GovernmentScheme`, `SchemeCategory` (`schemes`).

### Deployment & DevOps

- **Django WSGI / ASGI** — dual application server entry points for synchronous and asynchronous deployment.
- **Python venv** — isolated virtual environment with all dependencies.

### Security

- **Email OTP Authentication** — 10-minute time-limited OTP activation gating for all new accounts.
- **Django CSRF Middleware** — all POST views CSRF-protected.
- **Django Password Validators** — four built-in validators enforced on registration.
- **python-dotenv secrets management** — all API keys and credentials loaded exclusively from `.env`.
- **Role-Based Access Control** — `@user_passes_test(lambda u: u.is_superuser)` guards all Admin Command Center views.

---

## Dataset

### Crop Recommendation Dataset

**Source:** Kaggle — `atharvaingle/crop-recommendation-dataset` (via `kagglehub`)

**Volume:** 2,200 records across 22 balanced crop classes (100 records per crop).

**Input Features (7 columns):**

| Feature | Description | Unit |
|---|---|---|
| `N` | Nitrogen content in soil | mg/kg |
| `P` | Phosphorus content in soil | mg/kg |
| `K` | Potassium content in soil | mg/kg |
| `temperature` | Average temperature | °C |
| `humidity` | Relative humidity | % |
| `ph` | Soil pH value | — |
| `rainfall` | Annual rainfall | mm |

**Target:** `label` — 22 crop classes: rice, maize, wheat, cotton, jute, sugarcane, coconut, papaya, mango, banana, apple, orange, grapes, watermelon, muskmelon, pomegranate, lentil, blackgram, mungbean, mothbeans, pigeonpeas, kidneybeans.

**Preprocessing:** `StandardScaler` (zero mean, unit variance) fitted on training data → serialized as `crop_scaler.pkl`. `LabelEncoder` (integer class indices 0–21) → serialized as `crop_label_encoder.pkl`. Train/test split: 80% (1,760 records) / 20% (440 records).

### Crop Production in India Dataset

**Source:** Kaggle — `abhinand05/crop-production-in-india` (via `kagglehub`)

**Features:** `Area` (hectares) + one-hot encoded `Season_{name}` and `Crop_{name}` columns = 131 total features.

**Target:** Crop production yield in tonnes (continuous regression target).

**Preprocessing:** One-hot encoding via pandas `get_dummies`. `StandardScaler` fitted and serialized as `yield_scaler.pkl` with `feature_names_in_` preserved for ordered inference-time vector construction.

---

## System Architecture

```
┌──────────────────────────────────────────────────────────────────────┐
│                    Browser (Farmer / Admin)                          │
│   Django Template Engine (HTML + CSS + JS) — English / Hindi / Telugu│
└──────────────────────────┬───────────────────────────────────────────┘
                           │ HTTP + Session Cookie
                           ▼
┌──────────────────────────────────────────────────────────────────────┐
│                Django Application Server                             │
│              core/urls.py (i18n_patterns routing)                    │
│                                                                      │
│  ┌─────────────┐  ┌──────────┐  ┌──────────┐  ┌───────────────┐    │
│  │  accounts/  │  │   ml/    │  │ chatbot/ │  │   weather/    │    │
│  │ Login/OTP   │  │ Crop Rec │  │ Gemini   │  │ OpenWeather   │    │
│  │ Signup      │  │ Yield    │  │ Chatbot  │  │ GPS + City    │    │
│  └─────────────┘  └────┬─────┘  └──────────┘  └───────────────┘    │
│                        │                                            │
│  ┌─────────────┐  ┌────┴─────┐  ┌──────────┐                       │
│  │  schemes/   │  │  portal/ │  │  locale/ │                       │
│  │ Gov Schemes │  │ Landing  │  │ hi / te  │                       │
│  │ Bulk Upload │  │Dashboards│  │ .mo/.po  │                       │
│  └─────────────┘  └────┬─────┘  └──────────┘                       │
│                   AgriLens API (Gemini 2.5 Flash)                   │
└──────────────────────────┬───────────────────────────────────────────┘
             ┌─────────────┼──────────────────┐
             ▼             ▼                  ▼
┌────────────────┐  ┌──────────────┐  ┌────────────────────────────┐
│  SQLite DB     │  │  ML Models   │  │  External APIs             │
│  db.sqlite3    │  │ (ml/models/) │  │                            │
│  User, OTP,    │  │ crop_rec.h5  │  │  Google Gemini 2.5 Flash   │
│  CropLog,      │  │ yield.h5     │  │  OpenWeatherMap API        │
│  YieldLog,     │  │ scalers.pkl  │  │  Gmail SMTP (OTP)          │
│  ChatLog,      │  │ label_enc    │  │                            │
│  WeatherLog,   │  │ .pkl files   │  │                            │
│  Schemes       │  └──────────────┘  └────────────────────────────┘
└────────────────┘
```

**Crop Recommendation Data Flow:** User submits 7-parameter form → Django POST → `predict_crop()` in `ml/utils.py` → `StandardScaler.transform()` → `crop_model.predict()` (ANN forward pass) → `argmax` → `LabelEncoder.inverse_transform()` → crop name + confidence % → `CropRecommendationLog.objects.create()` → rendered result template.

---

## Model Workflow

### Crop Recommendation Model (ANN Classifier)

**Step 1 — Data Ingestion:** Crop Recommendation Dataset downloaded via `kagglehub`. Loaded with pandas, split into 7-column feature matrix `X` and crop label column `y`.

**Step 2 — Preprocessing:** `LabelEncoder` encodes 22 crop strings to integer indices. `StandardScaler` normalizes all seven features. 80/20 train/test split.

**Step 3 — Architecture:**
```
Input (7) → Dense(256, ReLU) → BatchNormalization → Dropout(0.4)
          → Dense(128, ReLU) → Dropout(0.3)
          → Dense(64, ReLU)
          → Dense(22, Softmax)
```
Trained with categorical cross-entropy loss and Adam optimizer over 100 epochs.

**Step 4 — Serialization:** Trained model saved as `crop_recommendation_model.h5`. Scaler and encoder serialized via `joblib.dump()`.

**Step 5 — Inference:** At Django startup, `ml/utils.py` loads model and artifacts once. On each POST, `predict_crop(n, p, k, temp, hum, ph, rain)` scales the 7-element array, runs forward inference, returns crop name via `inverse_transform([argmax])` and confidence from `np.max(prediction) * 100`.

### Yield Prediction Model (ANN Regressor)

**Step 1 — Data Ingestion:** Crop Production in India Dataset downloaded via `kagglehub`. One-hot encoding applied to `Season` and `Crop` columns, producing a 131-feature matrix.

**Step 2 — Architecture:**
```
Input (131) → Dense(256, ReLU) → BatchNormalization → Dropout(0.4)
            → Dense(128, ReLU) → Dropout(0.3)
            → Dense(64, ReLU)
            → Dense(1, Linear)
```
Trained with mean squared error loss.

**Step 3 — Inference:** `predict_yield(area, season, crop_name)` builds a zero-initialized 131-element dictionary using `yield_scaler.feature_names_in_`, sets `Area`, sets the matching `Season_{name}` and `Crop_{name}` flags, scales the vector, and returns the rounded regression output.

### AgriLens — Gemini Disease Diagnosis

The `api_agri_lens_analyze` view reads uploaded image bytes, constructs a Gemini 2.5 Flash request with a structured agricultural pathologist system prompt, and returns a 5-section plain-text diagnosis (Crop, Issues, Cause, Remedy, Expert Advice) after stripping markdown via `clean_markdown()`.

### AgriExpert AI Chatbot

The `chatbot_query` view detects user language via Unicode range checking, locks it in the session, and builds a multimodal Gemini request with a language-enforced AgriExpert system prompt. Text, image, or combined inputs are supported. Responses are cleaned and persisted to `ChatConversation`.

---

## Implementation Details

**Project Structure:**

```
AI-Based-Crop-Recommendation-for-Farmers-main/
│
├── manage.py
├── .env                              # Environment secrets
│
├── core/
│   ├── settings.py                   # All application settings (i18n, email, DB)
│   ├── urls.py                       # Root URL dispatcher with i18n_patterns
│   ├── wsgi.py / asgi.py
│
├── accounts/
│   ├── models.py                     # User (email-auth), EmailOTP, Notification
│   ├── views.py                      # login, signup, verify_otp, logout, profile
│   └── urls.py
│
├── ml/
│   ├── models.py                     # CropRecommendationLog, YieldPredictionLog
│   ├── views.py                      # crop_recommend_view, yield_predict_view
│   ├── utils.py                      # predict_crop(), predict_yield() — model loader
│   └── models/                       # Serialized artifacts
│       ├── crop_recommendation_model.h5
│       ├── yield_prediction_model.h5
│       ├── crop_scaler.pkl
│       ├── yield_scaler.pkl
│       └── crop_label_encoder.pkl
│
├── chatbot/
│   ├── models.py                     # ChatConversation
│   └── views.py                      # chatbot_query (lang detect + Gemini)
│
├── weather/
│   ├── models.py                     # WeatherLog
│   └── views.py                      # weather_dashboard_view
│
├── schemes/
│   ├── models.py                     # SchemeCategory, GovernmentScheme
│   └── views.py                      # scheme_list_view, bulk_upload_view
│
├── portal/
│   └── views.py                      # Landing, dashboards, AgriLens, analytics
│
├── templates/                        # All HTML templates by app
├── locale/
│   ├── hi/LC_MESSAGES/django.po/.mo  # Hindi translations
│   └── te/LC_MESSAGES/django.po/.mo  # Telugu translations
│
├── AI_CRP_RECOMMENDATION.ipynb.txt   # Model training notebook
└── db.sqlite3
```

**Custom User Model:** `accounts.User` extends `AbstractUser` with `username=None` and `email` as `USERNAME_FIELD`. Accounts are created with `is_active=False` and activated post-OTP. `CustomUserManager` handles both `create_user()` and `create_superuser()`.

**Language Architecture:** Django's `LocaleMiddleware` in the middleware stack. `i18n_patterns()` in `core/urls.py` prefixes all user-facing URLs with locale codes. Language is activated via `translation.activate(lang_code)` and persisted in the session cookie for one year. Scheme and crop name translations are applied server-side via Python dictionaries in `schemes/views.py` and `portal/views.py`, avoiding any external translation service dependency.

**Chatbot Language Detection:** `detect_language()` inspects Unicode ranges: Telugu (`\u0C00–\u0C7F`), Hindi (`\u0900–\u097F`), English (fallback). Detected language is stored in `request.session["chat_language"]` and injected into the Gemini system prompt as a target-language instruction to prevent language code-switching mid-response.

---

## Results

### Crop Recommendation Model (ANN Classifier)

| Metric | Value |
|---|---|
| Model Type | ANN — Keras Sequential Classifier |
| Input Features | 7 (N, P, K, Temperature, Humidity, pH, Rainfall) |
| Output Classes | 22 crop categories |
| Training Epochs | 100 |
| Training Accuracy (Epoch 1) | ~33.0% |
| Training Accuracy (Epoch 50) | ~97.9% |
| Training Accuracy (Epoch 100) | ~98.5% (peak: 98.7%) |
| Best Validation Accuracy | ~98.4% (observed at Epochs 38, 64, 79, 85) |
| Stable Validation Accuracy Range | 97.0% – 98.4% |
| Minimum Validation Loss | ~0.044 |
| Regularization | BatchNormalization + Dropout (0.4 / 0.3) |

**Convergence Behavior:** The model achieves 81% validation accuracy in the first epoch and exceeds 92% by epoch 2, demonstrating strong initial learning. Training stabilizes above 97% from epoch 20 onward with no signs of overfitting across all 100 epochs, attributable to the combined regularization of BatchNormalization and dual Dropout layers.

### Yield Prediction Model (ANN Regressor)

| Metric | Value |
|---|---|
| Model Type | ANN — Keras Sequential Regressor |
| Input Features | 131 (Area + one-hot Season and Crop) |
| Output | Yield in tonnes (continuous) |
| Loss Function | Mean Squared Error (MSE) |
| Architecture | Dense(256) → Dense(128) → Dense(64) → Dense(1) |

### System-Level Performance

| Component | Behavior |
|---|---|
| Crop Confidence Score | Returned as percentage (e.g., 94.73%) for farmer transparency |
| AgriLens Output | Structured 5-section response: Crop, Issues, Cause, Remedy, Expert Advice |
| Chatbot Language Accuracy | Deterministic Unicode detection — 100% accurate for Telugu and Hindi scripts |
| Weather API Latency | Real-time with 10-second timeout; defaults to "Bengaluru" if no city specified |

---

## Installation

### Prerequisites

- Python 3.10 or higher
- Git
- Google Gemini API key (Gemini 2.5 Flash access)
- OpenWeatherMap API key (free tier)
- Gmail account with App Password for SMTP

### Step 1 — Clone the Repository

```bash
git clone https://github.com/harisaigithub/AI-Based-Crop-Recommendation-for-Farmers.git
cd AI-Based-Crop-Recommendation-for-Farmers
```

### Step 2 — Create and Activate Virtual Environment

```bash
python -m venv venv
source venv/bin/activate       # Linux / macOS
venv\Scripts\activate          # Windows
```

### Step 3 — Install Dependencies

```bash
pip install django python-dotenv google-genai tensorflow keras scikit-learn numpy joblib requests pillow
```

### Step 4 — Configure Environment Variables

Create a `.env` file in the project root:

```
SECRET_KEY=your-django-secret-key
DEBUG=True
AI_API_KEY=your-google-gemini-api-key
WEATHER_API_KEY=your-openweathermap-api-key
EMAIL_HOST_USER=your-gmail-address@gmail.com
EMAIL_HOST_PASSWORD=your-gmail-app-password
```

### Step 5 — Train and Place ML Models

Run the Jupyter notebook `AI_CRP_RECOMMENDATION.ipynb` in Google Colab or locally to generate model artifacts. Place the following files in `ml/models/`:

```
ml/models/
├── crop_recommendation_model.h5
├── yield_prediction_model.h5
├── crop_scaler.pkl
├── yield_scaler.pkl
└── crop_label_encoder.pkl
```

### Step 6 — Apply Migrations

```bash
python manage.py migrate
```

### Step 7 — Create Superuser

```bash
python manage.py createsuperuser
```

### Step 8 — Compile Translations

```bash
python manage.py compilemessages
```

### Step 9 — Run the Development Server

```bash
python manage.py runserver
```

Access the application at `http://127.0.0.1:8000/`.

### Step 10 — Seed Government Schemes (Optional)

Log in as superuser, navigate to Admin Command Center → Bulk Upload, and import a JSON file:

```json
[
  {
    "category": "Income Support",
    "title": "PM-KISAN Samman Nidhi",
    "description": "Direct income support to eligible farmer families in three annual installments.",
    "eligibility": "Small and marginal farmer families with cultivable land records.",
    "benefits": "Rs 6000 per year directly credited to bank account.",
    "official_link": "https://pmkisan.gov.in",
    "is_active": true
  }
]
```

---

## Usage

### Registration and Login

1. Open `http://127.0.0.1:8000/` and click **Get Started**.
2. Enter email and password on the signup page.
3. Enter the 6-digit OTP received by email on the verification screen.
4. Log in to access the farmer dashboard.

### Crop Recommendation

1. Navigate to **Crop Recommendation** from the sidebar.
2. Enter N, P, K, temperature, humidity, soil pH, and rainfall values.
3. Click **Recommend** to receive the predicted crop name, confidence score, and cultivation guidance.

### Yield Prediction

1. Navigate to **Yield Prediction**.
2. Select crop type, season, and enter area in hectares.
3. Click **Predict** to see the estimated yield in tonnes.

### AgriLens Disease Diagnosis

1. Navigate to **AI Lens**.
2. Upload a photograph of the affected crop or leaf.
3. Receive a structured 5-section diagnosis: crop identification, issues, cause, remedy, and expert advice.

### AgriExpert AI Chatbot

1. Navigate to **Chatbot**.
2. Type your question in English, Hindi, or Telugu — or attach a crop image.
3. The chatbot auto-detects language and responds accordingly.

### Weather Dashboard

1. Navigate to **Weather**.
2. Allow browser geolocation for automatic weather data, or enter a city name manually.
3. View temperature, humidity, wind speed, rainfall, and heatmap intensity.

### Language Switching

1. Navigate to **Profile**.
2. Select the preferred language (English / Hindi / Telugu) from the dropdown.
3. The interface, including scheme content and crop names on the dashboard, switches to the selected language.

---

## Future Enhancements

- **IoT Soil Sensor Integration** — connect with smart soil sensors (NPK, moisture, pH probes) to auto-populate the crop recommendation form with live field readings, eliminating manual data entry.
- **Satellite NDVI Analysis** — integrate remote sensing APIs (Sentinel Hub, NASA Earthdata) for field-level vegetation health indices alongside crop recommendations.
- **Time-Series Yield Forecasting** — replace the static regression ANN with an LSTM or Temporal Fusion Transformer trained on multi-year production data for seasonal variability-aware yield forecasting.
- **Mobile PWA / Android Application** — convert to a Progressive Web App with offline capability for recommendations in low-connectivity rural areas, with camera integration for direct AgriLens use.
- **Real-Time Market Price Integration** — integrate commodity market price APIs (AgMarknet, Agriwatch) to show current Mandi prices alongside crop recommendations for economically informed crop selection.
- **Regional Language Expansion** — extend multilingual support to Tamil, Marathi, Bengali, Gujarati, and Kannada by adding compiled `.po`/`.mo` files and scheme translation dictionaries.
- **Pest and Disease Alert System** — build a subscription-based alert module monitoring regional crop disease outbreak reports from ICAR or state agriculture departments.
- **Precision Fertilizer Dosage Calculator** — extend crop recommendation output to include specific fertilizer application rates (kg/hectare) based on submitted NPK values and the recommended crop's nutrient requirements.
- **Farmer Community Forum** — add a peer discussion board within the portal where farmers can post queries and receive responses from AgriExpert AI and verified agricultural officers.
- **Cloud Deployment with PostgreSQL** — containerize with Docker, deploy on AWS EC2 or Google Cloud Run, serve static files via S3/Cloud Storage, replace SQLite with PostgreSQL, and configure HTTPS via Nginx and Let's Encrypt.

---

## Contributing

Contributions are welcome. Please follow the standard Git workflow:

1. **Fork** the repository to your GitHub account.
2. **Clone** your fork locally:
   ```bash
   git clone https://github.com/your-username/AI-Based-Crop-Recommendation-for-Farmers.git
   cd AI-Based-Crop-Recommendation-for-Farmers
   ```
3. **Create a feature branch:**
   ```bash
   git checkout -b feature/your-feature-name
   ```
4. **Implement** your changes. For new ML models, include training notebooks and place artifacts in `ml/models/`. For new languages, add `.po` files under `locale/{lang_code}/LC_MESSAGES/` and compile with `python manage.py compilemessages`.
5. **Test** the application locally, verifying authentication flows, ML inference endpoints, and language switching.
6. **Commit** with a descriptive message:
   ```bash
   git add .
   git commit -m "feat: add <brief description>"
   ```
7. **Push** and open a Pull Request against `main` with a clear description of changes, motivation, and evaluation metrics.

Ensure all new views use `@login_required` where appropriate and all API keys are accessed exclusively via `os.getenv()`.

---

## License

This project is licensed under the **MIT License**.

```
MIT License

Copyright (c) 2024 Hari Sai Parasa

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

<br><br>

## ***To acquire this project, please contact***

**GitHub:** https://github.com/harisaigithub
**Email:** [harisaiparasa@gmail.com](mailto:harisaiparasa@gmail.com)

For collaborations, customizations, or deployment support, feel free to reach out.
