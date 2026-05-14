# Security Agent Memory (Sage)

## Commands

| Comando | Descrição | Task |
|---------|-----------|------|
| `*audit [scope]` | OWASP Top 10 full audit | security-audit.md |
| `*threat-model {epic\|story}` | STRIDE threat model | security-threat-model.md |
| `*secrets-check [path]` | Scan de secrets expostos | security-secrets-check.md |
| `*rls-review [table]` | Auditoria RLS Supabase | security-rls-review.md |
| `*dep-audit [python\|node\|all]` | Auditoria de dependências Python + npm | security-python-dep-audit.md |
| `*create-remediation {id}` | Criar story de remediação | — |
| `*status` | Resumo de findings abertos | — |

### *dep-audit — Instruções de Uso

```bash
# Auditoria completa (Python + Node.js)
@security *dep-audit

# Somente Python (backend FastAPI)
@security *dep-audit python

# Somente Node.js (frontend Next.js)
@security *dep-audit node
```

**Saída:** `docs/qa/security-reports/dep-audit-YYYY-MM-DD.md`  
**Template:** `security-dep-audit-report-tmpl.md`  
**Task Python:** `security-python-dep-audit.md`  
**CI:** `backend/.github/workflows/security.yml` + `frontend/.github/workflows/security.yml`

**Gate Logic:**
- CRITICAL → FAIL (bloqueia CI, cria story de remediação)
- HIGH → CONCERNS (documenta como dívida técnica)
- MEDIUM/LOW → PASS (registra em MEMORY.md)
- pip-audit/npm indisponível → SKIPPED (warn, não bloqueia)

---

## Active Patterns

### AutoPost Stack — Known Risk Areas (baseline)

| Area | Risk | Severity | Status |
|------|------|----------|--------|
| Supabase RLS | Multi-tenant data exposure on new tables | CRITICAL | Monitorar |
| Railway env vars | Meta API keys, JWT_SECRET, Supabase URL | HIGH | Monitorar |
| Cloudflare R2 | Bucket policy — images privadas de clientes | HIGH | Monitorar |
| Meta OAuth tokens | Long-lived tokens (60d) armazenados em DB | HIGH | Monitorar |
| Celery workers | NullPool + asyncio.run() — DB access em processo fork | MEDIUM | Monitorar |
| JWT_SECRET | Sem rotação implementada | MEDIUM | Monitorar |
| CORS config | allow_origin_regex `.*\.vercel\.app` com credentials=True — EXPLOITÁVEL | HIGH | **SEC-01 aberta** |
| Onboarding API | Sem rate limiting em /message e /start — abuso de Claude API | MEDIUM | Tech debt |
| /health endpoint | Expõe env + erros de DB sem autenticação | MEDIUM | Tech debt |
| Meta access token | Armazenado plaintext no DB | MEDIUM | Tech debt |
| Push endpoint SSRF | Subscription endpoint usado sem validação de domínio | MEDIUM | Tech debt |
| Prompt injection | strategy/user_context passados direto para Claude sem sanitização | MEDIUM | Tech debt |

### Padrões de Verificação

- Toda tabela nova no Supabase requer RLS imediata — nunca deixar para depois
- Variáveis de ambiente com `_SECRET`, `_KEY`, `_TOKEN`, `_PASSWORD` = audit obrigatório
- Endpoints sem autenticação = verificar se intencional (ex: `/health` é esperado)
- Resposta de erro nunca deve expor stack trace ou dados internos

### Git Restrictions

- READ-ONLY: `git status`, `git diff`, `git log`, `git show`, `git grep`
- NEVER: `git add`, `git commit`, `git push`

## Audit History

<!-- Formato: - YYYY-MM-DD | Tipo | Score | Findings (C/H/M/L) | Report -->
- 2026-04-25 | Secrets Check (Smoke Test) | CLEAN | 0C 0H 1M 0L | security-secrets-check-2026-04-25.md
- 2026-04-25 | Dep Audit — Python (pip-audit 2.10.0) | PASS | 0C 0H 0M 0L | dep-audit-2026-04-25.md
- 2026-04-26 | Dep Audit — Node.js (npm audit 10.8.2) | PASS | 0C 0H 2M 0L | dep-audit-2026-04-26.md
- 2026-05-05 | OWASP Top 10 Full Audit (análise estática) | CONCERNS | 0C 1H 5M 3L | security-report-2026-05-05.md
- 2026-05-05 | Secrets Check (varredura completa) | CLEAN | 0C 0H 0M 3-INFO | security-secrets-check-2026-05-05.md
- 2026-05-05 | RLS Review (todas as tabelas — análise estática) | CONCERNS | 0C 1H 1M 0L | rls-review-2026-05-05.md
- 2026-05-05 | Threat Model STRIDE (sistema completo) | CONCERNS | 0C 0H 4M 3L + 1 BUG | threat-model-2026-05-05.md

## Recurring Findings
<!-- Vulnerabilidades que reaparecem entre auditorias — sinal de débito sistêmico -->

## Resolved Findings
<!-- Vulnerabilidades que foram remediadas — com data e story de referência -->
<!-- Formato: - YYYY-MM-DD | Finding | Story | Confirmed Fixed -->

## Promotion Candidates
<!-- Padrões que aparecem em 3+ auditorias — candidatos para global APRENDIZADOS.md -->

## Findings Abertos (não remediados)

| ID | Severidade | Descrição | Story |
|----|-----------|-----------|-------|
| HIGH-001 | HIGH | CORS regex `.*\.vercel\.app` + credentials=True | SEC-01 |
| HIGH-RLS-001 | HIGH | `clients_update_own` permite escalação de plano via PostgREST | SEC-02 |
| MEDIUM-001 | MEDIUM | Sem rate limit em /onboarding/message e /start | — |
| MEDIUM-002 | MEDIUM | /health expõe env + erros de DB sem auth | — |
| MEDIUM-003 | MEDIUM | meta_access_token plaintext no banco | — |
| MEDIUM-004 | MEDIUM | Prompt injection em strategy/user_context | — |
| MEDIUM-005 | MEDIUM | SSRF via push subscription endpoint | — |
| MEDIUM-RLS-001 | MEDIUM | `content_requests_update_own` sem restrição de colunas via PostgREST | — |
