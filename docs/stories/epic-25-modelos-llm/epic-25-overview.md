# Epic 25 — Atualização de Modelos LLM (Claude)

**Status:** Concluído (1/1 stories Done, gate PASS)
**Início:** 2026-07-26
**Concluído:** 2026-07-26
**Tipo:** Manutenção técnica (débito registrado no backlog de valor)
**Depende de:** Nenhuma dependência funcional — troca de configuração

---

## Visão

O backend referencia model IDs da geração anterior da família Claude (`claude-sonnet-4-6`) em
7 arquivos. A Anthropic disponibilizou a geração seguinte (Sonnet 5 / Opus 4.8 / Haiku 4.5).
Este epic atualiza as referências para os IDs vigentes, preservando o tier de modelo
originalmente escolhido por `@architect` em cada agente (não é uma revisão de arquitetura de
roteamento de modelo — ver [[adr_llm_model_routing]] na memória do projeto para o racional de
qual agente usa qual tier).

## Escopo

**IN:**
- Atualizar `claude-sonnet-4-6` → `claude-sonnet-5` em todos os usos no backend
- Confirmar que usos de `claude-haiku-4-5-20251001` já estão no ID vigente (não requer troca)

**OUT:**
- Rediscutir qual agente deveria usar qual tier de modelo (isso seria uma decisão de
  `@architect`, fora do escopo desta troca mecânica)
- Trocar para Opus 4.8 em qualquer agente — não há justificativa de escopo levantada para isso

## Stories

| Story | Título | Pontos | Prioridade | Status |
|-------|--------|--------|------------|--------|
| 25.1 | Atualizar model IDs de Sonnet 4.6 para Sonnet 5 no backend | 1 | 🟡 Should Have | Done (gate PASS 100) |

## Arquivos afetados (confirmado via grep)

- `backend/app/cerebro/promoter.py:18`
- `backend/app/cerebro/analyzer.py:16`
- `backend/app/agents/onboarding.py:21`
- `backend/app/agents/scout.py:35`
- `backend/app/agents/copywriter.py:24`
