# Product Requirements Document (PRD)
## Autometa School Enterprise Resource Planning (ERP) Platform
**Document Version:** 1.0 (Baseline V3 Alignment)  
**Status:** Approved for Implementation  
**Backend:** Python 3.13 / Django 5.x / Django REST Framework (DRF)  
**Database:** PostgreSQL 15+ (Schema-Based Multi-Tenancy)  
**Frontend:** HTML5, Tailwind CSS, Modern JavaScript (PWA Service Worker + IndexedDB)  
**Compliance Standards:** Digital Personal Data Protection (DPDP) Act 2023 (India), ISO/IEC 27001, WCAG 2.1 AA  

---

## 1. Executive Summary & Vision

### 1.1 Product Vision
The **Autometa School ERP Platform** is a cloud-native, multi-tenant Software-as-a-Service (SaaS) operating system engineered to consolidate academic governance, field operations, financial management, multi-board curriculum design, and stakeholder communication into a single unified platform. 

It solves institutional pain points around fragmented spreadsheets, unverified attendance records, manual timetable scheduling, audit vulnerabilities in fee collections, and data privacy risks when processing minor information.

### 1.2 Core Value Propositions
* **Schema-Isolated Multi-Tenancy**: Guaranteed cryptographic and database-level isolation (`tenant_<school_id>`) preventing cross-campus data leaks.
* **Offline-First Field Operations**: Progressive Web App (PWA) architecture allowing teachers to execute roll-call attendance and marks entry without internet, syncing seamlessly via IndexedDB upon reconnection.
* **Double-Entry Immutable Financial Accounting**: Strict audit-grade bookkeeping forbidding direct SQL record updates or deletions, enforcing compensating journal transactions and automated gateway reconciliation.
* **DPDP Act 2023 Native Compliance**: Built-in Verifiable Parental Consent (VPC) workflows, right-to-erasure lifecycle pipelines, and zero behavioral tracking or advertising on minors.
* **Intelligent Timetable & Substitution Engine**: Period collision validation and automated sub-3-second teacher leave substitution calculation.

---

## 2. Technical Stack & Architectural Baseline

```
┌────────────────────────────────────────────────────────────────────────┐
│                        FRONTEND CLIENT PORTALS                         │
│  • HTML5 Semantic App Shells + Tailwind CSS UI Design System          │
│  • Modular JavaScript (ES6+) for Dynamic DOM & API Orchestration       │
│  • PWA Service Worker (Cache Storage, Push API, Background Sync)       │
│  • Client-Side IndexedDB Engine (Offline Queue: max 5,000 mutations)  │
└───────────────────────────────────┬────────────────────────────────────┘
                                    │ HTTPS (TLS 1.3) / SSE / REST JSON
┌───────────────────────────────────▼────────────────────────────────────┐
│                    DJANGO BACKEND ARCHITECTURE                         │
│  • Django 5.x + Django REST Framework (DRF)                            │
│  • Tenant Middleware: search_path = tenant_<school_id>, public         │
│  • Celery 5.x Worker Nodes + Redis 7+ Broker (Async Tasks & Exports)   │
│  • RBAC & Token Governance (RS256 JWT, 15m access / 7d refresh)        │
│  • Ledger Engine (ACID atomic transactions, row-level locks)           │
│  • DPDP Statutory & Cryptographic Audit Interceptor                    │
└───────────────────────────────────┬────────────────────────────────────┘
                                    │ Psycopg3 / PgBouncer Connection Pool
┌───────────────────────────────────▼────────────────────────────────────┐
│                       POSTGRESQL 15+ DATA LAYER                        │
│  ┌────────────────────────┐  ┌───────────────────────────────────────┐ │
│  │     public Schema      │  │      tenant_<school_id> Schemas       │ │
│  │ • Tenant / Campus Reg  │  │ • Students, Guardians, Staff          │ │
│  │ • Domain Routing Table │  │ • Attendance Records, Time Slots      │ │
│  │ • Global Feature Flags │  │ • Immutable Fee Ledgers & Invoices    │ │
│  │ • System Audit Archive │  │ • Exams, Marks, Timetable Matrices    │ │
│  └────────────────────────┘  └───────────────────────────────────────┘ │
└───────────────────────────────────┬────────────────────────────────────┘
                                    │ S3 REST API (Pre-Signed URLs)
┌───────────────────────────────────▼────────────────────────────────────┐
│                    OBJECT STORAGE & EXTERNAL APIS                      │
│  • S3-Compatible Storage (AWS S3 / Cloudflare R2 / MinIO)              │
│  • Payment Gateways (Razorpay / Cashfree HMAC Webhooks)                │
│  • Notification Engines (Firebase FCM, WhatsApp Cloud API, DLT SMS)   │
└────────────────────────────────────────────────────────────────────────┘
```

### 2.1 Backend Specifications (Django)
* **Framework**: Django 5.x on Python 3.13 LTS.
* **API Engine**: Django REST Framework (DRF) with standard serializer validation and throttling.
* **Multi-Tenancy Engine**: PostgreSQL schema isolation via custom request-scoped middleware setting `SET search_path TO tenant_<school_id>, public` on connection checkout.
* **Background Processing**: Celery with Redis for asynchronous tasks (PDF report cards, monthly payroll runs, notification broadcast cascades).
* **Caching & Real-Time**: Redis 7+ for OTP caching, session blacklists, and Server-Sent Events (SSE) KPI metrics broadcasting.

### 2.2 Frontend Specifications (HTML + Tailwind + JS PWA)
* **Markup & Templates**: Semantic HTML5 rendered via Django templates for initial fast SSR paint, combined with modular JS components.
* **Styling**: Tailwind CSS configured with a unified institutional design token system (supporting responsive viewports: 360px mobile, 768px tablet, 1280px+ desktop).
* **Client Logic**: Vanilla ES6+ JavaScript modules. No heavy frontend framework runtime overhead, maximizing performance on low-end 2 GB RAM Android hardware.
* **PWA Offline Infrastructure**:
  * Service Worker (`sw.js`) intercepting fetch requests, caching App Shell assets.
  * Browser IndexedDB (`idb`) storing offline attendance rosters and mark sheets.
  * Sync Queue Engine executing automated background synchronization upon network reconnection.

### 2.3 Database & Tenancy Architecture (PostgreSQL)
* **Database Engine**: PostgreSQL 15+ with PgBouncer connection multiplexing (transaction-mode pooling, 20:1 ratio).
* **Multi-Tenant Separation**:
  * `public` schema: Contains `tenants_tenant`, `tenants_domain`, and global system configuration.
  * `tenant_<school_id>` schemas: Dynamic per-school schema containing all operational tables.
  * Strict prohibition of cross-tenant joins; all tenant queries resolve strictly inside their active `search_path`.

---

## 3. User Classes & Stakeholder Portals

| User Persona | Context | Key Responsibilities & Access Scope |
| :--- | :--- | :--- |
| **System Super Admin** | Desktop Web | Multi-tenant onboarding, campus node provisioning, global audit log inspection, feature flagging. |
| **Campus Principal / Administrator** | Desktop Web | School-wide operations, fee concession approvals, academic calendar freeze, audit dashboard reviews. |
| **Academic Coordinator** | Desktop Web | Board curriculum configuration, master timetable design, teacher allocation, substitution management. |
| **School Accountant / Fee Clerk** | Desktop Web | Dynamic fee structure configuration, double-entry ledger oversight, offline cash receipts, bank reconciliation. |
| **Teacher / Academic Staff** | Mobile PWA & Desktop | Roll-call attendance marking (online/offline), grade and marks logging, personal schedule and substitution alerts. |
| **Parent / Guardian** | Mobile PWA & Web | Multi-child context switching, real-time ward attendance monitoring, online fee payments, report card downloads, DPDP consent management. |
| **Student** | Mobile PWA & Web | Read-only timetables, published exam results, homework feed, institutional circulars, library issuance tracking. |

---

## 4. Functional Requirements Matrix (FR-01 through FR-21)

| Requirement ID | Module Name | Scope Status | Priority | Primary Users | Core Deliverable Summary |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **FR-01** | Dashboard & Analytics | In MVP | High | All Roles | Role-based real-time KPI cards, SSE live metrics stream, Redis caching layer. |
| **FR-02** | Admission & Enquiry | In MVP | High | Admin, Parents | Lead capture pipeline, prospect CRM, OTP verification, DPDP verifiable consent. |
| **FR-03** | Student Management | In MVP | Critical | Admin, Teachers | 360° student lifecycle profile, JSONB demographic data, medical records, section promotion. |
| **FR-04** | Staff / HR Management | In MVP | High | HR, Admin, Staff | Employee records, credential vault, biometric attendance ingestion, automated monthly payroll. |
| **FR-05** | Academic Management | In MVP | Critical | Coordinators, Admin | Multi-board structure (CBSE, ICSE, IB, State), terms, subjects, and section mapping. |
| **FR-06** | Timetable Management | In MVP | Critical | Coordinators, Staff | Zero-collision master grid, room/teacher constraints, sub-3s teacher substitution solver. |
| **FR-07** | Attendance Management | In MVP | Critical | Teachers, Parents | Offline-first PWA attendance capture, IndexedDB queue, batch sync, instant SMS alerts. |
| **FR-08** | Homework & Assignments | Phase 2 | Medium | Teachers, Students | LMS-lite digital submission repository, ClamAV virus scanning, deadline locking. |
| **FR-09** | Examination & Results | In MVP | High | Exam Officers, Staff | Multi-board grading scales (CCE/GPA), offline marks entry, audit-locked gradebooks, batch PDF cards. |
| **FR-10** | Fee Management | In MVP | Critical | Accountant, Parents | Double-entry ledger, dynamic fee heads, concessions, Razorpay/Cashfree webhook reconciliation. |
| **FR-11** | Parent Portal | In MVP | High | Parents | Multi-child switcher, attendance timeline, fee payment gateway checkout, DPDP privacy controls. |
| **FR-12** | Student Portal | In MVP | Medium | Students | Read-mostly view for class schedule, issued books, homework deadlines, and published grades. |
| **FR-13** | Teacher Portal | In MVP | Critical | Teachers | Touch-friendly PWA workspace for roll-call, marks entry, timetable, and substitution notices. |
| **FR-14** | Library Management | Extended | Low | Librarian, Students | ISBN cataloging, physical barcode/QR camera scanning, circulation logs, overdue fines. |
| **FR-15** | Transport Management | Extended | Medium | Transport Head, Parents | Fleet tracking, bus routes, stop optimization, GPS MQTT telemetry ingestion, live map. |
| **FR-16** | Communication (Notices) | In MVP | High | Admin, All Users | WYSIWYG circular publisher, audience targeting filters, approval workflow, emergency push. |
| **FR-17** | Reports & Analytics | In MVP | High | Management, Auditors | Asynchronous Celery export engine (CSV, Excel, formatted PDF) running on read replicas. |
| **FR-18** | Document Management | Extended | High | Admin, Registrar | Transfer Certificates (TC), Bonafide certificates with cryptographic HMAC QR verification. |
| **FR-19** | User & Role Management | In MVP | Critical | Super Admin, IT | Granular RBAC, JWT session rotation (RS256), MFA (TOTP/SMS), Redis session blacklisting. |
| **FR-20** | Notification Engine | In MVP | High | System-wide | Unified dispatch cascade: FCM Push → WhatsApp Business API → DLT SMS fallback. |
| **FR-21** | Setting & Administration | In MVP | Critical | Super Admin, Admin | Multi-tenant provisioning wizard, academic session rollover, immutable audit trails. |

---

## 5. Granular Module Specifications

### 5.1 FR-01: Real-Time Dashboard & KPI Engine
* **Purpose**: Provide role-specific, latency-optimized visual analytics across operational domains.
* **Backend Implementation (Django)**:
  * Endpoint: `GET /api/v1/dashboard/summary` (Cached with Redis TTL of 60 seconds).
  * Endpoint: `GET /api/v1/dashboard/stream` (Django ASGI / SSE emitting live attendance and collection pings).
  * Database Model: `DashboardWidgetConfig` storing per-role layout JSON.
* **Frontend Implementation (HTML/Tailwind/JS)**:
  * Dynamic CSS Grid layout rendering role-specific metrics: Daily Attendance %, Daily Fee Collections, Pending Concessions, Teacher Absences.
  * Native `EventSource` connection for real-time SSE stream with automatic fallback to 30-second polling.

### 5.2 FR-02: Admission & Enquiry CRM
* **Purpose**: Track prospective applicants from initial inquiry to enrollment with strict DPDP compliance.
* **Backend Implementation (Django)**:
  * Endpoint: `POST /api/v1/enquiries/public/submit` (Public endpoint with rate limiting: max 5 requests/IP/hour).
  * Endpoint: `POST /api/v1/enquiries/verify-otp` (Validates 6-digit OTP stored in Redis with 5-minute expiry).
  * Endpoint: `PATCH /api/v1/enquiries/<uuid:id>/stage` (State machine: `NEW` → `CONTACTED` → `ASSESSMENT` → `OFFERED` → `ENROLLED` → `REJECTED`).
  * Cryptographic Parental Consent: Captures guardian IP, timestamp, and SHA-256 hash of agreed terms.

### 5.3 FR-03: 360° Student Lifecycle Management
* **Purpose**: Comprehensive student profiles maintaining unbroken historical records across academic years.
* **Backend Implementation (Django)**:
  * PostgreSQL JSONB column for demographic attributes (blood group, nationality, languages, special needs).
  * Endpoints: `GET/POST /api/v1/students`, `GET /api/v1/students/<uuid:id>/profile`, `PUT /api/v1/students/<uuid:id>/promotion`.
  * Soft-delete flag (`is_active = False`) to prevent destructive data loss; statutory anonymization routine for DPDP erasure requests.

### 5.4 FR-04: Staff / HR & Automated Payroll
* **Purpose**: Staff lifecycle, credentials, biometric timeclock integration, and salary calculation.
* **Backend Implementation (Django)**:
  * Endpoints: `POST /api/v1/hr/biometric/ingest` (Webhook receiver verifying device HMAC signature), `POST /api/v1/hr/payroll/run`, `GET /api/v1/hr/staff/<uuid:id>/payslip`.
  * Celery monthly background task executing formula-based net salary calculations: `Base + Allowances - Deductions (Tax/PF/ESI) - Unpaid Leave Days`.

### 5.5 FR-05: Multi-Board Academic Management
* **Purpose**: Flexible configuration of curriculum structures (CBSE, ICSE, IB, State Boards).
* **Backend Implementation (Django)**:
  * Hierarchical modeling: `Board` → `AcademicYear` → `Grade` → `Section` → `Subject` → `SubjectAssignment`.
  * Enforces zero orphan sections; validates academic year end dates strictly succeed start dates.

### 5.6 FR-06: Timetable & Substitution Engine
* **Purpose**: Master timetable generation, period collision prevention, and real-time teacher absence substitution.
* **Algorithm Requirements**:
  * Evaluates hard constraints: No teacher double-booking, no classroom double-booking.
  * Evaluates soft constraints: Workload balancing, consecutive period limits.
  * Substitution Engine: When teacher leave is logged, the engine filters available teachers with matching subject qualifications and ranks by lowest weekly class allocation in under 3 seconds.

### 5.7 FR-07: Offline-First Attendance Management (PWA)
* **Purpose**: Zero-latency roll-call marking inside classrooms with weak or zero Wi-Fi connectivity.
* **Frontend Architecture (HTML/JS/IndexedDB)**:
  1. Class roster cached locally in IndexedDB store `rosters`.
  2. Teacher toggles roll-call states: `PRESENT`, `ABSENT`, `LATE`, `EXCUSED`.
  3. Writes mutation object to `pending_mutations` store with local UUID and UTC timestamp.
  4. Service Worker listens for browser `online` event and flushes mutations via `POST /api/v1/attendance/batch-sync`.
* **Backend Conflict Resolution**:
  * Verifies if attendance session is marked `server_locked = TRUE`.
  * If unlocked: commits using server-authoritative timestamp and returns UUID sync receipts.
  * If locked: rejects delayed write and records entry in administrative review log.
  * Triggers automated absentee SMS/WhatsApp notification cascade to parents within 15 minutes.

### 5.8 FR-09: Examination & Result Management
* **Purpose**: Assessment scheduling, offline-capable marks entry, grading scale evaluation, and locked gradebooks.
* **Grading Scales**: Configurable strategy adapters supporting CBSE CCE, numerical percentage marks, and IB criterion scales (1–7).
* **Audit Locks**: Once principal marks exam as locked, database trigger blocks further modifications unless an admin override is authorized.
* **Report Card Engine**: Asynchronous Celery worker compiling standardized A4 PDF report cards via headless Chrome or WeasyPrint.

### 5.9 FR-10: Double-Entry Immutable Fee Management
* **Purpose**: Financial governance, tuition invoicing, payment gateway reconciliation, and immutable audit ledgers.
* **Core Accounting Rules**:
  * **Zero Updates/Deletes**: `fee_ledger_entries` table forbids direct SQL `UPDATE` or `DELETE`.
  * **Compensating Entries Only**: Any reversal, scholarship, or waiver creates a linked offsetting entry with explicit administrative remarks.
  * **Idempotent Webhooks**: Razorpay/Cashfree webhook receiver verifies HMAC SHA-256 signature; enforces database unique constraints on `payment_gateway_ref` to prevent double-crediting.
  * **Row-Level Locking**: Webhook processing wraps ledger writes in `select_for_update()` inside `transaction.atomic()`.

### 5.10 FR-11 & FR-13: Portals (Parent & Teacher)
* **Parent Portal**:
  * Multi-child switcher: Parent JWT carries claims for all linked students; portal toggles active ward context without re-authentication.
  * Displays attendance percentages, pending fee invoices, online payment checkout modal, and term report cards.
* **Teacher Portal**:
  * Touch-optimized PWA layout: Fast 1-tap roll-call toggle buttons, offline indicators, timetable view, and daily substitution popups.

### 5.11 FR-19: User & Role-Based Access Control (RBAC)
* **Authentication**: Short-lived JWT (15-minute access) with RS256/Ed25519 signature + 7-day refresh token in `HttpOnly`, `SameSite=Strict` cookie.
* **RBAC Enforcement**: Django custom authorization permission classes evaluated at the view/endpoint layer. UI element hiding alone is never relied upon for security.
* **MFA**: Time-based One-Time Password (TOTP) enforcement for Administrator and Accountant roles.

### 5.12 FR-20: Unified Notification Pipeline
* **Cascade Dispatch Algorithm**:
  1. **Tier 1 (Instant / Free)**: Firebase Cloud Messaging (FCM) Push to PWA / Mobile app shell.
  2. **Tier 2 (High Priority)**: If unread after 10 minutes or designated urgent, dispatch via WhatsApp Business API.
  3. **Tier 3 (Regulatory Fallback)**: If WhatsApp delivery fails, dispatch via DLT-registered transactional SMS gateway.
  4. Real-time per-tenant monthly notification cost guardrails and budget caps.

---

## 6. Core Database Schema & Data Models

All models below live inside the tenant-isolated schema (`tenant_<school_id>.*`).

```sql
-- 1. Students Table
CREATE TABLE tenant_schema.students (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    admission_no VARCHAR(64) NOT NULL UNIQUE,
    first_name VARCHAR(128) NOT NULL,
    last_name VARCHAR(128) NOT NULL,
    dob DATE NOT NULL,
    gender VARCHAR(16) NOT NULL,
    board_registration_no VARCHAR(128),
    current_grade_id UUID NOT NULL,
    current_section_id UUID NOT NULL,
    vpc_consent_token VARCHAR(256) NOT NULL,
    vpc_timestamp TIMESTAMPTZ NOT NULL,
    status VARCHAR(32) DEFAULT 'ACTIVE' CHECK (status IN ('ENROLLED', 'ACTIVE', 'ALUMNI', 'WITHDRAWN')),
    profile_data JSONB DEFAULT '{}'::jsonb,
    is_active BOOLEAN DEFAULT TRUE,
    created_at TIMESTAMPTZ DEFAULT clock_timestamp(),
    updated_at TIMESTAMPTZ DEFAULT clock_timestamp()
);

-- 2. Attendance Records Table (Offline-reconciliation ready)
CREATE TABLE tenant_schema.attendance_records (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    student_id UUID NOT NULL REFERENCES tenant_schema.students(id),
    timetable_slot_id UUID,
    attendance_date DATE NOT NULL,
    status VARCHAR(16) NOT NULL CHECK (status IN ('PRESENT', 'ABSENT', 'LATE', 'EXCUSED')),
    recorded_by_staff_id UUID NOT NULL,
    offline_sync_uuid UUID UNIQUE,
    client_recorded_at TIMESTAMPTZ NOT NULL,
    server_synced_at TIMESTAMPTZ DEFAULT clock_timestamp(),
    is_session_locked BOOLEAN DEFAULT FALSE,
    CONSTRAINT uq_student_slot_date UNIQUE (student_id, timetable_slot_id, attendance_date)
);

-- 3. Fee Ledger Entries (Immutable Double-Entry Ledger)
CREATE TABLE tenant_schema.fee_ledger_entries (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    transaction_group_id UUID NOT NULL,
    student_id UUID NOT NULL REFERENCES tenant_schema.students(id),
    account_code VARCHAR(32) NOT NULL,
    entry_type VARCHAR(8) NOT NULL CHECK (entry_type IN ('DEBIT', 'CREDIT')),
    amount NUMERIC(12, 2) NOT NULL CHECK (amount > 0),
    description TEXT NOT NULL,
    reversal_reference_id UUID REFERENCES tenant_schema.fee_ledger_entries(id),
    idempotency_key VARCHAR(128) UNIQUE NOT NULL,
    payment_gateway_ref VARCHAR(128),
    created_by_user_id UUID NOT NULL,
    created_at TIMESTAMPTZ DEFAULT clock_timestamp()
);

-- Prevent UPDATE or DELETE on Fee Ledger
CREATE OR REPLACE FUNCTION tenant_schema.prevent_fee_ledger_tampering()
RETURNS TRIGGER AS $$
BEGIN
    RAISE EXCEPTION 'Destructive UPDATE or DELETE is strictly forbidden on immutable fee_ledger_entries. Use compensating credit/debit records.';
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER trg_protect_fee_ledger
BEFORE UPDATE OR DELETE ON tenant_schema.fee_ledger_entries
FOR EACH ROW EXECUTE FUNCTION tenant_schema.prevent_fee_ledger_tampering();
```

---

## 7. Non-Functional Requirements (NFRs)

### 7.1 Quantitative Latency Benchmarks
* **Authentication & JWT Verification**: P95 ≤ 400ms | P99 ≤ 800ms
* **Standard CRUD Reads (Cached Profiles, Timetables)**: P95 ≤ 250ms | P99 ≤ 500ms
* **Standard CRUD Writes (Roll-call commit, Ledger line)**: P95 ≤ 350ms | P99 ≤ 750ms
* **Client-Side PWA Local Operations (IndexedDB execution)**: P95 ≤ 50ms | P99 ≤ 100ms
* **Complex Ledger & Longitudinal Search Queries**: P95 ≤ 600ms | P99 ≤ 1200ms
* **Live GPS Telematics WebSocket Render**: P95 ≤ 200ms | P99 ≤ 400ms

### 7.2 Scalability & Concurrency Targets
* **Baseline Active Users**: 15,000 active concurrent user sessions across campus instances.
* **Peak Surge Traffic**: 50,000 concurrent sessions during fee cutoffs and result publications.
* **Sustained API Throughput**: 3,500 Requests Per Second (RPS) sustained; 7,000 RPS burst capacity for up to 10 minutes.
* **Batch Document Processing**: 10,000 PDF report cards generated in < 15 minutes.
* **Notification Queue Dispatch**: 100,000 push/SMS alerts queued in < 5 minutes.

### 7.3 Frontend Core Web Vitals (PWA Shell)
* **Largest Contentful Paint (LCP)**: ≤ 2.0s on desktop; ≤ 2.5s on mobile 4G.
* **Interaction to Next Paint (INP)**: ≤ 150ms across all interactions.
* **Cumulative Layout Shift (CLS)**: ≤ 0.05 across portal landing views.
* **Memory & Storage Budget**: Active PWA RAM usage ≤ 150 MB; local storage footprint ≤ 60 MB.

### 7.4 High Availability & Disaster Recovery
* **Cluster Uptime SLA**: 99.9% monthly availability (excluding scheduled off-peak maintenance).
* **Recovery Point Objective (RPO)**: ≤ 5 minutes via continuous PostgreSQL Write-Ahead Logging (WAL) archiving.
* **Recovery Time Objective (RTO)**: ≤ 30 minutes for cross-AZ standby failover.

---

## 8. Failure Modes and Effects Analysis (FMEA)

| Failure Mode | Root Cause | Severity (1-10) | Occurrence (1-10) | Detection (1-10) | RPN | Engineered Failsafe & Mitigation |
| :--- | :--- | :---: | :---: | :---: | :---: | :--- |
| **Cross-Tenant Data Leak** | Missing or misconfigured tenant search path on connection pool | 10 | 1 | 2 | **20** | Connection-pool level Django middleware enforcing `SET search_path TO tenant_<id>`. Automated regression test suite rejecting cross-tenant leakage. |
| **Offline PWA Data Corruption** | Client browser crash or irregular IndexedDB quota purge | 6 | 3 | 3 | **54** | Fallback to session memory storage; immediate sync banner upon recovery; structured snapshot backups in localStorage. |
| **Duplicate Fee Payment** | Network timeout during parent payment callback | 8 | 3 | 2 | **48** | Database-level unique constraint on `idempotency_key` and `payment_gateway_ref`. Idempotent webhook listener. |
| **Unresolvable Timetable Loop** | Over-constrained teacher subject/period limits | 5 | 4 | 2 | **40** | Heuristic relaxation fallback: algorithm flags soft-constraint trade-offs to the coordinator when hard limits conflict. |
| **Outbound SMS Blacklisting** | Spike in broadcast messages exceeding gateway throughput | 7 | 2 | 2 | **28** | Dynamic token-bucket rate limiter; automated priority cascade routing (In-App → FCM → WhatsApp → DLT SMS). |

---

## 9. Implementation Roadmap & WBS (32-Week Baseline)

```
2026                 2027
Sep   Oct   Nov   Dec   Jan   Feb   Mar   Apr
Phase 0 [██]
Phase 1     [████]
Phase 2           [████]
Phase 3                 [███]
Phase 4                     [██]
Phase 5                         [██]
        ▲     ▲     ▲     ▲     ▲     ▲
      Gate0 Gate1 Gate2 Gate3 Beta  Launch
```

### Phase 0: Inception, Architecture & Multi-Tenancy (Weeks 01–04)
* Deliverables:
  * Django project initialization with custom PostgreSQL schema routing middleware.
  * Tailwind CSS design tokens and base HTML5 PWA Shell architecture (`sw.js`, `manifest.json`).
  * DPDP Verifiable Parental Consent (VPC) cryptographic design sign-off.
* **Gate 0 Sign-Off**: Multi-tenant schema routing verified; zero cross-tenant leakage in test harness.

### Phase 1: Core Academic & Administrative Backbone (Weeks 05–12)
* Deliverables:
  * FR-03 (Student 360), FR-04 (Staff/HR & Payroll computation), FR-05 (Multi-Board Academics).
  * FR-19 (Granular RBAC, JWT session rotation, MFA), FR-21 (Tenant provisioning).
* **Gate 1 Sign-Off**: Student enrollment, staff payroll calculation, and board configuration live in staging.

### Phase 2: Operational Engines & Offline PWA (Weeks 13–20)
* Deliverables:
  * FR-06 (Master timetable scheduler and sub-3-second substitution engine).
  * FR-07 (Offline-first PWA attendance with IndexedDB queue and batch sync).
  * FR-09 (Examination scheduling, offline marks entry, locked gradebooks, PDF batch generator).
* **Gate 2 Sign-Off**: Offline attendance sync stress-tested under simulated 3G network dropouts.

### Phase 3: Financial Accounting & Portals Ecosystem (Weeks 21–26)
* Deliverables:
  * FR-10 (Immutable double-entry fee ledger, payment gateway webhooks for Razorpay/Cashfree).
  * FR-11 (Parent Portal with multi-child context switcher), FR-13 (Teacher PWA portal), FR-12 (Student Portal).
* **Gate 3 Sign-Off**: Financial ledger debit/credit balancing verified; zero tampering allowed on locked records.

### Phase 4: Unified Communications & Extended Modules (Weeks 25–28)
* Deliverables:
  * FR-20 (Unified notification queue: FCM → WhatsApp Cloud API → DLT SMS cascade).
  * FR-16 (Circulars & emergency broadcast engine), FR-17 (Async Celery report exporter).
  * Extended operational interfaces: FR-14 (Library barcode scanner), FR-15 (Fleet telematics), FR-18 (QR certificate generator).

### Phase 5: Hardening, Statutory Compliance Audit & Production Launch (Weeks 29–32)
* Deliverables:
  * Formal DPDP Act 2023 compliance verification (right to erasure, consent token hashing).
  * Independent VAPT security audit for ISO 27001 readiness.
  * 50,000 concurrent user load testing and pilot campus onboarding.
* **Gate 4 Sign-Off**: Production Go-Live.

---

## 10. Traceability & Acceptance Verification Criteria

| Req ID | Acceptance Test Definition (BDD / Criteria) | Target Verification Metric |
| :--- | :--- | :--- |
| **FR-01** | Load role dashboard under 15k simulated users | Dashboard initial render ≤ 400ms; KPI data matches transactional DB. |
| **FR-02** | Submit inquiry, issue OTP, capture consent token | 6-digit OTP expires in 5 mins; duplicate leads rejected by phone/email. |
| **FR-06** | Run substitution solver on teacher morning leave | Qualified substitute suggested in < 3 seconds; zero room/teacher overlap. |
| **FR-07** | Mark 50 student attendance records offline and reconnect | Mutations sync within 1.5s on 3G; absentee SMS queued within 15 minutes. |
| **FR-09** | Batch generate 1,000 student PDF report cards | Asynchronous batch job finishes in < 3 minutes on Celery worker. |
| **FR-10** | Concurrent payment webhook simulation | Ledgers strictly balance (`Debit == Credit`); idempotent replay rejected. |
| **FR-11** | Cross-child IDOR security test on parent portal | Attempting to access non-authorized child ID returns HTTP 403 Forbidden. |
| **FR-19** | Gateway route permission evaluation | Unauthorized API call fails with HTTP 403; expired JWT rejected immediately. |
| **FR-20** | Simulate primary FCM push failure | Dispatch engine automatically cascades to WhatsApp, then DLT SMS in < 30s. |
| **FR-21** | Execute annual academic session rollover | Prior session historical records locked immutably; audit event logged. |
