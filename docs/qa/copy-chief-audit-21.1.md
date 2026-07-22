# Auditoria Científica de Copywriting — Story 21.1

> **Executor:** @analyst (Atlas) — Story 11.1 dynamic executor assignment
> **Data:** 2026-07-08
> **Story:** [21.1.story.md](../stories/epic-21-copy-chief/21.1.story.md) · **Epic:** 21 — Copy-Chief
> **Metodologia:** Hopkins, Schwartz, Sugarman, Bencivenga — fontes primárias documentadas nos Dev Notes da story
> **Escopo:** Auditoria de conteúdo. Nenhum arquivo de produção alterado (AC7, AC8).

---

## 0. Achado prévio — o prompt é mais sofisticado do que a story original presumia

Ao ler `copywriter.py` por completo (não só os trechos citados nos Dev Notes), encontrei um bloco não mapeado anteriormente: **`#DIRETRIZES_ALGORITMICAS — MODO ENGENHARIA`** (linhas 296-307), fruto do Epic 14 (Modo Engenharia). Ele já implementa, sem nomear, boa parte do que Schwartz e Sugarman prescreveriam:

- *"HOOK SEM CONTEXTO... trabalha para um desconhecido total"* → é literalmente a estratégia de copy para o **Nível 1 de consciência (completamente inconsciente)** de Schwartz, embora o prompt não nomeie o framework.
- *"SINAIS QUE ESCALAM... salvamentos... compartilhamentos"* → alinhado com a pesquisa de mercado do Plano Mestre (DM shares pesam 3-5× mais que likes no algoritmo 2026).
- A `_ATTACK_DIRECTIVES` (10 posts, linhas 310-325) já varia o objetivo psicológico por posição — retenção, salvamentos, comentários, qualificação de audiência, replays.

**Implicação para esta auditoria:** não vou tratar o `_SYSTEM_PROMPT` como se estivesse vazio de sofisticação — ele já cobre parte do terreno de Schwartz/Sugarman de forma orgânica. O trabalho da v2 é in **fechar gaps específicos**, não reescrever do zero.

---

## 1. Auditoria Hopkins

**Critérios aplicados** (ver Dev Notes da story): reason-why específico, especificidade vs. generalidade, preemptive claim (mecanismo), testabilidade.

### 1.1 `_SYSTEM_PROMPT` — Score: **76/100** (abaixo do mínimo de 85)

| Critério | Nota | Evidência |
|---|---|---|
| Testabilidade | 9/10 | Regra 1 ("nunca invente dados") é Hopkins quase literal |
| Especificidade | 8/10 | "mencione materiais, técnicas, detalhes do serviço" (linha 273) é reason-why explícito |
| Reason-why nas 3 variações | 6/10 | Regra 3 exige "abordagens completamente diferentes" mas não exige que **cada uma** ancore num fato específico — permite 3 variações de *ângulo* sem 3 razões concretas distintas |
| CTAs (regra 9) | 5/10 | Os 5 exemplos são fórmulas conversacionais fixas ("Me chama no DM...") sem razão anexada — é Sugarman (envolvimento), não Hopkins (nenhum "porque agir agora") |
| Hooks (regra 6) | 6/10 | 5 templates fixos — risco real de virarem fórmulas repetidas em vez de reason-why genuíno por post |
| Preemptive claim / mecanismo | 3/10 | **Não existe regra própria.** O mecanismo específico só aparece nos `_VIRAL_TRIGGERS` por segmento (ex: "material adaptado que superou o padrão") — não é reforçado como princípio geral do `_SYSTEM_PROMPT` |

**Maior gap:** falta uma regra explícita de "preemptive claim" — instruir o modelo a sempre ancorar a legenda num mecanismo específico (o *como*), não em adjetivo de qualidade genérico. Isso é o elo entre Hopkins (especificidade) e a sofisticação de mercado de Schwartz (ver seção 2).

### 1.2 `STRATEGY_PROMPTS` — 23 entradas, Score médio: **79/100**

| Chave | Score | Fraqueza principal |
|---|---|---|
| `feed_photo__prova_social` | 82 | "sem exagerar" é orientação vaga, não regra acionável |
| `feed_photo__ancora_de_marca` | 68 | "reforça identidade" é abstrato — maior risco de fluff genérico do grupo |
| `feed_photo__curiosidade_pergunta` | 80 | Pergunta não é obrigada a citar fato real |
| `feed_photo__bastidores` | 84 | Processo = naturalmente específico |
| `feed_photo__hero_shot` | 74 | "adjetivo de impacto" contraria o princípio anti-superlativo de Hopkins |
| `carousel__antes_depois` | 88 | Contraste é inerentemente testável |
| `carousel__passo_a_passo` | 85 | Promessa numerada = concreta |
| `carousel__erros_mitos` | 79 | Risco de generalização sem evidência ("X erros que...") |
| `carousel__case_estudo` | **90** | Melhor da lista — exige "dados quando disponíveis" |
| `carousel__comparativo` | 76 | Neutro, sem exigência de reason-why |
| `carousel__checklist` | 83 | Utilidade prática = específica por natureza |
| `reels__hook_choque` | 70 | "frase provocativa" sem exigir substância por trás do choque |
| `reels__timelapse` | 87 | "dado concreto" explicitamente exigido |
| `reels__tutorial_pov` | 72 | POV sem exigência de especificidade do conteúdo |
| `reels__trend_nicho` | **65** | Pior da lista — adaptar trend é o maior risco de genérico/copycat |
| `reels__bastidores_autenticos` | 81 | Curiosidade forte, especificidade do processo só implícita |
| `reels__depoimento_video` | 89 | Depoimento = padrão-ouro de prova Hopkins |
| `story__bastidores_dia` | 73 | Limite de 10 palavras impede profundidade |
| `story__enquete` | 77 | Estrutura A/B dá alguma concretude |
| `story__cta_link` | 79 | "urgente e sem rodeios" é urgência fabricada, não genuína |
| `story__countdown` | 75 | Já reconhece o risco ("escassez genuína — não exagere") mas sem mecanismo de verificação |
| `story__caixa_perguntas` | 81 | Convite direto e específico |
| `story__repost_feed` | 71 | O mais raso e de menor valor do conjunto |

**Cinco mais fracas (candidatas a revisão prioritária futura, fora do escopo desta story):** `reels__trend_nicho` (65), `feed_photo__ancora_de_marca` (68), `reels__hook_choque` (70), `story__repost_feed` (71), `reels__tutorial_pov` (72).

---

## 2. Diagnóstico Schwartz — Consciência e Sofisticação

### 2.1 Nível de consciência predominante

Quem vê um post do AutoPost no feed orgânico **não está buscando ativamente** uma empresa de construção civil/arquitetura/paisagismo — está sendo interrompido durante o scroll. Isso posiciona o público majoritariamente entre:

- **Nível 1 (completamente inconsciente)** — nem pensou no problema (ex: reforma futura, projeto de paisagismo)
- **Nível 2 (consciente do problema)** — sabe que precisa reformar/construir, não conhece a empresa

O `_SYSTEM_PROMPT` já reconhece isso implicitamente ("hook trabalha para um desconhecido total") mas **não nomeia a progressão**: o corpo da copy (contexto → processo → resultado) pula direto para a solução sem garantir que o Nível 1 reconheça o problema primeiro.

**Aplicação na v2:** para os formatos que dependem de reconhecimento de problema (`carousel__antes_depois`, `carousel__case_estudo`, `carousel__erros_mitos`), a v2 deve reforçar 1 frase de validação do problema antes de ir para a solução — sem isso, o leitor Nível 1 não se identifica com o post.

*(Este diagnóstico é genérico por nicho, não persistido por cliente — conforme escopo do Epic 21, Modo B fica fora.)*

### 2.2 Estágio de sofisticação de mercado

O nicho de construção civil/arquitetura no Instagram já está saturado de "qualidade", "compromisso" e "excelência" — todo concorrente reivindica os três. Aqui está uma força que **já existe** no código: `_VIRAL_TRIGGERS` evita esses clichês e usa mecanismos específicos ("material adaptado que superou o padrão", "detalhe técnico invisível que faz toda diferença") — isso é precisamente o conceito de "novo mecanismo" de Schwartz, aplicado sem nomeação.

**Gap:** essa disciplina existe só nos `_VIRAL_TRIGGERS` por segmento — não é reforçada como **regra geral** no `_SYSTEM_PROMPT`. Se o segmento do cliente cair no fallback `"default"` (ver `theme_library.py` — nichos fora de construção civil/saúde/advocacia/estética caem em genérico), a disciplina de mecanismo específico se perde.

---

## 3. Mapeamento Sugarman (24 gatilhos documentados na story)

| Presente no `_SYSTEM_PROMPT` atual | Ausente ou fraco |
|---|---|
| Curiosidade (regra 6, hooks) | **Exclusividade/raridade** — nenhuma menção |
| Especificidade (regra 273 + "nunca invente") | **Convicção de satisfação** — sem garantia/social proof numérica |
| Simplicidade (regra 7, ritmo visual) | **Autoridade** — sem instrução para credenciais/tempo de mercado |
| Timing (`suggested_time`) | **Desejo de pertencer** — sem menção de comunidade/coletivo |
| Familiaridade (tom de voz do cliente) | **Justificar a compra** — sem rationalização pós-decisão |
| Senso de urgência (parcial — só em `story__countdown`) | **Urgência genuína no nível do system prompt geral** — ausente fora do Stories |

**Gatilhos a incorporar na v2 (priorizados por relevância ao nicho B2B local):** Autoridade (tempo de mercado, especialização), Prova de valor (reforçar `_ATTACK_GOALS` posição 2 "comentários e shares" com pedido explícito de depoimento), Exclusividade (atendimento local/regional como diferencial, já que a `_VIRAL_TRIGGERS` menciona cidade).

---

## 4. v2 do `_SYSTEM_PROMPT` (proposta — não aplicada nesta story)

Adições propostas às regras existentes (o texto abaixo é o **diff conceitual**; a aplicação literal em `copywriter.py` é escopo da Story 21.2):

```
10. MECANISMO ESPECÍFICO, NUNCA ADJETIVO GENÉRICO (Hopkins/Schwartz):
    Toda legenda deve ancorar o diferencial num MECANISMO concreto — a técnica, o material,
    o processo específico usado — nunca em "qualidade", "compromisso" ou "excelência" isolados.
    Certo: "Argamassa polimérica em vez da tradicional — por isso não trinca no primeiro verão."
    Errado: "Trabalho com qualidade e compromisso." ✗

11. AUTORIDADE SEM AUTOPROMOÇÃO (Sugarman — Estabelecer Autoridade):
    Quando o contexto fornecer tempo de mercado, número de projetos ou especialização,
    mencione UMA vez de forma factual — nunca como alarde ("somos os melhores").
    Certo: "12 anos entregando reforma em Floripa — isso ensina o que não fazer."

12. RECONHECIMENTO DO PROBLEMA ANTES DA SOLUÇÃO (Schwartz — públicos Nível 1-2):
    Para formatos de transformação (antes/depois, case, erros/mitos): a primeira metade do
    corpo deve validar que o leitor reconhece o problema — antes de revelar a solução.
    O leitor não decidiu que precisa de você; primeiro precisa reconhecer que precisa de algo.
```

**Preservado sem alteração** (não duplicar `_VIRAL_TRIGGERS`/`_VOICE_TONE_MAP`, não quebrar contrato JSON, manter as 4 keywords do teste): `hook`, `salvar` (via regra 3 "SINAIS QUE ESCALAM... orientado a salvar"), `desconhecido` (regra 1 "LINGUAGEM PARA DESCONHECIDOS"), `retenção` (regra 2 "RETENÇÃO ACIMA DA MÉDIA") — todas mantidas literalmente no texto original, as adições são regras 10-12 novas.

---

## 5. v2 das estratégias mais usadas — Fascination Bullets (Bencivenga)

Priorizadas conforme Dev Notes: as 5 `feed_photo__*` e as 6 `carousel__*` (formatos mais comuns). Técnica aplicada: gerar curiosidade sobre o benefício **sem entregá-lo por completo** — o leitor precisa continuar (ou salvar/comentar) para "descobrir".

| Chave | Elemento de fascination bullet adicionado |
|---|---|
| `feed_photo__prova_social` | Hook vira pergunta implícita: em vez de só "Entregue. ✓", adicionar "...e não foi do jeito que o cliente esperava" — cria curiosidade sobre o "como" |
| `feed_photo__ancora_de_marca` | Trocar "reforça identidade" por bullet de mecanismo: "1 frase que revela COMO a empresa decide o que aceita fazer — sem dizer o critério completo" |
| `feed_photo__curiosidade_pergunta` | Já é fascination por natureza — reforçar: a pergunta deve ter resposta **parcialmente revelada** no corpo, não totalmente, para sustentar comentários |
| `feed_photo__bastidores` | "Por trás de..." + bullet: prometer 1 detalhe específico do processo sem revelar a técnica completa ("a parte que quase deu errado — e como resolvemos") |
| `feed_photo__hero_shot` | Substituir "adjetivo de impacto" por bullet de mecanismo: nomear o material/técnica sem explicar o processo completo, convidando a perguntar |
| `carousel__antes_depois` | Hook já forte — adicionar bullet no slide intermediário: "o detalhe que quase inviabilizou a obra" (curiosidade sustentada entre slides) |
| `carousel__passo_a_passo` | Bullet no hook: "passo 3 é o que 90% pula — e paga caro depois" (fascination clássica de lista) |
| `carousel__erros_mitos` | Já é fascination por natureza (mitos = curiosidade); reforçar que cada erro deve ter uma consequência específica, não genérica |
| `carousel__case_estudo` | Bullet: "o resultado só apareceu depois de uma mudança que ninguém esperava no meio do projeto" |
| `carousel__comparativo` | Bullet: revelar só 1 critério de comparação no hook, guardar o resto para os slides |
| `carousel__checklist` | Bullet: "item 4 é o que mais gente esquece — e é o mais caro de corrigir depois" |

---

## 6. Confirmações finais

- [x] `_SYSTEM_PROMPT` e as 23 `STRATEGY_PROMPTS` auditados com critérios Hopkins reais (não genéricos) — AC1
- [x] Diagnóstico Schwartz (consciência + sofisticação) aplicado ao nicho, genérico, não persistido por cliente — AC2
- [x] Gatilhos Sugarman mapeados against o `_SYSTEM_PROMPT` atual — AC3
- [x] v2 proposta do `_SYSTEM_PROMPT` entregue (seção 4) — AC4
- [x] v2 das 11 estratégias mais usadas com fascination bullets (seção 5) — AC5
- [x] Lista das 5 estratégias mais fracas documentada, mesmo sem reescrita (seção 1.2) — AC6
- [x] Nenhum arquivo em `backend/app/agents/copywriter.py` foi alterado — AC7
- [x] Nenhuma dependência criada em `.claude/agents/` — AC8

**Score geral da auditoria:** `_SYSTEM_PROMPT` 76/100 + `STRATEGY_PROMPTS` 79/100 (média) = **77.5/100** — confirma que há espaço real de melhoria (mínimo esperado: 85/100). A v2 desta story é o material de entrada para a Story 21.2 aplicar e validar via A/B.

**Próximo passo:** `@po *validate-story-draft` desta entrega (Definition of Done) → se aprovado, Story 21.1 vai a Done → Story 21.2 pode iniciar.
