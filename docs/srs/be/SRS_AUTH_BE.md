# SRS Modul AUTH: Authentication — Backend (BE)

**Module:** Authentication
**Module Code:** `AUTH`
**Route:** `/login, /register`
**Priority:** Very High
**Version:** 1.0
**Implementation Status:** Planned — implementation follows PRD and current repository stack.
**Source of truth:** `01_PRD.md`, `00_PROJECT_CONTEXT.md`, `TECH_STACK_AND_TANSTACK_QUERY.md`.

---

## 1. Scope and Objective

Modul ini mendefinisikan pekerjaan Backend (BE) untuk Authentication. Dokumen ini sengaja hanya membahas satu modul agar dapat dikerjakan paralel dengan pasangan dokumen lainnya.
Target Tahap 1 adalah alur stabil, dapat diuji juri, mobile-first, aman, serta konsisten dengan kontrak API. Fitur P1/P2 tetap dicatat tetapi tidak boleh diklaim aktif sebelum implementasi dan integrasi diverifikasi.

### 1.1 PRD Feature Coverage

- F-AUTH-01 Registrasi & Login
- F-AUTH-02 Protected Routes
- F-AUTH-03 Isolasi Data Antar-Usaha
- F-AUTH-04 Akun Demo Juri
- F-AUTH-05 Rate Limit & Anti-abuse

### 1.2 Actors and Access

| Actor | Access | Constraint |
|---|---|---|
| `CRAFTER` | Read/write tenant sendiri | Tidak melihat usaha lain |
| `DEMO_JUDGE` | Sandbox sesuai modul | Data contoh, reset terbatas |
| `ADMIN` | Monitoring/dukungan sesuai kebutuhan | Tidak membaca token mentah |
| `ADVISOR` | P2 read-only ringkasan | Tidak menjadi scope Tahap 1 |

### 1.3 Shared Contract

- TypeScript strict mode; naming mengikuti module code dan Feature ID.
- Semua request memakai session cookie Supabase; client tidak mengirim atau menyimpan secret.
- Semua response error memakai `{ error: { code, message, details? } }`.
- Semua timestamp memakai ISO 8601 UTC; UI mengonversi ke timezone pengguna.
- Semua list memakai `{ items, nextCursor }` bila berpotensi besar.

## 2. BE Responsibilities

### FR-BE-AUTH-001: API Surface

Endpoint wajib/terkait: POST /api/auth/signup; POST /api/auth/signin; POST /api/auth/signout; GET /api/auth/session; GET /api/auth/me.
Semua handler memakai session Supabase, authorization tenant, validation schema, structured error, dan audit untuk aksi sensitif.

### FR-BE-AUTH-002: Domain Behavior

Entity domain: User; Business; Session; AuditEvent; RateLimitBucket.
Business rules utama: Supabase Auth cookie session; server-side session validation; business ownership verified from session; demo credentials never committed; generic credential errors; rate limit auth attempts.

### FR-BE-AUTH-003: Input Contract

- Parse JSON/form-data secara eksplisit; reject unknown atau malformed input sesuai endpoint.
- ID dari client hanya kandidat; server mengambil business scope dari session dan memverifikasi ownership.
- File/URL/text divalidasi ukuran, format, panjang, content type, dan batas rate.
- Mutation idempotency key dipakai untuk publish, webhook, reset, dan operasi yang dapat diulang.

## 3. Data Model

### 3.1 Entities

| Entity | Required fields | Notes |
|---|---|---|
| User | `id`, `business_id`, `created_at`, `updated_at` | Tenant-scoped |
| AuditEvent | `id`, `actor_id`, `action`, `resource`, `created_at` | Immutable, redacted |
| Usage/Job record | `id`, `status`, `attempts`, `error_code` | Retry-safe when applicable |

### 3.2 Common Field Rules

- UUID primary keys; foreign keys enforced in PostgreSQL.
- `business_id` indexed and present on every tenant-owned table.
- Created/updated timestamps UTC; status values use explicit enum.
- Soft delete only when audit/history requires it; hard delete follows privacy scope.
- Media stores URL/key metadata, never arbitrary executable content.
- Secrets/tokens encrypted at rest and excluded from API serializers/logs.

### 3.3 Index and Pagination

- Index `(business_id, created_at)` for list queries.
- Add foreign-key indexes for every relation.
- Use cursor pagination for messages, generations, notifications, logs, and gallery.
- Select only required columns; never use unrestricted `select *` for large lists.
- Verify expensive SQL with `EXPLAIN ANALYZE`.

## 4. API Contract

### 4.1 Request/Response

| HTTP | Meaning | Required behavior |
|---|---|---|
| 200/201 | Success | Stable typed JSON |
| 202 | Async accepted | Return job/entity ID immediately |
| 400/422 | Invalid input | Field-safe details |
| 401 | No/expired session | No data leakage |
| 403 | Wrong role/tenant | Deny without resource details |
| 404 | Missing scoped resource | Generic not found |
| 409 | State/version conflict | Explain retry/current state |
| 429 | Rate/quota limit | Retry-after/cooldown |
| 500/503 | Dependency/system failure | Correlation ID; safe message |

Success list shape: `{ items: [], nextCursor: string | null }`.
Success mutation shape: `{ data: ..., requestId: string }`.
Error shape: `{ error: { code: string, message: string, details?: unknown, requestId: string } }`.

### 4.2 Authorization Matrix

| Action | CRAFTER | DEMO_JUDGE | ADMIN | ADVISOR |
|---|---|---|---|---|
| Read own tenant | Yes | Sandbox | Status/aggregate only | P2 read |
| Create/update own data | Yes | Sandbox | No unless support path | No |
| Approve/publish/delete | Owner explicit | Sandbox | No impersonation | No |
| Operational data | No | No | Yes | No |

Authorization is enforced server-side before database query/result serialization. RLS remains active as defense in depth.

## 5. Workflow and Jobs

```mermaid
flowchart TD
A[Request] --> B[Session validation]
B --> C[Tenant/RBAC guard]
C --> D[Schema validation]
D --> E[Idempotency check]
E --> F[DB transaction / enqueue]
F --> G[Response]
G --> H[Worker if async]
H --> I[Persist result + audit + notification]
```

Workflow domain: Pengguna membuka landing; memilih daftar atau masuk; mengisi kredensial; server memvalidasi; session cookie dibuat; business aktif dimuat; pengguna diarahkan ke onboarding atau dashboard.

Background work must be bounded, observable, retry-limited, and safe on duplicate execution.
For AI work, all providers are called through `lib/ai/xkiro`/`shared ai-clients` equivalent; logging includes model, token, latency, cost, and error.

## 6. Business Rules

- `BR-AUTH-001`: Supabase Auth cookie session.
- `BR-AUTH-002`: server-side session validation.
- `BR-AUTH-003`: business ownership verified from session.
- `BR-AUTH-004`: demo credentials never committed.
- `BR-AUTH-005`: generic credential errors.
- `BR-AUTH-006`: rate limit auth attempts.
- `BR-AUTH-099`: No operation may cross business tenant boundary.
- `BR-AUTH-100`: Sensitive mutation produces an audit event with actor and resource.

## 7. Validation and Security

- Validate auth/session on every protected endpoint; do not trust client-supplied businessId.
- Apply rate limit to auth, AI generation, uploads, imports, webhooks, and expensive reports.
- Enforce content type, byte limit, image dimensions, URL scheme, and SSRF-safe website fetch.
- Imported content is untrusted data; isolate it from system instructions and sanitize output.
- Redact tokens, API keys, passwords, raw prompts containing personal data, and provider secrets.
- Use parameterized Drizzle queries and database constraints.
- Publish only explicit approved output; approval status checked again immediately before publish.

## 8. Error and Recovery Matrix

| Failure | BE response | Recovery |
|---|---|---|
| Email already registered | Safe `AUTH_...` error with request ID | Retry/manual fallback where safe |
| invalid email | Safe `AUTH_...` error with request ID | Retry/manual fallback where safe |
| weak password | Safe `AUTH_...` error with request ID | Retry/manual fallback where safe |
| invalid credentials | Safe `AUTH_...` error with request ID | Retry/manual fallback where safe |
| expired session | Safe `AUTH_...` error with request ID | Retry/manual fallback where safe |
| offline request | Safe `AUTH_...` error with request ID | Retry/manual fallback where safe |
| rate limit | Safe `AUTH_...` error with request ID | Retry/manual fallback where safe |
| demo account unavailable | Safe `AUTH_...` error with request ID | Retry/manual fallback where safe |
| Duplicate request | Existing result/status | Return same idempotent result |
| Dependency timeout | 503 or failed job state | Bounded retry; notify when terminal |

## 9. Integration and Observability

- Database: PostgreSQL/Supabase via Drizzle; migrations are versioned.
- Auth: Supabase Auth server session/cookie; RLS policy mirrors business ownership.
- Storage: HTTPS media URL requirement for Instagram; signed/private URLs where appropriate.
- AI: Xkiro adapter with timeout, fallback, safe errors, quota, and cost logging.
- Queue/scheduler: internal mechanism for async generation, publish, tasks, reminders, token refresh.
- Audit: consent, approval, delete, disconnect, reset, admin access, and security failures.
- Metrics: latency, error rate, queue age, retry count, quota, cost, and feature success.

## 10. Acceptance Criteria

- `AC-BE-AUTH-001`: Valid credentials create a session.
- `AC-BE-AUTH-002`: invalid credentials reveal no account existence.
- `AC-BE-AUTH-003`: protected route redirects.
- `AC-BE-AUTH-004`: logout invalidates session.
- `AC-BE-AUTH-005`: every business query is tenant-scoped.
- `AC-BE-AUTH-006`: rate limit returns a recoverable error.
- `AC-BE-AUTH-099`: Contract tests cover auth, tenant isolation, validation, error shape, and idempotency.

## 11. BE File Convention

- Route handlers: `app/api/auth/` or resource-oriented paths from the API contract.
- Domain logic: `modules/auth/server/`.
- Validation: shared schema module; no duplicated ad-hoc parsing across handlers.
- DB schema: `lib/db/schema.ts`; migrations under `drizzle/`.
- External integrations: `lib/ai`, `lib/integrations`; secrets only environment variables.
- Every implementation task records an entry in `docs/EXECUTION_LOG.md`.

## 12. BE Test Checklist

- Unit-test validation, status transitions, quota, and business rules.
- Integration-test route handler with authenticated owner, wrong tenant, missing session, and admin.
- Test duplicate mutation/webhook/publish with same idempotency key.
- Test dependency timeout, retry ceiling, failed terminal state, and audit creation.
- Test migrations, indexes, cursor pagination, and RLS policies.
- Run `npm run lint` and `npm run build`; keep a small runnable check for non-trivial logic.

## 13. Handoff Notes

FE dan BE wajib menggunakan Feature ID, enum status, error code, and response shape dari dokumen ini. Perubahan kontrak harus mengubah pasangan SRS FE/BE secara bersamaan.
Tidak ada secret, credential demo, atau token eksternal di repository.

*End of SRS_BE_AUTH — Authentication*
