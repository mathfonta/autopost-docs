# Security Agent Memory (Sage)

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

## Recurring Findings
<!-- Vulnerabilidades que reaparecem entre auditorias — sinal de débito sistêmico -->

## Resolved Findings
<!-- Vulnerabilidades que foram remediadas — com data e story de referência -->
<!-- Formato: - YYYY-MM-DD | Finding | Story | Confirmed Fixed -->

## Promotion Candidates
<!-- Padrões que aparecem em 3+ auditorias — candidatos para global APRENDIZADOS.md -->
