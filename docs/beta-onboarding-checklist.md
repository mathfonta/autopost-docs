# ✅ Checklist — Onboarding do 1º Beta Tester (colega da imobiliária)

> **Objetivo:** colocar um usuário real testando o AutoPost para gerar feedback antes de comercializar.
> **Criado:** 2026-07-25 · @aios-master (Orion)
> **Contexto:** app em modo de desenvolvimento na Meta (App Review pendente) → conexão do Instagram exige adicionar o colega como Tester.

---

## PARTE 1 — O que VOCÊ (Matheus) faz antes de enviar

### 1.1 Adicionar o colega como Tester na Meta (destrava a publicação)
Sem isso, o colega consegue usar tudo **menos** conectar o Instagram e publicar.

- [ ] Acesse https://developers.facebook.com → app **AutoPost** (ID: `1438969821361138`)
- [ ] **App Roles → Roles** → adicionar o colega como **Instagram Tester** (pelo @usuário do Instagram dele)
- [ ] Peça para o colega **aceitar o convite**: no Instagram dele → **Configurações → Apps e sites → Convites de testador** → aceitar
- [ ] Confirme que a conta Instagram do colega é **Business ou Creator** (conta pessoal não publica via API). Converter é grátis e leva ~1 min: Instagram → Configurações → Conta → Mudar para conta profissional

### 1.2 Conferir que o app está no ar
- [ ] Abrir `https://autopost.app.br` e checar que carrega (se cair com "tenant not found", o Supabase free pausou — restaurar no dashboard do Supabase)

---

## PARTE 2 — O que ENVIAR para o colega

**Link:** https://autopost.app.br

**Mensagem sugerida (copie e ajuste):**

> Oi! Tô desenvolvendo um app que cria posts de Instagram com IA — você tira a foto do imóvel, ele escreve a legenda, monta o design e sugere a estratégia. Queria muito seu feedback como quem entende do mercado.
>
> Link: https://autopost.app.br
>
> É só criar uma conta e seguir o passo a passo. Importante: pra conectar o Instagram e publicar de verdade, sua conta precisa ser **Business ou Creator** (se não for, dá pra converter grátis em 1 minuto nas configurações do Instagram) — e eu já te adicionei como testador, então vai funcionar.
>
> Testa à vontade e me conta o que achou, o que faltou, o que te confundiu. Qualquer coisa me chama!

---

## PARTE 3 — O que o colega vai encontrar (fluxo esperado)

1. **Cadastro** → cria conta (e-mail/senha)
2. **Onboarding etapa 1 — Perfil da Marca:** dados da imobiliária (nome, nicho, tom de voz)
3. **Onboarding etapa 2 — Conectar Instagram:** OAuth da Meta (só funciona com os passos da Parte 1 feitos)
4. **Onboarding etapa 3 — Concluído:** o **Agente Scout** analisa o perfil dele em background e sugere refinamento de segmento
5. **Dashboard:** enviar foto de um imóvel → IA gera copy + design + estratégia → aprovar/editar → publicar

---

## PARTE 4 — Limitações conhecidas (avise ou espere feedback sobre)

Para o feedback ser útil, saiba o que **ainda não existe** — se ele pedir, já está no radar, não é bug:

| Item | Situação |
|------|----------|
| Agendamento de horário ("postar agora ou agendar de manhã") | 🎯 Na fila de produção (Epic 19) — ainda não construído |
| Card com múltiplas fotos (carrossel) no feed do app | 🔧 Carrossel funciona no envio, mas o card mostra 1 foto (Story 6.3 pendente) |
| Analytics de desempenho do Instagram dentro do app | ⏸️ Desabilitado até a Meta aprovar `instagram_manage_insights` |
| App na Play Store | 💡 Futuro (hoje é web/PWA — funciona no navegador do celular) |

---

## PARTE 5 — Feedback a coletar (roteiro para comercialização)

Perguntas que geram decisão de produto/preço:

**Valor percebido**
- [ ] A copy gerada ficou boa o suficiente para você postar de verdade? O que mudaria?
- [ ] O design/arte serve para a sua imobiliária ou precisaria de ajuste?
- [ ] A estratégia sugerida fez sentido para o seu público?

**Usabilidade**
- [ ] Em que ponto você travou ou ficou em dúvida?
- [ ] Quantos minutos levou do "abrir o app" até "post pronto para publicar"?
- [ ] Faria no celular no dia a dia, ou só no computador?

**Comercial**
- [ ] Você pagaria por isso? Quanto por mês pareceria justo?
- [ ] O que faria você escolher isto em vez de fazer manualmente / contratar um social media?
- [ ] Que 1 funcionalidade, se existisse, te faria assinar na hora?

**Confiabilidade**
- [ ] O app ficou fora do ar ou lento em algum momento?
- [ ] Algum post falhou ao publicar?

---

## PARTE 6 — Registrar o resultado

- [ ] Anotar o feedback no Obsidian (`💡 Ideas` ou `📊 Status do Projeto`)
- [ ] Priorizar melhorias na fila de produção (`docs/SESSAO-HANDOFF.md`)
- [ ] Bugs reais → criar story via `@sm *draft`
