# AICTE Approval System

> **An AI-powered compliance management and approval workflow platform that automates mandatory disclosure analysis, risk scoring, and institutional review for AICTE-regulated engineering institutions in India.**

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

The All India Council for Technical Education (AICTE) requires all affiliated technical institutions to submit Mandatory Disclosure documents annually across six regulatory domains: Faculty Details, Laboratory Details, Infrastructure, Student Enrollment, Financial Details, and Accreditation. The current review process is largely manual, paper-intensive, and prone to delays — creating a significant bottleneck for thousands of institutions seeking annual approval renewals.

The AICTE Approval System addresses this bottleneck by digitising and automating the entire compliance lifecycle. Institutions upload their disclosure PDFs through a secure web portal; the system extracts structured data from each PDF using the Groq LLM API (LLaMA 3.1), runs a 16-point rule-based risk engine to compute compliance scores, and applies an XGBoost-backed machine learning pipeline to predict institutional risk levels and approval probabilities. AICTE authority reviewers access a dedicated dashboard to evaluate pending submissions, apply per-section review decisions, and issue approvals or rejections — all with a full audit trail.

The system consolidates what would otherwise require weeks of manual verification into an automated, near-real-time compliance review workflow.

---

## Objectives

- Build a secure, role-based web portal serving two distinct user types: institutions submitting disclosures and AICTE authority reviewers conducting compliance evaluations.
- Automate structured data extraction from uploaded mandatory disclosure PDFs using PyMuPDF for text extraction and the Groq LLaMA 3.1 API for schema-guided JSON parsing.
- Implement a 16-point rule-based compliance risk engine that evaluates faculty adequacy, infrastructure metrics, accreditation status, financial completeness, and student data consistency against AICTE norms.
- Train and deploy an XGBoost-based ML pipeline (risk regression, approval classification, faculty shortage prediction) on a 50,000-record synthetic institutional dataset to provide data-driven risk scores and approval probability estimates.
- Provide authority reviewers with granular per-section review capabilities, section-level decision tracking, and automated notification dispatch to institutions upon review completion.
- Generate fully formatted multi-sheet Excel reports (`openpyxl`) exportable per institution, covering all disclosure sections, risk analysis history, and section-by-section compliance status.
- Detect wrong-file uploads (e.g., a faculty PDF uploaded under the infrastructure section) through AI response key validation and surface actionable correction messages.

---

## Summary

The AICTE Approval System is a full-stack compliance automation platform with a Django REST Framework backend and a single-page HTML/CSS/JavaScript frontend. The backend provides 14 REST API endpoints covering institution registration, authentication, PDF upload and AI extraction, risk analysis, approval submission, authority review workflow, notifications, and Excel report download. The frontend is a single-file SPA (`index.html`, ~1,892 lines) styled in the Indian government colour scheme (navy, saffron, and India green) with the Rajdhani and Noto Sans typefaces.

The ML subsystem consists of three pre-trained models — an XGBoost risk score regressor, an XGBoost approval classifier, and a Linear Regression faculty shortage predictor — trained on a synthetically generated 50,000-row dataset that simulates realistic AICTE institutional parameters with calibrated risk labels. The rule-based `_compute_risk()` engine runs in parallel to the ML pipeline and produces human-readable risk factors and remediation suggestions grounded in actual AICTE norms (1:15 faculty-student ratio, 15 sqft/student area, 10,000 library book minimum, 1:3 computer ratio, 30% PhD faculty requirement).

---

## Features

### Before Login (Public / Landing)

- **Government-Styled Landing Page** — Full-screen hero with India tricolour accent bar, Ashoka emblem, saffron/navy/green colour palette, animated stats bar (institutions registered, approvals, pending), and dual portal entry cards for institutions and AICTE authority.
- **Institution Registration** — Multi-field form capturing institution name, AICTE ID, institution type, category, affiliated university, state, district, pincode, principal name, email, mobile, and password. Password stored using Django's `make_password` (PBKDF2-SHA256).
- **Institution Login** — Credential-based login with hashed password verification; returns institution ID, approval status, state, and AICTE ID.
- **Authority Login** — Separate login endpoint with hard-coded reviewer credentials for AICTE authority access.

### After Login — Institution Portal

- **Dashboard** — Summary panel displaying total sections uploaded, latest risk score and level, compliance percentage, pending notifications, and approval request status.
- **PDF Upload & AI Extraction** — Upload interface for six mandatory disclosure sections. Each uploaded PDF is text-extracted via PyMuPDF (`fitz`), passed to Groq LLaMA 3.1 with a section-specific JSON schema prompt, and the extracted data is merged into the institution's `InstitutionData` record. Wrong-file detection validates extracted keys against the declared section type and surfaces an error message when a mismatch is found.
- **AI Risk Analysis** — Triggers the 16-point rule-based risk engine and the ML pipeline simultaneously. Outputs a risk score (0–100), risk level (Low / Medium / High), compliance percentage, faculty shortage flag, infrastructure deficit flag, expired certification flag, faculty ratio, per-section score breakdown, risk factors list, and actionable suggestions.
- **Submit for Approval** — Packages the current risk snapshot, list of uploaded sections, and institution data into an `ApprovalRequest` record and notifies the AICTE authority.
- **Approval Status Tracking** — Real-time status view showing current request state (Submitted / Under Review / Approved / Rejected / Resubmitted), reviewed-by, authority notes, and per-section decisions.
- **Disclosures List** — Chronological log of all uploaded disclosure PDFs with section type, academic year, upload timestamp, review status, and AI extraction summary.
- **Notifications** — In-app notification feed with categorised alerts (info, warning, success, danger) covering upload confirmations, approval decisions, and compliance warnings.
- **Excel Report Download** — Generates and streams a formatted multi-sheet `.xlsx` report covering Overview, Faculty Details, Laboratory Details, Infrastructure, Student Details, Financial Details, Accreditation, AI Risk Analysis, and Disclosure Log sheets — each styled with navy headers, saffron section titles, and alternating row shading.

### After Login — AICTE Authority Portal

- **Pending Approvals Queue** — Lists all institutions with `submitted` or `under_review` approval requests, including risk score at submission, sections submitted, and submission timestamp.
- **Institution Review Panel** — Per-institution review view with full data breakdown across all six sections. Authority can assign per-section decisions (approved / rejected) with notes and issue an overall approval or rejection decision.
- **All Institutions Overview** — Paginated list of all registered institutions with approval status, risk level, state, and section upload completeness.
- **Authority Statistics Dashboard** — Aggregate metrics: total institutions, pending count, approved count, rejected count, average risk score, and section-wise upload completion rates.

---

## Technologies Used

### Programming Languages
- Python 3.10+
- JavaScript (ES2020, vanilla)
- HTML5 / CSS3

### Libraries & Frameworks
- **Django 5.1** — Backend web framework
- **Django REST Framework** — API view base classes and response serialisation
- **django-cors-headers** — CORS middleware for frontend-backend communication

### Machine Learning / AI
- **XGBoost** — Risk score regression (`XGBRegressor`) and approval classification (`XGBClassifier`), both with `n_estimators=200`, `max_depth=5`, `learning_rate=0.05`, histogram tree method
- **scikit-learn** — `LinearRegression` for faculty shortage amount prediction; `train_test_split` for dataset partitioning
- **Groq Python SDK** — LLaMA 3.1 8B Instant model (`llama-3.1-8b-instant`) for PDF-to-JSON structured data extraction with section-specific schema prompts
- **joblib** — Model serialisation and lazy-loading of `.pkl` files

### Backend
- **PyMuPDF (fitz)** — PDF text extraction from uploaded disclosure documents
- **openpyxl** — Multi-sheet Excel report generation with custom styling (fills, fonts, borders, merged cells)
- **python-dotenv** — Environment variable management for API keys
- **NumPy / Pandas** — Feature engineering during model training and inference

### Frontend
- **Vanilla JavaScript** — SPA routing, API communication (`fetch`), DOM manipulation
- **Chart.js 4.4** — Risk score trend charts, compliance bar charts, section score visualisations
- **Google Fonts** — Rajdhani (headings) and Noto Sans (body text)
- CSS custom properties for the Indian government design system (navy `#0A2240`, saffron `#FF6B00`, India green `#138808`)

### Database
- **SQLite3** — Default development database via Django ORM
- Django ORM migrations for schema management

### Deployment & DevOps
- **Uvicorn / Django ASGI** — ASGI application entry point (`asgi.py`)
- **Django `manage.py`** — Development server and migration management
- **SQLite CLI tools** (`sqlite3.exe`, `sqldiff.exe`, `sqlite3_analyzer.exe`) — Bundled for Windows database inspection

### Security
- **Django `make_password` / `check_password`** — PBKDF2-SHA256 hashed password storage for institutions
- **CSRF exemption** — Applied selectively to API views via `@csrf_exempt`
- **CORS** — `CORS_ALLOW_ALL_ORIGINS = True` (development); restrict in production

---

## Dataset

### Synthetic AICTE Institutional Dataset (`aicte_synthetic_dataset.csv`)
- **Generation:** Programmatically generated via `generate_aicte_dataset.py` using NumPy random distributions seeded at `random_state=42`.
- **Size:** 50,000 rows × 26 columns.
- **Schema:** `institution_id`, `total_students`, `ug_students`, `pg_students`, `total_faculty`, `required_faculty`, `faculty_phd_count`, `total_labs`, `total_classrooms`, `computer_count`, `library_books`, `total_area_sqft`, `hostel_capacity`, `annual_budget`, `naac_grade`, `iso_certified`, `nba_programs`, `faculty_ratio`, `phd_percentage`, `area_per_student`, `computer_student_ratio`, `faculty_shortage_amt`, `risk_score`, `risk_level`, `approved`, `approval_probability`.

**Key distributional parameters:**
- Total students: Normal distribution μ=1800, σ=900, clipped to [300, 6000]
- Faculty ratio: Derived from `total_students / total_faculty` with AICTE norm of 1:15
- NAAC grades: Multinomial distribution — A++ (8%), A+ (12%), A (20%), B++ (20%), B+ (18%), B (12%), None (10%)
- ISO certification: Bernoulli with p=0.70
- Risk labels: Computed via a deterministic scoring function matching the production risk engine (faculty shortage +25, high ratio +20, low PhD% +10, low area/student +20, high computer ratio +10, low library books +8, no labs +15, no NAAC +10, no ISO +3)
- Approval label (`approved`): Binary, derived from `approval_probability ≥ 50`

**Training / Test Split:** 80/20 with `random_state=42`.

**Feature Vector (18 features used for ML inference):**
`total_faculty`, `required_faculty`, `faculty_phd_count`, `total_students`, `total_labs`, `total_classrooms`, `computer_count`, `library_books`, `total_area_sqft`, `hostel_capacity`, `annual_budget`, `iso_certified`, `nba_count`, `faculty_ratio`, `phd_percentage`, `area_per_student`, `computer_student_ratio`, `naac_encoded`

NAAC grades are ordinal-encoded: A++=6, A+=5, A=4, B++=3, B+=2, B=1, None=0.

---

## System Architecture

```
┌──────────────────────────────────────────────────────────────────────┐
│               Single-Page Frontend (index.html)                      │
│  Screens: Landing, Institution Auth, Authority Auth,                 │
│           Institution Dashboard, Authority Dashboard                  │
│  Charts: Chart.js risk trend, compliance bars, section scores        │
└────────────────────────┬─────────────────────────────────────────────┘
                         │ fetch() REST API calls
┌────────────────────────▼─────────────────────────────────────────────┐
│               Django REST Framework Backend                           │
│                                                                      │
│  Auth:       /vvit/register/  /vvit/login/  /vvit/authority/login/   │
│  Upload:     /vvit/upload/   (PDF → PyMuPDF → Groq LLM → DB)         │
│  Risk:       /vvit/ai-risk/  (Rule Engine + ML Pipeline)             │
│  Workflow:   /vvit/submit-approval/  /vvit/approval-status/          │
│  Authority:  /vvit/authority/pending|review|all|stats/               │
│  Reporting:  /vvit/download-excel/  (openpyxl multi-sheet .xlsx)     │
│  Misc:       /vvit/dashboard/  /vvit/disclosures/  /vvit/notifications│
└────┬────────────────┬──────────────────┬──────────────────────────────┘
     │                │                  │
┌────▼───────┐  ┌─────▼──────────┐  ┌───▼──────────────────────────┐
│ SQLite DB  │  │  ML Subsystem  │  │  External APIs               │
│ (Django    │  │  (ml/)         │  │  Groq: llama-3.1-8b-instant  │
│  ORM)      │  │  XGBRegressor  │  │  (PDF structured extraction) │
│            │  │  XGBClassifier │  └──────────────────────────────┘
│ Models:    │  │  LinearReg     │
│ Institution│  │  risk_model.pkl│
│ InstitData │  │  approval_.pkl │
│ Disclosure │  │  faculty_.pkl  │
│ AIRiskAna- │  └────────────────┘
│ lysis      │
│ ApprovalReq│
│ Notification│
└────────────┘
```

---

## Model Workflow

### PDF Upload & AI Extraction Pipeline

1. **File Validation** — Uploaded file is validated for `.pdf` extension and non-empty content server-side.
2. **Text Extraction** — PyMuPDF (`fitz.open(stream=..., filetype='pdf')`) extracts all text from all pages; first 4,000 characters are passed to the LLM.
3. **Groq LLM Prompt** — A section-specific prompt instructs LLaMA 3.1 to return only valid JSON conforming to a predefined schema (e.g., `{"total_faculty":0,"faculty_phd_count":0,"faculty_details":[...]}` for the faculty section).
4. **Response Parsing** — Markdown fences are stripped; JSON is parsed. On failure, section defaults are applied.
5. **Wrong-File Detection** — Extracted keys are cross-checked against the declared section's expected key set. If no expected keys match but another section's keys are present, a descriptive error is returned.
6. **Data Merge** — Non-zero extracted values are selectively merged into the institution's `InstitutionData` record, preserving previously uploaded data from other sections.
7. **DisclosureSection Record** — A `DisclosureSection` object is created with the extracted text, AI response JSON, section type, academic year, and upload timestamp.
8. **Notification** — An in-app notification is created confirming the upload and noting the number of extracted data points.

### Risk Analysis Pipeline

9. **Rule-Based Engine (`_compute_risk()`)** — Executes 16 compliance checks against AICTE norms:
   - Faculty: shortage (actual vs. required), student-faculty ratio > 1:20, PhD% < 30%, duplicate faculty names
   - Infrastructure: area/student < 15 sqft, no labs, no classrooms, computer ratio > 1:3, library books < 10,000, students/classroom > 70, no hostel for large institutions
   - Accreditation: missing NAAC grade, expired certificates, no NBA programs, no ISO certification
   - Financials: missing budget data
   - Students: missing program list, UG+PG inconsistency with total
   - Historical: previous rejection count penalty
   
   Produces: risk score (0–100), risk level, compliance%, per-section scores dict, factors list, suggestions list.

10. **ML Pipeline (`run_ml_pipeline()`)** — Constructs an 18-feature DataFrame from `InstitutionData` via `feature_builder.py`, lazily loads three pre-trained models, and produces:
    - `risk_score` (XGBRegressor, clamped to [0, 100])
    - `approval_probability` (XGBClassifier `predict_proba`, converted to percentage)
    - `faculty_shortage_pred` (Linear Regression, binary threshold)

11. **AIRiskAnalysis Record** — Both pipeline results are stored in an `AIRiskAnalysis` record linked to the institution and optionally a specific `DisclosureSection`.

### Approval Workflow

12. **Submission** — Institution submits approval request; current risk snapshot, uploaded section list, and risk factors are frozen into the `ApprovalRequest` record.
13. **Authority Review** — Reviewer accesses pending queue, reviews per-section data, sets `section_decisions` dict with per-section status and notes, and issues overall `approved` or `rejected` decision.
14. **Notification Dispatch** — On review completion, the institution receives a formatted notification (success for approval, danger for rejection) with authority notes and section-level feedback.

---

## Implementation Details

**Dual Risk Engine Architecture:** The system runs both a deterministic rule-based engine (`_compute_risk()`) and an ML pipeline (`run_ml_pipeline()`) in parallel. The rule engine produces human-interpretable factors and suggestions grounded in specific AICTE regulatory thresholds, while the ML pipeline provides probabilistic approval likelihood. The rule engine's output is persisted in the `AIRiskAnalysis` model; the ML output augments the API response with `approval_probability`.

**Lazy Model Loading:** All three ML models are loaded from disk once per process via a module-level `_load_models()` function guarded by `None` checks. This prevents repeated disk I/O on every request while remaining stateless across requests.

**Per-Section Score Decomposition:** The risk engine populates a `section_scores` dict (`{'faculty': N, 'infrastructure': N, 'accreditation': N, 'financials': N, 'students': N}`) enabling the frontend to render a per-section compliance breakdown chart.

**Institutional Data Merge Strategy:** The `_merge_ai_data_to_institution()` function applies a non-destructive merge — only non-zero / non-empty values from the latest AI extraction overwrite existing fields. This preserves data from previously uploaded sections when a new section is uploaded.

**Historical Pattern Detection:** The risk engine queries `ApprovalRequest.objects.filter(institution=..., status='rejected').count()` and adds a penalty of `min(10, rejections × 3)` to the risk score for institutions with repeat rejections, incentivising systematic compliance improvement.

**Excel Report Generation:** The `DownloadInstitutionExcelView` builds a 9-sheet workbook using `openpyxl` with a consistent AICTE-branded style system: navy (`#0A2240`) header fills, saffron (`#FF6B00`) section title fills, alternating gray/white row shading, thin borders, and Arial/Rajdhani typography. Infrastructure sheets include AICTE minimum benchmarks in a dedicated column for direct comparison.

---

## Results

| Component | Metric | Value |
|---|---|---|
| Dataset Size | Rows | 50,000 institutions |
| Dataset Size | Feature Columns | 26 columns (18 used for ML) |
| Risk Model | Algorithm | XGBRegressor (`n_estimators=200`, `max_depth=5`, `lr=0.05`) |
| Approval Model | Algorithm | XGBClassifier (`n_estimators=200`, `max_depth=5`, `lr=0.05`) |
| Faculty Model | Algorithm | Linear Regression |
| Train / Test Split | Ratio | 80% / 20% |
| Risk Engine | Compliance Checks | 16 rule-based checks |
| Risk Thresholds | Low / Medium / High | < 30 / 30–59 / ≥ 60 |
| ML Risk Thresholds | Low / Medium / High | < 30 / 30–59 / ≥ 60 |
| Supported Sections | Disclosure Types | 6 (Faculty, Labs, Infrastructure, Students, Financials, Accreditation) |
| API Endpoints | Count | 14 REST endpoints |
| Excel Report | Sheets | 9 (Overview, Faculty, Labs, Infrastructure, Students, Financials, Accreditation, AI Risk, Disclosures Log) |
| LLM Model | Provider / Model | Groq / llama-3.1-8b-instant |
| LLM Max Tokens | Per extraction | 1,200 |
| LLM Temperature | Setting | 0.1 (near-deterministic) |

The rule-based risk engine enforces 16 AICTE-specific compliance norms, ensuring that risk scores are fully explainable with direct regulatory citations. The XGBoost models provide a complementary probabilistic layer trained on the distributional properties of 50,000 synthetic institutions, enabling generalisation beyond the hard thresholds of the rule engine.

---

## Installation

### Prerequisites

- Python 3.10 or higher
- `pip` package manager
- A Groq API key (free tier available at `console.groq.com`)

---

### Backend Setup

**1. Clone the repository:**
```bash
git clone https://github.com/harisaigithub/aicte-approval-system.git
cd aicte-approval-system/AICTE-Approval-System-backend
```

**2. Create and activate a virtual environment:**
```bash
python -m venv venv
source venv/bin/activate        # Linux / macOS
venv\Scripts\activate           # Windows
```

**3. Install Python dependencies:**
```bash
pip install django djangorestframework django-cors-headers \
            pymupdf groq xgboost scikit-learn joblib \
            pandas numpy openpyxl python-dotenv
```

**4. Configure environment variables:**

Create a `.env` file inside `AICTE-Approval-System-backend/`:
```env
GROQ_API_KEY=your_groq_api_key_here
```

**5. Apply database migrations:**
```bash
cd aicteapproval
python ../manage.py migrate
```

**6. (Optional) Regenerate the synthetic dataset and retrain models:**
```bash
# Regenerate dataset
python aicte_synthetic_dataset_generator.py

# Retrain ML models
cd ml
python train_models.py
```
Pre-trained `.pkl` files are already included in `ml/` and ready to use.

**7. Start the development server:**
```bash
python manage.py runserver 0.0.0.0:8000
```

The API will be available at `http://localhost:8000`.

---

### Frontend Setup

The frontend is a single static HTML file requiring no build step or package installation.

**1. Open the frontend directly in a browser:**
```bash
# From the project root
open AICTE-Approval-System-frontend/index.html
```

Or serve it via a simple HTTP server to avoid CORS issues:
```bash
cd AICTE-Approval-System-frontend
python -m http.server 3000
```

Then navigate to `http://localhost:3000`.

**2. Ensure the API base URL in `index.html` points to your backend:**

The frontend uses `http://localhost:8000` as the default API base. Update this constant if deploying the backend to a remote server.

---

## Usage

**Institution Portal:**

1. Navigate to the landing page and click **Institution Portal → Register**.
2. Complete the registration form with institution details and credentials; click **Register**.
3. Log in with your registered email and password.
4. From the **Upload Disclosures** section, upload PDFs one section at a time (Faculty, Labs, Infrastructure, Students, Financials, Accreditation). Each upload triggers AI extraction and risk recalculation.
5. Navigate to **AI Risk Analysis** to view the computed risk score, per-section breakdown, risk factors, and improvement suggestions.
6. When ready, click **Submit for Approval** to send the application to the AICTE reviewer queue.
7. Monitor approval status under **Approval Status** and check **Notifications** for reviewer feedback.
8. Use **Download Excel Report** to export a comprehensive compliance report in `.xlsx` format.

**AICTE Authority Portal:**

1. Click **Authority Login** on the landing page.
2. Use authority credentials:
   - `reviewer@aicte-india.org` / `demo1234`
   - `authority@aicte.in` / `admin1234`
3. Review pending institution submissions from the **Pending Approvals** queue.
4. Open any institution's review panel to inspect per-section data, AI risk analysis, and uploaded disclosures.
5. Assign per-section decisions and submit an overall approval or rejection with notes.

---

## Future Enhancements

- **JWT / Token-Based Authentication** — Replace session-based institution identification with proper Bearer token authentication for stateless, scalable API security.
- **Production Database Migration** — Migrate from SQLite3 to PostgreSQL for concurrent write support and production-grade reliability.
- **OCR for Scanned PDFs** — Integrate Tesseract OCR as a fallback for disclosure PDFs that are scanned images rather than text-layer PDFs, expanding compatibility with legacy document submissions.
- **Document Authenticity Verification** — Implement digital signature validation and hash-based document integrity checks to detect tampered or forged disclosure PDFs.
- **Automated Email Notifications** — Integrate Django's email backend (SMTP / SendGrid) to dispatch approval decisions, deadline reminders, and resubmission alerts directly to institution emails.
- **Multi-Year Trend Analysis** — Extend the data model to support multi-year disclosure submissions and surface compliance trend charts showing improvement or deterioration across academic years.
- **NAAC / NBA API Integration** — Connect to official NAAC and NBA accreditation databases to auto-validate accreditation claims against live records, eliminating manual cross-checking.
- **Role-Based Access Control** — Implement fine-grained RBAC with multiple authority roles (national reviewer, state reviewer, regional director) and institution sub-user accounts.
- **Mobile-Responsive Frontend** — Refactor the CSS layout to be fully responsive for tablets and mobile devices used by field officers during site inspections.
- **Bulk Institution Import** — Provide a CSV/Excel bulk upload endpoint allowing AICTE administrators to onboard hundreds of institutions simultaneously.
- **Audit Log** — Maintain a tamper-evident audit log recording every status change, review decision, and data modification for regulatory accountability.

---

## Contributing

Contributions are welcome. Please follow the standard Git workflow below.

**1. Fork the repository:**
```bash
git clone https://github.com/harisaigithub/aicte-approval-system.git
```

**2. Create a feature branch:**
```bash
git checkout -b feature/your-feature-name
```

**3. Make your changes and commit:**
```bash
git add .
git commit -m "feat: describe your change clearly"
```

**4. Push to your fork:**
```bash
git push origin feature/your-feature-name
```

**5. Open a Pull Request** against the `main` branch with a clear description of the change, the problem it solves, and any relevant test observations.

Please follow PEP 8 for Python code. Ensure that existing API endpoints and ML model inference remain unaffected by new changes.

---

## License

```
MIT License

Copyright (c) 2026 Harisai Parasa

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
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN
THE SOFTWARE.
```

---

## Contact

&nbsp;

## ***To acquire this project, please contact***

**GitHub:** https://github.com/harisaigithub
**Email:** [harisaiparasa@gmail.com](mailto:harisaiparasa@gmail.com)

For collaborations, customizations, or deployment support, feel free to reach out.