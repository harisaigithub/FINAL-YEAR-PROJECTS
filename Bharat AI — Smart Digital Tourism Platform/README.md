# Bharat AI — Smart Digital Tourism Platform

> **An AI-Enabled Full-Stack Tourism Portal for Intelligent Trip Planning, Sentiment-Driven Discovery, and National Destination Management**

[![Python](https://img.shields.io/badge/Python-3.11+-blue.svg)](https://www.python.org/)
[![Django](https://img.shields.io/badge/Django-5.2-green.svg)](https://www.djangoproject.com/)
[![Gemini AI](https://img.shields.io/badge/Gemini-2.5_Flash-orange.svg)](https://ai.google.dev/)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

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

India's tourism sector encompasses thousands of destinations spanning diverse cultural, ecological, and heritage categories across 28 states and 8 Union Territories. Despite this richness, tourists frequently rely on fragmented, static information sources that lack personalization, real-time reviews, or intelligent trip planning. Local guides and homestay operators remain largely undiscoverable through digital channels, and administrative management of tourism infrastructure is often disconnected from traveller feedback.

**Bharat AI** addresses these gaps with a full-stack, AI-powered digital tourism platform that integrates three distinct intelligence layers: automated sentiment analysis of tourist reviews using TextBlob-based Natural Language Processing, AI-generated day-wise travel itineraries using Google Gemini 2.5 Flash, and an AI-powered conversational travel assistant (Bharat AI Chatbot) capable of processing both text queries and image-based landmark identification.

The platform serves tourists seeking curated, AI-ranked destination discovery alongside verified local services, and administrators requiring a centralized governance interface for managing India's national destination catalogue, registered service providers, and user moderation.

---

## Objectives

- Build a role-stratified Django web application with OTP email-verified authentication for Administrator and Tourist roles.
- Implement an AI-powered recommendation engine that ranks destinations using a composite score derived from average star rating, NLP sentiment polarity, and review volume.
- Integrate Google Gemini 2.5 Flash for structured, day-wise itinerary generation across any Indian state and city, supporting both Eco-Tourism and Cultural Heritage themes.
- Integrate a multimodal AI chatbot (Bharat AI) capable of responding to text queries and identifying tourist landmarks from uploaded images, returning structured JSON travel intelligence.
- Deploy a TextBlob-based sentiment analysis pipeline that classifies every tourist review as Positive, Neutral, or Negative, stores the polarity score, and triggers an in-app notification upon classification completion.
- Maintain a national destination catalogue covering all 28 Indian states and Union Territories, with geolocation coordinates supporting Google Maps integration.
- Provide an interactive map view rendering all destinations as navigable map markers linked to their detail pages.
- Enable an admin governance layer for destination CRUD operations, local guide and homestay management, review moderation, and user account administration.
- Support a live review ticker on the landing page that surfaces real-time sentiment-tagged feedback from the community.
- Persist all AI-generated trip plans to user profiles for historical trip tracking and retrieval.

---

## Summary

Bharat AI is a Django 5.2 web application structured as five modular apps: `accounts`, `destinations`, `ai_engine`, `services`, and `notifications`. The platform's core value proposition is the convergence of AI intelligence with tourism discovery — replacing static directory listings with dynamically ranked, sentiment-informed destination recommendations.

The **Recommendation Engine** (`ai_engine/logic.py`) computes a Smart Score for each destination by combining three normalized signals: average star rating (weight 0.4), NLP sentiment polarity of all reviews (weight 0.4), and review volume as a popularity proxy (weight 0.2). The resulting score out of 100 drives the default sort order on the explore page.

The **AI Trip Planner** (`ai_engine/itinerary_service.py`) accepts user inputs for duration (days), destination category (Eco or Cultural), target state, and optional city, then constructs a structured prompt for Google Gemini 2.5 Flash. Gemini returns a raw JSON array of day objects — each containing timestamped activities with location details — which are parsed, rendered in a visual day-by-day format, and automatically saved to the user's trip history.

The **Bharat AI Chatbot** (`destinations/views.py`) operates as a multimodal conversational agent. It accepts text queries and base64-encoded images, sends them to Gemini with a culturally rich system prompt, and returns both a travel narrative and a structured list of database-matched destination suggestions extracted from the model's `PLACES_FOUND:` signal.

The **Sentiment Pipeline** (`services/views.py`, `ai_engine/logic.py`) runs on every review submission: TextBlob computes polarity on a −1.0 to 1.0 scale, which is thresholded into three labels (Positive > 0.15, Negative < −0.15, Neutral otherwise). The result is stored with the review and immediately surfaced to the user through an in-app notification.

---

## Features

### Before Login (Public-Facing)

- **Landing Page with Live Review Ticker** — A scrolling ticker bar displays the latest sentiment-tagged community reviews in real time, showing reviewer name, sentiment label (colour-coded green/red/gray), and destination name. The ticker pauses on hover.
- **Public Destination Catalogue** — A publicly accessible browse page listing all destinations with average ratings and review counts, sortable by creation date.
- **Destination Detail Pages** — Each destination has a full detail page showing name, state, category, description, historical background, best season, visiting hours, GPS coordinates, verified local guides (with languages and fees), verified homestays (with amenities and price per night), and the five most recent reviews with sentiment badges.
- **Interactive Map View** — All destinations are plotted as clickable markers on an embedded Google Maps interface, rendered from a serialized JSON array of coordinates. Each marker links to the corresponding destination detail page.
- **About Page** — Explains the platform's problem statement, vision, and AI capabilities.
- **User Registration** — Role-based registration (Tourist / Admin) with full name, email, phone number, and profile picture upload. Account activation requires OTP email verification.
- **OTP Email Verification** — A 6-digit random OTP is delivered via SMTP upon registration; it expires after 10 minutes. Unverified accounts cannot log in.
- **Secure Login** — Email-based authentication with login count tracking and role-based redirection (Admin Dashboard vs. Tourist Dashboard).
- **Password Reset** — Complete Django-native password reset flow with email-delivered reset links.

### After Login — Tourist

- **Tourist Dashboard** — A personalized home screen showing recent activity, quick-access navigation to AI tools, and a notification count indicator.
- **AI Explore Page** — Category-filtered (Eco / Cultural) and keyword-searchable destination browser with Smart Score rankings computed in real time from the NLP + Rating + Popularity composite formula.
- **AI Trip Planner** — A form accepting trip duration (days), category theme, Indian state, and optional city. Submitting the form calls Google Gemini 2.5 Flash, which returns a structured day-wise itinerary rendered as a visual timeline. Every generated plan is automatically saved to trip history.
- **Trip History** — A chronological list of all previously generated itineraries. Each item is expandable to view the full day-by-day plan.
- **Bharat AI Chatbot** — An interactive chat interface where tourists can ask text questions about any Indian state or upload a photo of a landmark. The chatbot returns travel intelligence (safety, transport, culture, hidden gem) as structured cards and, when landmarks are identified, displays matching destination tiles with images and links.
- **Review Submission** — Tourists can rate a destination (1–5 stars) and submit a written review. Upon submission, the TextBlob sentiment engine classifies the review, the result is stored, and an in-app notification confirms the AI analysis outcome.
- **Profile Management** — Tourists can update their full name, phone number, and profile picture.
- **Notifications Inbox** — Displays all system notifications (review analysis completions, admin alerts) with read/unread status.

### After Login — Administrator

- **Admin Dashboard** — A centralized overview showing platform-wide statistics: total destinations, total users, total reviews, and a breakdown of content by category.
- **Add Destination** — A fully validated form for registering new destinations with all metadata: name, state (all 28 states + UTs supported), category, description, historical narrative, GPS coordinates, best time to visit, visiting hours, and image upload.
- **Edit Destination** — Inline editing of any existing destination record.
- **Destination Manager** — A tabulated list of all destinations with quick-edit and delete actions.
- **Manage Services** — A dual-form interface for attaching new verified local guides (name, language proficiencies, contact, daily fee) and verified homestays (name, amenities, price per night, contact) to specific destinations.
- **User Manager** — A paginated table of all registered users (excluding the admin themselves) with account activation/deactivation toggle controls.
- **Review Moderation** — Admin can delete any review from the platform to enforce content standards.
- **Admin Moderation Panel** — Centralized content moderation view for reviewing and removing flagged or inappropriate content.

---

## Technologies Used

### Programming Languages
- **Python 3.11+** — Backend application logic, ML inference, and AI integrations.
- **HTML5 / CSS3 / JavaScript** — Frontend templating, responsive layout, and dynamic interactions.

### Libraries & Frameworks
- **Django 5.2** — Core MVT web framework, ORM, session management, authentication, and email.
- **django-crispy-forms 2.5** with **crispy-bootstrap5** — Structured form rendering.
- **Tailwind CSS (CDN)** — Utility-first CSS framework; theme colors: Emerald `#2E7D32`, Heritage Gold `#FFD700`, Sky Wash `#F0F8FF`.
- **Font Awesome (CDN)** — Iconography across all templates.
- **TextBlob** — NLP library for sentiment polarity analysis on tourist reviews.
- **Pillow 12.1.0** — Image processing for profile pictures and destination image uploads.
- **qrcode[pillow] 8.0** — QR code generation support.
- **ReportLab 4.4.9** — PDF generation for exportable reports.

### Machine Learning / AI
- **TextBlob NLP Sentiment Engine** — Rule-based sentiment classification using lexicon polarity scoring. Thresholds: Positive (> 0.15), Negative (< −0.15), Neutral (−0.15 to 0.15). Stores both the label and the raw polarity float for downstream scoring.
- **AI Recommendation Engine** — Weighted composite scoring formula combining normalized star rating, sentiment polarity, and review popularity into a Smart Score out of 100.
- **Google Gemini 2.5 Flash Lite (`gemini-2.5-flash-lite`)** — Powers the AI Trip Planner, generating structured JSON day-wise itineraries for any Indian location with Eco or Cultural theme.
- **Google Gemini 2.5 Flash Lite (`models/gemini-2.5-flash-lite`)** — Powers the Bharat AI Chatbot with structured JSON state-level travel intelligence (safety, transport, culture, hidden gem).
- **Multimodal Gemini (`google-genai` SDK with `types.Part.from_bytes`)** — Destination chatbot processes base64-encoded JPEG images alongside text for visual landmark identification and place matching.

### Backend
- **Django ORM** — All database interactions via model-level abstractions.
- **Custom Abstract User Model** — Email-based authentication replacing the default username field; two roles: `admin` and `tourist`.
- **Django Management Commands** — `setup_demo_data` for curated development content; `seed_india.py` for seeding 1,000 national locations.
- **Django Context Processors** — `notifications.context_processors.notification_count` injects unread notification count globally.
- **SMTP (Gmail)** — Email backend for OTP delivery and password reset links.
- **python-dotenv 1.2.1** — API key management via `.env` (`AI_API_KEY`).

### Frontend
- **Django Template Language** — Server-side HTML rendering with two-layer base template inheritance (`base1.html` public / `base2.html` authenticated).
- **Tailwind CSS** — Responsive grid layouts, component styling, and themed color tokens.
- **JavaScript (Vanilla)** — Chatbot AJAX POST, image upload and base64 encoding, category filter toggling, and ticker animation control.
- **Google Maps JavaScript API** — Interactive destination map with JSON-serialized coordinate markers and info windows.

### Database
- **SQLite3** — File-based relational database; pre-seeded `db.sqlite3` included for immediate local demonstration.

### Deployment & DevOps
- **Django WSGI** (`config/wsgi.py`) and **ASGI** (`config/asgi.py`) — Both sync and async entry points configured.
- **Django staticfiles** — Static asset collection pipeline for production.

### Security
- **Django CSRF Middleware** — All POST endpoints protected against cross-site request forgery.
- **Django Password Validators** — Similarity, minimum length, common password, and numeric-only checks enforced.
- **Email OTP Verification** — Accounts inactive until email ownership confirmed; OTP expires after 10 minutes.
- **Role-based Access Control** — `@login_required`, `@staff_member_required`, and inline `request.user.role != 'admin'` guards on all sensitive views.
- **Environment Variable Isolation** — `AI_API_KEY` stored in `.env`, never hardcoded in source.

---

## Dataset

### Destination Data
The platform covers all 28 Indian states and 8 Union Territories across two destination categories: **Eco-Tourism (ECO)** and **Cultural Heritage (CULTURAL)**. Each destination record stores name, state, category, description, historical narrative, GPS coordinates, best visiting season, daily visiting hours, and an image file.

The `seed_india.py` script populates 1,000 destination records across 12 representative states, each with 2–3 verified local guides and 1–2 verified homestays with randomized pricing (homestay: ₹1,500–₹8,000/night). The `DestinationForm` state dropdown covers all 28 states and all Union Territories for admin-managed data entry.

### Review Data (Runtime Generated)
Each `Review` record captures: star rating (1–5), free-text comment, AI-assigned sentiment label, polarity float, timestamp, and user-destination foreign keys.

### Trip Plan Data (Runtime Generated)
AI-generated itineraries are persisted as JSON blobs in the `TripPlan` model (`itinerary_data` JSONField), keyed to the requesting user with day count, category, title, and creation timestamp.

### Chat Conversation Logs
Every chatbot interaction is persisted in `ChatConversation` with user reference (nullable for anonymous access), raw message, model response, anonymity flag, and timestamp.

---

## System Architecture

```
tourism-website/
├── config/              # Settings, root URL conf, WSGI/ASGI
├── accounts/            # Custom user model, OTP auth, roles, profiles
├── destinations/        # Destination, guides, homestays, map, admin chatbot
├── ai_engine/           # TextBlob NLP, recommendation scoring, Gemini planner, chatbot
├── services/            # Review submission, sentiment pipeline, homepage feed
├── notifications/       # In-app notification model and context processor
├── templates/           # Global Django templates organized by app
├── static/              # CSS, JS, and image assets
├── media/               # User-uploaded destination images and profile pictures
├── seed_india.py        # National destination seeder (1,000 locations)
└── req.txt              # Pinned dependencies
```

### Data Flow

1. A **Tourist** submits a review → `services/views.py` calls `analyze_sentiment()` → TextBlob computes polarity → label and score stored → `Notification` created confirming AI classification.
2. On the **Explore Page** → destinations annotated with `avg_rating`, `avg_sentiment`, `review_count` → `calculate_recommendation_score()` called per destination → results sorted by Smart Score descending.
3. **Trip Planner** form submitted → `generate_smart_itinerary()` calls Gemini 2.5 Flash → JSON parsed → rendered as day-by-day timeline → `TripPlan` record saved automatically.
4. **Chatbot** POST received → text + optional image assembled → Gemini queried with structured system prompt → `PLACES_FOUND:` signal extracted → matching `Destination` records returned as cards.
5. All unread notification counts injected globally via `notification_count` context processor.

---

## Model Workflow

### Sentiment Analysis Pipeline

**Step 1 — Review Submission:** Tourist submits star rating and free-text comment.

**Step 2 — TextBlob Inference:**
```
polarity = TextBlob(comment).sentiment.polarity   # Range: −1.0 to 1.0
```

**Step 3 — Label Classification:**

| Polarity Range | Label    |
|----------------|----------|
| > 0.15         | POSITIVE |
| < −0.15        | NEGATIVE |
| −0.15 to 0.15  | NEUTRAL  |

**Step 4 — Persistence:** Label and polarity score stored on `Review`. In-app `Notification` created for the submitter with the classification result.

---

### AI Recommendation Scoring

**Step 1 — Feature Aggregation:** ORM annotates each destination with `avg_rating`, `avg_sentiment`, and `review_count`.

**Step 2 — Normalization:**

| Signal | Formula |
|---|---|
| Rating | `(avg_rating − 1) / 4` → 0 to 1 |
| Sentiment | `(sentiment_score + 1) / 2` → 0 to 1 |
| Popularity | `min(review_count / 100, 1.0)` → capped at 1.0 |

**Step 3 — Weighted Composite Score:**
```
smart_score = (norm_rating × 0.4) + (norm_sentiment × 0.4) + (popularity × 0.2)
final_score = round(smart_score × 100, 1)   # Out of 100
```

**Step 4 — Ranked Rendering:** Explore page sorts destinations by `smart_score` descending, with category and keyword filters applied upstream.

---

### AI Trip Planner Workflow

**Step 1:** User inputs days, category (Eco / Cultural), state, optional city.

**Step 2:** Structured prompt constructed for Gemini 2.5 Flash, enforcing raw JSON output with schema: `day`, `theme`, `activities[]` (each with `time`, `activity`, `description`, `location`).

**Step 3:** Gemini response sanitized (markdown fences stripped) → `json.loads()` → rendered as visual day-card timeline.

**Step 4:** Parsed JSON saved as `TripPlan.itinerary_data` (JSONField) and linked to the requesting user.

---

### Bharat AI Chatbot Workflow

**Step 1:** Chat interface POSTs JSON with `msg` text and optional `image` (base64 JPEG).

**Step 2:** If image present, decoded via `base64.b64decode()` and wrapped as `types.Part.from_bytes(data=img_bytes, mime_type="image/jpeg")`.

**Step 3:** Gemini called with a system instruction enforcing strict JSON output:
```json
{
  "state": "...",
  "safety": ["...", "...", "..."],
  "transport": ["...", "...", "..."],
  "culture": ["...", "...", "..."],
  "hidden_gem": { "name": "...", "description": "..." }
}
```

**Step 4:** Destination chatbot variant extracts `PLACES_FOUND:` signal → queries `Destination.objects.filter(name__in=names)` → returns matching destination cards with images alongside the text response.

**Step 5:** Interaction logged in `ChatConversation`.

---

## Implementation Details

### Custom User Model (`accounts/models.py`)
Django's `AbstractUser` is extended with `email` as `USERNAME_FIELD`, two roles (`admin`, `tourist`), `is_verified` for OTP gating, `login_count` for session analytics, `full_name`, `profile_picture`, and `phone_number`. The `CustomUserManager` handles both regular and superuser creation with email normalization.

### OTP Authentication (`accounts/views.py`)
On registration, the user is created with `is_active=False`. A 6-digit OTP (`random.randint(100000, 999999)`) is stored in `EmailOTP` with a creation timestamp, sent via SMTP, and enforced with a 10-minute expiry via `is_expired()`. Login is blocked for unverified accounts.

### Recommendation Engine (`ai_engine/logic.py`)
`calculate_recommendation_score()` receives pre-annotated ORM values. Rating normalization maps the 1–5 integer scale to 0–1 via `(x−1)/4`. Sentiment normalization shifts the −1 to 1 polarity float to 0–1 via `(score+1)/2`. Popularity uses a linear ramp capped at 100 reviews. The weighted sum is multiplied by 100 and rounded to one decimal place.

### Gemini Itinerary Service (`ai_engine/itinerary_service.py`)
Uses the `google-generativeai` SDK with `GenerativeModel("gemini-2.5-flash-lite")`. The f-string prompt injects user parameters and mandates raw JSON output. Response sanitization strips `\`\`\`json` and `\`\`\`` markers before `json.loads()`. On parse failure, `None` is returned and the view handles the empty state gracefully.

### Multimodal Chatbot (`destinations/views.py`)
Uses the newer `google-genai` SDK with `genai.Client` and `client.models.generate_content`. Image data arriving as a data URL is split on `base64,`, the payload decoded, and passed as `types.Part.from_bytes`. System instructions are passed via `types.GenerateContentConfig(system_instruction=...)`. A separate structured-JSON chatbot variant in `ai_engine/views.py` handles the state-level travel intelligence query with a different Gemini model invocation pattern.

### Interactive Map (`destinations/views.py`)
`interactive_map_view` serializes all `Destination` objects into a Python list of dicts with `name`, `lat`, `lng`, `url`, and `category`, then passes them to the template as `json.dumps(places_data)`. JavaScript initializes the Google Maps instance, iterates the parsed JSON, and creates markers with info windows containing linked destination names.

### Live Review Ticker (`services/views.py`, `templates/landing.html`)
The homepage view queries the five most recent `Review` objects, builds dictionaries with reviewer display name (full name or email prefix), lowercased sentiment label for CSS class binding, destination name, and a 50-character comment preview. The template renders the list twice (duplicated) inside a CSS `@keyframes ticker-scrolling` animation for seamless infinite loop. The ticker pauses on hover via `animation-play-state: paused`.

### Notification Context Processor (`notifications/context_processors.py`)
Queries `Notification.objects.filter(user=request.user, is_read=False).count()` on every request for authenticated users, injecting the count into all templates for the navigation badge — without requiring per-view boilerplate.

### National Seeder (`seed_india.py`)
A standalone Django-environment Python script that iterates 1,000 times, randomly selecting a state and destination name from a curated `states_data` dictionary, creating the `Destination`, then creating 2–3 `LocalGuide` and 1–2 `Homestay` records per destination with randomized pricing.

---

## Results

### AI Recommendation Engine

| Parameter | Value |
|---|---|
| Rating weight | 0.4 (40%) |
| Sentiment weight | 0.4 (40%) |
| Popularity weight | 0.2 (20%) |
| Score range | 0.0 – 100.0 |
| Popularity saturation point | 100 reviews |
| Rating normalization | `(rating − 1) / 4` |
| Sentiment normalization | `(polarity + 1) / 2` |

### Sentiment Analysis

| Metric | Value |
|---|---|
| Model | TextBlob (lexicon-based NLP) |
| Polarity range | −1.0 to 1.0 |
| Positive threshold | > 0.15 |
| Negative threshold | < −0.15 |
| Default pre-analysis state | PENDING |
| Stored fields | Label (string) + polarity (float) |

### Platform Functional Coverage

| Feature | Status |
|---|---|
| OTP email authentication (Tourist / Admin) | Fully implemented |
| TextBlob AI sentiment analysis on reviews | Fully implemented |
| AI composite recommendation scoring | Fully implemented |
| Gemini trip planner with auto-save | Fully implemented |
| Multimodal Bharat AI chatbot (text + image) | Fully implemented |
| Google Maps interactive destination map | Fully implemented |
| Live sentiment-tagged review ticker | Fully implemented |
| National destination catalogue (all states/UTs) | Fully implemented |
| Local guide and homestay management | Fully implemented |
| Review moderation (admin) | Fully implemented |
| User activation/deactivation (admin) | Fully implemented |
| In-app notifications with context processor | Fully implemented |
| Password reset via email | Fully implemented |
| Trip history tracking and retrieval | Fully implemented |
| 1,000-location national seeder | Fully implemented |

---

## Installation

### Prerequisites

- Python 3.11 or higher
- pip
- Git
- A Google Gemini API key (from [Google AI Studio](https://aistudio.google.com/))

### Step 1 — Clone the Repository

```bash
git clone https://github.com/harisaigithub/tourism-website.git
cd tourism-website
```

### Step 2 — Create and Activate a Virtual Environment

```bash
python -m venv venv

# Windows
venv\Scripts\activate

# macOS/Linux
source venv/bin/activate
```

### Step 3 — Install Dependencies

```bash
pip install -r req.txt
```

### Step 4 — Configure Environment Variables

Create a `.env` file in the project root:

```env
AI_API_KEY=your_google_gemini_api_key_here
```

Update email credentials in `config/settings.py`:

```python
EMAIL_HOST_USER = 'your_gmail@gmail.com'
EMAIL_HOST_PASSWORD = 'your_gmail_app_password'
DEFAULT_FROM_EMAIL = 'Smart Tourism Support <your_gmail@gmail.com>'
```

### Step 5 — Apply Migrations

```bash
python manage.py migrate
```

### Step 6 — Seed Destination Data

For the full national dataset (1,000 destinations):

```bash
python seed_india.py
```

For a curated development dataset:

```bash
python manage.py setup_demo_data
```

### Step 7 — Create an Administrator Account

```bash
python manage.py createsuperuser
```

### Step 8 — Collect Static Files (Production)

```bash
python manage.py collectstatic
```

### Step 9 — Start the Development Server

```bash
python manage.py runserver
```

Access the application at `http://127.0.0.1:8000/`.

---

## Usage

### Tourist Workflow

1. Visit the landing page to see the live sentiment ticker and public destination catalogue.
2. Register with email, full name, phone number, and profile picture. Select the **Tourist** role.
3. Enter the 6-digit OTP sent to your email to activate your account.
4. Log in and reach the **Tourist Dashboard**.
5. Navigate to **Explore Destinations** to browse AI-ranked places. Filter by Eco or Cultural category, or search by keyword.
6. Open any destination to view its full profile, verified guides, homestays, and recent reviews.
7. Click **Submit a Review** to rate and comment — AI sentiment analysis runs automatically.
8. Open **Plan My Trip**, enter your preferences, and generate a Gemini-powered day-wise itinerary.
9. Access all saved plans from **My Trip History**.
10. Chat with **Bharat AI** for state-level safety, transport, culture, and hidden gem recommendations — or upload a landmark photo for AI identification.

### Administrator Workflow

1. Log in with an admin-role account.
2. The **Admin Dashboard** shows platform-wide statistics and management shortcuts.
3. Use **Add Destination** to register new tourism sites with GPS coordinates and all metadata.
4. Navigate to **Manage Services** to attach local guides and homestays to destinations.
5. Open **User Manager** to view tourist accounts and toggle active status.
6. Use the **Moderation Panel** to delete inappropriate reviews.

---

## Future Enhancements

- **Collaborative Filtering** — Augment content-based scoring with user-based collaborative filtering, recommending destinations rated highly by tourists with similar review histories.
- **QR Code Destination Passes** — Generate downloadable QR codes (via `qrcode[pillow]`, already in dependencies) for each destination, embeddable in printed brochures.
- **Multi-Language Support** — Add Django i18n translations for Hindi, Telugu, Tamil, and Bengali for regional accessibility.
- **PDF Itinerary Export** — Use ReportLab (already in dependencies) to export AI-generated trip plans as formatted downloadable PDFs.
- **Booking Integration** — Connect verified homestays to Razorpay or Stripe for direct in-platform booking and confirmation.
- **Real-Time Sentiment Analytics Dashboard** — Extend admin dashboard with Chart.js sentiment trend charts per destination over time.
- **Progressive Web App (PWA)** — Add service worker and offline caching for previously viewed destination details, useful in low-connectivity areas.
- **Advanced NLP Pipeline** — Replace TextBlob with a fine-tuned DistilBERT classifier for higher accuracy on travel-domain and code-mixed review language.
- **PostgreSQL Migration** — Transition from SQLite3 to PostgreSQL for production deployments requiring concurrent access and full-text search.
- **Guide Availability Calendar** — Add real-time availability management for local guides, enabling tourists to check and request bookings within the platform.

---

## Contributing

1. Fork the repository and clone locally:
   ```bash
   git clone https://github.com/your-username/tourism-website.git
   ```

2. Create a feature branch:
   ```bash
   git checkout -b feature/your-feature-name
   ```

3. Make changes, maintaining consistency with the modular Django app structure.

4. Commit with a descriptive message:
   ```bash
   git commit -m "feat: add PDF itinerary export via ReportLab"
   ```

5. Push and open a Pull Request against `main`. Describe the problem solved and approach taken.

Ensure all new views are protected with `@login_required` or `@staff_member_required` as appropriate, and all new models include `__str__` methods and appropriate `related_name` attributes.

---

## License

This project is licensed under the **MIT License**.

```
MIT License

Copyright (c) 2025 Hari Sai Parasa

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

**GitHub:** [harisaigithub](https://github.com/harisaigithub)

**Email:** harisaiparasa@gmail.com

---


&nbsp;

&nbsp;

## ***To acquire this project, please contact***

**GitHub:** https://github.com/harisaigithub  
**Email:** [harisaiparasa@gmail.com](mailto:harisaiparasa@gmail.com)  

For collaborations, customizations, or deployment support, feel free to reach out.