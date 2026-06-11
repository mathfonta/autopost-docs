# Epic 13 — Inteligência Externa via Exa Search

**Status:** Draft  
**Criado:** 2026-05-12  
**Owner:** @pm (Morgan)  
**Origem:** Proposta Claude Web Session → adaptada para codebase real (Epic 8 já ocupado por Copy Quality)

---

## Contexto

O Epic 8 (Qualidade de Copy) equipou o AutoPost com tom de voz, múltiplos formatos e contexto
do usuário. O Epic 12 foca no lançamento. O Epic 10 adicionou transcrição de áudio.

Hoje o pipeline é **fechado**: todo conhecimento vem do que o usuário digita (`user_context`)
ou do que a IA vê/ouve na mídia. Não existe nenhuma fonte de dados **externa ao momento do upload**.

Este epic adiciona o **Exa Search API** como camada de inteligência externa — buscas web
semânticas em tempo real alimentando o pipeline antes da geração de copy.

---

## O que já está implementado (não duplicar)

| Funcionalidade | Onde | Status |
|---------------|------|--------|
| Contexto manual do usuário | `user_context` + `#CONTEXTO_DO_USUARIO` | ✅ Story 8.1 |
| Tom de voz configurável | `voice_tone` + `#TOM_DE_VOZ` | ✅ Story 8.2 |
| 3 variações de copy (long/short/stories) | `caption_long/short/stories` | ✅ Story 8.3 |
| Hashtags geradas pelo LLM | `_VIRAL_TRIGGERS` dict no copywriter | ✅ Epic 8/12 |
| Transcrição de áudio → contexto copy | `audio_transcript` | ✅ Story 12.3 |
| Análise visual de frames (Claude Haiku) | `analyze_video_with_ai()` | ✅ Story 10.1 |
| Provider plugável (COPY_PROVIDER, etc.) | `config.py` | ✅ Story 12.3 |
| Exa MCP configurado no AIOS | `.aios-core/infrastructure/tools/mcp/exa.yaml` | ✅ Infra pronta |
| `httpx` no requirements | já existente | ✅ |

---

## Problema que resolve

| | AutoPost hoje | AutoPost com Exa |
|---|---|---|
| Copy gerada | "Mais uma obra finalizada com qualidade!" | "Revestimento 120x120 — a tendência que chegou nas obras de Floripa." |
| Hashtags | Genéricas: #construção #reforma | Em alta na semana: #tendenciacasa2025 #revestimentomoderno |
| Calendário | Tipos de post fixos | Inclui datas: Dia do Engenheiro, Feirão da Casa Própria |
| Diferencial de venda | "IA gera post a partir da foto" | "IA gera post contextualizado com o mercado atual" |

---

## Arquitetura — Fluxo completo

```
Pipeline ATUAL:
  upload → analyze_photo/video → generate_copy → prepare_design → publish

Pipeline com Exa (Story 13.2+):
  upload → [ analyze_photo/video  ]  (paralelo via asyncio.gather)
          → [ exa_search_trends   ] ↗
          → generate_copy (enriquecido)
          → prepare_design → publish
```

### Decisão arquitetural importante

A integração usa **httpx direto** para a Exa REST API — **não** o MCP server node.
O `.aios-core/infrastructure/tools/mcp/exa.yaml` é para uso no ambiente Claude Code
(sessões de desenvolvimento). No worker Celery Python em produção, é HTTP puro.

### Ponto de injeção no copywriter (mesmo padrão existente)

```python
# copywriter.py — após #CONTEXTO_DO_USUARIO e #TOM_DE_VOZ
if exa_context:
    prompt += f"\n\n#TENDENCIAS_DO_NICHO (dados Exa, últimos 30 dias):\n{exa_context}"
```

---

## Stories do Epic

| Story | Nome | Dependências | Prioridade |
|-------|------|-------------|-----------|
| **13.1** | Infraestrutura Exa no Pipeline | Nenhuma | 🔴 Bloqueante |
| **13.2** | Tendências de Nicho no Copywriter | 13.1 | 🔴 Core |
| **13.3** | Hashtags Inteligentes via Exa | 13.1, 13.2 | 🟡 Alta |
| **13.4** | Weekly Content Intelligence (Celery Beat) | 13.1 | 🟢 Média |

---

## Análise de Custos

| Clientes ativos | Req Exa/mês | Custo Exa | Custo Claude extra | Total adicional |
|----------------|------------|-----------|-------------------|----------------|
| 1 (piloto) | ~35 | $0 (free) | $0.03 | **$0.03/mês** |
| 10 clientes | ~350 | $0 (free) | $0.30 | **$0.30/mês** |
| 50 clientes | ~1.750 | $25 (Starter) | $1.50 | **$26.50/mês** |
| 100 clientes | ~3.500 | $25 (Starter) | $3.00 | **$28.00/mês** |

**Free tier:** 1.000 req/mês → suficiente até ~28 clientes.  
**Starter:** $25/mês → 5.000 req → suficiente até ~140 clientes.  
**Token overhead:** ~300 tokens de contexto extra por post → ~$0.001/post.

Com 100 clientes pagando R$67/mês = R$6.700 de receita, o custo Exa representa < 3%.

---

## Timing recomendado

A Story 13.1 pode ser desenvolvida e testada agora.
O valor real é percebido com posts sendo publicados no Instagram —
timing ideal: **após a aprovação do Meta App Review** (ou em paralelo).

---

## Checklist de pré-implementação

- [ ] Criar conta Exa.ai → exa.ai → signup → free tier disponível imediatamente
- [ ] Obter `EXA_API_KEY`
- [ ] Adicionar `EXA_API_KEY` no Railway (worker + api)
- [ ] Declarar `EXA_PROVIDER=disabled` por default em `config.py`
