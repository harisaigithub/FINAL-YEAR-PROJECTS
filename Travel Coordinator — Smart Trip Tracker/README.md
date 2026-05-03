# Travel Coordinator — Smart Trip Tracker

> **An AI-Powered, GPS-Enabled Travel Tracking Platform with Real-Time Analytics, Smart City Intelligence, and Native Android Support**

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

Urban and personal mobility generates enormous amounts of untapped data — GPS coordinates, travel durations, transport modes, costs, and route histories — that, if properly harnessed, could transform how individuals understand their movement patterns and how city planners allocate transportation resources.

**Travel Coordinator** is a full-stack, AI-enhanced travel tracking application that captures this data and turns it into actionable intelligence. Built with React 19 on the frontend, Flask 3 on the backend, and Capacitor 8 for native Android deployment, the platform serves two distinct stakeholder groups: **individual travelers** who want to log, visualize, and analyze their personal trips, and **smart city analysts** who need region-level traffic intelligence, heatmaps, peak-hour analysis, and AI-driven congestion forecasts.

At the core of the analyst-facing platform is a **TensorFlow LSTM (Long Short-Term Memory)** time-series model that predicts next-hour traffic volume, estimates CO₂ emissions per transport mode, assigns risk levels to congestion zones, and generates policy recommendations — all in real time based on live trip data sourced from the SQLite database.

The application is fully deployable as a Progressive Web App and as a native Android APK via Capacitor, with background geolocation tracking, local notifications for trip events, and CSV export to the device's Documents folder on mobile platforms.

---

## Objectives

- Design and implement a dual-role travel platform supporting individual trip tracking and smart city analytics from a single unified codebase.
- Enable continuous GPS-based route recording using Capacitor's Geolocation and Background Geolocation plugins, with auto-resume on app restart via `localStorage` state persistence.
- Build a REST API with Flask and Flask-JWT-Extended to securely manage user authentication, trip data, and analyst-specific endpoints, all protected with JWT token middleware.
- Implement an LSTM-based time-series model trained on historical hourly trip distribution to predict next-hour traffic volume with RMSE-based accuracy reporting.
- Compute real-time CO₂ emission estimates per trip using transport-mode-specific emission factors and surface these insights through both per-user and city-level dashboards.
- Provide interactive, region-searchable heatmaps of trip density using Leaflet.js and `leaflet.heat`, powered by a geographic bounding-box query against the trip database.
- Support full Android native packaging via Capacitor with local push notifications and native file system access for CSV report downloads.
- Implement OTP-based email verification during user registration using Gmail SMTP via Flask-Mail with a 5-minute expiry mechanism.

---

## Summary

Travel Coordinator is a cross-platform travel intelligence application with two independent but data-sharing modules: a **User Module** and an **Analyst Module**.

The **User Module** allows registered individuals to log trips by recording GPS routes in real time via the device's geolocation API. Each trip captures start and end coordinates, route waypoints as a JSON array, transport mode (Walk, Bike, Car, Bus, Auto, Cycle, Train), trip purpose, duration, distance (computed via the Haversine formula), cost, and companion count. Users access a personalized dashboard showing aggregate statistics — total trips, cumulative distance, total cost, dominant transport mode, and the most and least travelled geographic areas identified through reverse geocoding via the Nominatim API. Analytics pages render monthly trends and weekly summaries using Recharts, and the advanced analytics view supports custom date-range filtering with CSV export.

The **Analyst Module** provides a role-restricted smart city intelligence console. Analysts authenticate via a separate JWT-secured endpoint and gain access to platform-wide dashboards, interactive heatmaps, peak-hour detection by region, mode distribution charts, AI insights from the LSTM model, a retrain trigger, and a Smart City Simulation projection. All analyst views are geographically scoped — the analyst selects a city or region via Nominatim autocomplete, and all queries are filtered by a configurable ±10 km bounding box from the selected coordinates.

The backend is deployed at `https://travel-tracker-0ykf.onrender.com` and serves a React frontend delivered as a static build, also packaged as an Android APK under the bundle ID `com.raviteja.traveltracker`.

---

## Features

### Pre-Login / Authentication

- Animated welcome screen with navigation to login and registration flows.
- OTP-based email verification: a 6-digit OTP is generated, emailed via Gmail SMTP, stored in memory with a 5-minute expiry, and verified before account creation.
- OTP resend functionality with expiry refresh.
- JWT-based login with Bearer token storage in `localStorage`; all protected routes check token presence before rendering.
- `AuthRedirect` component prevents authenticated users from revisiting login or register pages.
- Dark mode toggle available globally via `ThemeContext` and persistent CSS variable switching.

### User Module (Post-Login)

- **Home Dashboard**: Displays total trips, total distance (km), total cost (₹), top transport mode, most travelled area, and least travelled area — all resolved with reverse geocoding. Quick-access buttons to Analytics and Smart Stats.
- **Live Trip Tracker**: Start/stop GPS recording with Capacitor Geolocation; route polyline rendered in real time on a Leaflet map with auto-centering. Haversine distance calculation accumulates incrementally as new GPS coordinates arrive. Trip state (active route, distance, start time) is persisted to `localStorage` to survive app background or restart.
- **Trip Logging Form**: Users specify transport mode, trip purpose, cost, and companion count before saving. Each trip is assigned a unique UUID-based `trip_no`.
- **Trip History**: List of all past trips with details; individual trip deletion via authenticated DELETE endpoint.
- **Local Notifications**: Capacitor Local Notifications fire at trip start and stop events on Android.
- **Analytics Dashboard**: Recharts-powered monthly trip trend (line chart), distance by month (bar chart), and weekly transport mode breakdown (pie chart). KPI cards for trips, distance, cost, and time over the last 7 days. AI-generated smart insights (cost warnings, fitness encouragement, CO₂ estimates, transport mode suggestions).
- **Advanced / Smart Analytics**: Date-range filtering for all metrics (trips, distance, time, cost, mode split, CO₂ footprint). AI-generated textual insights conditional on thresholds. CSV export — handled as a browser download on web and written to the Android Documents folder via Capacitor Filesystem on native.
- **User Profile**: View name, email, mobile, place; update profile photo (uploaded to backend `/uploads/` directory); change password with old-password verification.

### Analyst Module (Role-Restricted)

- **Analyst Login**: Separate credential set (`analyst@smartcity.com`) producing a role-scoped JWT with 8-hour expiry; stored as `analystToken` in `localStorage`.
- **Analyst Dashboard**: Platform-wide aggregate totals — trips, distance, cost, average duration — queried across all users.
- **Interactive Heatmap**: Region search via Nominatim autocomplete; configurable radius (km) bounding box query; heatmap rendered with `leaflet.heat` using a 6-color gradient (blue → cyan → green → yellow → orange → red); optional `CircleMarker` point overlay; automatic fly-to animation on region selection. Top transport modes displayed for the selected zone.
- **Peak Hour Analysis**: Region and date-aware query returning the busiest hour and trip count; transport mode distribution for the selected area and date.
- **Analytics Data**: Region and date-range filtered mode distribution and hourly distribution charts rendered in the analyst console.
- **AI Intelligence Engine (LSTM)**: Region and date-scoped AI insights including congestion score (0–100), risk level classification (Low / Medium / High), predicted next-hour traffic volume, average trip distance and duration, total CO₂ emission estimate, and a policy recommendation. Full metric definitions displayed inline.
- **Model Retrain**: On-demand LSTM model retraining against the full trip dataset, with updated weights saved to `traffic_lstm.keras`.
- **Smart City Simulation**: Projects trip growth at 18% increase rate; estimates CO₂ at projected volume; provides policy recommendation text.

---

## Technologies Used

### Programming Languages
- JavaScript (ES2023) — React frontend
- Python 3.10 — Flask backend and ML pipeline
- Java — Android native wrapper (`MainActivity.java`)
- TypeScript — Capacitor configuration (`capacitor.config.ts`)

### Libraries & Frameworks
- **React 19** — frontend SPA framework
- **React Router DOM 7** — client-side routing with protected and analyst-protected route wrappers
- **Recharts 3** — line, bar, and pie charts for analytics dashboards
- **Axios 1** — HTTP client with JWT interceptor for all API calls
- **Leaflet 1.9 + React-Leaflet 5** — interactive maps for route visualization and heatmaps
- **leaflet.heat** — heatmap layer plugin for Leaflet
- **html2canvas** — map screenshot capture for the `map_image` field
- **lodash** — utility functions
- **react-icons 5** — UI icon set

### Machine Learning / AI
- **TensorFlow 2.15 / Keras 2.15** — LSTM model for next-hour traffic volume prediction
- **Scikit-learn 1.4** — `MinMaxScaler` for data normalization; `mean_squared_error` for RMSE evaluation
- **NumPy 1.26** — array manipulation in the ML pipeline
- **SciPy 1.12** — scientific computing support

### Backend
- **Flask 3.1** — REST API framework
- **Flask-JWT-Extended 4.7** — JWT issuance and `@jwt_required` route protection for user endpoints
- **PyJWT 2.11** — manual JWT encoding/decoding for the analyst role-based auth decorator
- **Flask-Bcrypt 1.0** — password hashing and verification
- **Flask-SQLAlchemy 3.1** — ORM for `User` and `Trip` models
- **Flask-CORS 6.0** — cross-origin resource sharing for the React development server
- **Flask-Mail 0.10** — OTP dispatch via Gmail SMTP
- **geopy 2.4** — reverse geocoding via Nominatim for most/least travelled area resolution
- **requests 2.32** — HTTP calls to Nominatim search API from the analyst backend
- **h5py 3.10** — HDF5 file support for Keras model weights

### Frontend
- React functional components with hooks (`useState`, `useEffect`, `useRef`)
- `ThemeContext` for global dark/light mode with CSS variable injection
- `useBackHandler` custom hook for Android hardware back button interception via Capacitor App plugin

### Database
- **SQLite 3** — embedded relational database (`instance/app.db`)
- **SQLAlchemy 2.0** — ORM layer; tables: `user`, `trip`

### Deployment & DevOps
- **Capacitor 8** — cross-platform native bridge packaging the React build as an Android APK
- **Capacitor Geolocation** — foreground GPS coordinate access with permission request flow
- **@capacitor-community/background-geolocation** — continuous GPS tracking when app is backgrounded
- **Capacitor Local Notifications** — trip start/stop event notifications on Android
- **Capacitor Push Notifications** — push notification support
- **Capacitor Filesystem** — native file write for CSV export on Android (`Directory.Documents`)
- Backend deployed on **Render** (`https://travel-tracker-0ykf.onrender.com`)
- Android bundle ID: `com.raviteja.traveltracker`

### Security
- JWT Bearer token authentication for all user-protected API routes
- Role-scoped JWT for analyst routes with `role: analyst` claim validation in the `analyst_required` decorator
- `Flask-Bcrypt` password hashing (bcrypt default cost factor)
- In-memory OTP store with timestamp-based 5-minute expiry
- `secure_filename` enforcement on all uploaded profile photo files
- CSRF-safe stateless API (token-based, no session cookies)

---

## Dataset

The application does not rely on a pre-collected external dataset. All data is generated organically through user interaction and persisted in the SQLite database.

**Trip Data Schema (`Trip` model):**

| Field | Type | Description |
|---|---|---|
| `trip_no` | String | Unique UUID-prefixed trip identifier (`TRIP-xxxxxxxx`) |
| `user_id` | Integer (FK) | Owner user reference |
| `start_lat`, `start_lng` | Float | GPS coordinates of trip origin |
| `end_lat`, `end_lng` | Float | GPS coordinates of trip destination |
| `route` | JSON | Array of `{lat, lng}` waypoints recorded during trip |
| `mode` | String | Transport mode (Walk, Bike, Car, Bus, Auto, Cycle, Train) |
| `purpose` | String | Trip intent (Work, Study, Shopping, Leisure, etc.) |
| `distance` | Float | Haversine-computed distance in kilometres |
| `duration` | Float | Trip duration in minutes |
| `cost` | Float | User-entered travel cost in INR |
| `companions` | Integer | Number of co-travellers |
| `start_time`, `end_time` | DateTime | ISO 8601 trip timestamps |
| `trip_date` | Date | Calendar date of the trip |
| `map_image` | Text | Base64 or URL of captured Leaflet map screenshot |
| `frequency` | Integer | Repeat frequency indicator (default 1) |

**Seed Data:** The included `seed.py` script populates 9 representative dummy trips across 3 geographic clusters in Bengaluru (approximately 12.97°N, 77.59°E), covering all available transport modes and trip purposes, spanning different dates and durations, for testing the ML pipeline without requiring real user activity.

**LSTM Training Input:** Hourly trip counts aggregated from the `start_time` field of all database trips form the univariate time-series input to the LSTM. The sliding window length is 3 (three consecutive hours predict the fourth), normalized via `MinMaxScaler` to `[0, 1]` before training.

---

## System Architecture

```
Travel Coordinator
├── Frontend (React 19 SPA)
│   ├── src/
│   │   ├── App.js                   ← Root router — User + Analyst route trees
│   │   ├── pages/
│   │   │   ├── Welcome.js           ← Landing / splash screen
│   │   │   ├── Login.js             ← JWT user login
│   │   │   ├── Register.js          ← OTP-based registration
│   │   │   ├── Home.js              ← User dashboard (stats + geocoding)
│   │   │   ├── Trips.js             ← Live GPS tracker + Leaflet map
│   │   │   ├── History.js           ← Past trips list + delete
│   │   │   ├── Analytics.js         ← Monthly/weekly Recharts dashboard
│   │   │   ├── AdvancedAnalytics.js ← Date-range smart stats + CSV export
│   │   │   ├── Profile.js           ← User profile management
│   │   │   ├── AnalystLogin.jsx     ← Analyst credential login
│   │   │   ├── Dashboard.jsx        ← Analyst platform-wide stats
│   │   │   ├── HeatMap.js           ← leaflet.heat density visualization
│   │   │   ├── PeakHour.js          ← Peak hour detection by region + date
│   │   │   ├── Analytic.js          ← Analyst regional analytics charts
│   │   │   ├── Ai.js                ← LSTM AI insights + retrain trigger
│   │   │   └── Simulation.js        ← Smart City growth projection
│   │   ├── components/
│   │   │   ├── ProtectedRoute.js    ← User auth guard
│   │   │   ├── AuthRedirect.js      ← Prevent re-login for authenticated users
│   │   │   ├── AnalystLayout.jsx    ← Shared analyst page shell + nav
│   │   │   ├── BottomNav.js         ← Mobile bottom navigation bar
│   │   │   ├── DarkToggle.js        ← Light/dark mode switcher
│   │   │   └── Bottom.jsx           ← Footer component
│   │   ├── context/
│   │   │   └── ThemeContext.js      ← Global dark mode state provider
│   │   ├── hooks/
│   │   │   └── useBackHandler.js    ← Capacitor Android back button hook
│   │   └── services/
│   │       └── api.js               ← Axios instance with JWT interceptor
│   └── public/                      ← Static assets, manifest, robots.txt
│
├── Backend (Flask 3 REST API)
│   ├── app.py                       ← Main Flask app + all user-facing routes
│   ├── analyst.py                   ← Analyst Blueprint (heatmap, AI, peak hour)
│   ├── ml_model.py                  ← LSTM training, prediction, AI engine
│   ├── models.py                    ← SQLAlchemy User + Trip models
│   ├── config.py                    ← App config (DB URI, JWT key, SMTP)
│   ├── seed.py                      ← Development trip data seeder
│   ├── traffic_lstm.keras           ← Saved LSTM model weights
│   ├── uploads/                     ← Profile photo storage
│   └── instance/app.db              ← SQLite database
│
└── Android (Capacitor 8)
    ├── android/app/src/main/
    │   ├── AndroidManifest.xml      ← Permissions + activity config
    │   └── java/com/raviteja/
    │       └── traveltracker/
    │           └── MainActivity.java
    └── res/                         ← Splash screens + launcher icons
```

**Data Flow — User Trip Recording:**
Capacitor Geolocation `watchPosition` → incremental Haversine accumulation → Leaflet Polyline real-time render → Stop → POST `/api/trips` (JWT) → SQLAlchemy → SQLite → GET `/api/dashboard` → Nominatim reverse geocoding → React dashboard render.

**Data Flow — Analyst AI Insights:**
Region search → Nominatim coordinate resolution → POST `/api/analyst/ai-insights` (analyst JWT) → Bounding-box Trip query → `run_ai_engine()` → Hourly aggregation → LSTM train + predict → CO₂ calculation → Policy recommendation → JSON → `Ai.js` metric render.

---

## Model Workflow

### Data Preparation

1. All trips in the database are queried and their `start_time` fields are parsed to extract the hour of day.
2. An `hour_count` dictionary accumulates trip counts per hour (0–23).
3. Hourly counts form a univariate time series, reshaped to `(-1, 1)` and normalized using `MinMaxScaler` to `[0, 1]`.
4. Sliding window sequences of length 3 are created: each input `[h_t, h_t+1, h_t+2]` maps to target `h_t+3`. A minimum of 5 data points is required to proceed.

### Model Architecture

```
Layer         Output Shape    Parameters
─────────────────────────────────────────
Input         (None, 3, 1)    0
LSTM          (None, 50)      10,400
Dense         (None, 1)       51
─────────────────────────────────────────
Total trainable parameters: 10,451
```

Compiled with `Adam` optimizer and `mean_squared_error` loss. Trained for **40 epochs**.

### Training & Persistence

- `model.fit(X, y, epochs=40, verbose=0)` trains on the normalized sequences.
- Post-training predictions are generated on the training set; RMSE is computed and returned.
- The trained model is saved to `traffic_lstm.keras` using `model.save()` for subsequent prediction calls without retraining.

### Prediction (Inference)

1. The saved model is loaded with `keras.models.load_model(MODEL_PATH, compile=False)`.
2. The last 3 hourly counts from the live dataset are taken, normalized with a freshly fitted `MinMaxScaler`, and reshaped to `(1, 3, 1)`.
3. `model.predict()` returns the scaled output, inverse-transformed to the original count scale.
4. The result is clipped to a minimum of 0 and returned as an integer — the predicted trip volume for the next hour.

### AI Engine (`run_ai_engine`)

The `run_ai_engine(trips)` function in `ml_model.py` orchestrates the full analytics pipeline:

- Aggregates hourly trip counts and mode distribution from the filtered trip list.
- Calls `train_model()` to fit the LSTM and obtain RMSE accuracy.
- Calls `predict_next()` for the next-hour forecast value.
- Computes `congestion_score` as the total sum of all hourly counts, capped at 100.
- Maps congestion score to risk level: `< 30` → Low, `30–69` → Medium, `≥ 70` → High.
- Calculates total CO₂ using per-mode emission factors (Car: 0.21 kg/km, Bike: 0.09, Bus: 0.05, Train: 0.04, Walk/Cycle: 0.00).
- Generates a textual policy recommendation based on dominant transport mode and CO₂ thresholds.
- Returns a complete JSON payload including `metric_meanings` definitions for inline UI rendering.

---

## Implementation Details

**Dual-Role JWT Architecture:** User authentication uses `Flask-JWT-Extended`'s `create_access_token` with the user's integer database ID as identity, verified via `@jwt_required()` on protected routes. The analyst system uses `PyJWT` directly to encode a `{"role": "analyst"}` claim with an 8-hour expiry, verified in the custom `analyst_required` decorator by decoding and inspecting the role field. The two token systems operate independently, stored in `localStorage` under the keys `token` and `analystToken` respectively.

**OTP Registration Flow:** On form submission, the frontend POSTs to `/api/send-otp`; the backend generates a 6-digit random integer, stores it in a Python in-memory dictionary keyed by email with a `datetime` expiry 5 minutes ahead and the full registration payload, then dispatches the OTP via Gmail SMTP through Flask-Mail. On OTP submission, `/api/verify-otp` validates the expiry and equality before creating the `User` database record and deleting the OTP entry. A `/api/resend-otp` endpoint refreshes the OTP and expiry for the same email session. The database row is only created after successful OTP verification.

**GPS Tracking Engine:** `Geolocation.requestPermissions()` is called before initiating any watch. `Geolocation.watchPosition()` fires on each significant position change; new `{lat, lng}` points are appended to the `route` state array and the incremental Haversine distance is added to `distance` state. All tracking state (`trip_active`, `trip_route`, `trip_distance`, `trip_start_time`) is mirrored to `localStorage` on each update so that an interrupted session can auto-resume on component mount via the `useEffect` restore block.

**Heatmap Bounding Box Query:** Analyst-specified radius in kilometres is converted to degrees via `delta = radius / 111`. SQLAlchemy filters `Trip.start_lat.between(lat - delta, lat + delta)` and equivalently for longitude. Heat points are aggregated by rounding coordinates to 3 decimal places (approximately 100 m spatial resolution) and counted; intensity values are multiplied by 8 for `leaflet.heat` visibility. The 6-stop gradient maps trip density from low (blue) to high (red).

**Reverse Geocoding for Dashboard:** The `/api/dashboard` endpoint identifies the most and least frequently visited coordinate clusters by rounding all route waypoints to 3 decimal places and counting occurrences in a dictionary. The extreme coordinates are resolved via `geopy.geocoders.Nominatim`, preferring `suburb` → `neighbourhood` → `road` → `village` → `town` → `city` from the address fields, formatted as `"Area, City"`.

**CSV Export with Capacitor Filesystem:** On native Android (`Capacitor.isNativePlatform() === true`), the response blob is converted to base64 via a `FileReader` `readAsDataURL` promise wrapper (splitting on the comma to extract the base64 payload), then written to `Directory.Documents` with a timestamp-suffixed filename via `Filesystem.writeFile()`. On web, a standard `<a>` element with a `createObjectURL` blob URL triggers the browser's download dialog.

**`useBackHandler` Hook:** A custom React hook registers a Capacitor `App.addListener("backButton", ...)` listener on mount and removes it on unmount. This prevents the Android hardware back button from exiting the app when on protected pages, instead navigating to the previous route in React Router's history.

---

## Results

### LSTM Traffic Prediction Model

The LSTM model is trained on-demand from live trip data per analyst request. Accuracy metrics are returned in the API response and rendered directly in the AI page UI.

| Parameter | Value |
|---|---|
| Model Architecture | Input(3,1) → LSTM(50, relu) → Dense(1) |
| Training Epochs | 40 |
| Optimizer | Adam |
| Loss Function | Mean Squared Error (MSE) |
| Reported Accuracy Metric | RMSE (Root Mean Square Error) |
| Sequence Window Length | 3 consecutive hourly counts |
| Minimum Data Requirement | 5 data points |
| Model Persistence | `traffic_lstm.keras` (Keras native format) |

**Congestion Risk Classification:**

| Congestion Score (0–100) | Risk Level |
|---|---|
| 0 – 29 | Low |
| 30 – 69 | Medium |
| 70 – 100 | High |

**CO₂ Emission Factors (kg CO₂ per km):**

| Transport Mode | Factor |
|---|---|
| Car | 0.21 |
| Bike | 0.09 |
| Bus | 0.05 |
| Train | 0.04 |
| Cycle | 0.00 |
| Walk | 0.00 |
| Other (default) | 0.15 |

**Smart City Simulation:**
Projected trip growth is modelled at a **1.18× (18%) increase** over the current total trip count. CO₂ projection is computed at the Car emission factor (0.21 kg/km) applied to the projected trip volume. A static policy recommendation is returned: increase public transport capacity and optimize traffic signal timings.

**User-Facing Smart Insights Thresholds:**

| Condition | Insight |
|---|---|
| 7-day cost > ₹500 | Suggest switching to public transport |
| 7-day travel time > 600 min | Suggest route optimization |
| Walk mode trips > 5 in 7 days | Fitness encouragement |
| CO₂ emission > 0 | Carbon footprint estimate in kg |
| Car trips > Bus trips (range) | Suggest bus for savings |
| Carbon < 2 kg and trips > 0 | Eco-friendly travel acknowledgement |

---

## Installation

### Prerequisites

- Node.js 18+ and npm
- Python 3.10+
- Android Studio (required only for Android APK build)
- A Gmail account with App Password configured for SMTP

### Step 1: Clone the Repository

```bash
git clone https://github.com/harisaigithub/Travel-coordinator-React-APP_RV.git
cd Travel-coordinator-React-APP_RV
```

### Step 2: Backend Setup

```bash
cd backend
python -m venv venv

# Windows
venv\Scripts\activate
# macOS / Linux
source venv/bin/activate

pip install -r requirements.txt
```

Update `config.py` with your credentials:

```python
class Config:
    SECRET_KEY = "your-strong-secret-key"
    JWT_SECRET_KEY = "your-jwt-secret-key"
    SQLALCHEMY_DATABASE_URI = "sqlite:///app.db"
    MAIL_USERNAME = "your-email@gmail.com"
    MAIL_PASSWORD = "your-gmail-app-password"
    MAIL_DEFAULT_SENDER = "your-email@gmail.com"
```

Initialize the database and optionally seed demo data:

```bash
python -c "from app import app, db; app.app_context().__enter__(); db.create_all()"
python seed.py   # optional — seeds 9 demo trips for ML testing
```

Start the backend server:

```bash
python app.py
# API available at http://0.0.0.0:5000
```

### Step 3: Frontend Setup

```bash
# From the project root directory
npm install
```

Update the API base URL in `src/services/api.js` for local development:

```javascript
const API = axios.create({
  baseURL: "http://localhost:5000/api"
});
```

Start the React development server:

```bash
npm start
# Opens http://localhost:3000
```

### Step 4: Android Build (Optional)

```bash
# Build the React production bundle
npm run build

# Sync the build with the Capacitor Android project
npx cap sync android

# Open in Android Studio for build and deployment
npx cap open android
```

Alternatively, run directly on a connected device or emulator:

```bash
npx cap run android
```

---

## Usage

### User Workflow

1. Open the app at `http://localhost:3000` or the deployed Render URL.
2. On the Welcome screen, tap **Register** and complete the form (name, email, mobile, place, password).
3. Enter the 6-digit OTP dispatched to your email; tap **Verify** to activate the account.
4. Log in with your credentials.
5. The **Home** dashboard displays cumulative trip statistics and geocoded area insights.
6. Navigate to **Trips** to start live GPS recording — select transport mode, tap start, travel, then stop; enter cost and companions, then save the trip.
7. Review all past trips in **History** with the option to delete individual entries.
8. Access **Analytics** for monthly trend charts (line, bar, pie) and a weekly KPI summary.
9. Access **Smart Stats** for date-range analytics, CO₂ footprint insights, smart tips, and CSV report download.
10. Update your profile photo and change password under **Profile**.

### Analyst Workflow

1. Navigate to `/analyst/login`.
2. Log in with credentials: `analyst@smartcity.com` / `SmartCity@123`.
3. The **Analyst Dashboard** displays platform-wide aggregate totals.
4. Go to **Heatmap** → search a city → configure search radius → view trip density gradient and dominant transport modes.
5. Go to **Peak Hour** → search a region → optionally select a date → view the busiest hour and mode breakdown.
6. Go to **AI Insights** → search a region → optionally filter by date → view LSTM congestion forecast, CO₂ estimate, and policy recommendation; tap **Retrain Model** to update weights on the latest data.
7. Go to **Simulation** to view projected trip volume growth and associated CO₂ estimate.

---

## Future Enhancements

- **Route Optimization Suggestions:** Integrate OSRM or the Google Directions API to suggest faster or lower-emission alternative routes based on historical peak-hour data along the user's typical corridors.
- **PostgreSQL with PostGIS:** Replace SQLite with PostgreSQL and the PostGIS extension to enable native geospatial radius queries (`ST_DWithin`), multi-user concurrency, and production-grade indexing.
- **Real-Time WebSocket Dashboard:** Push live trip count increments and congestion alerts to the analyst heatmap via Flask-SocketIO, eliminating the need for manual page refresh.
- **iOS Capacitor Support:** Extend the native build target to iOS, resolving platform-specific permission handling for background geolocation and the `Documents` directory.
- **Multi-Step LSTM Forecasting:** Replace the single-step predictor with a Seq2Seq LSTM incorporating attention mechanisms to produce 24-hour traffic forecasts rather than one-hour-ahead predictions.
- **Offline-First PWA:** Implement a Service Worker with IndexedDB caching so users can record trips offline, with background sync uploading data to the backend once connectivity is restored.
- **Carbon Offset Gamification:** Introduce a points system rewarding Walk and Cycle trips, with redeemable in-app badges displayed on the user profile.
- **Auto Trip Purpose Detection:** Use a lightweight NLP classifier trained on historical `purpose` labels and destination coordinates to auto-suggest trip purpose when starting a new trip near a previously recorded endpoint.
- **Multi-Tenant Analyst System:** Scope analyst access per city, so separate city planners can only query trips originating within their administrative boundary.
- **Trip Comparison View:** Allow users to compare two custom date ranges side-by-side with overlaid Recharts series for distance, cost, and mode split.

---

## Contributing

Contributions are welcome from developers interested in urban mobility analytics, React, Flask, or LSTM time-series modelling.

1. **Fork** the repository on GitHub.
2. **Clone** your fork locally:
   ```bash
   git clone https://github.com/<your-username>/Travel-coordinator-React-APP_RV.git
   ```
3. **Create a feature branch**:
   ```bash
   git checkout -b feature/your-feature-name
   ```
4. **Make your changes**, following the existing code conventions; add comments for any ML model changes.
5. **Commit with a clear message**:
   ```bash
   git commit -m "feat: add PostgreSQL support with PostGIS radius queries"
   ```
6. **Push** your branch:
   ```bash
   git push origin feature/your-feature-name
   ```
7. **Open a Pull Request** against the `main` branch with a detailed description.

Never commit API keys, SMTP passwords, or JWT secrets — use environment variables for all sensitive configuration.

---

## License

```
MIT License

Copyright (c) 2026 Ravi Teja

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