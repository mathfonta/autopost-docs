# Epic 15 — Estratégias Explicadas: UX de Seleção de Conteúdo

**Status:** Draft
**Data de início:** 2026-06-10
**Prioridade:** Alta
**Origem:** Análise arquitetural @aios-master 2026-06-10 — documento de visão de produto

---

## Objetivo

Tornar cada estratégia de conteúdo legível e decidível para o usuário antes de ele escolher. O motor existe (23 STRATEGY_PROMPTS no copywriter, 23 cards em strategies.ts), mas o usuário não entende o que cada estratégia faz, qual entrada precisa, qual resultado produz.

Esse epic não muda o backend de geração — muda a camada de apresentação.

---

## Problema

Hoje o `SubStrategySelector` mostra: ícone + label + desc (12 palavras) + badge de objetivo.

O usuário de uma empresa de construção civil que nunca usou marketing digital não entende:
- O que é "Âncora de Marca" vs "Prova Social" vs "Hero Shot"
- O que precisa enviar para cada um funcionar
- Como vai ser o resultado antes de commitar

Resultado: escolha aleatória, frustração quando o output não atende a expectativa.

---

## Solução

Enriquecer os metadados de cada estratégia com:
- **Quando usar** — situação ideal de uso (1 linha)
- **O que você precisa** — inputs específicos (foto de resultado? equipe? depoimento?)
- **Exemplo de saída** — frase de exemplo de copy gerada por essa estratégia

Apresentar essas informações no `SubStrategySelector` como card expansível, sem redesenho total da tela.

---

## Entregas

| Story | Título | Esforço |
|-------|--------|---------|
| 15.1 | Metadados Ricos das Estratégias (Frontend Config) | Pequeno |
| 15.2 | Cards de Estratégia Expandidos no SubStrategySelector | Médio |

---

## Dependências de Entrada

- `frontend/lib/strategies.ts` — estrutura atual (id, label, desc, objective, icon)
- `frontend/components/dashboard/SubStrategySelector.tsx` — componente a ser evoluído
- Nenhuma mudança de backend necessária

---

## Definition of Done do Epic

- [ ] Todas as 23 estratégias têm `when_to_use`, `input_needed`, `output_example` preenchidos
- [ ] SubStrategySelector exibe conteúdo expandido de forma acessível
- [ ] Nenhuma regressão no fluxo de seleção existente
- [ ] Deploy Vercel sem downtime
