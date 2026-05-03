# Automated Student Attendance Monitoring and Analytics System

> **A Real-Time Face Recognition–Powered Academic Attendance Platform with Role-Based Dashboards, AI Analytics, and PDF Reporting**

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

Manual attendance marking in academic institutions is time-consuming, error-prone, and susceptible to proxy attendance — a persistent challenge in colleges and universities. Traditional roll-call systems waste significant instructional time and produce unreliable data that makes meaningful attendance analytics nearly impossible.

The **Automated Student Attendance Monitoring and Analytics System** is a full-stack Django 5 web application that eliminates manual attendance entirely by leveraging **OpenCV's LBPH (Local Binary Patterns Histogram) Face Recognizer** and **Haar Cascade frontal face detection** to identify and record student attendance in real time through a live camera stream. The system is designed around three distinct user roles — Administrator, Faculty, and Student — each with purpose-built dashboards, access controls, and analytics views.

Beyond recognition, the platform integrates **Google Gemini AI** for intelligent report generation, **ReportLab** for downloadable PDF attendance reports, an in-app notification engine, a 75% attendance shortage watchlist for faculty, and an administrative summary dashboard aggregating department-level attendance averages. The system is production-ready for deployment in B.Tech and other undergraduate academic environments and is time-zoned for the Indian Standard Time (IST) zone.

---

## Objectives

- Automate student attendance marking entirely through real-time facial recognition, eliminating manual roll-calls and proxy attendance.
- Implement a three-tier role-based access system (Admin, Faculty, Student) with email OTP verification for secure account registration and activation.
- Train and deploy an LBPH Face Recognizer model using student-uploaded face images, with a confidence threshold of 45 for accurate identity matching.
- Provide faculty members with a live MJPEG camera stream that detects faces, identifies registered students, marks attendance in the database, and triggers in-app notifications — all in real time.
- Deliver per-student subject-wise attendance statistics and downloadable PDF reports styled with academic formatting.
- Automatically identify and surface students with attendance below the 75% threshold (shortage cases) for faculty-level intervention.
- Generate department-level attendance averages and platform-wide statistics for administrative decision-making.
- Support full academic management CRUD operations — departments, subjects, sections, faculty, students, and timetable slots — from a single admin interface.

---

## Summary

The Automated Student Attendance Monitoring and Analytics System is a modular Django 5 web application organized into four primary Django apps: `accounts`, `attendance`, `analytics`, and `management`.

The **accounts** app provides the custom `AbstractUser`-based user model keyed on email (not username), role-based routing on login, OTP-based email verification with a 10-minute expiry, and three role-specific dashboards. The **attendance** app houses the `VideoCamera` class that wraps OpenCV's camera capture, Haar Cascade face detection, and LBPH face recognition into a MJPEG streaming view. A separate `recognition_utils.py` module handles the training pipeline — iterating student face image folders, extracting face regions, and fitting the LBPH model to produce a `trainer.yml` weights file. The **analytics** app computes subject-wise attendance percentages, identifies shortage-case students, generates downloadable PDF reports via ReportLab, and maintains an in-app notification feed with an unread count surfaced globally via a custom context processor. The **management** app provides administrators with full CRUD control over departments, subjects, sections, faculty members, students, and timetable mappings.

All components interact through a shared SQLite database, with media files (face images, PDF reports, profile pictures) managed through Django's media file handling infrastructure.

---

## Features

### Public / Pre-Login

- Professional academic landing page with navigation to login and signup.
- Unified login page supporting all three user roles (Admin, Faculty, Student) with role-based post-login redirection.
- OTP-based email verification: a 6-digit OTP is generated on signup, dispatched via Gmail SMTP, and must be verified (within 10 minutes) before the account is activated.
- Password reset flow with email-based token links (Django's built-in `PasswordResetView` with custom templates).

### Administrator

- **Admin Dashboard**: Platform-wide statistics — total students, total faculty, total departments, overall average attendance percentage, and the 5 most recent attendance log entries.
- **Department Management**: Create, list, and delete academic departments with unique name and code (e.g., CSE, ECE, MECH).
- **Faculty Management**: Add faculty members with direct account creation (bypassing OTP flow), view the full faculty list, edit faculty details, and delete faculty accounts.
- **Student Management**: Add students with roll number as `user_id_code`, view all registered students, and delete student accounts.
- **Timetable Management**: Create timetable slots binding a faculty, subject, section, weekday, and start/end times; view all timetable entries ordered by day and time.
- **Face Recognition Model Training**: Trigger the LBPH training pipeline from the admin dashboard; the system iterates all student face image folders, extracts face samples, trains the recognizer, and saves the `trainer.yml` weights file.
- **Admin Attendance Summary**: Department-level attendance averages computed dynamically from the `AttendanceRecord` table; platform-wide average displayed as a headline metric.

### Faculty

- **Faculty Dashboard**: Assigned timetable slots, recent session summaries (present count per date and subject), and today's date indicator.
- **Live Attendance Scanner**: Select a subject and section from the faculty's assigned timetable, then launch the real-time MJPEG camera stream. Detected faces are matched against the trained LBPH model; recognized students are automatically marked present in the database for the current date, and a notification is dispatched to the student.
- **Faculty Analytics**: Ten most recent attendance session trends (attendance percentage per date), watchlist count (students below 75%), and watchlist percentage of the total student body.
- **Shortage Cases**: List of all students with attendance below the 75% threshold for the faculty's assigned subjects, including precise present count, total count, and calculated percentage.
- **Session Detail View**: Per-date drill-down showing all attendance records for the faculty's subjects on a given day, including student identity and subject.

### Student

- **Student Dashboard**: Overall attendance percentage, total present count, total attendance records, recent 5 activity logs, and face data registration status indicator.
- **Face Data Upload**: Upload a minimum of 5 face images for LBPH training; images are saved to `media/face_dataset/<roll_number>/`; the user's `has_face_data` flag is set to `True` on completion.
- **Subject-wise Attendance Report**: Per-subject breakdown showing attended classes, total classes held, and computed attendance percentage for the logged-in student.
- **PDF Report Download**: ReportLab-generated academic attendance report with header, student name and roll number, colour-coded status column (Teal for Present, Red for Absent), and pagination for up to 25 records per page. File named `Attendance_Report_<roll_number>.pdf`.
- **In-App Notifications**: Real-time notification feed showing all system-generated alerts (attendance marked, etc.) with read/unread status; unread count displayed in the navigation bar via a global context processor.

---

## Technologies Used

### Programming Languages
- Python 3.x
- HTML5, CSS3, JavaScript

### Libraries & Frameworks
- **Django 5.2** — web framework, ORM, authentication, session management, CSRF protection
- **django-crispy-forms 2.5** with **crispy-bootstrap5** — form rendering with Bootstrap 5 styling
- **python-dotenv 1.2** — environment variable management

### Machine Learning / AI
- **OpenCV (cv2)** — `CascadeClassifier` with `haarcascade_frontalface_default.xml` for frontal face detection; `cv2.face.LBPHFaceRecognizer_create()` for face recognition
- **NumPy** — array operations for face sample preparation during LBPH training
- **Pillow 12.1** — image loading and grayscale conversion (`Image.open().convert("L")`) in the training pipeline
- **Google Generative AI (`google-genai` 1.61, `google-generativeai` 0.8)** — Gemini AI SDK integrated for intelligent analytics and report generation

### Backend
- **Django 5.2** — project core (`core` project package with `accounts`, `attendance`, `analytics`, `management` apps)
- **Django AbstractUser** — custom `User` model with email as `USERNAME_FIELD`, role field (admin/faculty/student), `user_id_code`, `has_face_data`, `is_verified`, and `login_count`
- **Django StreamingHttpResponse** — MJPEG multipart streaming for the live camera feed
- **Django Email Backend** — Gmail SMTP via `django.core.mail` for OTP dispatch and password reset emails
- **ReportLab 4.4** — PDF generation with `canvas.Canvas`, custom fonts, colour-coded status cells, and pagination
- **qrcode 8.0** — QR code generation support

### Frontend
- Django Template Engine with template inheritance (`base1.html`, `base2.html`)
- Bootstrap 5 (via crispy-bootstrap5)
- Frozen Light academic theme — Deep Academic Blue (`#1E3A8A`), Soft Teal Accent (`#14B8A6`), Light Gray-White background (`#F8FAFC`)

### Database
- **SQLite 3** — development database (`db.sqlite3`)
- **Django ORM** — all database interactions; tables: `accounts_user`, `accounts_emailotp`, `management_department`, `management_subject`, `management_section`, `management_timetable`, `attendance_attendancerecord`, `attendance_studentfacedata`, `analytics_notification`, `analytics_attendancereport`

### Deployment & DevOps
- WSGI-compatible application server ready (`core/wsgi.py`)
- ASGI support included (`core/asgi.py`)
- Django management command `setup_demo_data` for populating academic infrastructure
- Static files via `STATICFILES_DIRS`; media files via `MEDIA_ROOT`

### Security
- Custom `AbstractUser` with email-based authentication (no username field)
- OTP-based two-factor account activation with 10-minute expiry (`EmailOTP.is_expired()`)
- Role-based access control enforced at every view with guard-clause redirects
- Django's full password validation suite (similarity, minimum length, common, numeric)
- CSRF middleware on all POST routes
- Clickjacking protection via `XFrameOptionsMiddleware`
- `is_active=False` account creation before OTP verification prevents premature access

---

## Dataset

The system does not use a pre-collected external face dataset. All training data is contributed organically by students through the face upload module.

**Face Data Structure:**

```
media/
└── face_dataset/
    └── <student_roll_number>/
        ├── user_<id>_0.jpg
        ├── user_<id>_1.jpg
        ├── user_<id>_2.jpg
        ├── user_<id>_3.jpg
        └── user_<id>_4.jpg   (minimum 5 images required)
```

Each image is stored using the student's `user_id_code` (roll number) as the directory name, enabling the training script to resolve the corresponding database record via `User.objects.get(user_id_code=student_folder)`.

**Training Data Preparation:**
- Each uploaded image is opened with Pillow and converted to grayscale (`"L"` mode).
- Converted to a NumPy `uint8` array for OpenCV compatibility.
- Haar Cascade face detection (`detectMultiScale`) extracts the face region of interest (ROI) from each image.
- The cropped face ROI and the student's database integer `id` are appended to `face_samples` and `ids` lists respectively.
- The LBPH recognizer is trained on the collected `(face_samples, ids)` pairs and serialized to `opencv_files/trainer.yml`.

**Confidence Threshold:** During live recognition, a confidence score below **45** (lower is better in LBPH) is required to accept a match and mark attendance. Scores ≥ 45 result in an "Unknown" label with no attendance action.

---

## System Architecture

```
Automated-Student-Attendance-Monitoring-and-Analytics-System/
├── core/                          ← Django project root
│   ├── settings.py                ← Configuration (DB, SMTP, media, auth, theme)
│   ├── urls.py                    ← Root URL dispatcher
│   ├── wsgi.py / asgi.py          ← WSGI/ASGI entry points
│
├── accounts/                      ← Identity, roles, authentication
│   ├── models.py                  ← Custom User (email-based) + EmailOTP
│   ├── views.py                   ← Login, signup, OTP verify, 3x dashboards
│   ├── urls.py                    ← Auth + dashboard + password reset routes
│   └── management/commands/       ← setup_demo_data management command
│
├── attendance/                    ← Face recognition & attendance marking
│   ├── models.py                  ← AttendanceRecord, StudentFaceData
│   ├── views.py                   ← VideoCamera class, streaming view,
│   │                                 upload_faces_view, train_model_view
│   ├── recognition_utils.py       ← train_student_faces() LBPH pipeline
│   └── urls.py                    ← upload-faces, launch-scanner, video-feed, train-model
│
├── analytics/                     ← Reports, notifications, shortage detection
│   ├── models.py                  ← Notification, AttendanceReport
│   ├── views.py                   ← student_report, faculty_analytics,
│   │                                 shortage_cases, admin_summary,
│   │                                 download_attendance_pdf, notifications_list
│   ├── utils.py                   ← send_system_notification() helper
│   ├── context_processors.py      ← Global unread notification count injector
│   └── urls.py                    ← Report, notification, shortage, PDF routes
│
├── management/                    ← Academic entity CRUD
│   ├── models.py                  ← Department, Subject, Section, Timetable
│   ├── views.py                   ← Full CRUD for dept, faculty, student, timetable
│   └── urls.py                    ← Management routes
│
├── opencv_files/
│   └── trainer.yml                ← Serialized LBPH face recognition model weights
│
├── templates/
│   ├── base1.html / base2.html    ← Base layout templates
│   ├── landing.html               ← Public landing page
│   ├── accounts/                  ← Login, signup, OTP, dashboards
│   ├── attendance/                ← Upload faces, attendance form, camera stream
│   ├── analytics/                 ← Reports, notifications, shortage, admin summary
│   ├── management/                ← CRUD templates for all entities
│   └── registration/              ← Password reset email templates
│
├── media/                         ← Runtime-generated uploads
│   ├── face_dataset/              ← Student face images (per roll number folder)
│   ├── profiles/                  ← Profile pictures
│   └── reports/                   ← Generated PDF attendance reports
│
├── req.txt                        ← Python dependency manifest
└── db.sqlite3                     ← SQLite development database
```

**Request Flow (Attendance Marking):**
Faculty selects subject + section → POST to `take_attendance_view` → redirect to `video_feed` → `VideoCamera` initializes OpenCV capture and loads `trainer.yml` → MJPEG frames streamed to browser → per frame: Haar Cascade detects faces → LBPH `predict()` returns `(id, confidence)` → confidence < 45: `User.objects.get(id=id_num)` → `AttendanceRecord.get_or_create()` with `is_present=True` → `send_system_notification()` → label drawn on frame → encoded as JPEG → streamed as multipart response.

---

## Model Workflow

### Step 1 — Student Face Data Upload

1. Student logs in and navigates to the face upload page.
2. A minimum of 5 images are required; fewer triggers a validation error.
3. Images are saved to `media/face_dataset/<roll_number>/` with filenames `user_<db_id>_<index><ext>`.
4. A `StudentFaceData` record is created per image; the user's `has_face_data` field is set to `True`.

### Step 2 — LBPH Model Training (Admin Triggered)

1. The admin triggers training from the dashboard; `train_student_faces()` is called from `recognition_utils.py`.
2. The function iterates all subdirectories in `media/face_dataset/`.
3. For each folder named by roll number, the corresponding `User` is resolved via `User.objects.get(user_id_code=student_folder)`.
4. Each image is loaded with Pillow, converted to grayscale, converted to a NumPy `uint8` array.
5. `cv2.CascadeClassifier.detectMultiScale()` runs on the array to locate face regions.
6. Detected face ROIs are cropped from the array and appended to `face_samples`; the student's database integer `id` is appended to `ids`.
7. `cv2.face.LBPHFaceRecognizer_create().train(face_samples, np.array(ids))` fits the model.
8. The trained model is serialized via `recognizer.write("opencv_files/trainer.yml")`.
9. Returns `True` on success; `False` if no face samples were found.

### Step 3 — Real-Time Recognition and Attendance Marking

1. The `VideoCamera` class loads `trainer.yml` using `recognizer.read()` on instantiation.
2. `cv2.VideoCapture(0)` opens the default webcam.
3. Per frame: `cv2.cvtColor(image, cv2.COLOR_BGR2GRAY)` converts to grayscale.
4. `face_cascade.detectMultiScale(gray, 1.3, 5)` detects face bounding boxes.
5. For each detected face: `recognizer.predict(gray_face_roi)` returns `(id_num, confidence)`.
6. If `confidence < 45`: the student is fetched from the database; `AttendanceRecord.objects.get_or_create()` creates the record (if not already marked today); the student ID is added to `marked_students` to prevent duplicate marking; `send_system_notification()` dispatches an in-app notification.
7. A rectangle and label are drawn on the frame using `cv2.rectangle` and `cv2.putText`.
8. The frame is JPEG-encoded and yielded as a multipart MJPEG stream.

### Step 4 — Analytics and Reporting

1. Subject-wise statistics are computed dynamically: `total_classes = AttendanceRecord.objects.filter(subject=sub).count()` and `attended = logs.filter(subject=sub, is_present=True).count()`.
2. Shortage detection iterates all students; those with `(present / total) * 100 < 75` are flagged.
3. PDF generation uses ReportLab's `canvas.Canvas` with custom font settings, colour-coded status strings, and automatic pagination at `y < 50`.

---

## Implementation Details

**Custom User Model:** The `User` model extends `AbstractUser` with `username = None` and `USERNAME_FIELD = 'email'`, backed by `CustomUserManager` that overrides `create_user` and `create_superuser`. Additional fields include `role` (admin/faculty/student), `user_id_code` (roll number or faculty ID), `has_face_data` (recognition readiness flag), `is_verified` (OTP completion), and `login_count` (incremented on each successful login).

**OTP Registration Flow:** New accounts are created with `is_active=False`. A 6-digit random OTP is generated and stored in the `EmailOTP` model (one-to-one with `User`) with a `created_at` timestamp. `is_expired()` returns `True` if `timezone.now() > created_at + timedelta(minutes=10)`. On correct OTP entry, `user.is_active = True`, `user.is_verified = True`, and the `EmailOTP` record is deleted. The user's database ID is stored in the Django session (`request.session['verify_user']`) to link the verification page to the pending account.

**MJPEG Live Streaming:** The `video_feed` view returns a `StreamingHttpResponse` with `content_type='multipart/x-mixed-replace; boundary=frame'`. The generator function `gen(camera)` yields frames prefixed with the multipart boundary and `Content-Type: image/jpeg` header. This standard MJPEG protocol is natively rendered by all modern browsers without a plugin.

**Duplicate Attendance Prevention:** Two layers protect against double-marking. The `AttendanceRecord` model enforces a database-level `unique_together = ('student', 'subject', 'date')` constraint. Additionally, the `VideoCamera` class maintains a `marked_students` in-memory set per session; once a student ID is added, subsequent frames will not trigger another `get_or_create` call.

**Global Notification Counter:** The `analytics/context_processors.py` file defines `notification_context(request)`, registered in `settings.py` under `TEMPLATES[0]['OPTIONS']['context_processors']`. This injects `unread_notifications_count` into every template context, enabling the navigation bar unread badge to update on every page without a dedicated API call.

**Faculty Scope Isolation:** All faculty-facing queries (attendance scanner subjects/sections, analytics, shortage cases, session details) are scoped to the faculty's assigned timetable entries via `Timetable.objects.filter(faculty=request.user).values_list('subject', flat=True)`. This ensures no cross-faculty data leakage.

**PDF Academic Styling:** ReportLab's `canvas.Canvas` renders the report on US Letter paper size. The header uses Deep Academic Blue (`#1E3A8A`) via `p.setFillColor(colors.HexColor(...))`. Status cells use Soft Teal (`#14B8A6`) for Present and red for Absent. The report is returned as an in-memory `BytesIO` buffer via `FileResponse` with `Content-Disposition: attachment`.

---

## Results

### Face Recognition System Performance

| Parameter | Value |
|---|---|
| Algorithm | Local Binary Patterns Histogram (LBPH) Face Recognizer |
| Face Detector | Haar Cascade (`haarcascade_frontalface_default.xml`) |
| Confidence Threshold | < 45 (lower = better match) |
| Minimum Training Images | 5 per student |
| Training Input | Grayscale face ROIs extracted by Haar Cascade |
| Model File | `opencv_files/trainer.yml` (YAML serialized LBPH weights) |
| Camera Source | `cv2.VideoCapture(0)` — default system webcam |
| Stream Format | MJPEG multipart (`multipart/x-mixed-replace`) |

**Attendance Logic Outcomes:**

| Condition | Result |
|---|---|
| Confidence < 45 and student found | Attendance marked Present; in-app notification sent |
| Confidence ≥ 45 or prediction fails | Labelled "Unknown"; no database action |
| Student already in `marked_students` set | Skipped — duplicate prevention |
| `unique_together` constraint violation | `get_or_create` returns existing record; no duplicate |

**Analytics Thresholds:**

| Metric | Threshold | Action |
|---|---|---|
| Student Attendance % | < 75% | Added to shortage watchlist |
| Department Average | Computed dynamically | Displayed in admin summary |
| Overall Platform Average | Computed dynamically | Displayed in admin dashboard |

---

## Installation

### Prerequisites

- Python 3.10 or higher
- pip package manager
- Git
- A webcam connected to the system (for attendance scanning)
- A Gmail account with App Password enabled

### Step 1: Clone the Repository

```bash
git clone https://github.com/harisaigithub/Automated-Student-Attendance-Monitoring-and-Analytics-System.git
cd Automated-Student-Attendance-Monitoring-and-Analytics-System
```

### Step 2: Create and Activate a Virtual Environment

```bash
python -m venv venv

# Windows
venv\Scripts\activate

# macOS / Linux
source venv/bin/activate
```

### Step 3: Install Dependencies

```bash
pip install -r req.txt
```

> **Note:** OpenCV with the `face` module is required. Install it with:
> ```bash
> pip install opencv-contrib-python
> ```

### Step 4: Configure Environment Variables

Create a `.env` file in the project root or update `core/settings.py` directly:

```python
EMAIL_HOST_USER = 'your-email@gmail.com'
EMAIL_HOST_PASSWORD = 'your-gmail-app-password'
DEFAULT_FROM_EMAIL = 'Attendance System <your-email@gmail.com>'
```

For Google Gemini AI features, set your API key:

```python
GEMINI_API_KEY = 'your-gemini-api-key'
```

### Step 5: Apply Migrations

```bash
python manage.py makemigrations
python manage.py migrate
```

### Step 6: Create a Superuser (Admin)

```bash
python manage.py createsuperuser
# Provide email and password when prompted; role defaults to 'admin'
```

### Step 7: Collect Static Files (Production)

```bash
python manage.py collectstatic
```

### Step 8: Run the Development Server

```bash
python manage.py runserver
```

The application is accessible at `http://127.0.0.1:8000/`.

---

## Usage

### Administrator Workflow

1. Log in at `/accounts/login/` with the superuser credentials.
2. Navigate to **Departments** (`/management/departments/`) → add departments (e.g., CSE, ECE, MECH).
3. Navigate to **Faculty** (`/management/faculty/`) → add faculty members with their faculty IDs and email addresses.
4. Navigate to **Students** (`/management/students/`) → add students with roll numbers and email addresses.
5. Navigate to **Timetable** (`/management/timetable/`) → create slots assigning faculty, subjects, sections, weekdays, and times.
6. After students upload face data, trigger model training from the Admin Dashboard → **Train AI Model**.
7. Monitor platform-wide statistics via the Admin Dashboard and the Analytics Summary at `/analytics/admin-summary/`.

### Faculty Workflow

1. Log in with faculty credentials.
2. The Faculty Dashboard displays assigned timetable slots and recent session summaries.
3. Navigate to **Launch Scanner** (`/attendance/launch-scanner/`), select the subject and section, then click start.
4. The camera stream opens; recognized students are automatically marked present.
5. View attendance trends and watchlist at `/analytics/faculty-trends/`.
6. View shortage-case students at `/analytics/shortage-cases/`.
7. Click any session date to drill into `/analytics/session/<date>/` for a full session record.

### Student Workflow

1. Register at `/accounts/signup/` with roll number, email, and password; verify the OTP sent to your email.
2. Log in and navigate to **Upload Face Data** (`/attendance/upload-faces/`) — upload at least 5 clear face photos.
3. The Student Dashboard shows overall attendance percentage and the 5 most recent records.
4. View subject-wise attendance breakdown at `/analytics/student-report/`.
5. Download a PDF attendance report at `/analytics/download-pdf/`.
6. Check notifications at `/analytics/notifications/` for attendance confirmation alerts.

---

## Future Enhancements

- **Anti-Spoofing Layer:** Integrate a liveness detection module to prevent attendance fraud using printed photos or screen-displayed images, using blink detection or 3D depth analysis.
- **PostgreSQL Migration:** Replace the SQLite development database with PostgreSQL for multi-user concurrency, connection pooling, and production-grade reliability.
- **Gemini AI Integration Expansion:** Fully leverage the included Google Gemini AI SDK to generate natural-language attendance insights, personalized recommendations for at-risk students, and automated progress summaries.
- **QR Code Backup Attendance:** Utilize the included `qrcode` library to generate time-bound, session-specific QR codes as a fallback attendance mechanism when camera hardware is unavailable.
- **WebSocket-Based Live Dashboard:** Replace polling with Django Channels (WebSocket) to push real-time attendance counts to the faculty dashboard as students are recognized.
- **Mobile-Responsive PWA:** Package the frontend as a Progressive Web App with service worker caching, enabling offline access to student reports and notification history.
- **Multi-Camera Support:** Extend `VideoCamera` to support multiple simultaneous camera streams, enabling large lecture halls with multiple camera installations.
- **Automated Shortage Email Alerts:** Schedule a Celery periodic task that emails faculty and affected students when a student's attendance drops below the 75% threshold.
- **Biometric Fallback:** Integrate a manual biometric (fingerprint) module as an alternative identification method for students whose face data yields low recognition accuracy.
- **CSV and Excel Export:** Add CSV and Excel export options for attendance records alongside the existing PDF functionality, using `openpyxl` or `pandas`.

---

## Contributing

Contributions are welcome from developers and researchers interested in computer vision, academic ERP systems, or Django development.

1. **Fork** the repository on GitHub.
2. **Clone** your fork locally:
   ```bash
   git clone https://github.com/<your-username>/Automated-Student-Attendance-Monitoring-and-Analytics-System.git
   ```
3. **Create a feature branch**:
   ```bash
   git checkout -b feature/your-feature-name
   ```
4. **Make your changes** — follow the existing module structure; add docstrings to any new OpenCV or analytics functions.
5. **Commit with a clear message**:
   ```bash
   git commit -m "feat: add liveness detection for anti-spoofing"
   ```
6. **Push** your branch:
   ```bash
   git push origin feature/your-feature-name
   ```
7. **Open a Pull Request** against the `main` branch with a detailed description.

Never commit email credentials, Gmail App Passwords, or Gemini API keys — use `.env` files and `python-dotenv` for all sensitive configuration.

---

## License

```
MIT License

Copyright (c) 2026 Hari Sai Parasa

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

For support, academic inquiries, or collaboration opportunities related to this project, please reach out through the channels below.

---

&nbsp;

&nbsp;

## ***To acquire this project, please contact***

**GitHub:** https://github.com/harisaigithub  
**Email:** [harisaiparasa@gmail.com](mailto:harisaiparasa@gmail.com)  

For collaborations, customizations, or deployment support, feel free to reach out.