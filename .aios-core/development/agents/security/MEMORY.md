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
| CORS config | allow_origin_regex — verificar permissividade | LOW | Monitorar |

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

## Recurring Findings
<!-- Vulnerabilidades que reaparecem entre auditorias — sinal de débito sistêmico -->

## Resolved Findings
<!-- Vulnerabilidades que foram remediadas — com data e story de referência -->
<!-- Formato: - YYYY-MM-DD | Finding | Story | Confirmed Fixed -->

## Promotion Candidates
<!-- Padrões que aparecem em 3+ auditorias — candidatos para global APRENDIZADOS.md -->
