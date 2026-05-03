# Smart Classroom Timetable Scheduler

### *A Heuristic-Driven, AI-Assisted Django Web Application for Automated Academic Schedule Generation with Multi-Constraint Enforcement*

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

Manual timetable construction in academic institutions is a computationally complex and error-prone process. Schedulers must simultaneously satisfy dozens of constraints — faculty availability windows, subject credit requirements, classroom capacity limits, lab-type restrictions, workload ceilings, and no-collision rules — across multiple departments, sections, and semesters. Errors cascade rapidly, resulting in scheduling conflicts, underutilised rooms, and overloaded faculty.

The **Smart Classroom Timetable Scheduler** is a full-stack Django web application that eliminates this operational burden through an automated, heuristic-based schedule generation engine. The system ingests structured academic data — departments, semesters, sections, subjects, faculty profiles, and classroom inventories — and produces a complete, conflict-free weekly timetable across all sections in a single click. The generated schedule respects faculty availability matrices, enforces per-faculty daily and weekly teaching limits, matches laboratory subjects to appropriate room types, and prevents both teacher and classroom double-bookings simultaneously.

The platform is designed for Head-of-Department (admin) users who manage institutional infrastructure and faculty members who interact with their personalised schedules, with a public-facing view that allows students and stakeholders to access timetables without authentication.

---

## Objectives

- Design and implement a four-module heuristic scheduling engine capable of generating conflict-free timetables across multiple sections, departments, and semesters in a single atomic database transaction.
- Enforce multi-dimensional constraint satisfaction including faculty availability windows, daily and weekly workload limits, department-subject-faculty alignment, classroom capacity validation, and lab-type room matching.
- Build a role-segregated web platform with distinct Admin/HOD and Faculty/User dashboards, each surfacing contextually relevant analytics and management controls.
- Provide faculty members with an interactive availability matrix through which they can mark busy periods that the scheduler respects during generation.
- Enable one-click PDF export of section-specific timetables in landscape A4 format with subject codes, classroom names, and faculty identifiers.
- Implement a timetable publication workflow that dispatches in-app notifications to all assigned faculty upon schedule release.
- Support a public-facing timetable view accessible without login for student and stakeholder visibility.
- Integrate Google Gemini AI for intelligent assistance within the scheduling workflow.
- Deliver a demo data seeding command that bootstraps a realistic academic infrastructure for rapid evaluation and testing.

---

## Summary

The Smart Classroom Timetable Scheduler is a Django-based academic management platform built around a four-module heuristic timetable generation engine. It replaces manual scheduling workflows with a single-click automated process that produces a complete weekly timetable for all configured sections while enforcing a comprehensive constraint ruleset.

The system operates under two user roles. Administrators (HODs) manage all academic infrastructure — departments, semesters, sections, subjects, classrooms, and faculty profiles — and trigger the scheduling engine. Faculty members access personalised dashboards showing their assigned schedule, a today's classes view based on the current weekday, and a workload ring visualisation. They also submit their availability constraints via an interactive matrix that the engine consults during generation.

The generation engine iterates over all sections and available teaching time slots, applying four sequential modules per slot: heuristic subject selection (respecting weekly credit targets and preventing consecutive repetition), room allocation (matching lab subjects to lab-type rooms and validating seating capacity), smart faculty allocation (ensuring department alignment, availability, daily/weekly limit compliance, and collision freedom), and final entry creation inside a single database transaction.

Generated timetables are viewable as interactive grid tables filterable by section, exportable as formatted PDF documents via ReportLab, and publishable with automatic in-app notification dispatch to all affected faculty members.

---

## Features

### Before Login (Public Access)

- **Landing Page** — Informative homepage presenting platform features, the scheduling workflow, and institutional benefits.
- **Public Timetable View** — A publicly accessible section-filterable timetable grid displaying the published schedule without requiring login, suitable for students and institutional stakeholders.
- **User Registration** — Email-based signup supporting Admin/HOD and Faculty/User roles with OTP-based email verification.
- **Email OTP Verification** — Upon registration, a six-digit OTP with a ten-minute expiry is dispatched via SMTP. Accounts remain inactive until OTP confirmation.
- **Password Reset** — Full tokenised password reset flow via email using Django's built-in secure reset framework with custom templates.

### After Login — Admin / HOD Role

**Dashboard and Analytics**
- Platform-wide statistics: total departments, classrooms, faculty members, and active sections.
- Room utilisation ring (SVG-based circular indicator) computed as the ratio of filled timetable entries to the theoretical maximum of active time slots multiplied by available classrooms.
- Department-wise class count bar chart rendered via Chart.js.
- Recent timetable activity feed and busy faculty count indicator.

**Department Management**
- Full CRUD operations for academic departments with name and code fields.
- Protected deletion preventing cascading loss of linked subjects or faculty.

**Classroom Management**
- CRUD for classroom inventory with name, seating capacity, room type (Classroom or Laboratory), and availability toggle.
- Dynamic weekly utilisation percentage computed per room from assigned timetable entries.
- Protected deletion for classrooms currently assigned in an active timetable.

**Subject Management**
- CRUD for subjects with code, name, department FK, semester FK, subject type (Theory or Laboratory), credit hours, and classes-per-week targets.
- Subject type and code suffix drive the engine's room-type routing logic.

**Semester and Section Management**
- Creation of Semester records with academic year and number fields.
- Section configuration linking department, semester, and student count for capacity validation and subject filtering.

**Faculty Management**
- CRUD for faculty profiles linked to system user accounts of role 'faculty'.
- Configurable per-faculty teaching limits: maximum classes per day and maximum classes per week.
- Department assignment ensuring only department-matched subjects are allocated.

**Timetable Generation**
- Single-click full-week timetable generation via the heuristic engine.
- Pre-generation confirmation page showing the number of configured sections.
- Atomic database transaction ensuring all-or-nothing generation with rollback on failure.
- Post-generation message reporting actual versus expected slot fill count.

**Timetable Grid View**
- Section-filterable weekly grid displaying subject codes, classroom names, and faculty identifiers per slot.
- Break slots rendered distinctly with their configured break labels.

**PDF Export**
- Per-section timetable export as a landscape A4 PDF with a formatted table of time labels, subject codes, classroom names, and teacher identifiers.
- Generated dynamically via ReportLab with styled table headers (blue background, white text).

**Timetable Publication**
- One-click publish action creating in-app Notification records for every faculty member with at least one assigned class.

**Constraint Configuration**
- View and manage ConstraintRule records: no faculty overlap, no room overlap, capacity check, lab timing constraint.
- FixedSlot records for pre-pinning specific subject-faculty-classroom-timeslot-section combinations the engine must preserve.

**Feedback Management**
- Admin-facing list of all user-submitted feedback with subject, message, timestamp, and status.
- Status workflow: Pending → In Progress → Resolved.

### After Login — Faculty / User Role

**Faculty Dashboard**
- Personalised view of all assigned timetable entries.
- Today's schedule panel dynamically filtered to the current weekday.
- Workload ring (SVG) showing percentage of weekly load relative to the configured maximum.
- Total distinct subjects count.

**Availability Matrix**
- Interactive grid spanning all configured days and teaching periods.
- Faculty mark individual slots as Busy; these are read by Module 3 of the engine during generation.
- Submission atomically clears and rewrites the faculty's availability records.

**Notifications**
- Notification list in reverse chronological order; all notifications marked as read upon page access.

**Feedback Submission**
- Submit feedback with a subject line and message body; tracked with timestamps and admin-managed resolution statuses.

---

## Technologies Used

### Programming Languages
- Python 3.x

### Libraries and Frameworks
- Django 5.2.10
- django-crispy-forms 2.5
- crispy-bootstrap5 2025.6
- ReportLab 4.4.9
- Pillow 12.1.0
- python-dotenv 1.2.1

### Artificial Intelligence / AI Integration
- Google Generative AI SDK (`google-genai` 1.61.0)
- API key managed via `.env` environment variable (`AI_API_KEY`)
- Integrated for AI-assisted scheduling decision support via the Gemini model

### Backend
- Django MVT architecture with Class-Based Views (CRUD) and Function-Based Views (domain logic)
- Django ORM with SQLite persistence
- Custom User model extending AbstractUser with email as USERNAME_FIELD (no username field)
- Custom CustomUserManager overriding create_user and create_superuser
- Django Sessions for authentication state
- SMTP email backend for OTP and password reset
- `transaction.atomic()` wrapping timetable generation for full rollback safety

### Frontend
- Django Template Language with multi-level block inheritance (base1.html, base_dashboard.html)
- Bootstrap 5 via crispy-bootstrap5
- Chart.js for admin bar charts and SVG workload ring rendering
- HTML5 / CSS3 / JavaScript
- Custom CSS (static/css/style.css)

### Database
- SQLite (default; suitable for single-instance academic deployments)

### Deployment and DevOps
- Django development server (manage.py runserver)
- `.env`-based configuration via python-dotenv
- Custom management command: `setup_demo_data` for bootstrapping academic infrastructure

### Security
- OTP-based email verification with a ten-minute expiry on all new accounts
- Email-as-username authentication (no username field on the User model)
- Role-based access control enforced at every view (user.role checks, LoginRequiredMixin)
- CSRF protection enabled globally via Django middleware
- Tokenised password reset via Django's secure built-in framework
- Account activation gate: is_active=False accounts blocked at login

---

## System Architecture

The platform follows a modular Django MVT architecture composed of six discrete Django applications.

```
config/                         # Project settings, root URLconf, WSGI/ASGI
├── settings.py                 # Centralised configuration with dotenv
├── urls.py                     # Root URL dispatcher

accounts/                       # Custom User model (email-based auth), OTP,
│                               # admin/faculty dashboards, login/signup/verify
├── management/commands/
│   └── setup_demo_data.py      # Demo data seeding (idempotent)

core/                           # Department, Classroom, Semester, Section,
│                               # Subject, Feedback — full CRUD via CBVs

faculty/                        # Faculty profile (workload limits, department),
│                               # FacultyAvailability (busy-slot matrix)

scheduler/                      # TimeSlot (day/time/break), ScheduleConfig,
│                               # FixedSlot, ConstraintRule — config layer

timetable/                      # TimetableEntry model, 4-module heuristic engine,
│                               # grid view, PDF export, publish, public view

notifications/                  # Per-user Notification records, list view
```

**Data Flow:**
1. Requests dispatched by root URLconf to sub-application URL modules.
2. Views enforce `@login_required` and `user.role` checks before processing.
3. The generation engine executes inside `transaction.atomic()`, iterating section × day × slot, applying four constraint modules, writing TimetableEntry records.
4. ORM communicates with SQLite for all persistence operations.
5. ReportLab builds PDF responses directly into the HTTP response stream.
6. The Google Gemini SDK is invoked via AI_API_KEY for AI-assisted features.

---

## Model Workflow

### Four-Module Heuristic Generation Engine

Triggered by Admin POST to `/timetable/generate/`. Executes atomically.

**Pre-generation:** All existing TimetableEntry records are deleted. Classrooms, faculty, and sections are loaded into memory. Per-section subject usage counters are initialised from `classes_per_week` targets.

**Module 1 — Heuristic Subject Selection**
For each (section, day, teaching slot) triplet, subjects are filtered to the section's department and semester. A subject is skipped if its weekly usage count has reached `classes_per_week` (credit gate) or if it matches the previous slot's subject (consecutive-repetition prevention).

**Module 2 — Room Allocation**
For each candidate subject, available classrooms are iterated. Lab-typed subjects (subject_type='lab' or code ending in 'L') are restricted to lab-type or Lab-named rooms. Rooms with capacity below section's `student_count` are rejected. Rooms already booked at the same day-slot for any section are rejected via ORM `exists()`.

**Module 3 — Smart Faculty Allocation (Balanced and Rotational)**
Faculty are shuffled randomly before each iteration to ensure balanced distribution. Filters applied sequentially: department match, availability matrix check (is_available=False rejects), daily load check (max_classes_per_day), weekly load check (max_classes_per_week), and collision check (already assigned another section at same day-slot).

**Module 4 — Entry Creation**
A TimetableEntry record is created if both suitable_room and suitable_teacher are identified. Subject usage counter is incremented; previous-subject tracker is updated.

**Post-generation Validation:** Engine compares actual vs. expected entry count and issues a success or partial-fill warning message.

### Faculty Availability Workflow
1. Faculty access `/faculty/availability/`.
2. All existing FacultyAvailability records for the user are deleted.
3. Checked "Busy" cells create new FacultyAvailability records with `is_available=False`.
4. These records are consulted by Module 3 during generation.

### Publication Workflow
1. Admin POSTs to `/timetable/publish/`.
2. Distinct faculty user IDs present in TimetableEntry records are queried.
3. A Notification record is created for each affected faculty user.
4. Admin redirected to grid view with success confirmation.

---

## Implementation Details

**Email-First Authentication:** The custom User model sets `USERNAME_FIELD = 'email'` and nullifies `username`. CustomUserManager normalises and validates email. OTP dispatched via SMTP activates accounts.

**Atomic Generation:** Deletion of prior entries and creation of all new entries are wrapped in `transaction.atomic()`. Any exception rolls back all changes, preserving database integrity.

**Classroom Utilisation:** Each Classroom exposes `get_weekly_utilization()` — a dynamic ORM method computing the percentage of non-break slots the room is occupied.

**SVG Ring Metric:** Both the admin utilisation ring and faculty workload ring use `stroke-dashoffset = 502.4 − (502.4 × pct / 100)`, where 502.4 is the circumference of a radius-80 circle, computed server-side and injected into the template context.

**PDF Generation:** `export_timetable_pdf` writes a SimpleDocTemplate directly to an HttpResponse. Faculty identifiers are extracted as the email prefix before `@`. Table styling uses a blue header (`#1E3A8A`), grid lines, centred alignment, and 7pt font for landscape A4 legibility.

**Demo Data Command:** `setup_demo_data` creates departments (CSE, ECE, MECH), classrooms (LH-101, LH-102, CS-LAB-1), time slots (09:00–10:00, 10:00–11:00, 11:00–11:30 break, 11:30–12:30) across Mon–Fri, a demo faculty user, and Semester 4 / Section A for CSE. All entities use `get_or_create` for idempotency.

**Constraint Rule Registry:** ConstraintRule records (faculty_clash, room_clash, capacity_check, lab_timing) document the enforced ruleset and are surfaced in a dedicated admin view, providing an auditable record of scheduling policy.

**Fixed Slots:** FixedSlot records allow pre-pinning of specific subject-faculty-classroom-timeslot-section combinations that the engine must not move, supporting recurring institutional requirements such as dedicated lab sessions.

---

## Results

| Metric | Value |
|---|---|
| User Roles | 2 (Admin/HOD, Faculty/User) |
| Generation Engine Modules | 4 (Subject Selection, Room Allocation, Faculty Allocation, Entry Creation) |
| Constraint Checks per Slot | 5 (Department, Availability, Daily Limit, Weekly Limit, Collision) |
| Constraint Rule Types | 4 (Faculty Clash, Room Clash, Capacity, Lab Typing) |
| Working Days Supported | 6 (Monday – Saturday) |
| Room Types | 2 (Classroom, Laboratory) |
| Subject Types | 2 (Theory, Laboratory) |
| PDF Export Format | Landscape A4 per section |
| Notification Trigger | Timetable Publication (per assigned faculty) |
| OTP Validity Window | 10 minutes |
| Generation Scope | All sections in a single atomic transaction |
| Demo Seed Command | `python manage.py setup_demo_data` |

**Conflict Prevention:** All collision checks in Modules 2 and 3 use ORM `exists()` queries against the live TimetableEntry table during the atomic transaction, ensuring every newly created entry is validated against all previously committed entries within the same generation run.

**Balanced Allocation:** Random shuffling of the faculty list before each slot assignment prevents deterministic favouritism of any single faculty member, producing a more evenly distributed workload across the academic week.

---

## Installation

### Prerequisites

- Python 3.8 or higher
- pip
- Git
- Gmail account or SMTP credentials for OTP email delivery

### Step 1 — Clone the Repository

```bash
git clone https://github.com/harisaigithub/Smart-Classroom-Timetable-Scheduler.git
cd Smart-Classroom-Timetable-Scheduler
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

### Step 4 — Configure Environment Variables

Create a `.env` file in the project root:

```env
AI_API_KEY=your-google-gemini-api-key
EMAIL_HOST_USER=your-email@gmail.com
EMAIL_HOST_PASSWORD=your-gmail-app-password
```

To obtain a Gmail App Password, enable two-factor authentication and generate an app-specific password under Google Account Security. To generate a Django `SECRET_KEY`:

```bash
python -c "from django.core.management.utils import get_random_secret_key; print(get_random_secret_key())"
```

Update `config/settings.py` with the generated key before deploying to production.

### Step 5 — Apply Database Migrations

```bash
python manage.py migrate
```

### Step 6 — Create an Admin Account

```bash
python manage.py createsuperuser
```

Provide an email address and password. Assign the `admin` role via the Django admin panel at `/admin/`.

### Step 7 — (Optional) Seed Demo Data

```bash
python manage.py setup_demo_data
```

Creates three departments, three classrooms, a morning time slot grid (Mon–Fri), a demo faculty user (`faculty_demo@college.edu` / password `demo1234`), and a sample Semester 4 / Section A for CSE.

### Step 8 — Start the Development Server

```bash
python manage.py runserver
```

Access the application at `http://127.0.0.1:8000`.

---

## Usage

### Admin / HOD Workflow

1. Register with the Admin/HOD role and verify email via OTP.
2. Log in — redirected to the **Admin Dashboard**.
3. Add **Departments**, **Classrooms**, **Semesters**, **Sections**, and **Subjects** via the core management pages.
4. Ensure faculty users are registered, verified, and have **Faculty profiles** created with department and workload limits.
5. Review faculty **Availability Matrices** to confirm busy period submissions.
6. Navigate to `/timetable/generate/` and click **Generate** to produce the full weekly schedule.
7. View the grid at `/timetable/view/`, filter by section, and export per-section PDFs at `/timetable/export/pdf/<section_id>/`.
8. Click **Publish** at `/timetable/publish/` to notify all assigned faculty.

### Faculty / User Workflow

1. Register with the Faculty/User role and verify email.
2. Log in — redirected to the **Faculty Dashboard** with assigned schedule, today's classes, and workload ring.
3. Go to `/faculty/availability/` to mark Busy periods before the next generation run.
4. Check `/notifications/` after timetable publication for schedule alerts.
5. Submit feedback via `/core/feedback/add/` for scheduling concerns.

### Public Access

Students and guests access `/timetable/public/`, select a section from the dropdown, and view the complete weekly timetable without login.

---

## Future Enhancements

- **Genetic Algorithm / CP-SAT Engine** — Replace heuristic generation with a formal optimisation solver for large-scale institutions with hundreds of sections, enabling globally optimal schedules with soft-constraint weighting.
- **Drag-and-Drop Timetable Editor** — JavaScript-based interactive grid editor allowing manual slot adjustments with real-time conflict warnings without full regeneration.
- **PostgreSQL Migration** — Transition from SQLite to PostgreSQL for concurrent access, row-level locking during generation, and production-grade query performance.
- **Multi-Semester Concurrent Scheduling** — Extend the engine to generate and manage timetables for multiple semesters simultaneously.
- **iCalendar Export** — Generate `.ics` files from faculty schedules for direct import into Google Calendar and Outlook.
- **Substitution Management** — Module for recording faculty substitutions with automatic notification to affected parties.
- **Mobile-Responsive PWA** — Progressive Web App with offline timetable caching for connectivity-free access.
- **AI-Powered Conflict Resolution** — Leverage the integrated Gemini API to suggest optimal resolution paths when the scheduler cannot fill all slots due to resource constraints.
- **Student Role** — Add a student user role with section-specific timetable and announcement notification access.
- **Audit Log** — Timestamped trail of all generation events, publications, and manual overrides for institutional governance and accreditation reporting.

---

## Contributing

Contributions are welcome. Please follow the standard Git workflow:

1. Fork the repository.
2. Create a feature branch:
   ```bash
   git checkout -b feature/your-feature-name
   ```
3. Commit your changes:
   ```bash
   git commit -m "Add: brief description of the change"
   ```
4. Push to your fork:
   ```bash
   git push origin feature/your-feature-name
   ```
5. Open a Pull Request against the `main` branch.

Ensure code follows PEP 8, existing migrations remain unmodified, and new views include appropriate role checks and `@login_required` decorators. For engine changes, describe the constraint behaviour in the PR description.

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