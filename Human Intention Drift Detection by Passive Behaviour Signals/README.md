# Human Intention Drift Detection by Passive Behaviour Signals

> **A lightweight, real-time anomaly detection system that identifies shifts in user intent by passively monitoring keyboard and mouse behaviour — no cameras, no intrusion, no explicit input required.**

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

Human intention drift refers to the phenomenon where a person's cognitive state, focus, or behavioural pattern shifts meaningfully from their established baseline — often indicating fatigue, distraction, stress, unauthorised access, or a change in task context. Detecting such drift in real time, without relying on intrusive biometric hardware or explicit user interaction, is a significant challenge in the fields of human-computer interaction, insider threat detection, and adaptive user interfaces.

This project presents a passive, non-intrusive solution that captures raw keyboard press timing and mouse movement trajectories through background system listeners, extracts statistical behavioural features from each observation window, and applies an Isolation Forest anomaly detection model to classify whether the current behaviour represents a continuation of the user's normal pattern or a statistically significant drift.

The system requires no external sensors, no cloud connectivity, and no modification to existing applications. It operates entirely on the local machine and is designed to be modular, interpretable, and computationally lightweight.

---

## Objectives

- Design a passive behaviour data collection pipeline that captures keystroke timing and mouse movement metrics without interrupting user workflow.
- Engineer a compact, noise-robust feature set from raw behavioural signals — specifically average inter-keystroke interval and average per-step mouse displacement — that effectively characterises a user's interaction style.
- Implement a baseline profiling phase that establishes a statistically representative model of a user's normal behaviour by aggregating multiple observation windows.
- Train an unsupervised Isolation Forest anomaly detector on the normalised baseline profile to enable session-agnostic drift detection without labelled training data.
- Provide a real-time continuous monitoring loop that evaluates each new observation window against the baseline and surfaces a clear, interpretable alert when intention drift is detected.
- Handle edge cases robustly, including low-activity windows with insufficient data, by implementing activity threshold guards that prevent false positives from idle periods.

---

## Summary

The system operates in two sequential phases: a **training phase** and a **monitoring phase**. During training, the system collects five 60-second observation windows of the user's natural behaviour, extracts two behavioural features from each, and concatenates them into a baseline profile CSV. During monitoring, the system runs a continuous cycle — capturing a new 60-second window, extracting features, and querying the Isolation Forest model — every 10 seconds of inter-cycle delay.

The Isolation Forest model is re-fitted against the baseline on each monitoring cycle and evaluates the current feature vector to produce either a `NORMAL BEHAVIOR` classification with a stability score or an `INTENTION DRIFT DETECTED` alert with an anomaly score. The pipeline is implemented across four Python scripts (`main.py`, `data_collector.py`, `feature_engineering.py`, `model.py`) and depends exclusively on four lightweight libraries: `pynput`, `pandas`, `numpy`, and `scikit-learn`.

---

## Features

- **Zero-Intrusion Data Collection** — Uses `pynput` background listeners for keyboard and mouse events. The user is not required to interact with any special interface; normal computer use generates all the data needed.
- **Dual-Signal Behavioural Feature Extraction** — Derives two expressive, session-comparable features: average inter-keystroke interval (typing rhythm) and average per-step Euclidean mouse displacement (movement intensity).
- **Low-Activity Guard** — Automatically skips AI processing for observation windows with fewer than 10 keystroke events or fewer than 300 total input events, preventing noise-driven false positives during idle periods.
- **Unsupervised Anomaly Detection** — Applies a StandardScaler-normalised Isolation Forest (`n_estimators=200`, `contamination=0.05`) trained on the user's own baseline, requiring no labelled data or prior training corpus.
- **Interpretable Score Output** — Outputs a signed anomaly score alongside the binary classification, allowing the severity of drift to be quantified and thresholded by downstream systems.
- **Automatic Baseline Bootstrapping** — On first run, the system automatically enters the training phase and collects five baseline samples. Subsequent runs skip training and proceed directly to monitoring.
- **Continuous Monitoring Loop** — Runs indefinitely with configurable inter-cycle delay (default 10 seconds), supporting long-session surveillance scenarios.
- **Fully Local Execution** — All processing, storage, and inference occurs on-device. No data is transmitted externally.
- **Cross-Platform Compatibility** — Runs on Windows, macOS, and Linux; uses OS-agnostic terminal clearing and Python executable path resolution.

---

## Technologies Used

### Programming Languages
- Python 3.8+

### Libraries & Frameworks
- **pynput** — Cross-platform keyboard and mouse event listening
- **pandas** — CSV I/O, DataFrame construction, and data manipulation
- **numpy** — Numerical feature computation (inter-event differences, Euclidean distances)
- **scikit-learn** — `IsolationForest` anomaly detection model and `StandardScaler` feature normalisation

### Machine Learning / AI
- **Isolation Forest (scikit-learn)** — Unsupervised ensemble anomaly detection algorithm that isolates observations by randomly partitioning the feature space; anomalous points require fewer partitions to isolate
- **StandardScaler (scikit-learn)** — Zero-mean, unit-variance normalisation applied to baseline and current feature vectors before model training and inference

### Backend / Runtime
- Python standard library (`os`, `time`, `csv`, `sys`) — Pipeline orchestration, file I/O, and timer management

### Storage
- **CSV files (local filesystem)** — `behavior.csv` for raw event logs, `features.csv` for the current observation window's extracted features, `baseline.csv` for the accumulated training profile

---

## Dataset

### Raw Behavioural Data (`behavior.csv`)
- **Collection Method:** Real-time passive capture via `pynput` keyboard and mouse listeners during a configurable observation window (default: 60 seconds).
- **Schema:** `type` (key | mouse), `timestamp` (Unix epoch float), `x` (mouse x-coordinate or empty for key events), `y` (mouse y-coordinate or empty for key events).
- **Typical Volume:** ~1,400+ rows per 60-second observation window (approximately 8 key events and 1,400 mouse position samples in a moderately active session).
- **Storage:** Overwritten on each new collection cycle; not accumulated across sessions.

### Baseline Profile (`baseline.csv`)
- **Schema:** `avg_key_interval` (mean time in seconds between consecutive keystrokes), `avg_mouse_distance` (mean Euclidean pixel distance between consecutive mouse position samples).
- **Construction:** Aggregated from five independent observation windows collected during the training phase.
- **Sample Baseline Values:**

| avg_key_interval (s) | avg_mouse_distance (px) |
|---|---|
| 0.2457 | 7.5943 |
| 0.5502 | 5.2089 |
| 0.9864 | 6.9673 |

### Intermediate Feature Data (`features.csv`)
- **Schema:** Identical to `baseline.csv` — two columns representing the current window's extracted features.
- **Lifecycle:** Created by `feature_engineering.py` and consumed by `model.py` within the same monitoring cycle; deleted if the activity guard threshold is not met.

---

## System Architecture

```
┌──────────────────────────────────────────────────────┐
│                main.py  (Orchestrator)               │
│                                                      │
│  ┌──────────────────────────────────────────────┐   │
│  │         TRAINING PHASE  (first run)           │   │
│  │  Loop × 5:                                    │   │
│  │    data_collector.py → feature_engineering.py │   │
│  │  Concatenate results → baseline.csv           │   │
│  └──────────────────────────────────────────────┘   │
│                                                      │
│  ┌──────────────────────────────────────────────┐   │
│  │         MONITORING LOOP  (continuous)         │   │
│  │  data_collector.py    (60s passive capture)   │   │
│  │        ↓                                      │   │
│  │  feature_engineering.py  (feature extract)    │   │
│  │        ↓                                      │   │
│  │  model.py             (drift detection)       │   │
│  │        ↓                                      │   │
│  │  Console alert + 10s delay → repeat           │   │
│  └──────────────────────────────────────────────┘   │
└──────────────────────────────────────────────────────┘

Input Layer:      pynput Keyboard + Mouse Listeners
Feature Layer:    avg_key_interval, avg_mouse_distance
Detection Layer:  IsolationForest (StandardScaler normalised)
Output Layer:     Console classification + anomaly/stability score
Storage Layer:    behavior.csv, features.csv, baseline.csv (local)
```

---

## Model Workflow

### Phase 1 — Baseline Training

1. **Observation Collection** — `data_collector.py` starts background keyboard and mouse listeners. All key press timestamps and mouse (x, y, t) tuples are captured for 60 seconds, then written to `behavior.csv`.
2. **Feature Extraction** — `feature_engineering.py` reads `behavior.csv`, separates key events from mouse events, and applies activity threshold guards. If sufficient activity is present, it computes:
   - **avg_key_interval**: Mean of `numpy.diff()` applied to the sorted keystroke timestamp array (seconds between consecutive presses).
   - **avg_mouse_distance**: Mean of per-step Euclidean distances across the sequence of (x, y) mouse positions.
3. **Baseline Accumulation** — The extracted two-feature vector is saved to `features.csv`. After five such cycles, all vectors are concatenated into `baseline.csv`.

### Phase 2 — Continuous Monitoring

4. **New Window Collection** — Same 60-second passive capture procedure as Step 1, overwriting `behavior.csv`.
5. **Feature Extraction** — Same pipeline as Step 2. If the low-activity guard fires (insufficient keystrokes or total events), `features.csv` is deleted and the cycle skips inference.
6. **Normalisation** — `StandardScaler` is fitted on the baseline feature matrix. Both `baseline_scaled` and `current_scaled` are produced via `transform()`.
7. **Anomaly Detection** — `IsolationForest` is fitted on `baseline_scaled`. `predict()` returns `+1` (normal) or `-1` (anomalous) for `current_scaled`. `decision_function()` returns the raw anomaly score — negative values indicate greater anomaly depth.
8. **Output** — The terminal displays either `✅ NORMAL BEHAVIOR` with the stability score or `⚠️ INTENTION DRIFT DETECTED` with the absolute anomaly score.
9. **Cycle Delay** — A 10-second sleep separates consecutive monitoring cycles before the loop restarts.

---

## Implementation Details

**Modular Pipeline Design:** The system is decomposed into four single-responsibility scripts, each executable independently. `main.py` acts as a pure orchestrator, invoking the other three scripts via `os.system()` subprocess calls. This design allows individual modules to be tested, replaced, or extended without modifying the pipeline controller.

**Activity Threshold Guards:** Two guards are applied in `feature_engineering.py` before any feature computation occurs. A minimum of 10 key events (`LOW_KEY_EVENTS = 10`) and 300 total input events (`LOW_TOTAL_ACTIVITY = 300`) are required. Windows failing either threshold produce no `features.csv`, and `model.py` detects this absence and skips inference — preventing the model from classifying idle or low-data windows as anomalous.

**Isolation Forest Configuration:** The model uses `contamination=0.05`, indicating an expected 5% anomaly rate in the training data — a conservative estimate appropriate for a clean baseline collected under normal conditions. `n_estimators=200` provides a stable ensemble with low variance in anomaly scores. `random_state=42` ensures reproducibility of isolation partitions across monitoring cycles.

**Stateless Per-Cycle Model Fitting:** The Isolation Forest is re-fitted from the baseline on every monitoring cycle rather than persisting a serialised model. This design choice trades minor computational overhead for simplicity and eliminates model staleness without requiring a model update mechanism.

**Baseline Persistence:** `baseline.csv` is written once during the training phase and read on every subsequent monitoring cycle. If the file already exists on startup, the training phase is bypassed entirely, enabling the system to resume monitoring across reboots or session restarts without re-profiling.

---

## Results

| Metric | Value |
|---|---|
| Feature Dimensionality | 2 (`avg_key_interval`, `avg_mouse_distance`) |
| Baseline Sample Count | 5 observation windows |
| Observation Window Duration | 60 seconds per window |
| Monitoring Cycle Interval | 10 seconds between cycles |
| IsolationForest Estimators | 200 trees |
| Contamination Parameter | 0.05 (5% expected anomaly rate) |
| Minimum Key Events (guard) | 10 events per window |
| Minimum Total Events (guard) | 300 events per window |
| Inference Latency | Sub-second (2D feature vector) |
| Memory Footprint | Minimal — no GPU, no large model weights |
| External Dependencies | 4 Python packages |
| Output Format | Console classification + numeric anomaly/stability score |

The Isolation Forest approach is well-suited to this problem because it makes no assumptions about the distribution of normal behaviour, scales linearly with the number of training samples, and produces an interpretable continuous anomaly score alongside the binary classification. The two-feature representation captures complementary aspects of user behaviour: typing rhythm encodes cognitive pace and task engagement, while mouse displacement encodes physical interaction intensity.

---

## Installation

### Prerequisites

- Python 3.8 or higher
- `pip` package manager
- A system where keyboard and mouse input is available

> **Note:** On Linux, `pynput` may require membership in the `input` group to access device events. On macOS, the terminal or IDE running the script must be granted Accessibility permissions under `System Preferences → Security & Privacy → Privacy → Accessibility`.

**1. Clone the repository:**
```bash
git clone https://github.com/harisaigithub/human-intention-drift-detection.git
cd human-intention-drift-detection
```

**2. Create and activate a virtual environment (recommended):**
```bash
python -m venv venv
source venv/bin/activate        # Linux / macOS
venv\Scripts\activate           # Windows
```

**3. Install dependencies:**
```bash
pip install -r requirements.txt
```

---

## Usage

**Run the full pipeline:**
```bash
python main.py
```

**First run — training phase:**

The system detects that no `baseline.csv` exists and automatically initiates the training phase across five 60-second observation windows. Use your computer normally during each window.

```
========================================
 AI INTENTION DRIFT DETECTION SYSTEM
========================================
=== TRAINING PHASE ===
Collecting 5 baseline samples of normal behavior

--- Baseline Sample 1/5 ---
[RECORDING] Listening for 60 seconds...
[INFO] Type or move mouse to generate behavior data
...
[SYSTEM] Baseline training completed
```

**Subsequent runs — monitoring phase:**

Once `baseline.csv` exists, the system enters the continuous monitoring loop and produces one of the following outputs per cycle:

```
=============================================
✅ NORMAL BEHAVIOR
Stability Score: 0.0842
=============================================
```

or

```
=============================================
⚠️  INTENTION DRIFT DETECTED
Anomaly Score: 0.1563
=============================================
```

**Stop monitoring:**
```
Ctrl + C
```

**Run individual modules independently:**
```bash
python data_collector.py        # Collect one 60-second observation window
python feature_engineering.py   # Extract features from existing behavior.csv
python model.py                 # Run anomaly detection on existing features.csv
```

**Reset and re-profile the user:**
```bash
rm baseline.csv     # Linux / macOS
del baseline.csv    # Windows
python main.py      # Re-triggers the training phase on next run
```

---

## Future Enhancements

- **Extended Feature Set** — Incorporate click frequency, scroll velocity, dwell time per application, and typing error rate to increase detection resolution and reduce false negatives.
- **Adaptive Baseline Updating** — Implement a sliding window or exponential moving average mechanism to allow the baseline to evolve gradually over time, accommodating legitimate long-term behavioural shifts.
- **Multi-User Profile Management** — Support named user profiles stored in separate baseline files, enabling operation on shared machines with per-user anomaly models.
- **Configurable Parameters** — Expose observation window duration, monitoring interval, and activity thresholds as command-line arguments or a configuration file.
- **GUI Dashboard** — Develop a lightweight desktop dashboard that visualises real-time behaviour metrics, historical anomaly scores, and baseline feature distributions.
- **Persistent Model Serialisation** — Serialise the fitted `IsolationForest` and `StandardScaler` objects using `joblib` to eliminate per-cycle model re-fitting and reduce CPU overhead in long-running deployments.
- **Alert Escalation** — Integrate desktop notifications, email alerts, or webhook triggers upon detection of consecutive anomalous windows, enabling security operations integration.
- **Alternative Anomaly Algorithms** — Evaluate One-Class SVM, Local Outlier Factor, and Autoencoder-based approaches as complementary or ensemble detection strategies.
- **Deployment as a Background Service** — Package the system as a `systemd` service (Linux), `launchd` agent (macOS), or Windows Service for persistent, session-independent monitoring.

---

## Contributing

Contributions are welcome. Please follow the standard Git workflow below.

**1. Fork the repository:**
```bash
git clone https://github.com/harisaigithub/human-intention-drift-detection.git
```

**2. Create a feature branch:**
```bash
git checkout -b feature/your-feature-name
```

**3. Commit your changes:**
```bash
git add .
git commit -m "feat: describe your change clearly"
```

**4. Push and open a Pull Request:**
```bash
git push origin feature/your-feature-name
```

Please adhere to PEP 8 coding conventions and ensure that existing pipeline functionality is unaffected by new changes.

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