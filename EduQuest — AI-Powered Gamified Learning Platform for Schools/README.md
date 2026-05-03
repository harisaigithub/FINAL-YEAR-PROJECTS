# 🎮 EduQuest — AI-Powered Gamified Learning Platform for Schools

> **A full-stack Django web application that transforms K–12 education through game-based learning, XP progression, AI-powered tutoring, and real-time mentorship — designed for students, teachers, guardians, and administrators across multiple institutes.**

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

EduQuest is an AI-augmented gamified education platform built to address a persistent challenge in modern schooling: declining student engagement and motivation in traditional classroom settings. By applying game design mechanics — experience points (XP), levels, badges, daily streaks, and competitive leaderboards — to academic subjects, the platform transforms learning from a passive experience into an active, rewarding journey.

The system supports students from Class 6 through Class 12, with content segmented by grade, stream (MPC/BIPC for Classes 11–12), and institute affiliation. Four distinct user roles — Student, Teacher, Guardian, and Admin — each receive a purpose-built interface and workflow. Students earn XP and badges by playing educational games linked to their curriculum. Teachers monitor cohort health, track engagement metrics, and communicate with students via a built-in messaging system. Guardians receive a dedicated portal to monitor their children's academic progress. Administrators manage the entire ecosystem — institutes, subjects, games, and platform-wide analytics.

A Google Gemini-powered AI chatbot serves as an always-available subject-specific academic tutor. An advanced Scholar Mode allows students to upload their own PDF or text documents and engage in AI-driven Q&A sessions grounded in that content — essentially enabling personalized, document-aware tutoring at scale.

---

## Objectives

- Implement a complete gamification engine with XP progression, configurable levels, badge unlocks, daily streaks, and a global leaderboard.
- Design four educational game types: Animated Quiz, Canvas Action Game, Interactive Drag-and-Drop Sorting, and Scientific Simulation — each with configurable difficulty and XP rewards.
- Integrate Google Gemini (`gemini-flash-latest`) as a subject-scoped AI academic tutor with rate-limiting and multi-turn conversation support.
- Build an AI Scholar Mode enabling students to upload PDF or TXT documents and conduct AI-powered Q&A sessions anchored to the document content.
- Implement a four-role access control system (Admin, Teacher, Student, Guardian) with role-based decorators and OTP-verified email registration.
- Provide per-role analytics dashboards: XP trends, engagement time, subject mastery, cohort health, and institute-wide statistics.
- Enable real-time mentor-student matching through a formal request/approval workflow and an in-platform direct messaging system.
- Support a multi-institute architecture where subjects, games, and users are partitioned by institute affiliation.
- Deliver a signal-driven notification system that automatically alerts users on XP earnings, badge unlocks, level-ups, and mentor messages.
- Generate downloadable intelligence reports in CSV format for teachers and provide per-student audit profiles.

---

## Summary

EduQuest is a Django 5 multi-app web platform serving as a comprehensive academic engagement ecosystem for Indian school education. Students log in to access curriculum-aligned educational games, earn XP per correct answer or game completion, level up through a configurable tier system, and collect achievement badges — all automatically tracked and persisted in a relational database. Their progress is visualized on personal dashboards and subject-level detail pages.

The platform is modularized into nine Django applications — `accounts`, `academics`, `institutes`, `games`, `gamification`, `chatbot`, `analytics`, `mentorship`, and `notifications` — each owning a discrete educational domain. Gemini powers both the subject chatbot and the Scholar document-reading mode, turning uploaded materials into interactive learning sessions. The teacher dashboard exposes XP health indicators, flags lagging students (below 300 XP), highlights top performers, and renders Chart.js visualizations of cohort XP distribution. Guardians monitor their children's total XP, games played, and engagement time in real time.

---

## Features

### Before Login (Public Access)

- **Landing Page** — A public-facing marketing page introducing the platform's capabilities with a call-to-action for registration or login.
- **Role-Based Registration** — Email-based signup with role selection (Student, Teacher, Guardian, or Admin). Students additionally select their institute, academic class, stream (for Classes 11–12), and optionally link a guardian and mentor at registration.
- **Email OTP Verification** — A six-digit, time-bound (10-minute) OTP is dispatched via Gmail SMTP on signup. The account is activated only upon successful OTP entry.
- **Login with Remember Me** — Credential authentication with a "Remember Me" option that controls Django session expiry.
- **Password Reset** — Full Django password reset flow via SMTP email.

---

### After Login — Student Portal

- **Student Dashboard** — Personalized overview displaying total XP, current level with progress bar (percentage toward next level), daily streak count, global rank, subject-wise XP breakdown, earned badges, recent XP log entries, and total engagement time in minutes.
- **Subject Explorer** — A filtered subject list scoped to the student's academic class, stream, and institute. Displays per-subject XP and a mastery percentage bar (calculated against a 500 XP cap per subject).
- **Subject Detail Page** — Deep-dive view for a single subject showing linked active games, subject-level XP, current subject level, next level threshold, and a progress percentage bar.
- **Game Library** — Filterable catalog of all active games available for the student's grade. For Classes 11–12, games are additionally filtered by stream.
- **Game Detail Page** — Displays game description, difficulty, XP reward, game type, and the student's most recent attempt score and XP earned.
- **Play Game Engine** — Four distinct game rendering modes:
  - **Animated Quiz** — Up to 15 multiple-choice questions per session rendered dynamically. Answers are submitted as a form batch; scoring is computed server-side.
  - **Canvas Action Game** — `play_canvas.html` renders a JavaScript canvas-based action game (e.g., "Equation Archer") driven by the game's JSON config field.
  - **Interactive Drag-and-Drop** — `play_interactive.html` renders sorting/categorization games with client-side interactivity.
  - **Scientific Simulation** — `play_simulation.html` renders simulation-based exercises.
- **Game Result Page** — Displays final score, question count, XP earned, and level-up notification if applicable. Supports AJAX response for real-time XP award without full page reload.
- **AI Subject Chatbot** — Subject-scoped conversational AI powered by Google Gemini. Maintains multi-turn chat sessions per subject. Includes a rate limit of 15 messages per minute per user. Chat history is stored per session and displayed chronologically.
- **Scholar Mode (AI Document Tutor)** — Allows students to upload a PDF (up to 20 pages) or TXT file. The extracted text is stored in the Django session. Students then ask questions answered by Gemini using the document as the primary context — with clear fallback to general knowledge when the answer is not in the document.
- **Gamification Dashboard** — Dedicated page showing global XP profile, per-subject XP records, and all earned badges with unlock timestamps.
- **Badge Gallery** — Full catalog of all platform badges with XP thresholds; earned badges are visually highlighted.
- **Mentorship Search** — Browse all registered teachers and submit a formal mentor request. Request status (pending/approved/rejected) is tracked and notified.
- **Mentor Messaging** — Direct one-on-one message thread between student and assigned mentor, with read/unread status tracking.
- **Add Guardian** — Students can link an existing guardian account by email after registration.
- **Analytics Page** — Dedicated view showing total XP, engagement time, games played, and subject-wise progress bars (normalized as a percentage of total XP across all subjects).
- **Notifications Feed** — In-app notification list covering XP earned, badge unlocks, level-ups, mentor request status changes, and incoming mentor messages. Supports archiving.

---

### After Login — Teacher Portal

- **Teacher Dashboard** — Comprehensive cohort health hub showing total assigned students, average XP, lagging student count (XP < 300), top 5 performers, total engagement time, recent XP log feed, a sorted student roster table (with per-student XP, games played, engagement time, and health label: excellent/good/lagging), and a Chart.js bar chart of per-student XP distribution.
- **Intelligence Alerts Panel** — Auto-generated insight cards flagging lagging students with warnings and top performers with success indicators.
- **Teacher Analytics** — Institute-filtered view showing top 5 students by XP, lagging students below 200 XP threshold, total student count, and average XP.
- **Student Audit Profile** — Individual student detail view showing XP profile and engagement data — accessible from the teacher's analytics panel.
- **Mentor Request Management** — A queue of pending mentorship requests from students with approve/reject actions. Approval triggers a `StudentMentor` record creation and a notification to the student.
- **Mentor Messaging** — Same direct messaging interface shared with students, enabling bi-directional communication.
- **Subject Management** — Institute-filtered view of all active subjects assigned to the teacher's institute, ordered by grade.
- **Mission Control** — Per-subject view listing all linked games for quick access and classroom planning.
- **Download Intelligence Report** — One-click CSV download containing student email, class, and total XP for all students in the teacher's institute.

---

### After Login — Guardian Portal

- **Guardian Dashboard** — Overview of all linked children's academic progress. Per-child cards show total XP, total games played, and total engagement time in minutes.

---

### After Login — Admin Panel

- **Admin Dashboard** — Platform-wide statistics: total users, students, teachers, guardians, institutes, games, subjects, and game attempts. Includes a list of the five most recently created institutes.
- **Institute Management** — Full CRUD: create institutes with name, code, city, state, and address; toggle active/inactive status; view per-institute student and teacher rosters.
- **Subject Registry** — Create and list all subjects with class, stream, and institute mapping. Enforces stream assignment for Classes 11–12 and prevents null-institute records.
- **Game Creation** — Admin form to deploy new games linked to subjects, with game type, grade range, and XP reward configuration.
- **Admin Analytics** — Platform-wide view including total user counts by role, institute count, global top 10 students by XP, and per-institute student/teacher counts.
- **Django Admin Panel** — Full model-level data management via the built-in Django admin at `/admin/`.

---

## Technologies Used

### Programming Languages
- Python 3.10+

### Libraries & Frameworks
- Django 5.x — Core web framework, ORM, session management, Django signals, URL routing
- django-extensions — Enhanced management commands and development shell utilities
- PyPDF2 — PDF text extraction for Scholar Mode document upload and processing
- python-dotenv — Environment variable management from `.env`
- Pillow — Profile picture and game thumbnail image handling

### Machine Learning / AI
- Google Generative AI SDK (`google-generativeai`) — Gemini `gemini-flash-latest` model powering the subject chatbot and Scholar document Q&A
- Prompt engineering — System-level subject-scoping instructions and document-context injection for grounded, structured AI responses

### Backend / Frontend
- Django template engine — HTML rendering with Jinja-style inheritance (`base_auth.html`, `base_public.html`)
- Bootstrap 5 — Responsive UI framework across all role-specific portals and dashboards
- Chart.js — Client-side XP bar charts and subject distribution visualizations on teacher and analytics dashboards
- JavaScript Canvas API — Canvas-based action game rendering (Equation Archer and similar canvas-type games)
- AJAX/JSON (`XMLHttpRequest`) — Asynchronous game score submission and real-time XP award without full page reload

### Database
- SQLite3 — Default development database (production-ready migration path to PostgreSQL)
- Django ORM — All data access, aggregation (`Count`, `Avg`, `Sum`), annotation, and atomic transaction management

### Deployment & DevOps
- Django WSGI (`config/wsgi.py`) and ASGI (`config/asgi.py`) — Production server interface support
- `manage.py` — Django management command entry point
- Seed scripts: `seed_master_games.py`, `seed_questions.py`, `seed_test_data.py`, `seed_test_users.py` — Automated data population for institutes, subjects, games, questions, and test users

### Security
- Custom `AbstractUser` replacing username with email as the primary identifier
- Email OTP verification with 10-minute expiry before account activation
- Role-based access decorators: `@role_required(role)` wrapping `@login_required` for fine-grained view-level protection
- Django CSRF middleware on all POST endpoints
- SMTP-based transactional email via Gmail with App Password (TLS, port 587)
- Rate limiting on Gemini chatbot: maximum 15 user messages per minute per student
- Session-based OTP state and Scholar document context storage

---

## Dataset

### Academic Question Bank
The platform ships with a comprehensive seeded question bank (`seed_questions.py`) covering over 120 curriculum-aligned multiple-choice questions across Mathematics, Science, Biology, Physics, and Chemistry for Classes 6 through 12. Questions are structured with four options and a single correct answer key, mapped to specific game titles and grade levels.

### Game Catalog
The master game seed (`seed_master_games.py`) defines 12 game archetypes spanning grades 6–12: Equation Archer, Fraction Pizza, Circuit Connector, State Shifter, Organelle Sort, and Pattern Path (Grades 6–10), and Projectile Pilot, Ray Tracer, Bond Builder, Titration Pro, DNA Splicer, and Heart Pump (Grades 11–12). XP rewards range from 50 to 150 per game depending on complexity and grade level.

### Gamification Configuration
Level thresholds, badge XP requirements, and subject mastery caps are configurable through the Django admin panel and seed scripts. `seed_test_data.py` populates sample XP profiles, badges, levels, and engagement records for testing and demonstration purposes. `seed_test_users.py` creates sample student and teacher accounts for rapid onboarding during development.

### Scholar Mode Input
Student-uploaded PDFs (up to 20 pages) and plain TXT files are processed at runtime using PyPDF2. Extracted text (capped at 10,000 characters) is stored in the Django session and injected into Gemini prompts as document context — not persisted to the database, ensuring student privacy and avoiding storage growth from uploaded materials.

---

## System Architecture

The application follows a modular monolithic architecture. Each Django application owns a distinct educational domain, communicating through shared model imports and a centralized signal dispatcher.

```
EduQuest
│
├── config/               → Project settings, root URL configuration, WSGI/ASGI entry points
├── accounts/             → Custom User (email-based), OTP registration, four role dashboards, login/logout
├── institutes/           → Institute model, CRUD views, status toggling, roster views
├── academics/            → AcademicClass (6–12), Stream (MPC/BIPC), Subject models and views
├── games/                → Game model (quiz/canvas/drag_drop/simulation), Questions, StudentAttempts, play engine
├── gamification/         → StudentXP, SubjectXP, Level, Badge, StudentBadge, XPLog; XP/badge award services
├── chatbot/              → ChatSession, ChatMessage, Gemini integration, Scholar Mode PDF/TXT processing
├── analytics/            → StudentEngagement, PerformanceSnapshot, analytics views, CSV intelligence report
├── mentorship/           → MentorRequest, StudentMentor, MentorMessage; request approval workflow and chat
├── notifications/        → Notification model, Django signals (XPLog → notify, Badge → notify), context processor
│
└── templates/            → Per-app HTML templates; base_auth.html and base_public.html as shared layout bases
```

**Data Flow (Game Session):** Student selects a game → `game_detail` view shows metadata and prior attempt → Student clicks Play → `play_game` GET renders the game template with questions and JSON config → Student submits answers/score → `play_game` POST computes XP, creates `StudentAttempt`, calls `award_xp()` service → `award_xp` atomically updates `StudentXP`, `SubjectXP`, creates `XPLog` → Django signal fires on `XPLog` creation → `Notification` created → `StudentEngagement` record written → Result rendered (or AJAX JSON response returned).

---

## Model Workflow

### Step 1 — User Onboarding
The registration form collects email, password, role, and optional institute/class/stream/guardian/mentor references. The user is created with `is_active=False`. A six-digit OTP is generated and sent via SMTP. Upon OTP verification, `is_active` and `is_verified` are set to `True`. On first login, students receive a streak initialization through `StudentXP.update_streak()`.

### Step 2 — Game Execution
The `play_game` view supports two execution paths. For quiz-type games, a standard form POST triggers server-side correct-answer comparison across up to 15 questions. For canvas and drag-drop games, the client-side score is passed directly as form data. XP earned is calculated as `(score / total_questions) × game.xp_reward`, clamped to the range `[0, xp_reward]`.

### Step 3 — XP & Level Progression
The `award_xp()` service runs inside `transaction.atomic()`. It increments `StudentXP.total_xp`, recomputes the active `Level` by querying the highest level whose `xp_required ≤ total_xp`, updates `SubjectXP` for the relevant subject, writes an `XPLog` entry, and calls `check_badges()` to unlock newly eligible badges. The `update_streak()` method increments the streak if the last activity was exactly one day ago, resets to 1 if more than one day has elapsed, and initializes on first activity.

### Step 4 — AI Chatbot Interaction
`chatbot/services.py` invokes `genai.GenerativeModel("gemini-flash-latest").generate_content()` with a composed prompt consisting of a system instruction (subject-scoping rules and formatting guidelines) followed by the student's question. The response is stored as a `ChatMessage` with `role="bot"`. Rate limiting checks user-role message counts in the past 60 seconds against a threshold of 15.

### Step 5 — Scholar Mode Pipeline
The student uploads a PDF or TXT document. PyPDF2 extracts text from up to 20 pages. Up to 10,000 characters are stored in `request.session['scholar_context']`. Subsequent questions invoke `generate_subject_response()` with the document injected as context, instructing Gemini to use it as the primary reference and to explicitly acknowledge when information is sourced from general knowledge instead.

### Step 6 — Analytics & Reporting
`StudentEngagement` records are created after every game session and every chatbot message. Teacher analytics aggregate XP and engagement data via Django ORM `Sum`/`Avg`/`Count` queries filtered by institute. CSV intelligence reports are generated with Python's `csv.writer` and streamed as `HttpResponse` attachments without writing to disk.

---

## Implementation Details

**Multi-Role Custom User Model:** The default `AbstractUser` was replaced with a custom `User` model using email as the primary identifier. A `role` field (admin/teacher/student/guardian) drives routing and view-level access control. Students carry foreign keys to `Institute`, `AcademicClass`, `Stream`, `mentor` (self-referential FK to a teacher), and `guardian` (self-referential FK to a guardian). This single-model design avoids separate profile tables while maintaining full relational integrity.

**Role-Based Access Decorator:** `accounts/decorators.py` implements `@role_required(role)` which checks `request.user.role` after `@login_required` and returns an HTTP 403 or redirects unauthorized access attempts, ensuring zero cross-role view contamination.

**Atomic XP Service:** All XP mutations are wrapped in `django.db.transaction.atomic()` to prevent partial updates from concurrent requests. The `award_xp` function in `games/services.py` is the single point of XP mutation, ensuring consistency between `StudentXP`, `SubjectXP`, `XPLog`, and badge award records.

**Django Signals for Notifications:** `notifications/signals.py` registers `post_save` receivers on `XPLog` and `StudentBadge`. This decouples the gamification engine from the notification system entirely — XP earning and badge unlocking automatically trigger notifications without any direct coupling in game views or service functions.

**Notification Context Processor:** `notifications/context_processors.py` is registered globally in settings and injects the unread notification count into every request context, enabling the navigation bar badge counter to update on every page load without explicit per-view logic.

**Game Config JSON Field:** Each `Game` record carries a `config = JSONField(default=dict)` that passes arbitrary game-specific parameters (gravity, speed, time limits, target counts) to frontend JavaScript as a serialized object. This allows admins to tune canvas games and simulations without code changes, supporting a no-code configuration model for game designers.

**Scholar Mode Session Architecture:** Document text is stored in `request.session['scholar_context']` (capped at 10,000 characters) rather than the database, preserving student privacy and avoiding storage growth from uploaded materials. The session context is replaced on each new document upload, ensuring query freshness and preventing cross-document context bleed.

**Stream-Gated Content:** All subject and game queries for Classes 11–12 apply an additional `stream` filter. Students without a stream assignment for these grades receive empty querysets, preventing cross-stream content exposure. This logic is consistently enforced across `academics/views.py`, `games/views.py`, and `accounts/views.py`.

---

## Results

| Metric | Value |
|---|---|
| Supported Grade Range | Class 6 – Class 12 |
| Supported Streams | MPC, BIPC (Classes 11–12) |
| User Roles | Admin, Teacher, Student, Guardian |
| Game Types | Animated Quiz, Canvas Action, Drag-and-Drop, Simulation |
| Seeded Game Catalog | 12 game archetypes across Grades 6–12 |
| Seeded Question Bank | 120+ curriculum-aligned MCQs |
| AI Engine | Google Gemini (`gemini-flash-latest`) |
| Scholar Mode Input | PDF (up to 20 pages) + TXT |
| Scholar Context Window | Up to 10,000 characters per session |
| Chatbot Rate Limit | 15 user messages per minute |
| XP Award Mechanism | Atomic transactional service with streak update |
| Badge System | XP-threshold-based auto-unlock with signal-driven notification |
| Analytics Export | CSV report (student email, class, total XP) |
| Notification Triggers | XP earned, badge unlocked, level-up, mentor messages, mentor request updates |
| Engagement Tracking | Per-session minutes logged in `StudentEngagement` |

The platform demonstrates that gamification mechanics — streaks, levels, leaderboards, and badges — combined with AI-assisted subject tutoring and curriculum-aligned game content, create a self-reinforcing motivation loop. Teachers receive actionable cohort intelligence (lagging vs. excellent health labels) without requiring manual data aggregation. Guardians gain full transparency into academic engagement through a dedicated portal, without accessing internal teacher or student tooling.

---

## Installation

### Prerequisites
- Python 3.10 or later
- pip
- Git
- A valid Google Gemini API key (obtainable from [Google AI Studio](https://aistudio.google.com/))
- Gmail account with App Password enabled for SMTP delivery

### Step 1 — Clone the Repository

```bash
git clone https://github.com/harisaigithub/Gamified-Project-.git
cd Gamified-Project-
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
pip install django python-dotenv google-generativeai PyPDF2 django-extensions Pillow
```

### Step 4 — Configure Environment Variables

Create a `.env` file in the project root:

```env
GEMINI_API_KEY=your_google_gemini_api_key_here
EMAIL_HOST_USER=your_email@gmail.com
EMAIL_HOST_PASSWORD=your_gmail_app_password_here
```

> **Note:** Generate a Gmail App Password by enabling 2-Factor Authentication on your Google account and creating an App Password under Account Settings → Security → App Passwords.

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

### Step 7 — Seed Initial Data

Run the seed scripts in the following order to populate institutes, academic classes, subjects, games, questions, and test users:

```bash
python seed_master_games.py
python seed_questions.py
python seed_test_users.py
python seed_test_data.py
```

### Step 8 — Configure Gamification Levels and Badges (via Django Admin)

1. Navigate to `/admin/` and log in with superuser credentials.
2. Under **Gamification**, create `Level` entries (e.g., Level 1 at 0 XP, Level 2 at 100 XP, Level 3 at 300 XP, etc.).
3. Create `Badge` entries with names, descriptions, icons, and XP thresholds (e.g., "First Steps" at 50 XP, "Scholar" at 500 XP).

### Step 9 — Collect Static Files (for Production)

```bash
python manage.py collectstatic
```

### Step 10 — Run the Development Server

```bash
python manage.py runserver
```

Access the application at `http://127.0.0.1:8000/`.

---

## Usage

### As a Student

1. Navigate to `http://127.0.0.1:8000/` and click **Sign Up**.
2. Select the **Student** role, enter your email and password, choose your institute, academic class, and stream (if Classes 11–12), and optionally link a guardian and mentor.
3. Verify your account using the OTP sent to your email inbox.
4. Log in — the system redirects you to your **Student Dashboard**.
5. Review your XP total, current level and progress bar, streak count, global rank, and subject XP breakdown.
6. Click **Subjects** to browse your curriculum subjects and view per-subject mastery progress.
7. Click **Games** to access your grade and stream-filtered game catalog. Select a game and click **Play**.
8. Complete the game and submit your answers. View your score, XP earned, and any level-up or badge notification on the result page.
9. Navigate to **AI Tutor** to open a subject-scoped chat session with Gemini.
10. Switch to **Scholar Mode** to upload a PDF or TXT file and ask document-specific questions.
11. Visit **Gamification** to review your global XP profile and earned badges.
12. Navigate to **Mentorship** to search for and send a request to a teacher mentor, then message them once approved.
13. Check **Notifications** for a consolidated feed of XP alerts, badge unlocks, and mentor updates.

### As a Teacher

1. Log in with teacher credentials. The system redirects to the **Teacher Dashboard**.
2. Review the cohort XP bar chart, intelligence alerts (lagging students and top performers), and the full student roster table.
3. Navigate to **Analytics** for XP averages, top/lagging student lists, and to download the CSV intelligence report.
4. Click a student in the analytics view to access their individual audit profile.
5. Navigate to **Mentorship → Requests** to review and approve or reject pending student mentor requests.
6. Use the **Message** feature in the student list to send direct messages to students.
7. Navigate to **Subjects → Mission Control** to review the games linked to each subject.

### As an Admin

1. Log in with superuser credentials. The system redirects to the **Admin Dashboard**.
2. Navigate to **Institutes** (`/institutes/`) to create and manage school registrations.
3. Navigate to **Academics** (`/academics/`) to create subjects mapped to specific classes, streams, and institutes.
4. Navigate to **Analytics** (`/analytics/admin/`) for global platform statistics.
5. Access the **Django Admin** panel at `/admin/` for Level and Badge configuration, game creation, and direct model management.

---

## Future Enhancements

- **Real-Time Multiplayer Games** — Introduce WebSocket-based (Django Channels) head-to-head quiz competitions between students of the same class to drive competitive social engagement.
- **Adaptive Difficulty Engine** — Analyze per-student answer history to dynamically adjust question difficulty within a game session, personalizing the challenge level to each student's proficiency.
- **Automated Guardian Progress Reports** — Schedule weekly email digests to guardians summarizing their child's XP earned, games played, streak status, and newly unlocked badges.
- **Mobile Application** — Develop a cross-platform React Native or Flutter mobile app enabling offline game access and push notifications for streak reminders and badge unlocks.
- **Teacher-Authored Question Builder** — Provide teachers with a UI to create and submit custom quiz questions for their subjects, enabling institute-specific content beyond the seeded question bank.
- **PostgreSQL Migration** — Transition from SQLite to PostgreSQL for production-grade concurrency, full-text search, and horizontal scalability under load.
- **Containerized Deployment** — Package the application with Docker and provide a `docker-compose.yml` with Nginx + Gunicorn + PostgreSQL for one-command production deployment.
- **Advanced Leaderboard Segmentation** — Add class-level, stream-level, and subject-specific leaderboards to complement the global ranking, making competition contextually relevant.
- **AI-Generated Question Bank Expansion** — Use Gemini to auto-generate curriculum-aligned MCQs for new subjects or grade ranges, reducing the manual effort of question seeding.
- **Attendance Integration** — Track physical classroom attendance alongside digital engagement to provide teachers a unified view of student participation across both domains.

---

## Contributing

Contributions to EduQuest are welcome. Please adhere to the following Git workflow:

1. **Fork** the repository on GitHub.
2. **Clone** your fork locally:
   ```bash
   git clone https://github.com/your-username/Gamified-Project-.git
   cd Gamified-Project-
   ```
3. **Create a feature branch** from `main`:
   ```bash
   git checkout -b feature/your-feature-name
   ```
4. **Implement your changes**, ensuring adherence to the existing code style and the Django app modularization pattern.
5. **Commit** with a descriptive message:
   ```bash
   git add .
   git commit -m "feat: describe your feature concisely"
   ```
6. **Push** your branch to your fork:
   ```bash
   git push origin feature/your-feature-name
   ```
7. **Open a Pull Request** against the `main` branch of the original repository with a clear description of the problem solved, approach taken, and any relevant test cases.

Please ensure all existing migrations remain intact and that seed scripts remain functional before submitting a pull request.

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