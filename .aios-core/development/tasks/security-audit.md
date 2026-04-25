# Task: Security Audit (OWASP Top 10)

**Task ID:** security-audit
**Agent:** @security (Sage)
**Version:** 1.0
**Purpose:** Execute structured OWASP Top 10 audit of the AutoPost codebase, outputting a prioritized findings report and creating remediation stories for CRITICAL and HIGH findings.

---

## Inputs

- `scope` (optional): `full` | `api` | `frontend` | `infra` — default: `full`
- Project codebase at working directory
- MEMORY.md — prior audit history and known patterns

## Outputs

- `docs/qa/security-reports/security-report-{YYYY-MM-DD}.md`
- Remediation stories for CRITICAL/HIGH findings
- Updated MEMORY.md

---

## Execution Steps

### Step 0 — Load Context
1. Read `.aios-core/development/agents/security/MEMORY.md`
2. Read `.aios-core/development/checklists/security-owasp-checklist.md`

### Step 1 — Scope (full | api | frontend | infra)

### Step 2 — A01: Broken Access Control
- [ ] All API endpoints require auth (except /health)
- [ ] No horizontal privilege escalation (user only sees own data)
- [ ] Supabase RLS active on all user-data tables
- [ ] No hidden admin routes

AutoPost: check app/api/ for get_current_user() on every endpoint.

### Step 3 — A02: Cryptographic Failures
- [ ] No sensitive data in logs (tokens, passwords)
- [ ] Meta tokens not over-exposed in responses
- [ ] JWT_SECRET not hardcoded
- [ ] R2 credentials not hardcoded

### Step 4 — A03: Injection
- [ ] All DB queries via ORM (no raw SQL with f-strings)
- [ ] File upload names sanitized
- [ ] No eval()/exec() on user input

### Step 5 — A04: Insecure Design
- [ ] Rate limiting on auth and AI generation endpoints
- [ ] Failed auth doesn't reveal user existence
- [ ] Image upload validates file type (magic bytes)

### Step 6 — A05: Security Misconfiguration
- [ ] CORS not wildcard in production
- [ ] Debug mode disabled in Railway
- [ ] /docs (Swagger) restricted in production
- [ ] Error responses don't expose stack traces

### Step 7 — A06: Vulnerable Components
- [ ] Note packages with known CVEs (full audit is v1.1)

### Step 8 — A07: Auth Failures
- [ ] Passwords hashed with bcrypt/argon2
- [ ] JWT has reasonable expiry
- [ ] No credentials in URL params

### Step 9 — A08: Software Integrity
- [ ] Celery tasks validate input
- [ ] Webhook signatures validated (if any)

### Step 10 — A09: Logging & Monitoring
- [ ] Auth failures logged with IP + timestamp
- [ ] No sensitive data in log lines

### Step 11 — A10: SSRF
- [ ] No endpoints fetch external URLs from user input
- [ ] Instagram image URLs validated

---

## Report Generation

Use template `security-audit-report-tmpl.md`.
Save to: `docs/qa/security-reports/security-report-{YYYY-MM-DD}.md`

## Post-Audit Actions
1. CRITICAL finding → `*create-remediation {id}` immediately
2. HIGH finding → `*create-remediation {id}`
3. MEDIUM/LOW → document in report and MEMORY.md
4. Update MEMORY.md: Audit History + Recurring Findings
