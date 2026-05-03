# Industry Training and AI Mock Interview Platform

### *A Role-Based Django Web Application Integrating Google Gemini 2.5 Flash for Conversational AI Mock Interviews, Behavioural Authenticity Detection, and End-to-End Industry Training Placement Management*

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

The transition from academic learning to industry placement demands more than technical knowledge — it requires interview readiness, professional communication, and the ability to perform under real-world evaluation conditions. Traditional campus placement systems lack dynamic interview practice tools, offer no objective candidate scoring, and provide recruiters with no validated pre-screening mechanism.

The **Industry Training and AI Mock Interview Platform** is a full-stack Django web application that bridges this gap. It provides students with a fully automated, conversational AI mock interview system powered by Google Gemini 2.5 Flash that adapts questions to the candidate's declared role and skills, progressing through Introductory, Technical, and Behavioural stages. At the conclusion of each session, the AI generates an objective performance report with a numerical score and a three-line evaluation summary.

Recruiters post industry training and internship opportunities, review ranked applicant lists ordered by AI scores, manage candidate pipeline stages from application through final selection, and access detailed interview transcripts. Administrators oversee the entire platform through a system-wide dashboard with live metrics, user management controls, placement statistics, and a tamper-evident audit log of all administrative actions.

---

## Objectives

- Develop a conversational AI mock interview engine using Google Gemini 2.5 Flash that conducts adaptive, role-specific, multi-stage interviews and generates structured performance reports with numerical scores.
- Implement a real-time behavioural authenticity detection mechanism that flags interviews where multiple faces are detected or gaze stability is compromised during the session.
- Build a multi-role platform (Student, Recruiter, Administrator) with role-specific dashboards, navigation bases, and access controls enforced at every view.
- Provide students with a complete placement journey: profile building, opportunity discovery, one-click application, interview scheduling, AI interview conduction, and result tracking.
- Enable recruiters to post opportunities with eligibility criteria and minimum CGPA filters, manage applicant pipelines through six progressive status stages, and review AI-evaluated candidate reports.
- Implement a real-time in-app notification system with type-categorised alerts (opportunity, status update, interview, selection) and a context-processor-injected unread count badge.
- Maintain a `SystemAuditLog` recording every administrative action with actor identity, action type, target user, IP address, and timestamp for institutional accountability.
- Support profile completion scoring, resume file uploads, QR code generation, and placement statistics tracking for institutional reporting.

---

## Summary

The Industry Training and AI Mock Interview Platform is a Django 5.2.10 application organised into five domain-specific apps — `accounts`, `student`, `recruiter`, `adminpanel`, and `pages` — with a centralised `core` module housing the project configuration and the `InterviewEngine` AI utility class.

The platform's distinguishing feature is its AI interview engine. When a student initiates a mock interview for a specific application, the system instantiates an `InterviewEngine` object initialised with the student's name, target role (from the opportunity title), declared skills, and full session history reconstructed from the Django session store. Each student response is submitted via an asynchronous JSON endpoint, enriched with a behavioural metadata payload (face count, gaze stability), and dispatched to the Gemini 2.5 Flash model which returns the next contextually appropriate question. Upon session completion, the engine calls `generate_final_report()` which instructs Gemini to analyse the full conversation and return a structured JSON object containing a score, a three-line evaluation summary, and an authenticity flag.

The recruiter module provides a complete opportunity management and candidate evaluation workflow. Applicants are ranked by AI score, enabling recruiters to identify top candidates objectively. Status updates trigger automatic notifications via a Django signal connected to the Application model's `post_save` event.

The administrator module provides institution-level oversight through live metrics, user approval and suspension controls, system-wide placement statistics, audit log review, and a broadcast notification system targeting students, recruiters, or all users.

---

## Features

### Before Login (Public Access)

- **Welcome / Landing Page** — Introductory page presenting the platform's purpose, key features for each role, and call-to-action navigation to registration and login.
- **User Registration** — Email and role-based signup (Student, Recruiter, Administrator). The very first registered user is automatically assigned the Administrator role with superuser privileges.
- **OTP Email Verification** — A six-digit OTP is generated and dispatched to the registrant's email via SMTP. The account remains inactive (`is_email_verified=False`) until the OTP is entered correctly on the verification page.
- **Login** — Email and password authentication with role-based post-login redirect via `dashboard_redirect`. Unverified users are intercepted and redirected to the verification page.
- **Password Reset** — Full tokenised password reset flow via email using Django's built-in secure framework with custom templates for request, confirmation, and completion pages.

### After Login — Student Role

**Student Dashboard**
- Summary cards: total applications submitted, active interview rounds, selections achieved, and a dynamically computed profile strength percentage.
- Profile strength is computed by counting filled fields (CGPA, skills, resume, SSC percentage, bio) out of five and expressed as a percentage.
- Recent applications list showing the five most recently submitted applications with current status.

**Profile Management**
- View and edit academic details: CGPA, SSC percentage, Intermediate percentage.
- Update professional information: skills (comma-separated), interested roles, bio.
- Upload or replace resume (PDF, stored under `media/resumes/YYYY/MM/`).
- Profile picture upload and management.

**Opportunity Discovery**
- Browse all active industry training and internship opportunities posted by verified recruiters, ordered by most recent.
- Keyword search filtering opportunities by title.
- Opportunity cards displaying title, company, location, minimum CGPA requirement, and application deadline.
- One-click application with duplicate-submission prevention.

**Application Tracking**
- My Applications view listing all submitted applications with current pipeline stage.
- Visual step indicator showing progression through Applied → Round 1 → Round 2 → Final stages.
- Application detail view showing full opportunity description, eligibility criteria, required skills, current status, and AI interview score (post-interview).

**AI Mock Interview Module**
- Interview selection page listing all eligible applications.
- Session-based, multi-turn conversational interview conducted within the browser.
- Questions progress through three stages: Introductory → Technical → Behavioural, adapting to the student's target role and declared skills.
- Student answers submitted asynchronously via JavaScript fetch API; next question returned in real time from Gemini 2.5 Flash.
- Behavioural analysis integration: the client submits face detection count and gaze stability flag with each answer; the server records `is_authentic=False` if anomalies are detected (multiple faces or unstable gaze).
- Skip mechanism: empty answers are substituted with "I would like to skip this question" to maintain interview flow.
- Full session history persisted in the Django session store under key `chat_{app_id}` and replayed as Gemini chat history on each turn.
- Final report generation: after the last question, `generate_final_report()` sends the complete conversation to Gemini, which returns a JSON object with numeric score, three-line summary, and authenticity assessment.
- Interview report page displaying score, evaluation summary, and authenticity status for self-review.

**Notifications**
- Categorised notification list: New Opportunity, Application Status Update, Interview Scheduled, Selection Result.
- Mark-all-as-read action.
- Real-time unread notification count badge injected into every page via a registered Django context processor.

### After Login — Recruiter Role

**Recruiter Dashboard**
- Metric cards: total opportunity postings, total applicants across all postings, shortlisted candidate count, and AI-verified interview count (applications with a non-null AI score).
- Recent postings list for quick access.

**Company Profile Management**
- Create or edit company profile: company name, industry, location, description, contact email, company logo upload, and website URL.

**Opportunity Management**
- Post new internship or industry training opportunities with title, description, eligibility criteria, required skills, minimum CGPA filter, application deadline, and location.
- View, manage, and delete existing postings.
- Toggle opportunity active/inactive status.

**Applicant Pipeline Management**
- Applicant list for each opportunity, ranked in descending order of AI interview score.
- Applicant detail view showing student profile (name, email, skills, CGPA), resume access link, AI interview score, behavioral notes (from Gemini-generated summary), authenticity status, and shortlist indicator.
- Status update interface: advance candidates through Applied, AI Interview Pending, Round 1, Round 2, Final Interview, Selected, or Rejected stages.
- Status changes to Round 1, Round 2, Final, or Selected automatically set `is_shortlisted=True`; Rejected sets it to `False`.
- Status updates trigger automatic notifications to the student via a Django post_save signal.

### After Login — Administrator Role

**Admin Dashboard**
- System-wide metric cards: total students, total recruiters, total active opportunities, total applications.
- AI performance insights: average AI interview score across all applications.
- Selection statistics: total selected count and overall selection rate percentage.
- Recent user registrations list (five most recent).
- Pending recruiter approval count.

**User Management**
- Master user list filterable by role (All, Students, Recruiters).
- Approve or suspend individual user accounts with one-click action URLs.
- All approval and suspension actions are logged to `SystemAuditLog` with actor identity, description, IP address, and timestamp.

**System Reports**
- Application status breakdown aggregated across all opportunities (Applied, Interview Pending, Round 1–2, Final, Selected, Rejected counts).
- Top five recruiters ranked by opportunity posting count.

**Audit Logs**
- Paginated activity log of all administrative actions: User Status Change, Opportunity Modification, System Configuration Update, Report Generation.
- Each entry records the acting admin, action type, target user (if applicable), description, IP address, and timestamp.

**Placement Statistics**
- Batch-level placement snapshot model: total eligible students, total placed, highest package, average package, and top recruiter name.
- Supports multi-batch tracking for institutional placement reporting.

**Broadcast Notifications**
- Admin can create and broadcast system notifications with title, message, and target role (All Users, Students Only, or Recruiters Only).

---

## Technologies Used

### Programming Languages
- Python 3.x
- JavaScript (Fetch API for async AI interview turn processing)

### Libraries and Frameworks
- Django 5.2.10
- django-crispy-forms 2.5
- crispy-bootstrap5 2025.6
- ReportLab 4.4.9
- Pillow 12.1.0
- qrcode[pillow] 8.0
- python-dotenv 1.2.1

### Artificial Intelligence / AI Integration
- Google Generative AI SDK (`google-generativeai` 0.8.5, `google-genai` 1.61.0)
- Model: **Gemini 2.5 Flash** (`gemini-2.5-flash`)
- `InterviewEngine` class (`core/ai_utils.py`):
  - `get_next_question(user_response, behavioral_data)` — submits student answer with behavioural context and returns the next adaptive interview question
  - `generate_final_report()` — instructs the model to analyse the full conversation and return structured JSON: `{ "score": int, "summary": str, "is_authentic": bool }`
- Multi-turn chat managed via `model.start_chat(history=[...])` with full session replay on each request
- Behavioural authenticity detection: face count and gaze stability flags sent client-side and evaluated server-side

### Backend
- Django MVT architecture with Function-Based Views
- Django ORM with SQLite persistence
- Custom User model extending AbstractUser with `role` field and `is_email_verified` gate
- Django Sessions for AI interview chat history persistence (`chat_{app_id}`)
- Django Signals (`post_save` on Application model) for automatic notification dispatch on status changes
- Custom context processor (`notification_count`) injecting unread notification count into every template
- SMTP email backend (Gmail) for OTP delivery and password reset
- `user_passes_test` decorator for admin-role access control
- Media file serving for resume uploads (`media/resumes/YYYY/MM/`) and profile/company images

### Frontend
- Django Template Language with role-specific base templates: `base_public.html`, `base_student.html`, `base_recruiter.html`, `base_admin.html`
- Bootstrap 5 via crispy-bootstrap5
- JavaScript Fetch API for asynchronous AI interview turn submission
- CSRF token injection for secure AJAX POST requests
- HTML5 / CSS3 with role-specific stylesheets (`public.css`, `student.css`, `recruiter.css`)

### Database
- SQLite (default; PostgreSQL recommended for production per settings comment)

### Deployment and DevOps
- Django development server (`manage.py runserver`)
- Media URL configuration for development (`DEBUG` mode static/media serving)
- GEMINI_API_KEY and SMTP credentials configured directly in `settings.py` (environment variable management via python-dotenv recommended for production)

### Security
- OTP-based email verification gate on all new non-admin accounts
- Role-based access control enforced at every view (`user.role` checks, `@user_passes_test`)
- CSRF protection enabled globally
- `is_email_verified=False` login block with redirect to verification page
- Tamper-evident `SystemAuditLog` recording all administrative actions with IP address
- Behavioural authenticity detection flagging multi-face or gaze-unstable interview sessions

---

## System Architecture

The platform follows a modular Django MVT architecture with five domain apps and a central `core` configuration module.

```
core/                           # Project settings, root URLconf, WSGI/ASGI,
│                               # and InterviewEngine AI utility class
├── settings.py                 # Centralised config (SMTP, Gemini API key, AUTH_USER_MODEL)
├── urls.py                     # Root URL dispatcher
├── ai_utils.py                 # InterviewEngine: Gemini 2.5 Flash wrapper
│                               # with multi-turn chat, question generation, final report

accounts/                       # Custom User model (role, is_email_verified, otp),
│                               # Notification model (4 types), auth views (signup,
│                               # login, OTP verify, password reset, dashboard redirect)

student/                        # StudentProfile (academic + professional data, resume),
│                               # Application (6-stage pipeline + AI interview fields),
│                               # views (dashboard, profile, opportunities, apply,
│                               # ai_mock_interview, process_ai_interview, reports),
│                               # signals (post_save → Notification),
│                               # context_processors (unread notification count)

recruiter/                      # RecruiterProfile (company data),
│                               # Opportunity (title, skills, CGPA filter, deadline),
│                               # views (dashboard, post/manage opportunities,
│                               # applicant list ranked by AI score, detail, status update)

adminpanel/                     # SystemAuditLog, PlacementStatistic, AdminNotification,
│                               # views (dashboard, user management, reports,
│                               # audit logs, broadcast notifications)

pages/                          # Public welcome/landing page
```

**Data Flow — AI Interview Turn:**
1. Student submits answer via JavaScript Fetch POST to `/student/ai-interview/process/`.
2. View deserialises JSON body: `answer`, `application_id`, `behavioral` (faces, gaze_stable), `is_last`.
3. Session history (`chat_{app_id}`) is retrieved and converted to Gemini's `{role, parts}` format.
4. `InterviewEngine` is instantiated with student context and replayed history.
5. Behavioural flags are evaluated: `faces > 1` or `gaze_stable=False` → `application.is_authentic = False`.
6. If `is_last=True`: `generate_final_report()` is called; AI score, notes, and authenticity are persisted to the Application model.
7. If `is_last=False`: `get_next_question()` is called; the response is appended to session history and returned as JSON.

**Notification Flow:**
1. Recruiter updates Application status via `/recruiter/update-status/<app_id>/`.
2. Django `post_save` signal fires on Application save.
3. `trigger_status_notification` creates a `Notification` record for the student.
4. `notification_count` context processor injects the unread count into every subsequent page render.

---

## Model Workflow

### AI Mock Interview Session Lifecycle

**Stage 1 — Session Initiation**
Student selects an application and navigates to `/student/ai-interview/session/<app_id>/`. The browser renders the `ai_mock_interview.html` template, which contains the JavaScript interview client. No session history exists yet.

**Stage 2 — First Question**
The page auto-submits a blank opener via Fetch POST to `/student/ai-interview/process/` with `answer=""`. The `InterviewEngine` is instantiated with empty history and returns an introductory question (e.g., "Tell me about yourself and your interest in [role].").

**Stage 3 — Conversational Turns**
For each subsequent turn:
- Student types and submits an answer.
- Client sends `{ answer, application_id, behavioral: { faces, gaze_stable }, is_last: false }`.
- Server retrieves session history, replays it into Gemini's chat format, calls `get_next_question()`, appends the turn to session history, checks behavioural flags, and returns `{ question, is_finished: false }`.
- Questions progress Intro → Technical → Behavioural based on the model's internal logic.

**Stage 4 — Final Report Generation**
On the last question turn, the client sends `is_last: true`.
- Server calls `generate_final_report()` which prompts Gemini to return ONLY a JSON object: `{ "score": int, "summary": str, "is_authentic": bool }`.
- JSON is parsed; `application.ai_score`, `application.ai_behavioral_notes`, and `application.is_authentic` are updated and saved.
- Session history key `chat_{app_id}` is cleared from the session store.
- Response returns `{ is_finished: true, redirect_url: "/student/ai-interview/report/<app_id>/" }`.

**Stage 5 — Report View**
Student is redirected to `/student/ai-interview/report/<app_id>/` which renders the score, summary, and authenticity status from the saved Application record.

### Application Status Pipeline

```
Applied → AI Interview Pending → Round 1 → Round 2 → Final Interview → Selected / Rejected
```
Each forward transition (Round 1–Final, Selected) sets `is_shortlisted=True`. Rejection sets it to `False`. Every status save triggers the `post_save` signal which creates a status-type `Notification` for the student.

### Account Verification Flow
1. User submits registration form with email, role, and password.
2. If first user: auto-assigned admin role, `is_email_verified=True`, `is_staff=True`, `is_superuser=True` — no OTP required.
3. For all other users: OTP generated, stored on User model, dispatched via SMTP, `is_email_verified=False`, role profile created (`StudentProfile` or `RecruiterProfile`).
4. User enters OTP on verify page; on match, `is_email_verified=True`, OTP field cleared.
5. Login gate checks `is_email_verified` before allowing session creation.

---

## Implementation Details

**InterviewEngine Architecture:** The engine class (`core/ai_utils.py`) wraps `genai.GenerativeModel('gemini-2.5-flash')`. On instantiation, the full session history from the Django session store is converted from the application's `[{user, ai}]` format into Gemini's native `[{role, parts}]` format and passed to `model.start_chat(history=[...])`. This ensures the model has full conversational context on every turn without storing the chat object server-side between requests.

**Session History Management:** Interview turn history is stored per-application in the Django session under the key `chat_{app_id}` as a list of `{ "user": str, "ai": str }` dicts. Each turn appends to this list. On report generation (`is_last=True`), the key is removed from the session to free memory.

**Behavioural Authenticity:** The `ai_mock_interview.html` template submits a `behavioral` object with each answer containing `faces` (integer from client-side face detection) and `gaze_stable` (boolean). If `faces > 1` or `gaze_stable=False`, the view sets `application.is_authentic=False` immediately and persists it, flagging the session for recruiter review.

**Automatic Role Profile Creation:** On user registration, the view calls `RecruiterProfile.objects.get_or_create(user=user)` or `StudentProfile.objects.get_or_create(user=user)` immediately after user creation. This prevents 404 errors in dashboard views that call `get_object_or_404` on the profile.

**Profile Completion Scoring:** Five fields are checked on the StudentProfile (CGPA, skills, resume, SSC percentage, bio). The number of non-empty fields is divided by five and multiplied by 100 to yield a completion percentage rendered as a progress indicator on the student dashboard.

**Post-Save Signal:** The `student/signals.py` module registers a `post_save` receiver on the `Application` model. When `created=False` (i.e., an update, not a new record), a `Notification` object is created for the student with type `'status'` and a message interpolating the opportunity title and new status display name.

**Context Processor:** `student/context_processors.py` defines `notification_count(request)` which queries `Notification.objects.filter(recipient=request.user, is_read=False).count()` and injects `unread_notifications` into every template context. This enables a persistent notification badge across all authenticated pages without per-view query duplication.

**Admin Audit Logging:** Every call to `toggle_user_status` creates a `SystemAuditLog` entry capturing the acting admin FK, `action_type='user_status'`, target user FK, a human-readable description string, and `request.META.get('REMOTE_ADDR')` for IP-level accountability.

**First-User Auto-Admin:** The `signup_view` checks `User.objects.count() == 0` before creating the first user and, if true, assigns `role='admin'`, `is_staff=True`, `is_superuser=True`, and `is_email_verified=True` — bypassing OTP and enabling immediate platform administration.

---

## Results

| Metric | Value |
|---|---|
| User Roles | 3 (Student, Recruiter, Administrator) |
| AI Model | Gemini 2.5 Flash (`gemini-2.5-flash`) |
| Interview Stages | 3 (Introductory, Technical, Behavioural) |
| AI Report Fields | 3 (Score 0–100, 3-line Summary, Authenticity flag) |
| Application Pipeline Stages | 7 (Applied, AI Interview Pending, Round 1, Round 2, Final, Selected, Rejected) |
| Notification Types | 4 (Opportunity, Status Update, Interview, Selection) |
| Audit Log Action Types | 4 (User Status, Opportunity Mod, System Config, Report Gen) |
| Behavioural Checks | 2 (Face count > 1, Gaze instability) |
| Profile Completion Fields | 5 (CGPA, Skills, Resume, SSC%, Bio) |
| Resume Upload Path | `media/resumes/YYYY/MM/` |
| OTP Validity | Single-use, session-scoped |
| Time Zone | Asia/Kolkata |

**AI Score Integration:** The `ai_score` field on the Application model is used by the recruiter's applicant list view to rank candidates in descending order by interview performance, providing an objective, AI-validated candidate ranking without manual screening overhead.

**Authenticity Detection:** The platform records `is_authentic=False` on the Application record when behavioural anomalies are detected, providing recruiters with a visible flag in the applicant detail view to identify potentially compromised interview sessions.

---

## Installation

### Prerequisites

- Python 3.8 or higher
- pip
- Git
- Gmail account (or other SMTP provider) for OTP delivery and password reset
- Google Gemini API key (from Google AI Studio)

### Step 1 — Clone the Repository

```bash
git clone https://github.com/harisaigithub/Industry-training-and-ai-mock-interview.git
cd Industry-training-and-ai-mock-interview
```

### Step 2 — Create and Activate a Virtual Environment

```bash
python -m venv venv

# Linux / macOS
source venv/bin/activate

# Windows
venv\Scripts\activate
```

### Step 3 — Install Dependencies

```bash
pip install --upgrade pip
pip install -r req.txt
```

### Step 4 — Configure Settings

Open `core/settings.py` and update the following values:

```python
# Gemini AI
GEMINI_API_KEY = "your-google-gemini-api-key"

# Email (SMTP)
EMAIL_HOST_USER = 'your-email@gmail.com'
EMAIL_HOST_PASSWORD = 'your-gmail-app-password'
DEFAULT_FROM_EMAIL = f"Internship Portal <your-email@gmail.com>"

# Security (generate a new key for production)
SECRET_KEY = 'your-new-secret-key'
```

To generate a secure Django `SECRET_KEY`:

```bash
python -c "from django.core.management.utils import get_random_secret_key; print(get_random_secret_key())"
```

To obtain a Gmail App Password, enable two-factor authentication on your Google account and generate an app-specific password under Google Account Security settings.

To obtain a Gemini API key, visit [Google AI Studio](https://aistudio.google.com/) and create a new API key.

For production, it is recommended to move all credentials to a `.env` file and load them via `python-dotenv` (already listed in dependencies).

### Step 5 — Apply Database Migrations

```bash
python manage.py migrate
```

### Step 6 — Create Media Directories

```bash
mkdir -p media/resumes media/profiles media/logos
```

### Step 7 — (Optional) Collect Static Files

```bash
python manage.py collectstatic
```

### Step 8 — Start the Development Server

```bash
python manage.py runserver
```

Access the application at `http://127.0.0.1:8000`.

The first account registered automatically receives Administrator privileges with no OTP requirement.

---

## Usage

### Administrator Workflow

1. Register the first account at `/accounts/signup/` — it is automatically granted the Admin role and full superuser access.
2. Log in — redirected to the **Admin Dashboard** at `/adminpanel/dashboard/`.
3. Navigate to **Manage Users** at `/adminpanel/users/` to approve or suspend student and recruiter accounts.
4. Review **System Reports** at `/adminpanel/reports/` for application status breakdowns and top recruiters.
5. Monitor **Activity Logs** at `/adminpanel/activity-logs/` for a tamper-evident record of all admin actions.
6. Post **Broadcast Notifications** targeting all users, students only, or recruiters only.

### Recruiter Workflow

1. Register with the Recruiter role and verify your email via OTP.
2. Log in — redirected to the **Recruiter Dashboard** at `/recruiter/dashboard/`.
3. Edit your **Company Profile** at `/recruiter/profile/edit/`.
4. Post new opportunities at `/recruiter/post-opportunity/` with role title, required skills, minimum CGPA, and application deadline.
5. View applicants for any posting at `/recruiter/applicants/<opp_id>/` — listed in descending AI score order.
6. Review individual **Applicant Detail** pages for AI interview scores, behavioral notes, authenticity flags, and resume links.
7. Advance candidates through pipeline stages at `/recruiter/update-status/<app_id>/`. Status changes trigger automatic student notifications.

### Student Workflow

1. Register with the Student role and verify your email via OTP.
2. Log in — redirected to the **Student Dashboard** at `/student/dashboard/`.
3. Complete your **Profile** at `/student/profile/edit/` with CGPA, skills, interested roles, bio, and resume upload.
4. Browse **Opportunities** at `/student/opportunities/` and apply to suitable internships.
5. Track applications at `/student/my-applications/` with pipeline stage indicators.
6. Start an **AI Mock Interview** at `/student/ai-interview/start/`, select an application, and complete the conversational session.
7. Review your **AI Interview Report** at `/student/ai-interview/report/<app_id>/` showing score, evaluation, and authenticity status.
8. Check **Notifications** at `/student/notifications/` for recruiter status updates and opportunity alerts.

---

## Future Enhancements

- **Video-Based Interview Mode** — Integrate WebRTC for live video capture during AI interviews, enabling server-side facial recognition, emotion analysis, and confidence scoring using computer vision APIs, replacing the current client-side face detection approach.
- **Resume Parsing and Auto-Profile Fill** — Use a PDF parsing library (such as PyMuPDF or pdfplumber) combined with the Gemini API to automatically extract CGPA, skills, and experience from uploaded resumes and pre-populate the student profile form.
- **AI-Powered Resume Scoring** — Extend the AI engine to score uploaded resumes against opportunity requirements and provide actionable improvement suggestions before the student applies.
- **PostgreSQL Migration** — Transition from SQLite to PostgreSQL for production-grade concurrent writes, especially during simultaneous AI interview sessions with session-locked JSON operations.
- **Celery + Redis Background Task Queue** — Offload AI report generation and SMTP notifications to background workers to prevent request-response timeout on slow Gemini API calls.
- **Two-Factor Authentication (2FA)** — Add TOTP-based 2FA for Admin and Recruiter accounts to strengthen account security beyond email OTP.
- **Mobile Application** — Develop a Flutter or React Native companion app consuming the platform's views, enabling students to complete AI interviews on mobile devices with native camera access for better face and gaze detection.
- **Leaderboard and Gamification** — Introduce a student-visible AI score leaderboard (optionally anonymised) and practice-streak badges to incentivise regular mock interview use.
- **Bulk Notification Management** — Allow admins to schedule, preview, and A/B test broadcast notifications with open-rate analytics.
- **Placement Analytics Export** — Generate formatted placement statistics PDFs per batch year (leveraging the existing ReportLab dependency) and QR-code-linked digital offer letters (leveraging the existing qrcode[pillow] dependency).

---

## Contributing

Contributions are welcome. Please follow the standard Git workflow:

1. Fork the repository.
2. Create a feature branch:
   ```bash
   git checkout -b feature/your-feature-name
   ```
3. Commit your changes with a clear, descriptive message:
   ```bash
   git commit -m "Add: brief description of the change"
   ```
4. Push to your fork:
   ```bash
   git push origin feature/your-feature-name
   ```
5. Open a Pull Request against the `main` branch of the original repository.

Ensure all new views include `@login_required` and appropriate role checks. For AI engine changes, include a description of the prompt modifications and their intended behavioural effect in the PR description. Do not commit API keys or SMTP credentials.

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
