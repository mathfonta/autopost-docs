# Story SEC-02 — Remover política `clients_update_own` do RLS (escalação de plano via PostgREST)

**ID:** SEC-02
**Epic:** Security
**Status:** Ready
**Prioridade:** HIGH — deve ser feito antes do lançamento público
**Criada por:** @security (Sage) — autoridade direta conforme aprovação Matheus 2026-04-25
**Data:** 2026-05-05

---

## Descrição

A política RLS `clients_update_own` na tabela `clients` permite que qualquer usuário autenticado faça `PATCH` diretamente no Supabase PostgREST e altere qualquer campo do próprio registro — incluindo `plan` (nível de assinatura), `is_active`, e `brand_profile`.

**Exemplo de exploit:**
```http
PATCH https://{project}.supabase.co/rest/v1/clients?id=eq.{uuid}
Authorization: Bearer {valid_jwt}
Content-Type: application/json

{"plan": "enterprise"}
```

Isso resulta em **escalação de privilégios** — usuário se promove de `starter` para `enterprise` sem pagar.

**Achado de auditoria:** HIGH-RLS-001 no relatório `docs/qa/security-reports/rls-review-2026-05-05.md`
**OWASP:** A01 — Broken Access Control

---

## Por que a política pode ser removida com segurança

Todos os updates legítimos de `clients` no AutoPost passam pelo backend FastAPI, que usa `SUPABASE_SERVICE_KEY` (service_role). O service_role bypassa RLS por design. Não existe nenhum fluxo no app em que o usuário faça update direto na tabela `clients` via SDK.

Removendo a política: **nenhum usuário pode mais alterar diretamente seu registro de client** — inclusive campos legítimos como `brand_profile`. Mas isso não quebra nada porque o backend (via service_role) continua tendo acesso irrestrito.

---

## Critérios de Aceite

- [ ] Política `clients_update_own` removida via migration Alembic
- [ ] FORCE RLS continua habilitado em `clients`
- [ ] Política `clients_select_own` permanece (usuário ainda lê o próprio registro se precisar via SDK no futuro)
- [ ] Teste: PATCH direto via PostgREST com JWT válido retorna `403 Forbidden` (ou `0 rows affected`)
- [ ] Teste: Backend API (via service_role) continua conseguindo atualizar `clients` normalmente
- [ ] Migration tem downgrade que restaura a política

---

## Escopo

**IN:**
- Nova migration Alembic que dropa a política `clients_update_own`
- Incluir também a política análoga de `content_requests_update_own` se decidido pelo time

**OUT:**
- Não alterar as políticas SELECT (usuários ainda podem ler o próprio registro)
- Não alterar as políticas INSERT/DELETE (já corretas)
- Não alterar lógica da API

---

## Tarefas

- [ ] T1: Criar migration `{hash}_drop_clients_update_policy.py` com upgrade/downgrade
- [ ] T2: Verificar se há código no frontend que usa Supabase SDK diretamente para atualizar `clients` (se sim, documentar como exceção)
- [ ] T3: Aplicar migration em staging e testar via curl
- [ ] T4: Commitar

---

## Implementação de Referência

```python
# migration: drop_clients_update_policy.py

def upgrade() -> None:
    op.execute('DROP POLICY IF EXISTS "clients_update_own" ON clients')
    # OPCIONAL: também remover content_requests_update_own
    # op.execute('DROP POLICY IF EXISTS "content_requests_update_own" ON content_requests')

def downgrade() -> None:
    op.execute("""
        CREATE POLICY "clients_update_own"
        ON clients FOR UPDATE
        USING (auth.uid() = id)
        WITH CHECK (auth.uid() = id)
    """)
```

---

## Riscos

- **Baixo:** Remover UPDATE policy para usuários é seguro — o service_role (backend) não é afetado. O único risco é se houver código de frontend fazendo update direto via SDK (improvável, mas verificar no T2).

---

## Critério de Done

- Migration aplicada em produção
- Teste confirma que POST direto ao PostgREST é rejeitado
- QA gate PASS

---

## Change Log

| Data | Agente | Ação |
|------|--------|------|
| 2026-05-05 | @security (Sage) | Story criada a partir de HIGH-RLS-001 da auditoria RLS |
