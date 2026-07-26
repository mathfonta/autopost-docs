# Epic 27 — Inteligência de Mercado Multi-Nicho

**Status:** Concluído (3/3 stories Done)
**Criado:** 2026-07-26
**Owner:** @pm (Morgan) · Arquitetura: @architect (Aria)
**Origem:** Backlog de valor (item 5) — generalização do Epic 13 (Exa Intelligence), cuja Story 13.4 nasceu hardcoded para "Construção civil".
**Classe de complexidade:** STANDARD (após split — capacidades novas movidas ao Epic 28)

---

## Problema

A inteligência de mercado semanal (Exa) só existe para "Construção civil". Três pontos de falha no código: tarefa Beat única e global com queries/segmento hardcoded ([pipeline.py:938](../../../backend/app/tasks/pipeline.py)), viés de nicho nos templates de `exa_search.py`, e **bug** no consumidor `/insights/weekly` que lê uma coluna inexistente (`business_segment`) e serve dado global a todos. Cliente de outro segmento recebe zero ou dado errado.

## Escopo (Epic 27)

**IN (FR-1..6):** fan-out do weekly intelligence por segmento distinto ativo; queries e prompts parametrizados por segmento; generalização de `_CONTENT_TYPE_QUERIES`; fix do bug `/weekly` + matching normalizado; `suggested_time` por segmento com "sem dados" honesto.

**OUT → Epic 28:** análise de concorrentes (FR-7) e re-análise periódica do Scout (FR-8).

## Stories propostas

| Story | Título | Rastreia | Risco | Depende | Status |
|-------|--------|----------|-------|---------|--------|
| 27.1 | Fix do consumidor `/weekly` + normalização | FR-5, NFR-4/5 | Baixo | — | Done (gate PASS 100) |
| 27.2 | Queries e prompts por segmento (produtor) | FR-2/3/4 | Médio | — | Done (gate PASS 100) |
| 27.3 | Fan-out do weekly intelligence por segmento | FR-1/6, NFR-1/2/3 | Médio | 27.2 | Done (gate PASS 95) |

**Sequência:** 27.1 → 27.2 → 27.3.

## Decisões de arquitetura (travadas)

- **Geração de query:** Gemini Flash gera 3 queries/segmento (fallback template).
- **Taxonomia:** free-text + `_normalize` (sem taxonomia fechada na v1).
- **Fan-out:** dispatcher + subtarefa Celery por segmento (isola falha).
- **Gravação do segmento (CRIT-1):** grava bruto para exibição, deduplica/casa por normalizado.
- **Guardrail de custo (CRIT-2):** `MAX_SEGMENTS_PER_RUN` + log de N.

## Viabilidade Exa

**VIÁVEL.** Fan-out por segmento distinto (não por cliente) mantém N pequeno. Estimativa: ~10 segmentos ≈ dentro do free tier; ~20 → Starter ($25/mês). Custo <3% da receita.

## Validações pendentes antes de produção

- Confirmar N real de segmentos ativos (Supabase) — CON-3.
- Confirmar plano Exa e setar `MAX_SEGMENTS_PER_RUN` no Railway.

## Artefatos do Spec Pipeline (6 fases)

`spec/requirements.json` · `spec/complexity.json` · `spec/research.json` · `spec/spec.md` · `spec/critique.json` (APPROVED 4.4) · `spec/implementation.yaml`

## Próximo passo

Epic 27 concluído. O valor central está entregue: cada segmento ativo de cliente gera sua própria inteligência de mercado semanal (tendências, hashtags, horário sugerido), em vez de todos receberem dado de "Construção civil".

**Antes de habilitar `EXA_PROVIDER=exa` com fan-out em produção (gate operacional, @devops):**
- Confirmar N real de segmentos ativos (CON-3, pendente desde o planejamento).
- Setar `MAX_SEGMENTS_PER_RUN` explicitamente no Railway.

**Próximo epic:** Epic 28 (análise de concorrentes + re-análise periódica do Scout — FR-7/FR-8) precisa de spec pipeline próprio.
