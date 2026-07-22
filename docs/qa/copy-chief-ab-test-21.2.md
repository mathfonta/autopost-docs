# A/B Test — Story 21.2 (Epic 21 — Copy-Chief)

> **Executor:** @dev (Dex) — Story 21.2
> **Data:** 2026-07-22
> **Método:** 10 cenários sintéticos (analysis_result + brand_profile representativos, sem foto real — o copywriter consome apenas o output estruturado do Analista), gerados 2x cada: 1x com `copywriter.py` em `HEAD` (prompts v1, baseline) e 1x com o working tree atual (prompts v2, Story 21.1). Chamadas reais à API Claude Sonnet (`claude-sonnet-4-6`), sem mock — 20 gerações no total.
> **Cobertura:** as 11 estratégias elevadas na Story 21.1 (5 `feed_photo__*` completas + 6 `carousel__*`, das quais 10 foram sorteadas para o teste, variando segmento: construção civil, arquitetura, paisagismo).
> **Evidência bruta:** outputs completos das 20 gerações preservados no scratchpad da sessão (`ab_test_21_2.py`, `old_results.json`, `new_results.json`) — não versionados no repositório por serem grandes e derivados.

---

## 1. Comparação hook-a-hook (AC6a)

| # | Estratégia | Hook v1 (baseline) | Hook v2 (novo) | Veredito |
|---|-----------|---------------------|------------------|----------|
| 1 | feed_photo·prova_social | "Fachada entregue. Resultado que fala por si. ✓" — genérico | "Entregue. ✓ ...e o cinza-grafite foi a virada que o projeto precisava." — abre um loop de curiosidade | **v2 melhor** — hook v1 é conclusivo/fechado; v2 convida a continuar lendo |
| 2 | feed_photo·ancora_de_marca | "A viga de madeira não estava no projeto original." | "A viga aparente não estava no briefing original." + depois nomeia o critério de decisão da empresa | **v2 levemente melhor** — v2 executa a regra 6 (autoridade sem autopromoção) revelando "como a empresa decide", v1 fica na autodescrição genérica |
| 3 | feed_photo·curiosidade_pergunta | "Você sabe o que acontece com uma fundação sem impermeabilização?" | "Você sabe o que acontece com a fundação antes de cobrir com concreto?" | **Equivalente** — ambos hooks de mesmo padrão e força; v2 não mostra ganho claro aqui |
| 4 | feed_photo·bastidores | "O que ninguém vê antes do jardim ficar pronto" | mesmo hook + "O jardim começa a funcionar antes mesmo de estar visível" | **v2 levemente melhor** — frase adicional ancora num mecanismo concreto (irrigação) em vez de ficar só na promessa de bastidores |
| 5 | feed_photo·hero_shot | "Meses de projeto. Uma fachada que para o trânsito." — adjetivo/hipérbole | "O cobogó não é decoração. É o projeto funcionando." — nomeia o mecanismo (material) explicitamente | **v2 melhor** — é o exemplo mais claro da regra 5 (mecanismo específico, nunca adjetivo genérico) funcionando; v1 usa exatamente o tipo de superlativo que a regra existe para evitar |
| 6 | carousel·antes_depois | Narrativa linear problema→solução, sem reviravolta | Inclui explicitamente "o detalhe que quase atrasou tudo" (irregularidade na alvenaria) no meio da narrativa | **v2 melhor** — fascination bullet da Story 21.1 aparece literalmente no output, sustentando curiosidade entre início e fim |
| 7 | carousel·passo_a_passo | Procedural, sem tensão | "Tem uma etapa no meio do processo que quase ninguém considera... e que, quando pulada, compromete tudo" | **v2 melhor** — fascination bullet do "passo negligenciado" presente e funcional |
| 8 | carousel·erros_mitos | "A laje não infiltrou na chuva. Infiltrou 6 meses depois." — hook forte, reviravolta temporal | "A laje estava impermeabilizada. Mas a infiltração veio mesmo assim." — também forte | **Equivalente, v1 levemente mais afiado** — único caso onde o baseline iguala ou supera o v2 |
| 9 | carousel·case_estudo | Descritivo, sem reviravolta central | "Esse último ponto foi o que mudou o prognóstico do projeto" — reviravolta ancorada na automação da irrigação | **v2 melhor** — fascination bullet presente, ainda que mais sutil que nos outros casos |
| 10 | carousel·comparativo | Entrega os dois critérios completos na legenda | Revela só 1 critério, guarda o resto para "próximos posts" | **v2 melhor** — execução mais literal da instrução v2 (retenção de informação para sustentar curiosidade) |

**Resultado:** 7/10 melhor no v2, 2/10 equivalente, 1/10 baseline levemente superior. Nenhuma regressão real — o pior caso é "empate técnico".

---

## 2. Variação entre as 3 cópias — `caption_long` / `caption_short` / `caption_stories` (AC6b)

A regra 3 do `_SYSTEM_PROMPT` (não alterada por esta story) exige que as 3 variações tenham "hook diferente, estrutura diferente, ângulo diferente" entre si. **Achado honesto:** em ambas as versões (v1 e v2), `caption_short` e `caption_stories` seguem funcionando como *resumos comprimidos* de `caption_long` — mesmo ângulo, não um ângulo genuinamente distinto. Exemplo (#1): as 3 variações v2 repetem "reboco liso, cinza-grafite, portão de ferro preto" nas 3, só com nível de detalhe decrescente.

**Isso não é uma regressão do v2** — o problema já existia no v1 e as regras 5-7 desta story não visavam esse ponto (elas atuam sobre mecanismo/autoridade/reconhecimento-do-problema, não sobre a distinção entre os 3 formatos). Fica registrado como gap pré-existente, fora do escopo desta story — candidato a story futura se o produto priorizar.

---

## 3. Autenticidade / risco de soar "vendedor demais" (AC6c)

Risco mapeado no Epic 21 (tendência 2026 anti-IA-genérica) — avaliado explicitamente em cada um dos 10 pares:

- Nenhum dos 10 CTAs do v2 usa linguagem de venda agressiva — seguem no mesmo registro soft do v1 ("me chama no DM", "conta o seu projeto", "salva esse post").
- A regra 6 (autoridade sem autopromoção) não pôde ser plenamente testada nesta rodada: nenhum dos 10 `brand_profile` sintéticos incluía dado de "tempo de mercado" ou "nº de projetos" — os únicos gatilhos que ativam essa regra no prompt. **Ressalva para a decisão:** validar a regra 6 especificamente quando houver esse dado real disponível (a maioria dos clientes reais do AutoPost tem esse campo preenchido no onboarding).
- Os fascination bullets (retenção parcial de informação — ex. #10) introduzem um risco sutil diferente do "vendedor demais": a copy pode soar *incompleta* se o cliente não perceber que é proposital. Risco baixo, mas vale observar em produção real.

**Conclusão: o risco mapeado no epic não se materializou** — v2 não soa mais artificial ou "forçado" que v1 em nenhum dos 10 casos.

---

## 4. Regressão técnica (AC8)

- `pytest backend/tests/test_copywriter_agent.py backend/tests/test_copywriter_strategy.py -v` → **26/26 passed** (rodado com o v2 aplicado)
- As 4 keywords obrigatórias (`hook`, `salvar`, `desconhecido`, `retenção`) confirmadas presentes via `test_system_prompt_contains_algorithmic_directives`
- Contagem de `STRATEGY_PROMPTS` = 23, distribuição intacta (5/6/6/6) via `test_strategy_prompts_count` e os 4 testes de formato
- Formato JSON de saída (`caption_long`, `caption_short`, `caption_stories`, `hashtags`, `cta`, `suggested_time`) idêntico nas 20 gerações reais — nenhuma quebra de parsing

---

## 5. Decisão (AC7)

**✅ Aprovado para produção.**

**Justificativa:** melhoria mensurável e atribuível em 7 de 10 cenários, concentrada exatamente onde a Story 21.1 identificou os maiores gaps (regra 5 — mecanismo específico vs. adjetivo genérico; fascination bullets nas estratégias de carousel). Nenhuma regressão de teste, nenhum aumento percebido de tom "vendedor", contrato JSON intacto. O único ponto sem melhoria (variação entre os 3 formatos de caption) é um gap pré-existente fora do escopo desta story, não uma regressão introduzida por ela.

**Ressalva registrada para acompanhamento (não bloqueia aprovação):**
1. Regra 6 (autoridade) não testada com dado real de "tempo de mercado" — validar em produção quando esse campo estiver preenchido.
2. Gap de variação entre `caption_long`/`caption_short`/`caption_stories` permanece — candidato a story futura, não desta.

---

## 6. Custo

20 chamadas reais à API Claude Sonnet (`claude-sonnet-4-6`) — custo incorrido na conta do usuário durante este teste. Sem chamadas adicionais além das documentadas aqui.
