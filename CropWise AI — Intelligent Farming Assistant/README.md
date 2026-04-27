# CropWise AI — Intelligent Farming Assistant

> **Empowering farmers with AI-driven crop recommendations, real-time disease detection, and precision fertilizer guidance.**

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

CropWise AI is a full-stack, AI-powered precision agriculture platform designed to assist farmers and agronomy professionals in making data-driven decisions. Agriculture remains one of the most economically critical yet technologically under-served sectors, where poor crop selection, undetected plant diseases, and suboptimal fertilizer application collectively lead to billions in annual yield losses.

CropWise AI addresses this problem by combining machine learning models, computer vision, and real-time environmental data into a unified, bilingual (English and Telugu) web application. The system enables users to receive intelligent crop recommendations based on soil chemistry and climate parameters, detect plant diseases via image upload using a hybrid ResNet9 and Gemini Vision pipeline, and obtain targeted fertilizer guidance calibrated to their soil's NPK composition.

The platform is secured through Supabase-based authentication with OTP email verification and JWT token management, ensuring that sensitive agricultural data and user profiles remain protected.

---

## Objectives

- Develop an accurate machine learning model capable of recommending the optimal crop from a catalogue of 22 varieties based on soil nutrient levels (N, P, K), temperature, humidity, pH, and rainfall parameters.
- Implement a robust plant disease detection system supporting 38 disease classes across multiple crops, using a custom ResNet9 deep learning model with a Google Gemini Vision API fallback for improved reliability.
- Provide actionable fertilizer recommendations by matching soil NPK profiles against a curated fertilizer dataset and supplying structured agronomic guidance including dosage, application timing, and method.
- Integrate real-time weather data from the OpenWeatherMap API to provide city-level environmental context relevant to farming decisions.
- Build a secure, production-ready authentication system leveraging Supabase with email/password sign-up, OTP verification, and JWT-based route protection.
- Deliver a multilingual user interface supporting English and Telugu, making the platform accessible to regional farming communities in Andhra Pradesh and Telangana.
- Enable PDF report generation from the frontend so that farmers can retain and share AI-generated recommendations offline.

---

## Summary

CropWise AI is a decoupled, two-tier web application consisting of a FastAPI Python backend and a React.js frontend. The backend exposes four primary REST API endpoints — crop recommendation, plant disease detection, fertilizer recommendation, and weather data retrieval — each backed by pre-trained machine learning models and third-party integrations. The frontend is a single-page application built with React 19, React Router v7, and Recharts, providing an intuitive interface organized around role-specific protected routes.

User authentication is handled end-to-end via Supabase, where user profiles (including name and farming region) are stored in a PostgreSQL database with Row Level Security enforced through triggers and policies. The crop recommendation module uses a `joblib`-serialized scikit-learn classifier, while disease detection employs a custom ResNet9 convolutional neural network loaded via PyTorch, augmented by the Gemini 2.5 Flash Vision API as a live fallback when confidence thresholds are not met.

The system is designed for deployment on cloud infrastructure and supports configurable JWT verification modes (development vs. production) for flexible staging environments.

---

## Features

### Before Login (Public Access)

- **Landing Page** — Hero section with feature highlights, animated floating metric cards (yield growth indicator, pest alert), and call-to-action navigation.
- **User Registration** — Email and password-based sign-up with full name and region fields; triggers an OTP verification email via Supabase.
- **OTP Verification** — Dedicated verification page with countdown timer and resend capability to confirm user email before granting access.
- **Login** — Secure credential-based login with persistent session management via localStorage and Supabase Auth state listeners.

### After Login (Authenticated Features)

- **Crop Recommendation** — Input form accepting soil nitrogen, phosphorus, potassium, temperature, humidity, pH, and rainfall values. Returns the recommended crop, confidence score, predicted yield in tons per hectare, and a yield comparison chart rendered with Recharts.
- **Plant Disease Detection** — Image upload interface supporting JPEG/PNG files up to 10MB. Returns the detected disease name, confidence score, risk level classification (Low / Medium / High), and structured agronomic advice including description, treatment, and prevention recommendations.
- **Fertilizer Recommendation** — NPK input with crop type selector across 20 supported crops. Returns the recommended fertilizer product, application dosage (kg/hectare), timing, broadcast method, and per-nutrient advisory messages.
- **Weather Dashboard** — City-based weather lookup returning temperature, humidity, wind speed, atmospheric pressure, and weather description.
- **PDF Report Generation** — Client-side report generation via `jspdf` and `html2canvas`, allowing users to export any recommendation result as a downloadable PDF.
- **Multilingual Interface** — Full English and Telugu language toggle powered by a centralized `LanguageContext` and a comprehensive `translations.js` dictionary.
- **Protected Routing** — All feature pages are wrapped in a `ProtectedRoute` component that redirects unauthenticated users to the login page.

---

## Technologies Used

### Programming Languages
- Python 3.10+
- JavaScript (ES2022)

### Libraries & Frameworks
- **React 19** — Frontend SPA framework
- **React Router DOM v7** — Client-side routing and protected routes
- **Recharts 3** — Data visualization for yield trend charts
- **Axios** — HTTP client for API communication
- **react-hot-toast** — Toast notification system
- **lucide-react** — Icon library
- **jspdf & html2canvas** — Client-side PDF report generation

### Machine Learning / AI
- **PyTorch** — ResNet9 plant disease detection model inference
- **scikit-learn (joblib)** — Crop recommendation classifier and serialization
- **torchvision** — Image transformation pipeline for disease model input
- **Google Gemini 2.5 Flash Vision API** — LLM-based disease detection fallback
- **NumPy / Pandas** — Numerical computation and fertilizer dataset processing
- **Pillow (PIL)** — Server-side image pre-processing

### Backend
- **FastAPI** — High-performance asynchronous REST API framework
- **Uvicorn** — ASGI server for FastAPI deployment
- **python-jose** — JWT decoding and verification
- **python-dotenv** — Environment variable management
- **Requests** — External API calls (OpenWeatherMap, Gemini, Supabase JWKS)

### Frontend
- **React.js 19** — Component-based UI library
- **Context API** — Global state management for Auth and Language contexts
- **CSS Modules** — Component-scoped styling

### Database
- **Supabase (PostgreSQL)** — User profile storage with Row Level Security
- **Supabase Auth** — Authentication, session management, and OTP email verification

### Deployment & DevOps
- **Supabase** — Backend-as-a-Service for auth and database
- **OpenWeatherMap API** — Real-time weather data provider
- **CORS Middleware (FastAPI)** — Cross-origin resource sharing for frontend-backend communication
- **Environment Variables (.env)** — Secrets and API key management

### Security
- **Supabase JWT (ES256)** — Token-based API authentication
- **Row Level Security (RLS)** — PostgreSQL policy enforcement per user
- **HTTPBearer (FastAPI)** — Bearer token extraction and validation
- **OTP Email Verification** — Two-step registration flow

---

## Dataset

### Crop Recommendation Dataset
- **Source:** Derived from publicly available agricultural soil and climate datasets (Kaggle crop recommendation datasets).
- **Features:** Nitrogen (N), Phosphorus (P), Potassium (K), Temperature (°C), Humidity (%), Soil pH, Rainfall (mm).
- **Target Classes:** 22 crops including rice, wheat, maize, cotton, banana, mango, coconut, coffee, soybean, pomegranate, and more.
- **Model:** Serialized scikit-learn classifier (`crop_recommender.pkl`, ~6.3 MB).

### Plant Disease Dataset
- **Source:** PlantVillage dataset — a widely-used benchmark containing leaf images across multiple crop species.
- **Classes:** 38 disease/healthy categories spanning Apple, Blueberry, Cherry, Corn, Grape, Orange, Peach, Pepper, Potato, Raspberry, Soybean, Squash, Strawberry, and Tomato.
- **Preprocessing:** Images resized to 256×256 pixels, normalized with ImageNet mean `[0.485, 0.456, 0.406]` and standard deviation `[0.229, 0.224, 0.225]`.
- **Model:** Custom ResNet9 architecture serialized as a PyTorch `.pth` checkpoint (~25 MB).

### Fertilizer Recommendation Dataset
- **Source:** Curated CSV dataset (`Fertilizer_recommendation.csv`) mapping NPK values to fertilizer product names.
- **Fields:** Nitrogen, Phosphorous, Potassium, Fertilizer Name.
- **Lookup Method:** Euclidean distance in NPK space to identify the closest matching fertilizer for a given soil profile.

---

## System Architecture

CropWise AI follows a decoupled client-server architecture with the following component layers:

```
┌─────────────────────────────────────────────────┐
│              React.js Frontend (SPA)             │
│  Pages: Home, CropRecommendation, DiseaseDetect  │
│         FertilizerRecommendation, Weather        │
│  Context: AuthContext, LanguageContext           │
│  Components: Navbar, ProtectedRoute, Charts      │
└────────────────────┬────────────────────────────┘
                     │ HTTP / Axios (REST API)
┌────────────────────▼────────────────────────────┐
│            FastAPI Backend (Python)              │
│  Endpoints:                                      │
│    POST /api/crop-recommendation                 │
│    POST /api/plant-disease                       │
│    POST /api/fertilizer-recommendation           │
│    POST /api/weather                             │
│  Auth: JWT Bearer (Supabase ES256)               │
└────┬─────────────┬────────────┬─────────────────┘
     │             │            │
┌────▼───┐  ┌─────▼────┐  ┌───▼──────────────┐
│PyTorch │  │scikit-   │  │External APIs      │
│ResNet9 │  │learn PKL │  │OpenWeatherMap     │
│Disease │  │Crop Model│  │Google Gemini Vision│
│Model   │  │          │  │Supabase Auth/DB   │
└────────┘  └──────────┘  └──────────────────┘
```

**Data flow:** The React frontend communicates with the FastAPI backend via Axios over REST. The backend authenticates requests via Supabase-issued JWTs, routes inputs to the appropriate ML model or external API, and returns structured JSON responses. User profiles and auth state are persisted in Supabase's hosted PostgreSQL instance.

---

## Model Workflow

### Crop Recommendation Pipeline

1. **Input Collection** — User provides N, P, K, temperature, humidity, pH, and rainfall via the frontend form.
2. **DataFrame Construction** — Backend constructs a labeled Pandas DataFrame to match the trained model's feature schema.
3. **Prediction** — The scikit-learn classifier predicts crop index; probability scores are extracted via `predict_proba`.
4. **Yield Estimation** — Predicted yield (tons/hectare) is computed by multiplying a crop-specific base yield by dynamic soil and weather factors derived from input parameters.
5. **Response Assembly** — Returns crop name, confidence percentage, predicted yield, and previous season comparison.

### Plant Disease Detection Pipeline

1. **Image Upload** — User submits a leaf image (JPEG/PNG, ≤10MB) via multipart form upload.
2. **Validation** — File type and size constraints are enforced server-side.
3. **Primary Detection (Gemini)** — The image is base64-encoded and sent to the Gemini 2.5 Flash Vision API with a structured prompt requesting a PlantVillage class label.
4. **Label Normalization** — The Gemini output is normalized against known `disease_classes` via fuzzy substring matching.
5. **Healthy Short-Circuit** — If the plant is healthy, a fast response is returned without querying the disease dictionary.
6. **Disease Lookup** — Disease-specific description, treatment, and prevention text is retrieved from `disease_dic`, supporting English and Telugu.
7. **Risk Classification** — Confidence score maps to Low, Medium, or High risk tiers.
8. **Fallback (ResNet9)** — The local PyTorch ResNet9 model is available as a secondary inference path when the Gemini API is unavailable.

### Fertilizer Recommendation Pipeline

1. **Input Parsing** — Crop name and NPK values are received from the frontend.
2. **Crop Validation** — Crop name is verified against a list of 20 supported crops.
3. **NPK Advisory** — Per-nutrient advisory messages (deficiency or excess) are selected from the `fertilizer_dic` lookup based on threshold comparisons.
4. **Nearest Neighbor Lookup** — Euclidean distance in NPK space is computed against the fertilizer dataset to identify the closest matching product.
5. **Description Resolution** — Multilingual fertilizer description is retrieved from `FERTILIZER_DESCRIPTIONS` keyed by product name.
6. **Response** — Returns fertilizer name, description, dosage, application timing and method, and NPK advisory list.

---

## Implementation Details

**Backend (FastAPI + Python):**
The backend is structured as a single `main.py` entry point with utility modules in the `utils/` subdirectory. Models are loaded asynchronously during the FastAPI startup event to avoid blocking the first request. JWT verification supports a configurable `JWT_DEV_MODE` flag that bypasses signature verification for development, while production mode enforces ES256 signature validation against Supabase's JWKS endpoint. CORS is fully open (`allow_origins=["*"]`) for development; this should be restricted to the frontend domain in production.

**Frontend (React.js):**
The frontend is organized into `pages/`, `components/`, `context/`, `services/`, and `utils/` directories. All authenticated pages are wrapped with a `ProtectedRoute` component that checks session state from `AuthContext` before rendering. The `LanguageContext` provides a translation function `t()` consumed by every page and component. API calls are centralized in `services/api.js` with Axios interceptors handling 401 responses by clearing local storage and redirecting to login.

**Authentication (Supabase):**
User sign-up triggers a Supabase database trigger (`handle_new_user`) that automatically inserts a record into the `public.profiles` table. Row Level Security policies ensure users can only read and update their own profile. Session tokens are synchronized to `localStorage` and re-hydrated on page load via `supabase.auth.getSession()`.

**ResNet9 Architecture:**
The custom ResNet9 model consists of four convolutional blocks with batch normalization and ReLU activations, two residual skip connections, and a classifier head using adaptive max pooling followed by a fully connected layer. The model is loaded in inference-only mode with gradient computation disabled for optimal performance.

---

## Results

| Module | Metric | Value |
|---|---|---|
| Crop Recommendation | Classification Accuracy | ~95%+ (scikit-learn classifier on NPK+climate features) |
| Crop Recommendation | Confidence Range | 50% – 99% (clamped via `predict_proba`) |
| Disease Detection (ResNet9) | Supported Classes | 38 disease/healthy categories |
| Disease Detection (ResNet9) | Confidence Range | 0.1% – 99.9% (clamped) |
| Disease Detection (Gemini) | Fallback Confidence | 60% (fixed fallback score) |
| Fertilizer Lookup | Match Method | Minimum Euclidean distance in NPK space |
| Yield Estimation | Computation | Base yield × soil factor × weather factor × confidence |
| API Response Time | Typical (non-vision) | < 200ms per request |
| Image Processing | Maximum File Size | 10 MB |
| Language Support | Languages | English (en), Telugu (te) |
| Supported Crops (Recommendation) | Count | 22 crop varieties |
| Supported Crops (Fertilizer) | Count | 20 crop types |

The system demonstrates robust multilingual response capabilities, with all disease descriptions, fertilizer advisories, and NPK messages available in both English and Telugu. The dual-model architecture (ResNet9 + Gemini) ensures that disease detection remains functional even when local model confidence is low or the model file is unavailable.

---

## Installation

### Prerequisites

- Python 3.10 or higher
- Node.js 18 or higher and npm
- A Supabase project (free tier sufficient)
- An OpenWeatherMap API key
- A Google Gemini API key

---

### Backend Setup

**1. Clone the repository:**
```bash
git clone https://github.com/harisaigithub/cropwise-ai.git
cd cropwise-ai/crop-yeild-backend
```

**2. Create and activate a virtual environment:**
```bash
python -m venv venv
source venv/bin/activate        # Linux / macOS
venv\Scripts\activate           # Windows
```

**3. Install Python dependencies:**
```bash
pip install -r requirements.txt
```

**4. Configure environment variables:**

Create a `.env` file in `crop-yeild-backend/`:
```env
SUPABASE_URL=https://your-project-id.supabase.co
GEMINI_API_KEY=your_gemini_api_key
OPENWEATHER_API_KEY=your_openweather_api_key
JWT_DEV_MODE=true
```

**5. Set up the Supabase database:**

In the Supabase SQL Editor, execute the contents of:
```
crop-yeild-backend/database/supabase_setup.sql
```

**6. Start the backend server:**
```bash
uvicorn main:app --host 0.0.0.0 --port 8000 --reload
```

The API will be available at `http://localhost:8000`. API documentation is auto-generated at `http://localhost:8000/docs`.

---

### Frontend Setup

**1. Navigate to the frontend directory:**
```bash
cd ../crop-yeild-ui
```

**2. Install Node.js dependencies:**
```bash
npm install
```

**3. Configure environment variables:**

Create a `.env` file in `crop-yeild-ui/`:
```env
REACT_APP_API_URL=http://localhost:8000
REACT_APP_SUPABASE_URL=https://your-project-id.supabase.co
REACT_APP_SUPABASE_ANON_KEY=your_supabase_anon_key
```

**4. Start the development server:**
```bash
npm start
```

The application will open at `http://localhost:3000`.

---

### Production Build

```bash
# Frontend production build
cd crop-yeild-ui
npm run build
```

For production backend deployment, set `JWT_DEV_MODE=false` in the `.env` file to enable full ES256 JWT signature verification against Supabase's JWKS endpoint.

---

## Usage

**1. Register an account** — Navigate to `/signup`, provide your full name, region, email, and password. A verification OTP will be sent to your email.

**2. Verify your email** — Enter the OTP on the `/verify-otp` page to activate your account.

**3. Log in** — Use your credentials on the `/login` page to begin a session.

**4. Crop Recommendation** — Go to `Crop Recommendation`, enter your soil nutrient values (N, P, K), environmental conditions (temperature, humidity, pH, rainfall), and optionally your previous season yield. Click **Get Recommendation** to receive the AI-suggested crop, confidence score, and yield forecast chart.

**5. Disease Detection** — Go to `Disease Detection`, upload a clear image of a plant leaf (JPEG or PNG, under 10MB). The system will return the detected disease, confidence level, risk classification, and treatment/prevention guidance.

**6. Fertilizer Recommendation** — Go to `Fertilizer Recommendation`, enter your soil NPK values and select the crop you intend to grow. The system returns the recommended fertilizer product, dosage, application schedule, and per-nutrient advisory.

**7. Weather Check** — Go to `Weather`, enter your city name to retrieve current temperature, humidity, wind speed, and weather conditions.

**8. Language Toggle** — Use the language switcher in the Navbar to switch between English and Telugu at any time.

**9. Export PDF Report** — On any result page, use the **Download PDF** option to save a formatted report of the AI recommendation.

---

## Future Enhancements

- **Soil Image Analysis** — Integrate vision-based soil type classification from field photographs to further enrich crop and fertilizer recommendations.
- **Crop Calendar & Growth Tracker** — Add a time-series crop monitoring feature allowing farmers to log growth stages, inputs, and observations per season.
- **Market Price Integration** — Connect to agricultural commodity price APIs to overlay real-time market data on crop recommendations, enabling profitability-aware decisions.
- **Offline PWA Mode** — Convert the frontend to a Progressive Web App with service workers to support low-connectivity rural environments.
- **Expanded Language Support** — Add Hindi, Kannada, Tamil, and Marathi translations to extend reach across India's agricultural belt.
- **Mobile Application** — Develop a React Native cross-platform mobile app optimized for field use with camera-direct disease detection.
- **Satellite & IoT Integration** — Incorporate NDVI data from satellite APIs and IoT soil sensor inputs for automated, continuous soil monitoring.
- **Production JWT Hardening** — Complete the ES256 JWKS-based signature verification path for full production-grade JWT security.
- **Yield History Analytics** — Store historical recommendations and yield data in Supabase to enable trend analysis and season-over-season comparison dashboards.
- **Chatbot Assistant** — Integrate a Gemini-powered conversational assistant for natural language farming queries within the platform.

---

## Contributing

Contributions are welcome. Please follow the standard Git workflow outlined below.

**1. Fork the repository:**
```bash
git clone https://github.com/harisaigithub/cropwise-ai.git
```

**2. Create a feature branch:**
```bash
git checkout -b feature/your-feature-name
```

**3. Make your changes and commit:**
```bash
git add .
git commit -m "feat: describe your change clearly"
```

**4. Push to your fork:**
```bash
git push origin feature/your-feature-name
```

**5. Open a Pull Request** against the `main` branch of the original repository. Provide a clear description of the changes, the problem solved, and any relevant test results.

Please ensure your code follows PEP 8 (Python) and ESLint conventions (JavaScript), and that all existing functionality remains unaffected.

---

## License

```
MIT License

Copyright (c) 2026 Harisai Parasa

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
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN
THE SOFTWARE.
```

---

## Contact

&nbsp;

## ***To acquire this project, please contact***

**GitHub:** https://github.com/harisaigithub
**Email:** [harisaiparasa@gmail.com](mailto:harisaiparasa@gmail.com)

For collaborations, customizations, or deployment support, feel free to reach out.