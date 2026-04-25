# Task: STRIDE Threat Model

**Task ID:** security-threat-model
**Agent:** @security (Sage)
**Version:** 1.0
**Purpose:** Apply the STRIDE threat modeling framework to an Epic or Story, identifying threats per component and producing a structured threat report with mitigations.

---

## Inputs

- `target`: Epic ID (e.g., `epic-2`) or Story ID (e.g., `2.7`) — **required**
- Story/Epic file from `docs/stories/`
- `MEMORY.md` — prior threat findings

## Outputs

- Threat model report appended to the relevant story file's Dev Notes (or as standalone doc)
- Remediation stories for HIGH/CRITICAL threats without mitigations

---

## STRIDE Framework

| Letter | Threat | Question |
|--------|--------|---------|
| **S** | Spoofing | Can an attacker impersonate a user or service? |
| **T** | Tampering | Can data be modified without authorization? |
| **R** | Repudiation | Can actions be denied without audit trail? |
| **I** | Information Disclosure | Can sensitive data be exposed? |
| **D** | Denial of Service | Can the service be made unavailable? |
| **E** | Elevation of Privilege | Can a low-privilege user gain higher access? |

---

## Execution Steps

### Step 1 — Load Target
1. Read the story/epic file from `docs/stories/`
2. Identify the components involved (API endpoints, DB tables, external integrations, queues)
3. Read `MEMORY.md` for prior findings on these components

### Step 2 — Component Inventory
List all components in the target:
- API endpoints (FastAPI routes)
- Database tables (Supabase)
- External services (Meta API, Cloudflare R2, Railway)
- Background jobs (Celery tasks)
- Frontend pages/actions

### Step 3 — STRIDE Analysis per Component

For each component, evaluate all 6 threat categories:

**Format per finding:**
```
Threat: {S|T|R|I|D|E} — {description}
Component: {file or service}
Attack vector: {how it could be exploited}
Impact: CRITICAL | HIGH | MEDIUM | LOW
Mitigation: {existing control or recommended fix}
Status: Mitigated | Partial | Open
```

### Step 4 — AutoPost-Specific STRIDE Checklist

**Spoofing:**
- [ ] OAuth state parameter validated to prevent CSRF
- [ ] JWT signature verified on every protected endpoint
- [ ] Meta webhook source validated (if applicable)

**Tampering:**
- [ ] Content request status transitions validated (can't skip from `pending` to `published`)
- [ ] Caption edits rejected if status ≠ `awaiting_approval`
- [ ] R2 object keys not predictable/guessable

**Repudiation:**
- [ ] All publish actions logged with user ID + timestamp
- [ ] Meta API calls logged with response status
- [ ] Failed auth attempts logged

**Information Disclosure:**
- [ ] Meta tokens not returned in list endpoints
- [ ] Error messages generic (no stack traces)
- [ ] Supabase RLS prevents cross-tenant reads

**Denial of Service:**
- [ ] AI generation endpoints rate-limited (cost + availability)
- [ ] File upload size limited
- [ ] Celery queue not exploitable by one user hogging resources

**Elevation of Privilege:**
- [ ] No admin bypass via parameter manipulation
- [ ] Content from other users inaccessible even with valid JWT
- [ ] Celery tasks don't execute with broader permissions than the requesting user

### Step 5 — Report Assembly

Structure:
```markdown
## STRIDE Threat Model — {Target} — {Date}

**Components analyzed:** {count}
**Threats identified:** {count} (C: X, H: X, M: X, L: X)
**Open mitigations:** {count}

### Findings
{table of all findings}

### Risk Summary
{narrative of top risks and recommended priority}
```

### Step 6 — Post-Analysis Actions

1. For CRITICAL/HIGH open threats: `*create-remediation {threat-id}`
2. Update MEMORY.md with new threat patterns
3. Note any components that need re-modeling after implementation
