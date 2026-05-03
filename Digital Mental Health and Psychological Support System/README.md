# Digital Mental Health and Psychological Support System

### *An AI-Powered, Role-Based Web Platform for Student Mental Wellness, Clinical Assessments, and Confidential Counselling*

---

## Table of Contents

1. [Introduction](#introduction)
2. [Objectives](#objectives)
3. [Summary](#summary)
4. [Features](#features)
5. [Technologies Used](#technologies-used)
6. [System Architecture](#system-architecture)
7. [Model Workflow](#model-workflow)
8. [Implementation Details](#implementation-details)
9. [Results](#results)
10. [Installation](#installation)
11. [Usage](#usage)
12. [Future Enhancements](#future-enhancements)
13. [Contributing](#contributing)
14. [License](#license)
15. [Contact](#contact)

---

## Introduction

Mental health among students remains a critically underserved domain. A large proportion of undergraduate and postgraduate students experience anxiety, depression, and psychological distress, yet they lack convenient, confidential, and technically accessible channels to seek support. Traditional counselling systems are burdened with stigma, logistical friction, and limited outreach.

The **Digital Mental Health and Psychological Support System (DMH)** addresses this gap by providing a unified, web-based platform built on Django that integrates clinical mental health screening tools, a context-aware AI chatbot powered by Google Gemini, structured appointment booking, a curated resource library, and multi-role administrative dashboards — all under a privacy-first architecture with OTP-based authentication and anonymous identifier support.

The system is designed for deployment in educational institutions and is capable of running on Hugging Face Spaces, Docker containers, or conventional server environments.

---

## Objectives

- Provide clinically validated self-assessment tools (PHQ-9 and GAD-7) accessible to students with and without account login, while preserving privacy via anonymous identifiers.
- Develop an empathetic, context-aware AI chatbot capable of detecting mental health crises and directing users to emergency resources in real time.
- Implement a structured counsellor appointment booking system with tri-party email notifications (student, counsellor, and administrator).
- Build role-segregated dashboards for students, counsellors, peer volunteers, and administrators with relevant analytics and management controls.
- Enforce an institutional approval workflow for counsellor and peer volunteer accounts to ensure professional accountability.
- Support exportable assessment reports in PDF format for clinical tracking and record-keeping.
- Enable a counsellor-managed mental wellness resource library with severity-tagged filtering.
- Deploy the system as a containerised application using Docker and Gunicorn, ready for both on-premise and cloud-hosted environments.

---

## Summary

The Digital Mental Health and Psychological Support System is a full-stack Django web application that provides end-to-end mental wellness support for students. It combines established psychiatric screening instruments with modern AI capabilities to deliver a holistic, student-centric platform.

At its core, the system features four primary user roles — Student, Counsellor, Peer Volunteer, and Administrator — each granted access to a tailored dashboard and set of capabilities. Students can log daily mood check-ins, take PHQ-9 (depression) and GAD-7 (anxiety) assessments, converse with an AI chatbot that reads their recent mood and assessment history as context, book confidential counselling sessions, and access curated mental wellness resources. Counsellors can view booked appointments, submit post-session guidance messages, manage the resource library, and review anonymised assessment data. Peer volunteers participate in community-building activities. Administrators oversee user management, approve counsellor/peer accounts, and access platform-wide analytics including assessment severity distributions and appointment statistics.

The platform is deployed on Hugging Face Spaces using Gunicorn as the WSGI server and WhiteNoise for static file serving, making it production-ready without a separate web server such as Nginx.

---

## Features

### Before Login (Public Access)

- **Landing Page** — Informational home page introducing the platform's purpose and capabilities.
- **User Registration** — Students, counsellors, and peer volunteers can self-register with role selection. Counsellor and peer accounts require admin approval before login is granted.
- **OTP Email Verification** — Upon registration, a six-digit time-limited OTP is dispatched to the user's email. Accounts remain inactive until the OTP is verified, preventing unauthorised access.
- **Password Reset** — Secure password reset flow via email-based tokenised links using Django's built-in password reset framework.
- **Anonymous Self-Assessments** — The PHQ-9 and GAD-7 tools are accessible without authentication, with results stored against a randomly generated UUID for privacy preservation.

### After Login — Student Role

- **Student Dashboard** — Displays a dynamic mood trend chart (Chart.js), recent mood entries, and a quick emoji-based mood check-in (Sad / Okay / Good / Great mapped to numeric scores 20, 50, 80, 100).
- **Daily Mood Logging** — Students log their emotional state via a single-click emoji interface; all entries are persisted and graphed over time.
- **PHQ-9 Assessment** — Nine-question Patient Health Questionnaire scored 0–27, categorised into Minimal, Mild, Moderate, Moderately Severe, and Severe depression bands.
- **GAD-7 Assessment** — Seven-question Generalised Anxiety Disorder scale scored 0–21, categorised into Minimal, Mild, Moderate, and Severe anxiety bands.
- **Assessment History** — Full chronological history of all completed assessments with severity classifications.
- **PDF Report Export** — Students can download a formatted PDF report of all past assessments generated using ReportLab.
- **AI Mental Wellness Chatbot** — Contextual conversational AI powered by Google Gemini. The chatbot receives the student's recent mood history, assessment scores, and appointment data as structured context to personalise responses. Emergency keywords (e.g., "suicide", "hurt myself", "end my life") trigger immediate escalation messaging with the India Mental Health Helpline number.
- **Counsellor Appointment Booking** — Students can browse approved counsellors, select a date and time slot, and book a confidential session. Tri-party email notifications are dispatched automatically to the student, selected counsellor, and all platform administrators.
- **Mental Wellness Resources** — Browse a categorised and severity-tagged library of guides, videos, and audio content curated by counsellors.
- **Community Links** — Access peer-support community forms and group resources managed by staff.
- **In-App Notifications** — System notifications for appointment updates and administrative actions, rendered via a context processor integrated across all pages.

### After Login — Counsellor Role

- **Counsellor Dashboard** — Overview of upcoming and past appointments, student counts, and session statistics.
- **Student Appointment List** — View all appointments booked with the counsellor, with the ability to update status to Completed and add a personalised post-session guidance message for the student.
- **Resource Management** — Full CRUD interface for creating and managing mental wellness resources by category (Anxiety, Depression, Stress, Sleep, Exam Stress, General) and severity targeting.
- **Assessment Overview** — Review anonymised student assessment data to understand platform-wide mental health trends.
- **Ethics Guidelines** — Dedicated page presenting professional ethics standards for counsellors.

### After Login — Peer Volunteer Role

- **Community Management** — Peer volunteers can create and manage community support links and group activity forms.

### After Login — Administrator Role

- **Admin Dashboard** — Platform-wide statistics including total users, pending approvals, total assessments, appointment counts (booked vs. completed), and recent activity logs.
- **User Management** — Full CRUD for all user accounts: create users, approve counsellor/peer registrations, and deactivate or delete accounts.
- **User Edit** — Update role assignments and account status for existing users.
- **Assessment Analytics** — Visual breakdown of assessment severity distribution across the student population.
- **Community & Resource Management** — Administrative control over all community links and resource entries.
- **PDF Analytics Export** — Download a formatted system analytics report in PDF format.

---

## Technologies Used

### Programming Languages
- Python 3.11

### Libraries and Frameworks
- Django 5.2.10
- Django REST Framework 3.16.1
- Whitenoise 6.8.2
- ReportLab 3.6.12
- BeautifulSoup4 4.14.3
- NLTK 3.9.2
- Pillow 12.1.0
- python-dotenv 1.2.1
- Tenacity 9.1.2

### Artificial Intelligence / AI Integration
- Google Generative AI SDK (`google-genai` 1.57.0)
- Google GenerativeAI (`google-generativeai` 0.8.6)
- Google AI Generative Language API 0.6.15
- Context-aware prompt engineering with student mood, assessment, and appointment data
- Emergency keyword detection with crisis escalation logic

### Backend
- Django MVT (Model-View-Template) architecture
- Django ORM with SQLite
- Django Channels / ASGI support via ASGIRef 3.11.0 and WebSockets 15.0.1
- Django Sessions and Authentication
- Django Sites Framework (SITE_ID configuration)
- Custom management command: `createdefaultadmin`
- SMTP email backend for OTP delivery, appointment notifications, and password resets

### Frontend
- Django Template Language (DTL) with Jinja2-style block inheritance
- Chart.js (mood trend visualisations)
- HTML5 / CSS3 / JavaScript
- django-leaflet 0.33.0 (geospatial map support)

### Database
- SQLite (default, production-ready for single-instance deployments)

### Deployment and DevOps
- Docker (multi-stage `python:3.11-slim` image)
- Gunicorn 23.0.0 (WSGI production server, 2 workers, 120s timeout)
- Hugging Face Spaces (primary cloud deployment target)
- WhiteNoise 6.8.2 (static file serving without Nginx)
- `.env`-based environment configuration via `python-dotenv`

### Security
- OTP-based email verification on account registration
- Role-based access control enforced at the view layer via `@login_required` and role guards
- Admin approval workflow for counsellor and peer volunteer accounts
- UUID-based anonymous identifiers for unauthenticated assessment submissions
- CSRF protection enabled globally
- Tokenised password reset via Django's built-in secure reset framework
- Non-root Docker user (`appuser`) for container security

---

## System Architecture

The DMH platform follows a monolithic Django MVT architecture composed of seven discrete Django applications, each encapsulating its own models, views, URLs, templates, and migration history.

```
config/                     # Project settings, root URLconf, WSGI/ASGI entry points
├── settings.py             # Centralised configuration with dotenv support
├── urls.py                 # Root URL dispatcher routing to all sub-applications
├── wsgi.py / asgi.py       # Server gateway interfaces

accounts/                   # Authentication, user model, OTP, notifications
assessments/                # PHQ-9 & GAD-7 tools, scoring logic, PDF export
chatbot/                    # AI chatbot, session/message persistence, Gemini integration
appointments/               # Booking, tri-party email notifications, guidance messages
resources/                  # Mental wellness library, community links
dashboard/                  # Role-segmented dashboards, admin analytics, PDF export
pages/                      # Public-facing landing pages
```

**Data Flow:**

1. HTTP requests arrive at Gunicorn (production) or the Django development server.
2. The root URLconf dispatches requests to the appropriate application URL module.
3. Views enforce authentication and role checks before processing business logic.
4. The Django ORM communicates with SQLite for all persistence operations.
5. Templates inherit from `base.html` / `base2.html`, which inject notification context via a registered context processor.
6. The AI chatbot integrates outbound API calls to Google Gemini, enriched with a structured student context block assembled from ORM queries.
7. Completed responses are rendered via DTL templates and served through WhiteNoise-managed static assets.

---

## Model Workflow

### Assessment Pipeline

1. **Selection** — The user navigates to the assessment home and selects PHQ-9 (depression) or GAD-7 (anxiety).
2. **Questionnaire** — The user responds to 9 or 7 validated clinical questions on a 0–3 Likert scale.
3. **Scoring** — Answers are summed server-side. Severity is classified using clinical threshold functions (`get_phq9_severity`, `get_gad7_severity`).
4. **Persistence** — The result is stored in the `SelfAssessment` model with user FK (or anonymous UUID for unauthenticated access) and a timestamp.
5. **Result Display** — The score, severity band, and clinical interpretation are rendered on the result page.
6. **PDF Export** — The user may download a formatted multi-assessment history report via ReportLab.

### AI Chatbot Pipeline

1. **Session Initialisation** — A `ChatSession` record is created or retrieved for the authenticated (or anonymous) user.
2. **Context Assembly** — `_build_student_context()` queries DailyMood, SelfAssessment, and Appointment tables to construct a structured context block for the Gemini prompt.
3. **Memory Construction** — `_build_chat_memory()` retrieves the last six messages from the current session to provide conversational continuity.
4. **Emergency Screening** — The user's message is scanned against a predefined set of crisis keywords before dispatch to the AI.
5. **Prompt Construction** — The system prompt, tone instructions, student context, chat memory, and user message are assembled into the Gemini API payload.
6. **AI Response** — The Gemini model returns a response, which is extracted, sanitised, and stored as a `ChatMessage`.
7. **Crisis Escalation** — If emergency keywords are detected, a hard-coded crisis response appending the India Mental Health Helpline (9152987821) is injected regardless of the AI output.
8. **Persistence** — Both the user message and the bot response are saved to the `ChatMessage` table linked to the active session.

### Appointment Workflow

1. **Counsellor Selection** — The student views a list of approved counsellors and selects one.
2. **Booking** — The student selects a date and time slot and submits the form.
3. **Record Creation** — An `Appointment` record is created with status `BOOKED`.
4. **Tri-party Notification** — SMTP emails are dispatched concurrently to the student, counsellor, and all admin accounts.
5. **Guidance** — After the session, the counsellor updates status to `COMPLETED` and optionally attaches a guidance message visible on the student's appointment history.

---

## Implementation Details

**Custom User Model:** The default Django `AbstractUser` is extended with `role` (student, counsellor, peer, admin), `anonymous_id` (UUID), `is_approved` (boolean), and `login_count` fields. Role-based access is enforced at every view with direct `user.role` checks following the `@login_required` decorator.

**OTP Verification:** On registration, the user account is created with `is_active=False`. A six-digit OTP is generated, stored in the `EmailOTP` model with a ten-minute expiry, and dispatched via SMTP. The account is activated only upon successful OTP entry.

**Mood Tracking:** The `DailyMood` model stores per-entry mood scores (20/50/80/100) and labels. The student dashboard queries the seven most recent entries, serialises them to JSON, and passes them to Chart.js for a wavy line graph rendering. Login count is tracked session-scoped to prevent duplicate increments within a browser session.

**Anonymous Assessments:** Users without accounts can complete assessments. The system generates a fresh UUID for each submission. If the user subsequently registers, their anonymous assessments can be linked to their account via the anonymous_id field on the User model.

**PDF Generation:** Both the student assessment report and the admin analytics PDF are generated using ReportLab's `canvas` and `SimpleDocTemplate` APIs within dedicated `pdf_utils.py` modules per application.

**Deployment Configuration:** The `Dockerfile` builds a `python:3.11-slim` image, installs dependencies, collects static files at build time via `manage.py collectstatic`, creates a non-root system user, and starts the application with `manage.py migrate`, `createdefaultadmin`, and Gunicorn in a single `CMD` chain. The application binds to port 7860 for Hugging Face Spaces compatibility.

**Notification System:** A `Notification` model stores per-user in-app notification records. A registered Django context processor (`notification_context`) injects unread notification counts into every template context for authenticated users, enabling a persistent notification badge across the entire UI.

---

## Results

The system delivers measurable outcomes across its core functional domains:

| Metric | Value |
|---|---|
| Clinical Assessment Tools Implemented | 2 (PHQ-9, GAD-7) |
| Severity Classification Bands (PHQ-9) | 5 (Minimal, Mild, Moderate, Moderately Severe, Severe) |
| Severity Classification Bands (GAD-7) | 4 (Minimal, Mild, Moderate, Severe) |
| User Roles Supported | 4 (Student, Counsellor, Peer Volunteer, Admin) |
| AI Chatbot Emergency Keywords Monitored | 6 (suicide, kill myself, end my life, hurt myself, self harm, die) |
| Email Notification Parties per Appointment | 3 (Student, Counsellor, All Admins) |
| Resource Categories | 6 (General, Stress, Anxiety, Sleep, Depression, Exam Stress) |
| OTP Validity Window | 10 minutes |
| Deployment Target | Docker / Gunicorn / Hugging Face Spaces |

**Chatbot Contextualisation:** The AI receives up to five recent mood entries, five assessment records, and three recent appointments as structured context per conversation turn, enabling genuinely personalised and clinically-aware responses rather than generic mental health advice.

**Assessment Anonymity:** Both authenticated and unauthenticated users can complete assessments, with UUID-based separation ensuring no personally identifiable information is required for basic mental health screening.

**Administrative Observability:** The admin analytics dashboard aggregates assessment severity distributions and appointment statistics in real time, giving institutional administrators actionable oversight of platform-wide mental health trends.

---

## Installation

### Prerequisites

- Python 3.11 or higher
- pip (Python package manager)
- Git
- A Gmail account or SMTP provider credentials for email functionality (optional for local development)

### Step 1 — Clone the Repository

```bash
git clone https://github.com/harisaigithub/Digital-Mental-Health-and-Psychological-Support-System.git
cd Digital-Mental-Health-and-Psychological-Support-System
```

### Step 2 — Create and Activate a Virtual Environment

```bash
python -m venv venv

# On Linux / macOS
source venv/bin/activate

# On Windows
venv\Scripts\activate
```

### Step 3 — Install Dependencies

```bash
pip install --upgrade pip
pip install -r requirements.txt
```

### Step 4 — Configure Environment Variables

Copy the provided example file and populate it with your values:

```bash
cp .env.example .env
```

Edit `.env` with your configuration:

```env
SECRET_KEY=your-django-secret-key
DEBUG=True

EMAIL_HOST=smtp.gmail.com
EMAIL_PORT=587
EMAIL_USE_TLS=True
EMAIL_HOST_USER=your-email@gmail.com
EMAIL_HOST_PASSWORD=your-app-password
DEFAULT_FROM_EMAIL=your-email@gmail.com
```

To generate a secure `SECRET_KEY`:

```bash
python -c "from django.core.management.utils import get_random_secret_key; print(get_random_secret_key())"
```

### Step 5 — Apply Database Migrations

```bash
python manage.py migrate
```

### Step 6 — Create the Default Admin Account

```bash
python manage.py createdefaultadmin
```

### Step 7 — Collect Static Files (Production Only)

```bash
python manage.py collectstatic --noinput
```

### Step 8 — Run the Development Server

```bash
python manage.py runserver
```

Access the application at `http://127.0.0.1:8000`.

### Docker Deployment (Alternative)

```bash
docker build -t dmh-system .
docker run -p 7860:7860 dmh-system
```

The container automatically runs migrations, creates the default admin account, and starts Gunicorn on port 7860.

---

## Usage

### Registering an Account

Navigate to `/auth/signup/`. Select your role (Student / Counsellor / Peer Volunteer). Enter your credentials and submit. A six-digit OTP will be sent to your registered email address. Enter the OTP on the verification page to activate your account. Counsellor and Peer Volunteer accounts require administrator approval before login.

### Student Workflow

1. Log in and access the **Student Dashboard** to view your mood trend chart.
2. Use the emoji check-in to log your current mood.
3. Navigate to **Assessments** to complete a PHQ-9 or GAD-7 questionnaire.
4. Review your **Assessment History** or download a PDF report.
5. Open the **Chatbot** to converse with the AI wellness assistant.
6. Go to **Appointments** to browse counsellors and book a confidential session.
7. Explore **Resources** for curated mental health guides and content.

### Counsellor Workflow

1. Log in to access the **Counsellor Dashboard**.
2. Review **Student Appointments** and update session statuses.
3. Add post-session **Guidance Messages** for students.
4. Manage the **Resource Library** by adding or updating content.

### Administrator Workflow

1. Log in as the default admin (created via `createdefaultadmin`).
2. Access the **Admin Dashboard** for platform-wide statistics.
3. Navigate to **User Management** to approve pending counsellor/peer registrations, create new users, or modify existing accounts.
4. Review **Assessment Analytics** for system-wide mental health data.
5. Download the **Analytics PDF Report** as needed.

---

## Future Enhancements

- **PostgreSQL Integration** — Migrate from SQLite to PostgreSQL for production scalability, concurrent user support, and advanced query performance.
- **Real-Time Chat** — Implement WebSocket-based real-time messaging between students and counsellors using Django Channels, leveraging the already-configured ASGI infrastructure.
- **Multi-Language Support** — Integrate Django's i18n framework to support regional Indian languages, significantly extending accessibility for rural and non-English-speaking student populations.
- **Mobile Application** — Develop a companion mobile application using React Native or Flutter consuming the existing Django REST Framework endpoints.
- **Appointment Calendar View** — Integrate a calendar-based appointment visualisation (FullCalendar.js) for more intuitive scheduling by both students and counsellors.
- **Advanced AI Personalisation** — Extend the chatbot to incorporate Retrieval-Augmented Generation (RAG) using a vector store of verified mental health resources to ground responses in curated clinical content.
- **Push Notifications** — Add browser push notifications using the Web Push API to supplement the current in-app notification system.
- **Automated Assessment Reminders** — Schedule periodic email or push reminders prompting students to complete regular check-ins using Django-Q or Celery with Redis.
- **Counsellor Video Sessions** — Integrate a WebRTC-based video conferencing module for fully remote, in-platform counselling sessions.
- **Analytics Dashboard Enhancements** — Introduce time-series charts, geographic heatmaps (leveraging existing django-leaflet integration), and cohort-based analysis for institutional mental health research.

---

## Contributing

Contributions are welcome. Please follow the standard Git workflow:

1. Fork the repository.
2. Create a feature branch:
   ```bash
   git checkout -b feature/your-feature-name
   ```
3. Commit your changes with a descriptive message:
   ```bash
   git commit -m "Add: brief description of the change"
   ```
4. Push the branch to your fork:
   ```bash
   git push origin feature/your-feature-name
   ```
5. Open a Pull Request against the `main` branch of the original repository.

Please ensure that your code adheres to PEP 8 style guidelines, that all existing migrations remain unmodified, and that new features include appropriate view-level access control.

---

## License

This project is licensed under the **MIT License**.

```
MIT License

Copyright (c) 2026

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

&nbsp;

## ***To acquire this project, please contact***

**GitHub:** https://github.com/harisaigithub  
**Email:** [harisaiparasa@gmail.com](mailto:harisaiparasa@gmail.com)  

For collaborations, customisations, or deployment support, feel free to reach out.