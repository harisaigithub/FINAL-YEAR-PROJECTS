# MilletChain — AI-Powered Millet Agri-Commerce Platform

> **A full-stack, multilingual agricultural marketplace connecting millet farmers and buyers through AI-driven price prediction, real-time order management, and demand forecasting — built to digitise India's millet value chain.**

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

Millets — including Ragi (Finger Millet), Jowar (Sorghum), Bajra (Pearl Millet), Foxtail, Proso, and Little Millet — are nutritionally dense, drought-resistant crops central to food security in rural India. Despite their agronomic significance and government-backed promotion under the International Year of Millets (2023), millet farmers continue to face structural disadvantages: fragmented market access, price opacity, inefficient buyer-seller matching, and lack of data-driven decision support.

MilletChain is a Django-based agricultural commerce platform that directly addresses these challenges. It provides millet farmers a digital storefront to list and manage their produce, buyers a searchable marketplace to discover and procure millet varieties, and administrators an analytics command centre powered by a machine learning price prediction pipeline. The platform supports three languages — English, Hindi, and Telugu — ensuring accessibility for regional farming communities across India.

The system integrates a scikit-learn regression pipeline trained on 160,000 records of synthetic millet transaction data to deliver AI-powered pricing suggestions, a rule-based demand forecasting view, and a live notification engine that keeps all stakeholders informed in real time.

---

## Objectives

- Design and implement a role-based, multi-stakeholder web platform serving three user types: Farmers (Millet Producers), Buyers (Millet Consumers), and Platform Administrators.
- Build a secure authentication system with email-based OTP verification, 10-minute token expiry, and role-specific post-login routing.
- Provide farmers with a complete product lifecycle management interface: create, edit, delete, and track availability of millet listings with image upload support.
- Enable buyers to discover millet products through a searchable, filterable marketplace and initiate purchase requests with quantity validation and automatic total price computation.
- Implement a four-stage order workflow (Pending → Accepted → Rejected → Completed) with real-time notification dispatch to both farmers and buyers at every status transition.
- Train and deploy a scikit-learn ML pipeline for millet price prediction based on millet type, quantity, location, and season, with a historical average fallback for edge cases.
- Support multilingual interfaces in English, Hindi, and Telugu using Django's i18n framework with compiled `.po`/`.mo` locale files.
- Provide administrators with an analytics dashboard covering platform-wide sales, ML model performance metrics, user activity feeds, and the ability to upload new training datasets via CSV.

---

## Summary

MilletChain is a modular Django 5.2 web application composed of five Django apps: `accounts` (authentication and user management), `products` (millet listings and marketplace), `orders` (purchase request and fulfilment workflow), `analytics` (ML price prediction, demand forecasting, and admin intelligence), and `notifications` (in-app alert system). Each app maintains its own models, views, URL configuration, and migration history.

The ML subsystem consists of a pre-trained scikit-learn pipeline (`millet_price_model.pkl`, ~31 MB) loaded at application startup via `joblib`. It accepts a structured input DataFrame with four features — `millet_type`, `quantity_kg`, `location`, and `season` — and returns a predicted price per kilogram. The model was trained on a 160,000-row dataset covering six millet types, multiple Indian state locations, and three seasons. A graceful fallback to a database-computed historical average activates automatically when the ML model is unavailable or encounters a prediction error.

The notification engine uses Django's ORM to create and persist `Notification` objects linked to each user, with a global context processor (`notification_stats`) injecting unread counts into every template for real-time badge rendering.

---

## Features

### Before Login (Public Access)

- **Landing Page** — Displays live platform statistics (total product listings, active market data nodes) with navigation to Farmer and Buyer portals.
- **Marketplace Browse** — Public-facing marketplace allowing unauthenticated visitors to discover available millet products with search and filter support.
- **User Registration** — Role-selection signup (Farmer or Buyer) with email-based OTP verification. User accounts are created as inactive and activated only upon OTP confirmation.
- **Email OTP Verification** — 6-digit OTP sent via SMTP Gmail; 10-minute expiry enforced server-side through `EmailOTP.is_expired()`. Dedicated verification success and failure pages.
- **Login** — Email-and-password login with role-based redirect: Admins → Analytics Dashboard, Farmers → My Listings, Buyers → Marketplace.
- **Password Reset** — Full Django password reset flow with email dispatch, token validation, confirmation, and success screens.

### After Login — Farmer Portal

- **My Listings** — Personalised view of all products listed by the authenticated farmer, showing millet type, quantity, unit, price, location, harvest date, and availability status.
- **Add Product** — Listing form with fields for millet type (6 varieties), quantity, unit (kg/quintal), expected price, location, harvest date, product image upload, and description.
- **Edit Product** — Pre-populated update form for modifying any existing listing.
- **Delete Product** — Confirmation-gated deletion workflow removing the listing from the marketplace.
- **Incoming Orders** — Order management panel listing all purchase requests from buyers. Farmer can accept, reject, or mark orders as completed. On acceptance, product quantity is decremented and availability is toggled when stock reaches zero.
- **AI Price Predictor** — Input form accepting millet type, location, quantity, and season. Returns AI-predicted price per kg from the scikit-learn pipeline, labelled as "AI Engine". Falls back to database historical average labelled as "Historical Average".
- **Demand Forecast** — Aggregated monthly demand chart showing total quantity sold per month across all sales history records, with a 3-month moving average forecast value.
- **Notifications** — Real-time in-app notification inbox displaying order updates (new order received, order status changes), price alerts, and system announcements with read/unread tracking.
- **Profile** — User profile page displaying account details.

### After Login — Buyer Portal

- **Marketplace** — Searchable product catalogue of all available millet listings. Supports keyword search (description and location), millet type filter, and location filter. Displays farmer, price, quantity, and location for each listing.
- **Product Detail** — Detailed product page with full farmer information, harvest date, and order quantity form with available stock validation.
- **Place Order** — Purchase request submission with quantity validation (positive, ≤ available stock), automatic total price computation (`quantity × unit_price`), and immediate notification dispatch to the listing farmer.
- **Buyer Order History** — Chronological history of all purchase requests showing product details, quantity, total price, current status, and timestamps.
- **Notifications** — In-app notification inbox for order status updates (accepted, rejected, completed) dispatched by the farmer.

### After Login — Admin Portal

- **Analytics Dashboard** — Platform-wide command centre displaying total sales transactions, average market price across all millets, top-selling millet type, recent user registrations, and recent crop listings activity feed.
- **ML Model Metrics** — Table of registered `MLModelMetric` records showing model name, accuracy score, and last training timestamp.
- **Upload Dataset** — CSV upload interface allowing administrators to bulk-import sales history records into the `SalesHistory` table for ML training data enrichment. Expected CSV columns: `millet_type`, `month`, `location`, `quantity_sold`, `price`.
- **User Management** — Full list of all registered platform users with role, verification status, and account details.

### Platform-Wide Features

- **Multilingual Interface** — Full UI translation support for English (`en`), Hindi (`hi`), and Telugu (`te`) using Django's `LocaleMiddleware` with compiled `.po`/`.mo` files. Language preference stored in a persistent cookie (`LANGUAGE_COOKIE_AGE = 31536000`).
- **Notification Context Processor** — `notification_stats` context processor injects `unread_count` into every template globally, enabling live notification badge rendering in the navigation bar.
- **Role-Based Access Control** — All views enforce `@login_required` and role checks, redirecting unauthorised users appropriately. Admin-only views validate `request.user.role == 'admin'`.
- **Responsive Templates** — Two base templates (`base1.html`, `base2.html`) providing layout variants for different portal contexts.

---

## Technologies Used

### Programming Languages
- Python 3.10+
- HTML5 / CSS3 / JavaScript

### Libraries & Frameworks
- **Django 5.2** — Full-stack web framework
- **django-crispy-forms 2.5 + crispy-bootstrap5** — Form rendering with Bootstrap 5 styling
- **Django i18n** — Internationalisation framework with `LocaleMiddleware`, `gettext_lazy`, and `.po`/`.mo` locale files

### Machine Learning / AI
- **scikit-learn 1.7** — ML pipeline for millet price regression (model serialised as `.pkl`)
- **pandas 2.2** — DataFrame construction for ML inference input
- **numpy 1.26** — Numerical operations during model training and feature engineering
- **joblib 1.4** — Model serialisation and lazy loading at application startup
- **Google Generative AI / Gemini SDK** — Available as a dependency for potential LLM-based advisory features (`google-genai`, `google-generativeai`)

### Backend
- **Django ORM** — Database abstraction across all five apps
- **Django Auth** — Extended via `AbstractUser` with email-as-username and role field
- **Django Email (SMTP)** — Gmail SMTP backend for OTP delivery and password reset emails
- **Pillow 12** — Image processing for product photo uploads
- **reportlab 4** — PDF export capability
- **qrcode 8** — QR code generation support
- **python-dotenv** — Environment variable management

### Frontend
- **Django Templates** — Server-rendered HTML with template inheritance (`base1.html`, `base2.html`)
- **Bootstrap 5** — Responsive grid and UI components via crispy-bootstrap5
- **Chart.js** — Role-specific dashboard charts (bar charts for farmer stock, line charts for price trends and buyer procurement history, global average price line for admin)

### Database
- **SQLite3** — Default development database (Django ORM managed)
- Django migration system for schema versioning

### Deployment & DevOps
- **Django ASGI** (`asgi.py`) — ASGI application entry point for async-capable deployment
- **Django WSGI** (`wsgi.py`) — Traditional WSGI server compatibility
- **python-dotenv** — `SECRET_KEY`, `EMAIL_HOST_USER`, `EMAIL_HOST_PASSWORD` loaded from `.env`

### Security
- **Django AbstractUser** — PBKDF2-SHA256 hashed passwords
- **Email OTP** — 6-digit random OTP with 10-minute server-side expiry; user account remains inactive until verified
- **CSRF Protection** — Django's built-in CSRF middleware active on all forms
- **Login-required decorators** — All transactional views protected with `@login_required`
- **Role enforcement** — Per-view role checks prevent cross-role access

---

## Dataset

### Millet Price Dataset (`millet_price_dataset_.csv`)
- **Size:** 160,000 rows × 6 columns.
- **Schema:** `millet_type`, `quantity_kg`, `location`, `month`, `season`, `price_per_kg`.
- **Millet Types:** Ragi, Jowar, Bajra, Foxtail, Proso, Little Millet.
- **Locations:** Multiple Indian states including Andhra Pradesh, Uttar Pradesh, and others.
- **Seasons:** Summer, Monsoon, Kharif (reflecting Indian agricultural calendar).
- **Price Range:** Realistic per-kg prices based on Indian millet market conditions.
- **Usage:** Used to train the scikit-learn ML pipeline (`millet_price_model.pkl`). Administrators can also upload additional sales history CSVs (5-column format) via the platform's dataset upload interface to enrich the `SalesHistory` database table.

### SalesHistory Database Table
- **Purpose:** Stores platform-accumulated transaction records; used for demand forecasting aggregations and historical average price fallback.
- **Fields:** `millet_type`, `month` (1–12), `location`, `quantity_sold`, `price`, `sale_date`.
- **Population:** CSV bulk upload via the admin portal's dataset upload view.

---

## System Architecture

```
┌────────────────────────────────────────────────────────────────┐
│                    Django Template Frontend                     │
│  Landing, Marketplace, Product Detail, Farmer Listings,        │
│  Order Management, Analytics, Notifications, Dashboard         │
│  Multilingual: English | Hindi | Telugu                        │
└────────────────────────┬───────────────────────────────────────┘
                         │ HTTP (Django Views, Sessions, CSRF)
┌────────────────────────▼───────────────────────────────────────┐
│                Django Backend — 5 Apps                         │
│                                                                │
│  accounts/   — Auth, OTP, role-based routing                   │
│  products/   — Millet listings CRUD, marketplace search        │
│  orders/     — Purchase requests, 4-stage status workflow      │
│  analytics/  — ML price prediction, demand forecast, admin     │
│  notifications/ — In-app alerts, context processor badges      │
└──────┬──────────────────┬────────────────────┬─────────────────┘
       │                  │                    │
┌──────▼──────┐  ┌────────▼────────┐  ┌───────▼───────────────┐
│  SQLite DB  │  │  ML Subsystem   │  │  External Services    │
│  (Django    │  │  analytics/     │  │  Gmail SMTP           │
│   ORM)      │  │  ml_models/     │  │  (OTP + Password      │
│             │  │  millet_price_  │  │   Reset Email)        │
│  6 models:  │  │  model.pkl      │  └───────────────────────┘
│  User       │  │  (scikit-learn  │
│  EmailOTP   │  │   pipeline,     │
│  MilletProd │  │   ~31 MB)       │
│  Order      │  │  joblib loaded  │
│  SalesHist  │  │  at startup     │
│  Notifictn  │  └─────────────────┘
│  MLModelMet │
└─────────────┘
```

---

## Model Workflow

### User Registration & Authentication

1. **Signup** — User submits email, password, and role selection. Account created with `is_active=False`.
2. **OTP Generation** — 6-digit OTP generated via `random.randint(100000, 999999)`, stored in `EmailOTP`, and dispatched via Gmail SMTP.
3. **OTP Verification** — User submits OTP; server checks expiry (`timezone.now() > created_at + timedelta(minutes=10)`) and string equality. On success, `is_active` and `is_verified` are set to `True`.
4. **Login** — Email/password authentication via Django `authenticate()`. Successful login increments `login_count` and redirects based on role.

### Millet Price Prediction Pipeline

5. **Input Collection** — Farmer submits millet type, location, quantity (kg), and season via the price predictor form.
6. **DataFrame Construction** — Inputs are normalised (`.capitalize()`) and assembled into a single-row pandas DataFrame matching the training feature schema.
7. **ML Inference** — `trained_pipeline.predict(input_df)` returns the predicted price per kg. The prediction is rounded to 2 decimal places and labelled as "AI Engine".
8. **Fallback** — If the ML model is unavailable or raises an exception, the view queries `SalesHistory.objects.filter(millet_type=..., location__icontains=...).aggregate(Avg('price'))`. Defaults to ₹45.00 if no matching records exist. Labelled as "Historical Average".

### Order Lifecycle

9. **Place Order** — Buyer submits quantity; system validates `0 < quantity ≤ product.quantity`, computes `total_price = quantity × product.price`, creates `Order` with `status='pending'`, and creates a notification for the farmer.
10. **Farmer Review** — Farmer views incoming orders and updates status to `accepted`, `rejected`, or `completed`.
11. **Acceptance Logic** — On `accepted`, product quantity is decremented (`product.quantity -= order.quantity`); if quantity reaches zero, `product.is_available` is set to `False`.
12. **Notification Dispatch** — Every status update triggers `Notification.objects.create()` for the buyer with the new status, message, and redirect link.

### Demand Forecasting

13. **Aggregation** — `SalesHistory.objects.values('month').annotate(total_qty=Sum('quantity_sold')).order_by('month')` produces monthly demand totals.
14. **3-Month Moving Average Forecast** — `sum(quantities[-3:]) / 3` applied to the last 3 months of data to project near-term demand.

---

## Implementation Details

**Custom User Model:** `accounts.User` extends `AbstractUser`, replacing `username` with `email` as the `USERNAME_FIELD`. A `CustomUserManager` handles both `create_user` and `create_superuser`. The `role` field (`admin`, `farmer`, `buyer`) drives all access control and dashboard routing logic throughout the application.

**Notification Context Processor:** `notifications.context_processors.notification_stats` is registered in `settings.TEMPLATES` and injects `unread_count` (the count of `is_read=False` notifications for the authenticated user) into every template's context. This enables the navigation bar to display a live notification badge without any per-view boilerplate.

**ML Model Startup Loading:** `analytics/views.py` loads `millet_price_model.pkl` at module import time via `joblib.load()`, wrapped in a `try/except` that falls back to `trained_pipeline = None` and prints a warning. This ensures a single disk read per process and prevents request-time loading latency.

**Product Availability Automation:** When a farmer accepts an order, the `update_order_status` view decrements `product.quantity` by the ordered amount and automatically sets `product.is_available = False` if quantity reaches zero. This prevents buyers from placing further orders on out-of-stock products without manual intervention.

**Multilingual Translation:** All user-facing strings are wrapped with `gettext_lazy(_('...'))` across models, views, and templates. Compiled `.mo` files for Hindi and Telugu are stored under `locale/hi/LC_MESSAGES/` and `locale/te/LC_MESSAGES/`. `Django.middleware.locale.LocaleMiddleware` detects the user's language preference from the `django_language` cookie (`LANGUAGE_COOKIE_AGE = 31536000`) and activates the appropriate translation.

**Role-Based Dashboard Data:** The `dashboard_view` in `core/views.py` serves a single URL (`/dashboard/`) with role-specific query sets and Chart.js configuration injected into the context. Farmers receive a bar chart of their stock quantities; Buyers receive a line chart of price trends from sales history; Admins receive a global average price line chart aggregated across all months.

---

## Results

| Component | Metric | Value |
|---|---|---|
| ML Training Dataset | Rows | 160,000 transactions |
| ML Training Dataset | Features | 4 (`millet_type`, `quantity_kg`, `location`, `season`) |
| ML Model File | Size | ~31 MB (`millet_price_model.pkl`) |
| ML Model Type | Framework | scikit-learn Pipeline (joblib serialised) |
| ML Fallback | Method | Historical average via `SalesHistory` DB query |
| Supported Millet Varieties | Count | 6 (Ragi, Jowar, Bajra, Foxtail, Proso, Little) |
| Language Support | Languages | English, Hindi, Telugu |
| Order Workflow Stages | Count | 4 (Pending, Accepted, Rejected, Completed) |
| OTP Expiry Window | Duration | 10 minutes |
| Notification Types | Categories | 3 (Order Update, Price Alert, System) |
| Django Apps | Count | 5 (accounts, products, orders, analytics, notifications) |
| Database | Engine | SQLite3 (development) |
| Demand Forecast Method | Algorithm | 3-month simple moving average |
| Admin Dataset Upload | Format | CSV (5 columns: type, month, location, qty, price) |
| Image Upload | Storage | `media/products/YYYY/MM/DD/` |

---

## Installation

### Prerequisites

- Python 3.10 or higher
- `pip` package manager
- A Gmail account with App Password enabled (for SMTP email delivery)

---

**1. Clone the repository:**
```bash
git clone https://github.com/harisaigithub/milletchain.git
cd milletchain
```

**2. Create and activate a virtual environment:**
```bash
python -m venv venv
source venv/bin/activate        # Linux / macOS
venv\Scripts\activate           # Windows
```

**3. Install dependencies:**
```bash
pip install -r req.txt
```

**4. Configure environment variables:**

Create a `.env` file in the project root:
```env
SECRET_KEY=your-django-secret-key-here
EMAIL_HOST_USER=your-gmail-address@gmail.com
EMAIL_HOST_PASSWORD=your-gmail-app-password
```

> **Note:** Use a Gmail App Password (not your account password). Generate one at `myaccount.google.com → Security → App passwords`.

**5. Apply database migrations:**
```bash
python manage.py migrate
```

**6. Create a superuser (Admin account):**
```bash
python manage.py createsuperuser
```
When prompted, enter an email and password. The superuser is automatically assigned the `admin` role.

**7. Compile translation files:**
```bash
python manage.py compilemessages
```

**8. Start the development server:**
```bash
python manage.py runserver
```

The platform will be available at `http://127.0.0.1:8000`.

**9. (Optional) Load the millet price dataset into SalesHistory:**

Log in as Admin → navigate to **Analytics → Upload Dataset** → upload `millet_price_dataset_.csv` (or a 5-column subset) to populate the `SalesHistory` table for demand forecasting and historical average fallback.

---

## Usage

**As a Farmer:**

1. Register at `/accounts/signup/` with role set to **Farmer**.
2. Verify your email using the OTP sent to your inbox.
3. Log in — you will be redirected to **My Listings**.
4. Add a new millet listing via **Add Product**, entering millet type, quantity, price, location, harvest date, and an optional photo.
5. Use the **AI Price Predictor** (`/analytics/price-predictor/`) to get an AI-suggested price before listing.
6. Monitor incoming purchase requests from buyers at `/orders/requests/` and accept, reject, or complete orders.
7. Check the **Notification Inbox** for real-time order alerts.

**As a Buyer:**

1. Register at `/accounts/signup/` with role set to **Buyer**.
2. Verify your email and log in — you will be redirected to the **Marketplace**.
3. Browse and search millet listings by keyword, millet type, or location.
4. Click a product to view its details and submit a purchase request with your desired quantity.
5. Track all your orders at `/orders/my-procurement/`.
6. Check the **Notification Inbox** for order status updates from farmers.

**As an Admin:**

1. Log in with your superuser credentials — you will be redirected to the **Analytics Dashboard**.
2. Monitor platform-wide sales, top-selling millets, recent user and product activity.
3. Upload additional sales history data via **Upload Dataset**.
4. Manage all registered users at `/accounts/user-management/`.
5. Review ML model performance metrics registered in the system.

---

## Future Enhancements

- **Gemini AI Advisory** — Integrate the bundled `google-generativeai` SDK to provide farmers with crop advisory, pest alerts, and market timing recommendations through a conversational assistant powered by Gemini.
- **QR Code Product Listings** — Leverage the bundled `qrcode` library to generate scannable QR codes for each product listing, enabling offline sharing and traceability.
- **PDF Invoice Generation** — Use the bundled `reportlab` library to generate downloadable PDF invoices for completed orders, supporting formal transaction records for both farmers and buyers.
- **PostgreSQL Migration** — Replace SQLite3 with PostgreSQL for production deployment, enabling concurrent writes, connection pooling, and full-text search on product descriptions.
- **Real-Time Notifications via WebSockets** — Integrate Django Channels to push order status updates and price alerts to the browser in real time without page refresh.
- **ML Model Retraining Pipeline** — Build an automated retraining workflow that periodically re-trains the price prediction model on accumulated `SalesHistory` data and updates the `MLModelMetric` record.
- **Payment Gateway Integration** — Add Razorpay or UPI-based payment processing to enable end-to-end digital transactions within the platform.
- **Mobile-Responsive PWA** — Upgrade the frontend to a Progressive Web App with service worker caching for offline listing browsing in low-connectivity rural environments.
- **Government MSP Integration** — Integrate NAFED or AgMarkNet APIs to display Minimum Support Prices alongside AI predictions, ensuring farmers can make fully informed pricing decisions.
- **Expanded Language Support** — Add Kannada, Tamil, Marathi, and Odia translations using Django's existing i18n infrastructure to reach a broader farmer base across India's millet-producing states.

---

## Contributing

Contributions are welcome. Please follow the standard Git workflow below.

**1. Fork the repository:**
```bash
git clone https://github.com/harisaigithub/milletchain.git
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

**4. Push to your fork and open a Pull Request:**
```bash
git push origin feature/your-feature-name
```

Please follow PEP 8 for Python code and ensure all existing migrations, views, and ML model loading remain unaffected by your changes.

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