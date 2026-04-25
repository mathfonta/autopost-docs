# Security Checklist — OWASP Top 10 (AutoPost)

**Agente:** @security (Sage)
**Uso:** Executado durante `*audit`. Marcar cada item após verificação.

---

## A01 — Broken Access Control

- [ ] Todos os endpoints em `app/api/` verificam `get_current_user()` (exceto `/health`, `/auth/login`, `/auth/register`)
- [ ] `GET /content-requests` retorna apenas registros do usuário autenticado
- [ ] `PATCH /content-requests/{id}` verifica que o `id` pertence ao usuário autenticado
- [ ] `POST /content-requests/{id}/approve` verifica ownership
- [ ] `POST /content-requests/{id}/reject` verifica ownership
- [ ] `GET /meta/status` retorna dados apenas do usuário autenticado
- [ ] Não existe endpoint que retorne dados de todos os usuários sem verificação de admin
- [ ] Supabase RLS habilitado em: `users`, `brand_profiles`, `content_requests`, `instagram_accounts`, `push_subscriptions`

**Status:** [ ] PASS [ ] FAIL [ ] PARTIAL

---

## A02 — Cryptographic Failures

- [ ] `JWT_SECRET` carregado de variável de ambiente, não hardcoded
- [ ] `META_APP_SECRET` não aparece em nenhum log
- [ ] `DATABASE_URL` não aparece em nenhum log
- [ ] `CLOUDFLARE_R2_SECRET_ACCESS_KEY` não aparece em nenhum log
- [ ] Tokens Meta (`access_token`) não retornados em respostas de listagem
- [ ] Senhas armazenadas com bcrypt (verificar `app/core/security.py`)
- [ ] Comunicação HTTPS enforced em produção (Railway + Vercel)

**Status:** [ ] PASS [ ] FAIL [ ] PARTIAL

---

## A03 — Injection

- [ ] Todas as queries usam SQLAlchemy ORM ou parâmetros bindados (nenhum `f"SELECT * FROM {table}"`)
- [ ] Nomes de arquivos R2 sanitizados antes do upload (`uuid` gerado server-side, não baseado no nome do arquivo do usuário)
- [ ] Nenhum `eval()` ou `exec()` com dados de usuário
- [ ] Prompts enviados ao Claude não incluem dados de usuário sem sanitização (prompt injection)
- [ ] Metadados de imagem não passados diretamente para shell commands

**Status:** [ ] PASS [ ] FAIL [ ] PARTIAL

---

## A04 — Insecure Design

- [ ] Rate limiting em `POST /auth/login` (prevenir brute force)
- [ ] Rate limiting em `POST /content-requests` (prevenir abuso de custo de IA)
- [ ] Resposta de login não diferencia "usuário não existe" de "senha errada"
- [ ] Upload de imagem valida tipo de arquivo por magic bytes (não apenas extensão)
- [ ] Tamanho máximo de arquivo enforced no upload

**Status:** [ ] PASS [ ] FAIL [ ] PARTIAL

---

## A05 — Security Misconfiguration

- [ ] `CORS_ORIGINS` em produção não inclui `*`
- [ ] `DEBUG=False` no ambiente Railway de produção
- [ ] `/docs` e `/redoc` (Swagger) desabilitados ou protegidos em produção
- [ ] Respostas de erro 500 retornam mensagem genérica, não stack trace
- [ ] Headers de segurança presentes (X-Content-Type-Options, X-Frame-Options)
- [ ] Variáveis de ambiente com `_SECRET` ou `_KEY` não aparecem nos logs do Railway

**Status:** [ ] PASS [ ] FAIL [ ] PARTIAL

---

## A06 — Vulnerable Components

- [ ] Python packages: nenhum com CVE crítico conhecido (verificar PyPI advisories)
- [ ] Node packages: nenhum com CVE crítico conhecida
- [ ] Nota: `pip-audit` e `npm audit` completos são escopo v1.1 — registrar como debt se não executados

**Status:** [ ] PASS [ ] FAIL [ ] PARTIAL / [ ] DEFERRED (v1.1)

---

## A07 — Identification and Authentication Failures

- [ ] Senhas hasheadas com bcrypt (fator de trabalho ≥ 12) ou argon2
- [ ] JWT tem `exp` configurado (não eterno)
- [ ] Cookies de auth têm `httpOnly` e `secure` se usados
- [ ] Logout invalida o token/sessão do lado servidor (se stateful)
- [ ] Nenhuma credencial em query parameters nas URLs

**Status:** [ ] PASS [ ] FAIL [ ] PARTIAL

---

## A08 — Software and Data Integrity

- [ ] Tasks Celery (`generate_copy`, `retry_generate_copy`, `publish_post`) validam formato dos dados recebidos
- [ ] Nenhuma deserialização insegura de dados externos (pickle, yaml.load sem safe)
- [ ] Dependências instaladas de fontes confiáveis (PyPI oficial, npm oficial)

**Status:** [ ] PASS [ ] FAIL [ ] PARTIAL

---

## A09 — Security Logging and Monitoring

- [ ] Falhas de autenticação logadas com timestamp (sem expor senha ou token)
- [ ] Erros 500 logados com contexto suficiente para investigar
- [ ] Nenhuma linha de log contém: senha, token, connection string, chave de API
- [ ] Logs do Railway acessíveis para investigação pós-incidente

**Status:** [ ] PASS [ ] FAIL [ ] PARTIAL

---

## A10 — Server-Side Request Forgery (SSRF)

- [ ] Nenhum endpoint aceita URL de usuário e faz fetch dessa URL no servidor
- [ ] URLs de imagem para Instagram validadas (formato esperado, não redirecionamentos)
- [ ] Presigned URLs do R2 têm TTL configurado (não permanentes)

**Status:** [ ] PASS [ ] FAIL [ ] PARTIAL

---

## Score Final

| Categoria | Status |
|-----------|--------|
| A01 Broken Access Control | |
| A02 Cryptographic Failures | |
| A03 Injection | |
| A04 Insecure Design | |
| A05 Security Misconfiguration | |
| A06 Vulnerable Components | |
| A07 Auth Failures | |
| A08 Software Integrity | |
| A09 Logging & Monitoring | |
| A10 SSRF | |
| **Total PASS** | **/10** |
