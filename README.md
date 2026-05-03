# FINAL-YEAR-PROJECTS

### *A Curated Portfolio of Production-Grade, AI-Powered Final Year Engineering Projects*

> A comprehensive collection of 27 end-to-end, full-stack web applications built as B.Tech final-year capstone projects — each integrating cutting-edge Generative AI, Machine Learning, Computer Vision, and Natural Language Processing into real-world, role-based platforms spanning healthcare, agriculture, cybersecurity, education, governance, and beyond.

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

This repository serves as the master portfolio for a complete set of final-year B.Tech capstone projects authored by **Hari Sai Parasa**. Each project is a standalone, production-grade web system — not a prototype or a demonstration — built with professional software engineering principles: modular Django application architecture, role-based access control, OTP-verified authentication, AI-powered intelligence layers, multi-language internationalization, and structured data pipelines.

The projects collectively span nine distinct problem domains: **cybersecurity**, **agriculture and food systems**, **healthcare and public health**, **education and gamified learning**, **governance and compliance**, **tourism and cultural heritage**, **environmental intelligence**, **financial technology**, and **human-computer interaction**. Each application addresses a documented, real-world gap — most aligned with Smart India Hackathon (SIH) problem statements or national priorities identified by AICTE, CGWB, Ministry of Agriculture, or the Ministry of Tourism, Government of India.

Every project in this collection integrates at minimum one large language model or computer vision AI service — including Google Gemini 2.5 Flash, Gemini 1.5 Flash, Anthropic Claude (via OpenRouter), Groq LLaMA 3.1, and YOLOv8 — alongside locally trained machine learning models such as Artificial Neural Networks, Random Forest classifiers, XGBoost pipelines, Isolation Forest anomaly detectors, and scikit-learn regression models.

This repository is intended for academic reviewers, industry evaluators, potential collaborators, and developers seeking high-quality, fully documented project references for final-year engineering submissions.

---

## Objectives

- Demonstrate end-to-end software engineering capability across the full development lifecycle: requirements analysis, system design, ML model training, full-stack web development, UI/UX design, and deployment.
- Deliver practical, domain-specific AI solutions to genuine societal problems across healthcare, agriculture, governance, education, tourism, cybersecurity, and environmental sustainability.
- Integrate state-of-the-art Generative AI APIs — including Google Gemini, Anthropic Claude, and Groq LLaMA — with locally trained ML models to construct hybrid intelligence systems that balance cloud capability with on-device performance.
- Implement production-quality security practices in every project: OTP-verified email registration, role-differentiated access control, session-managed authentication, and audit trail persistence.
- Provide multi-language accessibility (English, Hindi, Telugu) in all citizen-facing platforms, ensuring equitable access for regional communities across India.
- Align project themes with national priorities: Smart India Hackathon problem statements, AICTE compliance mandates, CGWB groundwater assessment frameworks, millet promotion under the International Year of Millets, and the Ministry of Tourism's digital transformation agenda.
- Establish a reusable architectural template — Django multi-app project with modular AI integration, i18n support, and SQLite/PostgreSQL persistence — applicable across diverse problem domains.

---

## Summary

The repository contains **27 independent, full-stack AI-powered projects**, each residing in its own subdirectory with a comprehensive `README.md` documenting its architecture, ML pipeline, installation instructions, results, and usage. The projects are implemented in Python/Django and are deployable on standard WSGI servers, Docker containers, or Hugging Face Spaces.

### Complete Project Index

| # | Project Name | Domain | Core AI Technology |
|---|---|---|---|
| 1 | AI-Enabled Cyber Incident and Safety Web Portal for Defence | Cybersecurity / Defence | Google Gemini 2.5 Flash, NLP Risk Scoring |
| 2 | AICTE Approval System | Governance / Compliance | Groq LLaMA 3.1, XGBoost, PyMuPDF |
| 3 | AgriSense AI — AI-Based Crop Recommendation for Farmers | Agriculture | ANN (Keras), Google Gemini 2.5 Flash |
| 4 | Automated Student Attendance Monitoring and Analytics System | Education | Face Recognition, OpenCV |
| 5 | Bharat AI — Smart Digital Tourism Platform | Tourism | Google Gemini, NLP |
| 6 | Cattle Breed Recognition & Livestock Intelligence Portal | Agriculture / Livestock | CNN, Transfer Learning |
| 7 | CattleBreed AI — Intelligent Livestock Breed Recognition System | Agriculture / Livestock | Deep Learning, Image Classification |
| 8 | CropWise AI — Intelligent Farming Assistant | Agriculture | ML Recommendation, Google Gemini |
| 9 | CyberSentinel — URL-Based Attack Identification on IP Data | Cybersecurity | OWASP ZAP, Wapiti, Google Gemini 1.5 Flash |
| 10 | Digital Learning Platform for Rural Schools | Education / Rural | Django LMS, AI Assistance |
| 11 | Digital Mental Health and Psychological Support System | Healthcare / Mental Health | Google Gemini, PHQ-9, GAD-7 |
| 12 | EduQuest — AI-Powered Gamified Learning Platform | Education | Google Gemini Flash, KeyBERT |
| 13 | Human Intention Drift Detection by Passive Behaviour Signals | HCI / Security | Isolation Forest, Keystroke Biometrics |
| 14 | INGRES-AI — Intelligent Groundwater Resource & Environmental Sustainability | Environment / Governance | RAG, Google Gemini, ML Classification |
| 15 | Industry Training and AI Mock Interview Platform | EdTech / Placement | Google Gemini 2.5 Flash, Behavioural Detection |
| 16 | MilletChain — AI-Powered Millet Agri-Commerce Platform | Agriculture / Commerce | scikit-learn Regression, Demand Forecasting |
| 17 | QuantumPredict — Stock Trend Forecasting System | Financial Technology | Random Forest, ApexCharts |
| 18 | RuralEdu STEM Quest | Education / Rural | AI Tutoring, Gamification |
| 19 | Sikkim Tourism AI Platform | Tourism | Google Gemini, Itinerary Planning |
| 20 | Smart Classroom Timetable Scheduler | Education / Administration | Constraint Satisfaction, Scheduling Algorithms |
| 21 | Smart Health Surveillance and Early Warning System | Public Health | ML Risk Engine, Early Warning Alerts |
| 22 | SmartAttend — AI-Powered Attendance Management System for Rural Schools | Education | Face Recognition, OpenCV |
| 23 | SmartPilgrim | Tourism / Heritage | AI Crowd Prediction, QR Slot Booking |
| 24 | Trace AI — Finding Missing Person Intelligence Platform | Public Safety | Google Gemini 1.5 Flash, OpenCV |
| 25 | Travel Coordinator — Smart Trip Tracker | Tourism / Productivity | AI Planning, Itinerary Management |
| 26 | URL-Based Attack Identification on IP Data | Cybersecurity | Vulnerability Scanning, IP Threat Intelligence |
| 27 | Video Insights Detection | Media Intelligence | YOLOv8, Google Gemini Flash, Claude (OpenRouter) |

---

## Features

### Common Architectural Features Across All Projects

All 27 projects share a consistent, production-quality architectural baseline:

- **Role-Based Access Control:** Every system implements at minimum two user tiers (standard user and administrator), with most implementing three or more (e.g., Farmer / Buyer / Admin, Student / Teacher / Guardian / Admin, Pilgrim / Temple Authority). Access is enforced at the view level through role-based decorators.
- **OTP-Verified Authentication:** Accounts are created in an inactive state; a six-digit time-limited OTP is dispatched via Gmail SMTP and verified before account activation — preventing spam registrations and ensuring verified access.
- **Django Multi-App Architecture:** Each project is organized into modular Django applications, each encapsulating a distinct domain (accounts, ML inference, chatbot, dashboard, reports, portal), enabling clean separation of concerns.
- **AI Integration Layer:** Every project integrates at least one external AI API or locally trained ML model — providing intelligent automation, personalization, or predictive capability beyond what rule-based systems can deliver.
- **Responsive Frontend:** All UIs are built with Bootstrap 5 or Tailwind CSS, ensuring accessibility across desktop, tablet, and mobile form factors.
- **Audit Trails:** AI chatbot conversations, administrative actions, and inference logs are persisted to the database for accountability and forensic review.
- **PDF Reporting:** Most platforms support downloadable PDF reports (via ReportLab) summarizing key data, ML outputs, or AI analysis results.

### Domain-Specific Highlights

**Cybersecurity Projects (3 projects):**
- Multi-tool vulnerability scanning pipeline orchestrating OWASP ZAP, Wapiti, custom header inspection, and TCP port scanning through a unified Django interface.
- Keyword-weighted NLP risk scoring engine classifying incident severity without cloud ML dependency.
- Multimodal forensic analysis of suspicious images and screenshots using Google Gemini Vision API.
- Automatic CVSS scoring, severity tier assignment, and per-finding remediation guidance for every detected vulnerability.
- PCAP network capture ingestion via Scapy for offline traffic analysis.
- Blocked IP registry with middleware-enforced request interception.

**Agriculture & Food Systems Projects (6 projects):**
- ANN-based crop recommendation engine achieving over 98% validation accuracy across 22 crop classes from seven soil and climatic parameters.
- Separate ANN regression model predicting crop yield in tonnes from crop type, season, and cultivation area.
- Gemini-powered multimodal AgriLens disease diagnosis from leaf photographs, generating structured diagnoses covering root cause, remedy steps, and expert consultation advice.
- AI-driven millet price prediction trained on 160,000 synthetic transaction records with rule-based demand forecasting.
- Deep learning livestock breed classification via CNN and transfer learning from uploaded animal photographs.
- Multilingual farmer interfaces (English, Hindi, Telugu) with government scheme awareness and live OpenWeatherMap weather dashboards.

**Healthcare & Public Health Projects (2 projects):**
- Clinically validated PHQ-9 (depression) and GAD-7 (anxiety) self-assessment tools with anonymous submission support.
- ML-based community health risk engine analysing real-time citizen-submitted health and sanitation data for outbreak pattern detection.
- Context-aware mental health AI chatbot with crisis detection logic that automatically surfaces emergency helpline resources.
- ASHA worker and health administrator roles with area-level early warning alert dispatch.

**Education & Gamified Learning Projects (6 projects):**
- XP progression engine with configurable levels, badge unlocks, daily streaks, and a global competitive leaderboard.
- Four distinct educational game types: Animated Quiz, Canvas Action Game, Drag-and-Drop Sorting, and Scientific Simulation.
- AI Scholar Mode enabling students to upload PDFs and conduct document-grounded AI Q&A sessions.
- Face recognition-based automated attendance with fraud prevention and analytics dashboards for teachers.
- Multi-institute architecture partitioning subjects, games, and users by institutional affiliation.
- AI mock interview engine with multi-stage adaptive questioning (Introductory → Technical → Behavioural) and real-time behavioural authenticity detection.

**Governance & Compliance Projects (2 projects):**
- Automated AICTE mandatory disclosure extraction from PDFs using Groq LLaMA 3.1 with schema-guided JSON parsing.
- 16-point compliance risk engine evaluating faculty adequacy, infrastructure, accreditation, and financial data against AICTE regulatory norms.
- XGBoost-backed ML pipeline predicting institutional risk level and approval probability from a 50,000-record training dataset.
- RAG-powered groundwater decision support with deterministic database retrieval before any generative AI output — preventing statistical hallucination in official reports.
- Intervention scenario simulation for government officials modelling projected outcomes of recharge enhancement and conservation policies.

**Tourism & Heritage Projects (4 projects):**
- AI crowd prediction for Hindu temples using festival calendars, public holidays, weekend heuristics, and live booking data.
- QR-based darshan slot booking engine with database-level capacity enforcement and automated QR code generation.
- Geolocation-based nearby temple discovery via OpenStreetMap Overpass API with Haversine distance calculation.
- SOS alert system with GPS coordinate capture and real-time admin monitoring console.
- AI-powered multi-language tourism itinerary planning for Sikkim and Bharat-wide destinations.

**Human-Computer Interaction & Anomaly Detection (1 project):**
- Passive, non-intrusive intention drift detection using keystroke timing and mouse trajectory analysis.
- Isolation Forest anomaly detection requiring no cameras, no cloud connectivity, and no application modification.
- Configurable observation window and anomaly threshold, with a Tkinter-based local monitoring dashboard.

**Media Intelligence (1 project):**
- Dual-stage video analysis pipeline: Gemini Flash multimodal cloud analysis followed by local YOLOv8 Nano, OpenCV Haar Cascade, and KeyBERT processing.
- Eight-category activity recognition from combined YOLO object classes, face counts, motion magnitude, and frame composition.
- Claude-powered (via OpenRouter) conversational chatbot grounded in full video analysis context for natural language Q&A over video content.
- YouTube ingestion via `yt-dlp` and downloadable structured PDF insight reports.

**Financial Technology (1 project):**
- Random Forest classifier trained over a decade of historical equity OHLCV data for next-day price direction prediction.
- Interactive visualization suite: OHLC candlestick charts, feature importance donut charts, confusion matrix heatmaps, and prediction stability area charts — all rendered via ApexCharts.
- Market data integrity validation preventing logically inconsistent OHLCV inputs from reaching the inference engine.

---

## Technologies Used

### Programming Languages

- **Python 3.11+** — Core language for all backend development, ML pipelines, and data processing.
- **JavaScript (ES6+)** — Frontend interactivity, ApexCharts visualizations, and real-time UI updates.
- **HTML5 / CSS3** — Semantic markup and responsive styling.
- **SQL** — Relational data modelling via Django ORM targeting SQLite (development) and PostgreSQL (production).

### Libraries & Frameworks

- **Django 5.x / 5.2** — Primary web framework across all 27 projects, providing ORM, middleware, i18n, session management, and templating.
- **Bootstrap 5 / Tailwind CSS** — UI component libraries for responsive, mobile-first frontend design.
- **ApexCharts** — Interactive chart rendering for dashboards (stock prediction, health surveillance, analytics).
- **ReportLab** — Programmatic PDF report generation.
- **OpenPyXL** — Excel report generation and multi-sheet workbook management.
- **Pillow** — Server-side image processing and thumbnail generation.
- **Scapy** — PCAP network capture parsing and URL extraction for cybersecurity analysis.
- **PyMuPDF (fitz)** — PDF text extraction for AICTE disclosure processing.
- **yt-dlp** — YouTube video stream downloading for the Video Insights Detection platform.
- **Gunicorn + WhiteNoise** — Production WSGI server and static file serving.
- **qrcode** — QR code generation for darshan slot booking in SmartPilgrim.

### Machine Learning & AI

- **Google Gemini 2.5 Flash / 1.5 Flash** — Multimodal LLM used across 12+ projects for document analysis, image forensics, chatbot intelligence, disease diagnosis, and itinerary generation.
- **Anthropic Claude (via OpenRouter API)** — Conversational AI for video-grounded Q&A in Video Insights Detection.
- **Groq LLaMA 3.1** — High-throughput LLM inference for PDF data extraction in the AICTE Approval System.
- **YOLOv8 Nano (Ultralytics)** — Real-time object detection and human activity recognition for video frame analysis.
- **TensorFlow / Keras** — ANN model training for crop recommendation (4-layer classifier, 256→128→64→22 units) and yield prediction (regression head).
- **scikit-learn** — Random Forest (stock prediction), Isolation Forest (intention drift detection), XGBoost pipelines (AICTE risk scoring), StandardScaler, LabelEncoder, and joblib serialization.
- **OpenCV** — Haar Cascade face detection across multiple projects; optical flow motion analysis in video intelligence; histogram equalization for low-light face preprocessing.
- **KeyBERT** — Keyword extraction and semantic density scoring from video transcripts.
- **BERT (sentence-transformers)** — Embedding generation for RAG retrieval in INGRES-AI.

### Backend

- **Django REST-compatible views** — JSON endpoint design for AI inference, chatbot, and scan result retrieval.
- **Django Signals** — Event-driven notification dispatch on XP earn, badge unlock, and mentor message events.
- **Django LocaleMiddleware + `.po`/`.mo` files** — Full i18n pipeline for English, Hindi, and Telugu language support.
- **SMTP (Gmail)** — Transactional email for OTP dispatch, password reset, and admin notification.
- **Socket Programming** — TCP port scanning in CyberSentinel without external library dependency.

### External APIs & Services

- **Google Gemini Files API** — Async video upload and state polling before multimodal inference.
- **OpenWeatherMap API** — Live weather data with GPS and city-based lookup in AgriSense AI.
- **OpenStreetMap Overpass API** — Geospatial temple discovery with radius-based filtering in SmartPilgrim.
- **OWASP ZAP REST API** — Automated spider and active vulnerability scanning in CyberSentinel.
- **Wapiti (subprocess)** — External vulnerability scanner orchestrated from Django views with JSON output parsing.

### Database

- **SQLite** — Default persistence layer for development and capstone deployment.
- **PostgreSQL** — Production-ready alternative supported by Django ORM configuration.
- **Django ORM JSONField** — Structured storage of ML analysis results, AI responses, and scan findings.

### Deployment & DevOps

- **Docker** — Container-based deployment for INGRES-AI on Hugging Face Spaces.
- **Hugging Face Spaces** — Cloud hosting for select projects requiring persistent ML model serving.
- **Gunicorn** — Production WSGI server for Django applications.
- **WhiteNoise** — Static file serving middleware for production Django deployments without a CDN.

### Security

- **OTP-Based Email Verification** — Six-digit, time-limited (10-minute TTL) account activation codes via Gmail SMTP.
- **Django CSRF Middleware** — Cross-site request forgery protection enabled globally across all projects.
- **Role-Based Decorators** — Custom `@login_required` and role-checking decorators enforcing access at every view.
- **Blocked IP Middleware** — Request-level IP blocking in CyberSentinel against known malicious sources.
- **Base64 Multimodal Encoding** — Secure image transmission to Gemini Vision API without URL exposure.
- **Password Complexity Validation** — Server-side enforcement of password strength rules at registration.

---

## Dataset

Each project manages its own dataset, either sourced from public repositories or synthetically generated for training. Key datasets used across the collection include:

**Kaggle Crop Recommendation Dataset (AgriSense AI, CropWise AI):** 2,200 records with seven soil and climate parameters (N, P, K, temperature, humidity, pH, rainfall) across 22 crop classes. Preprocessed via StandardScaler; label-encoded target. Split 80/20 for training and validation.

**Crop Production in India Dataset (AgriSense AI):** State-wise historical crop production records used to train the ANN yield regression model. One-hot encoded across crop type (131 features), season, and area.

**Millet Transaction Dataset (MilletChain):** 160,000 synthetic records of millet market transactions (crop variety, region, season, quantity, price) generated under realistic agricultural price distribution assumptions for training the scikit-learn regression pipeline.

**Ten-Year Historical Equity Data (QuantumPredict):** A `s.csv` file containing 20 financial features per trading day over a decade, with an engineered binary directional target (next-day close above current close = 1, else 0). Trained a 100-estimator Random Forest classifier.

**AICTE Synthetic Institutional Dataset:** 50,000 synthetic records of engineering institution compliance profiles covering faculty ratios, lab counts, infrastructure scores, student enrollment, financial completeness, and accreditation status. Used to train the XGBoost risk regression and approval classification pipeline.

**Andhra Pradesh Groundwater Assessment 2024 (INGRES-AI):** District and mandal-level groundwater extraction, recharge, and stage-of-development data from the Central Ground Water Board (CGWB) 2024 annual assessment report. Loaded directly into the Django database and queried deterministically for RAG responses.

**Keystroke and Mouse Trajectory Logs (Human Intention Drift Detection):** Self-collected baseline behaviour sequences captured via `pynput` during a controlled profiling session. Feature vectors include inter-key intervals, dwell times, mouse velocity, acceleration, and path curvature. Used to train the Isolation Forest anomaly detection model.

**Video Frame Samples (Video Insights Detection):** Up to 40 uniformly sampled frames extracted per uploaded video using NumPy `linspace` over the video's total frame count. Each frame is processed independently by YOLOv8 Nano and OpenCV Haar Cascade; motion deltas are computed between consecutive frame pairs.

---

## System Architecture

All 27 projects follow a common layered architecture adapted to domain-specific requirements:

```
┌────────────────────────────────────────────────────────────────────┐
│                        Presentation Layer                          │
│  Bootstrap 5 / Tailwind CSS Templates  |  Django Template Engine   │
│  ApexCharts / Chart.js Dashboards      |  i18n Locale Middleware    │
└────────────────────────────────┬───────────────────────────────────┘
                                 │
┌────────────────────────────────▼───────────────────────────────────┐
│                      Application Layer (Django)                    │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐             │
│  │  accounts/   │  │  portal/     │  │  chatbot/    │             │
│  │  OTP Auth    │  │  Core Views  │  │  AI Chat     │             │
│  └──────────────┘  └──────────────┘  └──────────────┘             │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐             │
│  │  ml/ or      │  │  dashboard/  │  │  reports/    │             │
│  │  analysis/   │  │  Analytics   │  │  PDF Export  │             │
│  │  ML Inference│  │  & Metrics   │  │  (ReportLab) │             │
│  └──────────────┘  └──────────────┘  └──────────────┘             │
└────────────────────────────────┬───────────────────────────────────┘
                                 │
┌────────────────────────────────▼───────────────────────────────────┐
│                    Intelligence Layer                               │
│  ┌─────────────────────────┐   ┌─────────────────────────────────┐ │
│  │  Cloud AI APIs          │   │  Local ML Models                │ │
│  │  Google Gemini 2.5/1.5  │   │  Keras ANN (crop, yield)        │ │
│  │  Anthropic Claude       │   │  Random Forest (stock)          │ │
│  │  Groq LLaMA 3.1         │   │  XGBoost (AICTE risk)           │ │
│  │  OpenRouter             │   │  Isolation Forest (drift)       │ │
│  └─────────────────────────┘   │  YOLOv8 Nano (video)            │ │
│                                │  OpenCV Haar Cascade (face)     │ │
│                                │  KeyBERT (keywords, engagement) │ │
│                                └─────────────────────────────────┘ │
└────────────────────────────────┬───────────────────────────────────┘
                                 │
┌────────────────────────────────▼───────────────────────────────────┐
│                      Data Layer                                     │
│  SQLite / PostgreSQL  |  Django ORM  |  JSONField  |  Media Files   │
│  joblib-serialized ML models  |  .po/.mo locale files              │
└────────────────────────────────────────────────────────────────────┘
```

**Data Flow:** Authenticated requests traverse the presentation layer, where role-based decorators at the application layer gate access to domain-specific views. Views invoking ML inference load pre-serialized models at request time (or at application startup for high-frequency endpoints) and pass input vectors through the inference function. Results are persisted to the database and simultaneously rendered in the response. Requests requiring cloud AI dispatch a structured API call (with prompt engineering for JSON output), await the response, parse the payload, and persist the result. All AI interactions are logged to the audit table.

---

## Model Workflow

The following describes the generalized ML/AI pipeline shared across projects, with project-specific steps noted:

### Stage 1 — Data Preprocessing

- **Tabular data:** Feature vectors are normalized via `StandardScaler` or `MinMaxScaler`. Categorical features are encoded via `LabelEncoder` (ordinal) or `OneHotEncoder` (nominal). Engineered features (e.g., directional price target, binary stress category) are computed before splitting.
- **Image data:** Uploaded images are opened via Pillow, resized to model input dimensions (typically 224×224 for CNN-based models), converted to RGB, and normalized to [0, 1] range. For video frames, OpenCV reads the file and applies histogram equalization before face detection.
- **Text data:** Input descriptions are tokenized by the cloud LLM's internal tokenizer. For local NLP (risk scoring in the cyber portal), Python string operations extract keyword matches from a weighted term dictionary.
- **PDF data:** PyMuPDF extracts raw text page-by-page; the Groq API receives a structured prompt requesting JSON output conforming to a defined schema (faculty count, lab count, accreditation flag, etc.).

### Stage 2 — Model Training (offline, prior to deployment)

- **ANN Classifier (crop recommendation):** Four-layer sequential model (256 → 128 → 64 → 22 units, ReLU activations, softmax output). Trained with Adam optimizer, sparse categorical cross-entropy loss, batch size 32, for up to 100 epochs with early stopping. Achieved >98% validation accuracy on the Kaggle dataset.
- **ANN Regressor (yield prediction):** Four-layer regression network (256 → 128 → 64 → 1 unit, linear output). Trained with MSE loss. Serialized alongside StandardScaler and LabelEncoder via `joblib`.
- **Random Forest (stock prediction):** 100-estimator ensemble trained on 20 financial features with the engineered binary directional target. `n_jobs=-1` for parallel fitting. Serialized to `stock_model.pkl` via `joblib`.
- **XGBoost (AICTE risk):** Risk regression and approval classification pipelines with hyperparameter tuning on the 50,000-record synthetic dataset. Serialized via `joblib`.
- **Isolation Forest (intention drift):** Trained on a profiling session of approximately 100 keyboard/mouse observation windows. `contamination=0.05`. Serialized via `joblib`.

### Stage 3 — Evaluation

- **Classification:** Accuracy, precision, recall, F1-score, and confusion matrix computed on held-out test sets.
- **Regression:** MAE, RMSE, and R² evaluated on test partitions.
- **Anomaly detection:** Threshold calibration against labelled drift events; false positive rate measured at the 5th percentile contamination setting.
- **LLM outputs:** Structural validation of JSON payloads (key presence, type checking) before database persistence; failed parses trigger fallback messaging to the user.

### Stage 4 — Prediction (runtime)

- Input received from the web form, sanitized at the view level.
- Feature vector assembled, scaled using the serialized scaler.
- Inference called on the loaded model; output decoded using the serialized label encoder where applicable.
- Result persisted to the appropriate log model and returned in the HTTP response for rendering.

### Stage 5 — AI API Integration

- Structured prompt assembled with domain-specific context (incident description, crop parameters, video transcript, etc.).
- API call dispatched with `max_tokens` and JSON-enforcing instructions in the system prompt.
- Response payload validated for structural completeness; parsed into Python dict/list for template rendering.
- Full AI interaction (prompt + response) logged to the audit table with timestamp and model identifier.

---

## Implementation Details

**Django Project Structure:** Each project follows a flat multi-app Django layout under a single `manage.py`. Custom user models extend `AbstractBaseUser` or `AbstractUser` with a role field (e.g., `farmer`, `buyer`, `admin`; `student`, `teacher`, `guardian`, `admin`). Role assignment occurs at registration and is stored as a CharField on the user model.

**OTP Authentication Flow:** On registration, a six-digit OTP is generated via Python's `secrets.token_digits`, stored in a `PendingOTP` model alongside the user's email and a `created_at` timestamp, and dispatched via Django's `send_mail` using Gmail SMTP credentials. The verification view checks OTP value and timestamp within a ten-minute window before activating the account.

**Multimodal AI Invocation:** Images are read from the uploaded file object using Pillow, encoded to Base64, and passed to the Gemini multimodal API via the `inline_data` content type. Structured JSON prompts instruct the model to return a machine-parseable payload; the response is stripped of markdown fences and parsed with `json.loads`. Video files in Video Insights Detection are first uploaded to the Gemini Files API and polled at five-second intervals until state transitions to `ACTIVE` before inference is dispatched.

**i18n Pipeline:** Django's `LocaleMiddleware` intercepts each request and resolves the active language from the session cookie. Translation strings in all templates are wrapped in `{% trans %}` tags. Compiled `.mo` binary files under `locale/{lang}/LC_MESSAGES/` are loaded at server startup. Language switching is handled by a custom view that sets the session cookie and redirects.

**PDF Report Generation:** ReportLab's `SimpleDocTemplate` with `Paragraph`, `Table`, and `Spacer` flowables assembles structured reports. Each section (AI summary, ML results, chatbot log) is rendered as a separate flowable block, enabling modular report composition. Reports are served as HTTP responses with `Content-Type: application/pdf` and `Content-Disposition: attachment`.

**Behavioural Authenticity Detection (Mock Interview Platform):** During an active mock interview session, the browser's `getUserMedia` API captures webcam frames at two-second intervals. Each frame is encoded to Base64 and submitted to a Django endpoint that invokes OpenCV's face detection to count visible faces. Sessions where face count exceeds one or drops to zero for more than three consecutive checks are flagged as potentially inauthentic and annotated on the interview transcript.

**RAG Pipeline (INGRES-AI):** When a user submits a groundwater query, the system first executes a deterministic SQL query against the live database to retrieve district-level numerical data relevant to the query terms. The retrieved records are serialized to a structured string and injected into the Gemini prompt as grounded context before the generative model is invoked. The AI usage audit log records whether the database retrieval step was invoked and which records were surfaced.

---

## Results

The following summarises key performance metrics across the ML models deployed in this project collection:

| Project | Model | Metric | Value |
|---|---|---|---|
| AgriSense AI | ANN Crop Classifier (22 classes) | Validation Accuracy | > 98% |
| AgriSense AI | ANN Yield Regressor | MAE | < 5% of mean yield |
| QuantumPredict | Random Forest (100 estimators) | Test Accuracy | ~85–88% directional |
| AICTE Approval System | XGBoost Approval Classifier | Accuracy | > 90% on held-out test set |
| Human Intention Drift Detection | Isolation Forest | False Positive Rate | < 5% at contamination = 0.05 |
| CyberSentinel | Keyword NLP Risk Scorer | Precision / Recall | > 90% on labelled incident test set |
| Smart Health Surveillance | ML Risk Engine | Alert Precision | > 85% on synthetic outbreak dataset |
| Video Insights Detection | YOLOv8 Nano | mAP@0.5 | ~37.9% (COCO pretrained, zero-shot video) |

**Generative AI Quality:** All Gemini-integrated projects enforce structured JSON prompt templates validated by key-presence checks before database persistence. Malformed responses (missing keys, non-parseable JSON) are caught by try/except blocks and surfaced to users as retry-eligible error messages rather than silent failures.

**System Performance:** Django views performing ML inference (ANN, Random Forest) respond within 200–400ms under standard load, as models are loaded once at startup and held in memory. Cloud AI endpoints (Gemini, Groq) introduce 1–4 second latency per call, which is communicated to users via loading indicators in the frontend.

---

## Installation

The following instructions apply to any individual project within this repository. Navigate to the desired project's subdirectory and follow these steps.

### Prerequisites

- Python 3.11 or higher
- pip (Python package manager)
- Git
- A Gmail account with App Password enabled (for SMTP OTP dispatch)
- A valid Google Gemini API key (for Gemini-integrated projects)
- A valid Groq API key (for AICTE Approval System)
- A valid OpenRouter API key (for Video Insights Detection)

### Step 1 — Clone the Repository

```bash
git clone https://github.com/harisaigithub/FINAL-YEAR-PROJECTS.git
cd FINAL-YEAR-PROJECTS
cd "<Project-Name>"
```

### Step 2 — Create and Activate a Virtual Environment

```bash
python -m venv venv

# On Windows
venv\Scripts\activate

# On macOS / Linux
source venv/bin/activate
```

### Step 3 — Install Dependencies

```bash
pip install -r requirements.txt
```

> If a `requirements.txt` is not present in the project subdirectory, install the common dependencies manually:

```bash
pip install django pillow google-generativeai scikit-learn tensorflow keras joblib reportlab openpyxl whitenoise gunicorn
```

### Step 4 — Configure Environment Variables

Create a `.env` file in the project root (same directory as `manage.py`):

```env
SECRET_KEY=your-django-secret-key
DEBUG=True

# Email (for OTP dispatch)
EMAIL_HOST_USER=your-gmail@gmail.com
EMAIL_HOST_PASSWORD=your-gmail-app-password

# AI API Keys (include only keys relevant to your chosen project)
GEMINI_API_KEY=your-google-gemini-api-key
GROQ_API_KEY=your-groq-api-key
OPENROUTER_API_KEY=your-openrouter-api-key
OPENWEATHER_API_KEY=your-openweathermap-api-key
```

Load environment variables in `settings.py` using `python-dotenv`:

```bash
pip install python-dotenv
```

### Step 5 — Apply Database Migrations

```bash
python manage.py makemigrations
python manage.py migrate
```

### Step 6 — Compile Translations (for i18n-enabled projects)

```bash
python manage.py compilemessages
```

### Step 7 — Create a Superuser (Administrative Access)

```bash
python manage.py createsuperuser
```

### Step 8 — Load Seed Data (if applicable)

For projects with fixture data (government schemes, temple records, groundwater district data):

```bash
python manage.py loaddata fixtures/initial_data.json
```

### Step 9 — Train ML Models (if not pre-serialized)

For projects where serialized model files (`*.pkl`, `*.h5`, `*.keras`) are not included in the repository, execute the training script:

```bash
python train_model.py
# or
python ml/train.py
```

### Step 10 — Start the Development Server

```bash
python manage.py runserver
```

The application will be accessible at `http://127.0.0.1:8000/`.

### Production Deployment (Optional)

```bash
pip install gunicorn whitenoise
python manage.py collectstatic --noinput
gunicorn <project_name>.wsgi:application --bind 0.0.0.0:8000
```

---

## Usage

### Accessing the Application

Navigate to `http://127.0.0.1:8000/` in a web browser. The landing page is publicly accessible without authentication.

### Standard User Flow

1. **Registration:** Click **Register** on the landing page. Complete the registration form with your name, email, and password. Submit the form to receive a six-digit OTP at the provided email address.
2. **OTP Verification:** Enter the OTP on the verification screen. The account is activated upon successful verification.
3. **Login:** Authenticate with your email and password. You will be redirected to your role-specific dashboard.
4. **Core Feature Interaction:** Depending on the project, this may involve submitting a form for ML inference, uploading a file for AI analysis, starting an AI chatbot session, booking a slot, or taking a quiz.
5. **Results & History:** View prediction results, AI-generated recommendations, chatbot responses, or scan findings on the dashboard. Download PDF reports where available.
6. **Language Switching (i18n projects):** Use the language selector in the navigation bar to switch between English, Hindi, and Telugu. The selection persists across sessions via cookie.

### Administrative Flow

1. Log in with an account bearing the `admin` role, or use the superuser credentials created during installation.
2. Access the Admin Dashboard from the navigation bar to view platform-wide metrics: total users, total inferences, system health indicators, and recent activity logs.
3. Manage users (activate, deactivate, assign roles, delete), review AI interaction logs, update platform settings, and export data reports.
4. For security-critical projects (Cyber Portal, CyberSentinel), the admin panel provides additional controls: blocked IP management, incident reviewer assignment, and SOP audit review.

### API and External Tool Integration (CyberSentinel)

1. Ensure OWASP ZAP is running locally on port 8080 with the REST API enabled (`zap.sh -daemon -port 8080 -config api.key=your-key`).
2. Set the `ZAP_API_KEY` in your `.env` file.
3. Submit a target URL through the scanner interface. The system orchestrates ZAP, Wapiti, header inspection, and port scanning automatically.

---

## Future Enhancements

The following technically grounded improvements are identified across the project collection:

**AI & Machine Learning:**
- Replace SQLite with PostgreSQL and implement asynchronous Django Channels for real-time AI response streaming, eliminating page-reload latency for chatbot interactions.
- Extend the crop ANN to a Transformer-based model fine-tuned on satellite NDVI data to incorporate field-level crop health signals.
- Implement active learning pipelines where user-confirmed AI outputs (accepted vs. rejected SOP, confirmed vs. rejected attendance) are fed back into model retraining cycles.
- Deploy Intent Classification models for chatbot routing, enabling multi-domain chatbots to distinguish between informational queries, action requests, and crisis escalations.
- Replace the Isolation Forest in the drift detection system with a deep LSTM-based sequence model capable of capturing temporal dependencies in long keyboard/mouse session traces.

**Security & Compliance:**
- Integrate multi-factor authentication (TOTP via Google Authenticator) as an optional second factor for high-privilege administrative accounts.
- Add end-to-end encryption for stored AI chatbot conversations using AES-256 with user-managed key derivation.
- Implement differential privacy techniques for aggregated analytics dashboards to prevent re-identification of individual users from statistical reports.
- Obtain and integrate a formal CVSS scoring feed (NVD API) to replace heuristic CVSS assignments in CyberSentinel with NIST-verified scores.

**Scalability & Infrastructure:**
- Migrate ML inference to a dedicated FastAPI microservice with GPU-accelerated TensorFlow Serving or TorchServe, decoupling inference latency from the Django request-response cycle.
- Implement task queues (Celery + Redis) for all long-running AI operations (video analysis, PDF extraction, multi-tool vulnerability scanning) to prevent HTTP timeout failures on large inputs.
- Add Docker Compose orchestration across all projects for one-command multi-container deployment (web, database, Redis, Celery worker).
- Deploy on Kubernetes with horizontal pod autoscaling triggered by ML inference queue depth.

**User Experience:**
- Develop native Android and iOS companion apps using Flutter, sharing the Django REST backend via token-authenticated API endpoints.
- Implement WebSocket-based real-time dashboards (Django Channels) for live health surveillance alerts, pilgrim SOS monitoring, and cybersecurity scan progress updates.
- Add voice-based input for multilingual chatbots using browser Web Speech API, particularly for farmer-facing platforms where text literacy may be limited.

**Data & Analytics:**
- Integrate Apache Superset or Metabase as an embedded analytics layer for administrative dashboards, enabling drag-and-drop report customization without code changes.
- Connect real-time government data feeds (AgMarkNet for millet prices, IMD weather APIs, CGWB live monitoring stations) to replace or supplement synthetic and historical datasets.
- Implement federated learning across institutional deployments of the AICTE system to improve the XGBoost risk model with real compliance data without centralizing sensitive institutional records.

---

## Contributing

Contributions, issue reports, and feature requests are welcome. Please follow the standard Git workflow described below.

### Steps to Contribute

1. **Fork the repository** on GitHub.

2. **Clone your fork** locally:
   ```bash
   git clone https://github.com/<your-username>/FINAL-YEAR-PROJECTS.git
   cd FINAL-YEAR-PROJECTS
   ```

3. **Create a feature branch:**
   ```bash
   git checkout -b feature/your-feature-name
   ```

4. **Make your changes.** Ensure all new code follows the existing Django project structure, includes docstrings for all non-trivial functions, and does not introduce hardcoded API keys or credentials.

5. **Test your changes:**
   ```bash
   python manage.py test
   ```

6. **Commit your changes with a descriptive message:**
   ```bash
   git add .
   git commit -m "feat: add <feature description>"
   ```

7. **Push to your fork:**
   ```bash
   git push origin feature/your-feature-name
   ```

8. **Open a Pull Request** against the `main` branch of this repository. Include a clear description of what was changed and why.

### Contribution Guidelines

- Follow PEP 8 Python style conventions.
- Do not commit `.env` files, API keys, or database files (`db.sqlite3`).
- Add or update `requirements.txt` if new Python packages are introduced.
- For new projects, follow the existing README structure exactly.
- All pull requests will be reviewed before merging.

---

## License

This repository is licensed under the **MIT License**.

```
MIT License

Copyright (c) 2025 Hari Sai Parasa

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

This repository is authored and maintained by **Hari Sai Parasa**.

For academic inquiries, project demonstrations, technical questions, or collaboration discussions, please reach out through the channels below.

---

&nbsp;

&nbsp;

## ***To acquire this project, please contact***

**GitHub:** https://github.com/harisaigithub

**Email:** [harisaiparasa@gmail.com](mailto:harisaiparasa@gmail.com)

For collaborations, customizations, or deployment support, feel free to reach out.
