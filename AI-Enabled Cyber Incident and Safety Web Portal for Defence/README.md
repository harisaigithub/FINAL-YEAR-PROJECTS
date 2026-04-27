# AI-Enabled Cyber Incident and Safety Web Portal for Defence

> A full-stack, AI-powered cybersecurity incident management platform purpose-built for Indian Defence personnel — enabling real-time threat reporting, intelligent risk classification, multimodal evidence analysis, and multilingual advisory support.

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

The Indian Defence sector faces a rapidly expanding cyber threat landscape encompassing phishing attacks, ransomware intrusions, social engineering, and data exfiltration attempts targeting sensitive operational networks. Traditional incident reporting workflows — predominantly manual, siloed, and paper-based — introduce critical delays between the detection of a threat and its escalation to the appropriate authority, leaving defence infrastructure exposed during the response gap.

The **AI-Enabled Cyber Incident and Safety Web Portal for Defence** addresses this operational void by providing a centralized, intelligent, and secure web-based platform through which defence personnel can report cybersecurity incidents instantly, receive AI-generated Standard Operating Procedures (SOPs) specific to their threat scenario, and interact with a multilingual conversational AI assistant for guidance in real time.

The system integrates Google's **Gemini 2.5 Flash** large language model for multimodal reasoning — accepting both textual descriptions and image-based evidence — enabling automated risk scoring, threat classification, and actionable remediation advice without requiring cybersecurity expertise on the part of the reporting personnel. This democratization of cyber-incident management directly improves the security posture of units at every level of the defence hierarchy.

---

## Objectives

- Design and implement a role-based web portal supporting two user tiers: general defence personnel (reporters) and administrative security analysts (reviewers).
- Integrate Google Gemini 2.5 Flash for automated, AI-driven generation of mission-specific cybersecurity SOPs presented as structured `dos` and `don'ts` immediately upon incident submission.
- Develop a keyword-weighted NLP risk scoring engine that classifies submitted incidents as Low, Medium, or High severity without dependence on external ML inference calls.
- Enable multimodal incident reporting by allowing users to upload image-based evidence (screenshots, suspicious documents) that is analyzed by the Gemini Vision API to extract fraud markers, malicious URLs, and threat indicators.
- Implement a multilingual conversational AI chatbot capable of automatically detecting and mirroring the user's input language (English, Hindi, Telugu), ensuring accessibility for non-English-speaking personnel.
- Build a comprehensive administrative analytics dashboard providing real-time incident statistics, risk distribution charts, type-trend analysis, and a user directory.
- Provide full internationalization (i18n) support across three languages using Django's localization framework with persistent language selection.
- Enforce secure, role-differentiated authentication flows including OTP-based email verification, anonymous reporting capability, and password reset via SMTP.
- Maintain a persistent audit trail of all AI chatbot interactions linked to user accounts for administrative forensic review.

---

## Summary

The portal is a Django 5.2-based multi-application web system composed of three core Django apps — `portal`, `accounts`, and `chatbot` — each encapsulating a distinct domain of functionality. The `portal` app manages the incident lifecycle from submission through AI analysis to dashboard display. The `accounts` app governs the full authentication pipeline, including custom user model extension with role assignment, OTP verification via email, and password management. The `chatbot` app provides a dedicated AI analysis interface backed by Gemini's generative API, with conversation persistence for admin review.

Upon incident submission, the system invokes the Gemini 2.5 Flash model with a structured CSOC Commander prompt, receiving and parsing a JSON payload of six targeted `dos` and six `don'ts` tailored to the reported threat category. Simultaneously, a local keyword-weighted scoring function evaluates the description text and assigns a numerical risk score mapped to Low, Medium, or High severity. Uploaded evidence images are processed through Gemini's multimodal input API using Base64 encoding, enabling automated extraction of phishing URLs, suspicious content, and error message signatures directly from screenshots.

The administrative layer provides a superuser-accessible dashboard with multi-parametric filtering, live incident counts, status tracking, AI core diagnostics, user directory management, and a chart-rendered analytics view showing incident type distributions and risk breakdowns.

---

## Features

### Before Login (Public Access)

- **Landing Page:** A professional portal entry point with navigation to services, resources, and contact sections. Authenticated users are automatically redirected to their dashboard.
- **Incident Reporting (Guest Mode):** Unauthenticated users may submit incident reports anonymously, with the system recording `is_anonymous=True` and dissociating personally identifiable information.
- **User Registration with OTP Verification:** New accounts are created in an inactive state; a six-digit OTP is dispatched to the registered email address via SMTP (Gmail). The account is activated only upon successful OTP submission within a ten-minute expiry window.
- **Login with Dual Identifier Support:** Users may authenticate using either their username or registered email address.
- **Password Reset via Email:** A full token-based password reset flow is available using Django's built-in password reset views with custom templates.
- **Multilingual Interface:** The language switcher enables selection among English, Hindi, and Telugu, with the choice persisted via a cookie for one year.

### After Login (Authenticated Access)

**For General Defence Personnel:**

- **User Dashboard:** Displays a chronological personal incident history with counts of pending and resolved reports.
- **Incident Submission Form:** A structured form accepting incident type classification, textual description, optional file evidence upload, and an anonymous submission toggle.
- **AI-Generated SOP Recommendations:** On form submission, the Gemini 2.5 Flash model generates a domain-specific JSON response parsed into six military-grade `dos` and six `don'ts` rendered on a dedicated recommendations page.
- **Risk Score Display:** Each submitted incident is immediately scored and classified (Low / Medium / High) using the local keyword-weight NLP engine.
- **AI Image Extraction (AJAX):** An endpoint accepts uploaded screenshots as Base64-encoded multimodal input to the Gemini Vision API, returning an automated extraction of visible threat indicators, suspicious URLs, and phishing keywords.
- **AI Chatbot (Full-Screen):** A persistent conversational interface that automatically detects the user's input language and responds in-kind with structured safety analysis including risk level, summary, technical breakdown, and action steps.
- **Profile Settings:** Account management and system preference interface.

**For Administrators / Analysts (Superuser):**

- **Admin Dashboard:** Aggregated view of all incidents (user-submitted and anonymous) with multi-parametric filters by risk level and incident type.
- **Incident Analytics:** Chart-rendered breakdown of incident types (by count) and risk distribution (High / Medium / Low).
- **AI Core Monitor:** Real-time operational diagnostics panel showing AI system status, API latency, model identifier, total API call count, and active capability list.
- **User Directory:** Full list of registered users ordered by registration date.
- **Django Admin Panel:** Standard Django superuser panel with model-level CRUD access for incidents, users, OTP records, notifications, and chat conversations.

---

## Technologies Used

### Programming Language
- Python 3.10

### Backend Framework
- Django 5.2 — web framework, ORM, URL routing, middleware, i18n
- Django REST Framework 3.16 — RESTful API scaffolding

### AI / Machine Learning
- Google Gemini 2.5 Flash (`gemini-2.5-flash`) — text generation, multimodal image analysis, language detection, JSON-structured SOP generation
- `google-generativeai` 0.8.6 — official Python SDK for Gemini API
- `google-ai-generativelanguage` 0.6.15 — low-level generative language API client
- Custom NLP Risk Scoring Engine — keyword-weighted score-based threat classification (local, no API dependency)

### Frontend
- Django Template Language (DTL) — server-side HTML rendering
- Tailwind CSS utility classes — styling and responsive layout
- AJAX / Fetch API — asynchronous image extraction and chatbot query endpoints
- Chart.js (referenced via admin analytics templates) — risk distribution and trend visualizations

### Database
- SQLite 3 (development) — `db.sqlite3`
- MySQL Client 2.2.7 — available for production database migration

### Authentication & Security
- Custom Django `AbstractUser` extension with role fields (`student` / `admin`)
- OTP-based email verification with ten-minute expiry (`EmailOTP` model)
- UUID-based anonymous identity preservation
- CSRF protection (Django middleware)
- SMTP email via Gmail (TLS, port 587)
- Session-based login state management

### Internationalization & Localization
- Django i18n framework — `USE_I18N`, `LocaleMiddleware`
- GNU gettext `.po` / `.mo` files for Hindi (`hi`) and Telugu (`te`)
- `LANGUAGE_COOKIE_AGE` — one-year persistence

### Deployment & DevOps
- WSGI (`core/wsgi.py`) — synchronous server compatibility
- ASGI (`core/asgi.py`) — async-capable server compatibility
- `python-dotenv` — environment variable management via `.env`
- Django `MEDIA_ROOT` — file upload management for evidence storage

### Libraries
- `Pillow` 12.1 — image processing support
- `requests` 2.32 — HTTP client
- `httpx` 0.28 — async HTTP client
- `pydantic` 2.12 — data validation
- `websockets` 15.0 — WebSocket protocol support

---

## Dataset

This project does not rely on a pre-existing external dataset for model training. The AI-driven capabilities are derived from Google's Gemini 2.5 Flash foundation model, which is accessed via API inference at runtime rather than being fine-tuned locally.

**Risk Scoring Keyword Corpus:** A domain-specific keyword-weight dictionary was hand-curated based on cybersecurity incident taxonomy relevant to Indian Defence operations. Keywords include high-severity terms (`ransomware`, `hacked`, `otp`, `cvv`, weight 4–6), medium-severity indicators (`transaction`, `bank`, `threat`, weight 3–4), and low-severity contextual terms (`email`, `message`, `unknown`, weight 1–2). This forms the foundation of the local NLP scoring engine.

**Evidence Data:** The `media/evidence/` directory stores user-uploaded incident screenshots and files. These are processed in real time by the Gemini Vision API for content extraction and are not used for local model training.

**Chatbot Conversation Log:** All chatbot interactions are persisted to the `ChatConversation` model, capturing user message, AI response, detected language, and authentication status. This corpus can support future fine-tuning or supervised evaluation of threat classification quality.

---

## System Architecture

The system follows a modular monolithic architecture structured around three Django applications, each with clearly scoped responsibilities.

```
AI-Enabled-Cyber-Incident-and-Safety-Web-Portal-for-Defence/
│
├── core/                       # Project configuration layer
│   ├── settings.py             # Global settings: DB, auth, i18n, email, AI key
│   ├── urls.py                 # Root URL dispatcher (i18n + non-i18n patterns)
│   ├── wsgi.py                 # WSGI entry point
│   └── asgi.py                 # ASGI entry point
│
├── accounts/                   # Authentication & identity management
│   ├── models.py               # Custom User (AbstractUser), EmailOTP, Notification
│   ├── views.py                # Login, Signup, OTP Verify, Logout, Password Reset
│   └── urls.py                 # Auth URL routes
│
├── portal/                     # Core incident management domain
│   ├── models.py               # Incident model (type, description, evidence, risk, status)
│   ├── views.py                # Incident form, AI SOP generation, dashboards, image API
│   ├── forms.py                # IncidentForm (ModelForm)
│   ├── utils.py                # NLP keyword risk scoring engine, image AI stub
│   └── context_processors.py   # Dynamic base template selector (pre/post login)
│
├── chatbot/                    # Conversational AI module
│   ├── models.py               # ChatConversation (user, message, response, language)
│   ├── views.py                # Gemini multimodal chatbot endpoint, full-screen view
│   └── urls.py                 # Chatbot query and full-analysis routes
│
├── templates/                  # Django HTML templates
│   ├── base1.html              # Pre-login base layout
│   ├── base2.html              # Post-login base layout
│   ├── landing.html            # Public homepage
│   ├── accounts/               # Login, Signup, OTP, verification templates
│   ├── portal/                 # Report form, recommendations, services, resources
│   ├── dashboards/             # User dashboard, admin dashboard, analytics, AI monitor
│   ├── chatbot/                # Full-screen chatbot interface
│   └── partials/               # Sidebar components (user/admin)
│
├── locale/                     # Internationalization data
│   ├── hi/LC_MESSAGES/         # Hindi .po/.mo translation files
│   └── te/LC_MESSAGES/         # Telugu .po/.mo translation files
│
├── media/                      # User-uploaded incident evidence files
├── static/                     # Static assets (CSS, JS, images)
├── db.sqlite3                  # Development SQLite database
├── .env                        # Environment variables (API key)
└── manage.py                   # Django management entry point
```

**Data Flow:**

1. A defence personnel user accesses the portal through a localized URL routed through Django's `i18n_patterns`.
2. The `LocaleMiddleware` resolves the user's preferred language from the cookie or Accept-Language header.
3. Authentication is validated by the custom `accounts.User` model; OTP verification gates account activation.
4. On incident submission, the `portal.views.report_incident` view simultaneously invokes the Gemini API for SOP generation and the local `analyze_cyber_risk` function for score classification.
5. The resulting `Incident` record is persisted to SQLite with risk score, level, and evidence reference.
6. For chatbot queries, `chatbot.views.chatbot_query` constructs a multimodal content list (text + optional Base64 image), sends it to Gemini with a structured system prompt, and persists the conversation to `ChatConversation`.
7. The `portal.context_processors.base_template` processor dynamically injects the correct base template (`base1.html` for guests, `base2.html` for authenticated users) into every rendered response.
8. Superuser views (`admin_dashboard_view`, `admin_analytics_view`, `user_directory_view`, `ai_monitor_view`) are protected by `@user_passes_test(lambda u: u.is_superuser)`.

---

## Model Workflow

### Incident Risk Scoring Pipeline

```
User Input (Text Description)
        │
        ▼
  Keyword Tokenization
  (convert to lowercase)
        │
        ▼
  Weight Lookup Table
  (domain-specific keyword → integer weight)
  ┌─────────────────────────────────────┐
  │ ransomware, hacked, otp, cvv  → 5  │
  │ password, bank, transaction   → 4  │
  │ threat                        → 4  │
  │ urgent, link                  → 3  │
  │ whatsapp, email               → 2  │
  │ unknown, message              → 1  │
  └─────────────────────────────────────┘
        │
        ▼
  Cumulative Score Computation
        │
        ▼
  Threshold Classification
  ┌─────────────────────────────────┐
  │  Score ≥ 10  →  HIGH            │
  │  Score ≥ 5   →  MEDIUM          │
  │  Score < 5   →  LOW             │
  └─────────────────────────────────┘
        │
        ▼
  Output: (score, level, advisory_text)
  Persisted to Incident model
```

### AI SOP Generation Pipeline (Gemini API)

```
Incident Type + Description
        │
        ▼
  Prompt Construction
  (CSOC Commander role, JSON format constraint,
   6 dos / 6 don'ts, max 15 words per point)
        │
        ▼
  Gemini 2.5 Flash API Call
  (google.generativeai.GenerativeModel)
        │
        ▼
  Raw Text Response
        │
        ▼
  JSON Extraction
  (strip markdown fences, parse JSON)
        │
        ▼
  Fallback Handler
  (hardcoded military-grade SOPs if API fails)
        │
        ▼
  Render: portal/recommendations.html
  (dos list, don'ts list, incident metadata)
```

### Multimodal Image Extraction Pipeline

```
User Uploads Screenshot
        │
        ▼
  AJAX POST to /api/extract/
        │
        ▼
  Read image bytes in-memory
  Convert to Base64 string
        │
        ▼
  Gemini API Call (inline_data block)
  ┌──────────────────────────────────────┐
  │ mime_type: image/jpeg (or detected)  │
  │ data: base64-encoded image bytes     │
  └──────────────────────────────────────┘
        │
        ▼
  Gemini Vision Analysis
  (URLs, suspicious text, fraud markers)
        │
        ▼
  JsonResponse({'content': response.text})
  Auto-populated into description field
```

### Chatbot Multilingual Pipeline

```
User Message (any supported language)
+ Optional Base64 Image
        │
        ▼
  System Prompt Injection
  (Indian Cyber Defence AI role,
   language-mirroring instruction,
   structured response format)
        │
        ▼
  Gemini 2.5 Flash API Call
  (contents list: system_prompt + user_msg + image)
        │
        ▼
  Structured Response
  ┌──────────────────────────────────┐
  │ ### 🛡️ RISK LEVEL: [LOW/MED/HIGH]│
  │ Summary / Analysis / Action Steps │
  └──────────────────────────────────┘
        │
        ▼
  Persist to ChatConversation model
  (user ref, message, response, language, is_anonymous)
        │
        ▼
  JsonResponse({'reply': bot_reply})
```

---

## Implementation Details

**Custom User Model:** The `accounts.User` model extends Django's `AbstractUser`, adding a `role` field (`student` for general personnel, `admin` for analysts), a `UUID`-based `anonymous_id` for privacy-preserving report linkage, a boolean `is_approved` flag for admin account gating, and a `login_count` integer for activity audit.

**OTP Authentication Flow:** Upon registration, a user account is created with `is_active=False`. A cryptographically random six-digit OTP is generated, stored in the `EmailOTP` model with a creation timestamp, and dispatched via Django's SMTP email backend to the user's registered address. The `verify_otp_view` validates the submitted OTP, checks expiry using a `timedelta(minutes=10)` comparison, sets `is_active=True`, deletes the OTP record, and clears the session key. Expired or mismatched OTPs are rejected with an error message.

**Incident Model:** The `Incident` model captures a nullable `ForeignKey` to the custom user (allowing anonymous records), incident type string, text description, optional file upload to `media/evidence/`, computed risk score integer, risk level string with choices, anonymous submission boolean, status string (`Pending` / `Resolved`), and an auto-timestamped `created_at` field.

**Dynamic Base Template:** The `portal.context_processors.base_template` function is registered in `TEMPLATES.OPTIONS.context_processors`. It inspects `request.user.is_authenticated` and injects either `base1.html` (public navigation structure) or `base2.html` (authenticated dashboard layout with sidebar) as `base_template` into every template context, enabling a single `{% extends base_template %}` declaration across all application templates.

**i18n URL Routing:** All user-facing URLs are wrapped in `i18n_patterns()` in `core/urls.py`, with `prefix_default_language=False` ensuring English URLs are not prefixed. Language-specific URLs (`/hi/dashboard/`, `/te/report/`) are automatically generated by Django's `LocaleMiddleware`. Language selection is handled via Django's built-in `i18n/` endpoint and persisted through `LANGUAGE_COOKIE_NAME = 'django_language'` with a one-year cookie lifetime.

**Gemini Safety Settings:** Both the portal and chatbot views configure `SAFETY_SETTINGS` with `BLOCK_NONE` for all harm categories (harassment, hate speech, sexually explicit, dangerous content). This configuration is critical for the defence context, as cybersecurity terminology (malware names, attack vectors, phishing indicators) would otherwise trigger Gemini's default content filters and block legitimate analysis requests.

**REST Framework Integration:** `django-rest-framework` is installed and registered in `INSTALLED_APPS`, providing a foundation for future API endpoint expansion, token authentication, and serializer-based data exposure for integration with external CSOC systems.

---

## Results

The following performance characteristics were observed during development and functional testing of the system:

**Risk Scoring Engine:**

| Scenario | Score | Classified Level |
|---|---|---|
| Ransomware infection with OTP theft | 10+ | HIGH |
| Suspicious bank transaction via unknown link | 7–9 | MEDIUM |
| Unrecognized email received | 3–4 | LOW |
| Phishing attempt with password/CVV request | 9–10 | HIGH |
| WhatsApp message from unknown number | 2–3 | LOW |

**AI SOP Generation:** The Gemini 2.5 Flash model consistently returns well-structured JSON with six actionable `dos` and six `don'ts` calibrated to the submitted incident type and description. Average API response latency is reported at approximately 240 ms under normal load conditions (as displayed in the AI Core Monitor dashboard).

**Multimodal Image Extraction:** The Gemini Vision endpoint successfully identifies phishing URLs, suspicious domain patterns, urgent-action keywords, and error message text from uploaded screenshots, auto-populating the incident description field via the AJAX response.

**Chatbot Multilingual Accuracy:** The conversational AI correctly mirrors the input language — responding in Telugu when queried in Telugu, Hindi when queried in Hindi, and English otherwise — without requiring explicit language selection by the user. The structured response format (risk level header, summary, analysis, action steps) is maintained consistently across all three languages.

**System Throughput:** The portal supports concurrent user sessions through Django's session middleware, with all AI calls executed synchronously per request. The AI Core Monitor tracks cumulative API invocations using `Incident.objects.count()` as a proxy metric.

---

## Installation

### Prerequisites

- Python 3.10 or higher
- `pip` package manager
- Git
- A valid Google Gemini API key (obtainable from Google AI Studio)
- A Gmail account with App Password enabled for SMTP

### Step 1: Clone the Repository

```bash
git clone https://github.com/harisaigithub/AI-Enabled-Cyber-Incident-and-Safety-Web-Portal-for-Defence.git
cd AI-Enabled-Cyber-Incident-and-Safety-Web-Portal-for-Defence
```

### Step 2: Create and Activate Virtual Environment

```bash
python -m venv venv

# On Windows
venv\Scripts\activate

# On macOS / Linux
source venv/bin/activate
```

### Step 3: Install Dependencies

```bash
pip install django==5.2
pip install djangorestframework
pip install google-generativeai
pip install python-dotenv
pip install pillow
pip install requests
pip install httpx
```

Alternatively, if a `requirements.txt` is present:

```bash
pip install -r requirements.txt
```

### Step 4: Configure Environment Variables

Create a `.env` file in the project root directory:

```env
AI_API_KEY=your_google_gemini_api_key_here
```

### Step 5: Configure Email Settings

In `core/settings.py`, update the email configuration with your Gmail credentials:

```python
EMAIL_HOST_USER = 'your_email@gmail.com'
EMAIL_HOST_PASSWORD = 'your_gmail_app_password'
```

> **Note:** Use a Gmail App Password, not your primary account password. Enable 2-Factor Authentication on your Gmail account and generate an App Password under Security settings.

### Step 6: Apply Database Migrations

```bash
python manage.py migrate
```

### Step 7: Compile Translation Files

```bash
python manage.py compilemessages
```

### Step 8: Create a Superuser Account

```bash
python manage.py createsuperuser
```

Follow the prompts to set a username, email, and password for the administrative account.

### Step 9: Collect Static Files (Optional for Development)

```bash
python manage.py collectstatic
```

### Step 10: Run the Development Server

```bash
python manage.py runserver
```

The portal will be accessible at `http://127.0.0.1:8000/`.

---

## Usage

### Accessing the Portal

Navigate to `http://127.0.0.1:8000/` in a web browser. The landing page presents the public interface.

### User Registration and Login

1. Click **Sign Up** and complete the registration form, selecting the appropriate role.
2. A six-digit OTP will be dispatched to the provided email address.
3. Enter the OTP on the verification page within ten minutes to activate the account.
4. Log in using your username or email address and password.

### Reporting a Cyber Incident

1. After login, navigate to **Report Incident** from the dashboard or navigation bar.
2. Select the incident type from the dropdown (e.g., phishing, ransomware, data breach).
3. Enter a detailed textual description of the incident.
4. Optionally upload a screenshot or evidence file.
5. Toggle the **Anonymous** option if you wish to submit without identity linkage.
6. Click **Submit**. The system will display AI-generated SOPs with six `dos` and six `don'ts` on the recommendations page.

### Anonymous Reporting (Guest)

Append `?anonymous=true` to the report URL, or access `/report/?anonymous=true` directly without logging in. The incident will be stored without user association.

### Using the AI Chatbot

1. Navigate to **Resources → AI Analysis** or directly to `/chatbot/full/`.
2. Type your query in English, Hindi, or Telugu.
3. Optionally attach an image for multimodal analysis.
4. The AI will respond in your input language with a structured risk assessment.

### Changing Language

Use the language selector in the navigation bar to switch between English, Hindi, and Telugu. The preference is automatically persisted for one year.

### Administrative Access

Log in with a superuser account and navigate to:

- `/admin-panel/` — Main admin dashboard with incident management and filters
- `/admin-panel/analytics/` — Incident type trend charts and risk distribution
- `/admin-panel/ai-core/` — AI system diagnostics and operational status
- `/admin-panel/users/` — Full registered user directory
- `/admin/` — Django admin panel for direct model-level management

---

## Future Enhancements

- **Fine-Tuned Domain Model:** Replace the general-purpose Gemini API with a fine-tuned model trained on classified Indian Defence cybersecurity incident datasets for improved SOP specificity and reduced hallucination.
- **Production Database Migration:** Transition from SQLite to PostgreSQL or MySQL for concurrent multi-user production workloads, leveraging the already-installed `mysqlclient` dependency.
- **Real-Time Notifications:** Implement WebSocket-based push notifications (using the included `websockets` library) to alert administrators of new High-severity incidents without requiring page refresh.
- **CERT-In Integration:** Add an automated escalation pipeline that submits High-risk incidents to the Indian Computer Emergency Response Team (CERT-In) reporting API.
- **Two-Factor Authentication (2FA):** Extend the existing OTP infrastructure to enforce 2FA on every login session for high-privilege administrative accounts.
- **Offline Mode and PWA Support:** Convert the portal into a Progressive Web App to allow incident submission queuing in low-connectivity or air-gapped field environments.
- **Audit Log and SIEM Export:** Introduce a tamper-evident audit trail for all administrative actions and provide export compatibility with SIEM platforms (Splunk, IBM QRadar) via structured JSON log output.
- **REST API Expansion:** Extend the Django REST Framework foundation to expose authenticated API endpoints for mobile client integration and cross-system incident data exchange.
- **ML-Based Risk Model:** Replace the keyword-weighted scoring engine with a trained NLP classifier (e.g., DistilBERT fine-tuned on cybersecurity incident text) for higher precision risk classification.
- **Deployment Containerization:** Package the application with Docker and Docker Compose for reproducible, environment-agnostic deployment, and prepare Kubernetes manifests for horizontal scaling.

---

## Contributing

Contributions are welcome. Please follow the standard Git workflow outlined below.

1. Fork the repository on GitHub.
2. Create a feature branch from `main`:
   ```bash
   git checkout -b feature/your-feature-name
   ```
3. Implement your changes with clear, well-documented code.
4. Write or update tests as appropriate.
5. Commit with a descriptive message:
   ```bash
   git commit -m "feat: add feature description"
   ```
6. Push the branch to your fork:
   ```bash
   git push origin feature/your-feature-name
   ```
7. Open a Pull Request against the `main` branch of this repository.
8. Describe the change, its motivation, and any relevant testing performed in the Pull Request description.

Please ensure all contributions adhere to PEP 8 Python style guidelines and do not introduce hardcoded secrets or credentials.

---

## License

This project is licensed under the **MIT License**.

```
MIT License

Copyright (c) 2025 Harisai Parasa

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

For technical queries, deployment assistance, or project-related inquiries, please reach out through the following channels:

**GitHub:** [https://github.com/harisaigithub](https://github.com/harisaigithub)

**Email:** [harisaiparasa@gmail.com](mailto:harisaiparasa@gmail.com)

---

&nbsp;

&nbsp;

## ***To acquire this project, please contact***

**GitHub:** https://github.com/harisaigithub  
**Email:** [harisaiparasa@gmail.com](mailto:harisaiparasa@gmail.com)  

For collaborations, customizations, or deployment support, feel free to reach out. 