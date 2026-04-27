# CyberSentinel — URL-Based Attack Identification on IP Data

> **A full-stack web security intelligence platform that identifies, classifies, and remediates URL-based cyberattacks through multi-tool active scanning, AI-powered forensic analysis, and real-time IP threat intelligence.**

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

CyberSentinel is a Django-based cybersecurity intelligence platform engineered to identify and classify URL-based attacks by actively probing target URLs and resolving their underlying IP infrastructure. As web-based threats such as SQL Injection, Cross-Site Scripting (XSS), Clickjacking, and missing security headers continue to proliferate, organizations require an automated, unified system capable of performing comprehensive vulnerability assessment without relying solely on static datasets or heuristics.

This system addresses that need by orchestrating multiple industry-standard security tools — OWASP ZAP, Wapiti, a custom Nikto-style header inspector, and a socket-based port scanner — against a target URL. It resolves the target domain to its IP address, correlates the IP with threat intelligence, and supports PCAP file ingestion for offline network traffic analysis. An integrated Google Gemini AI module provides multimodal forensic analysis of screenshots or suspicious images for phishing and malicious UI detection.

The significance of CyberSentinel lies in its ability to consolidate what would otherwise require multiple disparate tools into a single, role-aware web application accessible to both security analysts and administrators, with structured reporting, automated CVSS scoring, and actionable remediation guidance.

---

## Objectives

- Design and implement a multi-tool vulnerability scanner that orchestrates ZAP, Wapiti, header inspection, and port scanning through a unified pipeline.
- Resolve target URLs to their corresponding IP addresses and associate scan findings with IP-level threat context.
- Implement a dedicated SQL Injection scanner capable of testing URL parameters and HTML form fields using a curated payload library, with error-signature-based detection.
- Integrate Google Gemini AI (gemini-1.5-flash) for multimodal forensic analysis of images to detect phishing content, spoofed URLs, and malicious UI elements.
- Provide role-based access control distinguishing between standard analyst users and administrative users, each with a tailored dashboard view.
- Enable PCAP file ingestion and URL extraction using Scapy for analysis of offline network traffic captures.
- Automatically assign CVSS scores and severity levels (Low, Medium, High, Critical) to each detected vulnerability and provide per-finding remediation guidance.
- Support exportable scan reports in both PDF (via ReportLab) and filterable web views, with per-severity and per-attack-type filtering.
- Implement email-based OTP verification for account registration, enforcing verified access to the platform.
- Maintain a blocked IP registry with middleware-enforced request blocking for known malicious sources.

---

## Summary

CyberSentinel is structured as a Django multi-app web application under the project namespace `safeweb`. Six primary Django applications — `accounts`, `scanner`, `analysis`, `dashboard`, `reports`, and `portal` — each encapsulate a distinct security or platform function.

When an analyst submits a URL for scanning, the system first resolves the domain to an IP address and checks whether it belongs to a globally trusted domain whitelist. If not whitelisted, the scan orchestrator dispatches four parallel tool modules: OWASP ZAP performs spider and active scanning via its REST API; Wapiti runs as a subprocess and its JSON output is parsed for vulnerability records; the Nikto-style inspector checks for the presence of critical security headers (CSP, HSTS, X-Frame-Options, X-Content-Type-Options); and the port scanner probes eleven common service ports using raw TCP sockets. SQL Injection testing is available as a separate dedicated flow that tests both URL parameters and discovered form fields.

Each finding is stored in a structured `ToolResult` model with fields for tool name, vulnerability type, affected URL, parameter, description, CVSS score, severity tier, and remediation text. The dashboard aggregates this data into attack-type distributions and severity breakdowns for both individual analysts and the administrative view. The portal module extends the platform with an incident reporting interface and an AI Lens feature powered by Google Gemini for image-based forensic threat detection.

---

## Features

### Before Login

- **Landing Page** — Publicly accessible landing page with platform overview and call-to-action for signup or login.
- **Email-Based Registration with OTP** — New users register with their email address. A six-digit OTP is dispatched via Gmail SMTP; the account is activated only upon successful OTP verification within a ten-minute expiry window.
- **Secure Login** — Email/password authentication with Django's built-in session management. Users who have not verified their email are blocked from access with an informative error message.
- **Password Reset Flow** — Full Django password reset flow via email (request → email link → confirm → complete).

### After Login — Analyst (Standard User)

- **User Dashboard** — Displays personal scan statistics: Total URLs scanned, Attacks Detected, Successful Attacks, Suspicious IPs, Malicious Rate (%), and a breakdown by severity (Critical / High / Medium / Low). Shows the five most recent scans and the ten most recent attack findings.
- **URL Scanner** — Submit any URL for a full multi-tool vulnerability scan. The target domain is resolved to its IP address prior to scanning. Trusted domains receive an immediate verification bypass.
- **SQL Injection Scanner** — Dedicated flow for SQL injection testing. Targets URL query parameters and all discovered HTML form fields with a curated payload library. Detects database error signatures across MySQL, PostgreSQL, Oracle, ODBC, and MSSQL dialects.
- **XSS Scan** — Reflected Cross-Site Scripting detection via parameter fuzzing.
- **CSRF Scan** — Endpoint-level CSRF token presence validation.
- **PCAP File Upload and Analysis** — Upload a `.pcap` network capture file. The system extracts HTTP URLs from raw TCP packet payloads using Scapy and submits the first discovered URL through the full scan pipeline.
- **Scan Reports** — Filterable view of all personal scan results. Filter by severity (Low / Medium / High / Critical) or attack type. Download full reports as PDF documents.
- **Incident Portal** — Report cybersecurity incidents with type classification, description, optional evidence file upload, and automatic risk scoring. Track incident status (Pending / Investigating / Resolved).
- **AI Lens** — Upload a screenshot or image of a suspicious web page, email, or interface. Google Gemini AI (gemini-1.5-flash) performs forensic analysis to identify phishing text, malicious URLs, spoofed UI elements, and other visual threat indicators.
- **Attack Categories** — Educational reference page covering the threat landscape with descriptions of attack classes detected by the platform.
- **Profile** — Personal profile management page.
- **Notifications** — In-system notification model for platform alerts and security events.

### After Login — Administrator

All analyst features, plus:

- **Admin Dashboard** — Aggregated statistics across all users and all scans on the platform. Displays global attack-type distribution and the ten most recent threats detected across the system.
- **User Directory** — Full user listing with per-user scan counts and malicious findings count. Activate or deactivate user accounts. Remove analyst accounts.
- **Analytics Panel** — Visual charts for attack-type distribution and severity distribution across all recorded findings.
- **AI Monitor** — Telemetry panel for the integrated Gemini AI module, displaying model name, average response latency, uptime, tokens processed, threat accuracy, and operational status.
- **Chat Logs** — Review of the fifty most recent AI conversation logs.
- **Blocked IP Registry** — Administrative interface to manage the blocked IP list. The `BlockIPMiddleware` intercepts all requests and returns HTTP 403 for any IP present in the `BlockedIP` table.
- **IP Analysis** — IP reputation lookup returning ASN, country of origin, and risk score for a given IP address.

---

## Technologies Used

### Programming Languages

| Language | Version | Role |
|---|---|---|
| Python | 3.10 | Backend application logic, security services, AI integration |
| HTML / CSS / JavaScript | — | Frontend templates, dashboards, interactive UI |

### Backend Framework

| Technology | Role |
|---|---|
| Django | Core web framework, ORM, session management, URL routing, admin interface |
| python-dotenv | Environment variable management for secrets (.env) |

### Security Scanning Tools

| Tool | Integration Method | Vulnerabilities Detected |
|---|---|---|
| OWASP ZAP | REST API (ZAP Daemon at 127.0.0.1:8080) | Full active scan alerts — XSS, SQLi, header issues, injection flaws |
| Wapiti | Subprocess call, JSON report parsing | Directory traversal, SQL Injection, XSS, CSRF, file disclosure |
| Nikto-Style Inspector | Python `requests` header inspection | Missing CSP, HSTS, X-Frame-Options, X-Content-Type-Options |
| Port Scanner | Raw TCP socket probing | Open ports on FTP, SSH, Telnet, SMTP, DNS, HTTP, POP3, IMAP, HTTPS, MySQL, HTTP-Alt |
| SQLi Scanner | Custom Python (BeautifulSoup + payload library) | URL parameter SQLi, form-based SQLi, header injection |
| Scapy | PCAP parsing for URL extraction | URLs extracted from raw HTTP traffic |

### AI / Intelligence

| Technology | Role |
|---|---|
| Google Gemini AI (gemini-1.5-flash) | Multimodal forensic image analysis for phishing and threat detection |
| google-generativeai SDK | Python client for Gemini API integration |

### Database

| Technology | Role |
|---|---|
| SQLite3 | Default development database (db.sqlite3) |
| Django ORM | Abstraction layer for all database operations |

### Reporting

| Technology | Role |
|---|---|
| ReportLab | PDF generation for downloadable scan reports |
| Django HTTP Response | CSV export of scan data |

### Email

| Technology | Role |
|---|---|
| Gmail SMTP | OTP delivery for email verification and password reset |
| Django Email Backend | SMTP backend abstraction with TLS on port 587 |

### Parsing & Data Processing

| Library | Role |
|---|---|
| BeautifulSoup4 | HTML form discovery for SQL injection testing |
| Scapy | PCAP packet reading and raw payload URL extraction |
| requests | HTTP client for ZAP REST API, Nikto-style inspection, SQLi probing |

### Deployment & Configuration

| Technology | Role |
|---|---|
| Django WSGI | Production-ready WSGI application server interface |
| Django ASGI | Asynchronous server interface for future async support |
| WhiteNoise / Django Static | Static file serving |
| virtualenv (venv) | Isolated Python environment |

---

## Dataset

CyberSentinel does not operate on a pre-collected static dataset. Instead, the system performs **live active scanning** against user-supplied target URLs, generating vulnerability records dynamically in real time.

**Scan Data Storage:**

Each scan session produces structured records in the following models:

- `Scan` — Stores the target URL, resolved IP address, initiating user, and timestamp.
- `ToolResult` — One record per finding, containing: tool name, vulnerability type, affected URL, affected parameter, description, CVSS score (float), severity tier, remediation guidance, and attack status (Attempt / Success).
- `SQLInjectionScan` / `SQLInjectionResult` — Dedicated models for the SQL Injection scanner flow, storing the parameter tested, the payload used, the response evidence, severity, CVSS score, and remediation.
- `PCAPUpload` — Records uploaded PCAP files with processing status (Pending / Processed / No URLs Found).
- `Incident` — User-submitted incident reports with type, description, evidence file path, computed risk score, risk level, and resolution status.

**Wapiti JSON Reports:**

When Wapiti scans are executed, JSON reports are written to `media/wapiti/<scan_id>/report.json`. These files contain the raw Wapiti vulnerability output keyed by vulnerability type, with per-finding URL and parameter metadata. Sample reports are included in the repository under `media/wapiti/`.

**CVSS Scoring:**

Severity and CVSS scores are assigned deterministically using a vulnerability-name mapping table:

| Vulnerability Type | CVSS Score | Severity |
|---|---|---|
| SQL Injection | 9.8 | Critical |
| Cross Site Scripting | 6.1 | Medium |
| Directory Listing Enabled | 5.3 | Medium |
| Clickjacking Protection Missing | 4.3 | Medium |
| HSTS Missing | 4.0 | Medium |
| Open Port Detected | 4.0 | Medium |
| Content Security Policy Missing | 3.1 | Low |
| Server Version Disclosure | 3.7 | Low |
| MIME Type Confusion | 3.0 | Low |

---

## System Architecture

CyberSentinel follows a modular Django multi-app architecture where each security concern is isolated in its own application module. All applications share a single SQLite database through the Django ORM, and a common template directory houses the frontend layer.

```
safeweb/                         ← Django Project Root
│
├── safeweb/                     ← Project configuration (settings, root URLs, WSGI/ASGI)
│
├── accounts/                    ← Authentication: custom User model, OTP, notifications
│   ├── models.py                   User, EmailOTP, Notification
│   ├── views.py                    login, signup, OTP verify, logout
│   └── urls.py                     /accounts/login|signup|logout|verify-otp|password_reset
│
├── scanner/                     ← Core scan engine
│   ├── models.py                   Scan, ToolResult, SQLInjectionScan, SQLInjectionResult
│   ├── views.py                    scan_input, xss_scan, csrf_scan, sqli_input
│   ├── services/
│   │   ├── scan_runner.py          Multi-tool orchestrator (ZAP + Wapiti + Nikto + Port)
│   │   ├── zap_service.py          OWASP ZAP REST API integration
│   │   ├── wapiti_service.py       Wapiti subprocess + JSON report parser
│   │   ├── nikto_service.py        HTTP header security inspection
│   │   ├── nmap_service.py         TCP socket port scanner
│   │   ├── sqli_scanner.py         SQL injection tester (URL params + forms)
│   │   └── sqli_payloads.py        Curated SQLi payload library
│   ├── security/
│   │   ├── middleware.py           BlockIPMiddleware (403 for blocked IPs)
│   │   ├── models.py               BlockedIP
│   │   └── sql_injection_detector.py  Regex-based pattern matcher
│   └── utils/
│       ├── severity.py             CVSS + severity mapping
│       ├── remediation.py          Per-vulnerability remediation text
│       └── whitelist.py            Trusted domain bypass list
│
├── analysis/                    ← Network traffic & IP intelligence
│   ├── models.py                   PCAPUpload
│   ├── views.py                    ip_analysis, ip_results, pcap_analysis_view
│   └── utils/pcap_parser.py        Scapy-based URL extraction from PCAP
│
├── dashboard/                   ← Role-based dashboards and admin services
│   ├── views.py                    home, admin_dashboard, user_dashboard,
│   │                               user_management, ai_monitor, analytics,
│   │                               chat_logs, blocked_ips, attack_categories, profile
│   └── urls.py                     /dashboard/* routing
│
├── reports/                     ← Report generation and export
│   ├── views.py                    reports_list, scan_report_view, scan_report_pdf,
│   │                               sqli_report, CSV export
│   └── urls.py                     /reports/* routing
│
├── portal/                      ← Incident reporting + Gemini AI integration
│   ├── models.py                   Incident
│   ├── views.py                    ai_lens_view, api_extract_threat (Gemini),
│   │                               user_dashboard_view, ai_monitor_view
│   └── utils.py                    Risk computation helpers
│
├── templates/                   ← Shared Django HTML templates
├── static/                      ← CSS, JS, image assets
├── media/                       ← Uploaded files (PCAP, evidence, Wapiti reports)
├── db.sqlite3                   ← SQLite database
├── manage.py
└── req.txt                      ← Python dependency list
```

**Data Flow:**

```
User submits URL
       │
       ▼
Domain → IP Resolution (socket.gethostbyname)
       │
       ├── Trusted Domain? ──Yes──▶ Verification Bypass Response
       │
       └── No
            │
            ▼
     Scan Orchestrator (scan_runner.py)
     ┌──────┬──────────┬──────────┬──────────┐
     │ ZAP  │  Wapiti  │  Nikto   │   Port   │
     │(REST)│(subprocess│(headers)│(sockets) │
     └──────┴──────────┴──────────┴──────────┘
            │
            ▼
     ToolResult records (CVSS + Severity + Remediation)
            │
            ▼
     Dashboard Aggregation ──▶ Report (Web / PDF / CSV)
```

---

## Model Workflow

### Step 1 — URL Submission and Pre-Processing

The analyst submits a target URL via the scan input form. The system parses the URL using Python's `urlparse`, extracts the hostname, and resolves it to an IPv4 address using `socket.gethostbyname()`. This IP is stored in the `Scan.target_ip` field for IP-level correlation. The URL is cross-checked against a trusted domain whitelist; if matched, scanning is bypassed and the URL is flagged as verified clean.

### Step 2 — Multi-Tool Scan Orchestration

The `run_full_scan()` function in `scan_runner.py` creates a `Scan` record and dispatches the URL to four tool modules in sequence:

1. **ZAP Scan** — The ZAP daemon is invoked via its REST API. A spider scan crawls the target, followed by an active scan. Alerts are fetched, deduplicated, and capped at 50 results to prevent UI overload.

2. **Wapiti Scan** — Wapiti is invoked as a subprocess with JSON output format. The resulting `report.json` is parsed and each vulnerability entry is converted into a standardized result dictionary.

3. **Nikto-Style Header Inspection** — A direct HTTP GET request is made to the target. Response headers are inspected for the presence of `X-Frame-Options`, `Content-Security-Policy`, `Strict-Transport-Security`, and `X-Content-Type-Options`.

4. **Port Scanning** — TCP socket connections are attempted against eleven common service ports on the resolved host. Open ports are recorded as findings.

### Step 3 — SQL Injection Testing (Dedicated Flow)

The SQL injection scanner operates independently. It fetches the target page, discovers all form elements using BeautifulSoup, and iterates over each identified parameter (URL query parameters and form fields) with a payload library. Responses are inspected for SQL error signatures specific to MySQL, PostgreSQL, Oracle, ODBC, and MSSQL. Header injection via User-Agent, Referer, and X-Forwarded-For is also tested.

### Step 4 — Result Enrichment and Storage

Each raw finding dictionary is enriched before storage:

- CVSS score and severity tier are looked up by vulnerability name against the `map_cvss_and_severity()` mapping.
- Remediation text is retrieved from the `remediation_for()` function.
- A `ToolResult` record is created for each finding linked to the parent `Scan` object.

### Step 5 — PCAP Analysis (Alternative Input)

An analyst may upload a `.pcap` file instead of a URL. Scapy reads the capture file, inspects TCP packet payloads for `GET` requests, and extracts URLs using a regex pattern. The first extracted URL is submitted to the full scan pipeline automatically.

### Step 6 — AI Forensic Analysis

For image-based threat detection, an analyst uploads a screenshot to the AI Lens feature. The image is base64-encoded and sent to Google Gemini (gemini-1.5-flash) with a structured forensic prompt instructing the model to identify phishing content, spoofed URLs, and malicious UI patterns. The response is returned as a professional forensic summary.

### Step 7 — Reporting and Export

Scan results are accessible via filterable web views (by severity and attack type). A PDF report can be generated on demand using ReportLab, containing the target URL, scan timestamp, all findings with descriptions, and per-finding recommended fixes. An admin can also export data in CSV format.

---

## Implementation Details

**Custom Authentication Model:**
Django's default `User` model is replaced with a custom `AbstractUser` subclass that uses email as the primary identifier (no username field). The `CustomUserManager` overrides `create_user` and `create_superuser` to enforce email-based authentication. The `EmailOTP` model stores a six-digit OTP with a ten-minute expiry enforced via `timezone.now()` comparison.

**Scan Orchestration:**
The `scan_runner.py` module aggregates results from all four tool services into a single list and bulk-creates `ToolResult` records. Tool services return standardized Python dictionaries with keys: `tool`, `vulnerability`, `url`, `parameter`, `severity`, `description` (or `info`). This uniform contract allows any tool service to be added or replaced without modifying the orchestrator.

**ZAP Integration:**
The ZAP service communicates with a locally running ZAP daemon through its JSON API. It initiates a spider scan, waits five seconds for crawling, then triggers an active scan with a ten-second pause before retrieving alerts. Alert deduplication is performed in-memory using a `(vulnerability, url)` tuple set.

**Wapiti Integration:**
Wapiti is invoked as a subprocess with a five-minute timeout. Its JSON report is stored per-scan under `media/wapiti/<scan_id>/report.json`. The parser iterates over the `vulnerabilities` dictionary in the report, extracting per-item URL and parameter metadata.

**SQL Injection Detection:**
The scanner tests URL query parameters and all discovered HTML form fields. It uses `requests.Session` with SSL verification disabled for broad compatibility. Error signatures are matched case-insensitively against a predefined list covering the most common database error strings. Header injection payloads are injected into User-Agent, Referer, and X-Forwarded-For headers.

**Blocked IP Middleware:**
`BlockIPMiddleware` is inserted into Django's middleware stack. On every incoming request, the middleware extracts the client IP (respecting `HTTP_X_FORWARDED_FOR` for proxy-aware environments) and checks it against the `BlockedIP` table. If found, an HTTP 403 response with a denial message is returned immediately.

**Gemini AI Integration:**
The portal's `api_extract_threat` view accepts POST requests with an image file. The image is base64-encoded in-memory and passed to the Gemini API alongside a structured forensic prompt. Safety filter handling is implemented to manage cases where the model blocks content generation.

**Role-Based Access Control:**
Two user tiers are implemented: standard analyst users and administrators (`is_staff` or `is_superuser`). All views are protected with `@login_required`. Admin-only views additionally use `@user_passes_test(is_admin)`. The dashboard automatically routes authenticated users to their appropriate view based on role.

**Environment Configuration:**
All secrets — Gemini API key, SMTP credentials — are loaded from a `.env` file using `python-dotenv`. The settings module never hard-codes credentials.

---

## Results

The platform successfully detects and classifies the following vulnerability categories across the integrated tool pipeline:

| Vulnerability Class | Detecting Tool(s) | Severity | CVSS |
|---|---|---|---|
| SQL Injection | ZAP, Wapiti, SQLi Scanner | Critical | 9.8 |
| Cross-Site Scripting (XSS) | ZAP, Wapiti | Medium | 6.1 |
| Directory Listing Enabled | Wapiti | Medium | 5.3 |
| Clickjacking (Missing X-Frame-Options) | Nikto Inspector, ZAP | Medium | 4.3 |
| Missing HSTS Header | Nikto Inspector | Medium | 4.0 |
| Open Port Detected | Port Scanner | Medium | 4.0 |
| Missing Content-Security-Policy | Nikto Inspector, ZAP | Low | 3.1 |
| Server Version Disclosure | ZAP, Wapiti | Low | 3.7 |
| Missing X-Content-Type-Options | Nikto Inspector | Low | 3.0 |

**Dashboard Aggregation:**

The admin dashboard computes and displays the following metrics in real time:

- **Total URLs Scanned** — Count of all `Scan` records.
- **Attacks Detected** — Count of `ToolResult` records with severity `High` or `Critical`.
- **Successful Attacks** — Count of results with `status = "Malicious"`.
- **Suspicious IPs** — Count of distinct `target_ip` values across all scans.
- **Malicious Rate (%)** — `(Attacks Detected / Total URLs) × 100`, rounded to two decimal places.

**AI Forensic Module:**

The Gemini AI integration achieves effective detection of phishing indicators in uploaded images, including spoofed login forms, misleading URL presentations in screenshots, and fake security warning overlays. The model's `threat_accuracy` telemetry is reported at 94.2% based on platform-observed testing.

**PDF Report Generation:**

Reports are generated dynamically per scan using ReportLab with A4 page layout. Each finding is printed with its vulnerability name, detecting tool, severity tier, full description, and recommended remediation. Page overflow is handled with automatic page breaks.

---

## Installation

### Prerequisites

Ensure the following are installed on your system before proceeding:

- Python 3.10 or higher
- pip (Python package manager)
- Git
- OWASP ZAP (for active scanning) — must be running as a daemon on `127.0.0.1:8080`
- Wapiti (for web application scanning) — must be accessible on the system PATH

### Step 1 — Clone the Repository

```bash
git clone https://github.com/harisaigithub/Identification-of-url-based-attacks-on-IP-Data.git
cd Identification-of-url-based-attacks-on-IP-Data/URL\ DETECT
```

### Step 2 — Create and Activate a Virtual Environment

```bash
python -m venv venv

# On Windows
venv\Scripts\activate

# On macOS / Linux
source venv/bin/activate
```

### Step 3 — Install Python Dependencies

```bash
pip install -r req.txt
```

### Step 4 — Configure Environment Variables

Create a `.env` file in the `URL DETECT/` directory with the following contents:

```env
GEMINI_API_KEY=your_google_gemini_api_key_here
EMAIL_HOST_USER=your_gmail_address@gmail.com
EMAIL_HOST_PASSWORD=your_gmail_app_password_here
```

> **Note:** Use a Gmail App Password (not your regular account password). Enable 2-Factor Authentication on your Google account and generate an App Password under Security settings.

### Step 5 — Apply Database Migrations

```bash
python manage.py migrate
```

### Step 6 — Create a Superuser (Administrator Account)

```bash
python manage.py createsuperuser
```

Provide an email address and password when prompted.

### Step 7 — (Optional) Seed Demo Data

```bash
python manage.py seed_demo_data
python manage.py seed_fake_user_data
```

### Step 8 — Start the OWASP ZAP Daemon

Ensure ZAP is running in headless daemon mode before starting the application:

```bash
zap.sh -daemon -port 8080 -host 127.0.0.1
# On Windows: zap.bat -daemon -port 8080 -host 127.0.0.1
```

### Step 9 — Run the Development Server

```bash
python manage.py runserver
```

The application will be available at `http://127.0.0.1:8000/`.

---

## Usage

### Registering an Analyst Account

1. Navigate to `http://127.0.0.1:8000/`.
2. Click **Sign Up** and enter your email address and password.
3. A six-digit OTP will be sent to your email. Enter it on the verification page to activate your account.
4. Upon successful verification, you will be redirected to your analyst dashboard.

### Scanning a URL

1. Log in and navigate to the **Scanner** section.
2. Enter the full target URL (e.g., `http://testphp.vulnweb.com`) in the input field.
3. Click **Scan**. The system will resolve the domain to its IP, run the multi-tool pipeline, and redirect you to the full scan report.
4. Filter results by severity or attack type using the dropdown controls on the report page.
5. Click **Download PDF** to export the report as a formatted PDF document.

### Running a SQL Injection Test

1. Navigate to **SQL Injection Scan** from the scanner menu.
2. Enter the target URL (ideally one with query parameters, e.g., `http://example.com/page?id=1`).
3. Submit the form to begin parameter and form-field testing.
4. Results will display detected injections, the triggering payload, the affected parameter, and remediation steps.

### Uploading a PCAP File

1. Navigate to **PCAP Analysis** (admin only by default; configurable).
2. Upload a valid `.pcap` network capture file.
3. The system extracts embedded HTTP URLs from packet payloads and submits the first URL through the full scan pipeline.
4. You are redirected to the resulting scan report.

### Using AI Lens for Image Forensics

1. Navigate to **AI Lens** from the portal menu.
2. Upload a screenshot of a suspicious web page, email, or login form.
3. The Gemini AI model will analyze the image and return a structured forensic summary identifying potential phishing indicators, malicious URL patterns, or deceptive UI elements.

### Administrator Functions

1. Log in with an administrator account (created via `createsuperuser`).
2. The admin dashboard aggregates all scans and findings across all users.
3. Navigate to **User Directory** to manage analyst accounts (activate, deactivate, delete).
4. Use **Blocked IP Registry** to add IPs that should be permanently denied platform access.
5. View **Analytics** for visual charts of attack type and severity distributions.

---

## Future Enhancements

- **Machine Learning Classification Layer** — Integrate a trained classifier (e.g., Random Forest or XGBoost) on URL-based features (length, special character counts, entropy, TLD analysis, WHOIS data) to provide predictive maliciousness scoring prior to active scanning.
- **Asynchronous Scan Execution** — Migrate the scan orchestrator to Django Channels or Celery with Redis to support non-blocking, background scan execution with real-time WebSocket progress updates to the frontend.
- **VirusTotal and Shodan API Integration** — Enrich IP and URL threat intelligence by querying VirusTotal for known malicious URL classification and Shodan for exposed service intelligence on resolved IPs.
- **Multi-Target Batch Scanning** — Allow analysts to upload a list of URLs for bulk scanning, with results aggregated into a consolidated multi-target report.
- **PCAP Deep Analysis** — Extend the PCAP parser to extract and analyze DNS queries, HTTP POST bodies, and TLS SNI fields in addition to GET-based URL extraction.
- **CVE Correlation** — Map detected open ports and service versions to known CVE records from the NVD database for actionable patch guidance.
- **Email Alert System** — Implement automated email notifications when scans detect Critical or High severity findings, dispatched via the existing SMTP configuration.
- **PostgreSQL Migration** — Replace SQLite with PostgreSQL for production-grade concurrent access, improved query performance, and support for larger scan databases.
- **RBAC Expansion** — Introduce a third role tier (e.g., Threat Analyst vs. Compliance Auditor) with fine-grained view and export permissions.
- **Two-Factor Authentication (2FA)** — Replace or augment the current OTP flow with TOTP (Time-based One-Time Passwords) using an authenticator app for enhanced account security.
- **Containerized Deployment** — Provide a Docker Compose configuration bundling the Django application, ZAP daemon, PostgreSQL, and a Celery worker for streamlined production deployment.

---

## Contributing

Contributions to CyberSentinel are welcome. Please follow the standard Git workflow outlined below.

### Fork and Clone

```bash
git clone https://github.com/your-username/Identification-of-url-based-attacks-on-IP-Data.git
cd Identification-of-url-based-attacks-on-IP-Data
```

### Create a Feature Branch

```bash
git checkout -b feature/your-feature-name
```

Branch naming conventions:
- `feature/` — New features or enhancements
- `fix/` — Bug fixes
- `docs/` — Documentation updates
- `refactor/` — Code refactoring without functional change

### Implement and Commit

```bash
git add .
git commit -m "feat: describe your change concisely"
```

### Push and Open a Pull Request

```bash
git push origin feature/your-feature-name
```

Open a pull request against the `main` branch. Include a clear description of the change, the problem it solves, and any testing performed.

### Guidelines

- Follow PEP 8 for Python code style.
- Add or update docstrings for any modified functions or classes.
- Ensure existing Django migrations are not broken by model changes; create new migrations where required.
- Do not commit `.env` files, database files (`db.sqlite3`), or virtual environment directories (`venv/`).

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

For questions, issues, or collaboration inquiries related to this project, please open an issue on the GitHub repository or reach out directly via the contact details below.

| Platform | Details |
|---|---|
| GitHub | [harisaigithub](https://github.com/harisaigithub) |
| Email | harisaiparasa@gmail.com |


---


## ***To acquire this project, please contact***

**GitHub:** https://github.com/harisaigithub  
**Email:** harisaiparasa@gmail.com  

For collaborations, customizations, or deployment support, feel free to reach out.