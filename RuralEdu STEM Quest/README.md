# RuralEdu STEM Quest

### *Bridging the Educational Divide Through Gamified Learning*

> A full-stack, gamified learning management system engineered to transform STEM education for rural students through interactive game-based learning, intelligent XP progression, real-time performance analytics, and AI-powered mentorship.

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

**RuralEdu STEM Quest** addresses a critical and persistent challenge in the Indian education system: the widening engagement and performance gap between rural and urban students in STEM disciplines. Students in rural institutes, particularly those preparing for science streams (MPC and BIPC) from Class 6 through Class 12, lack access to the interactive, technology-driven learning experiences that their urban counterparts take for granted.

This platform reimagines the standard Learning Management System (LMS) by embedding game design principles — XP systems, leveling mechanics, badges, daily streaks, and leaderboards — directly into the academic workflow. Rather than passive content delivery, RuralEdu STEM Quest positions every quiz, simulation, and drag-and-drop activity as a mission that rewards participation, accuracy, and consistency.

The platform is architecturally designed for multi-institute deployment, supporting a four-role hierarchy (Admin, Teacher, Guardian, Student) with purpose-built dashboards for each role. An integrated AI tutor powered by Google's Gemini model provides on-demand academic assistance, making individualized mentorship scalable across institutions with limited teaching staff.

---

## Objectives

- Design and deploy a gamified STEM learning platform specifically serving students from Classes 6 to 12 in rural educational institutes.
- Implement a robust XP and leveling engine that rewards engagement, accuracy, and daily activity consistency through streak mechanics.
- Provide teachers with actionable, real-time analytics — including an intel feed that surfaces at-risk students and top performers automatically.
- Enable guardians to monitor their ward's academic progress, game activity, and time-on-platform through a dedicated read-only dashboard.
- Integrate Google Generative AI (Gemini) as an AI Mentor chatbot scoped to subject-specific queries, enabling personalized academic support at scale.
- Support multi-institute management with institute-level filtering of subjects, academic classes, streams, and games through a centralized admin panel.
- Deliver a responsive, accessible UI using TailwindCSS, ensuring usability on low-bandwidth or mobile devices common in rural contexts.
- Enable flexible deployment on cloud infrastructure (AWS EC2) with a path to production-grade database backends (MySQL).

---

## Summary

RuralEdu STEM Quest is a Django 5.2-based web application comprising ten loosely coupled Django apps — `accounts`, `academics`, `analytics`, `chatbot`, `games`, `gamification`, `institutes`, `mentorship`, `notifications`, and `config`. Each app encapsulates a distinct domain of functionality, communicating through well-defined service layers and Django signals rather than direct cross-app model coupling.

The gamification engine maintains a `StudentXP` profile per student, tracking total XP, current level, streak days, and last activity date. Subject-specific XP is tracked independently via `SubjectXP`, enabling granular mastery analytics per subject. Badges unlock automatically when XP crosses predefined thresholds, and all XP transactions are logged for auditability. The game engine supports four distinct game types — animated quiz, canvas action game, drag-and-drop interactive, and scientific simulation — each with configurable difficulty levels, grade ranges, and JSON-based runtime configuration for parameters such as game speed, time limits, and gravity.

Authentication uses a custom `AbstractUser` model with email as the primary identifier, email OTP verification, and role-based access control enforced via custom decorators. The notification system uses Django signals to deliver real-time in-app notifications for level-ups, badge unlocks, and mentor messages.

---

## Features

### Public / Pre-Login

- Informational landing page introducing the platform's mission and capabilities.
- Role-based login portal with email and password authentication.
- Email OTP verification flow for new account activations.
- Password reset workflow with secure token-based email links.

### Student Dashboard

- **Mission Control:** A central game discovery hub filtered by the student's institute, academic class, and stream (MPC/BIPC).
- **Game Catalogue:** Browse games by subject, difficulty level (Easy / Medium / Hard), and game type. Each game displays XP reward potential before play.
- **Gameplay Modes:** Four interactive formats — animated MCQ quiz, canvas-based action game (e.g., Equation Archer), drag-and-drop sorting/matching, and scientific simulation.
- **XP & Leveling System:** Earn XP on every game completion proportional to score percentage. Levels are calculated automatically against a configurable XP threshold table.
- **Badge Gallery:** View earned and locked badges. Badges unlock when total XP crosses predefined thresholds (e.g., Quick Solver, Science Pro, STEM Champion).
- **Daily Streak Tracker:** Consecutive daily login and game activity streaks are tracked and displayed to reinforce habitual engagement.
- **Subject-Wise Mastery:** Visual breakdown of XP earned per subject, surfacing areas of strength and improvement.
- **Global Leaderboard:** Real-time rank among all students in the platform by total XP.
- **AI Mentor Chatbot:** Subject-scoped conversational tutor powered by Google Gemini. Enforces subject boundary rules to keep interactions academically focused.
- **Reading Mode:** Distraction-free document reading interface for study material within the chatbot module.
- **Mentor Messaging:** Direct in-platform messaging with assigned teacher-mentor.
- **Guardian Linking:** Students can add a guardian user to their profile to enable parental monitoring.
- **Analytics Dashboard:** Personal performance charts including XP progression over time, game attempt history, and engagement time per subject.

### Teacher Dashboard

- **Student Roster:** View all students assigned to the teacher's mentorship.
- **Intel Feed:** Automated alerts identifying students with declining performance, low engagement streaks, or zero recent activity, alongside top performers.
- **XP Bar Charts:** Visual comparison of student XP and game attempts at the class level.
- **Class Health Indicators:** Aggregate metrics reflecting overall class engagement and performance.
- **Mentor Messaging:** Bidirectional messaging system between teacher and assigned students.
- **Analytics View:** Per-student drill-down analytics including subject mastery, engagement time, and attempt history.

### Guardian Dashboard

- **Ward Progress Monitoring:** View linked student's total XP, level, games played, badges earned, and time-on-platform.
- **Notification Feed:** Receive in-app notifications on significant student milestones such as level-ups and badge unlocks.

### Admin Panel

- **Institute Management:** Create and manage institutes; configure which subjects, classes, and streams belong to each institute.
- **Subject & Curriculum Configuration:** Define academic classes (6–12), streams (MPC/BIPC), and subjects with their institute associations.
- **Game Management:** Create games with title, subject, type, difficulty, grade range, XP reward, thumbnail, and JSON config for runtime parameters.
- **Question Bank:** Add MCQ questions with optional image support (for Physics/Biology diagrams) per quiz game.
- **User Management:** Full CRUD over all user accounts across roles.
- **Platform Analytics:** Aggregate view of total students, games played, XP distributed, and engagement metrics across all institutes.
- **Seed Tooling:** Admin-invocable scripts (`seed_master_games.py`, `seed_questions.py`, `seed_test_users.py`) to bootstrap platforms with default STEM content.

---

## Technologies Used

### Programming Languages
- Python 3.10
- JavaScript (ES6+)
- HTML5 / CSS3

### Libraries & Frameworks
- Django 5.2 — Web framework
- TailwindCSS — Utility-first frontend styling
- Chart.js — Interactive data visualisation
- django-extensions — Development utilities

### Artificial Intelligence
- Google Generative AI SDK (`google-generativeai`) — Gemini Flash model
- Subject-scoped prompt engineering for AI tutor boundary enforcement

### Backend
- Django ORM — Database abstraction layer
- Django Signals — Event-driven notification dispatch
- Custom service layer (`gamification/services.py`, `analytics/utils.py`) — Business logic isolation
- Django SMTP — Email OTP and password reset delivery (Gmail SMTP / TLS)
- `python-dotenv` — Secure environment variable management
- `whitenoise` — Static file serving in production
- `gunicorn` — Production WSGI server

### Frontend
- Django Template Language (DTL) — Server-side rendering
- JavaScript (vanilla) — Game interactivity, canvas rendering, drag-and-drop logic
- HTML5 Canvas API — Canvas-based action game implementation

### Database
- SQLite 3 — Development environment
- MySQL — Production environment

### Deployment & DevOps
- AWS EC2 — Cloud hosting
- `gunicorn` + `whitenoise` — Production server stack
- `.env` file-based secrets management

### Security
- Django's built-in CSRF middleware
- Custom role-based access decorators (`accounts/decorators.py`)
- Email OTP verification for account activation
- Token-based password reset (`accounts/tokens.py`)
- Django password validators (similarity, minimum length, common password, numeric)

### Document & Media
- Pillow — Profile picture and game thumbnail processing
- ReportLab — PDF generation
- PyPDF2 — PDF parsing
- `qrcode` — QR code generation

---

## Dataset

RuralEdu STEM Quest does not consume an external ML dataset. Instead, its data layer is structured around an internally seeded relational schema:

**Academic Content:** Subject question banks for STEM disciplines (Physics, Chemistry, Mathematics, Biology, and related subjects) for Classes 6–12, seeded via `seed_questions.py`. Questions follow a four-option MCQ format with optional image attachments for diagram-based questions.

**Game Configurations:** Master game templates seeded via `seed_master_games.py`, defining game metadata, grade ranges, XP rewards, difficulty levels, and JSON runtime configurations per game type.

**User and Activity Data:** Transactional data generated organically through student gameplay — `StudentAttempt`, `StudentEngagement`, `XPLog`, `PerformanceSnapshot` — persists to the database and drives all analytics and gamification calculations.

**Test Users:** Role-stratified test users seeded via `seed_test_users.py`, covering student accounts per class and stream, teacher accounts, and guardian accounts, enabling QA without manual registration.

No personally identifiable training data is used. All AI inference is performed at runtime via the Gemini API with zero persistent model fine-tuning.

---

## System Architecture

The platform is built on a modular Django application architecture with a clear separation of concerns across ten apps:

```
┌──────────────────────────────────────────────────────────┐
│                     Presentation Layer                   │
│      Django Templates · TailwindCSS · Chart.js · Canvas  │
└────────────────────────┬─────────────────────────────────┘
                         │ HTTP / WSGI
┌────────────────────────▼─────────────────────────────────┐
│                     Application Layer                    │
│                                                          │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌─────────┐  │
│  │ accounts │  │ academics│  │  games   │  │chatbot  │  │
│  │  (auth / │  │(classes/ │  │ (engine/ │  │(Gemini  │  │
│  │  roles / │  │subjects/ │  │questions/│  │ tutor)  │  │
│  │dashboards│  │ streams) │  │ attempts)│  │         │  │
│  └──────────┘  └──────────┘  └────┬─────┘  └─────────┘  │
│                                   │                      │
│  ┌─────────────┐  ┌───────────┐   │  ┌────────────────┐  │
│  │gamification │  │ analytics │◄──┘  │  mentorship    │  │
│  │ (XP/levels/ │  │(engagement│      │ (requests/     │  │
│  │  badges/    │  │/snapshots)│      │  assignments/  │  │
│  │  streaks)   │  └───────────┘      │  messaging)    │  │
│  └──────┬──────┘                     └────────────────┘  │
│         │ Django Signals                                  │
│  ┌──────▼─────────┐   ┌────────────┐                     │
│  │ notifications  │   │ institutes │                     │
│  │ (in-app alerts)│   │(multi-inst)│                     │
│  └────────────────┘   └────────────┘                     │
└────────────────────────┬─────────────────────────────────┘
                         │ Django ORM
┌────────────────────────▼─────────────────────────────────┐
│                      Data Layer                          │
│             SQLite (dev) / MySQL (production)            │
└──────────────────────────────────────────────────────────┘
                         │
┌────────────────────────▼─────────────────────────────────┐
│                   External Services                      │
│    Google Gemini API · Gmail SMTP · AWS EC2 Storage      │
└──────────────────────────────────────────────────────────┘
```

**Data Flow Summary:**
1. A student authenticates via email OTP and is redirected to their role-specific dashboard.
2. Games are filtered by the student's institute, academic class, and stream before being presented.
3. Upon game completion, a `StudentAttempt` record is created and the `award_xp_to_student` service runs atomically — updating `StudentXP`, `SubjectXP`, and `XPLog`, checking level thresholds, and evaluating badge unlock eligibility.
4. Django signals dispatch in-app notifications for level-ups and badge unlocks without coupling the gamification app to the notifications app.
5. Analytics views aggregate `StudentAttempt`, `StudentEngagement`, and `PerformanceSnapshot` records to render charts and intel feeds for teachers.

---

## Model Workflow

### Gamification Pipeline

```
Student Completes Game
        │
        ▼
  Record StudentAttempt (score, xp_earned, time_spent)
        │
        ▼
  award_xp_to_student() [atomic transaction]
        │
        ├──► Update StudentXP.total_xp
        │         │
        │         ├──► calculate_level() → compare against Level table
        │         │         └──► Level Up? → create Notification("Level Up 🚀")
        │         │
        │         └──► update_streak() → check last_activity_date delta
        │
        ├──► Update SubjectXP.xp (per subject)
        │
        ├──► Create XPLog record (audit trail)
        │
        └──► unlock_badges_logic()
                  │
                  └──► For each Badge where xp_threshold ≤ total_xp:
                            └──► get_or_create StudentBadge
                                      └──► New? → create Notification("New Achievement 🏆")
```

### AI Mentor Workflow

```
Student Sends Message (subject context selected)
        │
        ▼
  ChatSession record created/retrieved
        │
        ▼
  generate_subject_response(subject_name, question)
        │
        ├──► Configure Gemini API (GEMINI_API_KEY from env)
        │
        ├──► Build system prompt with subject boundary rules
        │
        ├──► Call gemini-flash-latest model with full_prompt
        │
        └──► Return structured response (bold headings, bullets, summary)
                  │
                  ▼
        ChatMessage stored (sender=student, bot_response=text)
                  │
                  ▼
        Response rendered in chat interface
```

### Authentication Workflow

```
User Registers (email + role + institute + class/stream)
        │
        ▼
  EmailOTP created → 6-digit OTP dispatched via Gmail SMTP
        │
        ▼
  User submits OTP on verify_otp view
        │
        ├──► OTP valid and not expired (10-minute window)?
        │         └──► Set is_verified=True → Redirect to login
        │
        └──► OTP expired or invalid?
                  └──► Display error, allow resend
```

---

## Implementation Details

**Custom User Model:** The `accounts.User` model extends `AbstractUser` with email as the primary authentication field (`USERNAME_FIELD = "email"`), eliminating the username field. Four roles — Admin, Teacher, Student, Guardian — are enforced at the model level. Students carry foreign keys to `Institute`, `AcademicClass`, `Stream`, and optionally a `mentor` (teacher) and `guardian` (guardian user). This single-model approach avoids multi-table inheritance complexity while supporting all role-specific profile attributes.

**Role-Based Access Control:** A custom decorator in `accounts/decorators.py` inspects `request.user.role` and redirects unauthorized access attempts. Each view is decorated with the required role, ensuring strict separation between role-specific pages without relying solely on Django's built-in permission system.

**Transactional XP Service:** The `award_xp_to_student` function in `gamification/services.py` is wrapped in `@transaction.atomic`, guaranteeing that XP updates, subject XP updates, log creation, and badge evaluation either all succeed or all roll back atomically. This prevents data inconsistency when concurrent game submissions occur.

**Notification Signals:** The `notifications/signals.py` module listens for post-save events on `StudentBadge` and other models to dispatch notifications without creating hard imports between apps, maintaining decoupled app boundaries.

**Context Processors:** `notifications/context_processors.py` injects the unread notification count into every authenticated template render, enabling the notification badge in the navigation bar without explicit per-view queryset injection.

**Game JSON Configuration:** Each `Game` instance carries a `config` JSONField allowing admins to tune runtime parameters per game (time limits, question counts, physics parameters for canvas games) without code changes or redeployment, making content management accessible to non-developers.

**Analytics Snapshots:** The `PerformanceSnapshot` model caches aggregate student metrics (total XP, total games played, total minutes spent) and is refreshed via `analytics/utils.py`. This avoids expensive aggregate queries on every teacher dashboard load.

**Seed Scripts:** Python scripts (`seed_master_games.py`, `seed_questions.py`, `seed_test_users.py`, `seed_test_data.py`) are provided outside of Django's management command framework for portability. Each script uses Django ORM calls after `django.setup()` to populate the database with curriculum-aligned content for immediate demonstration.

---

## Results

RuralEdu STEM Quest demonstrates measurable impact across its core educational and technical objectives:

| Metric | Result |
|---|---|
| Supported Academic Classes | Class 6 to Class 12 |
| Streams Supported | MPC, BIPC |
| Game Types Implemented | 4 (Quiz, Canvas, Drag-Drop, Simulation) |
| Roles Supported | 4 (Admin, Teacher, Student, Guardian) |
| Gamification Mechanics | XP, Levels, Badges, Streaks, Leaderboard |
| AI Integration | Google Gemini Flash (subject-scoped) |
| Authentication Security | Email OTP + Token Password Reset |
| Deployment Target | AWS EC2 (Gunicorn + Whitenoise) |
| Database Support | SQLite (dev), MySQL (prod) |
| Notification Types | System, XP, Badge, Mentor, Message |

**Engagement Design Outcomes:**
- The XP and streak system creates measurable daily return incentives, with streak counters providing social accountability.
- Badge thresholds are calibrated to recognise early (Quick Solver), mid-tier (Science Pro), and expert-level (STEM Champion) performance milestones.
- The intel feed in teacher dashboards reduces the manual effort of identifying at-risk students by surfacing anomalies automatically.
- The AI chatbot's subject boundary enforcement prevents off-topic usage while maintaining the accuracy of curriculum-aligned responses.

---

## Installation

### Prerequisites

- Python 3.10 or higher
- pip (Python package manager)
- Git
- A Gmail account with App Password enabled (for email OTP)
- A Google Gemini API key

### Step 1 — Clone the Repository

```bash
git clone https://github.com/harisaigithub/Gamified-Project-.git
cd Gamified-Project-
```

### Step 2 — Create and Activate a Virtual Environment

```bash
python -m venv venv

# Windows
venv\Scripts\activate

# macOS / Linux
source venv/bin/activate
```

### Step 3 — Install Dependencies

```bash
pip install -r req.txt
```

### Step 4 — Configure Environment Variables

Create a `.env` file in the project root directory with the following variables:

```env
SECRET_KEY=your-django-secret-key
DEBUG=True
EMAIL_HOST_USER=your-gmail-address@gmail.com
EMAIL_HOST_PASSWORD=your-gmail-app-password
GEMINI_API_KEY=your-google-gemini-api-key
```

> **Note:** Generate a Gmail App Password at `https://myaccount.google.com/apppasswords` with 2-Step Verification enabled. Do not use your regular Gmail password.

### Step 5 — Apply Database Migrations

```bash
python manage.py migrate
```

### Step 6 — Create a Superuser (Admin Account)

```bash
python manage.py createsuperuser
```

Follow the prompts to set an email address and password for the admin account.

### Step 7 — Seed Initial Data (Recommended)

Populate the database with STEM game templates and question banks:

```bash
python seed_master_games.py
python seed_questions.py
python seed_test_users.py
```

### Step 8 — Collect Static Files (Production Only)

```bash
python manage.py collectstatic
```

### Step 9 — Run the Development Server

```bash
python manage.py runserver
```

Navigate to `http://127.0.0.1:8000/` to access the platform.

### Production Deployment (AWS EC2)

For production deployment with Gunicorn and Whitenoise:

```bash
pip install gunicorn whitenoise

# Start the application server
gunicorn config.wsgi:application --bind 0.0.0.0:8000 --workers 3
```

Configure your EC2 security group to allow inbound traffic on port 8000 (or use Nginx as a reverse proxy on port 80).

---

## Usage

### Accessing the Platform

After running the development server, navigate to `http://127.0.0.1:8000/`.

### Test Credentials

The following test users are available after running `seed_test_users.py`:

| Role | Email | Class / Stream |
|---|---|---|
| Student | student_class6@test.com | Class 6 |
| Student | student_class11mpc@test.com | Class 11 MPC |
| Student | student_class11bipc@test.com | Class 11 BIPC |
| Student | student_class12mpc@test.com | Class 12 MPC |
| Teacher | teacher@test.com | — |
| Guardian | guardian@test.com | — |

All test accounts use the password: `Test@1234` (verify with `seed_test_users.py` if customised).

### Student Workflow

1. Log in with a student account and complete email OTP verification on first use.
2. Navigate to **Mission Control** to browse available STEM games filtered to your class and stream.
3. Select a game, review the XP reward and difficulty, and click **Play**.
4. Complete the game to earn XP, which updates your profile, level, and streak in real time.
5. Visit **My Progress** to review subject-wise mastery charts and badge achievements.
6. Open the **AI Mentor** chatbot, select your subject, and ask academic questions.
7. Use **Mentor Messaging** to communicate with your assigned teacher.

### Teacher Workflow

1. Log in with a teacher account to access the Teacher Dashboard.
2. Review the **Intel Feed** for students requiring attention.
3. Navigate to **Student Analytics** for detailed per-student performance data.
4. Use **Mentor Messaging** to send academic guidance to students.

### Admin Workflow

1. Access the admin panel at `/admin/` with the superuser account.
2. Create an **Institute** and associate academic classes, streams, and subjects to it.
3. Create **Games** via the Games admin panel, setting subject, type, grade range, difficulty, XP reward, and JSON config.
4. Add **Questions** to quiz-type games through the Question admin model.
5. Manage all user accounts and view platform-wide analytics via the Admin Dashboard.

---

## Future Enhancements

- **AI-Personalised Learning Paths:** Integrate a recommendation engine that analyses `XPLog` and `StudentAttempt` history to suggest targeted game sequences and weak-area remediation.
- **Progressive Web App (PWA):** Add a service worker and web app manifest to enable offline game caching and mobile installation, addressing connectivity constraints in rural areas.
- **Offline Mode:** Cache question banks and game assets locally using the Cache API, allowing students to play previously loaded games without an active internet connection.
- **Inter-Institute Leaderboards and Competitions:** Extend the leaderboard to support time-bounded inter-institute tournaments with seasonal reset mechanics.
- **Predictive Analytics for At-Risk Detection:** Train a classification model on `PerformanceSnapshot` and `StudentEngagement` data to proactively surface students at risk of academic disengagement before their performance deteriorates.
- **Video Lesson Integration:** Embed subject-linked video lessons within the game detail pages, providing conceptual grounding before competitive gameplay.
- **Multi-Language Support:** Internationalise the UI with Django's i18n framework to support regional languages (Telugu, Hindi, Tamil) for improved accessibility in rural contexts.
- **Mobile Application:** Develop a Flutter or React Native companion app providing native mobile access to the game catalogue and progress dashboard.
- **Teacher-Authored Content:** Allow teachers to create and publish their own quiz games and question sets within their institute scope, reducing dependence on admin-level content creation.
- **Guardian Communication Channel:** Add a direct messaging module between guardians and teachers to facilitate parent-teacher academic discussions within the platform.

---

## Contributing

Contributions to RuralEdu STEM Quest are welcome. Please follow the standard Git workflow outlined below.

### Step 1 — Fork the Repository

Click **Fork** on the GitHub repository page to create a personal copy under your GitHub account.

### Step 2 — Clone Your Fork

```bash
git clone https://github.com/your-username/Gamified-Project-.git
cd Gamified-Project-
```

### Step 3 — Create a Feature Branch

Use a descriptive branch name that reflects the change:

```bash
git checkout -b feature/add-offline-mode
```

### Step 4 — Implement Your Changes

Make your code changes, ensuring adherence to Django's coding conventions and the project's modular app structure. Add tests where applicable.

### Step 5 — Commit with a Descriptive Message

```bash
git add .
git commit -m "feat: implement service worker for offline game caching"
```

### Step 6 — Push to Your Fork

```bash
git push origin feature/add-offline-mode
```

### Step 7 — Open a Pull Request

Navigate to the original repository on GitHub and open a **Pull Request** from your feature branch against the `main` branch. Include a clear description of the changes and the problem they address.

### Code Standards

- Follow PEP 8 for Python code formatting.
- Keep business logic within service modules (`<app>/services.py`) rather than in views.
- Do not commit secrets, API keys, or `.env` files to version control.
- Write docstrings for all service functions and non-trivial model methods.

---

## License

```
MIT License

Copyright (c) 2026 RuralEdu STEM Quest Team

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

This project was developed as a final-year engineering capstone by a four-member team at a rural engineering institute in Andhra Pradesh, India.
--- 
P HARI SAI 
---

---

&nbsp;

&nbsp;

## ***To acquire this project, please contact***

**GitHub:** https://github.com/harisaigithub

**Email:** [harisaiparasa@gmail.com](mailto:harisaiparasa@gmail.com)

For collaborations, customizations, or deployment support, feel free to reach out.