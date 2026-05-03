# Smart Health Surveillance and Early Warning System

> **Predictive Community Health Monitoring with Machine Learning — Detecting Outbreak Risks Before They Escalate**

[![Python](https://img.shields.io/badge/Python-3.11+-blue.svg)](https://www.python.org/)
[![Django](https://img.shields.io/badge/Django-5.2-green.svg)](https://www.djangoproject.com/)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![SIH Compliant](https://img.shields.io/badge/SIH25001-Compliant-teal.svg)]()

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

Rural and semi-urban communities in India face a persistent challenge: disease outbreaks are often detected only after they have already spread through multiple households or villages. By the time administrators receive reports, the window for early intervention has already closed. Conventional paper-based reporting mechanisms suffer from delayed data aggregation, lack of environmental context, and no predictive capability.

The **Smart Health Surveillance and Early Warning System** addresses this gap by providing a structured, multi-role digital platform that collects granular health and sanitation data from citizens and ASHA/health workers, runs a lightweight Machine Learning risk engine on this real-time data, and automatically triggers area-level early warning alerts when abnormal patterns are detected in a village.

Built as a B.Tech final-year capstone project aligned with the Smart India Hackathon (SIH25001) problem statement, the system closes the loop between community-level complaint submission and administrative decision-making — enabling proactive outbreak prevention rather than reactive crisis response.

---

## Objectives

- Design and implement a three-tier role architecture (Citizen, Health Worker, Administrator) with OTP-verified email-based authentication.
- Enable citizens to report health symptoms and environmental hazards through structured digital forms, with optional photographic evidence.
- Enable ASHA/Health Workers to submit community-level field reports linked to specific citizen complaints.
- Implement a logistic-regression-inspired ML risk engine that classifies villages into Low, Medium, and High risk levels based on complaint frequency, symptom burden, and environmental indicators.
- Compute individual personal risk scores for each citizen submission to provide immediate health guidance.
- Integrate Google Gemini AI to generate contextual, symptom-specific medical suggestions and medication guidance.
- Automate the early warning notification pipeline such that medium and high-risk determinations trigger alerts to both administrators and affected village citizens.
- Provide an administrative dashboard with real-time risk analytics, Chart.js visualizations, and exportable reports (CSV and print-optimized PDF).
- Support a hierarchical geographic data model (Country → State → District → Mandal → Village) imported from official LGD (Local Government Directory) sources.
- Maintain a complete complaint lifecycle management system with status tracking, field visit scheduling, auto-escalation, and resolution workflows.

---

## Summary

The Smart Health Surveillance and Early Warning System is a full-stack Django web application structured as six modular Django apps: `accounts`, `locations`, `complaints`, `analytics`, `dashboard`, and `notifications`. Each module handles a discrete domain of responsibility.

At the core of the system is the **ML Risk Engine** (`analytics/logic.py`), which operates on a rolling 7-day window of complaint data per active village. It extracts four normalized feature rates — symptom frequency, environmental sanitation score, cluster illness rate, and report pressure — and passes them through a logistic-style sigmoid function to compute a probability score between 0.0 and 1.0. Villages scoring above 0.72 are classified as High Risk; those between 0.45 and 0.72 as Medium Risk; and the remainder as Low Risk. This computation triggers automatic notification creation through the notifications subsystem.

The complaint pipeline supports two submission paths: citizen reports (symptom checkboxes and environmental indicators) and health worker field reports (community impact metrics, field visit status, escalation triggers). Each citizen submission also generates a **personal risk score** using a rule-based weighted formula and, when Gemini AI is configured, an AI-generated health suggestion and medication guidance tailored to the reported symptoms and environmental context.

Administrators access a centralized dashboard with live summary statistics, Chart.js-powered pie and bar charts of risk distribution and area-wise probability scores, a high-risk village alert table, and tools to manually retrigger ML recalculation, export complaint data as CSV, and print PDF risk assessment reports.

---

## Features

### Before Login (Public-Facing)

- **Landing Page** — A responsive hero section introducing the platform's purpose, core capabilities, and call-to-action for complaint submission and user registration.
- **User Registration** — Role-based signup (Citizen / Health Worker) with village-level geographic linkage through a cascading Country → District → Mandal → Village dropdown.
- **OTP Email Verification** — Account activation requires verification of a 6-digit, time-limited OTP sent to the registered email address via SMTP.
- **Secure Login** — Email-based authentication (username field replaced) with login count tracking and role-specific redirect after successful authentication.
- **Password Reset Flow** — Full Django-native password reset pipeline with email delivery of reset links.

### After Login — Citizen

- **Symptom Reporting Form** — Checkboxes for ten clinical symptoms (fever, diarrhea, vomiting, headache, body pain, cough, cold, skin rash, fatigue, breathing difficulty) and eight environmental indicators (stagnant water, waste issues, water contamination, drainage, garbage dumping, foul smell, unsafe drinking water, nearby illness cases).
- **Photo Evidence Upload** — Optional image attachment to complaints for visual documentation of environmental hazards.
- **Personal Risk Score** — Immediate post-submission feedback showing a computed risk score (0–100), a risk level badge (Low / Medium / High), and actionable health guidance.
- **AI Health Suggestions** — When configured with a Gemini API key, the system generates a contextual health suggestion and medication guidance based on the specific symptoms and environment reported.
- **Submission History** — A paginated view of all previous complaints with their current status, timestamps, and ML-derived personal risk.
- **Village Risk Status** — Citizen dashboard displays the latest ML-computed risk level for their registered village with historical context.
- **Alert Notifications** — Citizens receive system-generated notifications for complaint resolution, escalation, and area-level risk alerts.

### After Login — Health Worker (ASHA/Officer)

- **Community Field Report** — Structured report form capturing affected population count, vector severity (mosquito), environmental conditions, field visit status, camp recommendation, and detailed verification notes.
- **Complaint Linkage** — Field reports can be linked to specific citizen complaints submitted in the same village, enabling traceability between citizen-reported symptoms and official health worker verification.
- **Field Visit Tracking** — Dashboard displays assigned complaints with field visit due dates, overdue indicators, and completion status.
- **Auto-Escalation Logic** — Completing a field visit automatically evaluates escalation criteria (affected count ≥ 10, water contamination, camp recommended, high mosquito severity, nearby illness clusters) and escalates the complaint to PHC / Medical Officer level if thresholds are met, or resolves it otherwise.
- **Assignment Notifications** — Health workers receive targeted notifications when admin assigns them to a complaint, including field visit deadlines and actionable instructions.
- **Printable Field Report** — Health workers can generate a structured print report for physical documentation.

### After Login — Administrator

- **Master Dashboard** — Real-time summary tiles showing total complaints, total registered users, active high-risk village count, and total monitored villages.
- **Risk Visualization** — Chart.js-powered pie chart (risk level distribution) and bar chart (top villages by probability score).
- **High-Risk Alert Table** — Tabulated view of all villages currently in High Risk status with probability scores and last calculated timestamps.
- **Risk History** — Chronological log of all risk classification events per village with trend information.
- **Complaint Management** — Full list of all complaints with filtering by status and the ability to perform admin actions (assign health worker, change status, set priority, schedule field visits, add admin notes, escalate with target designation).
- **ML Engine Trigger** — Manual button to rerun `run_risk_analysis()` across all active villages on demand.
- **CSV Export** — Downloads all complaint data as a structured CSV aligned with feature set definitions.
- **PDF Risk Report** — Print-optimized HTML report of all current risk records suitable for PDF saving via browser print.
- **Notification Management** — View and resolve all active system-wide risk alerts and workflow notifications.

---

## Technologies Used

### Programming Languages
- **Python 3.11+** — Primary backend language.
- **HTML5 / CSS3 / JavaScript** — Frontend templates and interactive UI.

### Libraries & Frameworks
- **Django 5.2** — Core web framework (MVT architecture, ORM, sessions, auth, email).
- **django-crispy-forms 2.5** with **crispy-bootstrap5** — Form rendering and layout.
- **Tailwind CSS (CDN)** — Utility-first CSS framework for frontend styling.
- **Chart.js (CDN)** — JavaScript charting library for pie and bar visualizations.
- **Font Awesome (CDN)** — Icon library used across templates.
- **NumPy** — Sigmoid function computation and numerical operations in the ML engine.
- **Pillow 12.1.0** — Image processing for profile pictures and complaint photo uploads.
- **ReportLab 4.4.9** — PDF generation support.

### Machine Learning / AI
- **Custom Logistic-Regression Risk Engine** — Implemented from scratch using NumPy; computes a village-level probability score from four normalized feature rates using a sigmoid activation function with manually calibrated weights.
- **Personal Risk Scoring** — Rule-based weighted formula for individual symptom and environment burden assessment.
- **Google Gemini AI (google-generativeai 0.8.5 / google-genai 1.61.0)** — Generative AI integration for symptom-specific health suggestions and medication guidance per complaint submission.

### Backend
- **Django ORM** — Database abstraction for all models including custom user management.
- **Django Custom User Model** — Email-based authentication replacing the default username field.
- **Django Management Commands** — Custom commands for LGD data import, demo data setup, and fallback village creation.
- **SMTP (Gmail)** — Email backend for OTP delivery, password resets, and system notifications.
- **python-dotenv 1.2.1** — Environment variable management for API keys and secrets.

### Frontend
- **Django Template Language** — Server-side HTML rendering with template inheritance (base1.html / base2.html).
- **Tailwind CSS** — Responsive layout, color theming (Primary: `#0F766E` Deep Teal, Accent: `#22D3EE` Soft Cyan), and utility classes.
- **JavaScript (Vanilla)** — Cascading dropdown population for geographic selection, Chart.js initialization, and dynamic UI interactions.

### Database
- **SQLite3** — Default database backend; file-based relational database suitable for development and small-scale deployment.

### Deployment & DevOps
- **Django WSGI / ASGI** — Both WSGI (`config/wsgi.py`) and ASGI (`config/asgi.py`) configurations provided.
- **Django staticfiles** — Static asset collection for production deployment.

### Security
- **Django CSRF Middleware** — Cross-site request forgery protection on all POST endpoints.
- **Django Password Validators** — Enforces minimum length, similarity, and common password restrictions.
- **Email OTP Verification** — Prevents account activation without email ownership confirmation; OTP expires after 10 minutes.
- **Role-based Access Control** — `@user_passes_test` decorators enforce role boundaries across all views.
- **Environment-based Secret Management** — API keys and credentials externalized via `.env`.

---

## Dataset

### Source
The geographic dataset is sourced from the **Government of India's Local Government Directory (LGD)** — the official digital repository of administrative units. The system includes management commands to import this data from XLS format (`import_ap_lgd_xls.py`) and from a custom CSV master file (`import_location_master.py`).

A sample location CSV (`locations/location_master_sample.csv`) is included in the repository for testing and demonstration purposes.

### Structure
The hierarchical location model follows the official administrative hierarchy:

```
Country → State → District → Mandal → Village
```

Each `Village` record carries an `is_active` flag that controls inclusion in ML analysis and user registration dropdowns.

### Complaint Data (Runtime Generated)
The complaint dataset is entirely user-generated at runtime. Each complaint record captures:
- **10 symptom binary fields** (fever, diarrhea, vomiting, headache, body pain, cough, cold, skin rash, fatigue, breathing difficulty).
- **8 environmental binary fields** (stagnant water, waste issue, water contamination, drainage issue, garbage dumping, foul smell, unsafe drinking water, nearby illness cases).
- **Mosquito severity** (categorical: low / medium / high).
- **Affected count** (integer, health worker submissions).
- **Report source, village linkage, timestamps, status, and AI-generated text fields.**

### Preprocessing
The ML engine performs feature extraction inline per village on a rolling 7-day window. Raw boolean fields are aggregated into normalized rates (value between 0 and 1) before being passed to the risk model. No separate preprocessing pipeline or offline training is required — the model coefficients are statically calibrated and the computation executes in real time on each trigger.

---

## System Architecture

The system is organized as a monolithic Django project (`config/`) containing six independent Django applications, each responsible for a specific domain.

```
Smart-Health-Surveillance/
├── config/                  # Project settings, URL root, WSGI/ASGI
├── accounts/                # Custom user model, auth, OTP, roles, profiles
├── locations/               # Geographic hierarchy (Country→Village), LGD import
├── complaints/              # Complaint submission, forms, risk_utils, workflow
├── analytics/               # ML risk engine (logic.py), RiskRecord model
├── dashboard/               # Role-specific dashboard views and Chart.js rendering
├── notifications/           # Alert model, workflow notifications, context processor
├── templates/               # Global Jinja2-style Django templates per app
├── static/                  # Static assets (images, CSS base)
├── manage.py
└── req.txt                  # Pinned dependency list
```

### Data Flow

1. **Citizen / Health Worker** submits a complaint via a role-specific form.
2. The `complaints` app validates the form, saves the `Complaint` instance, computes a **personal risk score** via `risk_utils.compute_personal_risk()`, and optionally calls the **Gemini AI API** to generate health suggestions.
3. The `analytics` app's `run_risk_analysis()` is called (either manually by admin or post-submission) for the relevant village.
4. The ML engine queries all complaints from the last 7 days for each active village, extracts feature rates, computes the sigmoid probability score, classifies the risk level, and saves a `RiskRecord`.
5. If the risk level is Medium or High, the `notifications` app creates or updates a `Notification` with a cooldown and deduplication mechanism.
6. The `dashboard` app surfaces `RiskRecord` data through role-specific views with Chart.js visualizations.
7. Citizens and Health Workers see their relevant notifications via the `notifications.context_processors.alert_notifications` global context processor.

---

## Model Workflow

### Step 1 — Data Collection (Preprocessing)
Raw complaint submissions are stored as boolean flags and categorical values in the `Complaint` model. For each village, the engine filters to the rolling 7-day complaint window and counts occurrences of each indicator field.

### Step 2 — Feature Extraction
Four normalized feature rates are computed per village:

| Feature | Formula |
|---|---|
| `symptom_rate` | (fever + diarrhea + vomiting counts) / (total_reports × 3) |
| `sanitation_rate` | (stagnant water + waste + contamination counts) / (total_reports × 3) |
| `cluster_rate` | nearby_illness_cases count / total_reports |
| `report_pressure` | min(total_reports / 12.0, 1.0) |

### Step 3 — Logistic Risk Scoring
A logistic-style linear combination of the four features is computed:

```
z = -2.2 + (1.2 × report_pressure) + (2.0 × symptom_rate)
         + (1.8 × sanitation_rate) + (0.6 × cluster_rate)

probability = 1 / (1 + e^(-z))
```

The intercept of `-2.2` creates a conservative baseline — villages require meaningful complaint activity to register as medium or high risk, preventing false positives from isolated reports.

### Step 4 — Risk Classification
The probability score is mapped to a risk level:

| Probability Range | Risk Level | Action Triggered |
|---|---|---|
| ≥ 0.72 | High | Critical early warning alert created |
| 0.45 – 0.71 | Medium | Warning alert created |
| < 0.45 | Low | Any active alert resolved |

### Step 5 — Alert Generation
The `_upsert_risk_alert()` function enforces a single active alert per village. A cooldown mechanism prevents re-opening an alert at the same or lower risk level than the most recently resolved one, reducing notification fatigue. Escalation (e.g., from medium to high) always creates a new alert.

### Step 6 — Personal Risk Scoring (Per Submission)
Simultaneously, each citizen complaint generates a personal risk score (0–100) using a weighted sum of symptom severity and environmental exposure indicators, capped and mapped to Low / Medium / High levels with role-specific guidance text.

### Step 7 — Integration (AI Suggestions)
When `GEMINI_API_KEY` is present in the environment, the system constructs a structured prompt from the complaint's symptom and environment labels and queries the Gemini API for a contextual health suggestion and medication guidance, which are stored against the complaint record and displayed to the submitting user.

---

## Implementation Details

### Custom User Model (`accounts/models.py`)
The default Django `User` is replaced with a custom `AbstractUser` subclass that uses `email` as the `USERNAME_FIELD`. Three roles are defined as string constants: `admin`, `health_worker`, and `citizen`. A `village` ForeignKey links each user to their operational geographic unit, which drives both role-based data filtering and ML analysis scoping.

### OTP Authentication (`accounts/views.py`)
After registration, a 6-digit random OTP is generated, stored in `EmailOTP` with a creation timestamp, and delivered via SMTP. The `is_expired()` method on `EmailOTP` enforces a 10-minute validity window. Accounts are activated only upon successful OTP verification.

### Geographic Cascade (`locations/`)
A JavaScript event listener on the district dropdown triggers an AJAX-style fetch to `locations/api/mandals/<district_id>/` and `locations/api/villages/<mandal_id>/`, dynamically populating the downstream dropdowns. This ensures users are always registered to a valid, existing village in the LGD hierarchy.

### Complaint Workflow Engine (`complaints/workflow.py`)
The `ensure_complaint_workflow_fields()` function is called on every complaint save to automatically set `assigned_at`, compute `field_visit_due_at` (48 hours from assignment), update status from `submitted` to `assigned` when a health worker is assigned with a field visit requirement, and synchronize `resolved_at` with resolution status. This prevents orphaned or inconsistent workflow states.

### Auto-Escalation (`complaints/workflow.py`)
When a health worker marks a field visit as completed (`apply_health_worker_follow_up()`), the system evaluates six escalation criteria against the follow-up report. If any threshold is exceeded, the parent complaint is moved to `escalated` status, an escalation target is recorded, and an escalation notification is dispatched to the original reporting citizen.

### Admin Dashboard Data (`dashboard/views.py`)
The dashboard uses a subquery-based approach to retrieve exactly one latest `RiskRecord` per village — avoiding duplicate aggregation artifacts that arise from simple `order_by` + `distinct()` on related tables.

### Notification Deduplication (`analytics/logic.py`)
The `_upsert_risk_alert()` function queries for any existing active risk alert for a given village. If found, it updates the record in place (preventing duplicates). If not found, it applies a cooldown check against the most recently resolved alert. New alerts are created only if the new risk level represents an escalation over the previously resolved level.

### Gemini AI Integration (`complaints/views.py`)
If `GEMINI_API_KEY` is set, the system calls `genai.GenerativeModel('gemini-pro').generate_content()` with a structured prompt derived from the complaint's active symptom and environment labels. Fallback static advisory functions (`_fallback_citizen_advice()`, `_fallback_health_worker_advice()`) are used when the API is unavailable or not configured, ensuring the feature degrades gracefully.

---

## Results

### ML Risk Engine

The logistic-regression-inspired risk engine is designed for a regime where training data is scarce (newly deployed system) and correctness of risk classification direction is prioritized over probabilistic calibration. Key performance characteristics:

| Metric | Value |
|---|---|
| Risk Classification Levels | 3 (Low, Medium, High) |
| Feature Count | 4 normalized rates per village |
| Analysis Window | Rolling 7 days |
| High-Risk Threshold | Probability ≥ 0.72 |
| Medium-Risk Threshold | Probability ≥ 0.45 |
| Baseline Intercept | –2.2 (conservative; avoids false positives) |
| Report Saturation Point | 12 reports/week per village |

The intercept and weight values are manually calibrated to ensure that:
- A single isolated report never triggers a medium/high alert.
- A village with 10+ reports in a week with co-occurring symptoms and sanitation issues reliably classifies as High Risk.
- The sigmoid function prevents saturation at extreme ends, preserving meaningful score gradation.

### Personal Risk Scoring

The personal risk scorer assigns weighted scores to symptom severity and environmental exposure:

| Category | Score Contribution |
|---|---|
| Breathing difficulty | +24.0 |
| Each symptom (capped) | +6.0 per symptom, max 36.0 |
| Fever | +8.0 |
| Diarrhea / Vomiting | +7.0 each |
| Unsafe water / Contamination | +8.0 each |
| Nearby illness cases | +6.0 |
| Stagnant water / Garbage | +4.0 |

Scores map to: High Risk (≥ 70), Medium Risk (≥ 40), Low Risk (< 40).

### System Functional Coverage

| Module | Status |
|---|---|
| Role-based Authentication with OTP | Fully implemented |
| Citizen complaint submission with AI | Fully implemented |
| Health worker field reporting | Fully implemented |
| ML risk engine with auto-alerting | Fully implemented |
| Admin dashboard with charts | Fully implemented |
| CSV and PDF export | Fully implemented |
| Password reset flow | Fully implemented |
| Geographic hierarchy import (LGD) | Fully implemented |

---

## Installation

### Prerequisites

- Python 3.11 or higher
- pip (Python package manager)
- Git

### Step 1 — Clone the Repository

```bash
git clone https://github.com/harisaigithub/Smart-Health-Surveillance-and-Early-Warning-System.git
cd Smart-Health-Surveillance-and-Early-Warning-System
```

### Step 2 — Create and Activate a Virtual Environment

```bash
python -m venv venv

# On Windows
venv\Scripts\activate

# On macOS/Linux
source venv/bin/activate
```

### Step 3 — Install Dependencies

```bash
pip install -r req.txt
```

### Step 4 — Configure Environment Variables

Create a `.env` file in the project root:

```env
GEMINI_API_KEY=your_google_gemini_api_key_here
```

The email backend is pre-configured in `config/settings.py` using Gmail SMTP. Replace `EMAIL_HOST_USER` and `EMAIL_HOST_PASSWORD` (Gmail App Password) with your own credentials before running the server.

### Step 5 — Apply Database Migrations

```bash
python manage.py migrate
```

### Step 6 — Import Geographic Data

Use the sample location CSV included in the repository:

```bash
python manage.py import_location_master
```

Alternatively, import from LGD XLS format:

```bash
python manage.py import_ap_lgd_xls
```

To add fallback villages for demo use:

```bash
python manage.py add_fallback_villages
```

### Step 7 — Set Up Demo Data (Optional)

```bash
python manage.py setup_demo_data
```

### Step 8 — Create an Administrator Account

```bash
python manage.py createsuperuser
```

### Step 9 — Collect Static Files (Production)

```bash
python manage.py collectstatic
```

### Step 10 — Start the Development Server

```bash
python manage.py runserver
```

Access the application at `http://127.0.0.1:8000/`.

---

## Usage

### Citizen Workflow

1. Navigate to the landing page and click **Report Complaint**.
2. Complete registration by selecting your Country, District, Mandal, and Village. Choose the **Citizen** role.
3. Verify your email address using the OTP sent to your registered email.
4. Log in and access your **Citizen Dashboard** to view your village's current risk level.
5. Click **Submit Report** to open the symptom and environmental indicator form.
6. Check all applicable symptoms and environmental conditions, optionally attach a photo, and submit.
7. Review your **personal risk score**, AI health suggestions, and medication guidance on the confirmation page.
8. Track the status of your submission from the **My Submissions** view.

### Health Worker Workflow

1. Register using the **Health Worker** role and link to your operational village.
2. After OTP verification and login, access the **Health Worker Dashboard**.
3. View assigned complaints with field visit due dates and overdue indicators.
4. Submit community field reports by filling in the affected population count, environmental conditions, and verification notes. Optionally link to a related citizen complaint.
5. Mark field visits as complete; the system will auto-escalate or auto-resolve the parent complaint based on configured thresholds.
6. Monitor your notification inbox for new assignments from administrators.

### Administrator Workflow

1. Log in using an admin-role account or Django superuser credentials.
2. Access the **Admin Dashboard** to view summary cards, risk pie charts, and the high-risk village alert table.
3. Navigate to **All Complaints** to review incoming reports, apply admin actions, assign health workers, set priorities, and log escalation targets.
4. Click **Run ML Analysis** to retrigger the risk engine and refresh all village risk classifications.
5. Use **Export CSV** to download the full complaint dataset for external analysis.
6. Use **Print Risk Report** to generate a print-optimized PDF of all current risk records.
7. Monitor and resolve active risk alerts from the **Alerts** panel.

---

## Future Enhancements

- **Scheduled ML Recalculation** — Integrate Django-Q or Celery with a periodic task scheduler to run `run_risk_analysis()` automatically every 6 or 24 hours, eliminating dependency on manual admin triggers.
- **SMS/WhatsApp Alerts** — Extend the notification pipeline with Twilio or MSG91 integration to deliver early warning alerts to citizens without internet access via SMS.
- **Expanded Feature Set** — Incorporate seasonal data (monsoon proximity, humidity, temperature from a weather API) as additional features in the risk model to improve accuracy during outbreak-prone periods.
- **Trained ML Model with Historical Data** — Replace the statically calibrated coefficient weights with a scikit-learn LogisticRegression model trained on historical epidemic data from IDSP (Integrated Disease Surveillance Programme), with periodic retraining support.
- **GIS Map Integration** — Display village risk levels on an interactive Leaflet.js or Google Maps choropleth map, enabling spatial outbreak pattern recognition.
- **Mobile Application** — Develop a React Native or Flutter mobile app for citizens and health workers operating in low-connectivity field environments, with offline complaint drafting and background sync.
- **Multi-Language Support** — Add Django i18n translations for Telugu, Hindi, and other regional languages to improve accessibility for non-English-speaking rural users.
- **REST API Layer** — Implement Django REST Framework endpoints to allow external health information systems (HMIS, NHP portals) to consume risk data and complaint statistics programmatically.
- **Audit Logging** — Introduce a dedicated audit log for all admin actions on complaints, providing a tamper-evident compliance trail for official health records.
- **PostgreSQL Migration** — Transition from SQLite3 to PostgreSQL for production deployments requiring concurrent write access and advanced query optimization.

---

## Contributing

Contributions are welcome. Please follow the standard Git workflow outlined below.

1. Fork the repository to your GitHub account.

2. Clone your fork locally:
   ```bash
   git clone https://github.com/your-username/Smart-Health-Surveillance-and-Early-Warning-System.git
   ```

3. Create a feature branch from `main`:
   ```bash
   git checkout -b feature/your-feature-name
   ```

4. Make your changes, ensuring code quality and consistency with the existing module structure.

5. Commit with a clear, descriptive message:
   ```bash
   git commit -m "feat: add scheduled ML recalculation via Celery"
   ```

6. Push your branch to your fork:
   ```bash
   git push origin feature/your-feature-name
   ```

7. Open a Pull Request against the `main` branch of the original repository. Describe the changes made and the problem solved.

Please ensure all new views are protected with appropriate `@login_required` and `@user_passes_test` decorators consistent with the three-role access model.

---

## License

This project is licensed under the **MIT License**.

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

For questions, issues, or contributions related to the project, please open a GitHub Issue or reach out directly.

**GitHub:** [harisaigithub](https://github.com/harisaigithub)

**Email:** harisaiparasa@gmail.com

---


&nbsp;

&nbsp;

## ***To acquire this project, please contact***

**GitHub:** https://github.com/harisaigithub  
**Email:** [harisaiparasa@gmail.com](mailto:harisaiparasa@gmail.com)  

For collaborations, customizations, or deployment support, feel free to reach out.