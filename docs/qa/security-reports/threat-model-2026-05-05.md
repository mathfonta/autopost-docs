# Threat Model — AutoPost (STRIDE)
**Data:** 2026-05-05
**Agente:** @security (Sage)
**Metodologia:** STRIDE
**Escopo:** Sistema completo — todos os fluxos de dados

---

## Mapa do Sistema e Fronteiras de Confiança

```
                     ┌─────────────────────────────────────────────────────────┐
                     │                    INTERNET                              │
                     └──────────┬─────────────────────────┬────────────────────┘
                                │                         │
                    ┌───────────▼──────────┐   ┌──────────▼──────────┐
                    │   Frontend (Vercel)   │   │   Meta Graph API    │
                    └───────────┬──────────┘   └──────────┬──────────┘
                                │ HTTPS                   │ callback
              ══════════════════╪════B2═══════════════════╪════════════
                                │                         │
                    ┌───────────▼─────────────────────────▼──────────┐
                    │              Backend API (Railway)               │
                    │        FastAPI + Uvicorn — service_role          │
                    └──────┬─────────┬──────────┬──────────┬──────────┘
                           │         │          │          │
              ═════════════╪══B3═════╪══B4══════╪══B5══════╪══B6══════
                           │         │          │          │
               ┌───────────▼──┐  ┌───▼────┐ ┌──▼──────┐ ┌▼────────┐
               │   Supabase   │  │   R2   │ │ Claude  │ │  Meta   │
               │  Auth + DB   │  │   CF   │ │ Haiku/  │ │ Graph   │
               │  PostgREST◄──┼──┼────────┼─┼─────────┼─┼──── B10 ┤◄── Browser!
               └──────────────┘  └───┬────┘ │ Sonnet  │ │ API     │
                                     │      └─────────┘ └─────────┘
              ═══════════════════════╪═══B7════════════════════════════
                                     │ (presigned URLs)
                    ┌────────────────▼───────────────────────────────┐
                    │         Redis (Railway — privado)               │
                    │    Celery Broker + Backend + Push Subs          │
                    └────────────────┬───────────────────────────────┘
                                     │
              ═══════════════════════╪═══B8════════════════════════════
                                     │
                    ┌────────────────▼───────────────────────────────┐
                    │        Celery Worker (Railway)                   │
                    │  NullPool — acessa DB com service_role           │
                    │  Acessa Meta tokens de TODOS os clientes         │
                    └────────────────────────────────────────────────┘
```

**Fronteiras críticas:**
- **B2** — Frontend ↔ Backend: HTTPS, JWT obrigatório (exceto auth endpoints)
- **B3** — Backend ↔ Supabase: service_role bypassa RLS; conexão PostgreSQL direta
- **B7** — Backend ↔ Redis: rede interna Railway (não exposta à internet)
- **B8** — Celery ↔ DB: service_role, sem isolamento por tenant
- **B10** — Browser ↔ Supabase PostgREST: **ROTA DIRETA NÃO INTENCIONAL** — usuário com JWT válido acessa banco sem passar pelo backend

---

## Ativos de Valor (Assets)

| Asset | Onde está | Impacto se comprometido |
|-------|-----------|------------------------|
| Meta access tokens (60d) | Coluna `clients.meta_access_token` + Redis (task results) | Publicação não autorizada em Instagram/Facebook de todos os clientes |
| Anthropic API key | Railway env vars | Custo ilimitado + acesso ao Claude |
| JWT_SECRET | Railway env vars | Forge de tokens — impersonar qualquer usuário |
| SUPABASE_SERVICE_KEY | Railway env vars | Acesso irrestrito a todos os dados do banco |
| Cloudflare R2 keys | Railway env vars | Acesso a todas as fotos de todos os clientes |
| Brand profiles | Coluna `clients.brand_profile` JSONB | Dados estratégicos de empresas clientes |
| Conteúdo gerado (copy, design) | Colunas JSONB em `content_requests` | Dados privados de campanhas dos clientes |

---

## Análise STRIDE

---

### S — Spoofing (Falsificação de Identidade)

#### S-01 — Meta OAuth callback sem autenticação do chamador (MEDIUM)

**Fluxo:** Browser → Meta → `POST /meta/callback`
**Ameaça:** O endpoint `/meta/callback` não tem autenticação (correto por design OAuth). Um atacante que obtém um OAuth code válido (via phishing da URL de autorização) pode completar o fluxo e vincular a conta Meta ao perfil errado.

**Mitigações existentes:**
- ✅ State JWT assinado com `JWT_SECRET` — previne CSRF
- ✅ Meta valida codes server-side (single-use)
- ✅ HTTPS previne intercepção em trânsito

**Mitigação ausente:**
- ❌ O `state` é passado como query param no redirect — fica no URL, pode estar em logs de proxy/CDN

**Risco residual:** Baixo — requer engenharia social avançada. Documentado como LOW-TM-001.

---

#### S-02 — Celery task injection via Redis (LOW)

**Fluxo:** Redis → Celery Worker
**Ameaça:** As tasks Celery são enfileiradas no Redis sem assinatura HMAC. Um acesso ao Redis (mesmo que interno) permite injetar tasks falsas com `request_id` arbitrário.

**Mitigações existentes:**
- ✅ Redis está na rede privada Railway — não exposto à internet
- ✅ Tasks usam `request_id` como UUID — não adivinháveis
- ✅ `accept_content=["json"]` — sem desserialização pickle (protege contra RCE)

**Risco residual:** Muito baixo. Documentado como LOW.

---

### T — Tampering (Adulteração de Dados)

#### T-01 — Celery messages sem integridade criptográfica (MEDIUM)

**Fluxo:** Backend API → Redis Broker → Celery Worker
**Ameaça:** O payload JSON das tasks (contendo `request_id`) não tem HMAC ou assinatura. Comprometimento do Redis permite alterar tasks em trânsito.

**Mitigações existentes:**
- ✅ Rede privada Railway — acesso físico limitado
- ✅ `task_acks_late=True` — task permanece na fila até conclusão, reduz janela de ataque

**Remediação disponível:** Celery suporta `task_always_eager=False` + broker transport options para signing. Mas só vale se Redis for exposto.

**Risco residual:** Baixo dado isolamento de rede. MEDIUM se Railway tiver brecha de rede compartilhada.

---

#### T-02 — `publish_post` não é idempotente com `task_acks_late=True` (MEDIUM — Integridade de Negócio)

**Fluxo:** Celery Worker → Meta Graph API
**Ameaça:** `task_acks_late=True` faz a task só ser reconhecida (ACK) ao Redis após conclusão. Se o worker crasha APÓS publicar no Instagram mas ANTES do ACK, o Redis re-enfileira a task. Na próxima execução, `publish_post` publica o mesmo conteúdo uma segunda vez.

**Evidência:**
```python
celery_app.conf.update(
    task_acks_late=True,  # ack só após conclusão
)

@celery_app.task(bind=True, name="pipeline.publish_post", max_retries=3)
def publish_post(self, request_id: str):
    # sem verificação de "já foi publicado antes de tentar novamente"
```

**Impacto:** Cliente vê post duplicado no Instagram — dano à reputação, experiência degradada.

**Remediação:** Adicionar verificação idempotente no início de `publish_post`:
```python
req = _run_sync(_get_request(request_id))
if req.status == ContentStatus.published:
    logger.info(f"[publish_post] já publicado — idempotência ativada")
    return request_id
```

**Classificação:** Não é ameaça de segurança — é bug de integridade de negócio. Repasse ao **@dev** para correção urgente (pré-lançamento).

---

### R — Repudiation (Negação / Rastreabilidade)

#### R-01 — Sem audit trail imutável de publicações (MEDIUM)

**Ameaça:** O resultado de publicação (`publish_result`) é armazenado em JSONB na tabela `content_requests` — mutável via API backend. Se um post é publicado e depois deletado do Instagram, não há registro independente que comprove que a publicação ocorreu, quando, e com qual conteúdo.

**Impacto:** Impossível provar para um cliente que o post foi publicado se houver disputa.

**Remediação (tech debt):** Log de auditoria append-only para eventos de publicação (PostHog já está integrado via `analytics_track` — usar para registrar `post_published` com permalink e timestamp).

**Positivo:** `analytics_track` em `publish_post` já registra o evento — mas PostHog é externo e pode ser desabilitado se `POSTHOG_API_KEY` estiver vazio.

---

#### R-02 — Eventos de segurança sem log estruturado (LOW)

Auth failures, acessos negados (403), tentativas de escalação não geram eventos de segurança rastreáveis além dos logs de aplicação padrão.

---

### I — Information Disclosure (Exposição de Informação)

#### I-01 — Meta tokens no contexto de tasks Celery falhas (MEDIUM)

**Fluxo:** Celery Worker → Redis Backend (result storage)
**Ameaça:** `backend=settings.REDIS_URL` — o Celery armazena os resultados (inclusive exceções) no Redis. O dict `req` em `publish_post` contém `meta_access_token`. Se uma exceção inesperada inclui o dict (ex: `TypeError: unsupported type for req`), o token vai para o Redis result storage em texto puro.

**Mitigação parcial existente:**
```python
# publisher.py — docstring explícita:
# RESTRIÇÕES: Nunca expor access_token em logs
```
O `MetaAPIError` captura apenas a mensagem de erro, não o request. Mas exceções de bibliotecas (httpx, asyncio) podem incluir contexto.

**Remediação (tech debt):** Mascarar o token antes de logar: `req["meta_access_token"] = "***"` após uso no publish_post, ou não incluir o token no dict `req` — buscá-lo separadamente apenas no momento de uso.

---

#### I-02 — Supabase PostgREST como vetor de exfiltração direta (HIGH — já capturado)

**Referência:** HIGH-RLS-001 (SEC-02) — usuário lê/modifica dados diretamente via PostgREST sem passar pelo backend. Repete aqui porque é a ameaça arquitetural mais ampla: **o backend não é a única porta de entrada ao banco**.

---

#### I-03 — R2 presigned URLs em respostas da API (LOW)

O `GET /content-requests` retorna `design_result.r2_key` e URLs presigned com `expires_in=3600`. Qualquer pessoa que obtenha a resposta da API (incluindo via CORS com credenciais — ver SEC-01) tem 1 hora de acesso às imagens processadas sem autenticação.

---

### D — Denial of Service (Negação de Serviço)

#### D-01 — Sem isolamento de fila por tenant (MEDIUM)

**Ameaça:** Um cliente que envia 50 fotos simultâneas ocupa os workers Celery, atrasando o processamento de todos os outros clientes. Não há separação de filas por `client_id`.

**Impacto:** Em ambiente multi-tenant com muitos clientes, um cliente "pesado" degrada a experiência de todos.

**Remediação (tech debt):** Routing de tasks por fila dedicada por cliente ou limitação de tasks simultâneas por `client_id`. O rate limit de 10/hora no upload mitiga parcialmente.

---

#### D-02 — Falha do Celery Beat = tokens Meta não renovados (MEDIUM)

**Fluxo:** `renew_meta_tokens` (Celery Beat — diário 07h)
**Ameaça:** Se o Celery Beat process falha ou o worker não está rodando no horário agendado, tokens que expiram naquele dia não são renovados. Todas as publicações desses clientes começam a falhar com "token expirado".

**Mitigações existentes:**
- ✅ Token renovado quando falta ≤10 dias para expirar (janela ampla)
- ✅ `max_retries=1` na task de renovação
- ✅ Falha no token de um cliente não afeta os outros (loop independente por cliente)

**Mitigação ausente:** Nenhum alerta/notificação quando renovação falha para um cliente específico.

---

#### D-03 — Sem circuit breaker para Claude API (LOW)

**Ameaça:** Se a Anthropic API entrar em outage, todos os requests ficam presos em status `analyzing` ou `copy` até esgotar os 2 retries (2×30s = 60s). Nenhum fallback ou mensagem proativa ao usuário.

**Mitigação parcial:** `task_soft_time_limit=300` e `task_time_limit=600` matam tasks que travam por mais de 5-10 min.

---

### E — Elevation of Privilege (Escalação de Privilégio)

#### E-01 — Escalação de plano via PostgREST (HIGH — SEC-02 já criada)

Usuário autenticado faz PATCH direto no Supabase PostgREST e se autopromove de `starter` para `enterprise`.

---

#### E-02 — CORS regex → requisições autenticadas cross-origin (HIGH — SEC-01 já criada)

App maliciosa em `*.vercel.app` faz chamadas autenticadas à API em nome de usuários logados.

---

#### E-03 — Celery worker com acesso irrestrito a todos os tenants (MEDIUM)

**Ameaça:** O Celery worker usa `service_role` — a mesma chave que acessa TODOS os dados de TODOS os clientes. Se o worker for comprometido (ex: vulnerabilidade em dependência, RCE via deserialização), o atacante tem acesso a tokens Meta, brand_profiles, e conteúdo de todos os clientes simultaneamente.

**Mitigações existentes:**
- ✅ `accept_content=["json"]` — sem pickle (protege contra RCE por deserialização)
- ✅ Railway isola o container do worker
- ✅ Nenhuma entrada direta do usuário chega ao worker (apenas `request_id` via Redis)

**Mitigação ausente:** Não há isolamento de credenciais por tenant no worker — seria necessário arquitetura de vault por cliente.

**Risco residual:** Baixo dado isolamento de container, mas é o maior "raio de explosão" do sistema.

---

## Resumo de Achados

### Novos achados do threat model

| ID | STRIDE | Severidade | Descrição | Ação |
|----|--------|-----------|-----------|------|
| T-02 | Tampering | **BUG CRÍTICO** | `publish_post` não idempotente + `task_acks_late=True` → posts duplicados após crash de worker | @dev — fix urgente pré-lançamento |
| I-01 | Information | MEDIUM | Meta tokens no contexto de tasks Celery falhas → Redis result storage | Tech debt |
| D-01 | DoS | MEDIUM | Sem isolamento de fila por tenant → starvation | Tech debt |
| D-02 | DoS | MEDIUM | Celery Beat sem alerta de falha de renovação de token | Tech debt |
| S-01 | Spoofing | LOW | State JWT no URL do OAuth → logs de proxy | Tech debt |
| D-03 | DoS | LOW | Sem circuit breaker para Claude API | Tech debt |
| I-03 | Information | LOW | R2 presigned URLs com 1h de acesso sem auth | Aceitável |

### Achados anteriores confirmados pelo threat model

| ID | Severidade | Story |
|----|-----------|-------|
| E-01 / HIGH-RLS-001 | HIGH | SEC-02 |
| E-02 / HIGH-001 | HIGH | SEC-01 |
| E-03 | MEDIUM | — |
| MEDIUM-003 (meta token plaintext) | MEDIUM | — |

---

## Vetor de Ataque Mais Provável (Attack Path)

```
1. Atacante cria conta legítima no AutoPost
2. Obtém JWT válido via login normal
3. Faz PATCH direto ao Supabase PostgREST → altera plan para "enterprise"        [SEC-02]
4. OU: hospeda página em *.vercel.app com JS malicioso                            [SEC-01]
5. Usuário legítimo acessa a página → JS faz chamadas autenticadas à API
6. Extrai dados do cliente via GET /content-requests
```

**Vetores bloqueados após SEC-01 + SEC-02:** Os dois ataques mais viáveis são eliminados com as duas stories prontas.

---

## Ações Tomadas

- [x] Threat model documentado
- [x] T-02 (publish_post não idempotente) identificado como BUG crítico pré-lançamento → repassar ao @dev
- [x] Nenhuma nova story de segurança HIGH criada (todos os HIGHs já em SEC-01 e SEC-02)
- [x] MEDIUMs documentados como tech debt
- [x] MEMORY.md atualizado

— Sage, guardando as fronteiras 🛡️
