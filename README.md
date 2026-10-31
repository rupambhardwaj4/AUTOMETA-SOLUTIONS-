# Autometa School Enterprise Resource Planning (ERP) Platform

### *Cloud-Native, Schema-Isolated Multi-Tenant School Operating System*

[![Baseline](https://img.shields.io/badge/Baseline-v3.0-blue.svg)](prd.md)
[![Python](https://img.shields.io/badge/Python-3.13_LTS-3776AB.svg?logo=python&logoColor=white)](https://www.python.org/)
[![Django](https://img.shields.io/badge/Django-5.x-092E20.svg?logo=django&logoColor=white)](https://www.djangoproject.com/)
[![PostgreSQL](https://img.shields.io/badge/PostgreSQL-15%2B_(Schema_Isolated)-4169E1.svg?logo=postgresql&logoColor=white)](https://www.postgresql.org/)
[![Tailwind CSS](https://img.shields.io/badge/Tailwind_CSS-3.4%2B-38B2AC.svg?logo=tailwind-css&logoColor=white)](https://tailwindcss.com/)
[![PWA](https://img.shields.io/badge/PWA-Offline--First_(IndexedDB)-5A0FC8.svg?logo=pwa&logoColor=white)](architecture.md)
[![Compliance](https://img.shields.io/badge/Compliance-DPDP_Act_2023_%7C_ISO_27001-success.svg)](prd.md)

---

## 1. Overview & Executive Vision

The **Autometa School ERP Platform** is an enterprise-grade, cloud-native Software-as-a-Service (SaaS) operating system designed to manage academic, administrative, operational, and financial operations across individual campuses and federated multi-campus groups.

It supports diverse educational boards (**CBSE, ICSE, IB, and State Boards**) while providing field-resilient offline PWA capabilities for teachers, double-entry financial governance for accountants, and strict statutory data protection under India's **Digital Personal Data Protection (DPDP) Act, 2023**.

---

## 2. Core Architectural Pillars

```text
+------------------------------------------------------------------------+
|                        AUTOMETA SYSTEM TOPOLOGY                        |
|                                                                        |
|   [ Client Layer ]          [ Application Layer ]     [ Data Layer ]   |
|   * Semantic HTML5          * Python 3.13 / Django 5  * PostgreSQL 15+ |
|   * Tailwind CSS (Tokens)   * DRF REST & SSE Endpoints* Dynamic Schemas|
|   * Vanilla ES6+ JS         * Request Schema Router   * PgBouncer Pool |
|   * PWA Service Worker      * Celery 5.x Task Nodes   * Redis 7+ Cache |
|   * Local IndexedDB Queue   * Immutable Ledger Engine * WAL Archiving  |
+------------------------------------------------------------------------+
```

1. **PostgreSQL Schema Multi-Tenancy**: Every school campus operates inside an isolated PostgreSQL schema (`tenant_<school_id>`). Cross-schema joining is strictly prohibited at the database level.
2. **Offline-First Field Workspaces (PWA)**: Teachers can mark roll-call attendance (FR-07) and input examination marks (FR-09) with zero network connectivity. Data is safely queued in IndexedDB and synchronized automatically upon reconnection.
3. **Double-Entry Immutable Financial Ledgers**: Fee management operations conform to strict double-entry bookkeeping (`fee_ledger_entries`). Raw SQL `UPDATE` and `DELETE` queries are blocked by database triggers; only compensating transactions are permitted.
4. **DPDP Act 2023 Statutory Compliance**: Native enforcement of Verifiable Parental Consent (VPC), cryptographic consent text hashing (`vpc_consent_token`), zero third-party advertising tags, and automated right-to-erasure anonymization pipelines.
5. **Real-Time Timetable & Substitution Engine**: Master schedule generation with hard/soft period collision validation, and sub-3-second teacher leave substitution calculation.

---

## 3. Technology Stack

| Layer | Technology | Key Capabilities |
| :--- | :--- | :--- |
| **Backend Framework** | **Python 3.13 / Django 5.x** | Enterprise ORM, secure session handling, dynamic schema middleware, and modular architecture. |
| **API Architecture** | **Django REST Framework (DRF)** | Standardized RESTful JSON APIs, token-based authentication, and granular RBAC evaluation. |
| **Database** | **PostgreSQL 15+** | Relational integrity, JSONB support for student demographics, and schema-isolated multi-tenancy. |
| **Connection Pooling** | **PgBouncer** | Transaction-level connection multiplexing (20:1 ratio) keeping database CPU overhead < 5%. |
| **Caching & Messaging** | **Redis 7+** | 60-second KPI snapshot caching, short-lived OTP validation, session blacklisting, and Celery broker. |
| **Task Queue** | **Celery 5.x** | Asynchronous workers handling batch PDF report cards, payroll calculations, and notification cascades. |
| **Frontend Presentation** | **HTML5 + Tailwind CSS** | Server-rendered semantic templates with Tailwind utility styling (< 40 KB gzipped bundle). |
| **Frontend Runtime** | **Vanilla ES6+ JavaScript** | Zero framework bloat, targeting smooth 50+ FPS performance on low-end 2 GB RAM Android devices. |
| **Offline Storage** | **Browser IndexedDB (`idb`)** | Transactional client-side storage for rosters, marks, and pending mutations (up to 5,000 records). |
| **PWA Subsystem** | **Service Workers (`sw.js`)** | App Shell caching via CacheStorage API, background sync, and Firebase Cloud Messaging (FCM) web push. |

---

## 4. Documentation Suite

Detailed engineering and product documentation is available in the repository:

* [**`prd.md`**](prd.md) -- **Product Requirements Document**: Complete functional matrix (FR-01 to FR-21), non-functional requirements (NFRs), quantitative latency benchmarks, Core Web Vitals, and Failure Modes & Effects Analysis (FMEA).
* [**`architecture.md`**](architecture.md) -- **System Architecture & Application Flows**: Complete system design specifications with 6 end-to-end Mermaid sequence diagrams (Tenant Routing, Offline Sync, Invoicing & Webhooks, DPDP Lifecycle, Substitution Solver, Notification Cascade).
* [**`design.md`**](design.md) -- **Design System & UI/UX Guidelines**: Color palette tokens, font typography hierarchy (`Inter`, `JetBrains Mono`, `Merriweather`), responsive breakpoints (360px to 1920px+), PWA connectivity banners, and `tailwind.config.js`.
* [**`phase.md`**](phase.md) -- **Project Phasing & WBS (32-Week Baseline)**: Work breakdown structure across 6 phases (Phase 0 to Phase 5), milestone gates (Gate 0 through Go-Live), cross-phase dependencies, and resource allocations.
* [**`rules.md`**](rules.md) -- **Engineering Rules, Libraries & AI Boundaries**: Approved library whitelist vs. prohibitions, defensive error handling patterns, DPDP statutory boundaries, and coding standards.

---

## 5. Repository Directory Layout

```text
autometa-solutions/
|-- backend/                        # Django backend application
|   |-- manage.py                   # Django management script
|   |-- autometa/                   # Project configuration root
|   |   |-- __init__.py
|   |   |-- settings.py             # Multi-tenant settings, database routing
|   |   |-- urls.py                 # Master URL dispatcher
|   |   |-- wsgi.py                 # Production WSGI application
|   |   `-- asgi.py                 # ASGI application (SSE / WebSockets)
|   |-- apps/                       # Modular business domain apps
|   |   |-- core/                   # Tenant middleware, base models, helpers
|   |   |-- authentication/         # RBAC, JWT rotation, MFA, audit logs
|   |   |-- students/               # 360 student lifecycle, JSONB demographics
|   |   |-- academics/              # Multi-board curricula, grades, sections
|   |   |-- attendance/             # Attendance engine, /api/v1/sync/batch
|   |   |-- timetable/              # Master scheduling, sub-3s substitution engine
|   |   |-- finance/                # Double-entry ledger, webhook handlers
|   |   |-- communication/          # Circulars, notification cascade (FCM/WA/SMS)
|   |   `-- api/                    # Centralized DRF endpoints & serializers
|   |-- requirements.txt            # Python dependencies (whitelisted packages)
|   `-- Dockerfile                  # OCI-compliant container build
|-- frontend/                       # HTML5, Tailwind & JavaScript PWA
|   |-- static/
|   |   |-- js/
|   |   |   |-- sw.js               # Service Worker (CacheStorage & Sync)
|   |   |   |-- indexeddb-store.js  # Client offline mutation queue
|   |   |   `-- app.js              # DOM controllers & API connectors
|   |   |-- css/
|   |   |   `-- tailwind.css        # Compiled production Tailwind styles
|   |   `-- manifest.json           # Progressive Web App manifest
|   |-- templates/                  # Server-rendered semantic HTML5 templates
|   |   |-- layouts/
|   |   |   |-- base.html           # Universal base layout with PWA shell
|   |   |   |-- header.html         # Sticky header with tenant switcher
|   |   |   `-- sidebar.html        # Collapsible navigation drawer
|   |   `-- portals/
|   |       |-- admin/              # High-density management views
|   |       |-- teacher/            # Touch-friendly PWA roll-call & marks
|   |       |-- parent/             # Multi-child dashboard & fee checkout
|   |       `-- student/            # Timetables, notices & published grades
|   |-- package.json                # Tailwind CSS build scripts
|   `-- tailwind.config.js          # Design system token configuration
|-- docker-compose.yml              # PostgreSQL 15, Redis 7, PgBouncer, Web services
|-- prd.md                          # Product Requirements Document
|-- architecture.md                 # System Architecture & Flows
|-- design.md                       # Design System & Tokens
|-- phase.md                        # Project Phasing & 32-Week WBS
|-- rules.md                        # Engineering Rules & AI Boundaries
`-- README.md                       # Project Entrypoint (this document)
```

---

## 6. Local Development Quickstart

### 6.1 Prerequisites
* **Python**: `3.13+`
* **Node.js**: `20+` (for compiling Tailwind CSS)
* **Docker & Docker Compose**: `24+` (for PostgreSQL 15 and Redis 7)

---

### 6.2 Environment Setup

1. **Prepare environment variables**:
   ```bash
   cp .env.example .env
   ```

2. **Configure your `.env` parameters**:
   ```ini
   DEBUG=True
   SECRET_KEY=your-secure-development-secret-key
   DATABASE_URL=postgres://postgres:postgres@localhost:5432/autometa_erp
   REDIS_URL=redis://localhost:6379/0
   DEFAULT_TENANT_SCHEMA=public
   ```

---

### 6.3 Start Database & Cache via Docker Compose

```bash
# Launch PostgreSQL 15 and Redis 7 containers in the background
docker compose up -d postgres redis
```

---

### 6.4 Initialize the Backend (Django)

```bash
# Navigate to backend directory
cd backend

# Create and activate Python virtual environment
python -m venv venv
# On Windows:
venv\Scripts\activate
# On Linux/macOS:
source venv/bin/activate

# Install whitelisted backend dependencies
pip install -r requirements.txt

# Run initial migrations on public schema
python manage.py migrate

# Provision a development test tenant schema
python manage.py provision_tenant --code="demo" --name="Demo International Academy"

# Start the Django development server
python manage.py runserver 127.0.0.1:8000
```

---

### 6.5 Compile Tailwind CSS & Assets

```bash
# Navigate to frontend directory
cd ../frontend

# Install Tailwind dependencies
npm install

# Compile Tailwind styles in watch mode for development
npm run build:css:watch
```

---

## 7. Automated Testing & Verification

Run the test suite to verify schema isolation, DPDP compliance, and financial immutability:

```bash
cd backend

# Run all automated test suites
python manage.py test

# Verify multi-tenant schema isolation specifically
python manage.py test apps.core.tests.test_tenancy_isolation

# Verify double-entry ledger immutability & trigger locks
python manage.py test apps.finance.tests.test_ledger_immutability
```

---

## 8. Regulatory Compliance Standards

* **Digital Personal Data Protection (DPDP) Act, 2023 (India)**:
  * Section 6(1): Specific, informed, unconditional consent capture.
  * Section 9(1): Verifiable Parental Consent (VPC) with guardian mobile OTP verification prior to minor profile activation.
  * Section 9(2) & 9(3): Absolute ban on behavioral profiling, targeted advertising, and social comparison leaderboards.
  * Section 12(1): Automated pseudonymization / right-to-erasure pipeline on student exit.
* **ISO/IEC 27001**:
  * Role-Based Access Control (RBAC) evaluated at the API gateway layer.
  * Short-lived asymmetric JWTs (15 min) paired with 7-day single-use rotating refresh tokens in `HttpOnly`, `SameSite=Strict` cookies.
  * AES-256 encryption at rest and TLS 1.3 enforced for data in transit.

---

## 9. License & Governance

This software is developed and maintained for the **Autometa Solutions** ecosystem. All rights reserved. Proprietary and confidential institutional software.
