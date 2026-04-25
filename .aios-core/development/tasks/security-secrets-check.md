# Task: Secrets Check

**Task ID:** security-secrets-check
**Agent:** @security (Sage)
**Version:** 1.0
**Purpose:** Scan the codebase, configuration files, and git history for exposed secrets, API keys, tokens, and hardcoded credentials.

---

## Inputs

- `path` (optional): specific directory or file to scan — default: entire project
- Working directory: `c:\Projetos\autopost`

## Outputs

- Findings listed inline (if any secrets found)
- Remediation story created for each confirmed exposure
- MEMORY.md updated with findings

---

## Execution Steps

### Step 1 — Pattern Scan (Grep-based)

Search for common secret patterns in all non-binary files:

**High-confidence patterns:**
```
- Patterns containing: sk-, pk-, rk-, ey-, secret, token, password, api_key, apikey
- AWS patterns: AKIA, ASIA
- JWT patterns: eyJ (base64-encoded JWT)
- Connection strings: postgresql://, mysql://, mongodb://
- Private key blocks: -----BEGIN (RSA|EC|OPENSSH) PRIVATE KEY
```

**AutoPost-specific patterns to check:**
- [ ] `META_APP_SECRET` value hardcoded anywhere
- [ ] `JWT_SECRET` value hardcoded anywhere
- [ ] `DATABASE_URL` with credentials hardcoded
- [ ] `CLOUDFLARE_R2_SECRET` hardcoded
- [ ] Supabase service role key hardcoded
- [ ] Any `access_token` values in test fixtures or mock data

### Step 2 — File Inventory Check

Check that these files are in `.gitignore` (or don't exist):
- [ ] `.env` — must be in .gitignore
- [ ] `.env.local`, `.env.production` — must be in .gitignore
- [ ] Any `*credentials*`, `*secrets*`, `*keyfile*` files

### Step 3 — Code Pattern Check

Look for:
- [ ] Hardcoded strings that look like API keys (random alphanumeric 20+ chars)
- [ ] `os.environ.get("KEY", "hardcoded-fallback")` where fallback is a real value
- [ ] Test files with real credentials (should use mock values only)
- [ ] Comments containing credentials (`# password: abc123`)
- [ ] Config files with credentials committed to repo

### Step 4 — Environment Variable Usage Check

Verify all secrets are loaded from environment:
- [ ] `backend/app/core/config.py` — all sensitive values from `os.environ`
- [ ] No fallback values for production secrets (fail fast if missing)
- [ ] Railway env vars match what config.py expects

### Step 5 — Git History Check (if git repo exists)

```bash
git log --all --full-history -- "*.env" "*.pem" "*secret*" "*credential*"
```

Check if any sensitive files were ever committed and later removed.
If found: note that removal from git history requires `git filter-branch` or BFG — create story.

### Step 6 — Results Classification

For each finding:
- **CONFIRMED** — actual secret value found in code → CRITICAL, create story immediately
- **PROBABLE** — looks like a secret but needs verification → HIGH
- **FALSE POSITIVE** — example value, placeholder, or already properly env-loaded → document and skip

---

## Common False Positives (do not flag)

- `SECRET_KEY = os.environ["SECRET_KEY"]` — correct pattern, not a finding
- `"your-secret-key-here"` in documentation/README
- Test mock values clearly labeled as fake
- Base64-encoded non-sensitive strings that match JWT pattern

---

## Remediation Story Template (for confirmed findings)

```
Title: [SECURITY] Remove hardcoded {type} from {file}
Severity: CRITICAL
Description: {specific finding with file:line reference}
Steps: 1. Remove hardcoded value, 2. Rotate the exposed credential, 3. Add to .gitignore if file
```
