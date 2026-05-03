# 🐄 CattleBreed AI — Intelligent Livestock Breed Recognition System

> **An AI-powered web platform for real-time Indian cattle and buffalo breed identification using Google Gemini Vision API and a custom TensorFlow deep learning model — built for farmers, researchers, and livestock management professionals.**

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

CattleBreed AI is a full-stack intelligent web application designed to automate the identification and classification of Indian cattle and buffalo breeds from photographic images. The system addresses a significant gap in modern livestock management: the inability to quickly and accurately identify animal breeds without expert veterinary involvement.

India hosts one of the largest bovine populations in the world, encompassing numerous indigenous breeds — such as Gir, Sahiwal, and Red Sindhi — each carrying distinct agronomic traits including milk yield, disease resistance, and climate adaptability. Misidentification of these breeds leads to suboptimal breeding decisions, reduced productivity, and loss of genetic heritage. CattleBreed AI solves this by combining the power of Google Gemini Vision (generative AI) with a locally trained TensorFlow SavedModel, delivering instant, confidence-scored breed predictions through an intuitive web interface.

The platform is designed for two primary user categories: farmers and livestock owners who need rapid breed identification in the field, and administrators who oversee breed data, user activity, and AI model validation.

---

## Objectives

- Develop an end-to-end web application capable of identifying six major Indian cattle and buffalo breeds from user-submitted images.
- Integrate Google Gemini Vision API (`gemini-2.5-flash`) as the primary AI inference engine with structured JSON response parsing.
- Build and deploy a custom TensorFlow SavedModel as a secondary/offline inference fallback.
- Implement a role-based access control system differentiating farmers/users from administrators.
- Provide each user with a comprehensive personal dashboard featuring weekly, monthly, and yearly scan analytics.
- Maintain a persistent scan history with filterable views and downloadable individual scan reports.
- Enable administrators to manage the breed database, validate AI-generated dataset images, and monitor platform-wide usage analytics.
- Implement secure email-based OTP verification for all new user registrations.
- Generate PDF-based livestock statistical reports using ReportLab.
- Deliver a notification system that alerts users upon completion of each breed recognition scan.

---

## Summary

CattleBreed AI is a Django-based multi-app web platform that accepts cattle and buffalo images from authenticated users and returns an AI-generated breed classification with a confidence score. The recognition pipeline first invokes the Google Gemini Vision API; if that fails, it falls back to a locally deployed TensorFlow SavedModel trained on six breed categories. Each scan result is stored in the database, linked to the user profile, and immediately reflected in the user's analytics dashboard.

The application is modularized into nine Django apps: `accounts`, `portal`, `recognition`, `breeds`, `history`, `reports`, `notifications`, and `profiles`, each owning a discrete domain of the system. The admin panel provides real-time statistics, dataset review queues, user management, and global scan monitoring. The entire stack runs on Django with SQLite, is template-driven using Bootstrap 5, and is deployable via standard WSGI/ASGI servers.

---

## Features

### Before Login (Public Access)

- **Landing Page** — A publicly accessible marketing/informational landing page introducing the platform, its capabilities, and a call-to-action to register or log in.
- **User Registration** — Email-based account creation with role selection (Farmer/Administrator). Registration triggers OTP delivery via SMTP for email verification.
- **OTP Verification** — Time-bound (10-minute) six-digit OTP verification step before account activation.
- **Login & Role-Based Routing** — Credential-based login that redirects farmers to the user dashboard and administrators to the admin control panel.
- **Password Reset** — Full Django built-in password reset flow via email.

### After Login — Farmer/User Portal

- **User Dashboard** — Personalized analytics hub displaying total scans, average AI confidence score, success rate, unique breeds identified, and a cattle-vs-buffalo distribution chart. Includes weekly (7-day bar), monthly (4-week), and yearly (12-month) activity visualizations.
- **AI Lens (Breed Recognition)** — Image upload interface where users submit cattle photos for instant AI-powered breed identification. Displays breed name, category (Cattle/Buffalo), and confidence percentage on the result page.
- **Scan History** — Chronological list of all past scans with filter options for high-confidence (≥90%) and low-confidence (<70%) results. Each entry links to a detailed scan view.
- **Scan Detail View** — Displays scan image, breed name, confidence score, AI source, processing time, and a derived health/quality status label (Excellent / Good / Moderate / Needs Review).
- **Download Scan Report** — Per-scan downloadable plain-text report containing breed name, category, confidence score, health status, and scan timestamp.
- **Breed Encyclopedia** — Searchable and category-filterable directory of all active breed entries. Each breed page shows description, origin, milk yield, milk fat percentage, coat color, traits list, and an optional video reference link.
- **Statistical Reports** — A dedicated reports dashboard with breed-distribution pie chart and monthly scan bar chart rendered via Chart.js, plus a one-click PDF export generated by ReportLab.
- **Notifications** — In-app notification feed that automatically creates an entry upon each completed recognition scan. Supports notification types: success, info, warning, and feature.
- **Profile Management** — Editable user profile page with full name, phone number, farm location, bio, and avatar upload.

### After Login — Administrator Panel

- **Admin Dashboard** — System-wide overview displaying total registered users, total scans platform-wide, total breed entries, system status, model version, and a feed of the most recent scans.
- **Breed Management** — Full CRUD interface for the breed database: add, edit, delete, and list all breed entries. Each form supports thumbnail upload, production metric fields, trait tags, and optional video URL.
- **Analytics Panel** — Breed-level scan distribution visualization with a table of recent unvalidated scans and dataset counters (total, validated, pending).
- **Dataset Review Queue** — Lists all unvalidated scan images for human review. Admin can validate (approve for training use) or discard individual entries.
- **User Directory with Status Toggle** — Displays all users with activation status. Admins can activate or deactivate any non-superuser account.
- **AI Monitor** — Dedicated view for monitoring AI inference activity.
- **Bulk Upload Interface** — Supports bulk addition of training images to the dataset.

---

## Technologies Used

### Programming Languages
- Python 3.10

### Libraries & Frameworks
- Django 4.x — Core web framework, ORM, URL routing, middleware, admin
- django-crispy-forms + crispy-bootstrap5 — Form rendering with Bootstrap 5 styling
- Pillow (PIL) — Server-side image loading, conversion, and preprocessing
- python-dotenv — Environment variable management from `.env` file
- ReportLab — Programmatic PDF generation for livestock statistical reports

### Machine Learning / AI
- TensorFlow 2.x (SavedModel format) — Custom-trained deep learning model for cattle breed classification
- Google Generative AI SDK (`google-generativeai`) — Python client for Gemini Vision API (`gemini-2.5-flash`)
- NumPy — Numerical operations for image array preprocessing and argmax prediction extraction

### Backend / Frontend
- Django template engine — HTML rendering with Jinja-style template inheritance
- Bootstrap 5 — Responsive UI framework across all portal and admin templates
- Chart.js — Client-side rendering of pie charts and bar charts in dashboards and reports
- CSRF middleware, SecurityMiddleware, ClickjackingMiddleware — Django's built-in security stack

### Database
- SQLite3 — Default relational database for development (configurable for PostgreSQL in production)

### Deployment & DevOps
- Django WSGI (`core/wsgi.py`) and ASGI (`core/asgi.py`) — Production server interface support
- `manage.py` — Django management command entry point
- `populate_data.py` / `populate_admin.py` — Seed scripts for initial breed and admin data population

### Security
- Custom `AbstractUser` model with email as the primary identifier (username removed)
- Email OTP verification for new account activation
- Role-based access decorators (`@login_required`, `@staff_member_required`, `@user_passes_test`)
- Django CSRF protection on all POST endpoints
- Session-based OTP state management (`request.session['verify_user']`)
- SMTP email backend for transactional mail (OTP delivery, password reset)
- Environment variable isolation of API keys and credentials via `.env`

---

## Dataset

### Source
The system is trained to recognize six breed categories representing both indigenous Indian and commercially significant bovine breeds: Gir, Sahiwal, Red Sindhi, Murrah Buffalo, Jersey, and Holstein Friesian. Training images were sourced from publicly available livestock databases and curated domain-specific image collections.

### Structure
Images are stored in the `ml_models/cattle_saved_model/` directory as a TensorFlow SavedModel, with a `serving_default` signature used for inference. Dataset images for ongoing retraining are stored in the database under `DatasetImage` records (linked to individual breed entries) and uploaded to `media/dataset/training/`.

### Preprocessing
Each input image undergoes the following preprocessing pipeline before inference:
- Conversion to RGB color mode (strips alpha channels and handles grayscale inputs)
- Resizing to 224×224 pixels (standard input resolution for the model)
- Normalization to the [0.0, 1.0] floating-point range by dividing pixel values by 255
- Expansion of the array along axis 0 to produce a batch dimension of shape `(1, 224, 224, 3)`

### Usage
The preprocessed NumPy array is passed to the TensorFlow `serving_default` inference function. The output tensor is converted to a NumPy array, and `argmax` identifies the predicted class index. The confidence score is derived as `predictions[idx] * 100`, rounded to two decimal places. An `ai_source` field on each scan record tracks whether the result originated from Gemini or the local model.

---

## System Architecture

The application follows a modular monolithic architecture with clear separation of concerns across nine Django applications, each responsible for a distinct business domain.

```
CattleBreed AI
│
├── core/                  → Project settings, root URL configuration, WSGI/ASGI entry points
├── accounts/              → Custom User model (email-based), OTP logic, signup, login, logout
├── portal/                → Landing page, user dashboard, admin dashboard, dataset validation
├── recognition/           → Image upload view, Gemini AI integration, ML inference engine, BreedScan model
├── breeds/                → Breed database model, CRUD views, breed directory for users
├── history/               → Scan history views (list, detail, delete), per-scan report download
├── reports/               → Statistical report dashboard, Chart.js data APIs, PDF export via ReportLab
├── notifications/         → Notification model, context processor for unread count, list view
├── profiles/              → User profile model (farm info, bio, avatar), profile edit/view
│
├── ml_models/
│   └── cattle_saved_model/ → Trained TensorFlow SavedModel for offline breed inference
│
└── templates/             → Django HTML templates organized per application
```

**Data Flow:** A user submits an image through the upload form → `recognition/views.py` opens the image via Pillow → the Gemini Vision API is called with a structured prompt → the JSON response is parsed into breed name, type, and confidence → a `BreedScan` record is created and linked to the user → a `Notification` record is created → the result is rendered to the template. All analytics views query the `BreedScan` table using Django ORM aggregation functions.

---

## Model Workflow

### Step 1 — Preprocessing
The uploaded image file from the multipart form submission is opened using `PIL.Image.open()` and converted to RGB. For the local TensorFlow model, the image is resized to `(224, 224)` and normalized to float32 in `[0.0, 1.0]`. A batch dimension is added with `np.expand_dims`.

### Step 2 — Training
The TensorFlow model was trained offline on categorized cattle and buffalo images across six breed classes. The trained model was exported in TensorFlow's `SavedModel` format and stored in `ml_models/cattle_saved_model/`. The model exposes a `serving_default` signature compatible with `tf.saved_model.load()` and `.signatures["serving_default"]`.

### Step 3 — Evaluation
Model performance was evaluated using classification accuracy across the six breed classes. The confidence score produced during inference serves as a live quality indicator: scores ≥90% are labeled "Excellent", 75–89% as "Good", 60–74% as "Moderate", and below 60% as "Needs Review".

### Step 4 — Prediction
For every submitted image, the primary inference path calls `analyze_with_gemini()` in `recognition/gemini_service.py`. The Gemini `gemini-2.5-flash` model receives the PIL image and a structured prompt instructing it to return ONLY a JSON object containing `breed`, `type`, and `confidence`. The response text is searched with a regex pattern to extract the JSON block, which is then parsed and validated. If the Gemini call fails or returns no valid data, the local TensorFlow model is invoked via `predict_with_local_model()` in `recognition/ml_engine.py` as a fallback.

### Step 5 — Integration
The prediction result dictionary (`breed`, `type`, `confidence`) is persisted as a `BreedScan` record in the database with user association, image file path, AI source identifier, and millisecond-precision processing time. The result is immediately surfaced to the user in the recognition result template, logged in scan history, and reflected in all dashboard analytics.

---

## Implementation Details

**Custom User Authentication:** The default Django `User` model was replaced with a custom `AbstractUser` subclass that uses email as the primary login identifier (username field removed). Users are assigned one of two roles: `farmer` or `admin`. On login, role-based redirection routes farmers to the user portal and admins to the admin dashboard.

**Email OTP Registration:** On signup, the user account is created with `is_active=False`. A six-digit OTP is generated using Python's `random.randint`, stored in the `EmailOTP` model with a creation timestamp, and dispatched via Django's SMTP backend. The user must submit the correct OTP within a 10-minute expiry window. Upon verification, `is_active` and `is_verified` are set to `True` and the OTP record is deleted.

**Dual AI Inference Strategy:** The recognition module implements a try/except chain: Gemini Vision is attempted first (primary), and the TensorFlow SavedModel is available as a secondary path. Thread-safe lazy loading of the TensorFlow model is implemented using a module-level `threading.Lock`, ensuring the model is loaded only once regardless of concurrent requests.

**Dashboard Analytics:** The user dashboard computes weekly, monthly, and yearly scan activity using Django ORM aggregation (`Count`, `Avg`) and date range filtering. Each activity period is normalized to a percentage of the maximum value for proportional bar chart rendering without a JavaScript charting library.

**Admin Dataset Validation:** Every `BreedScan` record carries an `is_validated` boolean flag (default: `False`). Admins can review unvalidated scan images in the Dataset Review Queue and either validate them (marking them suitable as training data) or discard them (deleting the record). This creates a human-in-the-loop pipeline for continuous model improvement.

**PDF Report Generation:** The reports module uses ReportLab's `canvas` API to dynamically generate an A4-format PDF containing the user's aggregated livestock statistics. The PDF is streamed directly as an `HttpResponse` with `Content-Disposition: attachment` to trigger a browser download.

**Notification System:** A context processor (`notifications/context_processors.py`) is registered globally and injects the unread notification count into every request context for display in the navigation bar. Notifications are created programmatically at the point of scan completion.

---

## Results

The system achieved the following performance characteristics based on testing with the deployed dual-engine inference pipeline:

| Metric | Value |
|---|---|
| Supported Breed Classes | 6 (Gir, Sahiwal, Red Sindhi, Murrah Buffalo, Jersey, Holstein Friesian) |
| Primary AI Engine | Google Gemini Vision (`gemini-2.5-flash`) |
| Fallback AI Engine | Custom TensorFlow SavedModel |
| Image Input Resolution | 224 × 224 pixels (local model) |
| Confidence Score Range | 0 – 100% |
| High-Confidence Threshold | ≥ 90% (labeled "Excellent") |
| AI Source Tracking | Per-scan field: `gemini` or `local` |
| Processing Time Logging | Millisecond-precision per scan |
| PDF Report Generation | ReportLab A4 format with summary statistics |
| Dataset Validation Pipeline | Human-reviewed queue with approve/discard actions |

The Gemini Vision API consistently returns well-structured JSON with breed names and confidence scores for clearly visible cattle images. The local TensorFlow model serves as a reliable fallback, ensuring the system remains functional under API unavailability. Confidence scores below 70% trigger a "Needs Review" status, prompting users to submit a clearer image for improved accuracy.

---

## Installation

### Prerequisites
- Python 3.10 or later
- pip
- Git
- A valid Google Gemini API key (obtainable from [Google AI Studio](https://aistudio.google.com/))
- SMTP email credentials for OTP delivery

### Step 1 — Clone the Repository

```bash
git clone https://github.com/harisaigithub/CattleBreed2.git
cd CattleBreed2
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
pip install django pillow python-dotenv google-generativeai tensorflow numpy reportlab django-crispy-forms crispy-bootstrap5
```

Or, if a `req.txt` / `requirements.txt` file is present:

```bash
pip install -r req.txt
```

### Step 4 — Configure Environment Variables

Create a `.env` file in the project root with the following contents:

```env
AI_API_KEY=your_google_gemini_api_key_here
EMAIL_HOST_USER=your_email@gmail.com
EMAIL_HOST_PASSWORD=your_app_password_here
```

> **Note:** Use a Gmail App Password (not your account password) when using Gmail as the SMTP provider. Enable 2FA on your Google account and generate an App Password from Account Settings.

### Step 5 — Apply Database Migrations

```bash
python manage.py makemigrations
python manage.py migrate
```

### Step 6 — Create a Superuser (Admin Account)

```bash
python manage.py createsuperuser
```

Follow the prompts to set the admin email and password.

### Step 7 — Populate Seed Data (Optional)

```bash
python populate_data.py
python populate_admin.py
```

### Step 8 — Collect Static Files (for Production)

```bash
python manage.py collectstatic
```

### Step 9 — Run the Development Server

```bash
python manage.py runserver
```

Access the application at `http://127.0.0.1:8000/`.

---

## Usage

### As a Farmer/User

1. Navigate to `http://127.0.0.1:8000/` and click **Sign Up**.
2. Enter your email address, choose the **Farmer** role, and set a password.
3. Check your email inbox for the six-digit OTP and enter it on the verification page.
4. After successful verification, log in to access your personal dashboard.
5. Click **AI Lens** or navigate to `/recognition/` to upload a cattle or buffalo image.
6. View the recognized breed name, type, and confidence score on the result page.
7. Access **History** at `/history/` to review all past scans with filter options.
8. Visit **Reports** at `/reports/` to view breed distribution charts and download a PDF livestock report.
9. Click the **bell icon** to review recognition notifications.
10. Edit your profile at `/profiles/` to add farm location and personal details.

### As an Administrator

1. Log in with superuser credentials. The system automatically redirects to the Admin Dashboard at `/dashboard/admin/`.
2. Navigate to **Breed Management** at `/breeds/manage/` to add, edit, or remove breed entries.
3. Review the **Analytics Panel** at `/dashboard/analytics/` to monitor platform-wide scan distribution.
4. Process the **Dataset Review Queue** at `/dashboard/dataset-review/` to validate or discard unvalidated scan images.
5. Access the **Django Admin Panel** at `/admin/` for raw model-level data management.

---

## Future Enhancements

- **Expanded Breed Coverage** — Extend the classification model to support additional indigenous Indian breeds such as Ongole, Tharparkar, Kankrej, and Nili-Ravi Buffalo.
- **Mobile Application** — Develop a React Native or Flutter mobile app enabling real-time camera-based breed scanning directly from the field.
- **Model Retraining Pipeline** — Build an automated pipeline that uses admin-validated dataset images to periodically fine-tune the TensorFlow model and deploy the updated SavedModel without downtime.
- **Multi-Language Support** — Add Hindi and regional language (Telugu, Tamil, Marathi) support using Django's internationalization framework to improve accessibility for rural farmers.
- **RESTful API Layer** — Expose breed recognition as a JSON REST API using Django REST Framework, enabling third-party integrations with veterinary platforms and government livestock portals.
- **PostgreSQL Migration** — Transition from SQLite to PostgreSQL for production-grade concurrency, indexing, and scalability.
- **Cloud Deployment** — Containerize the application using Docker and deploy to AWS EC2, Google Cloud Run, or Heroku with Gunicorn and Nginx.
- **Health Advisory Module** — Integrate a breed-specific advisory engine that recommends feeding schedules, vaccination timelines, and productivity benchmarks based on the identified breed.
- **Geolocation Tagging** — Capture and store GPS coordinates with each scan to enable regional livestock distribution mapping and herd movement analytics.
- **Advanced Analytics Dashboard** — Incorporate heatmaps, trend forecasting, and breed-level performance comparisons using a richer visualization library such as Plotly or D3.js.

---

## Contributing

Contributions are welcome. Please follow the standard Git workflow outlined below.

1. **Fork** the repository on GitHub.
2. **Clone** your fork locally:
   ```bash
   git clone https://github.com/your-username/CattleBreed2.git
   cd CattleBreed2
   ```
3. **Create a feature branch** from `main`:
   ```bash
   git checkout -b feature/your-feature-name
   ```
4. **Make your changes**, ensuring code quality and consistency with the existing codebase.
5. **Commit** with a descriptive message:
   ```bash
   git add .
   git commit -m "feat: add your feature description"
   ```
6. **Push** your branch to your fork:
   ```bash
   git push origin feature/your-feature-name
   ```
7. **Open a Pull Request** against the `main` branch of the original repository with a clear description of the changes made and the problem solved.

Please ensure that any new functionality is accompanied by appropriate test coverage and that all existing tests pass before submitting a pull request.

---

## License

This project is licensed under the **MIT License**.

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
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
```

---

## Contact

For questions, issues, or usage inquiries, please open a GitHub Issue on the repository.


&nbsp;

&nbsp;

## ***To acquire this project, please contact***

**GitHub:** https://github.com/harisaigithub

**Email:** [harisaiparasa@gmail.com](mailto:harisaiparasa@gmail.com)

For collaborations, customizations, or deployment support, feel free to reach out.