# Epic 19 — Agendamento Inteligente

**Status:** Draft — epic estruturado pelo @pm; stories a serem criadas pelo @sm e validadas pelo @po
**Data:** 2026-07-25
**Prioridade:** Alta (fila de produção — aprovado pelo usuário)
**Origem:** Visão recorrente do usuário ("criei o post à noite, mas o ideal é postar de manhã") + especificado no plano mestre (`docs/plano-mestre-2026-07.md`, Epic 19) e no Obsidian Roadmap (`🗺️ Roadmap — Ideias e Pendências.md`)
**Decisões de produto travadas com o usuário (2026-07-25):** (1) sugestão de horário **editável** — o sistema sugere, mas o usuário pode escolher outro; (2) **incluir** tela de gerenciamento dos agendados (cancelar/reagendar); (3) publicação **automática** no horário (o usuário não precisa estar presente).

---

## Objetivo

Na tela de aprovação, além de "Publicar agora", o cliente ganha a opção **"📅 Agendar"** com o **melhor horário sugerido a partir de dados reais** — e o post é publicado sozinho nesse horário, sem o usuário precisar estar presente.

O cenário-alvo: o cliente monta o post à noite, mas o ideal para a construção civil é postar de manhã ou no início da noite. Em vez de publicar na hora errada (ou ter que lembrar de voltar ao app no horário certo), ele agenda em um toque — com o sistema já sugerindo *quando*.

---

## Problema

Hoje o fluxo de aprovação (`POST /content-requests/{id}/approve` em `app/api/content.py:420`) só tem um caminho: aprovar → `publish_post.delay()` → publica **imediatamente**. Não existe:

- Nenhuma forma de adiar a publicação para um horário melhor.
- Nenhuma inteligência sobre *qual* é o melhor horário — mesmo o AutoPost já tendo o dado (histórico de posts × horário × engajamento) e já rodando busca semanal de tendências do nicho via Exa.
- Nenhuma visão do que está na fila para sair.

Consequência: o cliente publica no horário em que *conseguiu montar* o post (muitas vezes à noite, fora do horário de pico do nicho), desperdiçando alcance orgânico — ou simplesmente não posta com regularidade porque depende de estar disponível no momento certo.

---

## Solução

Introduzir um estado de **agendamento** no ciclo de vida do post e um **executor periódico** que publica no horário marcado, com **sugestão de horário orientada a dados**.

```
Tela de aprovação
  ├── "Publicar agora"  → fluxo atual (status approved → publish_post.delay) [inalterado]
  └── "📅 Agendar"      → mostra horário sugerido (editável)
                           ↓
                     status scheduled + scheduled_for (timezone-aware)
                           ↓
        Celery Beat "publish-scheduled" (a cada minuto)
          → busca posts scheduled cujo scheduled_for já chegou
          → transição atômica scheduled → publishing (idempotente, sem publicar 2x)
          → publish_post.delay(id)  [reusa a task de publicação existente]
                           ↓
                     published (fluxo de publicação atual, intacto)

Sugestão de horário (GET /insights/best-posting-time):
  1. Histórico do próprio cliente (posts publicados × horário × engajamento) — dado já existe
  2. [FUTURO] Meta insights/audience_online — quando o App Review aprovar (NÃO bloqueia o epic)
  3. Exa semanal — melhores horários do nicho (já roda toda segunda)
  4. Fallback estático: 9h / 12h / 19h (construção civil)
```

**Princípios de design:**

- **Reuso máximo (REUSE > ADAPT > CREATE):** a publicação em si não muda — o Beat só chama a `publish_post` que já existe. O agendamento é uma **porta de entrada alternativa** para o mesmo executor.
- **Caminho feliz atual intacto:** "Publicar agora" continua idêntico. O agendamento é aditivo.
- **Não depende da Meta:** as fontes 1, 3 e 4 de horário já existem hoje. A fonte 2 (Meta `audience_online`) é um upgrade futuro, encaixado por trás da mesma interface — o epic **não fica bloqueado** pelo App Review.
- **Publicação idempotente:** o Beat roda a cada minuto; a transição `scheduled → publishing` precisa ser atômica para nunca publicar o mesmo post duas vezes se duas execuções coincidirem.
- **Timezone-aware:** o Celery já está em `America/Sao_Paulo` (`app/tasks/__init__.py`); `scheduled_for` deve ser comparado corretamente ao "agora" nesse fuso.
- **Degradação graciosa da sugestão:** cliente com poucos posts publicados (ex.: < 5) não tem histórico estatisticamente útil → cai direto no fallback estático, sem fingir precisão.

---

## Entregas

| Story | Título | Esforço | Camada |
|-------|--------|---------|--------|
| 19.1 | Modelo de agendamento: `scheduled_for` + status `scheduled` + endpoint de agendar | Médio | Backend + DB |
| 19.2 | Executor Celery Beat (publica no horário) + cancelar/reagendar | Médio | Backend |
| 19.3 | Sugestão de melhor horário (`GET /insights/best-posting-time`) | Médio | Backend |
| 19.4 | Frontend — botão "Agendar" na aprovação com horário sugerido editável | Médio | Frontend |
| 19.5 | Frontend — tela "Posts agendados" (listar, cancelar, reagendar) | Médio | Frontend |

**Ordem/dependências:** 19.1 é base para 19.2 e 19.5. 19.3 é independente (pode ser paralela). 19.4 depende de 19.1 (endpoint de agendar) + 19.3 (horário sugerido). 19.5 depende de 19.1/19.2 (endpoints de cancelar/reagendar).

---

## Arquitetura

### Mudanças no modelo (`app/models/content_request.py`)
- Novo valor no enum `ContentStatus`: `scheduled = "scheduled"` (entre `approved` e `publishing` no ciclo de vida).
- Novo campo `scheduled_for: Mapped[datetime | None]` (timezone-aware) — quando o post deve ser publicado.
- Migração Alembic correspondente.

### Backend — agendamento (Story 19.1)
- Endpoint para agendar um post que está `awaiting_approval` — decisão de design a travar: novo `POST /content-requests/{id}/schedule` (body: `scheduled_for`) **ou** parâmetro opcional no `/approve` existente. Ver "Decisões a travar".
- Valida que `scheduled_for` é futuro e dentro de um limite razoável (ex.: até 30 dias).

### Backend — executor (Story 19.2)
- Nova task Celery `publish_scheduled_posts` + entrada no `beat_schedule` de `app/tasks/__init__.py` rodando **a cada minuto** (`crontab(minute="*")`).
- A cada tick: `SELECT ... WHERE status = 'scheduled' AND scheduled_for <= now()`, transição atômica `scheduled → publishing` (com lock otimista/`UPDATE ... WHERE status='scheduled'` para idempotência) e `publish_post.delay(id)`.
- Endpoints de `cancelar` (volta a `awaiting_approval` ou vira `rejected`?) e `reagendar` (novo `scheduled_for`). Comportamento exato a travar.

### Backend — sugestão de horário (Story 19.3)
- `GET /insights/best-posting-time?format={content_type}` → `{ horario, fonte, confianca }`.
- Cascata de fontes conforme "Solução". Reusa o dado de histórico já persistido e o resultado da inteligência semanal Exa (`generate_weekly_intelligence`, já no beat).

### Frontend (Stories 19.4 e 19.5)
- 19.4: na tela de aprovação (`ApprovalButtons` / `ApprovalScreen`), botão secundário "📅 Agendar" que exibe o horário sugerido + razão inline e um seletor editável de data/hora.
- 19.5: seção/tela "Posts agendados" listando os `scheduled` com ação de cancelar e reagendar.

### Reuso (REUSE > ADAPT > CREATE)
- **Publicação:** `publish_post` (task Celery existente) é reusada sem alteração — o agendamento só muda *quando* ela é chamada.
- **Beat:** `app/tasks/__init__.py` já tem `beat_schedule` configurado (weekly-intelligence, keepalive) e timezone `America/Sao_Paulo` — só adicionar uma entrada.
- **Histórico e Exa:** o dado de engajamento por horário e a inteligência semanal do nicho já existem — a Story 19.3 consome, não recria.

### Sem mudanças em
- A task `publish_post` e todo o pipeline de publicação (analyze/copy/design/publish).
- O caminho "Publicar agora" da tela de aprovação.
- Escopos OAuth.

---

## Decisões que precisam ser travadas antes da implementação

1. **[Story 19.1] Endpoint de agendar:** novo `POST /{id}/schedule` (separa a intenção de "publicar já" da de "agendar") **ou** parâmetro opcional `scheduled_for` no `/approve` existente. Recomendação do @pm: endpoint separado (intenções distintas, resposta e status diferentes) — confirmar com @architect na Story 19.1.
2. **[Story 19.2] Idempotência do Beat:** garantir que dois ticks concorrentes nunca publiquem o mesmo post duas vezes — transição `scheduled → publishing` via `UPDATE ... WHERE status='scheduled' RETURNING` (lock otimista) antes de enfileirar. @architect valida a abordagem.
3. **[Story 19.2] Semântica de "cancelar":** um agendamento cancelado volta para `awaiting_approval` (usuário pode reaprovar/reagendar) ou vira `rejected`? Travar antes de codar o frontend (19.5).
4. **[Story 19.3] Limiar de histórico:** com quantos posts publicados a sugestão passa a ser "data-driven" vs cair no fallback estático (ex.: < 5 → fallback). Definir o número.
5. **[Story 19.3] Meta como fonte futura:** confirmar que a v1 NÃO depende de `audience_online` (bloqueado por App Review) — a interface `{horario, fonte, confianca}` deixa o encaixe pronto para quando aprovar, sem bloquear.

---

## Escopo

### IN
- Estado de agendamento no post (`scheduled` + `scheduled_for`).
- Agendar um post aprovado para um horário futuro (sugerido, editável).
- Publicação automática no horário via Celery Beat (idempotente).
- Sugestão de melhor horário a partir de histórico próprio + Exa + fallback.
- Tela de gerenciamento dos posts agendados (listar, cancelar, reagendar).

### OUT (futuro / outros epics)
- **Agendamento em lote / semana inteira** — é o Epic 20 (Biblioteca de Fotos), que depende deste.
- **Recorrência** ("todo dia às 19h") — v1 é agendamento pontual por post.
- **Notificação push/email** para o usuário — a publicação é automática, não "avise-me para confirmar".
- **Meta `audience_online` como fonte de horário** — futuro, pós-aprovação do App Review; a interface já deixa o encaixe pronto.
- **Score de engajamento pré-publicação** — é o Epic 23 (pode compartilhar o endpoint de insights com este epic no futuro).

---

## Definition of Done do Epic

- [ ] Na tela de aprovação, "📅 Agendar" mostra um horário sugerido com razão inline e permite editar.
- [ ] Agendar um post o coloca em `scheduled` com `scheduled_for`, sem publicar na hora.
- [ ] No horário marcado, o post é publicado automaticamente (Celery Beat), exatamente uma vez.
- [ ] O usuário consegue ver os posts agendados e cancelar/reagendar.
- [ ] A sugestão de horário usa histórico real quando disponível e cai para fallback estático quando não há dado suficiente.
- [ ] "Publicar agora" continua funcionando idêntico (sem regressão).
- [ ] Testes cobrindo: agendamento, publicação no horário, idempotência (não publica 2x), cancelar/reagendar, fallback de horário.

---

## Riscos

| Risco | Mitigação |
|-------|-----------|
| Beat publicar o mesmo post 2x (ticks concorrentes) | Transição atômica `scheduled → publishing` com lock otimista antes de enfileirar (Story 19.2) |
| Confusão de timezone (agenda às 19h mas publica em UTC) | Celery já em `America/Sao_Paulo`; `scheduled_for` timezone-aware; testes explícitos de fuso |
| Sugestão de horário imprecisa com pouco histórico | Limiar mínimo de posts → fallback estático; comunicar como "sugestão", não garantia |
| Post agendado enquanto o Supabase free está pausado | Beat de 1 min também mantém o banco acordado (efeito colateral positivo); keepalive (Story 1.5) já mitiga |
| Beat de 1 minuto adiciona carga contínua | Query simples e indexada (`status`, `scheduled_for`); custo desprezível |

---

## Dependências

- **Fluxo de aprovação/publicação atual** (`/approve`, `publish_post`) — reusado, não alterado.
- **Celery Beat** (`app/tasks/__init__.py`) — já configurado, só estender.
- **Histórico de engajamento por horário** e **inteligência semanal Exa** — já existem, consumidos pela Story 19.3.
- **Não depende** da aprovação do Meta App Review (fontes 1/3/4 de horário já disponíveis).

---

## Architecture Review (Aria, 2026-07-25)

Revisão técnica das 5 decisões sinalizadas, ancorada no código real. **Complexity class: STANDARD** (score ~15/25 — scope 4, integration 2, infra 3, knowledge 2, risk 4; o risco é alto porque a publicação automática toca o caminho de dinheiro — publicar de verdade no Instagram de um cliente).

### Decisão 1 [19.1] — Endpoint de agendar: **endpoint separado** `POST /content-requests/{id}/schedule`
Resolvido: criar endpoint dedicado, **não** parametrizar o `/approve`. Justificativa:
- Intenção e resposta distintas (`/schedule` retorna `scheduled` + `scheduled_for`; `/approve` retorna `publishing`).
- **Zero risco de regressão** no caminho crítico "Publicar agora" — o `/approve` (`app/api/content.py:420`) fica intocado.
- Fronteiras de story mais limpas.
- Body: `{ "scheduled_for": "<ISO8601 com timezone>" }`. Validações: status atual deve ser `awaiting_approval`; `scheduled_for` deve ser futuro e ≤ 30 dias à frente. Seta `status=scheduled` + `scheduled_for`, e **não** chama `publish_post`.

### Decisão 2 [19.2] — Idempotência do Beat: **claim atômico por UPDATE...RETURNING**
Resolvido: o Beat NÃO faz select-depois-update em dois passos. Faz **um UPDATE atômico que reivindica as linhas**:
```sql
UPDATE content_requests
   SET status = 'publishing'
 WHERE status = 'scheduled' AND scheduled_for <= now()
RETURNING id;
```
Para cada `id` retornado → `publish_post.delay(id)`. Como o UPDATE row-level do Postgres é atômico, **dois ticks concorrentes do Beat nunca reivindicam a mesma linha** — só um a pega no `RETURNING`. Isso resolve a duplicação na raiz.
- Camadas de defesa redundantes (aceitáveis): `publish_post` já seta `publishing` (agora redundante, inofensivo) e já tem a guarda `if status==published: return`.
- Consequência conhecida (não-nova): se `publish_post` falhar antes de publicar, o post fica em `publishing` — exatamente o mesmo comportamento do fluxo `/approve` atual, sem risco novo.
- **Índice necessário:** `(status, scheduled_for)` para o Beat de 1 min não fazer full scan → tarefa do @data-engineer na Story 19.1.

### Decisão 3 [19.2] — Cancelar: **volta para `awaiting_approval`** (não `rejected`)
Resolvido: cancelar um agendamento devolve o post para `awaiting_approval`. Justificativa: o usuário **já aprovou o conteúdo** — só quer mudar o *quando*. Cancelar não deve descartar o post; deve devolvê-lo à tela de aprovação para ele publicar agora, reagendar ou rejeitar explicitamente. `rejected` continua sendo a ação separada de "descartar" (semântica preservada).
- **Reagendar** = `UPDATE scheduled_for` mantendo `status=scheduled` (sem troca de status).

### Decisão 4 [19.3] — Limiar de histórico: **≥ 5 posts publicados com métricas**
Resolvido: com < 5 posts publicados (com `metrics` em `publish_result`), a sugestão cai direto no fallback. Com ≥ 5: agrupar por hora-do-dia do `published_at`, escolher a hora de melhor engajamento médio (likes+comments+reach); se a hora vencedora tiver < 2 posts, desconfiar e usar fallback. v1 mantém simples.
- **⚠️ Ancoragem de dados (crítico para o @dev):** o "histórico próprio" vem de **`ContentRequest.publish_result`** (que tem `published_at` — ver `pipeline.py:631` — e `metrics`), **NÃO** dos arquivos markdown do cérebro (`app/cerebro/writer.py`, que são contexto para LLM, não analytics queryável). A Story 19.3 agrega as linhas de `ContentRequest` publicadas do cliente.

### Decisão 5 [19.3] — Meta como fonte futura: **confirmado, v1 não depende da Meta**
Resolvido: o endpoint retorna `{ horario, fonte, confianca }` com `fonte ∈ {historico, exa, fallback}`. Quando o App Review aprovar, adiciona-se `fonte: "meta_audience"` como prioridade máxima **atrás da mesma interface** — zero rework nos consumidores (frontend 19.4).

### Achado de reuso (REUSE > CREATE) — impacta a Story 19.4
O **copywriter já gera `copy_result.suggested_time`** por post (`app/agents/copywriter.py:745`, com `_DEFAULT_TIMES` por segmento como fallback). Portanto:
- A Story 19.4 pode usar `copy_result.suggested_time` como **default imediato** no primeiro render (sem chamada de API), e a Story 19.3 fornece a versão **refinada por dados reais** quando o usuário abre o agendamento.
- Isso reduz acoplamento e dá um default gracioso mesmo se o endpoint de insights falhar.

### Delegações
- **@data-engineer:** migração Alembic (novo valor de enum `scheduled` + coluna `scheduled_for` timezone-aware) e o índice `(status, scheduled_for)` — Story 19.1.
- **@ux-design-expert (opcional):** o seletor de data/hora e a tela de agendados (19.4/19.5) podem passar por revisão de UX se o usuário quiser.

### Veredito
Arquitetura **aprovada** para prosseguir. As 5 stories estão bem delimitadas; nenhuma reescreve o pipeline de publicação. Próximo: `@sm *draft` para criar as 5 stories incorporando estas resoluções.

---

## Change Log

| Data | Autor | Ação |
|------|-------|------|
| 2026-07-25 | @pm (Morgan) | Epic estruturado a partir do plano mestre + Obsidian Roadmap; 3 decisões de produto travadas com o usuário (horário editável, tela de gerenciamento incluída, publicação automática). Próximo: @sm cria as 5 stories, @po valida. |
| 2026-07-25 | @architect (Aria) | Architecture Review: 5 decisões resolvidas (endpoint separado, claim atômico UPDATE...RETURNING para idempotência, cancelar→awaiting_approval, limiar ≥5 posts, Meta como fonte futura). Achado de reuso: copywriter já gera suggested_time. Ancoragem de dados: histórico vem de publish_result, não do cérebro markdown. Complexity: STANDARD. Aprovado para @sm. |
