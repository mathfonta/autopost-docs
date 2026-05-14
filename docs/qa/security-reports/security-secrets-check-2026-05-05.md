# Security Report — Secrets Check
**Data:** 2026-05-05
**Agente:** @security (Sage)
**Tipo:** `*secrets-check`
**Escopo:** Projeto completo (`c:\Projetos\autopost`) — backend, frontend, docs, AIOX

---

## Resumo Executivo

| Métrica | Valor |
|---------|-------|
| Arquivos rastreados pelo git verificados | Todos (backend .py, frontend .ts/.tsx, docs .md) |
| Exposições confirmadas | **0** |
| Achados informacionais | 3 |
| Histórico git varrido por tokens reais | LIMPO |
| **Status Geral** | **CLEAN** |

---

## Verificações Executadas

| Verificação | Comando / Padrão | Resultado |
|-------------|-----------------|-----------|
| Chaves Anthropic (`sk-ant-api*`) em código | `git grep` em *.py, *.ts | ✅ NENHUMA |
| Tokens Meta (`EAA*` 40+ chars) em código | `git grep` em *.py, *.ts | ✅ NENHUMA |
| JWTs (`eyJ*` 50+ chars) hardcoded | `git grep` em *.py, *.ts | ✅ NENHUMA |
| Secrets em variáveis de ambiente no código | `password=`, `secret=`, etc. em código não-teste | ✅ NENHUMA |
| Arquivos `.env` no git | `git ls-files` | ✅ NÃO rastreados |
| `.env` no `.gitignore` (raiz e backend) | `grep .gitignore` | ✅ CONFIRMADO |
| Secrets em arquivos `.env.example` versionados | Leitura direta | ✅ Apenas placeholders |
| Secrets em histórico git (commits anteriores) | `git log -p` com filtros | ✅ LIMPO |
| `tok.json` (achado anterior 2026-04-25) | `git ls-files --error-unmatch` + `git log` | ✅ RESOLVIDO — não existe e nunca foi commitado |
| Credenciais nas migrações Alembic | `git grep` em migrations/ | ✅ NENHUMA |
| Frontend source com secrets (`NEXT_PUBLIC_API_KEY`, etc.) | `git grep` em frontend/src/ | ✅ NENHUMA |
| Conexões postgres/redis com senha em docs | `git grep` em docs/ | ✅ NENHUMA |

---

## Achados Informacionais (não requerem remediação imediata)

### INFO-001 — `META_ACCESS_TOKEN=EAAUcv...` no `.env.example`

**Arquivo:** `backend/.env.example:37`

O placeholder usa o prefixo real dos tokens Meta (`EAAUcv...`). Scanners automáticos de secrets (truffleHog, gitleaks) podem gerar falso positivo. Recomendação: substituir por `META_ACCESS_TOKEN=EAAUcv[seu_token_aqui]` para deixar claro que é placeholder.

**Severidade:** Informacional — não expõe secret real, apenas afeta automação de scanning.

---

### INFO-002 — Conta de teste probe2026@gmail.com em `.claude/settings.local.json`

**Arquivo:** `.claude/settings.local.json` (NÃO rastreado pelo git)

O arquivo local de configurações do Claude Code contém uma chamada curl com `email: probe2026@gmail.com` e `password: Probe1234!`, usada como allow-list de ferramenta. O arquivo está gitignored. A conta foi analisada no secrets check anterior (2026-04-25) e confirmada como conta de teste.

**Status do tok.json anterior:** Resolvido — arquivo não existe mais no disco e nunca foi commitado.

**Severidade:** Informacional — arquivo local apenas, nunca exposto externamente.

---

### INFO-003 — Railway URL em `.env.example` do frontend

**Arquivo:** `frontend/.env.example:1`

```
NEXT_PUBLIC_API_URL=https://espectra-api-production.up.railway.app
```

Esta URL é tecnicamente pública (é o endpoint da API que clientes acessam), mas revela o nome do projeto Railway (`espectra-api-production`). Não é um secret, mas revela informação de infraestrutura. Aceitável para projeto atual.

**Severidade:** Informacional — URL pública, sem risco de segurança direto.

---

## Verificações Positivas (PASS)

| Check | Status | Detalhe |
|-------|--------|---------|
| `.env` nunca commitado | ✅ PASS | Histórico git limpo, gitignore correto |
| `config.py` — sem fallbacks hardcoded para secrets | ✅ PASS | Todos os campos críticos são `str` obrigatório sem default |
| Testes usam valores claramente fictícios | ✅ PASS | `"test-jwt-secret"`, `"sk-ant-test"`, `"test-access-key"` |
| Nenhum eval() com entrada de usuário | ✅ PASS | Nenhuma ocorrência em código de produção |
| Nenhuma chave privada VAPID hardcoded | ✅ PASS | Apenas via env var `VAPID_PRIVATE_KEY` |
| Credenciais de terceiros (Google Drive, Dropbox, OneDrive) | ✅ PASS | Apenas via env vars opcionais |
| R2 credentials | ✅ PASS | Apenas via env vars — nunca em código |

---

## Resolução de Achados Anteriores

| ID (2026-04-25) | Achado | Status Atual |
|-----------------|--------|-------------|
| SEC-001 | `tok.json` com JWT Supabase não em .gitignore | ✅ RESOLVIDO — arquivo deletado, nunca commitado, não está no índice git |

---

## Conclusão

O codebase está **CLEAN** em relação a secrets expostos. Nenhuma credencial real foi encontrada em código versionado ou no histórico git. Os únicos achados são informacionais e não requerem ação imediata.

**Próxima secrets check recomendada:** Antes do lançamento público ou quando novas integrações forem adicionadas (Google Drive, Dropbox, etc.).

— Sage, guardando as fronteiras 🛡️
