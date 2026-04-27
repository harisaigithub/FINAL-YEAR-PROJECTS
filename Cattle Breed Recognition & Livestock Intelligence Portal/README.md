# Cattle Breed Recognition & Livestock Intelligence Portal

> An AI-powered, full-stack web platform for real-time Indian cattle and buffalo breed identification using Google Gemini Vision and a locally trained TensorFlow model — built to empower farmers and livestock administrators with intelligent breed analytics, scan history, PDF reporting, and a multilingual chatbot assistant.

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

Indian agriculture and dairy farming depend critically on the identification and management of livestock breeds. Accurate breed recognition directly impacts breeding programs, productivity assessment, insurance evaluation, and government subsidy schemes. However, breed identification in rural settings is largely manual — relying on veterinary expertise that is geographically inaccessible to most small-scale farmers.

The **Cattle Breed Recognition & Livestock Intelligence Portal** addresses this gap by providing a mobile-accessible, AI-powered web application through which farmers and livestock administrators can upload a photograph of any cattle or buffalo and receive an instant, AI-generated breed identification with confidence scoring. The platform integrates Google Gemini 2.5 Flash as the primary vision model and falls back to a locally trained TensorFlow SavedModel when the API is unavailable, ensuring high system resilience.

Beyond recognition, the portal delivers a comprehensive livestock management ecosystem: a curated breed library with detailed agronomic profiles, a personal scan history with confidence-based filtering, a PDF report generator for livestock statistics, a real-time notification system, an editable user profile module linked to farm location, and an administrative panel for dataset validation, breed management, and explorer directory management. The system is designed for both individual farmers (explorers) and administrative analysts, with strict role-based access control and OTP-based email authentication to ensure secure multi-user operation.

---

## Objectives

- Design and deploy a dual-AI breed recognition pipeline: Google Gemini 2.5 Flash as the primary inference engine and a locally hosted TensorFlow SavedModel as the fallback, ensuring uninterrupted service under API downtime.
- Support identification of six major Indian livestock breeds — Gir, Sahiwal, Red Sindhi, Murrah Buffalo, Jersey, and Holstein Friesian — with confidence scores normalized to a 0–100 percentage scale.
- Build a thread-safe, lazy-loaded model serving architecture using Python's `threading.Lock` to prevent concurrent model initialization conflicts in production.
- Implement a secure, email-first authentication system using a custom `AbstractUser` model (email as the primary identifier, no username), OTP verification with a ten-minute expiry window, and role-based access differentiation between farmers and administrators.
- Develop a structured breed knowledge library with full CRUD operations for administrators, supporting thumbnails, origin location, daily milk yield data, and optional reference video links.
- Provide personal scan history tracking with high/low confidence filtering and a scan detail view, enabling farmers to audit their recognition history over time.
- Generate downloadable PDF livestock statistics reports using ReportLab, covering total scans and unique breed counts per user.
- Integrate a multilingual conversational AI chatbot backed by Gemini 2.5 Flash, with automatic markdown stripping for clean plain-text output in the UI.
- Deliver a real-time notification system with typed alerts (success, info, warning, feature) surfaced via a global context processor and a dedicated notifications page.
- Build an administrative intelligence analytics panel providing breed frequency charts, dataset validation workflows (approve/discard scan records), and a user explorer directory with per-user scan counts.
- Support internationalization in English, Hindi, and Telugu with persistent language selection stored as a one-year cookie.

---

## Summary

The Cattle Breed Recognition & Livestock Intelligence Portal is a Django 5.2 multi-application web system composed of nine specialized Django apps — `accounts`, `recognition`, `breeds`, `portal`, `chatbot`, `history`, `notifications`, `profiles`, and `reports` — each encapsulating a distinct functional domain.

The recognition pipeline accepts a cattle or buffalo image uploaded by an authenticated farmer, passes it first to the Gemini 2.5 Flash multimodal API with a structured JSON-output prompt, parses the breed name, animal type, and confidence score from the response, and falls back to a locally loaded TensorFlow SavedModel (`cattle_saved_model`) if the Gemini call fails. The local model preprocesses images to 224×224 RGB, runs inference through the SavedModel's `serving_default` signature, and maps the argmax prediction to one of six class names. Confidence values from both paths are normalized to a 0–100 integer scale before being saved as a `BreedScan` record.

The administrative layer provides superuser-accessible views for dataset review (validating or discarding unvalidated `BreedScan` records), breed library management (full CRUD with thumbnail upload), analytics charts (breed frequency distribution), and an explorer directory (all users with scan counts and recent activity tracking). A `populate_data.py` script seeds the database with five representative breeds, twenty-five randomized scan records, and a demo farmer account for rapid demonstration setup.

---

## Features

### Before Login (Public Access)

- **Landing Page:** A professional public homepage introducing the platform with navigation to the login and signup interfaces.
- **User Registration (Email-Based):** New accounts are registered using email as the unique identifier (no username required). A six-digit OTP is dispatched via SMTP and the account is kept inactive until OTP verification. Role selection (farmer or administrator) is captured at signup.
- **OTP Verification:** A dedicated OTP entry page validates the submitted code against the stored `EmailOTP` record, checking expiry with a ten-minute threshold. On success, `is_active` and `is_verified` are both set to `True`.
- **Login with Role-Based Redirection:** Farmer accounts are redirected to the user dashboard; administrator accounts are redirected to the admin management panel. Inactive accounts receive an explicit verification prompt.
- **Password Reset Flow:** A complete four-step token-based password reset flow using Django's built-in views with custom templates (request form, email sent confirmation, new password entry, completion screen).

### After Login (Authenticated — Farmer/Explorer)

- **User Dashboard:** Displays live statistics including total scans, scans performed this week, number of distinct breeds identified, average confidence accuracy, and a list of the five most recent scan records. Unread notification count is surfaced in the header.
- **AI Breed Recognition (AI Lens):** The core feature. The farmer uploads a photograph of a cattle or buffalo animal. The system sends the image first to Google Gemini 2.5 Flash with an expert livestock identification prompt requesting structured JSON output. If the API fails or returns no parseable JSON, the system automatically falls back to the local TensorFlow SavedModel. The result page displays the detected breed name, animal type (Cattle or Buffalo), and an animated SVG circular confidence meter calculated from the confidence score.
- **Breed Library:** A searchable and filterable directory of all active breeds in the system. Users can search by name or origin location and filter by Cattle or Buffalo category. Each breed card links to a detailed profile with description, daily milk yield, origin, and optional reference video.
- **Scan History:** A chronological personal history of all recognition events with filter options for high-confidence (≥90%) and low-confidence (<70%) results. Each scan entry links to a detailed view showing the uploaded image, detected breed, confidence score, and scan timestamp.
- **Analytics Reports:** A personal livestock statistics dashboard with two chart visualizations — a pie chart showing breed distribution across all scans and a bar chart showing monthly scan volume. A download button generates and serves a formatted PDF report (A4) containing total scans, unique breed count, and an automated generation timestamp.
- **Notification Center:** A paginated list of all system notifications for the user, typed as Recognition Complete, System Update, Maintenance, or New Feature. Notifications are automatically created when breed recognition is completed successfully.
- **Profile Management:** Displays the farmer's full name, phone number, farm location, bio, and profile picture alongside calculated statistics (total scans, this week's scans, distinct breeds found, average confidence score, and account join date). An edit form allows updating all profile fields including profile picture upload.
- **AI Chatbot:** A conversational chatbot interface powered by Gemini 2.5 Flash, configured for livestock guidance and agricultural advisory. The chatbot accepts both text and Base64-encoded image inputs, strips markdown from all responses for clean UI rendering, and logs all conversations to the `ChatConversation` model.

### After Login (Administrator)

- **Admin Dashboard:** Aggregated system overview showing total registered users, total global scans, total breeds in the database, system operational status, and model version. Displays a preview of the breed library and the eight most recent global scans across all users.
- **Intelligence Analytics & Dataset Review:** Breed frequency analysis with chart-rendered distribution data. Dataset validation workflow displaying unvalidated scan records with approve (mark `is_validated=True`) and discard (delete) actions per scan.
- **Breed Management (Admin CRUD):** Full create, read, update, and delete interface for the breed library. Administrators can add new breeds with name, category, thumbnail image, description, origin location, daily milk yield range, optional video link, and active status toggle. Duplicate breed names are prevented by case-insensitive validation in the form's `clean_name()` method.
- **Explorer Directory:** A paginated list of all registered users annotated with per-user scan counts, last login timestamp, and active node status (logged in within the last 24 hours). System status and model version metadata are displayed in the header.
- **Django Admin Panel:** Standard superuser CRUD access to all models via `/admin/`.

---

## Technologies Used

### Programming Language
- Python 3.10 / 3.13

### Backend Framework
- Django 5.2.10 — core web framework, ORM, URL routing, middleware stack, i18n

### AI / Machine Learning
- Google Gemini 2.5 Flash (`gemini-2.5-flash`) — primary multimodal vision inference for breed identification
- Google Gemini 1.5 Flash (`gemini-1.5-flash`) — secondary recognition API endpoint in `portal/views.py`
- `google-genai` 1.61.0 — official Google GenAI SDK (`from google import genai`)
- `google-generativeai` — legacy SDK used in the chatbot module
- TensorFlow 2.10.1 — local SavedModel inference engine (fallback breed predictor)
- Keras 2.10.0 — model serialization format (`cattle_model_fixed.keras`)
- NumPy 1.23.5 — image array manipulation and argmax computation
- Custom SavedModel: `cattle_saved_model/` — six-class Indian livestock breed classifier

### Frontend
- Django Template Language (DTL) — server-side HTML rendering
- Tailwind CSS utility classes — layout, responsive design, component styling
- Crispy Forms + `crispy-bootstrap5` 2025.6 — Bootstrap 5 form rendering
- Chart.js — pie charts (breed distribution) and bar charts (monthly scan volume)
- SVG Animation — circular confidence meter with `stroke-dashoffset` animation

### Database
- SQLite 3 — development database (`db.sqlite3`)

### Authentication & Security
- Custom `AbstractUser` with email as `USERNAME_FIELD` (no username), `CustomUserManager` for `create_user` and `create_superuser`
- OTP-based email verification with `EmailOTP` model and ten-minute expiry
- Role-based access: `farmer` and `admin` roles with URL-level enforcement via `@login_required` and `@user_passes_test(lambda u: u.is_superuser)`
- CSRF protection (Django middleware)
- Session-based login state

### Internationalization & Localization
- Django i18n framework with `LocaleMiddleware`
- GNU gettext `.po` / `.mo` translation files for Hindi (`hi`) and Telugu (`te`)
- `LANGUAGE_COOKIE_AGE = 31536000` — one-year language persistence

### PDF Generation
- ReportLab 4.4.9 — programmatic A4 PDF generation for livestock statistics reports

### Image Processing
- Pillow 12.1.0 — image loading, RGB conversion, and 224×224 resizing for TensorFlow preprocessing

### Deployment & Configuration
- WSGI (`core/wsgi.py`) and ASGI (`core/asgi.py`) entry points
- `python-dotenv` 1.2.1 — environment variable management via `.env`
- Django `MEDIA_ROOT` — user-uploaded image management for scans, breed thumbnails, and profile pictures

### Additional Libraries
- `requests` 2.32.5 — HTTP client
- `h5py` 3.15.1 — HDF5 model file support
- `protobuf` 3.19.6 — TensorFlow SavedModel format support
- `grpcio` 1.76.0 — gRPC transport
- `tensorboard` 2.10.1 — model training visualization

---

## Dataset

### Breed Classification Model (Local TensorFlow SavedModel)

The local model is trained to recognize six Indian livestock breed classes:

| Index | Breed Name | Type |
|---|---|---|
| 0 | Gir | Cattle |
| 1 | Sahiwal | Cattle |
| 2 | Red Sindhi | Cattle |
| 3 | Murrah Buffalo | Buffalo |
| 4 | Jersey | Cattle |
| 5 | Holstein Friesian | Cattle |

**Input Specification:** Images are resized to 224×224 pixels, converted to RGB, normalized to float32 in the range [0.0, 1.0], and expanded to shape `(1, 224, 224, 3)` before inference.

**Model Format:** The trained model is stored in both Keras format (`cattle_model_fixed.keras`) and TensorFlow SavedModel format (`cattle_saved_model/`) with a `serving_default` signature for production inference. The SavedModel format is used at runtime to avoid Keras version compatibility issues.

**Training Data:** The model was trained on curated image datasets of Indian cattle and buffalo breeds. Images were sourced from livestock databases and agricultural resources, representing diverse environmental conditions, coat variations, and physical angles typical of field photographs.

**Seed Data (populate_data.py):** The `populate_data.py` script populates the breed library with five initial breeds (Gir, Sahiwal, Murrah, Ongole, Jaffarabadi) and generates twenty-five randomized `BreedScan` records spanning a fourteen-day window for the demo farmer account (`demo@farmer.com`), enabling immediate dashboard and analytics visualization without requiring real scan data.

---

## System Architecture

The system follows a modular monolithic architecture with nine clearly scoped Django applications organized around a central `core` configuration project.

```
Cattel-breed-main/
│
├── core/                         # Project configuration layer
│   ├── settings.py               # DB, auth, i18n, email, AI key, installed apps
│   ├── urls.py                   # Root URL dispatcher (all app namespaces)
│   ├── wsgi.py / asgi.py         # Server entry points
│   └── views.py                  # Minimal landing view fallback
│
├── accounts/                     # Authentication & identity management
│   ├── models.py                 # Custom User (email-first), EmailOTP
│   ├── views.py                  # Login, Signup, OTP Verify, Logout
│   └── urls.py                   # Auth routes + full password reset flow
│
├── recognition/                  # Core AI recognition engine
│   ├── ml_engine.py              # TensorFlow SavedModel loader + predictor
│   ├── models.py                 # BreedScan (scan record), DatasetImage
│   ├── views.py                  # upload_image: Gemini → fallback → save → notify
│   └── urls.py                   # /recognition/upload/
│
├── breeds/                       # Breed knowledge library & admin CRUD
│   ├── models.py                 # Breed (name, category, thumbnail, yield, video)
│   ├── forms.py                  # BreedForm with case-insensitive name validation
│   ├── views.py                  # List, detail, add, edit, delete (role-gated)
│   └── urls.py                   # /breeds/ with app_name='breeds'
│
├── portal/                       # User & admin dashboard orchestration
│   ├── views.py                  # user_dashboard, admin panels, AI lens API
│   └── urls.py                   # /dashboard/, /management/, /explorers/
│
├── history/                      # Scan history module
│   └── views.py                  # history_list (filter: high/low), history_detail
│
├── notifications/                # Real-time notification system
│   ├── models.py                 # Notification (typed: success/info/warning/feature)
│   ├── context_processors.py     # Global unread count injection
│   └── views.py                  # Notification list + mark-read
│
├── profiles/                     # User profile & farm information
│   ├── models.py                 # Profile (full_name, phone, farm_location, bio, avatar)
│   └── views.py                  # profile_view (with stats), profile_edit
│
├── reports/                      # Analytics & PDF export
│   └── views.py                  # report_dashboard (charts), download_report (PDF)
│
├── chatbot/                      # Conversational AI module
│   ├── models.py                 # ChatConversation (user, message, response, language)
│   └── views.py                  # chatbot_query (Gemini + markdown strip), full_chat_view
│
├── ml_models/                    # Trained model artifacts
│   ├── cattle_model_fixed.keras  # Keras format model
│   └── cattle_saved_model/       # TensorFlow SavedModel (used at runtime)
│       ├── saved_model.pb
│       └── variables/
│
├── templates/                    # Django HTML templates
│   ├── base1.html / base_portal.html
│   ├── landing.html
│   ├── accounts/                 # Login, Signup, OTP, verification
│   ├── recognition/              # Upload form, result page
│   ├── breeds/                   # Breed list, detail, add/edit, delete confirm
│   ├── dashboards/               # Admin dashboard, analytics, explorer directory
│   ├── portal/                   # User dashboard, AI lens, home, about
│   ├── history/                  # Scan history list and detail
│   ├── notifications/            # Notification list
│   ├── profiles/                 # Profile view and edit
│   ├── reports/                  # Report dashboard
│   ├── chatbot/                  # Chatbot UI
│   ├── partials/                 # Sidebar components (admin/user)
│   └── registration/             # Password reset flow templates
│
├── locale/                       # i18n translation files
│   ├── hi/LC_MESSAGES/           # Hindi .po/.mo
│   └── te/LC_MESSAGES/           # Telugu .po/.mo
│
├── media/                        # User-uploaded files (scans, thumbnails, avatars)
├── static/                       # Static assets (CSS, JS, images)
├── db.sqlite3                    # Development SQLite database
├── populate_data.py              # Database seeding script
├── populate_admin.py             # Admin account seeding script
├── translate_project.py          # i18n translation automation helper
├── req.txt                       # Project dependencies
├── .env                          # Environment variables (API key, email credentials)
└── manage.py                     # Django management entry point
```

**Request Data Flow:**

1. A farmer authenticates via email + password (OTP-verified account).
2. On the user dashboard, statistics are computed live from `BreedScan.objects.filter(user=request.user)`.
3. The farmer uploads a cattle image at `/recognition/upload/`. The `upload_image` view opens the image with Pillow, sends it to Gemini 2.5 Flash, parses the JSON response, and falls back to the local TensorFlow SavedModel if needed.
4. A `BreedScan` record is persisted with `is_validated=False`. A `Notification` record is created automatically.
5. The notification count is injected into every template response via the `notification_count` context processor.
6. The scan appears immediately in the farmer's history, dashboard statistics, and report charts.
7. Administrators can review unvalidated scans in the analytics panel, approving or discarding each record.

---

## Model Workflow

### Primary Recognition Pipeline (Gemini Vision)

```
Farmer Uploads Image
        │
        ▼
  PIL.Image.open() → convert("RGB")
        │
        ▼
  Gemini 2.5 Flash API Call
  ┌──────────────────────────────────────────────────┐
  │ Prompt: "You are an expert in Indian livestock.  │
  │ Return ONLY valid JSON: {breed, type, confidence}"│
  │ Contents: [prompt_text, PIL_image]               │
  └──────────────────────────────────────────────────┘
        │
        ▼
  extract_json() — strip markdown fences, regex match {}
        │
        ▼
  normalize_confidence() — handle %, 0–1 scale, clamp [0,100]
        │
        ▼
  Output: breed (str), type (str), confidence (int)
```

### Fallback Recognition Pipeline (Local TensorFlow SavedModel)

```
Gemini Failure or No JSON Returned
        │
        ▼
  predict_with_local_model(PIL.Image)
        │
        ▼
  Thread-safe singleton model loading
  (threading.Lock → tf.saved_model.load)
  _infer = model.signatures["serving_default"]
        │
        ▼
  preprocess_image():
  → convert("RGB") → resize(224, 224)
  → float32 array / 255.0
  → expand_dims → shape (1, 224, 224, 3)
        │
        ▼
  infer(tf.constant(img)) → numpy output
  np.argmax → class index
        │
        ▼
  CLASS_NAMES[idx]:
  [Gir, Sahiwal, Red Sindhi, Murrah Buffalo, Jersey, Holstein Friesian]
        │
        ▼
  cattle_type = "Buffalo" if "Buffalo" in breed else "Cattle"
        │
        ▼
  Output: {breed, type, confidence}
```

### Scan Persistence & Notification Pipeline

```
Recognition Output (from either pipeline)
        │
        ▼
  BreedScan.objects.create(
    user, image, breed_name, cattle_type, confidence_score
  )
        │
        ▼
  confidence_score_negative = -(364.4 × confidence/100)
  [SVG stroke-dashoffset for circular meter animation]
        │
        ▼
  Notification.objects.create(
    user, title="Recognition Complete",
    message=f"{breed} detected with {confidence}%",
    type="success"
  )
        │
        ▼
  Render: recognition/result.html
  (breed, type, confidence, animated SVG meter)
```

### PDF Report Generation Pipeline

```
User Requests Report Download (/reports/download/)
        │
        ▼
  BreedScan.objects.filter(user=request.user)
  → total_scans, unique breeds count
        │
        ▼
  ReportLab Canvas (A4)
  → Title: "Livestock Statistics Report"
  → Body: Total Scans, Unique Breeds
  → Footer: "Generated automatically by the system"
        │
        ▼
  HttpResponse(content_type='application/pdf')
  Content-Disposition: attachment; filename="livestock_report.pdf"
```

---

## Implementation Details

**Email-First Custom User Model:** The `accounts.User` model sets `username = None` and `USERNAME_FIELD = 'email'`. A `CustomUserManager` overrides `create_user()` and `create_superuser()` to normalize and validate the email field. `REQUIRED_FIELDS = []` makes email and password the only required credentials across all authentication flows.

**Thread-Safe Model Loading:** `recognition/ml_engine.py` implements a double-checked locking pattern using a module-level `threading.Lock()` and `_model` singleton. The model is loaded exactly once on first request, regardless of concurrent workers. The `serving_default` signature is extracted at load time and cached in `_infer`, eliminating per-request initialization overhead.

**Dual-SDK Gemini Integration:** The `recognition` and `portal` apps use the newer `from google import genai; client = genai.Client(...)` pattern. The `chatbot` app uses the legacy `import google.generativeai as genai; genai.configure(...)` approach. Both share the same `AI_API_KEY` environment variable loaded via `python-dotenv`.

**JSON Extraction with Regex Fallback:** The `extract_json()` helper strips markdown code fences, then applies `re.search(r"\{[\s\S]*?\}", cleaned)` to extract the first JSON object from Gemini's response. This handles cases where Gemini prepends explanatory text before the JSON payload, returning `None` on any failure to trigger the local model fallback.

**Confidence Normalization:** `normalize_confidence()` handles three output formats from Gemini: string values with a percent sign (`"85%"`), floats on a 0–1 scale (`0.85`), and integers on a 0–100 scale. The function strips the percent sign, casts to float, multiplies by 100 if ≤ 1, rounds to the nearest integer, and clamps to [0, 100].

**SVG Circular Confidence Meter:** The result template renders an animated SVG circle where arc length is controlled by `stroke-dashoffset`. The view computes `confidence_score_negative = -(364.4 × confidence / 100)` as a dynamic property on the `BreedScan` instance before template rendering. `364.4` is the circumference of the SVG circle, calibrated to map 100% confidence to a fully filled arc.

**Breed Form Duplicate Prevention:** `BreedForm.clean_name()` performs `Breed.objects.filter(name__iexact=name).exists()` to prevent administrators from registering duplicate breeds under alternate capitalizations (e.g., `GIR` and `Gir`).

**Report Chart Data Serialization:** The report dashboard view serializes breed labels and monthly scan arrays with `json.dumps()` and injects them as JSON strings into the template context. Chart.js on the frontend parses these strings to render pie and bar charts without additional API calls.

**Notification Context Processor:** The `notifications/context_processors.notification_count` function is registered globally in `TEMPLATES.OPTIONS.context_processors`. It queries `Notification.objects.filter(user=request.user, is_read=False).count()` for authenticated users, injecting `notification_count` into every template without requiring explicit view-level handling.

---

## Results

**AI Recognition Pipeline Performance:**

| Pipeline | Condition | Output |
|---|---|---|
| Gemini 2.5 Flash (Primary) | API available, image provided | Breed name, type, confidence % |
| Gemini 2.5 Flash (Primary) | Response lacks parseable JSON | Triggers local model fallback |
| Local TensorFlow SavedModel | API unavailable | Six-class argmax prediction |
| Combined System (Demo) | Demo dataset (85–99% scores) | ~94.5% average confidence |

**Breed Classification Coverage (Local Model):**

| Breed | Type |
|---|---|
| Gir | Cattle |
| Sahiwal | Cattle |
| Red Sindhi | Cattle |
| Murrah Buffalo | Buffalo |
| Jersey | Cattle |
| Holstein Friesian | Cattle |

**System Performance Characteristics:** Gemini 2.5 Flash responses for structured JSON breed identification are typically returned within 1–3 seconds. The TensorFlow SavedModel executes inference in under 200 milliseconds per image after the one-time initialization (2–5 seconds on first request). ReportLab PDF generation completes in under 100 milliseconds for all practical dataset sizes.

**Database Seeding:** The `populate_data.py` script creates five breeds, a demo farmer account, and twenty-five historical scan records in under two seconds, enabling immediate dashboard and analytics visualization for demonstration purposes.

---

## Installation

### Prerequisites

- Python 3.10 or higher
- `pip` package manager
- Git
- A valid Google Gemini API key (from `aistudio.google.com`)
- A Gmail account with App Password enabled for SMTP

### Step 1: Clone the Repository

```bash
git clone https://github.com/harisaigithub/Cattel-breed.git
cd Cattel-breed
```

### Step 2: Create and Activate a Virtual Environment

```bash
python -m venv venv

# On Windows
venv\Scripts\activate

# On macOS / Linux
source venv/bin/activate
```

### Step 3: Install Dependencies

```bash
pip install -r req.txt
```

Or install the core packages individually:

```bash
pip install Django==5.2.10 google-genai==1.61.0 google-generativeai tensorflow==2.10.1 \
    keras==2.10.0 numpy==1.23.5 Pillow==12.1.0 reportlab==4.4.9 \
    django-crispy-forms==2.5 crispy-bootstrap5==2025.6 python-dotenv==1.2.1
```

### Step 4: Configure Environment Variables

Create a `.env` file in the project root:

```env
AI_API_KEY=your_google_gemini_api_key_here
EMAIL_HOST_USER=your_gmail_address@gmail.com
EMAIL_HOST_PASSWORD=your_gmail_app_password
```

> **Note:** Generate a Gmail App Password by enabling 2-Factor Authentication on your Google account, then navigating to Security → App Passwords.

### Step 5: Apply Database Migrations

```bash
python manage.py makemigrations
python manage.py migrate
```

### Step 6: Compile Translation Files

```bash
python manage.py compilemessages
```

### Step 7: Create an Administrator Account

```bash
python manage.py createsuperuser
```

When prompted, provide an email address and password. Alternatively, run the included seeding script:

```bash
python populate_admin.py
```

### Step 8: Seed the Database with Demo Data (Optional)

```bash
python populate_data.py
```

This creates five breeds, a demo farmer account (`demo@farmer.com` / `demo1234`), and twenty-five historical scan records for immediate dashboard visualization.

### Step 9: Collect Static Files

```bash
python manage.py collectstatic
```

### Step 10: Run the Development Server

```bash
python manage.py runserver
```

Access the portal at `http://127.0.0.1:8000/`.

---

## Usage

### Registration and Login

1. Navigate to `http://127.0.0.1:8000/accounts/signup/` and register with your email, password, and role (Farmer or Administrator).
2. Enter the six-digit OTP sent to your email on the verification page within ten minutes.
3. Log in at `/accounts/login/` using your email and password.

### Recognizing a Cattle Breed

1. From the user dashboard, click **AI Lens** or navigate to `/recognition/upload/`.
2. Upload a clear photograph of the cattle or buffalo animal.
3. The system identifies the breed via Gemini Vision (with automatic local model fallback).
4. The result page displays the breed name, animal type, and an animated confidence score meter.
5. The scan is saved to your history and a success notification is generated automatically.

### Browsing the Breed Library

1. Navigate to `/breeds/` to access the searchable breed knowledge base.
2. Use the search bar to filter by breed name or origin location.
3. Use the category filter to show Cattle or Buffalo breeds only.
4. Click any breed card to view the full profile including description, milk yield, origin, and optional video reference.

### Viewing Scan History

1. Navigate to `/history/` for a chronological list of all recognition events.
2. Apply confidence filters: high (≥90%) or low (<70%).
3. Click any scan entry for the full detail view with the uploaded image.

### Generating a Report

1. Navigate to `/reports/` for the analytics dashboard with breed distribution and monthly trend charts.
2. Click **Download PDF Report** to receive a formatted A4 livestock statistics report.

### Administrative Operations

Log in with an admin or superuser account and access:

- `/portal/management/dashboard/` — System overview and global scan feed
- `/portal/management/analytics/` — Breed frequency charts and dataset validation
- `/breeds/admin/manage/` — Breed library CRUD interface
- `/portal/explorers/` — Explorer (user) directory with scan counts
- `/admin/` — Django admin for direct model-level management

### Changing the Interface Language

Use the language selector in the navigation bar to switch among English, Hindi, and Telugu. The selection persists as a cookie for one year.

---

## Future Enhancements

- **Expanded Breed Coverage:** Extend the local TensorFlow model to cover additional Indian breeds — Ongole, Kankrej, Tharparkar, Jaffarabadi, Surti — by acquiring and annotating additional training images per class.
- **Mobile Application:** Develop a cross-platform mobile client (Flutter or React Native) interfacing with a Django REST API, enabling breed recognition directly from a smartphone camera in rural field conditions.
- **REST API Layer:** Implement a Django REST Framework API with token authentication to expose breed recognition, scan history, and breed library endpoints for integration with third-party platforms and government livestock registries.
- **Continuous Learning Pipeline:** Integrate a retraining loop where administrator-validated `BreedScan` records are automatically added to the training dataset, improving model accuracy over time as field images accumulate.
- **Production Database Migration:** Transition from SQLite to PostgreSQL for concurrent multi-user production deployments, with connection pooling and ACID transaction support.
- **Containerized Deployment:** Package the application using Docker and Docker Compose — separating the Django application server, TensorFlow inference layer, and database — for reproducible cloud deployment.
- **Government Scheme Integration:** Map identified breed profiles to applicable central and state livestock support schemes (PM Kisan, Rashtriya Gokul Mission), providing farmers with personalized scheme recommendations based on their scan history.
- **Veterinary Telemedicine Module:** Integrate video/chat consultation booking connecting farmers with registered veterinarians, using the breed scan result as pre-session context.
- **Geospatial Herd Mapping:** Add farm location–based herd density visualization using a map interface, enabling administrators to identify regional breed concentrations and target outreach programs geographically.
- **Offline Scan Queueing (PWA):** Convert the portal into a Progressive Web App supporting offline image capture and queued scan synchronization when connectivity is restored — critical for rural field use.

---

## Contributing

Contributions are welcome. Please follow the standard Git workflow:

1. Fork the repository on GitHub.
2. Create a feature branch from `main`:
   ```bash
   git checkout -b feature/your-feature-name
   ```
3. Implement your changes with clear, documented code.
4. Write or update tests where applicable.
5. Commit with a descriptive message:
   ```bash
   git commit -m "feat: describe the feature or fix"
   ```
6. Push to your fork and open a Pull Request against `main`, describing the change, motivation, and testing performed.

Please adhere to PEP 8 style guidelines. Do not commit hardcoded API keys, email credentials, or secret keys — all sensitive configuration must remain in the `.env` file, which is excluded from version control via `.gitignore`.

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

For technical queries, deployment assistance, academic collaboration, or project customization inquiries, please reach out through the following channels:

**GitHub:** [https://github.com/harisaigithub](https://github.com/harisaigithub)

**Email:** [harisaiparasa@gmail.com](mailto:harisaiparasa@gmail.com)

---

&nbsp;

&nbsp;

## ***To acquire this project, please contact***

**GitHub:** https://github.com/harisaigithub  
**Email:** [harisaiparasa@gmail.com](mailto:harisaiparasa@gmail.com)  

For collaborations, customizations, or deployment support, feel free to reach out.