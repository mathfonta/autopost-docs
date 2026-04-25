# Task: RLS Review

**Task ID:** security-rls-review
**Agent:** @security (Sage)
**Version:** 1.0
**Purpose:** Audit Row Level Security policies on Supabase tables to prevent cross-tenant data exposure.

---

## Inputs

- `table` (optional): specific table name — default: all tables

## AutoPost Tables to Audit

| Table | Has User Data? | RLS Required |
|-------|---------------|--------------|
| `users` | YES | CRITICAL |
| `brand_profiles` | YES | CRITICAL |
| `content_requests` | YES | CRITICAL |
| `instagram_accounts` | YES | CRITICAL |
| `push_subscriptions` | YES | CRITICAL |
| `alembic_version` | NO | Not required |

---

## Execution Steps

### Step 1 — List All Tables
Query Supabase or read migration files to get full table list. Add any table not in the inventory above.

### Step 2 — Per-Table RLS Check

For each table with user data:
- [ ] RLS is ENABLED (`ALTER TABLE {table} ENABLE ROW LEVEL SECURITY`)
- [ ] SELECT policy exists and filters by `auth.uid()` or equivalent
- [ ] INSERT policy restricts to authenticated users inserting their own data
- [ ] UPDATE policy restricts to owner only
- [ ] DELETE policy restricts to owner only (if delete is allowed)
- [ ] No policy allows `TRUE` (unrestricted access) for sensitive tables

### Step 3 — Policy Logic Verification

For each policy found:
- Read the USING clause
- Verify it references the user identifier (e.g., `user_id = auth.uid()`)
- Flag any policy with `USING (true)` — this is unrestricted

### Step 4 — Multi-Tenant Gap Analysis

AutoPost is single-tenant MVP but must be multi-tenant ready:
- [ ] All user-data tables filterable by `user_id`
- [ ] No queries that return ALL rows without user filter

---

## Output Format

```markdown
## RLS Audit — {date}

| Table | RLS Enabled | Policies | Status | Notes |
|-------|-------------|----------|--------|-------|
| users | YES | SELECT, INSERT | OK | |
| content_requests | YES | SELECT, INSERT, UPDATE | OK | |
| {table} | NO | — | CRITICAL | Missing RLS |
```

For each CRITICAL finding: `*create-remediation {table}-missing-rls`
