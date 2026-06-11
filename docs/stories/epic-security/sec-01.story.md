# Story SEC-01 — Corrigir CORS: substituir regex `.*\.vercel\.app` por allowlist explícita

**ID:** SEC-01
**Epic:** Security
**Status:** Ready
**Prioridade:** HIGH — deve ser feito antes do lançamento público
**Criada por:** @security (Sage) — autoridade direta conforme aprovação Matheus 2026-04-25
**Data:** 2026-05-05

---

## Descrição

O middleware CORS em `backend/app/main.py` usa um regex `r"https://.*\.vercel\.app"` com `allow_credentials=True`. Isso permite que **qualquer** app hospedada na plataforma Vercel (incluindo apps maliciosas criadas por atacantes) faça requisições autenticadas à API AutoPost com cookies/tokens reais dos usuários.

**Achado de auditoria:** HIGH-001 no relatório `docs/qa/security-reports/security-report-2026-05-05.md`
**OWASP:** A05 — Security Misconfiguration

---

## Critérios de Aceite

- [ ] `allow_origin_regex` removido de `main.py`
- [ ] `allow_origins` contém lista explícita e fechada de origens autorizadas
- [ ] `http://localhost:3000` permitido (desenvolvimento)
- [ ] `https://autopost.com.br` permitido (produção)
- [ ] `https://autopost.app.br` permitido (novo domínio registrado)
- [ ] URL atual do frontend Vercel (`autopost-frontend-one.vercel.app`) permitida explicitamente, se ainda necessária
- [ ] Nenhuma outra origem Vercel genérica é aceita
- [ ] Teste: requisição de `https://malicious.vercel.app` retorna erro CORS
- [ ] Teste: requisição do frontend legítimo continua funcionando normalmente

---

## Escopo

**IN:**
- Arquivo `backend/app/main.py` — bloco `CORSMiddleware`
- Documentação do motivo da mudança em `main.py` (comentário inline, se necessário)

**OUT:**
- Não alterar outros middlewares ou configurações
- Não alterar variáveis de ambiente

---

## Tarefas

- [ ] T1: Substituir configuração CORS em `main.py`
- [ ] T2: Verificar no frontend (`frontend/`) se há alguma origem Vercel hardcoded que precise ser adicionada à lista
- [ ] T3: Testar manualmente no ambiente de desenvolvimento
- [ ] T4: Commitar e documentar no Change Log

---

## Implementação de Referência

**Antes (inseguro):**
```python
app.add_middleware(
    CORSMiddleware,
    allow_origins=[
        "http://localhost:3000",
        "https://autopost.com.br",
    ],
    allow_origin_regex=r"https://.*\.vercel\.app",
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)
```

**Depois (seguro):**
```python
app.add_middleware(
    CORSMiddleware,
    allow_origins=[
        "http://localhost:3000",
        "https://autopost.com.br",
        "https://autopost.app.br",
        "https://autopost-frontend-one.vercel.app",
    ],
    allow_credentials=True,
    allow_methods=["GET", "POST", "PUT", "PATCH", "DELETE", "OPTIONS"],
    allow_headers=["Authorization", "Content-Type", "Accept"],
)
```

**Nota:** Se no futuro houver necessidade de múltiplos ambientes Vercel (preview deployments), adicionar cada URL explicitamente ou mover a lista para variável de ambiente `ALLOWED_ORIGINS`.

---

## Riscos

- **Baixo:** Risco de quebrar o frontend se a URL atual do Vercel não for incluída na lista. Mitigado pelo T2 (verificar URL atual antes de commitar).

---

## Critério de Done

- Código alterado e commitado
- Teste manual de CORS confirma bloqueio de origens não autorizadas
- QA gate PASS

---

## Change Log

| Data | Agente | Ação |
|------|--------|------|
| 2026-05-05 | @security (Sage) | Story criada a partir de achado HIGH-001 da auditoria OWASP |
