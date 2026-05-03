# Sikkim Tourism AI Platform

### *Discover the Himalayan Soul — Intelligently*

> A full-stack AI-powered tourism web platform for Sikkim, combining geo-spatial place discovery, Gemini-powered cultural intelligence, TF-IDF machine learning itinerary recommendations, image-based landmark recognition, and an interactive Leaflet.js map into a single cohesive travel companion.

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

Sikkim, nestled in the eastern Himalayas, is one of India's most ecologically and culturally rich states — home to ancient monasteries, alpine lakes, adventure trails, sacred viewpoints, and a biodiversity that spans subtropical forests to glacial heights. Yet the experience of planning a visit to Sikkim remains fragmented, heavily reliant on generic travel aggregators that lack the depth, specificity, and cultural sensitivity that this unique destination deserves.

The **Sikkim Tourism AI Platform** is a purpose-built digital tourism ecosystem that transforms how travellers discover, plan, and experience Sikkim. At its core, the platform fuses curated geo-spatial data with the generative and visual capabilities of Google Gemini, a TF-IDF content-based ML recommendation engine, and a GeoDjango spatial database, all delivered through a responsive, multi-language web interface.

Tourists can ask a culturally-aware AI guide about monasteries and trails, upload photographs to identify Sikkim landmarks using computer vision, generate personalised multi-day itineraries grounded in both ML-ranked place recommendations and destination-specific filtering, and save travel plans as downloadable PDF reports. Administrators manage all content, monitor AI system health, and analyse engagement through a dedicated admin console.

---

## Objectives

- Build a comprehensive, searchable directory of Sikkim tourist places categorised by type (monasteries, lakes, viewpoints, adventure, culture) with rich media, entry fees, timings, and geospatial coordinates.
- Implement a TF-IDF cosine-similarity ML engine to rank tourist places by relevance to a traveller's stated interests, feeding AI-generated itinerary prompts with contextually accurate recommendations.
- Integrate Google Gemini (1.5 Flash / 2.5 Flash) as a multi-modal AI guide capable of answering cultural and travel questions, processing uploaded images, and generating complete multi-day travel itineraries in structured JSON format.
- Deploy an AI Lens feature that uses Gemini vision to identify Sikkim landmarks from tourist-uploaded photographs and retrieve the matching place record from the database.
- Provide an interactive Leaflet.js map with PostGIS / SpatiaLite-backed geospatial point data, served as a GeoJSON API endpoint for real-time marker rendering.
- Deliver a user dashboard where authenticated travellers can manage saved places, review past itinerary plans, and download trip reports as formatted PDF documents.
- Support destination-scoped itinerary planning across ten major Sikkim destinations (Gangtok, Pelling, Lachung, Lachen, Namchi, Ravangla, Yuksom, Zuluk, Singtam, Jorethang).
- Enable anonymous and authenticated chatbot interactions, with all conversations persisted for admin review and AI system health monitoring.

---

## Summary

The Sikkim Tourism AI Platform is a Django-based web application spanning six core apps — `accounts`, `portal`, `tourism`, `itinerary`, `chatbot`, and `adminpanel` — integrated with GeoDjango and backed by SpatiaLite for spatial database operations. The application is configured for both development and production environments, with ngrok tunnel support for CSRF-trusted origin management and OpenStreetMap tiles served via the django-leaflet configuration module.

The tourism data layer is built around the `TouristPlace` model, which stores rich descriptive content alongside a `PointField` spatial geometry column (SRID 4326) for each location. Categories (monasteries, lakes, adventure, viewpoints, culture) are managed independently, and each place carries visitor metrics (`view_count`) updated on every AI Lens recognition event. `SavedPlace` enables authenticated users to bookmark destinations, and `Hotel` provides geospatially-linked accommodation data per region.

The AI subsystem operates across three distinct interaction modes: a conversational chatbot (Gemini 2.5 Flash, persona-driven as a Virtual Monk), a vision-based landmark identifier (AI Lens, Gemini 1.5 Flash), and a structured itinerary generator (Gemini 1.5 Flash, JSON-mode output). The ML recommendation engine uses scikit-learn's `TfidfVectorizer` and cosine similarity to rank all database places against the user's free-text interest statement, bridging the gap between unstructured user intent and structured content.

---

## Features

### Public / Pre-Login

- Responsive landing page with a curated grid of six featured tourist places sourced directly from the database (`is_featured=True`).
- Role-based registration (Tourist / Admin) with email OTP verification and a 10-minute OTP expiry window.
- Secure login with email as the primary identifier, session management, and CSRF protection with ngrok tunnel trusted origins.
- Password reset via tokenised email links through Gmail SMTP.

### Tourist Exploration

- **Explore Places:** Full catalogue view of all Sikkim tourist places, browsable by category and searchable by name, with main images, timings, entry fees, and location names.
- **Place Detail Page:** Deep-dive view for each place including full description, historical context, how-to-reach guide, best time to visit, nearby hotel listings (JSONField), and view count tracking.
- **Category Browser:** Visual grid of all tourism categories (Monasteries, Lakes, Adventure, Viewpoints, Culture) with category images and place counts.
- **Interactive Map:** Leaflet.js map centred on Gangtok (27.3314°N, 88.6138°E) with OpenStreetMap tiles. All places with spatial coordinates are rendered as markers via a GeoJSON API endpoint (`/api/places-geojson/`) served from SpatiaLite's PostGIS-compatible PointField.
- **Saved Places:** Authenticated users can save and unsave any tourist place. A uniqueness constraint (`unique_together`) prevents duplicate saves per user-place pair.

### AI Features

- **AI Travel Guide Chatbot:** A Gemini 2.5 Flash-powered conversational assistant adopting the persona of a Virtual Monk and Sikkim tourism expert. Responds in plain, structured text (numbered points, no markdown). Performs keyword-based ORM lookups on tourist place names before invoking the AI, enabling structured place cards alongside the conversational reply. Supports image uploads — tourists can share monastery photos and receive cultural identification and context.
- **AI Lens (Landmark Recognition):** Upload any photograph of a Sikkim landmark. Gemini 1.5 Flash identifies the location by name, the system performs a case-insensitive ORM lookup, increments the place's `view_count`, and returns the full matching place record with image, timings, and detail page link. Returns a graceful "not found" response with the AI's best guess when no database match exists.
- **AI Itinerary Generator:** A dual-path intelligent trip planner:
  - *Destination Mode:* Select a specific destination (e.g., Pelling, Lachung) to filter the itinerary to only places within that region. Places are queried live from the database before being embedded in the Gemini prompt.
  - *Smart ML Mode:* When no destination is selected, the TF-IDF cosine-similarity engine ranks all database places against the user's stated interests, and the top recommendations are passed as context to the Gemini prompt.
  - Supports inputs for trip duration (days), number of travellers, interests (free text), weather preference, and activity category (sightseeing, adventure, spiritual, nature).
  - Gemini returns a complete raw JSON array (3–4 activities per day) with day, time, activity, location, and place fields — no markdown, no truncation.
  - All generated itineraries for authenticated users are saved to the `Itinerary` model with full metadata (destination, category preference, weather preference, travellers count).

### User Dashboard

- Personal view of all saved itinerary plans with plan count and creation timestamps.
- Saved places collection with quick-access links to each place detail page.
- Saved places count and plan count displayed as summary metrics.
- Profile settings page for account details management.
- **Itinerary PDF Download:** Any saved itinerary can be exported as a formatted PDF report (`xhtml2pdf` / `pisa`) via `/itinerary/download/<id>/`, rendered from an HTML template and served as an attachment download.

### Admin Panel

- **Admin Dashboard:** Consolidated platform overview.
- **User Directory:** Full list of all registered tourist accounts with activation toggle (`is_active`) and account deletion capability. Superuser accounts are protected from self-modification.
- **Content Management:** Browse, manage, and delete tourist place records. Bulk upload of new places via JSON file upload (`/admin-panel/bulk-upload/`) — the file must contain a `places` array following the place schema.
- **Analytics Dashboard:** Category distribution charts (category name vs. place count), top 7 most-viewed places with view counts, total itineraries generated, and total registered users.
- **AI Core Monitor:** Superuser-only view displaying AI system operational status, response latency, active model name (Gemini 1.5 Flash), total API usage (derived from itinerary count), and enabled capabilities (Vision Recognition, Itinerary Generation, Cultural Q&A).
- **Chat Logs:** Full history of all `ChatConversation` records (both anonymous and authenticated) for AI quality review.
- **Incident Reporting:** The `Incident` model supports structured reporting of safety or content incidents with type, description, file evidence, risk score (0–100), risk level (Low / Medium / High), anonymous flag, and status tracking.

### Multi-Language Support

- Interface translated into English, Hindi, and Telugu via Django's i18n framework.
- Language preference persisted in a `django_language` cookie with a one-year expiry.
- `LocaleMiddleware` positioned correctly in the middleware stack for per-request language resolution.

---

## Technologies Used

### Programming Languages
- Python 3.10
- JavaScript (ES6+)
- HTML5 / CSS3

### Libraries & Frameworks
- Django 4.x — Web framework, ORM, and i18n
- GeoDjango — Spatial database extension (PointField, GeoJSON serialisation)
- django-leaflet — Leaflet.js map integration with Django template tags
- Django REST Framework — API response scaffolding
- `xhtml2pdf` (pisa) — HTML-to-PDF itinerary report generation
- `python-dotenv` — Environment variable management

### Machine Learning / AI
- scikit-learn (`TfidfVectorizer`, `cosine_similarity`) — Content-based place recommendation engine
- Google Generative AI SDK (`google-generativeai`, `google-genai`) — Gemini 1.5 Flash and Gemini 2.5 Flash
- Multi-modal AI — Text queries, image uploads, and structured JSON generation in a single model call

### Backend
- Django ORM with GeoDjango spatial extensions
- GeoJSON API endpoint (`places_geojson`) using Django's `serialize('geojson', ...)` with `geometry_field='location'`
- Haversine formula implementation for geographic distance calculations
- Gmail SMTP (TLS, port 587) — Email OTP and password reset
- `lru_cache` — ML model caching via scikit-learn in-memory vectorisation

### Frontend
- Django Template Language (DTL) — Server-side rendering
- Leaflet.js — Interactive map with OpenStreetMap tile layer
- JavaScript Fetch API — Asynchronous chatbot, AI Lens, and itinerary API calls

### Database
- SpatiaLite (`django.contrib.gis.db.backends.spatialite`) — Spatial SQLite with PostGIS-compatible geometry support
- QGIS 3.44.6 / GDAL — Geospatial library dependencies (Windows: `gdal312.dll`, `mod_spatialite.dll`)

### Deployment & DevOps
- Django development server with ngrok tunnel support (`CSRF_TRUSTED_ORIGINS`)
- Whitenoise-ready static file configuration (`STATICFILES_DIRS`, `STATIC_ROOT`)
- Media file serving for place images and category images

### Security
- Django CSRF middleware with ngrok tunnel trusted origins
- Email OTP verification with 10-minute expiry enforced at the model level
- `@user_passes_test(lambda u: u.is_superuser)` — Superuser guard on all admin-tier views
- `@login_required` — Dashboard and profile access enforcement
- Django password validators (similarity, minimum length, common password, numeric)

### External APIs
- Google Gemini API (`AI_API_KEY`) — Powers chatbot, AI Lens, and itinerary generation
- OpenWeatherMap API (`OPENWEATHERMAP_API_KEY`) — Weather preference integration
- OpenStreetMap — Map tile layer (no API key required)

---

## Dataset

The platform's knowledge base is sourced from a curated admin-managed database of Sikkim tourist places, structured across the following models:

**TouristPlace:** Each record contains the place name, category (FK), full description, historical narrative, main image, visiting timings, entry fee, GIS point coordinates (latitude/longitude as `PointField`, SRID 4326), location name (district/town), how-to-reach guide, best visiting season, featured flag, view count, and a JSON array of nearby hotel suggestions.

**Category:** Six tourism categories are seeded with descriptive text and representative imagery: Monasteries, Lakes, Adventure, Viewpoints, Culture, and Sangay (natural sites). Each category carries a Font Awesome icon class for UI rendering.

**Hotel:** Geospatially-located accommodation records with name, address, `PointField` coordinates, contact number, and price range — linked to place records via the `nearby_hotels` JSONField.

**Media Assets:** Tourism place images are stored under `media/places/main/`, and category header images under `media/categories/`. Documented real Sikkim landmarks include Yumthang Valley, Zero Point, Nathu La Pass, Phensang Monastery, Ralang Monastery, Kathok Lake, Chenrezig Statue, Hanuman Tok, Ganesh Tok, and Sangay Falls.

**ML Recommendation Dataset:** The TF-IDF engine operates directly on the live database — `TouristPlace.description` fields serve as the corpus. No offline dataset file is required; the vectoriser is constructed at runtime from all current database records.

**Bulk Upload Format:** Administrators can seed new places via a JSON file with a `places` array, enabling rapid content population without individual data entry.

---

## System Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                      Presentation Layer                         │
│  Django Templates · Leaflet.js · JavaScript Fetch API          │
│  Multi-language: English / Hindi / Telugu (LocaleMiddleware)    │
└──────────────────────────┬──────────────────────────────────────┘
                           │ HTTP / WSGI
┌──────────────────────────▼──────────────────────────────────────┐
│                      Application Layer                          │
│                                                                 │
│  ┌────────────┐  ┌────────────────────────────────────────────┐ │
│  │  accounts  │  │                  portal                    │ │
│  │ ─────────  │  │  ──────────────────────────────────────── │ │
│  │ User       │  │  Landing · Explore · Categories · Map     │ │
│  │ EmailOTP   │  │  Itinerary · AI Lens · Admin Dashboard    │ │
│  │ Notification│ │  Analytics · Chat Logs · User Directory   │ │
│  │ (Tourist / │  │  Bulk Upload · Incident Reporting         │ │
│  │  Admin)    │  │  AI Core Monitor · GeoJSON API            │ │
│  └────────────┘  └────────────────────────────────────────────┘ │
│                                                                 │
│  ┌────────────────────┐  ┌──────────────────────────────────┐  │
│  │      tourism       │  │           itinerary              │  │
│  │ ────────────────   │  │  ──────────────────────────────  │  │
│  │ TouristPlace       │  │  AI Generation (Gemini 1.5)      │  │
│  │ Category           │  │  ML Recommendations (TF-IDF)     │  │
│  │ SavedPlace         │  │  Destination Filtering           │  │
│  │ Hotel              │  │  PDF Export (xhtml2pdf)          │  │
│  │ PointField (GIS)   │  │  Itinerary Model + JSONField     │  │
│  └────────────────────┘  └──────────────────────────────────┘  │
│                                                                 │
│  ┌───────────────────────────────────┐                          │
│  │             chatbot               │                          │
│  │  ───────────────────────────────  │                          │
│  │  Gemini 2.5 Flash (Virtual Monk)  │                          │
│  │  Keyword ORM Lookup (pre-AI)      │                          │
│  │  Image Upload + Vision Query      │                          │
│  │  ChatConversation persistence     │                          │
│  └───────────────────────────────────┘                          │
└──────────────────────────┬──────────────────────────────────────┘
                           │ Django ORM + GeoDjango
┌──────────────────────────▼──────────────────────────────────────┐
│                        Data Layer                               │
│  SpatiaLite (SQLite + PostGIS geometry) — db.sqlite3            │
│  PointField (SRID 4326) for TouristPlace and Hotel              │
│  Media: places/main/ · categories/ · evidence/                  │
└──────────────────────────┬──────────────────────────────────────┘
                           │
┌──────────────────────────▼──────────────────────────────────────┐
│                    External Services                            │
│  Google Gemini API (1.5 Flash + 2.5 Flash)                     │
│  OpenWeatherMap API · OpenStreetMap Tiles · Gmail SMTP          │
└─────────────────────────────────────────────────────────────────┘
```

---

## Model Workflow

### AI Itinerary Generation Pipeline

```
User Input: Days, Travelers, Interests (text), Weather, Category, Destination (optional)
        │
        ▼
  Path A — Destination Selected (e.g., "Pelling")
  ├── get_destination_places("Pelling")
  │     └── TouristPlace.objects.filter(location_name__icontains="Pelling")
  │               └── Returns list of place names for that region
  └── Build location_context: "Use ONLY these places from Pelling: ..."
        │
  Path B — No Destination (Smart ML Mode)
  ├── get_ml_recommendations(interests, top_n=10)
  │     ├── Fetch all TouristPlace.description values from DB
  │     ├── Append user interests string as final document
  │     ├── TfidfVectorizer(stop_words='english').fit_transform(descriptions)
  │     ├── cosine_similarity(last_row, all_other_rows)
  │     └── Return top-N places sorted by similarity score
  └── Build location_context: "ML recommends: Place A at Location, Place B at Location..."
        │
        ▼
  Construct Full Prompt
  ├── "Generate COMPLETE {days}-day itinerary for {travelers} travelers"
  ├── "Include ALL {days} days — Day 1 to Day N, DO NOT stop early"
  ├── "{location_context}"
  ├── "Return ONLY raw JSON array — no markdown, no backticks"
  └── "Each object: day, time, activity, location, place (5 keys exactly)"
        │
        ▼
  Gemini 1.5 Flash → generate_content(prompt)
        │
        ▼
  Clean Response (strip ```json fences if present)
  → json.loads(raw_text) → plan[]
        │
        ▼
  If authenticated → Itinerary.objects.create(user, days, travelers,
                       interests, weather, category, destination, plan_data)
        │
        ▼
  JsonResponse({'plan': plan})  → Frontend renders day-by-day schedule
        │
        ▼
  Optional: download_itinerary_pdf() → render_to_pdf(template, context)
            → xhtml2pdf.pisa.pisaDocument() → HttpResponse(PDF attachment)
```

### AI Chatbot (Virtual Monk) Pipeline

```
User Sends Message / Uploads Image
        │
        ▼
  Step 1 — Pre-AI Keyword ORM Lookup (text queries)
  ├── Tokenise user_msg, remove stop words
  ├── Build Q() filter: name__icontains=word for each token
  ├── TouristPlace.objects.filter(query).distinct()
  └── Build suggested_places[] list (id, name, image, timings, location)
        │
        ▼
  Step 2 — Gemini 2.5 Flash (multi-modal)
  ├── System prompt: Virtual Monk persona, plain text rules, PLACES_FOUND: extraction
  ├── Contents: [system_prompt, user_msg (if any), image_bytes (if uploaded)]
  └── generate_content(contents) → raw_reply
        │
        ▼
  Step 3 — Post-Processing
  ├── clean_markdown(raw_reply) — strip **, *, #, bullets
  ├── Extract "PLACES_FOUND: A, B, C" → merge with ORM lookup results
  └── Deduplicate suggested_places by place ID
        │
        ▼
  Step 4 — Persist
  ChatConversation.objects.create(user/None, message, response, is_anonymous)
        │
        ▼
  JsonResponse({'reply': text, 'places': [...], 'structured': bool})
```

### AI Lens Landmark Recognition Pipeline

```
Tourist Uploads Photo
        │
        ▼
  base64.b64encode(image_file.read())
        │
        ▼
  Gemini 1.5 Flash
  Prompt: "Identify this Sikkim landmark. Return ONLY the name."
  Contents: [prompt_text, {inline_data: {mime_type, data: base64}}]
        │
        ▼
  detected = response.text.strip()
        │
        ├── TouristPlace.objects.filter(name__icontains=detected).first()
        │         │
        │    MATCH FOUND:
        │         ├── place.view_count += 1 → place.save()
        │         └── JsonResponse({status, name, location, image, timings, detail_url})
        │
        └── NO MATCH:
                  └── JsonResponse({status: 'not_found', detected_name: detected,
                                    message: 'Place not in database yet'})
```

---

## Implementation Details

**Dual Itinerary Path Architecture:** The `itinerary_view` in `itinerary/views.py` implements two distinct context-building strategies that funnel into a single Gemini prompt. When a destination is selected, place names are fetched live from the ORM and embedded directly in the prompt, grounding the AI in actual database content. When no destination is selected, the TF-IDF engine runs at request time — the vectoriser is built fresh from all current database descriptions and the user's interest string, ensuring recommendations always reflect the latest content state without requiring offline model training or serialised artifacts.

**TF-IDF Cosine Similarity Engine:** The `get_ml_recommendations` function in `itinerary/ml_service.py` treats `TouristPlace.description` fields as a text corpus. The user's interest string is appended as the final document, the full matrix is vectorised using `TfidfVectorizer(stop_words='english')`, and cosine similarity is computed between the last row (user interests) and all place rows. The top-N highest-scoring indices are returned as ordered `TouristPlace` objects. This approach requires no offline training — the corpus updates automatically as the admin adds or modifies place records.

**GeoDjango SpatiaLite Integration:** The `TouristPlace` and `Hotel` models use `gis_models.PointField(srid=4326)` from `django.contrib.gis.db`. On Windows, the settings file configures QGIS 3.44.6 as the GDAL/PROJ provider, setting `PATH`, `GDAL_DATA`, `PROJ_LIB`, and `GDAL_LIBRARY_PATH` environment variables at import time. The `places_geojson` endpoint serialises all spatially-enabled place objects using Django's built-in `serialize('geojson', queryset, geometry_field='location')`, producing a standards-compliant GeoJSON FeatureCollection for Leaflet.js consumption.

**Structured JSON Itinerary Enforcement:** The Gemini prompt explicitly instructs the model to return only a raw JSON array with exactly five keys per object — preventing markdown contamination and schema drift. A post-processing step strips any residual code fences (`\`\`\`json`) before calling `json.loads()`, making the parsing robust to minor model formatting inconsistencies.

**Chatbot Persona and Output Rules:** The system prompt for the chatbot enforces four strict output rules injected before every Gemini call: no markdown, no asterisks, no bullets, numbered points only. The `PLACES_FOUND:` extraction protocol allows the model to declare place names at the end of its response in a machine-parseable format, enabling the system to merge AI-identified places with keyword ORM matches without relying on regex parsing of the conversational body.

**Admin Bulk Upload:** The `bulk_upload_places` view accepts a JSON file containing a `places` array, iterates over each object, and creates `TouristPlace` records programmatically. This enables rapid population of large place catalogues without manual admin entry, supporting content seeding from external data exports.

**PDF Export:** The `render_to_pdf` utility in `itinerary/utils.py` uses `xhtml2pdf` (pisa) to convert an HTML template rendered with Django's template engine into a PDF byte stream. The result is served as an `HttpResponse` with `Content-Type: application/pdf` and a `Content-Disposition: attachment` header, triggering a browser download named `Sikkim_Trip_{id}.pdf`.

---

## Results

| Metric | Value |
|---|---|
| Tourism Destinations Covered | 10 (Gangtok, Pelling, Lachung, Lachen, Namchi, Ravangla, Yuksom, Zuluk, Singtam, Jorethang) |
| Place Categories | 6 (Monasteries, Lakes, Adventure, Viewpoints, Culture, Nature) |
| User Roles | 2 (Tourist / Admin) |
| AI Models Integrated | 2 (Gemini 1.5 Flash, Gemini 2.5 Flash) |
| ML Algorithm | TF-IDF Cosine Similarity (scikit-learn) |
| Itinerary Generation | Fully structured JSON, 3–4 activities/day, multi-day support |
| AI Lens Recognition | Landmark identification from uploaded photos with DB match + view_count tracking |
| Languages Supported | 3 (English, Hindi, Telugu) |
| Map Technology | Leaflet.js + OpenStreetMap + SpatiaLite GeoJSON API |
| PDF Export | xhtml2pdf — downloadable itinerary reports |
| Chat Persistence | All conversations (anonymous + authenticated) stored in `ChatConversation` |
| Geospatial Backend | SpatiaLite (PointField, SRID 4326) via QGIS/GDAL |
| Bulk Content Management | JSON file upload for batch place creation |

---

## Installation

### Prerequisites

- Python 3.10 or higher
- pip
- Git
- QGIS 3.44.6 (Windows) — required for GDAL/SpatiaLite geospatial support
- A Google Gemini API key
- An OpenWeatherMap API key (for weather preference features)
- A Gmail account with App Password enabled (for email OTP)

### Step 1 — Clone the Repository

```bash
git clone https://github.com/harisaigithub/Sikkim-Tourism-website.git
cd Sikkim-Tourism-website
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
pip install -r requirements.txt
```

### Step 4 — Configure Environment Variables

Create a `.env` file in the project root:

```env
AI_API_KEY=your-google-gemini-api-key
OPENWEATHERMAP_API_KEY=your-openweathermap-api-key
GOOGLE_MAPS_API_KEY=your-google-maps-api-key
```

> The `AI_API_KEY` is required for all three AI features (chatbot, AI Lens, itinerary generation). The OpenWeatherMap key enables weather-preference-aware itineraries.

### Step 5 — Configure GDAL / SpatiaLite (Windows)

Ensure QGIS 3.44.6 is installed at `C:\QGIS 3.44.6`. The `core/settings.py` file auto-configures `GDAL_LIBRARY_PATH`, `GDAL_DATA`, and `PROJ_LIB` from this path on Windows. On Linux/macOS, install `libgdal-dev` and `spatialite-bin`:

```bash
# Ubuntu / Debian
sudo apt-get install gdal-bin libgdal-dev spatialite-bin
```

### Step 6 — Apply Migrations

```bash
python manage.py migrate
```

### Step 7 — Create a Superuser

```bash
python manage.py createsuperuser
```

### Step 8 — Load Initial Content

Populate tourist places via the admin bulk upload feature at `/admin-panel/bulk-upload/` by uploading a JSON file with a `places` array, or add records individually via Django's admin panel at `/admin/`.

### Step 9 — Collect Static Files (Production)

```bash
python manage.py collectstatic
```

### Step 10 — Run the Development Server

```bash
python manage.py runserver
```

Navigate to `http://127.0.0.1:8000/` to access the platform.

---

## Usage

### Tourist Workflow

1. Register at `/accounts/signup/`, verify the emailed OTP, and log in.
2. Browse the **Explore** page (`/explore/`) to discover Sikkim tourist places by category.
3. Open the **Interactive Map** (`/map/`) to visualise all places geographically on the Leaflet.js map.
4. Use the **AI Travel Guide** (`/chatbot/`) to ask questions about monasteries, trails, timings, and cultural sites. Upload a photo of any Sikkim landmark for instant identification.
5. Visit **AI Lens** (`/ai-lens/`) to upload a photo and identify the Sikkim landmark it depicts.
6. Navigate to **Itinerary** (`/itinerary/`) to generate a personalised multi-day Sikkim travel plan. Choose a specific destination or let the ML engine recommend places across Sikkim based on your stated interests.
7. Access your **Dashboard** (`/dashboard/`) to view all saved places, review generated itineraries, and download any plan as a PDF.

### Admin Workflow

1. Log in with a superuser account and navigate to `/admin-panel/`.
2. Open **Content Management** to add, edit, or delete tourist places. Use **Bulk Upload** to import multiple places at once via JSON.
3. Visit **User Directory** to manage tourist accounts — toggle active status or remove accounts as needed.
4. Review **Analytics** for category distribution charts, most-viewed places, and platform-wide usage metrics.
5. Check **Chat Logs** for all AI chatbot interactions across anonymous and authenticated sessions.
6. Monitor **AI Core** for Gemini API operational status and total API usage statistics.

---

## Future Enhancements

- **Real-Time Weather Integration:** Display live Sikkim weather (temperature, precipitation, conditions) from OpenWeatherMap on place detail pages and the itinerary planner, moving beyond the current weather preference string.
- **Personalized Recommendation Learning:** Evolve the TF-IDF engine to a collaborative filtering model trained on `SavedPlace` and `view_count` patterns, enabling user-based recommendations that improve with platform usage.
- **Multi-Language AI Responses:** Extend Gemini prompts with language context to return chatbot and itinerary responses in the user's chosen interface language (Hindi, Telugu).
- **Route Planning Integration:** Use the Google Maps API key already configured in settings to overlay driving or trekking routes between itinerary waypoints on the Leaflet map.
- **Offline PWA Support:** Convert the frontend to a Progressive Web App with service workers, caching key place data and the interactive map for use in areas with limited connectivity.
- **Hotel Booking Integration:** Expand the `Hotel` model and nearby hotel JSONField into a live booking link integration with accommodation APIs, enabling end-to-end trip planning from discovery to reservation.
- **Community Reviews:** Add a user review and star-rating model per tourist place, feeding aggregate ratings back into the TF-IDF recommendation corpus to reflect real visitor sentiment.
- **Seasonal Availability Flags:** Tag places with accessibility flags (monsoon-closed, winter-restricted, permit-required) and surface these in the itinerary planner to prevent planning routes to temporarily inaccessible destinations.

---

## Contributing

Contributions from developers, GIS engineers, and Sikkim tourism domain experts are welcome.

### Step 1 — Fork and Clone

```bash
git clone https://github.com/your-username/Sikkim-Tourism-website.git
cd Sikkim-Tourism-website
```

### Step 2 — Create a Feature Branch

```bash
git checkout -b feature/realtime-weather-widget
```

### Step 3 — Implement and Commit

```bash
git add .
git commit -m "feat: integrate OpenWeatherMap live widget on place detail pages"
```

### Step 4 — Push and Open a Pull Request

```bash
git push origin feature/realtime-weather-widget
```

Open a Pull Request against `main` with a description of the change, the feature it addresses, and any API keys or dependencies introduced.

### Code Standards

- Follow PEP 8 for all Python source files.
- Keep AI prompts in the view functions they serve — do not centralise them without clear documentation of the expected output schema.
- All new models with geographic data should use `gis_models.PointField(srid=4326)` from `django.contrib.gis.db`.
- Never commit API keys or credentials. Use `.env` with `python-dotenv` for all secrets.

---

## License

```
MIT License

Copyright (c) 2026 Sikkim Tourism AI Platform Team

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

This project was developed as a final-year engineering capstone focused on intelligent tourism technology for the state of Sikkim, India.

---

&nbsp;

&nbsp;

## ***To acquire this project, please contact***

**GitHub:** https://github.com/harisaigithub

**Email:** [harisaiparasa@gmail.com](mailto:harisaiparasa@gmail.com)

For collaborations, customizations, or deployment support, feel free to reach out.