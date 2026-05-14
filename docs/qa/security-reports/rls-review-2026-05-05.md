# RLS Review — Supabase Row Level Security
**Data:** 2026-05-05
**Agente:** @security (Sage)
**Tipo:** `*rls-review` (todas as tabelas)
**Fonte de análise:** Migrações Alembic + modelos SQLAlchemy (análise estática)

---

## Mapa de Tabelas

| Tabela | Gerenciada por | RLS Enabled | RLS Force | Achados |
|--------|---------------|-------------|-----------|---------|
| `clients` | Alembic | ✅ | ✅ | HIGH-RLS-001 |
| `content_requests` | Alembic | ✅ | ✅ | MEDIUM-RLS-001 |
| `auth.users` | Supabase Auth | ✅ (Supabase) | n/a | Nenhum (gerenciado pelo Supabase) |

**Tabelas ausentes detectadas:** Nenhuma — O projeto não usa Supabase Storage (usa Cloudflare R2), não tem CLI Supabase config local, e o cerebro/analytics usam filesystem/PostHog (sem tabelas DB).

---

## Análise por Tabela

---

### Tabela: `clients`

**Migração RLS:** `b3663fc8e2c6_enable_rls_on_clients.py`

```sql
ALTER TABLE clients ENABLE ROW LEVEL SECURITY;
ALTER TABLE clients FORCE ROW LEVEL SECURITY;
```

#### Políticas configuradas

| Política | Operação | USING | WITH CHECK | Status |
|----------|----------|-------|------------|--------|
| `clients_select_own` | SELECT | `auth.uid() = id` | — | ✅ Correta |
| `clients_update_own` | UPDATE | `auth.uid() = id` | `auth.uid() = id` | ⚠️ AMPLA DEMAIS |
| *(sem política)* | INSERT | — | — | ✅ Bloqueado (default deny) |
| *(sem política)* | DELETE | — | — | ✅ Bloqueado (default deny) |

#### Análise de `clients_update_own`

A política permite que um usuário autenticado faça `UPDATE` em qualquer coluna do seu próprio registro via **Supabase PostgREST** (endpoint `https://{project}.supabase.co/rest/v1/clients`), que está habilitado por padrão em todos os projetos Supabase.

**Colunas sensíveis que podem ser alteradas diretamente:**

| Coluna | Impacto se alterada pelo usuário |
|--------|----------------------------------|
| `plan` | Escalação de privilégios — usuário pode se "promover" de `starter` para `enterprise` sem pagar |
| `is_active` | Pode se desativar (auto-lock) |
| `meta_access_token` | Pode substituir por token inválido, quebrando publicações |
| `brand_profile` | Pode injetar dados maliciosos que alimentam prompts Claude |

**Exemplo de exploit:**
```http
PATCH https://{project}.supabase.co/rest/v1/clients?id=eq.{user_uuid}
Authorization: Bearer {valid_user_jwt}
Content-Type: application/json

{"plan": "enterprise"}
```

**Por que é exploitável mesmo com FORCE RLS:** O `FORCE RLS` força o aplicativo do dono da tabela a respeitar RLS, mas o usuário autenticado com JWT válido pode chamar o PostgREST diretamente — e a política `clients_update_own` autoriza a mudança.

**Por que o backend não está em risco:** O backend usa `SUPABASE_SERVICE_KEY` (service_role), que bypassa RLS por design. O problema é o acesso direto ao PostgREST com chave de usuário.

**Achado:** **HIGH-RLS-001** — Story criada: **SEC-02**

---

### Tabela: `content_requests`

**Migração RLS:** `c5d6e7f8a9b1_enable_rls_on_content_requests.py`

```sql
ALTER TABLE content_requests ENABLE ROW LEVEL SECURITY;
ALTER TABLE content_requests FORCE ROW LEVEL SECURITY;
```

#### Políticas configuradas

| Política | Operação | USING | WITH CHECK | Status |
|----------|----------|-------|------------|--------|
| `content_requests_select_own` | SELECT | `client_id = auth.uid()` | — | ✅ Correta |
| `content_requests_insert_own` | INSERT | — | `client_id = auth.uid()` | ✅ Correta |
| `content_requests_update_own` | UPDATE | `client_id = auth.uid()` | `client_id = auth.uid()` | ⚠️ Ampla |
| `content_requests_delete_own` | DELETE | `client_id = auth.uid()` | — | ✅ Correta |

#### Análise de `content_requests_update_own`

O isolamento multi-tenant está correto: usuário só acessa os próprios registros. Porém, a política não restringe quais colunas podem ser modificadas. Via PostgREST, o usuário pode:

```http
PATCH /rest/v1/content_requests?id=eq.{request_uuid}&client_id=eq.{user_uuid}
{"status": "published", "publish_result": {"instagram_post_id": "fake123"}}
```

**Impacto:** Falsificação de estado do pipeline (marcar post como publicado sem ter publicado), injeção de dados em `analysis_result` ou `copy_result` que alimentam o Claude.

**Achado:** MEDIUM-RLS-001 — Documetado como tech debt.

---

## Verificações Positivas

| Check | Status | Detalhe |
|-------|--------|---------|
| RLS habilitado em `clients` | ✅ PASS | `ENABLE ROW LEVEL SECURITY` |
| RLS habilitado em `content_requests` | ✅ PASS | `ENABLE ROW LEVEL SECURITY` |
| FORCE RLS em ambas as tabelas | ✅ PASS | `FORCE ROW LEVEL SECURITY` |
| Nenhuma tabela sem RLS detectada | ✅ PASS | Apenas 2 tabelas custom; analytics e cerebro usam filesystem |
| Isolamento multi-tenant (SELECT) | ✅ PASS | `auth.uid() = id` e `client_id = auth.uid()` |
| INSERT em `content_requests` restrito ao próprio client_id | ✅ PASS | `WITH CHECK (client_id = auth.uid())` |
| INSERT/DELETE em `clients` bloqueado para usuários | ✅ PASS | Sem política = deny implícito |
| UPDATE em `clients` não permite trocar `id` | ✅ PASS | `WITH CHECK (auth.uid() = id)` previne alteração do id |
| Supabase Auth tables | ✅ N/A | Gerenciadas pelo Supabase internamente |
| Supabase Storage | ✅ N/A | Não utilizado — usando Cloudflare R2 |
| PostgREST role `anon` bloqueado | ✅ PASS | `auth.uid()` retorna NULL para anon — todas as políticas falham |

---

## Achados

### HIGH-RLS-001 — `clients_update_own` permite escalação de plano via PostgREST

**Tabela:** `clients`
**Política:** `clients_update_own`
**OWASP:** A01 — Broken Access Control

Um usuário autenticado pode fazer `PATCH` diretamente no Supabase PostgREST e alterar a coluna `plan` (e outras colunas sensíveis) do próprio registro, sem passar pelo backend.

**Remediação recomendada:** Remover a política `clients_update_own` inteiramente. Todos os updates legítimos de `clients` passam pelo backend com `service_role`, que bypassa RLS. Não há cenário legítimo em que um usuário precise fazer UPDATE diretamente no PostgREST.

**Story criada:** SEC-02

---

### MEDIUM-RLS-001 — `content_requests_update_own` sem restrição de colunas

**Tabela:** `content_requests`
**Política:** `content_requests_update_own`
**OWASP:** A01 — Broken Access Control (menor — afeta apenas dados próprios)

Usuário pode modificar colunas de sistema (`status`, `analysis_result`, `copy_result`, `publish_result`) diretamente via PostgREST, corrompendo o estado do pipeline.

**Remediação (tech debt):** Remover a política `content_requests_update_own` (igual à `clients` — todos os updates passam pelo service_role). OU usar trigger BEFORE UPDATE para bloquear alteração das colunas de sistema.

---

## Ações Tomadas

- [x] Story criada: `docs/stories/epic-security/sec-02.story.md` (HIGH-RLS-001)
- [x] MEDIUM-RLS-001 documentado como tech debt neste relatório
- [x] MEMORY.md atualizado

— Sage, guardando as fronteiras 🛡️
