# Spec — Epic 27: Inteligência de Mercado Multi-Nicho (generalização)

**Fase:** 4-write-spec
**Autor:** @pm (Morgan)
**Data:** 2026-07-26
**Classe:** STANDARD (score reclassificado após split — capacidades novas movidas ao Epic 28)
**Rastreia:** requirements.json · complexity.json · research.json
**Gate constitucional (Artigo IV — No Invention):** toda afirmação abaixo rastreia a um FR/NFR/CON ou achado de research. Sem features inventadas.

---

## 1. Problema

A inteligência de mercado semanal (Exa) só existe para "Construção civil". Três pontos de falha no código (FIND-1/2/3/4 do requirements.json):
1. `generate_weekly_intelligence` é tarefa única e global, queries e `segment` hardcoded (FIND-1).
2. `_CONTENT_TYPE_QUERIES` em `exa_search.py` tem viés de construção civil mesmo aceitando `segment` (FIND-2).
3. `/insights/weekly` lê `business_segment` (coluna inexistente) e sempre cai no fallback global — **bug** (FIND-3).
4. Prompts de resumo e de horário hardcoded para o nicho (FIND-4).

**Resultado:** cliente de outro segmento recebe zero (ou dado errado) de inteligência de mercado.

## 2. Escopo (Epic 27 apenas)

**IN** — FR-1 a FR-6:
- Fan-out do weekly intelligence por segmento distinto ativo (FR-1)
- Geração de queries por segmento (FR-2) e generalização de `_CONTENT_TYPE_QUERIES` (FR-4)
- Prompts de resumo e horário parametrizados por segmento (FR-3)
- Fix do bug do consumidor `/weekly` + normalização de matching (FR-5)
- `suggested_time` calculado por segmento, mantendo comportamento honesto de "sem dados" (FR-6)

**OUT** — movido ao Epic 28:
- Análise de concorrentes (FR-7)
- Re-análise periódica do Scout (FR-8)

**OUT** — geral: taxonomia fechada de segmentos, métricas Meta como fonte, boost pago.

## 3. Decisões de arquitetura adotadas (de complexity.json, validadas em research.json)

| Decisão | Escolha | Rastreia |
|---------|---------|----------|
| Geração de query | Gemini Flash gera 3 queries/segmento (arquétipos: notícia semanal / tendência técnica / mercado-consumo), fallback template | Q1, R-1 |
| Taxonomia | free-text + `_normalize` (lowercase/sem acento) para dedup e matching; sem taxonomia fechada | Q2, R-2, CON-1 |
| Fan-out | dispatcher + subtarefa Celery por segmento (isola falha) | Q3, NFR-2 |
| Horário por nicho | busca por segmento + manter "sem_dados" honesto quando sinal fraco | FR-6, R-3 |

## 4. Stories propostas

### Story 27.1 — Fix do consumidor `/weekly` + normalização de segmento
**Rastreia:** FR-5, NFR-4, NFR-5
**Problema:** `/insights/weekly` lê atributo inexistente `business_segment` e serve dado global a todos.
**AC (esboço):**
1. `/insights/weekly` lê o segmento de `brand_profile['segment']` (não `business_segment`).
2. Matching produtor↔consumidor usa `_normalize` (case/acento-insensível).
3. Cliente sem segmento definido recebe 404 honesto (não dado de outro nicho).
4. Zero regressão para cliente de construção civil (NFR-4).
**Risco:** baixo. **Independente** — pode ir primeiro, entrega correção imediata.

### Story 27.2 — Queries e prompts por segmento (produtor)
**Rastreia:** FR-2, FR-3, FR-4, CON-4
**Escopo:**
- Função que gera 3 queries de tendência a partir do segmento (Gemini Flash + fallback template).
- Parametrizar `_summarize_snippets` e o prompt de extração de horário pelo segmento.
- Generalizar `_CONTENT_TYPE_QUERIES` (`exa_search.py`) para compor a query a partir do `segment` recebido, sem termos fixos de construção civil.
**AC (esboço):**
1. Dado um segmento X, as queries de tendência mencionam X e não termos fixos de outro nicho.
2. Gemini indisponível → fallback template `f"{segment} Brasil notícias semana"` (degradação graciosa, NFR-3).
3. Resumo e extração de horário referenciam o segmento corrente, não "construção civil".
4. `search_exa_trends` para segmento X não injeta mais termos de construção civil.
5. Validado com ≥2 segmentos reais (ex: "Moda e vestuário", "Odontologia") + construção civil (R-1).
**Depende de:** nada (building block do produtor). **Risco:** médio (qualidade de query).

### Story 27.3 — Fan-out do weekly intelligence por segmento
**Rastreia:** FR-1, FR-6, NFR-1, NFR-2, NFR-3
**Escopo:**
- Dispatcher consulta segmentos distintos ativos (dedup via `_normalize`) e despacha 1 subtarefa Celery por segmento.
- Cada subtarefa usa as funções da 27.2, persiste `WeeklyContext` + `suggested_time` por segmento (upsert por (week_of, segment) já existe).
- Falha de um segmento não bloqueia os demais (try/except isolado ou group/chord).
**AC (esboço):**
1. Rodada semanal gera 1 `WeeklyContext` por segmento distinto ativo, não um global.
2. Segmento sem snippets → persiste honestamente (sem summary inventado), como já faz hoje.
3. Falha Exa em um segmento loga WARNING e não afeta os outros (NFR-2).
4. `suggested_time` gravado por segmento (FR-6).
5. Zero regressão: construção civil continua gerando como antes (NFR-4).
6. `EXA_PROVIDER=disabled` → tarefa encerra sem chamadas (NFR-6).
**Depende de:** 27.2. **Risco:** médio (fan-out/custo).

## 5. NFRs (de requirements.json)
NFR-1 custo Exa controlado (validar N real antes de ligar prod) · NFR-2 isolamento de falha por segmento · NFR-3 fallback gracioso · NFR-4 zero regressão construção civil · NFR-5 isolamento multi-tenant por segmento · NFR-6 `EXA_PROVIDER=disabled` default.

## 6. Matriz de rastreabilidade

| FR | Story | | NFR | Story |
|----|-------|---|-----|-------|
| FR-1 | 27.3 | | NFR-1 | 27.3 |
| FR-2 | 27.2 | | NFR-2 | 27.3 |
| FR-3 | 27.2 | | NFR-3 | 27.2, 27.3 |
| FR-4 | 27.2 | | NFR-4 | 27.1, 27.2, 27.3 |
| FR-5 | 27.1 | | NFR-5 | 27.1 |
| FR-6 | 27.3 | | NFR-6 | 27.3 |

Cobertura: FR-1..6 e NFR-1..6 todos cobertos. Sem FR órfão. Sem story sem FR (No Invention ✓).

## 7. Sequenciamento sugerido
27.1 (independente, fix rápido) → 27.2 (building block) → 27.3 (fan-out, depende de 27.2).

## 8. Dependências externas / validações pendentes
- **CON-3:** confirmar N de segmentos ativos em produção (Supabase) antes de habilitar `EXA_PROVIDER=exa` com fan-out.
- Conta/plano Exa: free tier cobre estimado ~10 segmentos; migrar a Starter se N≥20 (complexity.json).
