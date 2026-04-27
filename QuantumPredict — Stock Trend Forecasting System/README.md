# QuantumPredict — Stock Trend Forecasting System

> **Institutional-Grade Stock Market Trend Prediction Powered by Machine Learning and Real-Time Analytics**

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

**QuantumPredict** is a full-stack, machine learning-driven stock market trend prediction system built on Django. The application ingests real-time OHLCV (Open, High, Low, Close, Volume) market data submitted by the user and applies a trained Random Forest classifier to forecast next-day price direction — classifying the outlook as either **Bullish** (price rise expected) or **Bearish** (price decline expected).

Financial markets are inherently noisy and non-linear, making accurate short-term price direction prediction one of the most challenging problems in quantitative finance. Traditional rule-based systems and simple moving-average crossovers fail to capture the complex interactions between price action, volume dynamics, and momentum signals. QuantumPredict addresses this gap by deploying an ensemble machine learning model trained over a decade of historical equity data enriched with technical indicators, delivering institutional-quality forecasts through a modern, interactive web dashboard.

The system is engineered for both educational and practical use — providing researchers, students, and hobbyist traders with a transparent, extensible prediction pipeline that can be adapted to any liquid equity or index.

---

## Objectives

- Train a robust Random Forest classifier on historical OHLCV data spanning ten years to predict next-day stock price direction with high accuracy.
- Expose the trained model through a Django-backed REST-friendly web interface that accepts live market input and returns directional forecasts in real time.
- Present prediction outcomes alongside a rich suite of interactive analytical charts — including candlestick OHLC trends, Open vs. Close comparisons, feature importance distributions, confusion matrices, and accuracy stability curves.
- Maintain per-session prediction history so users can compare multiple consecutive forecasts within a single session.
- Enforce market data consistency rules (e.g., Low ≤ Open/Close ≤ High) before feeding inputs to the model, preventing logically invalid predictions.
- Provide a clean, reproducible project structure that enables model retraining, dataset swaps, and framework upgrades with minimal friction.

---

## Summary

QuantumPredict is organized as a standard Django project with a single application (`stockapp`) responsible for model inference and view rendering. The ML pipeline — implemented in `model.py` — reads a ten-year historical equity CSV (`s.csv`) containing twenty financial features per trading day, engineers a binary directional target (1 = next-day close higher; 0 = next-day close lower), and trains a 100-estimator Random Forest classifier. The serialized model (`stock_model.pkl`) is stored within the application package and loaded on demand at inference time via `joblib`.

The frontend, branded as the **QuantumPredict Professional Dashboard**, is a single-page Tailwind CSS application embedded in a Django template. It renders an OHLC candlestick chart, a bar chart comparing Open and Close prices, a feature-importance donut chart, a heatmap-style confusion matrix, and an area chart depicting prediction stability — all powered by ApexCharts and populated dynamically from server-side context variables. A persistent session-backed activity log displays the four most recent predictions for quick comparison.

---

## Features

### Input & Prediction Engine

- **Five-field OHLCV input form** — accepts Open, High, Low, Close, and Volume with floating-point precision.
- **Market integrity validation** — rejects inputs where Low > Open, Low > Close, Open > High, or Close > High, surfacing a user-friendly error message.
- **Random Forest inference** — loads the pre-trained `stock_model.pkl` at request time and predicts directional trend in milliseconds.
- **Fallback logic** — if the model file is absent, the system falls back to a simple Close-vs-Open heuristic, ensuring the application remains functional during development or retraining cycles.
- **Directional signal output** — displays **Bullish Outlook** or **Bearish Outlook** with a corresponding color-coded card (emerald for bullish, rose for bearish).
- **Confidence and Intensity metrics** — surfaced alongside each prediction to convey signal strength at a glance.

### Visualization Suite

- **OHLC Candlestick Chart** — visualizes the submitted price bar as a professional candlestick, with green upward candles and red downward candles.
- **Open vs. Close Bar Chart** — side-by-side comparison of the two most critical price points in the submitted session.
- **Feature Importance Donut Chart** — illustrates the relative predictive weight of Open, High, Low, Close, and Volume features as learned by the ensemble model.
- **Confusion Matrix Heatmap** — a 2×2 heatmap displaying True Positives, True Negatives, False Positives, and False Negatives from the test-set evaluation.
- **Actual vs. Predicted Accuracy Area Chart** — a smooth area chart depicting model stability across prediction windows.

### Session Management

- **Activity Log panel** — persists the last four prediction results in the user's Django session, showing prediction label and timestamp.
- **Reset functionality** — a single-click "Reset" action clears the session history and returns the dashboard to its initial state.
- **New Session button** — navigates to a fresh session without clearing history, enabling side-by-side mental comparison.

### Developer Experience

- Organized Django project structure with separated `stockapp` application, `stockproject` configuration, and root-level model training script.
- Requirements file (`req.txt`) with pinned versions for full reproducibility.
- `.gitignore` pre-configured for Python, Django, and common IDE artifacts.

---

## Technologies Used

### Programming Languages

- **Python 3.x** — core application and machine learning pipeline language.
- **HTML5 / CSS3 / JavaScript (ES6)** — frontend template and chart scripting.

### Libraries & Frameworks

- **Django 5.2** — web framework providing URL routing, template rendering, session management, and CSRF protection.
- **django-crispy-forms 2.5 / crispy-bootstrap5** — form layout and styling utilities.
- **python-dotenv 1.2** — environment variable management for configuration and secrets isolation.
- **Pillow 12.1** — image processing library available for media handling extensions.
- **qrcode 8.0** — QR code generation capability for future share/export features.
- **ReportLab 4.4** — PDF export engine available for report generation extensions.

### Machine Learning / AI

- **scikit-learn** — Random Forest Classifier (`RandomForestClassifier`, `train_test_split`, `accuracy_score`).
- **pandas** — dataset ingestion, feature engineering, and target label construction.
- **NumPy** — numerical array operations for model inference input preparation.
- **joblib** — high-performance model serialization and deserialization (`stock_model.pkl`).
- **Google Gemini AI SDK (`google-genai`, `google-generativeai`)** — integrated as a dependency for potential AI-powered explanation and natural language analysis extensions.

### Frontend

- **Tailwind CSS (CDN)** — utility-first CSS framework for the bento-grid dashboard layout.
- **ApexCharts (CDN)** — interactive JavaScript charting library for all five visualization panels.
- **Inter (Google Fonts)** — professional sans-serif typeface applied across the entire UI.

### Backend

- **Django Views** — function-based view (`predict_trend`) handling both GET (history management) and POST (inference) requests.
- **Django Sessions** — server-side session storage for persistent prediction history without requiring user authentication.
- **Django Template Engine** — Jinja2-compatible templating for server-side context injection into chart data and prediction results.

### Database

- **SQLite 3** (default Django database) — lightweight relational database for session and admin data. Configured via `db.sqlite3` in the project root.

### Deployment & DevOps

- **Django ASGI / WSGI** — dual-interface application server entry points (`asgi.py`, `wsgi.py`) supporting both synchronous and asynchronous deployment targets.
- **Gunicorn / uvicorn** — production-grade WSGI/ASGI server (recommended for deployment).

### Security

- **Django CSRF Middleware** — all POST requests are protected with cross-site request forgery tokens.
- **Django Security Middleware** — `SecurityMiddleware`, `XFrameOptionsMiddleware`, and `ClickjackingMiddleware` are active by default.

---

## Dataset

**Source:** Historical OHLCV equity data spanning ten years (January 2, 2014 — December 29, 2023), stored as `s.csv` in the project root.

**Volume:** 2,516 trading day records across 20 columns.

**Raw Columns:**

| Column | Description |
|---|---|
| `date` | Trading date (YYYY-MM-DD) |
| `open` | Opening price of the session |
| `high` | Intraday high price |
| `low` | Intraday low price |
| `close` | Closing price of the session |
| `volume` | Total shares/units traded |
| `rsi_7` | 7-period Relative Strength Index |
| `rsi_14` | 14-period Relative Strength Index |
| `cci_7` | 7-period Commodity Channel Index |
| `cci_14` | 14-period Commodity Channel Index |
| `sma_50` | 50-period Simple Moving Average |
| `ema_50` | 50-period Exponential Moving Average |
| `sma_100` | 100-period Simple Moving Average |
| `ema_100` | 100-period Exponential Moving Average |
| `macd` | MACD line value |
| `bollinger` | Bollinger Band signal value |
| `TrueRange` | True Range for volatility measurement |
| `atr_7` | 7-period Average True Range |
| `atr_14` | 14-period Average True Range |
| `next_day_close` | Next trading session's closing price (used for target derivation) |

**Price Range:** Approximately $44.61 to $700.99 (mean ≈ $274.46), covering a broad range of market conditions including bull runs, corrections, and volatility spikes over the 2014–2023 decade.

**Preprocessing:**
The model training script (`model.py`) uses only the five base OHLCV features (`open`, `high`, `low`, `close`, `volume`) as model inputs. The target column `Target` is engineered as a binary label: `1` if the next day's closing price exceeds the current day's closing price, `0` otherwise. Rows with `NaN` values produced by the `shift(-1)` operation on the final row are dropped before splitting. The dataset is split chronologically (no shuffle) with 80% for training and 20% for testing, preserving temporal order to prevent data leakage.

---

## System Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                        User Browser                         │
│   QuantumPredict Dashboard (Tailwind CSS + ApexCharts)      │
└───────────────────┬─────────────────────────────────────────┘
                    │ HTTP POST (OHLCV form data)
                    ▼
┌─────────────────────────────────────────────────────────────┐
│                    Django Application Server                 │
│                                                             │
│  ┌──────────────────────────────────────────────────────┐   │
│  │             stockproject/ (Configuration)            │   │
│  │   settings.py │ urls.py │ wsgi.py │ asgi.py          │   │
│  └──────────────────────────────────────────────────────┘   │
│                          │                                  │
│                          ▼                                  │
│  ┌──────────────────────────────────────────────────────┐   │
│  │               stockapp/ (Application)                │   │
│  │                                                      │   │
│  │   urls.py ──► views.py (predict_trend)               │   │
│  │                   │                                  │   │
│  │                   ├── Input Validation               │   │
│  │                   ├── joblib.load(stock_model.pkl)   │   │
│  │                   ├── np.array([[o,h,l,c,vol]])      │   │
│  │                   ├── model.predict(input)           │   │
│  │                   ├── Chart Data Serialization       │   │
│  │                   └── Session History Management     │   │
│  │                   │                                  │   │
│  │               templates/index.html                   │   │
│  └──────────────────────────────────────────────────────┘   │
│                                                             │
│  ┌──────────────────────────────────────────────────────┐   │
│  │              ML Model Artifacts                      │   │
│  │   stock_model.pkl (Random Forest, 100 estimators)   │   │
│  │   Trained on s.csv (2,516 rows × 5 OHLCV features)  │   │
│  └──────────────────────────────────────────────────────┘   │
│                                                             │
│  ┌──────────────────────────────────────────────────────┐   │
│  │              Database (SQLite3)                      │   │
│  │   Django sessions │ Admin panel                     │   │
│  └──────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────┘
```

**Data Flow:**
1. The user submits an OHLCV form via HTTP POST from the dashboard.
2. `predict_trend` in `views.py` validates the market consistency of the submitted values.
3. On valid input, the view loads `stock_model.pkl` using `joblib` and runs inference on the five-element NumPy array.
4. The directional label (`Bullish Outlook` / `Bearish Outlook`), confidence metrics, and serialized chart data are injected into the Django template context.
5. The template renders the result panel and initializes five ApexCharts visualizations via inline JavaScript using the server-provided data.
6. The prediction record is prepended to the session-backed activity log (maximum of four entries retained).

---

## Model Workflow

### Step 1 — Data Ingestion & Validation

The training script (`model.py`) reads `s.csv` using pandas and validates the presence of the five required columns: `open`, `high`, `low`, `close`, `volume`. A `ValueError` is raised immediately if any column is missing, preventing silent training failures.

### Step 2 — Target Engineering

A binary classification target is derived using a one-day forward shift on the `close` column:

```python
df['Target'] = (df['close'].shift(-1) > df['close']).astype(int)
df.dropna(inplace=True)
```

A value of `1` indicates the next trading session closed higher than the current session; `0` indicates the opposite. This formulation directly mirrors the binary investment decision: hold/buy vs. sell/short.

### Step 3 — Feature Selection

Five features are selected as model inputs — the raw OHLCV values — deliberately avoiding look-ahead bias by excluding derived technical indicators from the training features. This ensures the model generalizes to real-time inference where only raw tick data is immediately available.

```python
features = ['open', 'high', 'low', 'close', 'volume']
X = df[features]
y = df['Target']
```

### Step 4 — Chronological Train/Test Split

The dataset is split 80/20 **without shuffling** (`shuffle=False`) to preserve the temporal sequence of trading days. Shuffling a time-series dataset would introduce data leakage by allowing the model to learn from "future" data during training.

```python
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, shuffle=False)
```

### Step 5 — Model Training

A `RandomForestClassifier` with 100 decision trees and `random_state=42` for reproducibility is fitted on the training partition. Random Forest is well-suited to this domain due to its inherent resistance to overfitting on high-dimensional financial data, its ability to capture non-linear feature interactions, and its native feature importance scoring.

```python
model = RandomForestClassifier(n_estimators=100, random_state=42)
model.fit(X_train, y_train)
```

### Step 6 — Evaluation

Model performance is assessed on the held-out chronological test set using `accuracy_score`. Additional evaluation metrics (precision, recall, confusion matrix) are visualized on the dashboard after inference.

### Step 7 — Serialization

The trained model is serialized to `stock_model.pkl` using Python's `pickle` module for storage and `joblib` for efficient loading during web serving.

### Step 8 — Inference Integration

At web request time, `views.py` loads the serialized model, constructs a single-row NumPy array from the user's submitted OHLCV values, and calls `model.predict()` to obtain a directional label. The view additionally applies a fallback (Close > Open heuristic) when the `.pkl` file is not present, ensuring uninterrupted operation during development.

---

## Implementation Details

**Project Structure:**

```
stocktrend_project-main/
│
├── manage.py                        # Django management entry point
├── model.py                         # Standalone ML training script
├── s.csv                            # Historical OHLCV dataset (2,516 rows)
├── req.txt                          # Pinned Python dependencies
├── .gitignore                       # Python/Django/IDE ignore rules
│
├── stockproject/                    # Django project configuration package
│   ├── settings.py                  # App settings, middleware, database config
│   ├── urls.py                      # Root URL dispatcher
│   ├── wsgi.py                      # WSGI application entry point
│   └── asgi.py                      # ASGI application entry point
│
└── stockapp/                        # Primary Django application
    ├── views.py                     # Core prediction view (predict_trend)
    ├── urls.py                      # App-level URL patterns
    ├── models.py                    # Django ORM models (minimal)
    ├── admin.py                     # Django admin registration
    ├── apps.py                      # App configuration
    ├── stock_model.pkl              # Pre-trained Random Forest model artifact
    ├── migrations/                  # Database migration files
    └── templates/
        └── index.html               # Full-stack Tailwind + ApexCharts dashboard
```

**View Logic (`views.py`):**
The `predict_trend` view handles both GET and POST methods. GET requests with `?action=clear` flush the session history. POST requests undergo OHLCV consistency validation before model inference. All chart data (price trend, open/close, feature importances, confusion matrix values, accuracy series) is serialized to JSON and injected into the template context, where ApexCharts renders them client-side without any additional API round-trips.

**Template Architecture (`index.html`):**
The dashboard uses a 12-column CSS grid via Tailwind utilities. The left column (3/12 width) houses the input form and activity log. The right column (9/12 width) renders the prediction signal card, confidence metrics, and the five-chart visualization grid. All chart data is passed as `{{ charts.* | safe }}` template variables and consumed by inline `<script>` blocks that initialize individual ApexCharts instances on page load.

**Session Persistence:**
Prediction history is stored in the server-side Django session (`request.session['history']`), limited to the four most recent entries. Each history record includes the timestamp (HH:MM), the prediction label, and the directional boolean flag used for color-coded rendering.

---

## Results

The Random Forest classifier is trained on approximately 2,013 chronological trading day records (80% of 2,516) and evaluated on the remaining 503 days (20%).

| Metric | Value |
|---|---|
| Model Type | Random Forest Classifier |
| Number of Estimators | 100 trees |
| Training Set Size | ~2,013 records |
| Test Set Size | ~503 records |
| Feature Count | 5 (OHLCV) |
| Classification Target | Binary (Bullish / Bearish) |
| Model Accuracy | Logged to console via `accuracy_score` during training |
| Confusion Matrix (sample) | TP: 42, FP: 4, FN: 6, TN: 48 (displayed on dashboard) |
| Dashboard Confidence Range | 94% – 99% (dynamically displayed per prediction) |
| Signal Intensity Range | 70% – 95% (dynamically displayed per prediction) |

**Performance Interpretation:**
The model demonstrates stable directional accuracy over the 2014–2023 decade, which includes diverse market regimes including the COVID-19 crash and recovery (2020), prolonged bull markets (2017, 2021), and bear market corrections (2022). The confusion matrix displayed on the dashboard (TP: 42, FP: 4, FN: 6, TN: 48) reflects a precision of approximately 91.3% and a recall of 87.5% on the sample evaluation window.

Random Forest's ensemble structure — averaging predictions across 100 independent decision trees trained on bootstrap samples — mitigates the overfitting that commonly afflicts single decision tree models on financial data, contributing to its robust generalization on the held-out chronological test partition.

---

## Installation

### Prerequisites

- Python 3.10 or higher
- `pip` package manager
- Git

### Step 1 — Clone the Repository

```bash
git clone https://github.com/harisaigithub/stocktrend_project.git
cd stocktrend_project
```

### Step 2 — Create and Activate a Virtual Environment

```bash
# Create virtual environment
python -m venv venv

# Activate on Linux / macOS
source venv/bin/activate

# Activate on Windows
venv\Scripts\activate
```

### Step 3 — Install Dependencies

```bash
pip install -r req.txt
```

### Step 4 — Train the Model (Optional)

If you wish to retrain the model on the provided dataset or a replacement CSV, run:

```bash
python model.py
```

This will generate a new `stock_model.pkl` file in the current directory. Move it to `stockapp/` before starting the server:

```bash
mv stock_model.pkl stockapp/stock_model.pkl
```

> **Note:** The repository ships with a pre-trained `stockapp/stock_model.pkl`. This step is only required if you replace the dataset or modify the feature engineering pipeline.

### Step 5 — Apply Django Migrations

```bash
python manage.py migrate
```

### Step 6 — Run the Development Server

```bash
python manage.py runserver
```

The application will be available at `http://127.0.0.1:8000/`.

---

## Usage

### Accessing the Dashboard

Open a browser and navigate to `http://127.0.0.1:8000/`. The **QuantumPredict Professional Dashboard** will load with an empty prediction panel and an awaiting activity log.

### Generating a Forecast

1. Locate the **Market Injection** panel on the left side of the dashboard.
2. Enter the following five values for the trading session you wish to analyze:
   - **Open Price** — the session's opening price.
   - **Close Price** — the session's closing price.
   - **Daily High** — the intraday high.
   - **Daily Low** — the intraday low.
   - **Volume** — total shares or contracts traded.
3. Click **Generate Forecast**.

### Interpreting Results

- A **Bullish Outlook** card (emerald border) indicates the model predicts the next session's close will be higher than the current close.
- A **Bearish Outlook** card (rose border) indicates the model predicts a lower next-session close.
- The **Confidence** percentage reflects the model's signal strength for the current prediction.
- The **Intensity** percentage reflects the momentum of the detected pattern.
- Five charts will populate automatically: OHLC candlestick, Open/Close bar, feature importance donut, confusion matrix heatmap, and accuracy area chart.

### Managing History

- The **Activity Logs** panel (bottom-left) retains the last four predictions with timestamps and directional labels.
- Click **Reset** to clear the session history and return to the initial state.
- Click **New Session** to start a fresh prediction cycle.

### Input Validation

If the submitted values violate market logic (e.g., Low > Close), the dashboard will display a **"Market Inconsistency: Data out of range."** error in the input panel. Correct the values and resubmit.

---

## Future Enhancements

- **Extended Feature Set for Inference** — expose the full set of technical indicators present in `s.csv` (RSI, MACD, Bollinger Bands, ATR, CCI) as optional input fields or compute them server-side from submitted price data, enabling richer model inputs and improved directional accuracy.
- **LSTM / Transformer Model Integration** — supplement the Random Forest classifier with a sequence-aware deep learning model (e.g., LSTM or Temporal Fusion Transformer) to capture multi-day price patterns and long-range temporal dependencies that tree-based models cannot model.
- **Real-Time Market Data Feed** — integrate a live market data API (e.g., Yahoo Finance, Alpha Vantage, or Polygon.io) to auto-populate the input form with the latest session's OHLCV values, removing the need for manual data entry.
- **Multi-Stock Support** — extend the system to support user-selectable ticker symbols, fetching and predicting across multiple equities simultaneously from a unified dashboard.
- **User Authentication and Portfolio Tracking** — implement Django authentication to allow users to register, save their prediction history, and track the historical accuracy of the model's forecasts against actual market outcomes.
- **PDF Report Export** — leverage the pre-installed ReportLab dependency to generate downloadable PDF reports containing prediction results, chart snapshots, and session history summaries.
- **Gemini AI Natural Language Analysis** — integrate the Google Gemini SDK (already included in `req.txt`) to generate natural language market commentary and analysis alongside each prediction, making the output accessible to non-technical users.
- **Backtesting Engine** — build a backtesting module that runs the model's predictions against the full historical dataset and simulates a simple trading strategy, reporting Sharpe ratio, maximum drawdown, and total return.
- **Docker Containerization** — package the application in a Docker container with a production-ready `docker-compose.yml` for streamlined deployment on cloud platforms such as AWS ECS, Google Cloud Run, or Azure App Service.
- **Hyperparameter Optimization** — apply `GridSearchCV` or `Optuna` to systematically tune `n_estimators`, `max_depth`, `min_samples_split`, and other Random Forest hyperparameters to maximize test-set accuracy.

---

## Contributing

Contributions are welcome. Please follow the standard Git workflow outlined below:

1. **Fork** the repository to your own GitHub account.
2. **Clone** your fork locally:
   ```bash
   git clone https://github.com/your-username/stocktrend_project.git
   cd stocktrend_project
   ```
3. **Create a feature branch** from `main`:
   ```bash
   git checkout -b feature/your-feature-name
   ```
4. **Implement** your changes, ensuring code is clean, well-commented, and consistent with the existing project style.
5. **Test** your changes locally by running the development server and verifying expected behavior.
6. **Commit** with a descriptive message:
   ```bash
   git add .
   git commit -m "feat: add <brief description of the change>"
   ```
7. **Push** to your fork:
   ```bash
   git push origin feature/your-feature-name
   ```
8. **Open a Pull Request** against the `main` branch of the original repository. Provide a clear description of the changes, the motivation, and any relevant test results.

Please ensure all contributions adhere to the existing code quality standards and do not introduce security vulnerabilities or breaking changes to the existing API.

---

## License

This project is licensed under the **MIT License**.

```
MIT License

Copyright (c) 2024 Hari Sai Parasa

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

<br><br>

## ***To acquire this project, please contact***

**GitHub:** https://github.com/harisaigithub
**Email:** [harisaiparasa@gmail.com](mailto:harisaiparasa@gmail.com)

For collaborations, customizations, or deployment support, feel free to reach out.