# SmartAttend — Rural School Attendance Management System

> An AI-powered, full-stack attendance management platform for rural schools featuring real-time facial recognition, QR code scanning, three-role dashboards, activity management, and automated low-attendance alerting — built with Flask and React.

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

Rural schools across India frequently operate with limited administrative staff, unreliable internet connectivity, and a lack of digital infrastructure for tracking student attendance. Manual registers are prone to errors, fabrication, and loss; they provide no real-time visibility into attendance trends and offer no mechanism for alerting parents or administrators when students miss consecutive days or fall below attendance thresholds.

SmartAttend addresses these challenges with a lightweight yet comprehensive attendance management platform designed specifically for the rural school context. The system introduces two AI-driven attendance capture methods — **facial recognition** via the DeepFace Facenet512 model and **QR code scanning** via OpenCV — eliminating the need for physical registers entirely. A three-role architecture (Admin, Teacher, Student) ensures that each user type interacts only with data and functionality relevant to their responsibilities. Automated attendance alerts, activity assignment workflows, complaint management, and school-scoped notice broadcasting round out a system that goes well beyond basic attendance tracking.

The platform is built as a decoupled application: a **Flask REST API backend** with SQLite persistence accessed through SQLAlchemy, and a **React + Vite frontend** communicating via JWT-authenticated Axios requests.

---

## Objectives

- Implement a dual-method attendance capture system combining DeepFace-based facial recognition (Facenet512) and OpenCV QR code decoding, both accessible from a live webcam feed in the browser.
- Design a student enrollment pipeline that captures a minimum of five face frames through the browser webcam, computes a normalized average Facenet512 embedding, and stores it in the database for future recognition.
- Build a three-role authentication system (Admin, Teacher, Student) with JWT access tokens, role-based route protection on both backend and frontend, and automatic routing to role-specific dashboards on login.
- Provide a Teacher dashboard with student list management, face/QR attendance marking, notice broadcasting, activity creation, and submission verification workflows.
- Provide an Admin dashboard with teacher management (create, list, delete), school-wide statistics, system health analytics, and student complaint review.
- Provide a Student dashboard with personal attendance history, school-scoped notice inbox, activity submission portal, complaint raising, and profile/photo self-service management.
- Implement automated attendance alert logic that generates in-system notices for two consecutive absences and for monthly attendance falling below the 75% threshold.
- Enable CSV export of attendance records for both teachers (school-wide) and individual students (personal history).

---

## Summary

SmartAttend is structured as a Flask application factory (`create_app()`) that registers five blueprints — `auth`, `admin`, `teacher`, `attendance`, and `student` — each namespaced under a URL prefix. SQLAlchemy with SQLite provides ORM-managed persistence; Flask-Migrate with Alembic manages schema versioning; Flask-JWT-Extended handles stateless token authentication.

On startup, the application initializes the database, creates all tables if absent, and seeds a default ADMIN user (`admin@school.com` / `admin123`) if none exists. All protected endpoints require a valid Bearer token and validate the caller's role before processing requests.

Student enrollment is performed by teachers via a webcam-based face capture flow. The `AddStudent` React component automatically captures 12 frames at 300 ms intervals after the camera opens, sends them to `/teacher/add-student` as a multipart form, and the backend computes a 512-dimensional normalized average embedding via DeepFace. A unique QR code is simultaneously generated and stored for each student.

Attendance marking occurs through the `AttendancePage` component, which auto-scans the webcam every 4 seconds. In FACE mode, each captured frame is sent to `/attendance/face`, which runs DeepFace recognition against all stored embeddings using L2 distance with a threshold of 0.9. In QR mode, each frame is sent to `/attendance/qr-image`, which decodes the QR code using OpenCV's `QRCodeDetector` and matches it against the student's stored `qr_code` field. Both methods enforce a one-record-per-day deduplication check.

---

## Features

### Public Access

- **Landing Page** — Introductory page with navigation to staff login and student login portals.
- **Staff Login** (`/login`) — Email/password login for Admin and Teacher users. On success, a JWT is stored in `localStorage` and the user is redirected to their role-specific dashboard.
- **Student Login** (`/student-login`) — Roll number, password, school, class, and section login. Students authenticate against hashed passwords using Werkzeug.
- **Password Reset Pages** — Dedicated reset forms for teacher and student password self-service.

### After Login — Admin Portal (`/admin`)

- **Overview Dashboard** — Displays aggregate statistics: total teachers, total students, number of registered schools, total attendance records, active student count, and a system health score computed as `(active_students / total_students) * 100`.
- **Teacher Management** — Create new teacher accounts with name, email, password, school, and subject. View the full teacher roster with school and subject. Delete teacher accounts with confirmation.
- **School & Subject Registry** — List all distinct schools and subjects registered in the system via teacher account metadata.
- **Complaint Management** — View all student complaints across the system with student name, roll number, message, and status (OPEN / READ / RESOLVED). Mark complaints as read with timestamp recording.

### After Login — Teacher Portal (`/teacher`)

- **Dashboard Overview** — Displays total students in the teacher's school, overall attendance rate, and count of students below 70% attendance threshold.
- **Enroll Student** — Webcam-based face capture enrollment: fill student details (name, roll, class, section, optional email), click enroll, and the system automatically captures 12 frames, computes a Facenet512 embedding, generates a unique QR code, and registers the student. Default password is `student@123`.
- **Attendance Marking** — Dual-mode webcam attendance interface. FACE mode: camera scans every 4 seconds, sends frames to the facial recognition API, and displays recognition results in real time. QR mode: camera decodes QR codes from the frame using OpenCV and marks attendance. Both modes show student name, roll number, timestamp, and deduplicate same-day entries.
- **Student List** — Table of all students enrolled in the teacher's school with name, roll, class, and section. Supports student deletion with cascading attendance record removal.
- **Notice Broadcast** — Send school-scoped text notices to all students belonging to the teacher's school.
- **Activity Management** — Create activities (title, description, class, section) targeting specific class-section groups within the school. View all activities created by the teacher. Review photo submissions from students with student name and roll number. Approve or reject submissions.
- **CSV Export** — Download a complete attendance CSV for all students in the teacher's school with columns: Name, Roll, Date, Time, Method.

### After Login — Student Portal (`/student`)

- **Personal Dashboard** — Displays student identity (name, roll, class, section, email, profile photo), total attendance count, complete chronological attendance history with date/time/method, and the latest school notice.
- **My Attendance** — Dedicated attendance history view with date, time, and marking method (FACE / QR).
- **Activity Submission** — Browse activities assigned to the student's class and section. Submit a photo for each activity (one submission per activity enforced). View submission history with title, status (PENDING / APPROVED / REJECTED), and date.
- **Notices** — School-scoped notice inbox displaying the five most recent broadcasts from teachers in the same school.
- **Complaint Portal** — Raise grievances as free-text complaints with duplicate suppression (same text + OPEN status). View all own complaints with status and read timestamp. Mark complaints as self-resolved.
- **Profile Update** — Update email, password, and profile photo. Photos are saved to `uploads/students/<id>.jpg`.
- **CSV Download** — Download personal attendance records as a CSV with Date, Time, and Method columns.

---

## Technologies Used

### Programming Languages
- Python 3.x
- JavaScript (ES2022, ESM)

### Libraries & Frameworks
- Flask 3.1.2 — REST API server and application factory
- Flask-CORS 6.0.2 — Cross-origin request handling for the React frontend
- Flask-JWT-Extended 4.7.1 — JWT creation, verification, and identity extraction
- Flask-SQLAlchemy 3.1.1 — ORM-based database access
- Flask-Migrate 4.1.0 / Alembic 1.18.1 — SQLite schema version management
- React 19 — Component-based frontend SPA
- React Router DOM 7.13 — Client-side three-role routing with protected route guards
- Vite 7.2.4 — Frontend build tooling and development server
- Axios 1.13.2 — HTTP client with JWT interceptor
- react-webcam 7.2.0 — Browser webcam access for enrollment and attendance
- html5-qrcode / jsqr / @yudiel/react-qr-scanner — QR reading support libraries

### Machine Learning / AI
- DeepFace 0.0.79 (Facenet512 model) — Face embedding extraction (512-dimensional) for enrollment and recognition
- OpenCV (opencv-python 4.8.1.78) — Image decoding from uploaded frames, QR code detection and decoding via `cv2.QRCodeDetector`
- NumPy 1.26.4 — Embedding normalization (`v / np.linalg.norm(v)`) and L2 distance computation
- TensorFlow 2.15.0 / Keras 2.15.0 — DeepFace model backend

### Backend
- Flask REST API with Blueprint-based modular routing
- Werkzeug — Student password hashing (`generate_password_hash`, `check_password_hash`)
- qrcode 8.2 — QR code image generation for student enrollment
- pandas 2.3.3 — Data processing utilities
- Gunicorn 24.1.1 — Production-ready WSGI server

### Frontend
- React functional components with Hooks (`useState`, `useEffect`, `useRef`)
- `localStorage` for JWT token, role, and user ID persistence
- Axios request interceptor for automatic `Authorization: Bearer` header injection
- `RequireAuth` component — Role-validated route guard wrapping all protected pages

### Database
- SQLite 3 (`instance/attendance.db`) — Default embedded relational database
- SQLAlchemy 2.0.46 — ORM with declarative model definitions

### Deployment & DevOps
- Flask development server (`run.py`)
- Vite development server (`npm run dev`) with proxy-free Axios baseURL pointing to `localhost:5000`
- Alembic migration scripts for schema versioning

### Security
- JWT Bearer token authentication on all protected API routes via `@jwt_required()`
- Role validation (`admin.role != "ADMIN"`, `teacher.role != "TEACHER"`) inside every protected route handler
- Werkzeug password hashing for all student passwords
- School-scoped data isolation: student and teacher queries are filtered by `school` field to prevent cross-school data access
- Duplicate attendance prevention: one record per `(student_id, day)` enforced at the database query level before insert

---

## Dataset

SmartAttend does not use a pre-collected external dataset. Instead, all recognition data is generated at runtime through the enrollment pipeline:

**Face Embeddings:** During student enrollment, the teacher's browser webcam captures 12 JPEG frames at 300 ms intervals. Each frame is decoded by OpenCV and passed through the DeepFace Facenet512 model to extract a 512-dimensional embedding. All valid embeddings are averaged and L2-normalized to produce a single representative embedding per student, stored in SQLite as a `PickleType` column.

**QR Codes:** Each student's QR code is generated at enrollment using the `qrcode` library. The code string follows the format `ATT_{roll}_{uuid4_hex[:6]}`, providing uniqueness while embedding the roll number for traceability. The PNG image is saved to `uploads/qr/<roll>.png`.

**Attendance Records:** The `Attendance` table accumulates records over time, each storing `student_id`, `teacher_id`, `day`, `time`, `status`, and `method` (FACE or QR). This accumulated data powers attendance percentage calculations, CSV exports, and the automated alert system.

**Demo Data:** The `uploads/` directory in the repository contains pre-generated QR codes for various roll numbers and sample activity photos, enabling immediate testing without running the full enrollment pipeline.

---

## System Architecture

```
┌─────────────────────────────────────────────────────────────────────┐
│                     React + Vite Frontend                           │
│                                                                     │
│  / (Home) │ /login │ /student-login │ /admin │ /teacher │ /student │
│                                                                     │
│  ┌──────────────┐  ┌──────────────────┐  ┌────────────────────────┐│
│  │ AdminDashboard│  │  TeacherDashboard│  │   StudentDashboard    ││
│  │ Teacher CRUD │  │  Attendance Page │  │  History / Activities ││
│  │ Complaints   │  │  Student Enroll  │  │  Complaints / Notices ││
│  │ Analytics    │  │  Activities      │  │  Profile / CSV        ││
│  └──────────────┘  └──────────────────┘  └────────────────────────┘│
│                          │  Axios + JWT Bearer                     │
└──────────────────────────┼──────────────────────────────────────────┘
                           │ HTTP REST API (localhost:5000)
                           ▼
┌─────────────────────────────────────────────────────────────────────┐
│                  Flask Application Server (run.py)                  │
│                       create_app() factory                          │
│                                                                     │
│  /auth/*        /admin/*       /teacher/*    /attendance/*          │
│  Login          Add/Del/List   Add-Student   /face  (DeepFace)      │
│  JWT Issue      Teachers       Students      /qr    (OpenCV)        │
│                 Analytics      Activities    /qr-image              │
│                 Complaints     Notices                              │
│                                                                     │
│  /student/*                                                         │
│  Login, Dashboard, Attendance, Notices, Activities, Complaints      │
│                                                                     │
│  ┌──────────────────┐   ┌──────────────────┐                       │
│  │   face_service   │   │   qr_service     │                       │
│  │  DeepFace/Facenet│   │  qrcode library  │                       │
│  │  OpenCV decode   │   │  UUID-based codes│                       │
│  └──────────────────┘   └──────────────────┘                       │
└───────────────────────────────┬─────────────────────────────────────┘
                                │
                                ▼
┌─────────────────────────────────────────────────────────────────────┐
│                  SQLite Database (attendance.db)                    │
│  user │ student │ attendance │ notice │ complaint                  │
│  activity │ activity_submission                                     │
└─────────────────────────────────────────────────────────────────────┘
```

---

## Model Workflow

### Student Enrollment Pipeline

**Step 1 — Form Submission:** Teacher fills student details (name, roll number, class, section, optional email) in `AddStudent.jsx` and clicks Enroll.

**Step 2 — Webcam Activation:** The browser webcam opens. The component auto-triggers capture after 1.2 seconds.

**Step 3 — Frame Capture:** 12 JPEG frames are captured at 300 ms intervals using `react-webcam`'s `getScreenshot()`. Each frame is converted to a Blob and collected.

**Step 4 — Multipart Upload:** All frames are appended to a `FormData` object alongside the student form fields and POSTed to `/teacher/add-student`.

**Step 5 — Embedding Computation:** `get_average_embedding()` in `face_service.py` decodes each frame with OpenCV, passes it through `DeepFace.represent()` with `model_name="Facenet512"` and `enforce_detection=False`, extracts the 512-dimensional vector, and L2-normalizes it. At least 5 valid embeddings are required; otherwise enrollment is rejected.

**Step 6 — Averaging and Storage:** Valid embeddings are stacked and averaged (`np.mean(..., axis=0)`), then re-normalized. The resulting embedding is stored as a `PickleType` SQLAlchemy column in the `Student` table.

**Step 7 — QR Generation:** `generate_qr(roll)` creates a code string `ATT_{roll}_{uuid4[:6]}`, generates the QR image using `qrcode.make()`, saves it as a PNG, and stores both the code string and image path in the `Student` record.

### Attendance Recognition Pipeline

**Face Mode:**

1. `AttendancePage` captures a webcam frame every 4 seconds and POSTs it as `multipart/form-data` to `/attendance/face`.
2. `recognize_live()` in `face_service.py` decodes the image with OpenCV and computes the live Facenet512 embedding.
3. The function iterates over all `Student` records with stored embeddings, computes the L2 distance between the live and stored (normalized) embedding vectors, and identifies the closest match.
4. If the best distance is below the threshold of **0.9**, the matched student is returned; otherwise `NO_MATCH` is returned.
5. `mark()` checks whether an attendance record already exists for `(student_id, today)`. If not, it inserts a new `Attendance` record with `method="FACE"`.

**QR Mode:**

1. `AttendancePage` captures a webcam frame every 4 seconds and POSTs it as `multipart/form-data` to `/attendance/qr-image`.
2. The frame is decoded by OpenCV (`cv2.imdecode`) and processed by `cv2.QRCodeDetector().detectAndDecode()`.
3. The extracted code string is matched against `Student.qr_code` via a database query.
4. `mark()` applies the same one-per-day deduplication before inserting the record with `method="QR"`.

### Automated Alert Pipeline

The `attendance_alerts.py` module implements `process_attendance_alerts(student_id)`:

1. **Consecutive Absence Check:** Retrieves the two most recent attendance records. If both have `status="ABSENT"`, a notice is generated with the message "⚠ You were absent for 2 consecutive days." — with duplicate suppression.
2. **Monthly Threshold Check:** Aggregates all attendance records for the current month and year. If the present-to-total ratio falls below 75%, a notice is generated with the exact percentage and the 75% minimum requirement message — with duplicate suppression.
3. All generated notices are committed in a single `db.session.commit()`.

---

## Implementation Details

**Backend (`frontend/backend/`):**

`app/__init__.py` implements the Flask application factory. On first run, it seeds a default admin user (`admin@school.com`, `admin123`, role `ADMIN`) if the `user` table contains no admin. All blueprints are registered inside the factory to avoid circular imports.

`app/routes/auth.py` handles staff login via `/auth/login`. Passwords are stored and compared as plaintext for the `User` model (admin/teacher accounts). A JWT access token with no expiry (`JWT_ACCESS_TOKEN_EXPIRES = False`) is issued on success.

`app/routes/admin.py` provides teacher CRUD, school/subject enumeration (via `DISTINCT` queries on the `User` table), system-wide statistics, analytics with system health score, and complaint management.

`app/routes/teacher.py` handles student enrollment (multipart, includes face frames), student list retrieval filtered by teacher's school, attendance CSV export, notice sending (school-scoped), and activity lifecycle (create, list, view submissions, approve/reject).

`app/routes/attendance.py` provides three endpoints: `/attendance/face` (DeepFace recognition), `/attendance/qr` (code string lookup), and `/attendance/qr-image` (OpenCV image decode). All three call the shared `mark()` function which enforces the one-per-day constraint. The `attendance_open()` utility in `time_rules.py` currently returns `True` unconditionally; it is designed as an extension point for time-window enforcement.

`app/routes/student.py` handles student login (roll number + password + school + class + section), dashboard data aggregation, profile updates, CSV download, school/class/section discovery endpoints (unauthenticated, used to populate the student login form dropdowns), activity submission, and complaint management.

`app/services/face_service.py` is the core AI module. `_get_single_embedding()` wraps `DeepFace.represent()` with shape validation (must be 512-dimensional). `get_average_embedding()` averages multiple frame embeddings for robust enrollment. `recognize_live()` uses L2 distance between normalized embeddings rather than cosine similarity; the threshold of 0.9 was selected to balance false-positive rejection against genuine recognition recall.

`app/services/qr_service.py` generates QR codes with a UUID suffix to prevent code collision across students with the same roll number in different schools or sections.

**Frontend (`frontend/src/`):**

`src/app/Router.jsx` defines the route tree. `RequireAuth` wraps protected routes; it reads `role` from `localStorage` and redirects unauthorized users to `/`. Public `StudentLogin` uses a different login endpoint and credential set from the staff `Login` form.

`src/services/api.js` creates an Axios instance with `baseURL: "http://127.0.0.1:5000"` and an interceptor that reads the token from `localStorage` on every request, appending it as the Authorization header.

`src/utils/storage.js` provides simple `localStorage` wrappers (`setAuth`, `getToken`, `getRole`, `getUserId`, `logout`) used throughout the application for session state management.

`AttendancePage.jsx` uses a `setInterval` (4-second tick) combined with a `cooldown` state flag to prevent concurrent API calls. The cooldown is lifted after the result info is cleared by a 5-second `setTimeout`. This design avoids overlapping requests while keeping the UI responsive.

`AddStudent.jsx` uses a `for` loop with `await new Promise(r => setTimeout(r, 300))` to sequence frame capture with deliberate delays, giving the camera autofocus time to stabilize between shots and improving embedding quality.

---

## Results

SmartAttend demonstrates practical functionality across all core workflows:

**Facial Recognition Accuracy:** Using the Facenet512 model with L2 distance thresholding at 0.9, the system reliably identifies enrolled students under typical classroom lighting conditions. The average-embedding enrollment approach (minimum 5 of 12 captured frames) improves robustness compared to single-frame enrollment by smoothing pose and expression variation.

**QR Attendance:** OpenCV's `QRCodeDetector` processes frames captured from the browser webcam without requiring any external barcode library (zbar-free), making deployment simpler in low-dependency environments. The detection correctly handles common QR presentation scenarios including slight rotation and partial shadow.

**Deduplication Integrity:** The `mark()` function enforces one attendance record per `(student_id, date)` at the query level, preventing duplicate marking from either attendance method regardless of how many times a student appears in the camera frame within a session.

**Role Isolation:** School-scoped filtering ensures that teachers only see students from their registered school, and students only receive notices from their own school. Admin operations span all schools but are JWT-role-gated.

**Alert System:** Consecutive-absence and monthly-threshold alerts are generated with duplicate suppression, preventing notice spam while ensuring students are informed of attendance issues.

**CSV Export:** Both teacher-level (school-wide) and student-level (personal) attendance exports are available as streaming CSV responses, enabling integration with external record-keeping systems without database access.

---

## Installation

### Prerequisites

- Python 3.10 or higher
- Node.js 18 or higher and npm
- A system with a functioning webcam (for attendance and enrollment features)

### Backend Setup

```bash
# Clone the repository
git clone https://github.com/harisaigithub/attandance_management_rural_RV.git
cd attandance_management_rural_RV/frontend/backend

# Create and activate a virtual environment
python -m venv venv
source venv/bin/activate       # On Windows: venv\Scripts\activate

# Install dependencies
# Note: requirements.txt is UTF-16 encoded; use pip directly
pip install Flask Flask-CORS Flask-JWT-Extended Flask-SQLAlchemy Flask-Migrate \
            deepface opencv-python-headless numpy qrcode pillow werkzeug pandas \
            python-dateutil gunicorn

# Initialize the database and run the server
python run.py
```

The Flask API server will start at `http://127.0.0.1:5000`. A default admin account (`admin@school.com` / `admin123`) is seeded automatically on first run.

### Frontend Setup

```bash
# From the project root
cd frontend

# Install Node dependencies
npm install

# Start the Vite development server
npm run dev
```

The React application will be available at `http://localhost:5173`.

### Database Migrations (Optional)

If schema changes are required, use Alembic via Flask-Migrate:

```bash
cd frontend/backend
flask db init      # Only on first migration setup
flask db migrate -m "describe change"
flask db upgrade
```

---

## Usage

### Admin Login

1. Navigate to `http://localhost:5173/login`.
2. Enter credentials: `admin@school.com` / `admin123`.
3. The Admin Dashboard opens with system statistics, teacher management, and complaints.

### Teacher Workflow

1. Admin creates a Teacher account from the Admin Dashboard with school and subject.
2. Teacher logs in at `/login` with their assigned credentials.
3. Navigate to **Enroll Student**: fill student details, click Enroll, and hold each student's face in the webcam frame for automatic 12-frame capture.
4. Navigate to **Attendance**: choose FACE or QR mode, click Start Camera, and present students one by one. The system marks attendance automatically on recognition.
5. Send school-wide **Notices** and create **Activities** for specific class-section groups.
6. Review and verify student activity photo submissions.
7. Download school-wide attendance as CSV from **Reports**.

### Student Login

1. Navigate to `http://localhost:5173/student-login`.
2. Enter roll number, password (`student@123` by default), school, class, and section. School/class/section dropdowns are populated from the API.
3. The Student Dashboard shows attendance history, latest notice, and navigation to activities, complaints, and profile.

---

## Future Enhancements

- **Time-Window Enforcement** — Activate the `attendance_open()` function in `time_rules.py` to restrict attendance marking to configurable school hours, preventing off-hours manipulation.
- **SMS/WhatsApp Parent Alerts** — Integrate Twilio or a WhatsApp Business API to dispatch low-attendance alerts and consecutive-absence notices directly to parents' mobile numbers.
- **Anti-Spoofing (Liveness Detection)** — Add a liveness detection layer to the face recognition pipeline to reject photo-based spoofing attempts, using blink detection or depth estimation.
- **Offline PWA Mode** — Convert the React frontend to a Progressive Web App with offline-first capability so that teachers can mark attendance without live internet and sync records when connectivity is restored.
- **Real-Time Dashboard with WebSockets** — Replace polling-based stat loading with Flask-SocketIO push events for live attendance count updates on the teacher dashboard during an active session.
- **Multi-Class Teacher Support** — Allow a teacher to be associated with multiple classes or subjects, extending the current single-school constraint to class-level scoping.
- **PDF Attendance Reports** — Add ReportLab-based PDF generation for monthly and term-wise attendance summaries, providing printable records for school authorities.
- **Face Re-Enrollment** — Add a re-enrollment route that updates a student's embedding without deleting attendance history, accommodating appearance changes over time.
- **Docker Containerization** — Package the Flask backend and its heavy ML dependencies (TensorFlow, DeepFace, OpenCV) in a Docker container to simplify deployment in environments without Python environment management tools.
- **Role-Scoped Complaint Resolution** — Allow teachers to directly resolve complaints from their school's students, in addition to the current admin-only read workflow.

---

## Contributing

Contributions are welcome. Please follow the standard Git workflow:

```bash
# Fork the repository and clone your fork
git clone https://github.com/<your-username>/attandance_management_rural_RV.git
cd attandance_management_rural_RV

# Create a feature branch
git checkout -b feature/your-feature-name

# Make your changes
git add .
git commit -m "feat: describe your change"

# Push and open a Pull Request
git push origin feature/your-feature-name
```

**Contribution Guidelines:**
- All new Flask routes must use `@jwt_required()` and validate the caller's role before processing.
- New models must be added to `app/models/`, imported in `app/__init__.py`, and accompanied by an Alembic migration.
- Frontend route additions must be reflected in `src/app/Router.jsx` and protected with the `RequireAuth` component where applicable.
- Any changes to the face recognition threshold (currently 0.9) should be justified with benchmark results in the PR description.
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