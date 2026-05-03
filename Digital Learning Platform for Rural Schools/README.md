# Digital Learning Platform for Rural Schools

> **Bridging the Educational Divide — A Gamified, AI-Powered E-Learning Ecosystem Designed for Rural Learners**

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

Rural educational institutions across India face a persistent challenge: limited access to quality teaching resources, qualified instructors, and interactive learning tools. Traditional classroom environments in underserved regions frequently lack the infrastructure required to deliver modern, personalized education. The **Digital Learning Platform for Rural Schools** is a full-stack web application engineered to close this gap.

Built on Django 5.2 and powered by a suite of AI technologies — including Perplexity AI for generative feedback, a Bidirectional LSTM (BiLSTM) model for sentiment classification, and NLP-driven question generation — the platform delivers an engaging, multi-role educational experience accessible from any device with a browser.

The system supports three distinct user roles (Student, Instructor, and Administrator), each with purpose-built dashboards and workflows. Students progress through gamified learning modules — ranging from Listen & Spell challenges to collaborative storytelling — while instructors manage course content, assessments, and student performance. Administrators oversee platform-wide operations including instructor approvals, challenge management, and feedback analytics.

The platform is multilingual (English, Hindi, Telugu), OTP-authenticated, and equipped with real-time AI feedback mechanisms, making it particularly suited for diverse and linguistically varied rural school populations.

---

## Objectives

- Design and deploy a scalable, multi-role digital learning platform tailored to the constraints and context of rural schools.
- Integrate AI-generated question prompts and contextual feedback via the Perplexity API to enable adaptive, self-directed learning.
- Implement a BiLSTM-based sentiment analysis model trained on Amazon review data to automatically classify student feedback as positive, neutral, or negative.
- Gamify foundational literacy and communication skills through structured challenges — Listen & Spell, Story Weave, Leadership Challenges, and Reflection modules.
- Enable instructors to create and manage multi-format assessments (MCQ, descriptive, and image-based questions) with full grade-level segmentation.
- Provide administrators with real-time dashboards, visual feedback analytics, and comprehensive user management controls.
- Support multilingual access in English, Hindi, and Telugu using Django's internationalization (i18n) framework.
- Ensure secure authentication through OTP-based email verification for both students and instructors.

---

## Summary

The Digital Learning Platform for Rural Schools is a comprehensive, Django-based educational web application that unifies course management, gamified skill-building, AI-powered feedback, and NLP-driven sentiment analysis into a single cohesive platform.

At its core, the system hosts three inter-connected applications — `userapp`, `instructorapp`, and `adminapp` — each serving a distinct stakeholder. Students enroll in courses, take multi-format tests, participate in communication challenges (Listen & Spell), engage in solo and collaborative storytelling through StoryWeave, tackle leadership-based open-ended challenges evaluated by the Perplexity API, complete time-bound tasks, and submit self-reflections that receive AI-generated analytical feedback.

Instructors manage course content (with video URL support), author questions across three difficulty levels and three question formats, assign grade-level tasks with deadlines, monitor student performance, and respond to feedback. Administrators approve instructors, manage the platform's challenge bank, review aggregated feedback with sentiment visualizations, and maintain overall user integrity.

A pre-trained Bidirectional LSTM model, trained on over 28,000 Amazon product reviews, is embedded directly into the Django view layer to classify student-submitted platform feedback in real time. This model achieved an overall accuracy of 92.55% with a weighted F1 score of 0.9055, enabling meaningful automated sentiment tracking for continuous platform improvement.

---

## Features

### Public / Pre-Login

- Unified login and role-based routing for Students, Instructors, and Administrators.
- Separate registration flows for students and instructors with OTP-based email verification.
- Contact page for platform inquiries.
- Multilingual interface support (English, Hindi, Telugu) with persistent language cookies.

### Student (Post-Login)

- **Dashboard**: Personalized overview of enrolled courses, pending tasks, and progress.
- **Course Enrollment**: Browse and enroll in instructor-published courses with video content.
- **Multi-Format Online Tests**: MCQ, descriptive, and image-based question formats with difficulty tiers (Easy, Medium, Hard); instant result computation and detailed test review.
- **Listen & Spell (Communication Challenge)**: A three-level audio-based spelling game — Word, Sentence, and Paragraph — with level progression, incorrect attempt tracking, and completion state management.
- **StoryWeave Module**: Three storytelling modes — Solo (AI prompt, student writes, AI evaluates), Collaborative (two students co-author a story), and Challenge (prompt-based competitive mode).
- **Leadership Challenges**: Open-ended challenges categorized by type (Visionary, Technical, Creative, Leadership, Strategic Thinking, etc.); answers are evaluated and rated (out of 10) by the Perplexity AI API with structured feedback.
- **Reflection Module**: AI-driven self-assessment where students choose a learning category, receive a dynamically generated question from the Perplexity API, submit their answer, and receive personalized feedback.
- **Time Game (Task Manager)**: Grade-level task assignments from instructors with due-date tracking; students complete tasks, earning points on successful submission.
- **Feedback Submission**: Platform feedback collected with a star rating system; sentiment (positive, neutral, negative) is automatically inferred by the embedded BiLSTM model.
- **User Profile Management**: View and update personal information and profile photo.

### Instructor (Post-Login)

- **Instructor Dashboard**: Summary of courses, questions, student activity, and feedback.
- **Course Management**: Add, edit, and delete courses with image, category, language, description, video URL, duration, and pricing fields.
- **Question Bank**: Author MCQ, descriptive, and image-based questions per course, per difficulty level.
- **Student Monitoring**: View enrolled students and their test performance; drill into detailed per-test result breakdowns.
- **Feedback Management**: View student course feedback with graphical sentiment distribution; reply to feedback via email; remove resolved feedback entries.
- **Task Assignment**: Create time-bound tasks assigned to specific grade levels (A/B/C/D); view task completion status per student.

### Administrator (Post-Login)

- **Admin Dashboard**: Aggregate statistics across students, instructors, courses, and feedback.
- **Instructor Approval Workflow**: Review pending instructor registrations; accept or delete applications.
- **User Management**: View all students and instructors; remove accounts as necessary.
- **Listen & Spell Content Management**: Add, view, edit, and delete audio-based word/sentence/paragraph challenges across three difficulty levels.
- **Challenge Bank**: Create, manage, and delete leadership challenges with type categorization; view all submitted student answers.
- **Feedback Analytics**: View all submitted student feedback with sentiment labels; visualize feedback distribution via graphs.

---

## Technologies Used

### Programming Languages
- Python 3.10 / 3.13
- HTML5, CSS3, JavaScript

### Libraries & Frameworks
- Django 5.2 (web framework, ORM, i18n, session management, CSRF protection)
- Bootstrap 5 (responsive front-end UI)
- NLTK (tokenization, lemmatization, stopword removal)
- Scikit-learn (LabelEncoder, train-test split, metrics)
- Pandas, NumPy (data manipulation and preprocessing)
- Matplotlib, Seaborn (visualization during model training)
- WordCloud (exploratory data analysis)

### Machine Learning / AI
- **BiLSTM (Bidirectional Long Short-Term Memory)** — TensorFlow/Keras sequential model for sentiment classification
- **Perplexity AI API** (`llama-3.1-sonar-small-128k-online`) — used for story prompt generation, leadership challenge evaluation, and reflection question generation/feedback
- **CatBoost, XGBoost, LightGBM** — evaluated during comparative model analysis
- **MLP Stacking Classifier** — ensemble approach explored during experimentation
- **ALBERT Tokenizer** — included in the NLP research pipeline

### Backend
- Django `userapp`, `instructorapp`, `adminapp` — modular application architecture
- Django ORM with SQLite (development database)
- Django Email Backend via Gmail SMTP (OTP delivery, feedback reply)
- Django Middleware: Security, Sessions, Locale, CSRF, Auth, Messages

### Frontend
- Django Template Engine (Jinja2-style templates)
- Bootstrap 5 with SCSS source files
- Custom CSS and JavaScript per module

### Database
- SQLite 3 (development; `db.sqlite3`)

### Deployment & DevOps
- WSGI-compatible application server ready
- `manage.py` Django management interface
- Static files served via `STATICFILES_DIRS`; media uploads via `MEDIA_ROOT`

### Security
- OTP-based two-factor email verification for student and instructor registration
- CSRF middleware protection on all POST routes
- Django's built-in password validators (length, similarity, common, numeric)
- Session-based authentication for all three user roles
- Clickjacking protection via `XFrameOptionsMiddleware`

---

## Dataset

**Source:** Amazon Product Reviews dataset (`amazon_reviews.csv`)

**Structure:** The dataset contains product reviews with a `text` field (raw review text) and a `rating` field (integer, 1–5). A derived `sentiment` column maps ratings 4–5 to `positive`, rating 3 to `neutral`, and ratings 1–2 to `negative`.

**Size:** Approximately 28,000 records after cleaning; the cleaned version is stored as `reviews_clean.csv`.

**Preprocessing Pipeline:**
1. Removal of null/missing values via `dropna`.
2. Text normalization — removal of non-alphabetic characters using regex, lowercasing.
3. Tokenization and lemmatization using NLTK's `WordNetLemmatizer`.
4. Stopword removal using NLTK's English stopword corpus.
5. Feature extraction using `CountVectorizer` (top 2,500 n-gram features, unigram to trigram) for classical models.
6. Sequence tokenization and padding using Keras `Tokenizer` and `pad_sequences` for deep learning models.
7. Label encoding of the three-class sentiment target using `LabelEncoder` (serialized as `label_encoder.joblib`).

**Usage:** The trained BiLSTM model weights (`bylstm_model_weights.h5`) and architecture (`bylstm_model_architecture.json`) are loaded at Django application startup via `userapp/views.py` and used in the `predict_sentiment()` function to classify live feedback submissions.

---

## System Architecture

The platform follows a modular Django monorepo architecture organized around three primary applications:

```
gameProject/                   ← Django project root (settings, root URLs, WSGI)
├── userapp/                   ← Student-facing features, shared auth, AI integrations
│   ├── models.py              ← User, Feedback, Reflection, GameProgress, Answer,
│   │                             CollaborativeStory, StudentCourses, ResultModel, etc.
│   ├── views.py               ← All student-facing views + Perplexity API + BiLSTM inference
│   ├── urls.py                ← Student URL routing
│   └── templates/             ← HTML templates (dashboard, games, tests, storyweave, etc.)
├── instructorapp/             ← Instructor features
│   ├── models.py              ← InstructorRegModel, TaskModel, Addcourse, Question,
│   │                             DescriptiveQuestion, ImageQuestion
│   ├── views.py               ← Course, question, task, and feedback management
│   ├── urls.py                ← Instructor URL routing
│   └── templates/             ← Instructor HTML templates
├── adminapp/                  ← Administrator features
│   ├── models.py              ← ListenSpellWord, Challenge
│   ├── views.py               ← Admin dashboard, approvals, challenge and feedback management
│   ├── urls.py                ← Admin URL routing
│   └── templates/             ← Admin HTML templates
├── amazone review/            ← NLP research pipeline
│   ├── amazon_reviews.csv     ← Raw training data
│   ├── reviews_clean.csv      ← Preprocessed training data
│   ├── bylstm_model_architecture.json  ← Serialized Keras model structure
│   ├── bylstm_model_weights.h5         ← Trained BiLSTM weights
│   ├── label_encoder.joblib            ← Fitted LabelEncoder
│   ├── albert_tokenizer/              ← ALBERT tokenizer artifacts
│   └── final-review.ipynb             ← Model training and evaluation notebook
├── static/                    ← Global static assets (CSS, JS, SCSS, Bootstrap source)
├── media/                     ← User-uploaded files (profile photos, course images, audio)
├── locale/                    ← Internationalization (.po/.mo files for hi, te)
└── db.sqlite3                 ← Development SQLite database
```

**Data Flow:** A student request arrives at Django's URL router → the appropriate view function in `userapp/views.py` is invoked → database operations are handled via the ORM → if the feature requires AI (leadership, reflection, story), the Perplexity API is called via `requests.post` → if the feature involves feedback submission, the pre-loaded BiLSTM model's `predict_sentiment()` is invoked inline → the result is persisted to the database → a rendered HTML template is returned to the client.

---

## Model Workflow

### Preprocessing
1. Raw Amazon review text is ingested from `amazon_reviews.csv`.
2. Null records are dropped; sentiment labels are derived from the numeric `rating` column.
3. Each review undergoes regex-based cleaning, lowercasing, tokenization, and NLTK lemmatization with stopword removal.
4. For the BiLSTM pipeline, the cleaned corpus is tokenized using Keras `Tokenizer`, converted to integer sequences, and padded to a uniform length.
5. The label target is integer-encoded using `LabelEncoder` and the encoder is serialized for inference.

### Training
1. The dataset is split into training (80%) and test (20%) sets using `train_test_split`.
2. A Keras `Sequential` BiLSTM model is constructed with an `Embedding` layer, a `Bidirectional(LSTM)` layer, a `Dropout` regularization layer, a `Flatten` layer, and a softmax `Dense` output layer for three-class classification.
3. The model is compiled with the `Adam` optimizer and trained for 5 epochs with validation monitoring.
4. Comparative experiments are conducted with CatBoost, XGBoost, LightGBM, and an MLP-based Stacking Classifier ensemble to benchmark performance.

### Evaluation
- Primary evaluation metrics: Accuracy, Precision, Recall, and F1 Score computed via `sklearn.metrics`.
- Classification reports are generated per class (positive, neutral, negative) to surface per-class performance disparities.

### Prediction (Inference)
1. At Django startup, `bylstm_model_architecture.json` and `bylstm_model_weights.h5` are loaded to reconstruct the trained Keras model, and `label_encoder.joblib` is deserialized.
2. When a student submits platform feedback, the `predict_sentiment(text)` function tokenizes and pads the input using the saved tokenizer, runs forward inference on the BiLSTM model, and decodes the predicted class index via the label encoder.
3. The returned label (`positive`, `neutral`, or `negative`) is stored in the `Feedback.sentiment` database field.

### Integration
- The BiLSTM inference pipeline is embedded directly within `userapp/views.py` as a module-level load, ensuring zero-latency model instantiation on first request.
- The Perplexity AI API is called asynchronously within view functions for leadership challenge evaluation, story prompt generation, and reflection question/feedback generation using authenticated HTTPS POST requests.

---

## Implementation Details

**Role Management:** The platform implements session-based role management. On login, the user's role (student, instructor, or admin) is stored in the Django session and checked on every protected view via decorator-equivalent guard clauses. There is no use of Django's built-in `User` model — a custom `User` model in `userapp` and `InstructorRegModel` in `instructorapp` are used instead, allowing full control over profile fields.

**OTP Verification:** A 4-digit OTP is generated on registration and dispatched via Gmail SMTP using Django's email backend. The OTP is stored against the user record and validated on a separate verification page before the account is activated.

**Listen & Spell Game Engine:** The `GameProgress` model tracks a student's current level (`Level1` → `Level2` → `Level3`), incorrect attempt count, completed challenge IDs (via a `ManyToManyField` to `ListenSpellWord`), and current question index. This stateful model enables seamless pause-and-resume across sessions.

**Perplexity AI Integration:** Four distinct API call patterns are implemented — story prompt generation, story feedback evaluation, leadership challenge feedback with numeric rating extraction, and reflection question generation/answer feedback. Each call constructs a role-played system prompt, submits via `requests.post` to `https://api.perplexity.ai/chat/completions`, and parses the structured text response.

**Assessment Engine:** The `process_question` view handles MCQ auto-grading (exact answer matching), descriptive grading (submitted to Perplexity for evaluation), and image question evaluation. Results are persisted in the `ResultModel` table with per-question correctness records, enabling granular test review.

**Grade-Level Task System:** The `TaskModel` supports four grade levels (A, B, C, D). Each student is assigned a grade level at registration; the Time Game view filters visible tasks by the student's grade level and tracks completion via the `TaskCompletion` model with timestamps.

**Multilingual Support:** Django's `USE_I18N = True` configuration, `LocaleMiddleware`, and `.po`/`.mo` locale files under the `locale/` directory enable dynamic language switching between English, Hindi, and Telugu, persisted via a one-year language cookie.

**Feedback Analytics:** The admin feedback graph view aggregates `Feedback` records and renders a visual sentiment distribution using chart libraries embedded in the `graph-feedback.html` template.

---

## Results

### BiLSTM Sentiment Classification Model

| Metric    | Score  |
|-----------|--------|
| Accuracy  | 92.55% |
| Precision | 88.65% |
| Recall    | 92.55% |
| F1 Score  | 90.55% |

**Training Progression (BiLSTM — 5 Epochs):**

| Epoch | Train Accuracy | Val Accuracy | Train Loss | Val Loss |
|-------|---------------|--------------|------------|----------|
| 1     | 88.84%        | 90.60%       | 0.4312     | 0.3280   |
| 2     | 90.77%        | 91.90%       | 0.2779     | 0.2431   |
| 3     | 92.75%        | 92.74%       | 0.2140     | 0.2180   |
| 4     | 93.46%        | 93.18%       | 0.1858     | 0.2152   |
| 5     | 93.89%        | 93.10%       | 0.1702     | 0.2048   |

**Per-Class Classification Report:**

| Class    | Precision | Recall | F1-Score | Support |
|----------|-----------|--------|----------|---------|
| Negative | 0.73      | 0.70   | 0.71     | 325     |
| Neutral  | 0.55      | 0.26   | 0.35     | 254     |
| Positive | 0.96      | 0.98   | 0.97     | 5,088   |

The positive class — which dominates the dataset distribution — achieves near-perfect precision and recall. The neutral class exhibits the expected challenge of imbalanced classification, a known difficulty in three-class sentiment tasks. The overall weighted F1 score of 0.9055 reflects strong production-level performance for the automated feedback classification use case.

**Comparative Model Benchmark (from experimentation notebook):**

| Model                  | Test Accuracy |
|------------------------|---------------|
| BiLSTM (5 epochs)      | 92.55%        |
| BiLSTM (10 epochs)     | ~94.00%       |
| Stacking Ensemble      | ~92.94%       |
| CatBoost + LGBM + XGB  | Benchmarked   |

---

## Installation

### Prerequisites

- Python 3.10 or higher
- pip package manager
- Git
- A Gmail account with an App Password configured for SMTP

### Step 1: Clone the Repository

```bash
git clone https://github.com/harisaigithub/Digital-Learning-Platform-for-Rural-School.git
cd Digital-Learning-Platform-for-Rural-School
```

### Step 2: Create and Activate a Virtual Environment

```bash
python -m venv venv

# On Windows
venv\Scripts\activate

# On macOS/Linux
source venv/bin/activate
```

### Step 3: Install Dependencies

```bash
pip install django pillow requests tensorflow keras scikit-learn pandas numpy nltk joblib catboost xgboost lightgbm
```

Or, if a `requirements.txt` is present:

```bash
pip install -r requirements.txt
```

### Step 4: Configure Environment Variables

Open `gameProject/settings.py` and update the following:

```python
# Email configuration
EMAIL_HOST_USER = 'your-email@gmail.com'
EMAIL_HOST_PASSWORD = 'your-gmail-app-password'

# Perplexity AI API Key
PERPLEXITY_API_KEY = "your-perplexity-api-key"

# Generate a new secret key for production
SECRET_KEY = 'your-new-secret-key'
```

> **Security Note:** For production deployments, store secrets in environment variables or a `.env` file using `python-decouple` or `django-environ`. Never commit API keys to version control.

### Step 5: Apply Migrations

```bash
python manage.py makemigrations
python manage.py migrate
```

### Step 6: Download NLTK Resources

```bash
python -c "import nltk; nltk.download('stopwords'); nltk.download('wordnet')"
```

### Step 7: Collect Static Files (Production)

```bash
python manage.py collectstatic
```

### Step 8: Run the Development Server

```bash
python manage.py runserver
```

The application will be accessible at `http://127.0.0.1:8000/`.

---

## Usage

### Student Workflow

1. Navigate to `http://127.0.0.1:8000/signup/` and select the Student role.
2. Complete the registration form; an OTP will be dispatched to the provided email.
3. Verify the OTP at `/otp/` to activate the account.
4. Log in via `/login/` and access the student dashboard at `/dashboard/`.
5. Browse and enroll in available courses under `student/my-courses/`.
6. Take tests at `student/test/<course_id>/` and review results at `student/test-result/`.
7. Access gamified modules from the dashboard: Listen & Spell (`/communication/challenge/`), StoryWeave (`/choose/`), Leadership Challenges (`/leadership/`), and Reflection (`/start-reflection/`).
8. Complete grade-level tasks via the Time Game (`/time-game/`) before their due dates.
9. Submit platform feedback at `/feedback/`.

### Instructor Workflow

1. Register at `/instructor-register/`; the account enters a Pending state awaiting admin approval.
2. After approval, log in at `/instructor-login/` and access the instructor dashboard at `/instructor/dashboard/`.
3. Add courses at `/instructor/add-courses/` and author questions (MCQ at `/instructor/add-mcqs-questions/`, descriptive at `/instructor/add-descriptive-questions/`, image-based at `/instructor/add-image-questions/`).
4. Assign grade-level tasks at `/time/task/add/` and monitor completion at `/time/students/completed/`.
5. Review student test performance at `/instructor/test-results/`.

### Administrator Workflow

1. Log in via the unified login page with admin credentials.
2. Access the admin dashboard at `/dashboard/`.
3. Approve or reject pending instructors at `/courses-admin/pending-instructors/`.
4. Manage Listen & Spell content at `/listen_spell/add/` and `/listen_spell/list/`.
5. Add and manage leadership challenges at `/add_challenge/` and `/manage_challenges/`.
6. Review student feedback with sentiment labels at `/courses-admin/view-feedbacks/` and visualize distribution at `/courses-admin/feedback-graph/`.

---

## Future Enhancements

- **Recommendation Engine:** Integrate a collaborative filtering or content-based recommendation model to suggest courses and challenges based on individual student performance history.
- **Real-Time Collaboration:** Replace the current collaborative storytelling implementation with WebSocket-based real-time co-editing using Django Channels.
- **Offline-First PWA:** Convert the platform into a Progressive Web Application (PWA) with service workers and IndexedDB caching to support low-connectivity rural environments.
- **PostgreSQL Migration:** Replace the SQLite development database with PostgreSQL for production-grade scalability and concurrent access support.
- **Adaptive Difficulty:** Implement an adaptive question-selection algorithm that dynamically adjusts test difficulty based on a student's historical performance across difficulty tiers.
- **Voice-to-Text Input:** Integrate browser-based speech recognition (Web Speech API) for the Listen & Spell module, enabling hands-free answer submission for enhanced accessibility.
- **Extended Language Support:** Expand the multilingual framework to include additional Indian regional languages (e.g., Tamil, Kannada, Bengali, Marathi) to serve a broader rural demographic.
- **Mobile Application:** Develop a companion Android/iOS application using Flutter or React Native, leveraging the Django REST Framework as the backend API layer.
- **Analytics Dashboard for Instructors:** Build a rich data visualization layer using Chart.js or Plotly to provide instructors with per-question difficulty analytics, student performance heatmaps, and drop-off analysis.
- **Improved Sentiment Model:** Address class imbalance in the sentiment classifier through oversampling techniques (SMOTE) or class-weighted training to improve neutral-class F1 score.

---

## Contributing

Contributions are welcome. Please follow the standard GitHub pull request workflow:

1. **Fork** the repository to your GitHub account.
2. **Clone** your forked repository locally:
   ```bash
   git clone https://github.com/<your-username>/Digital-Learning-Platform-for-Rural-School.git
   ```
3. **Create a feature branch** from `main`:
   ```bash
   git checkout -b feature/your-feature-name
   ```
4. **Implement** your changes, ensuring code clarity and consistency with the existing codebase.
5. **Commit** with a descriptive message:
   ```bash
   git commit -m "feat: add adaptive difficulty question selection"
   ```
6. **Push** your branch to GitHub:
   ```bash
   git push origin feature/your-feature-name
   ```
7. **Open a Pull Request** against the `main` branch of the original repository, describing the changes made and their purpose.

Please ensure your contributions do not break existing functionality and that any new AI integrations include appropriate error handling for API failures.

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