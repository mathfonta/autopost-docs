# Security Report — Secrets Check
**Date:** 2026-04-25
**Agent:** @security (Sage)
**Type:** `*secrets-check`
**Scope:** Entire project (`c:\Projetos\autopost`)
**Story:** 7.1 — Fase 4 (Smoke Test)

---

## Executive Summary

| Metric | Value |
|--------|-------|
| Files scanned | All non-binary project files |
| Confirmed exposures | 0 |
| Probable findings | 1 |
| False positives | 4 |
| Remediation stories created | 0 (MEDIUM severity — threshold: HIGH+) |
| **Overall Status** | **CLEAN** |

---

## Findings

### SEC-001 — tok.json: credential file not in .gitignore

| Field | Value |
|-------|-------|
| **ID** | SEC-001 |
| **Severity** | MEDIUM |
| **Classification** | PROBABLE |
| **File** | `tok.json` (project root) |
| **OWASP** | A05 — Security Misconfiguration |

**Descrição:**
Arquivo `tok.json` na raiz do projeto contém um JWT Supabase (`access_token`) e um `refresh_token`. O arquivo não está listado no `.gitignore`.

**Conteúdo:**
- `access_token`: JWT Supabase, `role: authenticated`, email: `probe2026@gmail.com`
- `refresh_token`: `33frropp3kip`
- **Expirado:** 2026-04-24 17:59:04 (access token expirado ontem)
- **Conta:** `probe2026@gmail.com` — claramente conta de teste/probe

**Risco:**
- O access token está expirado — sem risco imediato de uso
- O refresh token pode ainda ser válido e permitir renovar o access token
- O arquivo NÃO está no `.gitignore` — se git for inicializado e este arquivo for commitado acidentalmente, o refresh token seria exposto
- O projeto ainda não é um repositório git — sem histórico comprometido

**Por que não CRITICAL:**
- Conta de teste (`probe2026` — claramente não é usuário real)
- Access token já expirado
- Projeto ainda não tem repositório git ativo

**Recomendação:**
1. Adicionar `tok.json` ao `.gitignore`
2. Deletar `tok.json` do filesystem (arquivo temporário de teste)
3. Se o refresh token for de conta real: revogar via Supabase dashboard

---

## False Positives Documentados

| Arquivo | Padrão encontrado | Motivo do FP |
|---------|------------------|--------------|
| `backend/tests/conftest.py:15` | `"test-jwt-secret-32-chars-minimum!!"` | Valor explicitamente marcado como test fixture |
| `backend/tests/conftest.py:11` | `postgresql://user:pass@localhost/test` | Placeholder óbvio de teste |
| `backend/tests/test_meta_oauth.py:84` | `settings.META_APP_SECRET = "test-app-secret"` | Mock de teste |
| `backend/.env` | DATABASE_URL, JWT_SECRET, ANTHROPIC_API_KEY, etc. | Correto — arquivo `.env` está no `.gitignore` ✅ |

---

## Verificações Positivas (PASS)

| Check | Status | Notas |
|-------|--------|-------|
| `.env` no `.gitignore` | ✅ PASS | Padrão `.env` cobre `backend/.env` |
| `config.py` sem fallbacks para secrets | ✅ PASS | Todos os campos críticos são `str` obrigatório (sem default) |
| Credenciais de teste claramente marcadas | ✅ PASS | Conftest e test_meta_oauth usam valores `"test-*"` |
| Nenhum `eval()` com dados de usuário | ✅ PASS | Nenhuma ocorrência encontrada |
| Nenhuma connection string hardcoded em código fonte | ✅ PASS | Apenas em `.env` (correto) |
| ANTHROPIC_API_KEY apenas em `.env` | ✅ PASS | Não aparece em código fonte |
| Nenhum JWT real em fixtures de teste | ✅ PASS | Todos os tokens em tests são strings `"test-*"` |

---

## Ações Recomendadas

### Imediato (MEDIUM)

**Ação 1:** Adicionar `tok.json` ao `.gitignore`:
```
# Tokens temporários de desenvolvimento
tok.json
*.token.json
```

**Ação 2:** Deletar `tok.json` do filesystem:
```bash
rm tok.json
```

**Ação 3 (opcional):** Revogar o refresh token no Supabase Dashboard se a conta `probe2026@gmail.com` for real.

---

## MEMORY.md Update

Adicionado ao Audit History do MEMORY.md do @security.

---

*Relatório gerado por @security (Sage) — Autoridade aprovada 2026-04-25*
