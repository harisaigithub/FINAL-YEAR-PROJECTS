# SmartAttend — AI-Powered Attendance Management System for Rural Schools

> **Tagline:** Bridging the technology gap in rural education through AI-driven facial recognition and QR-based attendance — built for schools with limited infrastructure, delivering enterprise-grade accuracy.

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

SmartAttend is a full-stack, AI-powered attendance management system designed specifically for rural and underserved schools in India. Traditional paper-based or manual attendance systems are not only time-consuming but also prone to proxy manipulation, data loss, and administrative overhead — challenges magnified in schools with limited staff and resources.

This system addresses those challenges by introducing two contactless, automated attendance modalities: **facial recognition** powered by deep learning and **QR code scanning** using computer vision. The platform is built on a React + Flask architecture with role-based access control, supporting three distinct user roles — Administrator, Teacher, and Student — each with a tailored dashboard and permission scope.

SmartAttend is more than an attendance tool; it provides real-time analytics, school-level performance health metrics, student complaint management, activity tracking, and notice broadcasting, all within a single unified platform. The system is lightweight enough to run on commodity hardware while remaining scalable for multi-school deployments.

---

## Objectives

- Design a multi-role attendance management system supporting Administrator, Teacher, and Student access levels with JWT-secured authentication.
- Implement real-time facial recognition attendance using the DeepFace Facenet512 model for accurate biometric identification.
- Provide an alternative QR code-based attendance mechanism using OpenCV's native QR decoder for environments without camera access.
- Enforce school-level data isolation so that teachers and students can only access records belonging to their respective institution.
- Deliver a responsive single-page application (SPA) frontend built with React and React Router, offering role-redirected navigation upon login.
- Enable administrators to manage multi-school deployments, register teachers, monitor system-wide analytics, and review student complaints.
- Allow teachers to enroll students with biometric face samples, manage class activities, download attendance reports as CSV, and send school-wide notices.
- Provide students with a dedicated dashboard to view their attendance history, submit complaints, track activity submissions, and review received notices.
- Implement time-gated attendance marking to ensure records are only captured during designated school hours.
- Support database schema evolution through Alembic-based migration management.

---

## Summary

SmartAttend is a web-based attendance automation platform that integrates deep learning-based facial recognition with QR code scanning to eliminate manual record-keeping in rural school environments. The backend is a Flask REST API with SQLite persistence via SQLAlchemy ORM, secured using Flask-JWT-Extended for stateless token authentication. The frontend is a modular React application with role-based routing, styled with custom CSS per module.

Student enrollment captures a minimum of five facial image samples per student, from which a normalized 512-dimensional average embedding is computed using the Facenet512 model via the DeepFace library. During attendance marking, a live image is submitted, an embedding is extracted, and cosine-distance comparison against all stored embeddings is performed to identify the closest match. QR attendance generates a unique UUID-encoded code per student at enrollment, stored as a PNG and verified during scanning via OpenCV.

The system maintains complete separation between schools — teachers see only their school's students, attendance records are scoped accordingly, and notice broadcasts are school-specific. Administrators operate at a global level, overseeing teacher management, multi-school analytics, and complaint resolution.

---

## Features

### Before Login (Public)

- **Home Page** — A branded landing page introducing SmartAttend with navigation links to the Admin/Teacher login portal and the Student login portal.
- **Dual Login Portals** — Separate authentication flows for staff (Admin and Teacher) and students, each with distinct credential validation and redirect logic.
- **Role-Based Redirect** — Authenticated users attempting to access the login page are automatically redirected to their respective dashboards based on stored role identity.

### After Login — Administrator

- **System Statistics Dashboard** — Aggregate counts of registered teachers, students, and active schools displayed in real time.
- **Analytics Overview** — System health score computed as the ratio of attendance-active students to total enrolled students, with breakdown metrics.
- **Teacher Management** — Provision new teacher accounts with name, email, password, school, and subject assignment. View all registered teachers. Delete teacher accounts with confirmation.
- **School & Subject Registry** — Dynamic enumeration of all schools and subjects drawn from teacher registrations, presented as filterable reference lists.
- **Complaint Management** — View all student-submitted complaints across schools with student identity, message content, submission timestamp, and status. Mark complaints as read.

### After Login — Teacher

- **Teacher Dashboard** — Summary metrics including total students in the teacher's school, overall attendance rate, and count of students below 70% attendance threshold.
- **Student Enrollment** — Register new students by capturing at least five facial photographs, which are processed into a single normalized face embedding. Simultaneously generates a unique QR code for each student. Duplicate roll number + school + section combinations are rejected.
- **Student List Management** — View all students belonging to the teacher's school. Delete student records along with associated attendance logs.
- **Attendance Marking — Face Mode** — Submit a live photograph via the frontend; the backend extracts a Facenet512 embedding and matches it against all enrolled students using Euclidean distance with a configurable threshold. Returns matched student identity with timestamp.
- **Attendance Marking — QR Code Mode** — Accept a decoded QR string or a raw QR image file (decoded server-side with OpenCV), verify against the student registry, and mark attendance.
- **Duplicate Prevention** — Any student already marked present for the current calendar day is rejected, regardless of attendance method.
- **Attendance Report Export** — Download a CSV file containing all attendance records (name, roll, date, time, method) for the teacher's school.
- **Activity Management** — Create class activities specifying title, description, class, and section. View submitted student responses and approve or reject each submission individually.
- **Notice Broadcasting** — Send school-scoped textual notices visible to all students within the same institution.

### After Login — Student

- **Student Dashboard** — Personal attendance history with date, time, and method of each recorded entry. Attendance percentage summary.
- **Profile View** — Display of student name, roll number, class, section, and email.
- **QR Code Access** — View and download personal QR code for use in QR-based attendance sessions.
- **Complaint Submission** — Lodge complaints or concerns directed to the administrative team with status tracking (OPEN / READ / RESOLVED).
- **Notice Board** — Receive and read school-specific notices broadcast by teachers.
- **Activity Submissions** — Upload photograph-based activity submissions for teacher review and track approval status.

---

## Technologies Used

### Programming Languages
- **Python 3.x** — Backend application logic, REST API, and AI services
- **JavaScript (ES2020+)** — Frontend application and API communication

### Libraries & Frameworks
- **React 18** — Component-based frontend SPA framework
- **React Router v6** — Client-side routing with role-protected routes
- **Axios** — HTTP client for API communication from the frontend
- **Flask 3.1.2** — Lightweight Python web framework for REST API construction
- **Flask-CORS 6.0.2** — Cross-Origin Resource Sharing support for SPA-backend communication
- **Werkzeug 3.1.5** — Password hashing utilities (PBKDF2-based)

### Machine Learning / AI
- **DeepFace 0.0.79** — Face embedding extraction framework
- **Facenet512** — Deep neural network model used for generating 512-dimensional facial embeddings
- **TensorFlow 2.15.0 / Keras 2.15.0** — Deep learning backend powering DeepFace model inference
- **OpenCV (opencv-python 4.8.1.78)** — Image decoding, preprocessing, and QR code detection
- **NumPy 1.26.4** — Numerical operations including embedding normalization and Euclidean distance computation
- **retina-face 0.0.13** — Face detection preprocessing used within the DeepFace pipeline

### Backend / Frontend
- **Flask Blueprints** — Modular API route organization by role (admin, teacher, student, attendance, auth)
- **Vite** — Frontend build tooling and development server
- **Custom CSS Modules** — Per-component stylesheets for admin, teacher, student, login, and attendance views

### Database
- **SQLite** — File-based relational database (`attendance.db`) for development and lightweight deployment
- **SQLAlchemy 2.0.46** — ORM for model definition, query construction, and relationship management
- **Flask-SQLAlchemy 3.1.1** — Flask integration layer for SQLAlchemy
- **Alembic 1.18.1 / Flask-Migrate 4.1.0** — Database schema migration management

### Deployment & DevOps
- **Gunicorn 24.1.1** — Production-grade WSGI HTTP server for Flask deployment
- **qrcode 8.2** — QR code image generation at student enrollment
- **Pillow 12.1.0** — Image file handling for QR code output

### Security
- **Flask-JWT-Extended 4.7.1** — JSON Web Token issuance and verification for stateless session management
- **PyJWT 2.10.1** — Underlying JWT encoding and decoding library
- **Werkzeug password hashing** — Secure password storage using PBKDF2-SHA256 with salting for student accounts

---

## System Architecture

SmartAttend follows a decoupled client-server architecture consisting of three logical tiers:

**Presentation Layer (React SPA)**
The frontend is a single-page application organized by feature module. Authentication state is stored in `localStorage` via utility functions (`getToken`, `getRole`, `setAuth`, `logout`). The `Router.jsx` component defines all routes with a `RequireAuth` guard that validates the stored JWT and role before rendering protected dashboard components. Each role has a dedicated dashboard component — `AdminDashboard.jsx`, `TeacherDashboard.jsx`, and `StudentDashboard.jsx` — which communicate with the backend through a centralized `api.js` Axios instance.

**Application Layer (Flask REST API)**
The backend is structured using Flask's Application Factory pattern. The `create_app()` function initializes extensions (SQLAlchemy, Flask-Migrate, Flask-JWT-Extended, CORS), registers all Blueprints, and seeds a default admin user on first launch. Routes are organized into five Blueprints: `/auth`, `/admin`, `/teacher`, `/student`, and `/attendance`. Every protected endpoint requires a valid JWT and performs role verification before processing the request.

**Data Layer (SQLite via SQLAlchemy)**
Six primary models are defined: `User` (admin and teacher accounts), `Student` (student records including biometric data), `Attendance` (per-student daily records with method and timestamp), `Activity` (teacher-created class tasks), `ActivitySubmission` (student responses to activities), `Notice` (school-scoped announcements), and `Complaint` (student-submitted grievances). School-level data isolation is enforced at the query layer — all teacher-scoped queries filter by `teacher.school`, and student access is bounded by their enrollment record.

**AI Services Layer**
Two service modules handle intelligent processing. `face_service.py` uses DeepFace to extract Facenet512 embeddings from uploaded images, normalizes them using L2 normalization, averages multiple enrollment samples into a single representative embedding, and performs nearest-neighbour matching at inference time. `qr_service.py` generates UUID-appended QR codes using the `qrcode` library at enrollment and saves them to the `/uploads/qr/` directory for student download.

---

## Model Workflow

### Student Enrollment (Biometric Registration)

1. **Image Capture** — Teacher uploads a minimum of five face photograph files for the new student via the enrollment form.
2. **Decoding** — Each file is read as binary and decoded into an OpenCV BGR image array using `cv2.imdecode`.
3. **Embedding Extraction** — DeepFace's `represent()` function is called with the Facenet512 model and `enforce_detection=False` to handle partially obscured faces. A 512-dimensional float vector is returned per image.
4. **Normalization** — Each embedding is L2-normalized to unit length, making cosine similarity equivalent to dot product.
5. **Averaging** — All valid normalized embeddings are stacked and averaged element-wise using NumPy, then re-normalized to produce a single, stable representative embedding.
6. **Persistence** — The averaged embedding is stored as a Python `PickleType` column in the `Student` model. A unique QR code string is simultaneously generated and its PNG saved to disk.

### Attendance Marking — Face Mode

1. **Image Submission** — Teacher uploads a photograph of the student via the attendance interface.
2. **Live Embedding Extraction** — The same Facenet512 pipeline extracts and normalizes a 512-dimensional embedding from the submitted image.
3. **Database Matching** — The system iterates over all enrolled students, retrieves their stored embeddings, and computes the Euclidean distance between the live embedding and each stored embedding.
4. **Threshold Decision** — The student with the minimum distance is selected. If the minimum distance is below a configurable threshold (default: 0.9), the match is accepted; otherwise, the response returns `NO_MATCH`.
5. **Duplicate Guard** — Before writing, the system checks for an existing attendance record with the same `student_id` and the current calendar date. Duplicate records are rejected with an `ALREADY` status.
6. **Record Creation** — An `Attendance` record is committed with `method="FACE"`, the teacher's ID, the current date, and a UTC timestamp.

### Attendance Marking — QR Mode

1. **Code Acquisition** — The frontend either submits a decoded QR string directly or uploads a QR code image file.
2. **Image Decoding (if image)** — The uploaded image is decoded into an OpenCV frame, and the `QRCodeDetector` extracts the encoded string without requiring external ZBar dependencies.
3. **Student Lookup** — The decoded QR string is matched against the `qr_code` field in the Student table.
4. **Duplicate Guard and Record Creation** — Identical to the face mode pipeline: duplicate check followed by an `Attendance` record commit with `method="QR"`.

### Time Gate Enforcement

The `attendance_open()` utility in `time_rules.py` governs whether attendance marking endpoints are active. Requests outside the permitted time window receive a `403 Closed` response, preventing off-hours tampering.

---

## Implementation Details

**Backend Structure**
The Flask backend is organized under `frontend/backend/` with the application package in `app/`. The factory function in `__init__.py` handles all initialization, including automatic admin seeding using a query-before-create pattern to avoid duplicate admin creation on restart. All routes follow a consistent pattern: extract the JWT identity, query the corresponding user, verify role, process the request, and return a JSON response.

**Biometric Storage**
Face embeddings are stored using SQLAlchemy's `PickleType`, which serializes the NumPy array directly into the SQLite BLOB column. This avoids the overhead of an external vector database while remaining adequate for school-scale deployments (hundreds of students per institution).

**QR Generation**
The `generate_qr` function in `qr_service.py` constructs a unique code string as `ATT_{roll_no}_{uuid4_hex[:6]}`, creates a QR image using the `qrcode` library, and persists the PNG to `uploads/qr/{roll_no}.png`. Both the code string and file path are stored in the Student record for future reference and student download.

**Frontend State Management**
The React frontend uses component-level `useState` and `useEffect` hooks for all data management. There is no global state library (Redux/Zustand); API responses are stored directly in component state. Authentication state is persisted to `localStorage` using three keys: `token`, `role`, and `user_id`. The `RequireAuth` component reads these on every protected route render and redirects to the appropriate login page if the token is absent or the role does not match.

**School Isolation**
School isolation is enforced at the data access layer rather than as a middleware concern. Every teacher-scoped query explicitly filters by `User.school` (fetched from the JWT-identified teacher record), and student enrollment binds the `school` field from the teacher's profile. This design makes it impossible for a teacher from one school to access or modify student records from another, even if the student's `id` is known.

**Migrations**
The project uses Flask-Migrate (Alembic) for schema versioning. Two migration scripts are present for adding the `subject` column to the `User` model, demonstrating the incremental schema evolution approach.

---

## Results

The system achieves the following performance and functional outcomes based on implementation analysis and component design:

| Metric | Value |
|---|---|
| Face Recognition Model | Facenet512 (512-dimensional embeddings) |
| Minimum Enrollment Samples Required | 5 facial images |
| Recognition Distance Threshold | 0.9 (Euclidean, post-normalization) |
| Duplicate Attendance Prevention | Per student, per calendar day |
| QR Decode Method | OpenCV QRCodeDetector (no external dependency) |
| Supported Attendance Methods | Face Recognition, QR String, QR Image |
| Roles Supported | Admin, Teacher, Student |
| Data Isolation | School-scoped at query layer |
| API Authentication | JWT (stateless, no expiry in development mode) |
| Database | SQLite with full ORM and migration support |
| Report Export | CSV (per school, all attendance records) |
| Student Complaint Lifecycle | OPEN → READ → RESOLVED |
| Activity Submission Review | APPROVED / REJECTED by teacher |

The Facenet512 model used within DeepFace is known to achieve verification accuracy exceeding 99% on standard face benchmarks (LFW dataset). In a school context with controlled lighting and frontal face capture, the system is expected to perform with high precision given the multi-sample enrollment strategy that averages out single-image noise.

---

## Installation

### Prerequisites

- Python 3.9 or higher
- Node.js 18 or higher
- npm or yarn
- Git

### Step 1: Clone the Repository

```bash
git clone https://github.com/harisaigithub/attendance-management-rural.git
cd attendance-management-rural
```

### Step 2: Set Up the Backend

```bash
cd frontend/backend

# Create and activate a virtual environment
python -m venv venv
source venv/bin/activate          # On Windows: venv\Scripts\activate

# Install Python dependencies
pip install -r requirements.txt
```

> Note: The `requirements.txt` includes TensorFlow, DeepFace, and OpenCV. Ensure your system has sufficient RAM (minimum 4 GB recommended) for model loading.

### Step 3: Initialize the Database

```bash
# Apply all migrations
flask db upgrade

# Alternatively, the app auto-creates tables on first run
python run.py
```

The default administrator account is seeded automatically on first launch:
- **Email:** `admin@school.com`
- **Password:** `admin123`

### Step 4: Set Up the Frontend

```bash
cd ../../1

# Install Node.js dependencies
npm install
```

### Step 5: Configure the API Base URL

Open `frontend/1/src/services/api.js` and verify the base URL points to your backend server:

```javascript
// Ensure this matches your Flask server address
baseURL: "http://localhost:5000"
```

### Step 6: Run the Application

**Terminal 1 — Backend:**
```bash
cd frontend/backend
source venv/bin/activate
python run.py
```
The Flask server will start on `http://localhost:5000`.

**Terminal 2 — Frontend:**
```bash
cd frontend/1
npm run dev
```
The React development server will start on `http://localhost:5173`.

---

## Usage

### Administrator

1. Navigate to `http://localhost:5173/login` and sign in with the default admin credentials.
2. Access the **Dashboard** to review system-wide statistics (total teachers, students, and active schools).
3. Use the **Analytics** section to monitor the system health score and active student ratios.
4. Navigate to **Add Teacher** to register a new teacher with their name, email, password, school, and subject.
5. View and delete existing teachers from the **Teacher List** panel.
6. Review and mark student complaints from the **Complaints** section.

### Teacher

1. Navigate to `http://localhost:5173/login` and sign in with teacher credentials provided by the admin.
2. On the **Dashboard**, review student count, attendance rate, and low-attendance alerts.
3. Go to **Add Student** to enroll a new student: fill in name, roll number, class, section, and email, then capture at least five face photographs. A QR code is generated automatically.
4. Go to **Attendance — Face** to mark attendance by uploading or capturing a student's photograph.
5. Go to **Attendance — QR** to mark attendance by entering or scanning the student's QR code.
6. Use **Reports** to download a full CSV attendance log for your school.
7. Create class activities via **Activities** and review student submissions using the approval workflow.
8. Broadcast school notices via the **Notice** panel.

### Student

1. Navigate to `http://localhost:5173/student-login` and sign in with roll number and the default password (`student@123`, changeable via profile settings).
2. View the **Attendance History** to review all past attendance records with date, time, and method.
3. Download your personal **QR Code** for use during QR-based attendance sessions.
4. Submit complaints via the **Complaint** section and track their resolution status.
5. Read school announcements from the **Notice Board**.
6. Submit activity responses and monitor their approval status in **Activities**.

---

## Future Enhancements

- **Liveness Detection** — Integrate anti-spoofing measures (blink detection, depth estimation) to prevent photo-based attendance fraud.
- **Mobile Application** — Develop a React Native or Flutter app for teachers to mark attendance directly from smartphones, leveraging the device camera for both face and QR capture.
- **Offline Mode** — Implement a local-first architecture with IndexedDB or SQLite on the client for schools with intermittent internet connectivity, with background sync to the server.
- **Multi-School Admin Hierarchy** — Introduce a regional coordinator role between admin and teacher, capable of managing a cluster of schools.
- **Face Re-enrollment** — Allow teachers to update a student's face embedding without deleting and re-creating the student record, improving accuracy as the student ages.
- **PostgreSQL Migration** — Replace SQLite with PostgreSQL for production deployments to support concurrent access and eliminate file-lock limitations.
- **Attendance Alert Notifications** — Implement SMS or push notification alerts to parents when a student's attendance drops below a configurable threshold, using services such as Twilio or Firebase Cloud Messaging.
- **Comprehensive Reporting Dashboard** — Add interactive charts (monthly trends, subject-wise attendance, class-level heatmaps) rendered with a charting library such as Recharts or Chart.js.
- **Docker Containerization** — Package the backend and frontend into Docker containers orchestrated with Docker Compose for reproducible, one-command deployment.
- **HTTPS and Secret Management** — Enforce TLS in production and migrate sensitive values (JWT secret, database URI) to environment variables or a secrets manager, replacing the current hardcoded configuration.
- **Audit Logging** — Maintain an immutable audit trail of all administrative actions (teacher creation/deletion, complaint resolution, student deletion) for accountability.
- **Biometric Fallback Strategy** — Implement a manual override workflow allowing teachers to mark attendance with an administrative note when both face and QR methods fail.

---

## Contributing

Contributions are welcome. Please follow the standard Git workflow outlined below.

1. Fork the repository on GitHub.
2. Create a feature branch from `main`:
   ```bash
   git checkout -b feature/your-feature-name
   ```
3. Implement your changes with clear, descriptive commits:
   ```bash
   git commit -m "feat: add liveness detection to face attendance endpoint"
   ```
4. Push your branch to your fork:
   ```bash
   git push origin feature/your-feature-name
   ```
5. Open a Pull Request against the `main` branch of the original repository with a detailed description of the changes and the problem they address.
6. Ensure all existing functionality is unaffected by running the application end-to-end before submitting.

Please adhere to PEP 8 for Python code and ESLint standards for JavaScript/React code. Open an issue before beginning work on a major feature to align on design and avoid duplication of effort.

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

For inquiries related to project architecture, technical implementation, or academic collaboration, please reach out through the channels below.

**GitHub:** https://github.com/harisaigithub  
**Email:** harisaiparasa@gmail.com  

---

&nbsp;

&nbsp;

## ***To acquire this project, please contact***

**GitHub:** https://github.com/harisaigithub  
**Email:** [harisaiparasa@gmail.com](mailto:harisaiparasa@gmail.com)

For collaborations, customizations, or deployment support, feel free to reach out.