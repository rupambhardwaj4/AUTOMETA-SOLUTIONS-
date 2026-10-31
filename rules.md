# Engineering Rules, Library Governance & AI Boundaries
## Autometa School Enterprise Resource Planning (ERP) Platform
**Document Version:** 1.0 (Baseline V3 Alignment)  
**Document Status:** Enforced Development Standard  
**Target Environment:** Python 3.13 / Django 5.x / PostgreSQL 15+ / HTML5 + Tailwind CSS + Vanilla JS (PWA)  

---

## 1. Approved vs Prohibited Technology Stack & Libraries

To preserve performance on entry-level client hardware (2 GB RAM baseline), guarantee regulatory compliance (DPDP Act 2023), and maintain zero-leak multi-tenant database isolation, all engineering workflows must strictly comply with the following library whitelist and blacklist.

### 1.1 Backend Library Whitelist vs Prohibitions (Django / Python)

| Category | Approved & Required (USE) | Strictly Prohibited (AVOID) | Architectural Justification |
| :--- | :--- | :--- | :--- |
| **Framework & API** | `django` (5.x), `djangorestframework`, `django-cors-headers` | Flask, FastAPI, Tornado, monolithic ad-hoc microframeworks | Django provides cohesive ORM, session governance, and built-in CSRF/security controls required across all portals. |
| **Database Driver** | `psycopg` (v3) or `psycopg2-binary` | Async raw drivers bypassing Django connection pool, pure SQLite in prod | PostgreSQL 15+ schema switching depends on deterministic `search_path` connection hooks. |
| **Tenancy Isolation** | Custom request middleware setting `search_path` or `django-tenants` | Shared multi-tenant tables relying solely on `tenant_id` WHERE clauses | Foreign key leaks and developer omission of `tenant_id` filters violate SRS Section 2.5.2. Schema isolation is mandatory. |
| **Caching & Broker** | `redis` (v5+), `django-redis` | In-memory Python global dicts, Memcached, SQLite cache | Redis provides atomic primitive locking (`SETNX`), pub/sub for SSE metrics, and Celery task coordination. |
| **Task Queue** | `celery` (v5.x) | Unmanaged background threads (`threading.Thread`), OS-level cron scripts | Long-running jobs (PDF compilation, batch payroll) must run in isolated, retry-aware worker nodes. |
| **Auth & Crypto** | `pyjwt` (with cryptography RS256/Ed25519), `argon2-cffi` | Legacy `MD5`, `SHA1`, unsalted `bcrypt`, plain Symmetric HS256 for public tokens | ISO 27001 requires asymmetric signatures and modern password hashing standards. |
| **Document Delivery** | `weasyprint`, `reportlab`, or Headless Chrome (`playwright`) | Proprietary office binaries (Word, LibreOffice automation), uncompiled HTML exports | Reports and certificates must be standardized PDF/A compliant for statutory archival. |

### 1.2 Frontend & Client Whitelist vs Prohibitions (HTML / CSS / JS)

| Category | Approved & Required (USE) | Strictly Prohibited (AVOID) | Architectural Justification |
| :--- | :--- | :--- | :--- |
| **CSS Architecture** | **Tailwind CSS (v3/v4)** with explicit utility tokens | Bootstrap, Material-UI, heavy Sass preprocessors with large bundle outputs | Tailwind compiles to $< 40\text{ KB}$ gzipped, ensuring fast 3G network load times. |
| **JavaScript Logic** | **Modular Vanilla ES6+** JavaScript (`import`/`export`) | Heavy client frameworks (React SPA, Angular, Vue runtime), jQuery | Mobile devices with 2 GB RAM freeze under heavy virtual DOM runtimes. Vanilla JS maintains $\ge 50\text{ FPS}$. |
| **Offline Storage** | Native **IndexedDB** (or lightweight `idb` wrapper $\le 2\text{ KB}$) | `localStorage` for transactional records, WebSQL (deprecated) | IndexedDB provides asynchronous, non-blocking transactional storage up to 5,000 mutation records. |
| **PWA Subsystem** | Native **Service Workers (`sw.js`)**, CacheStorage API | Deprecated AppCache, external unverified service worker libraries | Service workers enable background sync and app-shell caching for zero-connectivity field operations. |
| **Ad / Tracking Tags** | **ZERO third-party tracking scripts** | Google Analytics, Facebook Pixel, Mixpanel, Hotjar, behavioral session replays | **STRICT DPDP ACT 2023 BAN**: Section 9(3) legally forbids behavioral tracking and targeted advertising on minors. |

---

## 2. Defensive Error Handling & Anti-Pattern Prevention

### 2.1 Multi-Tenant Schema Leakage (How to Avoid & Mitigate)
* **The Danger**: When PgBouncer reuses a database connection, if the previous request set `search_path TO tenant_a`, a subsequent request for `tenant_b` could read `tenant_a` data if the tenant resolution fails.
* **Mandatory Pattern**: Always sanitize and wrap schema switching in a `try...finally` block inside the Django middleware:
  ```python
  # core/middleware.py
  from django.db import connection
  from django.http import Http404

  class TenantRoutingMiddleware:
      def __init__(self, get_response):
          self.get_response = get_response

      def __call__(self, request):
          tenant_schema = self.resolve_tenant_schema(request)
          if not tenant_schema:
              raise Http404("Invalid or inactive tenant domain.")

          cursor = connection.cursor()
          # Explicitly set search_path for this request
          cursor.execute(f"SET search_path TO {tenant_schema}, public;")
          request.tenant_schema = tenant_schema

          try:
              response = self.get_response(request)
          finally:
              # Always reset to public schema when connection returns to pool
              cursor.execute("SET search_path TO public;")
              cursor.close()

          return response
  ```

### 2.2 Financial Ledger Tampering & Race Conditions
* **The Danger**: Concurrent fee payment callbacks crediting an account twice or partial writes corrupting debit/credit equality.
* **Mandatory Pattern**:
  1. Wrap all ledger mutations inside `transaction.atomic()`.
  2. Use `select_for_update()` to lock the student fee profile row.
  3. Validate `Debits == Credits` before committing.
  4. Never execute `UPDATE` or `DELETE` on `fee_ledger_entries`. Use compensating rows only:
  ```python
  # finance/services.py
  from django.db import transaction
  from decimal import Decimal

  @transaction.atomic
  def record_fee_payment(student_id, amount, account_code, idempotency_key, gateway_ref, user_id):
      # 1. Check idempotency
      if FeeLedgerEntry.objects.filter(idempotency_key=idempotency_key).exists():
          return FeeLedgerEntry.objects.get(idempotency_key=idempotency_key)

      # 2. Lock student record
      student = Student.objects.select_for_update().get(id=student_id)
      group_id = uuid.uuid4()

      # 3. Create balancing entries
      # Debit: Cash/Bank Clearing Account
      debit_entry = FeeLedgerEntry.objects.create(
          transaction_group_id=group_id,
          student=student,
          account_code="1010_BANK_CLEARING",
          entry_type="DEBIT",
          amount=amount,
          idempotency_key=f"{idempotency_key}_DEBIT",
          payment_gateway_ref=gateway_ref,
          created_by_user_id=user_id
      )
      # Credit: Student Tuition Receivable
      credit_entry = FeeLedgerEntry.objects.create(
          transaction_group_id=group_id,
          student=student,
          account_code=account_code,
          entry_type="CREDIT",
          amount=amount,
          idempotency_key=f"{idempotency_key}_CREDIT",
          payment_gateway_ref=gateway_ref,
          created_by_user_id=user_id
      )
      return debit_entry
  ```

### 2.3 Offline Sync Conflicts & Session Lock Violations
* **The Danger**: A teacher submits attendance offline after an administrator has already officially frozen or locked the academic session.
* **Mandatory Pattern**:
  * Check `is_session_locked` on the server before committing.
  * If locked: DO NOT overwrite data. Return HTTP 409 Conflict with code `SESSION_LOCKED`, log an audit exception in `audit_logs`, and surface an administrative review task.

### 2.4 Browser IndexedDB Quota Exhaustion & Corrupted State
* **The Danger**: Client mobile storage exceeds quota, causing silent drop of recorded attendance mutations.
* **Mandatory Pattern**:
  * Wrap all IndexedDB transactions in structured `try...catch` handlers.
  * Monitor available storage using `navigator.storage.estimate()`.
  * If storage write fails, fall back immediately to in-memory array and display an urgent high-contrast amber banner warning the teacher not to close the browser tab.

---

## 3. Strict Boundaries for the AI (Safety & Compliance)

When generating code, designing databases, or refactoring components, the AI assistant must operate strictly within the following non-negotiable boundaries:

### 3.1 DPDP Act 2023 Statutory Boundaries
1. **Never Bypass Verifiable Parental Consent (VPC)**:
   * The AI must NEVER generate student onboarding workflows that bypass guardian mobile OTP verification and cryptographic terms consent hashing (`vpc_consent_token`).
   * Minor records (< 18 years) cannot be activated in the database without an immutable parent link.
2. **Absolute Ban on Profiling & Behavioral Algorithms**:
   * The AI must NEVER generate recommendation engines, psychological profiling algorithms, public student leaderboards, or grade-shaming UI components.
3. **Right to Erasure Compliance**:
   * The AI must always implement soft-deletion with automated pseudonymization routines that scrub identifiable PII (names, phone numbers, Aadhaar hashes) while preserving board transcripts and immutable accounting ledgers.

### 3.2 Database & Tenancy Boundaries
1. **Zero SQL Updates/Deletes on Financial Ledgers**:
   * The AI must NEVER write SQL `UPDATE` or `DELETE` statements targeting `fee_ledger_entries`. Any adjustments must be offsetting credit/debit entries.
2. **Never Execute Cross-Schema Queries**:
   * The AI must NEVER generate queries that join `tenant_a.*` with `tenant_b.*`.
   * Public schema queries are strictly restricted to tenant lookup, domain routing, and global SaaS feature configurations.
3. **No Unparameterized SQL**:
   * The AI must NEVER generate raw string concatenation for SQL queries. Always use Django ORM or parameterized cursor queries (`cursor.execute(sql, [param1, param2])`).

### 3.3 Security & Secret Governance Boundaries
1. **Never Hardcode Secrets**:
   * The AI must NEVER hardcode JWT secrets, database passwords, Razorpay keys, or AWS S3 credentials into code files. Always read from environment variables (`os.environ.get()` or `django-environ`).
2. **Never Disable Gateway RBAC**:
   * The AI must NEVER rely on frontend button hiding as the sole access control mechanism. Every DRF view must declare `permission_classes = [IsAuthenticated, HasTenantPermission]`.

---

## 4. Engineering Standards & Rules for the AI Assistant

When writing code in this repository, the AI assistant must adhere to these exact guidelines:

### 4.1 Python & Django Standards
* **Python Version**: Target Python 3.13 LTS features and strict type hints (`from typing import Optional, List, Dict, Any`).
* **Code Style**: Follow PEP 8 strictly. Classes use `PascalCase`, functions and variables use `snake_case`, constants use `UPPER_SNAKE_CASE`.
* **DRF Serializers**: Always declare `fields` explicitly in `ModelSerializer` classes. NEVER use `fields = '__all__'`.
* **Transactions**: Wrap any multi-table write operation in `with transaction.atomic():`.
* **Django Models**:
  * Every model must include `id = models.UUIDField(primary_key=True, default=uuid.uuid4, editable=False)`.
  * Every operational model must include `created_at = models.DateTimeField(auto_now_add=True)` and `updated_at = models.DateTimeField(auto_now=True)`.

### 4.2 Frontend (HTML5, Tailwind CSS, JavaScript) Standards
* **Semantic HTML**: Use proper tags (`<header>`, `<main>`, `<nav>`, `<section>`, `<article>`, `<table>`, `<button>`). Never use `<div onclick="...">` where a `<button type="button">` is appropriate.
* **Tailwind Class Organization**: Group classes logically:
  1. Layout / Display (`flex`, `grid`, `block`, `hidden`)
  2. Positioning (`relative`, `absolute`, `top-0`, `z-10`)
  3. Sizing (`w-full`, `max-w-md`, `h-10`)
  4. Spacing (`p-4`, `mx-auto`, `gap-3`)
  5. Typography (`text-sm`, `font-semibold`, `text-slate-900`)
  6. Visuals & Borders (`bg-white`, `border`, `rounded-lg`, `shadow-sm`)
  7. States (`hover:bg-blue-50`, `focus:ring-2`, `disabled:opacity-50`)
* **JavaScript Quality**:
  * Write modern vanilla ES6+ with `const` and `let` (never `var`).
  * Use `async/await` for asynchronous operations with complete `try...catch` blocks.
  * Encapsulate DOM manipulation into modular controllers or classes.

### 4.3 Testing & Quality Verification Rules
* For every new Django API endpoint, provide an automated test case inheriting from `APITestCase` verifying:
  1. Successful execution with valid tenant context.
  2. HTTP 401 Unauthorized when unauthenticated.
  3. HTTP 403 Forbidden when role lacks permission.
  4. Zero data exposure from adjacent tenant schemas.
* For PWA components, verify offline execution by testing IndexedDB queue insertion and sync receipt processing.

---

## 5. Architectural Checklist for AI Code Generation

Before outputting code or submitting changes, the AI must mentally verify:

```
[ ] 1. Is this code scoped to the active tenant schema? (No cross-tenant leak)
[ ] 2. Are financial ledger operations immutable? (No UPDATE/DELETE)
[ ] 3. Does this comply with the DPDP Act 2023? (No unauthorized minor processing)
[ ] 4. Does the UI conform to Tailwind tokens and WCAG 2.1 AA standards?
[ ] 5. Does the offline field operation function without internet? (IndexedDB ready)
[ ] 6. Are all credentials, tokens, and endpoints parameterized via environment variables?
[ ] 7. Are errors handled gracefully with user-friendly, non-blocking toast notifications?
```
