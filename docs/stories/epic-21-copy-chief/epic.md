# Epic 21 — Copy-Chief: Elevação Científica dos Prompts do Copywriter

> Status: Draft
> Criado por: @pm (Morgan) — 2026-07-07
> Origem: Plano Mestre (`docs/plano-mestre-2026-07.md`, Fase 1) + ideia registrada no Obsidian em 2026-06-17 (`ideia-copy-chief-elevacao-prompts.md`)
> Escopo decidido com o usuário: **Modo A apenas** (prompt engineering imediato, sem mudança de schema/onboarding)

---

## Epic Goal

Elevar a qualidade científica dos prompts do worker de produção `copywriter.py` aplicando a metodologia Schwartz/Hopkins/Sugarman/Bencivenga (pesquisada diretamente das fontes primárias — ver nota de mudança de abordagem abaixo) numa sessão Claude Code — sem alterar a infraestrutura de produção — para que os hooks sejam calibrados por nível de consciência do público e as 3 variações de copy sejam vozes de fato distintas, não o mesmo ângulo reformulado.

> **Mudança de abordagem (2026-07-07, validação @po):** o plano original previa portar o agente `copy-chief.md` de Obras SaaS. Validação da Story 21.1 encontrou que esse arquivo é um *spawner* cujas 4 dependências de metodologia (persona real, workflow de auditoria, checklist Hopkins, lista Sugarman) não existem como artefatos portáveis — nem na origem. Decisão do usuário: aplicar a metodologia diretamente, com pesquisa das fontes primárias documentada na própria Story 21.1, sem instanciar um agente wrapper. Ver Change Log.

## Epic Description

### Existing System Context

- **Funcionalidade atual:** `backend/app/agents/copywriter.py` (Claude Sonnet) gera legenda, hashtags e CTA a partir da análise de foto. Tem infraestrutura sólida: 23 combinações formato+estratégia (`STRATEGY_PROMPTS`), 5 tipos de conteúdo (`CONTENT_TYPE_PROMPTS`), 5 intenções de marketing (`INTENT_CONTEXT_PROMPTS`), 3 abordagens de retry forçadas (`_RETRY_APPROACHES`), cache de contexto Exa, integração com o Cérebro (`read_patterns()`) e `_SYSTEM_PROMPT` único.
- **O que falta:** nenhuma auditoria de qualidade antes da entrega e nenhum diagnóstico de nível de consciência do público — os prompts são tecnicamente corretos mas não calibrados cientificamente. As 3 variações de copy hoje usam a mesma voz com ângulos diferentes.
- **Tecnologia:** Claude Sonnet (`claude-sonnet-4-6`), prompts Python hardcoded (strings/dicts no próprio arquivo), sem camada de templates externa.
- **Pontos de integração:** `_SYSTEM_PROMPT`, `STRATEGY_PROMPTS`, `CONTENT_TYPE_PROMPTS` — consumidos por `generate_copy_with_ai()` e pela task Celery `pipeline.generate_copy`.

### Enhancement Details

- **O que está sendo adicionado:** um ciclo de auditoria e reescrita de prompts, aplicado diretamente pelo agente executor (`@analyst`) numa sessão Claude Code dentro do próprio AutoPost, usando a metodologia documentada na Story 21.1 (fontes primárias pesquisadas: Hopkins, Schwartz, Sugarman, Bencivenga). Produz uma v2 do `_SYSTEM_PROMPT` com gatilhos psicológicos de Sugarman embutidos, e eleva as entradas de `STRATEGY_PROMPTS` com fascination bullets (Bencivenga).
- **Como integra:** a auditoria **não roda em produção** — é feita uma vez (ou sob demanda) numa sessão de desenvolvimento para editar o arquivo `copywriter.py`. O worker Railway continua rodando exatamente como hoje, sem nova dependência, sem novo custo por geração.
- **Critério de sucesso:** prompts elevados aplicados, validados por teste A/B controlado (10 posts antes/depois) antes de substituir os prompts em produção, com resultado registrado no Segundo Cérebro.

### Fora de escopo (explicitamente adiado)

- **Modo B** (diagnóstico de consciência por cliente no onboarding, campos `awareness_level`/`sophistication` no `brand_profile`) — requer mudança de schema e ~2h de dev adicional; fica para um epic futuro se o Modo A validar a hipótese.
- **Modo C** (auditoria mensal automatizada via cron) — depende de volume de posts em produção; avaliar pós-lançamento.

---

## Stories

### Story 21.1 — Auditar os prompts atuais com metodologia científica de copywriting

**Descrição:** Aplicar diretamente a metodologia Hopkins/Schwartz/Sugarman/Bencivenga (pesquisada das fontes primárias, documentada na própria story) sobre `copywriter.py`, com foco em `_SYSTEM_PROMPT` e nas 23 entradas de `STRATEGY_PROMPTS`. **Não** envolve portar um agente externo — validação encontrou que o wrapper `copy-chief.md` de Obras SaaS não tem suas dependências de metodologia disponíveis como arquivos portáveis.

**Entregáveis:**
- Relatório de auditoria Hopkins (score atual de cada prompt, mínimo esperado 85/100)
- Diagnóstico de nível de consciência/sofisticação aplicável ao público de construção civil/arquitetura (mesmo sem persistir por cliente — Modo B fica fora)
- v2 proposta do `_SYSTEM_PROMPT` com os gatilhos Sugarman mapeados
- v2 proposta de pelo menos as estratégias mais usadas de `STRATEGY_PROMPTS` com fascination bullets (Bencivenga)
- Lista do que está fraco nas 23 estratégias atuais (documento, não necessariamente reescrita de todas)

**Executor Assignment:** `executor: @analyst` (auditoria/pesquisa aplicada de copy), `quality_gate: @pm`
**Quality Gate Tools:** `[content_review, methodology_check]`
**Quality Gates:**
- Pre-Commit: nenhum código de produção alterado nesta story (só o novo arquivo de agente + relatório)
- Pre-PR: n/a (deliverable é documento + agente, não precisa de PR de código)

**Foco:** rigor da auditoria (Hopkins ≥85/100), cobertura Sugarman (≥80%), evidência de que os prompts atuais realmente carecem de calibração — não aceitar auditoria genérica.

---

### Story 21.2 — Aplicar prompts elevados com validação A/B

**Descrição:** A partir dos artefatos da Story 21.1, aplicar a v2 do `_SYSTEM_PROMPT` e das entradas elevadas de `STRATEGY_PROMPTS` em `copywriter.py`. Rodar teste A/B controlado: gerar 10 posts com os prompts antigos e 10 com os novos (mesmas fotos/contextos de entrada, para comparação justa), avaliar qualidade percebida do hook e variação real entre as 3 cópias. Se aprovado, substituir em produção; documentar o resultado.

**Entregáveis:**
- `copywriter.py` atualizado com `_SYSTEM_PROMPT` v2 e `STRATEGY_PROMPTS` elevados
- Registro comparativo do A/B (antes/depois) — pode ser um markdown simples em `docs/qa/` ou anexo à story
- Entrada no Segundo Cérebro (`*cerebro-update`) com o resultado e o aprendizado (ex.: "prompts com N Sugarman triggers embutidos aumentaram variação percebida entre cópias")
- Testes existentes de `test_copywriter_agent.py` e `test_copywriter_strategy.py` continuam passando

**Executor Assignment:** `executor: @dev`, `quality_gate: @architect`
**Quality Gate Tools:** `[code_review, regression_test, prompt_snapshot_diff]`
**Quality Gates:**
- Pre-Commit: rodar suite de testes do copywriter (`pytest backend/tests/test_copywriter_agent.py backend/tests/test_copywriter_strategy.py`)
- Pre-PR: revisão de diff do prompt (comparação lado a lado v1/v2), confirmar que nenhuma lógica de parsing/JSON foi quebrada

**Foco:** mudança é só nos textos de prompt — nenhuma alteração de contrato de API, schema de resposta ou lógica de retry. Regressão zero é o critério não-negociável.

---

## Compatibility Requirements

- [x] Nenhuma API muda de contrato — `generate_copy_with_ai()` mantém assinatura e formato de retorno
- [x] Nenhuma mudança de schema de banco (Modo B explicitamente fora de escopo)
- [x] Task Celery `pipeline.generate_copy` não muda de nome nem de comportamento externo
- [x] Custo por geração permanece o mesmo (mesmo modelo, prompts mais longos mas dentro de `MAX_TOKENS` já configurado — validar em 21.2)

## Risk Mitigation

- **Risco primário:** prompts "elevados" soarem mais artificiais/vendedores, contrariando a tendência de 2026 de autenticidade anti-IA-genérica (ver Plano Mestre, seção 3.2) — o público de construção civil pode rejeitar copy que pareça "forçada".
- **Mitigação:** o teste A/B da Story 21.2 avalia explicitamente isso; auditoria da 21.1 deve equilibrar gatilhos psicológicos com o tom autêntico já validado no produto (não é para maximizar "venda agressiva", é para calibrar consciência + variação real).
- **Plano de rollback:** `_SYSTEM_PROMPT` e `STRATEGY_PROMPTS` v1 ficam preservados no histórico git — reverter é um `git revert` do commit da 21.2, sem migração de dados envolvida.

## Definition of Done

- [ ] Stories 21.1 e 21.2 completas com acceptance criteria atendidos *(21.1 ✅ Done 2026-07-08 · 21.2 pendente)*
- [ ] Testes de copywriter (existentes) passando sem regressão *(escopo da 21.2)*
- [ ] A/B documentado com decisão explícita (aprovar prompts v2 ou não) *(escopo da 21.2)*
- [x] Resultado registrado no Segundo Cérebro (`APRENDIZADOS.md`) — aprendizado da auditoria registrado 2026-07-08; o resultado do A/B (21.2) será acrescentado ao final
- [x] Relatório de metodologia (Story 21.1) documentado em `docs/qa/copy-chief-audit-21.1.md` como referência reutilizável para auditorias futuras

---

## Validation Checklist (auto-aplicado pelo @pm)

**Escopo:**
- [x] Epic completável em 1-3 stories (2 stories)
- [x] Nenhuma documentação de arquitetura formal necessária
- [x] Segue padrões existentes do projeto (prompts continuam em Python por ora — a externalização é Epic 22.2, fora deste epic)
- [x] Complexidade de integração mínima (mudança isolada em `copywriter.py`)

**Risco:**
- [x] Risco ao sistema existente é baixo (mudança só de texto de prompt, sem lógica)
- [x] Rollback viável (git revert)
- [x] Estratégia de teste cobre funcionalidade existente (suites já existentes + A/B novo)

**Completude:**
- [x] Objetivo do epic é claro e alcançável
- [x] Stories bem escopadas com executor/quality_gate definidos
- [x] Critérios de sucesso mensuráveis (Hopkins ≥85/100, Sugarman ≥80%, A/B decidido)
- [x] Dependências identificadas (nenhuma bloqueante — não depende de Meta Review nem de outros epics)

---

## Story Manager Handoff

Please develop detailed user stories for this brownfield epic. Key considerations:

- Esta é uma melhoria de qualidade sobre um sistema existente rodando FastAPI + Celery + Claude Sonnet em produção (Railway)
- Pontos de integração: `backend/app/agents/copywriter.py` (`_SYSTEM_PROMPT`, `STRATEGY_PROMPTS`, `CONTENT_TYPE_PROMPTS`)
- Padrões existentes a seguir: prompts continuam como strings/dicts Python nesta iteração (externalização é assunto do Epic 22, não deste epic); nomenclatura de testes segue `test_copywriter_*.py`
- Requisito crítico de compatibilidade: zero mudança de contrato de API/schema; regressão zero nos testes existentes
- Cada story deve incluir verificação de que a funcionalidade existente permanece intacta (rodar `test_copywriter_agent.py` e `test_copywriter_strategy.py`)
- Story 21.1 é majoritariamente um exercício de auditoria/conteúdo (executor `@analyst`) — não gera PR de código de produção
- Story 21.2 é a única que toca `copywriter.py` em produção — é onde o rigor de regressão importa

O epic deve manter a integridade do sistema entregando prompts calibrados cientificamente, com evidência de melhoria via A/B antes de qualquer substituição definitiva em produção.

---

## Change Log

| Data | Autor | Mudança |
|------|-------|---------|
| 2026-07-07 | @pm (Morgan) | Epic criado a partir do Plano Mestre + ideia do Obsidian (2026-06-17). Escopo confirmado com o usuário: Modo A apenas. |
| 2026-07-07 | @po (Pax) | Validação da Story 21.1 encontrou achado crítico: dependências de metodologia do wrapper `copy-chief.md` não existem em lugar nenhum, nem na origem (Obras SaaS). Epic e Story 21.1 atualizados para aplicar a metodologia diretamente, com pesquisa das fontes primárias documentada na story. |
| 2026-07-08 | @po (Pax) | Story 21.1 fechada (Done). Auditoria entregue: `_SYSTEM_PROMPT` 76/100, `STRATEGY_PROMPTS` média 79/100, v2 + fascination bullets para 11 estratégias. DoD parcial do epic atualizado. Próxima: Story 21.2 (Ready, bloqueio de sequência liberado). |
