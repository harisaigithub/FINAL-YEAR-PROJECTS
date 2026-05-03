# Trace AI — Finding Missing Person Intelligence Platform

> **An AI-powered, full-stack web platform for registering, managing, and forensically analyzing missing person cases using Google Gemini multimodal intelligence and OpenCV face detection.**

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

Missing person cases demand rapid, organized, and intelligent action. Traditional reporting systems are fragmented, slow, and incapable of leveraging modern AI to assist investigators. **Trace AI** addresses this critical gap by providing a centralized, role-based web platform where citizens can report missing individuals, administrators can manage and prioritize cases, and an embedded AI engine — powered by Google Gemini 1.5 Flash — generates forensic intelligence briefs from photographs and case metadata.

The platform combines **generative AI multimodal analysis**, **local machine learning face detection via OpenCV**, and a structured **Django-based case lifecycle management system** to deliver a production-grade investigative tool. It is designed with three distinct user tiers: public visitors, registered reporters, and staff administrators — each with a tailored interface and access scope.

This system is particularly significant in humanitarian contexts where speed of dissemination and accuracy of forensic description can directly impact recovery outcomes.

---

## Objectives

- Design a secure, role-based web application that manages the full lifecycle of a missing person case — from submission and approval to AI analysis and case closure.
- Integrate Google Gemini 1.5 Flash as a multimodal AI engine to extract structured forensic markers (skin tone, hair, clothing, scars) and generate investigative search strategies from uploaded photographs and case descriptions.
- Implement OpenCV Haar Cascade face detection as a local pre-screening layer to validate photograph quality before triggering cloud AI inference.
- Build a community sighting intelligence system, enabling anonymous tip submission with unique traceable report IDs.
- Develop separate administrative and user dashboards with real-time case statistics, pending approvals, and archived resolution logs.
- Enforce secure authentication through OTP-based email verification, session-managed login, and Django's built-in password reset flow.
- Deploy the system using Gunicorn and WhiteNoise for production-ready static file serving.

---

## Summary

Trace AI is a Django-based, AI-integrated web platform structured around six core application modules: `accounts`, `cases`, `ai_engine`, `reports`, `portal`, and `dashboard`. Each module handles a distinct domain of the investigation pipeline.

When a user registers, their account undergoes email OTP verification before activation. Authenticated users can submit missing person case files containing full biographical details, last-seen location, date, physical description, and a photograph. All submissions enter a **Pending** queue, where staff administrators review and either approve or reject each case.

Upon approval, cases become publicly visible on the platform's landing and search pages. Administrators can then trigger the **AI Intelligence Engine**, which performs a two-stage analysis: first, a local OpenCV Haar Cascade classifier validates face presence in the photograph; second, the case metadata and image are sent to the Gemini 1.5 Flash multimodal model via a structured forensic prompt. The model returns a raw JSON payload containing biometric markers and a tailored investigative strategy, which is persisted in the `AIAnalysis` model and displayed in the case intelligence brief.

Community members may submit sighting reports for any active case, generating a unique `TRC-XXXXXX` tracking ID. Administrators can verify or purge these reports and ultimately close cases by marking subjects as found, archiving them in the resolved case registry.

---

## Features

### Public (Before Login)

- **Landing Page with Live Cases** — Displays the four most recently approved missing person cases, including photographs and key details.
- **Public Case Directory** — Searchable and filterable list of all publicly approved cases by name and last-seen location.
- **Public Case Detail View** — Full profile page for each approved case with photograph, biography, and physical description.
- **Anonymous Sighting Reports** — Any visitor can submit an intelligence tip for an active case with location, timestamp, description, and an optional sighting photograph. A unique `TRC-XXXXXX` report ID is generated for tracking.

### Registered Users (After Login)

- **OTP-Verified Registration** — Email-based one-time password verification with a 10-minute expiry window before account activation.
- **Case Submission Portal** — Structured form for submitting new missing person case files with full personal details and photograph upload.
- **User Dashboard** — Personalized mission control displaying total cases, pending approvals, approved cases, and a recent case activity feed.
- **My Records** — Complete personal case log with status indicators and the ability to request priority admin review.
- **Case Update Requests** — Users can flag a specific case for administrative re-evaluation, triggering a priority notification.
- **Profile Management** — Users can update their display name and view account metadata.
- **Password Reset Flow** — Secure multi-step password recovery via email link.

### Staff / Administrator

- **Admin Dashboard** — Global intelligence overview showing total active investigations, pending approval count, live approved cases, and total resolved cases.
- **Pending Approval Queue** — Dedicated list of all unreviewed submissions with one-click Approve and Reject actions.
- **AI Intelligence Engine** — Per-case trigger for the Gemini multimodal forensic analysis pipeline, producing a structured intelligence brief with biometric markers and search strategy.
- **Intelligence Overview** — Registry of all processed AI analyses sorted by recency.
- **Sighting Report Management** — Full list of all community-submitted sighting intelligence with options to verify credibility or purge false reports.
- **Case Closure and Archiving** — Admin action to mark a case as Closed/Found, moving it to the resolved archive.
- **Archived Case Dossiers** — Detailed forensic dossiers for closed cases, including the full AI brief, linked sightings, and case history.
- **Notification System** — In-platform notification model tied to user accounts for status updates.

---

## Technologies Used

### Programming Languages
- Python 3.11.9

### Libraries & Frameworks
- Django 4.x — Core web framework, ORM, authentication, admin panel
- Pillow — Image field processing and photograph handling
- python-dotenv — Environment variable management via `.env` files
- WhiteNoise — Static file serving in production without a separate CDN
- Gunicorn — WSGI production server

### Machine Learning / AI
- **Google Gemini 1.5 Flash** (`google-generativeai`) — Multimodal generative AI for forensic marker extraction and investigative strategy generation via structured JSON prompts
- **OpenCV** (`opencv-python-headless`) — Local Haar Cascade face detection (`haarcascade_frontalface_default.xml`) for image pre-screening and face presence validation

### Backend
- Django ORM with SQLite (development) — Relational data management for cases, users, analyses, and reports
- Django's built-in authentication system extended with `AbstractUser` and custom `CustomUserManager`
- Django Email Backend — OTP delivery and password reset via SMTP

### Frontend
- Django Template Language — Server-side HTML rendering
- Tailwind CSS (utility classes in templates) — Responsive, utility-first UI styling
- HTML5 / CSS3 — Markup and styling

### Database
- SQLite 3 — Default development database (easily portable to PostgreSQL for production)

### Deployment & DevOps
- Gunicorn — Production-grade WSGI application server
- WhiteNoise — Zero-configuration static file serving middleware
- Python Virtual Environment (`venv`) — Dependency isolation

### Security
- Django CSRF middleware — Cross-site request forgery protection on all state-changing endpoints
- Email OTP verification — Account activation gating before user login
- `@login_required` and `@staff_member_required` decorators — Role-based route protection
- Session-based authentication — Secure user session management
- Django password validators — Enforced password complexity rules
- `.env`-based secret management — API keys and credentials excluded from source code

---

## Dataset

This platform does not rely on a pre-existing static dataset. Case data is dynamically generated through user submissions and stored in a relational SQLite database. The following data entities are managed:

**MissingPersonCase** — Full name, age, gender, photograph (uploaded to `missing_persons/`), last-seen location and date, physical description, case status, and reporter reference.

**AIAnalysis** — Linked one-to-one to each case; stores the Gemini-generated investigative strategy (`gemini_insight`), extracted forensic marker JSON (`extracted_tags`), face detection boolean (`face_detected`), confidence score (integer percentage), processing timestamp, and processing status flag.

**SightingReport** — Linked to a specific case; stores location seen, date/time, description, an optional sighting image (`sightings/`), a unique `TRC-XXXXXX` tracking ID, verification status, and creation timestamp.

**Preprocessing** — Photograph images are read as raw bytes, encoded in Base64, and transmitted to the Gemini API as inline image data with `image/jpeg` MIME type. For OpenCV analysis, images are decoded via `np.frombuffer` and `cv2.imdecode`, then converted to grayscale before applying the Haar Cascade classifier.

---

## System Architecture

The application follows a monolithic Django project structure organized into six registered apps, each with isolated models, views, URLs, and templates.

```
core/                    # Django project root — settings, root URLconf, WSGI/ASGI
accounts/                # User model, OTP verification, login/signup/logout, password reset
cases/                   # MissingPersonCase model, submission, approval, archival
ai_engine/               # AIAnalysis model, Gemini integration, OpenCV face detection
reports/                 # SightingReport model, community tip submission and management
portal/                  # Public-facing views (landing, case list, case detail, AI API endpoint)
dashboard/               # Role-aware dashboards (user dashboard, admin dashboard, profile)
templates/               # Shared and per-app HTML templates
```

**Data Flow:**

1. A visitor or registered user accesses the platform via the `portal` app (landing page, public directory).
2. Authenticated users interact with the `cases` app to submit missing person files. Submissions are stored with `status='Pending'`.
3. Staff administrators log in and are routed by `dashboard_redirect` to the admin dashboard, where they manage the approval queue via the `cases` app.
4. Upon approving a case, the administrator can trigger the `ai_engine` analysis pipeline for that case.
5. The `ai_engine` first validates the photograph locally using OpenCV, then sends a multimodal forensic prompt with the photograph and case metadata to Gemini 1.5 Flash.
6. The AI response is parsed from JSON, and the resulting markers and strategy are persisted in the `AIAnalysis` model.
7. Community members can submit sighting reports via the `reports` app. Administrators manage these reports through the reports management panel.
8. Administrators close resolved cases via the `portal` or `reports` app, archiving them in the case resolution log.

---

## Model Workflow

### Step 1 — Preprocessing

When an AI analysis is triggered for a case, the photograph stored in the `MissingPersonCase.photograph` field is read from Django's media storage. The image bytes are loaded into a NumPy array and decoded by OpenCV's `cv2.imdecode`. The resulting image matrix is converted to grayscale using `cv2.cvtColor` for face detection.

For the Gemini API call, the photograph is independently read, encoded as a Base64 string, and formatted as an inline image data block with `image/jpeg` MIME type.

### Step 2 — Face Detection (Local ML)

OpenCV's pre-trained Haar Cascade classifier (`haarcascade_frontalface_default.xml`) is applied to the grayscale image using `detectMultiScale` with a scale factor of 1.1 and minimum neighbor count of 4. The result determines whether a face is present and how many faces are detected. This boolean result is stored as `AIAnalysis.face_detected` and influences the initial confidence score assignment.

### Step 3 — Generative AI Forensic Analysis

The Gemini 1.5 Flash model receives a structured system instruction defining its role as a Senior Forensic Digital Analyst. The instruction mandates extraction of five specific biometric markers — Skin Tone, Hair Style/Color, Eye Shape/Color, Clothing Details, and Scars/Marks — and an investigative search strategy, returned strictly as a raw JSON object with two keys: `markers` (dictionary) and `strategy` (string).

The user prompt appends the subject's full name, age, and physical description. The photograph is included as inline multimodal content. The response is cleaned of Markdown artifacts using regex substitution and parsed using `json.loads`.

### Step 4 — Persistence and Confidence Scoring

The parsed `strategy` string is stored in `AIAnalysis.gemini_insight`. The `markers` dictionary is stored in `AIAnalysis.extracted_tags` as a JSONField. A confidence score is computed: 96% if a face was detected, 68% otherwise. The `is_processed` flag is set to `True` and the analysis timestamp (`processed_at`) is auto-populated.

### Step 5 — Integration and Display

The populated `AIAnalysis` object is retrieved and rendered in the `ai_engine/analysis_result.html` template alongside the case details. The Intelligence Overview page aggregates all processed analyses. Closed case dossiers pull the linked `ai_analysis` via the `ai_analysis` reverse relation for the full forensic report.

---

## Implementation Details

**Custom User Model** — Django's `AbstractUser` is extended to replace the default `username` field with a unique `email` field. The custom `CustomUserManager` handles user and superuser creation. An `is_verified` boolean and `login_count` integer are tracked on each user record. An `EmailOTP` model stores a six-digit OTP with a 10-minute TTL, validated on the server before account activation.

**Case Lifecycle State Machine** — `MissingPersonCase.status` progresses through four defined states: `Pending`, `Approved`, `Rejected`, and `Closed`. Transitions are managed by staff-gated view functions with explicit ORM updates and user-facing messages. An `update_requested` boolean flag allows registered reporters to request re-evaluation by administrators.

**Sighting Report Tracking** — The `SightingReport` model generates a unique `TRC-XXXXXX` ID at object creation using a randomized alphanumeric generator. Reports are linked to cases via a `ForeignKey` with a `related_name='sightings'`, allowing case dossiers to aggregate all linked intelligence in a single query.

**Gemini Prompt Engineering** — The system instruction is structured with an explicit role definition, a targeted output format, and a strict no-Markdown, no-preamble rule to ensure parseable JSON output. The user prompt injects subject identity fields, and the multimodal content list appends the Base64-encoded image as a separate list element.

**Environment Configuration** — API keys, email credentials, and the Django `SECRET_KEY` are managed via a `.env` file loaded at application startup using `python-dotenv`. The `GEMINI_API_KEY` is read from the environment and passed to `genai.configure` during application initialization.

**Role-Based Routing** — Post-login redirection is handled by the `dashboard_redirect` view, which inspects `request.user.is_staff` and routes to the appropriate dashboard. All administrative endpoints are protected by Django's `@staff_member_required` decorator. All user-facing endpoints require `@login_required`.

---

## Results

The system demonstrates the following performance characteristics based on its design and AI integration:

| Metric | Value |
|---|---|
| AI Confidence Score (with detected face) | 96% |
| AI Confidence Score (without detected face) | 68% |
| Face Detection Method | OpenCV Haar Cascade (`haarcascade_frontalface_default.xml`) |
| AI Model | Google Gemini 1.5 Flash (Latest) |
| Forensic Markers Extracted | Skin Tone, Hair Style/Color, Eye Shape/Color, Clothing Details, Scars/Marks |
| Response Format | Structured JSON (`markers` dict + `strategy` string) |
| OTP Validity Window | 10 minutes |
| Sighting Report ID Format | `TRC-` + 6 alphanumeric characters (unique per report) |
| Case Status States | Pending → Approved / Rejected → Closed |

The dual-stage inference pipeline — local OpenCV pre-screening followed by Gemini multimodal generation — reduces unnecessary API calls for images lacking a detectable human face. The structured JSON prompt strategy ensures consistent, machine-parseable output from the generative model, enabling reliable extraction of forensic tags into the `extracted_tags` JSONField for downstream querying and display.

---

## Installation

### Prerequisites

- Python 3.11.9
- pip
- Git
- A valid Google Gemini API key (`google-generativeai`)
- An SMTP-capable email account (e.g., Gmail with App Password)

### Step 1 — Clone the Repository

```bash
git clone https://github.com/harisaigithub/FindingMissingPerson_Trace_AI.git
cd FindingMissingPerson_Trace_AI
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
pip install -r req.txt
```

The `req.txt` includes: `Django`, `gunicorn`, `whitenoise`, `Pillow`, `python-dotenv`, `google-generativeai`, `opencv-python-headless`.

### Step 4 — Configure Environment Variables

Create a `.env` file in the project root with the following variables:

```env
SECRET_KEY=your_django_secret_key_here
AI_API_KEY=your_google_gemini_api_key_here
EMAIL_HOST_USER=your_email@gmail.com
EMAIL_HOST_PASSWORD=your_app_password_here
```

> **Note:** Never commit the `.env` file to version control. The `.gitignore` file is pre-configured to exclude it.

### Step 5 — Apply Database Migrations

```bash
python manage.py makemigrations
python manage.py migrate
```

### Step 6 — Create a Superuser (Administrator Account)

```bash
python manage.py createsuperuser
```

Follow the prompts to set an email and password for the admin account.

### Step 7 — Collect Static Files (for Production)

```bash
python manage.py collectstatic
```

### Step 8 — Run the Development Server

```bash
python manage.py runserver
```

The application will be accessible at `http://127.0.0.1:8000/`.

---

## Usage

### Public Access

Navigate to `http://127.0.0.1:8000/` to view the landing page displaying recently approved missing person cases. Use `http://127.0.0.1:8000/portal/cases/` to access the full public directory with name and location filters. Any visitor can submit a sighting report for an active case by navigating to a case's public detail page.

### Registered User Flow

1. Navigate to `/accounts/signup/` and register with a valid email address.
2. Check your email for the 6-digit OTP and enter it at the verification page within 10 minutes.
3. Log in at `/accounts/login/` using your email and password.
4. Access your dashboard at `/dashboard/user/` to view your case statistics.
5. Submit a new missing person case via `/cases/submit/`.
6. Monitor your cases under `/cases/my-records/` and request priority review if needed.

### Administrator Flow

1. Log in with a staff account. You will be automatically redirected to the admin dashboard at `/dashboard/admin/`.
2. Review the pending approval queue and approve or reject submissions.
3. For approved cases, navigate to the case and click the AI analysis trigger to run the Gemini forensic pipeline at `/ai-engine/analyze/<case_id>/`.
4. Review the intelligence brief including extracted biometric markers and the AI-generated investigative strategy.
5. Manage community sighting reports via `/reports/list/`.
6. Close resolved cases and access archived dossiers via `/cases/archives/closed/`.

---

## Future Enhancements

- **PostgreSQL Integration** — Migrate from SQLite to PostgreSQL for production-grade concurrent access and improved query performance at scale.
- **Face Similarity Matching** — Integrate a deep learning face recognition model (e.g., FaceNet or DeepFace) to compare incoming photographs against the active case registry and surface potential identity matches automatically.
- **Real-Time Notifications** — Implement WebSocket-based push notifications using Django Channels to alert administrators of new submissions and alert reporters of case status changes in real time.
- **Geospatial Sighting Mapping** — Integrate a mapping API (e.g., Leaflet.js with OpenStreetMap) to render sighting report locations and last-seen coordinates on an interactive map within each case dossier.
- **Multi-Language Support** — Expand the internationalization configuration (currently scaffolded with `USE_I18N = True`) to support regional languages, increasing accessibility for a broader user base.
- **PDF Report Generation** — Allow administrators to export a complete forensic dossier for any case as a downloadable PDF, including the AI intelligence brief, sighting report log, and case timeline.
- **Role-Based Access Control Expansion** — Introduce a dedicated Field Officer role with view-only access to approved cases and sighting reports, without case management privileges.
- **Audit Logging** — Implement a comprehensive audit trail for all administrative actions (approvals, rejections, closures, AI triggers) with timestamps and actor identities for accountability.
- **AI Model Versioning** — Abstract the Gemini model identifier into the environment configuration to allow seamless model upgrades without code changes.
- **Automated Email Alerts on Case Approval** — Trigger an automated notification email to the case reporter when their submission is approved or rejected by an administrator.

---

## Contributing

Contributions are welcome. Please follow the standard Git workflow outlined below.

### Steps to Contribute

1. **Fork** the repository to your own GitHub account.

2. **Clone** your fork locally:
   ```bash
   git clone https://github.com/your-username/FindingMissingPerson_Trace_AI.git
   cd FindingMissingPerson_Trace_AI
   ```

3. **Create a feature branch** with a descriptive name:
   ```bash
   git checkout -b feature/your-feature-name
   ```

4. **Make your changes** following the existing code style and project structure.

5. **Commit** your changes with a clear, descriptive commit message:
   ```bash
   git add .
   git commit -m "feat: describe your change concisely"
   ```

6. **Push** to your fork:
   ```bash
   git push origin feature/your-feature-name
   ```

7. **Open a Pull Request** against the `main` branch of the original repository. Provide a clear description of the changes made and the problem they solve.

### Guidelines

- Ensure all new views are protected with appropriate decorators (`@login_required` or `@staff_member_required`).
- Do not commit `.env` files, `db.sqlite3`, or any `__pycache__` directories.
- Write descriptive docstrings for all new view functions and utility methods.
- Test all form submissions and authentication flows before opening a pull request.

---

## License

This project is licensed under the **MIT License**.

```
MIT License

Copyright (c) 2024 Harisai Parasa

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

For technical questions, issue reporting, or collaboration inquiries, please use the contact information below.

**GitHub:** [https://github.com/harisaigithub](https://github.com/harisaigithub)

**Email:** [harisaiparasa@gmail.com](mailto:harisaiparasa@gmail.com)

---


&nbsp;

&nbsp;

## ***To acquire this project, please contact***

**GitHub:** https://github.com/harisaigithub
**Email:** [harisaiparasa@gmail.com](mailto:harisaiparasa@gmail.com)

For collaborations, customizations, or deployment support, feel free to reach out.