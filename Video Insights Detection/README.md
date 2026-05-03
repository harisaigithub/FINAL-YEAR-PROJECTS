# Video Insights Detection

> **A full-stack, AI-powered video intelligence platform that combines Google Gemini multimodal analysis, YOLOv8 object detection, OpenCV face recognition, and a Claude-powered conversational chatbot to extract deep, structured insights from any uploaded or YouTube-linked video.**

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

Video content has become the dominant medium of modern communication — yet the intelligence embedded within videos remains largely inaccessible: buried in spoken words, visual scenes, facial expressions, motion patterns, and temporal dynamics that traditional tools cannot surface automatically.

**Video Insights Detection** is a production-grade Django web application that solves this problem end-to-end. Users upload video files or provide YouTube links, and the platform automatically runs a **dual-stage AI analysis pipeline**: first, the Google Gemini Flash multimodal model processes the video for speech transcription, semantic summarization, keyword extraction, object identification, key moment detection, and activity classification; second, a suite of local machine learning models — YOLOv8 Nano for human presence and activity recognition, OpenCV Haar Cascades for face detection, and KeyBERT for engagement trend scoring — performs frame-level computer vision analysis entirely on-device.

The resulting insights are presented through an interactive analytics dashboard and a dedicated AI chatbot interface powered by Claude via the OpenRouter API, enabling users to ask natural language questions about any analyzed video. A full PDF export facility allows structured reporting with all insights, ML results, and chat history compiled into a professionally formatted downloadable document.

The system is designed with two distinct access tiers — regular users and administrators — each with tailored dashboards, access controls, and management capabilities.

---

## Objectives

- Build a secure, role-based Django web application supporting two user tiers (`user` and `admin`) with email OTP verification, session-based authentication, and password reset via SMTP.
- Integrate the **Google Gemini Flash** multimodal model to perform automated speech-to-text transcription, semantic summarization, keyword extraction, visual object detection, activity classification, and key moment identification from video content.
- Implement a **local ML analysis pipeline** using YOLOv8 Nano (human presence detection and activity recognition), OpenCV Haar Cascades (face detection), and KeyBERT with BERT embeddings (engagement trend analysis) — all executed without cloud dependency after model initialization.
- Develop a **multi-signal activity recognition system** that scores eight distinct activity categories (Lecture/Teaching, Presentation, Meeting/Discussion, Tutorial/Demo, Interview, Screen Recording, Physical Activity, Idle) using combined signals from YOLO object classes, face count, motion magnitude, and frame composition.
- Provide an **AI-powered conversational chatbot** (Claude via OpenRouter) that uses the full video analysis data — transcript, summary, keywords, objects, activity — as context to answer user questions about their videos.
- Enable **YouTube video ingestion** via `yt-dlp` stream downloading, seamlessly integrating remote video content into the local analysis pipeline.
- Generate downloadable, structured **PDF insight reports** using ReportLab, including all Gemini and ML outputs along with the full chatbot conversation history.
- Build an **administrative governance layer** with global video statistics, user management (activate/deactivate/delete), AI pipeline health monitoring, and aggregate engagement analytics across all platform users.

---

## Summary

Video Insights Detection is a Django 5.x web application organized into four registered application modules — `accounts`, `videos`, `analysis`, and `chatbot` — each responsible for a distinct domain of the intelligence pipeline.

When a user authenticates, they can ingest a video either by uploading an MP4 file (up to 100 MB) or submitting a YouTube URL. In the YouTube case, `yt-dlp` downloads the best available MP4 stream to the local `media/` directory, which is then treated identically to a direct upload.

Upon saving the video record, the platform triggers a two-stage pipeline. In Stage 1, the video file is uploaded to the Gemini Files API, polled until its state transitions to `ACTIVE`, and then submitted to the `gemini-3-flash-preview` model with a structured JSON-enforcing prompt. The returned JSON — containing transcript, summary, keywords, detected objects, activity type, and key moments — is parsed and persisted to the `VideoAnalysis` model.

In Stage 2, the local `ml_analyzer.py` module samples up to 40 frames from the video using NumPy's `linspace` for uniform temporal coverage, then runs four independent analyses: YOLOv8 Nano on every frame for human presence and object class detection; OpenCV Haar Cascade with histogram equalization for face detection; a multi-signal activity scorer combining YOLO class sets, face counts, motion deltas, and bounding box coverage; and KeyBERT for transcript segmentation and semantic density scoring, combined with optical flow motion signals to produce a per-segment engagement score.

All results are stored in dedicated JSONFields on the `VideoAnalysis` model. Users can then explore insights through an interactive dashboard with engagement wave charts, activity distribution breakdowns, and human presence timelines. The chatbot interface routes questions through the OpenRouter API to Claude, which uses the full analysis context to generate grounded responses. A ReportLab PDF export compiles all seven sections — summary, keywords, objects, key moments, transcript, ML analysis, and chatbot history — into a downloadable report.

---

## Features

### Public Access (Before Login)

- **Landing Page** — Introduces the platform with feature highlights and a call-to-action for registration or login.
- **User Registration** — Email-based signup with OTP verification (6-digit code, 10-minute TTL) before account activation. Password complexity validation enforced at registration.
- **Secure Login / Logout** — Role-aware login that redirects admin users to the admin dashboard and regular users to the video dashboard.
- **Password Reset Flow** — Full multi-step email-link password recovery with HTML-formatted SMTP email.

### Registered User Features (After Login)

- **Video Upload (MP4)** — Direct file upload with a server-side limit of 100 MB. Large files are written to disk via Django's `TemporaryFileUploadHandler` before processing.
- **YouTube Link Ingestion** — Enter any YouTube URL to trigger `yt-dlp` stream download in the best available MP4 quality, automatically stored to the media directory.
- **Gemini Multimodal Analysis** — Automated pipeline producing: full speech transcript, 3–5 sentence professional summary, five semantic keywords, detected visual objects, activity type classification, and timestamped key moments.
- **Local ML Video Analysis** — Four independent ML models run on sampled video frames:
  - **Human Presence Detection** (YOLOv8): Presence ratio, present/absent seconds, per-frame timeline.
  - **Face Detection** (OpenCV Haar Cascade): Face-visible ratio, average faces per frame, peak simultaneous face count, face visibility in seconds.
  - **Activity Recognition** (YOLOv8 + multi-signal): Eight-category classification with per-frame scoring, dominant activity, percentage distribution, and temporal segments.
  - **Engagement Trend** (KeyBERT + motion signals): Per-segment engagement scores (0–100), overall score, trend classification (Increasing, Decreasing, Stable, Mixed), and peak identification.
- **Interactive Insight Dashboard** — Per-video detail page with all Gemini outputs, ML metrics, engagement wave chart, human presence timeline, face detection stats, and activity breakdown visualization.
- **Analytics Hub** — Cross-video aggregated dashboard showing activity distribution (presentation vs. discussion modes) and an averaged engagement wave across all processed videos.
- **AI Chatbot** — Natural language Q&A interface powered by Claude (via OpenRouter). The chatbot receives the full video analysis data as system context and answers questions about transcript content, detected objects, activities, and more.
- **Chat History** — View previous chatbot conversations organized by video, with full message logs and timestamps.
- **PDF Report Export** — One-click download of a fully formatted A4 PDF report covering all seven insight sections including chatbot conversation history.
- **Video Library** — Paginated list of all uploaded videos with status indicators (`ai_processed`, `ml_processed`, `ai_failed`) and quick-action links.
- **Video Deletion** — Removes the database record and cleans up the media file from disk.
- **User Profile** — Update account display name and view account metadata.

### Administrator Features

- **Admin Dashboard** — Global overview of total users, total processed videos, total insights generated, and upload source breakdown (MP4 vs. YouTube percentage).
- **User Management** — Full list of all registered users with video counts, join dates, and actions to activate/deactivate accounts or permanently delete user identities.
- **AI Pipeline Health Monitor** — Status panel showing the operational state of the vision, speech, and analytics pipeline components with live system logs.
- **Global Analytics** — Platform-wide engagement trend chart averaged across all user videos, and video source distribution breakdown.
- **Admin Video Library** — Administrators have full visibility into all videos uploaded by any user on the platform.

---

## Technologies Used

### Programming Languages

- Python 3.10+

### Libraries & Frameworks

- **Django 5.2.10** — Core web framework, ORM, authentication, admin panel, URL routing
- **django-crispy-forms 2.5 + crispy-bootstrap5** — Enhanced, Bootstrap 5-integrated form rendering
- **Pillow 12.1.0** — Image field handling and media processing
- **python-dotenv 1.2.1** — Environment variable management via `.env` files
- **ReportLab 4.4.9** — Programmatic PDF generation for structured insight reports
- **qrcode 8.0** — QR code generation utilities
- **requests 2.32.5** — HTTP client for OpenRouter API calls

### Machine Learning / AI

- **Google Gemini Flash** (`google-genai 1.61.0`, `google-generativeai 0.8.5`) — Multimodal video analysis via the Gemini Files API for transcription, summarization, keyword extraction, object detection, key moment identification, and activity classification
- **Ultralytics YOLOv8 Nano** (`yolov8n.pt`) — COCO-pretrained real-time object detection for human presence identification and multi-signal activity recognition across video frames
- **OpenCV** (`opencv-python`) — Haar Cascade face detection with histogram equalization on grayscale video frames; optical flow motion magnitude computation for engagement scoring
- **KeyBERT** — BERT-based keyword extraction for transcript segment scoring used in engagement trend analysis
- **sentence-transformers** — Underlying BERT embedding model required by KeyBERT
- **scikit-learn 1.7.2 / NumPy 1.26.4 / SciPy 1.13.1** — Scientific computing stack supporting ML operations, frame index sampling (`np.linspace`), and statistical computations
- **pandas 2.2.2** — Data manipulation and tabular processing
- **Claude (Sonnet) via OpenRouter** — Conversational AI chatbot with video analysis context injection, accessed through the OpenRouter proxy API

### Backend

- Django ORM with SQLite (development) — Relational data management for users, video analyses, and chat messages
- Custom `AbstractUser` with email-based authentication and role-based (`admin`/`user`) access control
- `yt-dlp` — YouTube video stream downloading and format selection
- Django Email Backend (SMTP via Gmail) — OTP delivery and password reset

### Frontend

- Django Template Language — Server-side HTML rendering with base template inheritance (`base1.html`, `base2.html`)
- Tailwind CSS (utility classes in templates) — Responsive, utility-first UI layout and styling
- HTML5 / CSS3 / JavaScript — Client-side interaction for chatbot messaging, chart rendering, and dynamic UI updates

### Database

- SQLite 3 — Default relational database (suitable for development; portable to PostgreSQL for production)

### Deployment & DevOps

- Django `FILE_UPLOAD_HANDLERS` configured with both `MemoryFileUploadHandler` and `TemporaryFileUploadHandler` for robust large-file management
- `DATA_UPLOAD_MAX_MEMORY_SIZE` and `FILE_UPLOAD_MAX_MEMORY_SIZE` set to 100 MB for video upload support
- WhiteNoise-compatible static file configuration with `STATIC_ROOT` and `collectstatic` support

### Security

- Django CSRF middleware on all state-changing endpoints
- Email OTP verification with 10-minute expiry before account activation
- Role-based access control via `@login_required` and `@user_passes_test(is_admin)` decorators
- Django password validators enforcing complexity at registration
- `.env`-based secret and API key management — excluded from source control via `.gitignore`
- Session-based user authentication with server-side session storage

---

## Dataset

This platform does not use a fixed pre-compiled dataset. All analytical data is dynamically generated from user-provided video content. The following data entities are managed:

**VideoAnalysis** — Per-video record storing: user reference, title, uploaded file path (`videos/`), optional YouTube URL, Gemini outputs (`transcript`, `summary`, `keywords` JSONField, `objects_detected` JSONField, `key_moments` JSONField, `activity_type`), local ML outputs (`human_presence_data` JSONField, `face_detection_data` JSONField, `activity_ml_data` JSONField, `engagement_data` JSONField), and processing state flags (`ai_processed`, `ai_failed`, `ml_processed`).

**ChatMessage** — Per-conversation turn storing: user reference, video reference, user question text, AI response text, and creation timestamp.

**Frame Sampling** — The ML pipeline uniformly samples up to 40 frames from each video using `np.linspace` across the total frame count, ensuring even temporal coverage regardless of video duration. Each sampled frame carries its frame index, timestamp in seconds, and the decoded BGR image matrix.

**YOLOv8 Model** — `yolov8n.pt` is the pretrained YOLOv8 Nano model on the COCO dataset (80 classes). It is used with a confidence threshold of 0.30 for activity recognition and 0.35 for human presence detection. The model is lazy-loaded as a singleton to avoid repeated disk reads.

**OpenCV Haar Cascade** — `haarcascade_frontalface_default.xml` (bundled with OpenCV) applied with `scaleFactor=1.05`, `minNeighbors=4`, and `minSize=(20, 20)` after histogram equalization for improved detection under variable lighting conditions.

---

## System Architecture

The application follows a modular Django project structure with four registered application modules and a core configuration package.

```
core/                    # Django project root — settings, root URLconf, landing view, WSGI/ASGI
accounts/                # Custom User model (email auth + role), OTP, login/signup/logout, password reset
videos/                  # VideoAnalysis model, video upload, YouTube ingestion, dashboard, PDF export
    ml_analyzer.py       # Local ML pipeline: YOLOv8, OpenCV, KeyBERT — all four feature analyzers
    utils.py             # Gemini Files API upload + structured prompt analysis; yt-dlp downloader
analysis/                # Insight detail view, analytics hub, admin governance views
chatbot/                 # AI chatbot view, OpenRouter/Claude API integration, chat history
templates/               # Shared and per-app HTML templates (base templates, partials, admin panels)
static/                  # Custom JavaScript (insight_analytics.js)
media/videos/            # User-uploaded and yt-dlp-downloaded video files
yolov8n.pt               # Pre-trained YOLOv8 Nano model weights (COCO, included in repository)
```

**Data Flow:**

1. A user registers, verifies their email via OTP, and logs in. Role-aware routing sends admin users to the admin dashboard and regular users to the video dashboard.
2. The user submits a video via direct upload or YouTube URL. If YouTube, `yt-dlp` downloads the stream to a temporary path, which is then saved via Django's file storage API.
3. **Stage 1 — Gemini Analysis**: The saved video file path is passed to `analyze_video_with_gemini()`. The file is uploaded to the Gemini Files API, polled until `ACTIVE`, then submitted with a structured JSON-enforcing prompt to `gemini-3-flash-preview`. The parsed JSON fields are saved to `VideoAnalysis` with `ai_processed=True`.
4. **Stage 2 — Local ML Analysis**: `run_ml_analysis()` in `ml_analyzer.py` samples 40 frames using OpenCV's `VideoCapture`, runs all four ML analyzers (human presence, face detection, activity recognition, engagement trend), and stores the resulting dictionaries in the corresponding JSONFields with `ml_processed=True`.
5. The user navigates to the insight detail page (served by `analysis/views.py`), which aggregates all stored data and renders it alongside visual charts.
6. The user interacts with the chatbot. Each message POSTs to `chat_api`, which constructs a system prompt embedding all video analysis data and calls the OpenRouter API targeting the Claude model. Responses are saved as `ChatMessage` records.
7. The user downloads a PDF report, which calls `download_pdf_report()` in `videos/views.py` to build a ReportLab document from all stored data and the chat history, streamed as an HTTP attachment response.

---

## Model Workflow

### Step 1 — Frame Sampling

`sample_frames(video_path, max_frames=40)` opens the video with `cv2.VideoCapture`, reads `CAP_PROP_FRAME_COUNT` and `CAP_PROP_FPS`, computes `duration = total / fps`, then selects up to 40 frame indices via `np.linspace(0, total-1, count, dtype=int)`. Each selected frame is decoded and stored as a tuple of `(frame_index, timestamp_seconds, BGR_image_array)`.

### Step 2 — Human Presence Detection (YOLOv8)

`detect_human_presence(frames, duration)` runs the YOLOv8 Nano model on every sampled frame with `classes=[0]` (person only) and `conf=0.35`. For each frame, it records whether at least one person is present and the person count. The output includes presence ratio, present/absent seconds, total duration, and a per-frame presence timeline.

### Step 3 — Face Detection (OpenCV Haar Cascade)

`detect_faces(frames, duration)` converts each frame to grayscale, applies `cv2.equalizeHist` to improve contrast invariance, then runs the Haar Cascade classifier with `scaleFactor=1.05`, `minNeighbors=4`, and `minSize=(20, 20)`. Outputs include the face-detected frame ratio, average faces per frame, peak simultaneous face count, total face-visible seconds, and a per-frame face count timeline.

### Step 4 — Activity Recognition (Multi-Signal Scoring)

`recognize_activity(frames, duration)` processes each frame through YOLOv8 (for detected COCO object classes and person bounding boxes) and the Haar Cascade (for face count). It computes inter-frame motion magnitude using `cv2.absdiff` on consecutive grayscale frames. The `_score_activity()` function maps 13 COCO object class flags, person count, face count, motion level, frame dimensions, and person bounding box area coverage to additive scores across eight activity categories. The highest-scoring category labels each frame. The final output includes dominant activity, percentage distribution across all categories, temporal activity segments, and a text summary.

### Step 5 — Engagement Trend (KeyBERT + Motion)

`analyze_engagement(transcript, frames, duration)` divides the video into 6–16 segments based on duration using `max(6, min(16, int(duration / 30)))`. For each segment, it extracts a text score using KeyBERT on the corresponding transcript chunk (average relevance score of top-5 keyphrases × 100) and a visual score from mean inter-frame motion magnitude normalized to 0–100. The combined segment score is `text_score × 0.6 + visual_score × 0.4`, capped at 100. Trend classification compares the mean of the first and second halves: a difference of more than 8 points yields Increasing or Decreasing; standard deviation above 20 yields Mixed; otherwise the trend is Stable.

### Step 6 — Gemini Multimodal Analysis

`analyze_video_with_gemini(video_path)` uploads the video file to the Gemini Files API using `client.files.upload()`, polls with `client.files.get()` until the file state is `ACTIVE`, then submits it with a strictly formatted JSON-enforcing prompt to `gemini-3-flash-preview`. The prompt mandates six output keys: `transcript`, `summary`, `keywords`, `objects_detected`, `activity_type`, and `key_moments`. The response text is cleaned of Markdown fences and parsed with `json.loads`. After extraction, the uploaded file is deleted from Gemini's servers via `client.files.delete()`.

### Step 7 — Chatbot Integration (Claude via OpenRouter)

When a user submits a message, `chat_api()` checks whether the linked video has a valid summary. If analyzed, it constructs a detailed system prompt embedding the video title, summary, full transcript, detected objects, keywords, and activity type. The message is sent to the OpenRouter API targeting `anthropic/claude-4.5-sonnet` with `max_tokens=500`. The response is extracted from `choices[0].message.content` and persisted as a `ChatMessage` record.

### Step 8 — PDF Report Generation (ReportLab)

`download_pdf_report()` uses ReportLab's `SimpleDocTemplate` with seven custom `ParagraphStyle` objects to build a structured A4 PDF: (1) Summary, (2) Keywords & Topics, (3) Objects Detected, (4) Key Moments with timestamps, (5) Full Transcript in 1000-character monospace blocks, (6) ML Model Analysis summarizing all four local models, and (7) AI Chatbot Conversation with Q&A pairs and timestamps. The buffer is streamed as an HTTP response with `Content-Disposition: attachment`.

---

## Implementation Details

**Custom User Model** — `accounts.User` extends `AbstractUser`, replacing `username` with a unique `email` field. A `role` CharField with choices `admin` and `user` enables role-based routing. `is_verified` and `login_count` are tracked per user. The `CustomUserManager` automatically sets `role='admin'` for superusers.

**OTP Verification** — `EmailOTP` stores a 6-digit random integer with an `auto_now_add` timestamp. The `is_expired()` method checks whether `timezone.now() > created_at + timedelta(minutes=10)`. Users with `is_active=False` are redirected to OTP entry on login attempt rather than receiving a generic credentials error.

**Dual Video Ingestion** — The `upload_video` view handles both file uploads and YouTube URLs in a single POST handler. For YouTube, `download_youtube_video()` constructs a UUID-named output path, uses `yt-dlp` with `format='best[ext=mp4]'` and a 30-second socket timeout, then saves the downloaded file via Django's `File` wrapper before removing the temporary download path.

**ML Model Singleton Pattern** — Both the YOLOv8 model and the Haar Cascade classifier are initialized as module-level `None` globals and populated lazily on first use via `_get_yolo()` and `_get_face_cascade()`. This prevents repeated disk reads and model initialization overhead across multiple view calls within a single server process.

**Multi-Signal Activity Scoring** — The `_score_activity()` function maps 13 COCO object class presence flags (monitor, laptop, keyboard, mouse, chair, couch, book, phone, table, bottle, sports equipment) and four derived boolean signals (multi-person, large-person where any bounding box exceeds 8% of frame area, high-motion above 40 mean-abs-diff, low-motion below 15) to weighted additive scores for eight activity categories. Any category with a final score below 15 is labeled "General Activity" to avoid spurious classifications.

**Engagement Segment Normalization** — When aggregating engagement scores across multiple videos in the Analytics Hub, all per-video score lists are resampled or padded to a common length (16 or 20 bars depending on the view) before column-wise averaging. This ensures the resulting wave chart is meaningful even when individual videos have different durations and thus different segment counts.

**PDF Style System** — Seven named `ParagraphStyle` objects define the visual hierarchy: `ReportTitle` (22pt, centered, dark blue), `SubTitle` (11pt, centered, gray), `SectionHeading` (13pt, white on dark blue background), `Body` (10pt, justified), `Mono` (9pt Courier, light gray background for transcript), `UserMsg` (bold, 10pt, dark blue), and `AIMsg` (10pt, light blue background for AI responses).

**Multi-Language Support** — Django internationalization is configured with English, Hindi, and Telugu language options. `LANGUAGE_COOKIE_AGE` is set to 31,536,000 seconds (one year), and `LOCALE_PATHS` points to a `locale/` directory for translation files.

**Admin Governance** — All admin views are protected with `@user_passes_test(is_admin)`, where `is_admin` checks both `is_authenticated` and `role == 'admin'`. The `admin_user_management` view annotates each user with their video count via Django's `Count('videoanalysis')` annotation. Toggle and delete actions explicitly guard against modifying superuser accounts.

**File Upload Configuration** — `DATA_UPLOAD_MAX_MEMORY_SIZE` and `FILE_UPLOAD_MAX_MEMORY_SIZE` are both set to 104,857,600 bytes (100 MB). Both `MemoryFileUploadHandler` and `TemporaryFileUploadHandler` are registered to ensure large video files are buffered to disk during upload rather than consuming server RAM.

---

## Results

The platform delivers the following analytical outputs and performance characteristics based on its design and integrated models:

| Feature | Method | Key Output Metrics |
|---|---|---|
| Speech Transcription | Gemini Flash (multimodal) | Full verbatim transcript |
| Video Summarization | Gemini Flash | 3–5 sentence professional summary |
| Keyword Extraction | Gemini Flash | 5 semantic keyword tags |
| Object Detection (Gemini) | Gemini Flash (visual) | Named list of detected objects/people |
| Key Moments | Gemini Flash | Timestamped event labels (time + label) |
| Activity Classification (Gemini) | Gemini Flash | Single activity label |
| Human Presence Detection | YOLOv8 Nano (conf ≥ 0.35, class 0) | Presence ratio, timeline, present/absent seconds |
| Face Detection | OpenCV Haar Cascade (equalized) | Face-visible ratio, avg faces/frame, peak face count |
| Activity Recognition (ML) | YOLOv8 + 13-signal scorer | 8-category distribution, dominant activity, temporal segments |
| Engagement Trend | KeyBERT (0.6 weight) + motion (0.4 weight) | Per-segment score (0–100), overall score, trend label |
| Frame Sampling | NumPy `linspace` | Up to 40 uniformly distributed temporal frames |
| PDF Report | ReportLab A4 | 7-section formatted document with chat history |
| Chatbot Context | Claude via OpenRouter | Title + summary + transcript + objects + keywords + activity |

**Activity Categories Recognized:** Lecture/Teaching, Presentation, Meeting/Discussion, Tutorial/Demo, Interview, Screen Recording, Physical Activity, Idle/No Activity, General Activity.

**Engagement Score Labels:** High (≥75), Moderate (≥50), Low (≥25), Very Low (<25).

**Engagement Trend Classification:** Increasing (second half mean > first half mean by >8), Decreasing (first half mean > second half mean by >8), Mixed (standard deviation >20), Stable (all other cases).

---

## Installation

### Prerequisites

- Python 3.10 or higher
- pip
- Git
- FFmpeg (required by `yt-dlp` for YouTube stream processing)
- A valid **Google Gemini API key** (`google-genai` / `google-generativeai`)
- A valid **OpenRouter API key** (for Claude chatbot access)
- An SMTP-capable email account (e.g., Gmail with an App Password)

### Step 1 — Clone the Repository

```bash
git clone https://github.com/harisaigithub/Video-Insights-Detection.git
cd Video-Insights-Detection
```

### Step 2 — Create and Activate a Virtual Environment

```bash
python -m venv venv

# On Windows
venv\Scripts\activate

# On macOS / Linux
source venv/bin/activate
```

### Step 3 — Install Dependencies

```bash
pip install -r re.txt
```

Key packages installed: `Django`, `google-genai`, `google-generativeai`, `ultralytics`, `opencv-python`, `keybert`, `sentence-transformers`, `scikit-learn`, `pandas`, `numpy`, `scipy`, `reportlab`, `Pillow`, `qrcode`, `python-dotenv`, `django-crispy-forms`, `crispy-bootstrap5`, `requests`.

> **Note:** Installing `ultralytics` will automatically fetch required YOLOv8 dependencies. The `yolov8n.pt` model weights file is included in the repository root and will be loaded by the ML analyzer on first use.

### Step 4 — Configure Environment Variables

Create a `.env` file in the project root:

```env
SECRET_KEY=your_django_secret_key_here
GEMINI_API_KEY=your_google_gemini_api_key_here
OPENROUTER_API_KEY=your_openrouter_api_key_here
EMAIL_HOST_USER=your_email@gmail.com
EMAIL_HOST_PASSWORD=your_gmail_app_password_here
```

> **Important:** Never commit the `.env` file to version control. The `.gitignore` file is pre-configured to exclude it.

### Step 5 — Apply Database Migrations

```bash
python manage.py makemigrations
python manage.py migrate
```

### Step 6 — Create a Superuser (Administrator Account)

```bash
python manage.py createsuperuser
```

Enter an email and password when prompted. The superuser is automatically assigned `role='admin'`.

### Step 7 — Collect Static Files

```bash
python manage.py collectstatic
```

### Step 8 — Run the Development Server

```bash
python manage.py runserver
```

The application will be accessible at `http://127.0.0.1:8000/`.

---

## Usage

### Public Access

Navigate to `http://127.0.0.1:8000/` to view the landing page. Register at `/accounts/signup/`, verify your email with the 6-digit OTP, then log in at `/accounts/login/`.

### Uploading and Analyzing a Video

1. After login, navigate to `/portal/upload/` to access the video upload form.
2. Enter a title and either attach an MP4 file (up to 100 MB) or paste a YouTube URL.
3. Submit the form. The system will automatically run both the Gemini analysis pipeline and all four local ML models.
4. You will be redirected to your video library at `/portal/library/` upon completion.

### Viewing Insights

Navigate to `/analysis/insights/<video_id>/` to open the full insight dashboard for a specific video. This page displays the Gemini-generated transcript, summary, keywords, detected objects, key moments, and all local ML results including engagement wave, human presence timeline, face detection statistics, and activity distribution.

### Using the Chatbot

Navigate to `/neural-chat/` to open the AI chatbot interface. Select a video from the dropdown (defaults to the most recent). Type questions about the video content — the Claude model responds using the full analysis context. Access previous conversations at `/neural-chat/history/`.

### Downloading a PDF Report

From the video library or insight detail page, click the Download Report link to generate and download a structured PDF at `/portal/video/<video_id>/download-report/`.

### Analytics Hub

Navigate to `/analysis/hub/` to view the aggregated analytics dashboard combining engagement scores and activity distributions across all your processed videos.

### Administrator Access

Log in with an admin account to be automatically redirected to `/analysis/admin/dashboard/`. Access user management at `/analysis/admin/users/`, global analytics at `/analysis/admin/analytics/`, and AI pipeline health at `/analysis/admin/monitor/`.

---

## Future Enhancements

- **Real-Time Processing Progress** — Implement WebSocket-based push notifications using Django Channels to stream Gemini upload progress and ML analysis status to the user interface, replacing the current synchronous blocking approach.
- **PostgreSQL Migration** — Replace SQLite with PostgreSQL for production-grade concurrent access, improved query performance, and native JSONField indexing on analytical data at scale.
- **Speaker Diarization** — Integrate a speaker diarization model (e.g., pyannote.audio) to attribute transcript segments to individual speakers, enabling multi-speaker conversation analysis and per-speaker engagement scoring.
- **Emotion Recognition** — Add a facial expression classification model (e.g., DeepFace) to the ML pipeline, producing per-segment emotion distribution alongside existing face detection data.
- **Batch Video Processing** — Enable multiple video uploads with a queue-based background processing system using Celery and Redis, decoupling long-running ML jobs from the HTTP request-response cycle.
- **Video Clip Export** — Allow users to export specific key moments as short MP4 clips using `ffmpeg-python`, enabling quick identification and extraction of highlight segments.
- **Multilingual Transcription** — Extend the Gemini prompt to request transcription in the detected spoken language with automatic translation to English for non-English content.
- **Embeddings-Based Video Search** — Generate BERT embeddings for each video summary to enable semantic search across the video library rather than simple text matching.
- **QR Code Report Sharing** — The `qrcode` library is already installed; integrate it to generate shareable QR codes linking to each video's public insight summary page.
- **Engagement Benchmarking** — Compare individual video engagement scores against anonymized platform-wide averages by activity category, providing contextual performance benchmarks for content creators.

---

## Contributing

Contributions are welcome. Please follow the standard Git workflow below.

### Steps to Contribute

1. **Fork** the repository to your GitHub account.

2. **Clone** your fork locally:
   ```bash
   git clone https://github.com/your-username/Video-Insights-Detection.git
   cd Video-Insights-Detection
   ```

3. **Create a feature branch** with a descriptive name:
   ```bash
   git checkout -b feature/your-feature-name
   ```

4. **Make your changes** following the existing code structure and style.

5. **Commit** with a clear, descriptive message:
   ```bash
   git add .
   git commit -m "feat: describe your change clearly"
   ```

6. **Push** to your fork:
   ```bash
   git push origin feature/your-feature-name
   ```

7. **Open a Pull Request** against the `main` branch of the original repository with a clear description of the problem solved and the approach taken.

### Guidelines

- All new views must be protected with `@login_required` or `@user_passes_test(is_admin)` as appropriate.
- Do not commit `.env` files, `db.sqlite3`, `media/` directories, or `__pycache__` folders.
- Write docstrings for all new view functions and ML utility methods.
- Test the full upload → analyze → chatbot → PDF export flow before opening a pull request.
- Avoid introducing heavyweight blocking operations directly in view functions — document any processing that should be moved to a background task queue.

---

## License

This project is licensed under the **MIT License**.

```
MIT License

Copyright (c) 2024 Harisai Parasa

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

For technical questions, issue reporting, collaboration inquiries, or deployment support, please use the contact information below.

**GitHub:** [https://github.com/harisaigithub](https://github.com/harisaigithub)

**Email:** [harisaiparasa@gmail.com](mailto:harisaiparasa@gmail.com)

---


&nbsp;

&nbsp;

## ***To acquire this project, please contact***

**GitHub:** https://github.com/harisaigithub
**Email:** [harisaiparasa@gmail.com](mailto:harisaiparasa@gmail.com)

For collaborations, customizations, or deployment support, feel free to reach out.