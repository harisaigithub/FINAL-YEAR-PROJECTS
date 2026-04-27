# SmartPilgrim

> A comprehensive Django-based digital pilgrimage management platform integrating AI crowd prediction, QR-based darshan slot booking, real-time safety alerts, and temple donation management for Hindu sacred sites across South India.

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

Hindu temples across South India receive millions of pilgrims annually, yet the infrastructure supporting visitor management, crowd control, emergency response, and donation processing remains largely manual and fragmented. Long queues, overcrowded darshan halls, loss of children, and medical emergencies during festivals represent persistent challenges that neither pilgrims nor temple authorities have adequate digital tools to address.

SmartPilgrim bridges this gap by providing a full-featured, role-differentiated web platform that digitizes the entire temple visit lifecycle — from registration and slot booking to AI-driven crowd forecasting, on-site QR validation, emergency SOS dispatch, and financial reporting. The platform is designed around two user roles: **Pilgrims (Devotees)**, who interact with the booking, safety, and donation subsystems through a personalized dashboard, and **Admins (Temple Authorities)**, who oversee operations through a dedicated management console with real-time metrics, QR scanner integration, and SOS monitoring.

The project is implemented as a B.Tech 4th Year capstone using Django 5.2 and SQLite, with a modular app architecture that isolates each functional domain into a self-contained Django application.

---

## Objectives

- Implement a secure, email-verified user registration system with role-based access control distinguishing pilgrim and admin roles.
- Develop a temple registry supporting multi-state South Indian temples with geolocation, deity information, gate navigation instructions, and image assets.
- Build a darshan slot booking engine with per-user ticket limits, real-time capacity enforcement using database-level transactions, and automated QR code generation for each booking.
- Design an AI crowd prediction module that forecasts crowd levels for any temple on any date using festival calendars, public holidays, weekend heuristics, and live booking utilization data.
- Implement a geolocation-based nearby temple discovery feature using the OpenStreetMap Overpass API with Haversine distance calculation.
- Create a safety module enabling pilgrims to trigger categorized SOS alerts with GPS coordinates, and providing admins with a live monitoring console.
- Provide a digital Dakshan (donation) subsystem that records temple-specific contributions and generates transaction histories.
- Offer an admin management console with aggregate statistics, slot capacity management, QR-based entry validation, finance reporting, and system announcement broadcasting.
- Integrate a rule-based pilgrim chatbot for in-platform guidance on bookings, crowd forecasts, donations, and emergency procedures.

---

## Summary

SmartPilgrim is organized into seven independent Django applications — `accounts`, `temples`, `bookings`, `crowd_ai`, `contributions`, `safety`, and `management` — each managing its own models, views, URL routes, and admin registrations, all wired together through the central `config` project.

Pilgrims register with their email address (username is disabled in favor of email-based authentication via a custom `AbstractUser` model), complete OTP verification delivered via Gmail SMTP, and are directed to a role-specific dashboard upon login. The pilgrim dashboard aggregates upcoming bookings, total dakshan given, unread notifications, and a live crowd forecast for the first registered temple. The admin dashboard aggregates system-wide ticket counts, gate scan statistics, financial totals, and a live feed of pending SOS alerts.

Temple records store geolocation coordinates used both for a proximity search (Haversine formula against Overpass API results) and for crowd prediction calibration. Each darshan slot covers one of three daily windows (morning, afternoon, night) with configurable capacity; booking is transactionally safe using `select_for_update` row locking. Confirmed bookings generate QR codes via the `qrcode` library, which are scanned at the temple gate by admins using a camera-based web interface backed by a JSON API endpoint.

Crowd prediction is deterministic and rule-based: it checks for admin overrides first, then festival/holiday matches, then weekend status, then live booking utilization thresholds, falling back to "LOW" crowd as default. PDF finance reports are generated with ReportLab; the Gemini AI SDK is included in dependencies for potential future integration.

---

## Features

### Before Login (Public Access)

- **Landing Page** — Overview of SmartPilgrim's capabilities including crowd prediction, slot booking, safety features, and donation system, with navigation to signup and login.
- **Email-Verified Signup** — Pilgrims and admins register using email and password. A 6-digit OTP valid for 10 minutes is dispatched via Gmail SMTP; accounts remain inactive until OTP is verified.
- **OTP Verification Flow** — Dedicated OTP entry page with expiry enforcement; unverified accounts are blocked from login with an informative redirect.
- **Password Reset** — Full Django password reset flow via email with tokenized reset links, confirmation, and success pages.
- **Temple Directory (Public)** — Browsable temple listing with search by name or district and state-based filtering.

### After Login — Pilgrim (Devotee) Portal

- **Personalized Dashboard** — Displays upcoming valid bookings ordered by date, cumulative dakshan total, unread system notifications, and a real-time crowd forecast for the nearest/first registered temple.
- **Temple Detail Page** — Full temple profile including deity information, entry/exit/darshan hall navigation guidance, available darshan slots with remaining capacity, and today's crowd prediction with reason.
- **Darshan Slot Booking** — Reserve a slot for a chosen temple and time window. Per-user ticket limit of 5 per slot is enforced; capacity is checked and decremented atomically. A QR-encoded booking ID is auto-generated and stored.
- **Booking Confirmation** — Displays booking details and downloadable QR code immediately after successful reservation.
- **Booking History** — Chronological list of all bookings with status (Valid, Used, Expired, Cancelled). Future-dated bookings can be cancelled with automatic slot capacity restoration.
- **Nearby Temple Discovery** — Browser geolocation API captures the pilgrim's coordinates; the backend queries the Overpass API for Hindu temples within a 30 km radius and returns results plotted on a Leaflet map with individual temple detail pages.
- **Digital Dakshan (Donation)** — Select any registered temple and submit a donation amount with an optional prayer message. A unique transaction ID is auto-generated. Contribution history shows per-donation details and a lifetime total.
- **SOS Emergency Alert** — Categorized emergency form (Medical, Crowd Crush, Harassment, Missing Person, Other) with automatic GPS coordinate capture. Submitted alerts are logged and immediately escalated to admin monitoring.
- **Notifications Center** — Lists all system push-style notifications (darshan reminders, crowd alerts, system updates) with read/unread state.
- **Emergency Log** — Personal log of all past SOS submissions with issue type, timestamp, and resolution status.
- **Profile Settings** — Update full name, phone number, and profile picture.
- **Pilgrim Chatbot** — Keyword-driven in-system chatbot (no external API key required) providing instant answers about bookings, crowd status, donations, and SOS procedures.

### After Login — Admin (Temple Authority) Portal

- **Admin Dashboard** — Real-time aggregates: total pilgrims served (ticket count), scanned entry passes, valid booking count, total dakshan received across all temples, count and live feed of pending SOS alerts.
- **Temple Management** — Create, edit, and manage temple registry entries including state, district, deity, geolocation coordinates, gate navigation text, and cover image.
- **Darshan Slot Management** — Create and modify slot configurations (date, type, max capacity) per temple. Admin can adjust capacity at any time.
- **QR Scanner** — Camera-powered web page using the device camera to read pilgrim QR codes. Each scan triggers an AJAX call to the validation API, which checks booking status, confirms the slot date matches today, marks it as `USED`, and returns an access granted/denied response in real time.
- **SOS Monitor** — Live view of all pending emergency alerts with pilgrim identity, issue category, GPS coordinates, and timestamp. Admins can resolve alerts, triggering an `ActivityLog` entry.
- **Finance Report** — Temple-wise and aggregate financial summaries of all dakshan contributions, exportable as a PDF via ReportLab.
- **System Announcements** — Broadcast system-wide messages to all pilgrim accounts with optional expiry dates.
- **Activity Log** — Auditable record of all admin actions including QR validations, SOS resolutions, and slot updates with actor identity and timestamp.

---

## Technologies Used

### Programming Languages
- Python 3.x
- JavaScript (ES6 — geolocation, camera API, AJAX calls)
- HTML5, CSS3

### Libraries & Frameworks
- Django 5.2 — core MVC web framework
- django-crispy-forms 2.5 + crispy-bootstrap5 — form rendering
- Bootstrap 5 — responsive UI layout and component library

### Machine Learning / AI
- Rule-based crowd prediction engine (`crowd_ai/logic.py`) — deterministic multi-factor model using festival/holiday data, booking utilization, and weekend patterns
- Google Gemini AI SDK (`google-genai`, `google-generativeai`) — included in dependencies for future AI enhancement
- Keyword-based pilgrim chatbot — in-system knowledge base, no external API dependency

### Backend
- Django ORM with SQLite (WAL-safe transactions via `select_for_update`)
- `qrcode[pillow]` — QR code image generation for darshan bookings
- `reportlab` — PDF generation for finance reports
- `requests` — Overpass API querying for nearby temple discovery
- Django SMTP Email Backend — OTP delivery and password reset via Gmail

### Frontend
- Django Template Engine
- Leaflet.js (via CDN) — interactive map for nearby temples
- HTML5 Geolocation API — pilgrim location capture
- HTML5 MediaDevices API — camera access for QR scanner
- Chart.js — admin dashboard statistics visualization

### Database
- SQLite 3 — default Django relational database with session-based authentication

### Deployment & DevOps
- Django development server (`manage.py runserver`)
- `python-dotenv` — environment variable management
- ASGI/WSGI compatible (`asgi.py`, `wsgi.py` provided)
- Static files via `STATICFILES_DIRS`; media files via `MEDIA_ROOT`

### Security
- `AbstractUser` with email as `USERNAME_FIELD` — username field disabled system-wide
- OTP-based account activation (6-digit, 10-minute expiry, SMTP delivery)
- `@login_required` decorator on all 25+ protected views
- Role-based redirect logic (`admin` vs `pilgrim`) enforced at every protected route entry point
- Django CSRF middleware active on all form submissions
- Database-level atomic transactions for booking and cancellation operations
- `select_for_update()` row locking preventing double-booking under concurrent load

---

## Dataset

SmartPilgrim does not use a traditional ML training dataset. The `crowd_ai` module relies on a structured rule-based decision system populated from the following Django database models:

**Festival (`crowd_ai.Festival`):** Admin-created records mapping festival names to calendar dates with an impact level of HIGH or MEDIUM. Any date matching a festival entry triggers a HIGH crowd prediction.

**PublicHoliday (`crowd_ai.PublicHoliday`):** National and regional public holiday calendar maintained by admins. Matching dates return a HIGH crowd forecast.

**CrowdOverride (`crowd_ai.CrowdOverride`):** Per-temple, per-date manual overrides allowing admins to set an explicit crowd level and custom reason message, unconditionally superseding all algorithmic predictions.

**DarshanSlot booking utilization:** Live booking data is aggregated per temple per date at query time. Utilization above 80% returns HIGH; above 40% returns MEDIUM.

A demo data management command (`accounts/management/commands/setup_demo_data.py`) seeds initial temples, slots, users, and bookings for development and demonstration purposes.

---

## System Architecture

```
┌──────────────────────────────────────────────────────────────────────┐
│                      Browser (Pilgrim / Admin)                       │
│  Landing │ Auth │ Pilgrim Dashboard │ Admin Dashboard │ Chatbot      │
└────────────────────────────┬─────────────────────────────────────────┘
                             │ HTTP (Session Cookie / CSRF Token)
                             ▼
┌──────────────────────────────────────────────────────────────────────┐
│                   Django Application Server                          │
│                        config/urls.py                                │
│                                                                      │
│  ┌──────────┐  ┌─────────┐  ┌──────────┐  ┌──────────────────────┐ │
│  │ accounts │  │ temples │  │ bookings │  │      crowd_ai        │ │
│  │ Auth/OTP │  │Registry │  │ Slots/QR │  │  Prediction Engine   │ │
│  │ Profile  │  │ Nearby  │  │ Booking  │  │  Chatbot KB          │ │
│  └──────────┘  └─────────┘  └──────────┘  └──────────────────────┘ │
│                                                                      │
│  ┌──────────────┐  ┌────────────┐  ┌───────────────────────────────┐│
│  │ contributions│  │   safety   │  │         management            ││
│  │ Dakshan/TXN  │  │ SOS/Notify │  │ Admin Dashboard / QR Scanner  ││
│  │ History      │  │ EmergencyLog│  │ Finance Report / Logs        ││
│  └──────────────┘  └────────────┘  └───────────────────────────────┘│
└────────────────────────────┬─────────────────────────────────────────┘
                             │
                             ▼
┌──────────────────────────────────────────────────────────────────────┐
│                        SQLite Database                               │
│  accounts_user │ accounts_emailotp │ temples_temple                 │
│  bookings_darshanslot │ bookings_booking                            │
│  crowd_ai_festival │ crowd_ai_publicholiday │ crowd_ai_crowdoverride│
│  contributions_contribution │ safety_sosalert │ safety_notification  │
│  management_activitylog │ management_systemannouncement             │
│  management_adminprofile                                            │
└──────────────────────────────────────────────────────────────────────┘
                             │
                    External Services
          ┌──────────────────┴──────────────────┐
          │ Gmail SMTP (OTP / Password Reset)   │
          │ Overpass API (Nearby Temple Search) │
          └─────────────────────────────────────┘
```

---

## Model Workflow

### Crowd Prediction Pipeline

The crowd prediction engine in `crowd_ai/logic.py` follows a strict priority-ordered decision chain for any `(temple, target_date)` pair:

**Step 1 — Admin Override Check:** Query `CrowdOverride` for the given temple and date. If found, immediately return its `override_level` and `reason`. Manual corrections take unconditional precedence over all algorithmic signals.

**Step 2 — Festival Detection:** Query `Festival` for the target date. A match returns `HIGH` with "Festival Day — Traditional High Volume."

**Step 3 — Public Holiday Detection:** Query `PublicHoliday` for the target date. A match returns `HIGH` with "Public Holiday — Expect Crowds."

**Step 4 — Weekend Heuristic:** Evaluate `target_date.weekday() >= 5`. Saturday and Sunday return `MEDIUM` with "Weekend Flow."

**Step 5 — Booking Utilization:** Aggregate `reserved_count` and `max_capacity` across all `DarshanSlot` records for the temple on the target date. Utilization above 80% returns `HIGH`; above 40% returns `MEDIUM`.

**Step 6 — Default:** Return `LOW` with "Optimal time for Darshan."

### Booking Reservation Pipeline

1. Pilgrim selects a darshan slot from the temple detail page.
2. System checks the pilgrim's current `VALID` ticket count for that slot via `Sum` aggregate; blocks submission if `>= 5`.
3. On `POST`, a `select_for_update()` lock is acquired on the slot row inside `transaction.atomic()`.
4. Slot availability is re-verified under the lock to prevent race conditions.
5. A `Booking` record is created with a UUID `booking_id`.
6. `slot.reserved_count` is incremented using a Django `F()` expression to avoid lost-update anomalies.
7. A PNG QR code encoding the `booking_id` is generated via the `qrcode` library, saved to `media/qrcodes/`, and linked to the booking record.

### QR Gate Validation Pipeline

1. Admin navigates to the QR Scanner page; the browser activates the device camera via `navigator.mediaDevices.getUserMedia`.
2. A client-side QR decoder reads the booking UUID from the camera stream.
3. An AJAX POST is dispatched to `/management/validate/<booking_uuid>/` with `X-Requested-With: XMLHttpRequest`.
4. The view verifies admin role, retrieves the booking, checks `slot.date == timezone.localdate()`, and transitions status from `VALID` to `USED` if all checks pass.
5. A JSON response is returned immediately: `{"success": true, "message": "Access Granted: ..."}` or the appropriate denial reason without a page reload.

---

## Implementation Details

**`accounts` app:** Custom `User` model extends `AbstractUser` with `username = None` and `EMAIL_FIELD` as the login credential, supported by `CustomUserManager`. The `EmailOTP` model implements a one-to-one OTP relationship with 10-minute expiry enforced in Python (`timezone.now() > created_at + timedelta(minutes=10)`). Signup sets `is_active=False`; accounts are activated only on valid OTP submission. Login increments `login_count` using `update_fields=['login_count']` for efficiency. Role-based routing dispatches `admin` users to `admin_dashboard` and `pilgrim` users to `user_dashboard`.

**`temples` app:** `Temple` model stores name, deity, state (South Indian state choices: AP, TS, TN, KA, KL), district, address, gate navigation text for three zones, a cover image, and decimal GPS coordinates. `temple_list` supports combined text search via Django `Q` objects on name and district, plus state dropdown filtering. `temple_detail` assembles the temple record, future `DarshanSlot` records (`date__gte=timezone.localdate()`), and a fresh crowd prediction. `nearby_temples` builds an Overpass QL query for Hindu `amenity=place_of_worship` nodes within 30 km of the browser-provided coordinates, returning name/lat/lon tuples rendered on a Leaflet map. `live_temple_detail` renders enriched OpenStreetMap attributes (wheelchair accessibility, Wikipedia link, street, city) for Overpass-sourced temples.

**`bookings` app:** `DarshanSlot` enforces a `unique_together` constraint on `(temple, date, slot_type)`. The `available_tickets` property computes remaining capacity without an extra query. `book_darshan` uses `select_for_update()` combined with `transaction.atomic()` for safe concurrent reservation. Cancellations are restricted to future-dated slots; `reserved_count` is safely decremented using `F('reserved_count') - 1`. The `slot_availability_api` JSON endpoint provides real-time capacity data for potential frontend polling.

**`crowd_ai` app:** `predict_crowd` is a pure ORM function with no ML library dependencies, making it fast and deterministic. It is invoked from both `temples/views.py` and `accounts/views.py` to ensure consistent crowd status across the platform. The chatbot exposes a CSRF-exempt JSON POST endpoint (`/crowd-ai/api/`) that pattern-matches lowercased user messages against a Python dictionary knowledge base covering six topic keywords.

**`contributions` app:** `Contribution` records are linked to both `User` and `Temple` with a `transaction_id` generated from `uuid.uuid4().hex[:10].upper()` prefixed by `"TXN-"`, providing human-readable unique identifiers. Contribution history computes a lifetime total using `aggregate(Sum('amount'))` returning a safe default of `0.00` when no records exist.

**`safety` app:** `SOSAlert` captures five issue categories with optional decimal-precision GPS coordinates and a freeform message field. `Notification` supports three categories with an `is_read` boolean for notification center display. The SOS trigger immediately flashes an emergency confirmation message and redirects to the dashboard.

**`management` app:** The admin dashboard assembles six aggregates in a single view using `Count` and `Sum` across four models. `validate_booking` handles both AJAX (JSON response) and standard HTTP requests by inspecting the `X-Requested-With` header, enabling seamless QR scanner operation. `ActivityLog` records every admin action with actor identity, type, and timestamp for a full audit trail. Finance reports use ReportLab for structured PDF output. `AdminProfile` allows optional temple assignment to scope admin authority to a specific site.

---

## Results

SmartPilgrim delivers a functionally complete digital platform covering the full pilgrimage lifecycle:

**Authentication and Access Control:** Email-verified registration with 10-minute OTP expiry, role-segregated dashboards, and `@login_required` enforcement across all protected views. The custom `AbstractUser` implementation eliminates username-based authentication entirely in favor of email.

**Booking System Integrity:** Atomic slot reservation with row-level locking (`select_for_update`) prevents double-booking under concurrent load. Per-user ticket cap of 5 per slot is enforced at both the application and query aggregation level. QR code generation on confirmation and camera-based gate validation via AJAX provide a complete ticketing lifecycle.

**Crowd Prediction Reliability:** The rule-based engine returns deterministic results for all festival and public holiday dates with 100% recall against the admin-maintained calendar. Live booking utilization provides a dynamic fallback for non-calendar events, and admin overrides ensure operational flexibility for exceptional circumstances.

**Geolocation Accuracy:** The Haversine formula correctly computes great-circle distances between the pilgrim's GPS position and temple coordinates in the Overpass API response, delivering accurate proximity sorting for the nearby temple discovery feature.

**Safety Response Coverage:** SOS alerts capture GPS coordinates, issue category, and custom messages with immediate admin-side visibility. The five-category issue taxonomy (Medical, Crowd, Harassment, Lost, Other) covers the primary emergency scenarios encountered at large-scale pilgrimage sites.

**Financial Transparency:** Per-pilgrim contribution history with lifetime totals and admin-level aggregate finance reporting with PDF export supports both devotee accountability and temple financial governance.

---

## Installation

### Prerequisites

- Python 3.10 or higher
- pip
- A Gmail account with an App Password configured for SMTP (required for OTP delivery)

### Setup Instructions

```bash
# Clone the repository
git clone https://github.com/harisaigithub/piligrim-website.git
cd piligrim-website

# Create and activate a virtual environment
python -m venv venv
source venv/bin/activate        # On Windows: venv\Scripts\activate

# Install all dependencies
pip install -r req.txt

# Apply database migrations
python manage.py migrate

# (Optional) Load demo data for development
python manage.py setup_demo_data

# Create a superuser for the Django admin panel
python manage.py createsuperuser

# Start the development server
python manage.py runserver
```

The application will be available at `http://127.0.0.1:8000`.

### Gmail SMTP Configuration

Update the following fields in `config/settings.py` with your Gmail credentials, or load them from a `.env` file using `python-dotenv`:

```python
EMAIL_HOST_USER = 'your_email@gmail.com'
EMAIL_HOST_PASSWORD = 'your_app_password'   # Gmail App Password (not account password)
DEFAULT_FROM_EMAIL = 'SmartPilgrim Support <your_email@gmail.com>'
```

To generate a Gmail App Password: Google Account → Security → 2-Step Verification → App Passwords.

---

## Usage

### Pilgrim Registration and Login

1. Visit `http://127.0.0.1:8000` to access the landing page.
2. Click **Sign Up**, enter your email, password, and select role **Pilgrim/Devotee**.
3. Check your email for the 6-digit OTP and enter it on the verification page.
4. Log in with your credentials to access the Pilgrim Dashboard.

### Booking a Darshan Slot

1. Navigate to **Temples** and select a temple.
2. On the temple detail page, review available darshan slots with remaining capacity and the current crowd forecast.
3. Click **Book Slot** on an available slot.
4. Confirm the booking. A QR code is generated and displayed on the confirmation page.

### Discovering Nearby Temples

1. Navigate to **Nearby Temples**.
2. Allow browser location access when prompted.
3. A Leaflet map displays Hindu temples within 30 km of your location, sourced from OpenStreetMap via Overpass API.
4. Click any marker to view the temple's detail page.

### Making a Donation

1. Navigate to **Dakshan** (Contributions).
2. Select the target temple, enter an amount, and optionally add a prayer message.
3. Submit. A unique transaction ID is generated and the contribution is recorded instantly.

### Triggering an SOS Alert

1. Navigate to **Safety → SOS**.
2. Select the emergency category and allow location access.
3. Optionally add a descriptive message and submit. The alert is immediately visible in the admin SOS Monitor.

### Admin Operations

1. Log in with an account having role **Admin/Temple Authority**.
2. Access the **Admin Dashboard** for real-time metrics across all temples.
3. Navigate to **QR Scanner** and allow camera access to validate pilgrim entry passes at the gate.
4. Manage **Temples**, **Slots**, **Announcements**, and **Finance Reports** from the admin navigation.

---

## Future Enhancements

- **Gemini AI Integration** — The `google-generativeai` SDK is already included in `req.txt`. Future versions will upgrade the keyword chatbot to a fully conversational Gemini-powered assistant capable of nuanced pilgrimage guidance.
- **Real-time WebSocket Notifications** — Replace polling-based notification delivery with Django Channels WebSocket connections for instant crowd alerts and booking reminders.
- **SMS OTP via Twilio** — Supplement or replace Gmail SMTP with an SMS gateway for OTP delivery, improving reliability for pilgrims without consistent email access.
- **Progressive Web App** — Convert the Django templates into a PWA with offline capability, home screen installation, and push notification support using service workers.
- **Multi-Temple Admin Scoping** — Activate the `AdminProfile.assigned_temple` field to restrict dashboard visibility and slot management to a single assigned temple per admin account.
- **Crowd Heatmap Visualization** — Render a historical crowd density heatmap on the temple detail page by hour and day of week, derived from aggregated booking and QR scan timestamps.
- **UPI Payment Gateway Integration** — Replace the mock donation system with a real UPI or Razorpay gateway to process verified Dakshan contributions with digital receipts.
- **Multi-Language Support** — Add Telugu, Tamil, Kannada, and Malayalam translations using Django's i18n framework for pilgrims who prefer regional languages.
- **Wheelchair Accessibility Route Planner** — Extend the `Temple` model with accessible route instructions for differently-abled pilgrims, surfaced on temple detail pages.
- **Docker Containerization** — Package the Django application in Docker with a `docker-compose.yml` for reproducible deployment across development, staging, and production environments.

---

## Contributing

Contributions to SmartPilgrim are welcome. Please follow the standard Git workflow:

```bash
# Fork the repository and clone your fork
git clone https://github.com/<your-username>/piligrim-website.git
cd piligrim-website

# Create a feature branch
git checkout -b feature/your-feature-name

# Make your changes, run tests, and commit
python manage.py test
git add .
git commit -m "feat: describe your change clearly"

# Push and open a Pull Request
git push origin feature/your-feature-name
```

**Contribution Guidelines:**
- All new views must be protected with `@login_required` and include role-based access checks where applicable.
- New Django apps must be registered in `INSTALLED_APPS` in `config/settings.py` and their URL patterns included in `config/urls.py`.
- Database schema changes must be accompanied by a migration file generated via `python manage.py makemigrations`.
- Follow existing conventions: singular model names, snake_case fields, and explicit `__str__` methods on all models.
- Commit messages should follow the [Conventional Commits](https://www.conventionalcommits.org/) specification.

---

## License

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

For collaborations, customizations, or deployment support, feel free to reach out.