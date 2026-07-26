# 🔄 Handoff de Sessão — AutoPost

> **Propósito:** ponto de retomada do projeto. Ao dar `resume` ou `*retomar`, o agente lê ESTE arquivo + o doc de implementação (`docs/plano-mestre-2026-07.md`) para saber onde paramos.
> **Como manter:** este arquivo é sobrescrito ao FECHAR cada sessão de trabalho, refletindo as pendências reais do momento. Não é histórico — é sempre o estado atual.

**Última atualização:** 2026-07-26 · sessão de resposta a incidente pós-Epic 19

---

## 📍 Onde paramos

**Epic 19 (Agendamento Inteligente) está completo e validado em produção de verdade** — fluxo completo de agendar/sugestão/editar/cancelar/reagendar confirmado pelo usuário no celular. Mas o deploy inicial do Epic 19 revelou **dois incidentes de infraestrutura sérios** (não eram bugs de lógica) que quebraram produção por um tempo — ambos diagnosticados e corrigidos nesta sessão. Ver seção "Incidentes desta sessão" abaixo para o histórico completo.

---

## 🔴 Pendências abertas (ordem de prioridade)

### 1. Meta App Review / InstagramAnalyticsCard (bloqueado no usuário)
`InstagramAnalyticsCard` desabilitado em `frontend/app/dashboard/page.tsx`. Gargalo: o usuário ainda precisa **gravar os vídeos de demonstração** exigidos pelo App Review. Não é tarefa de código.

### 2. Beta test com colega da imobiliária
Planejado para a semana de 2026-07-27. Checklist pronto em `docs/beta-onboarding-checklist.md` — passo crítico: adicionar o colega como Tester no painel Meta antes de enviar.

### 3. Backlog de valor (não priorizado)
- **0.3 Analytics PostHog** — só 2/5 eventos emitem (`post_created`, `post_published`); faltam `post_approved`, `upgrade_clicked`, `churn`. (`post_scheduled` foi adicionado no Epic 19.)
- **Epic 25 — Modelos LLM** — código ainda em `claude-sonnet-4-6` / `claude-haiku-4-5`; Sonnet 5 / Opus 4.8 disponíveis.
- **Thumbnail de vídeo no PostCard** — posts de vídeo mostram tela preta no card, sem frame/preview.
- **Epic 23/24 (Score de engajamento, Mix editorial)** — não iniciados.

### 4. 🆕 PLANEJAMENTO FUTURO (demanda grande — não implementar sem passar por @pm/@architect primeiro)

**Inteligência de mercado (Exa/Weekly Intelligence) hardcoded para "construção civil" — não é multi-tenant.**

Investigação disparada por uma dúvida do usuário sobre a sugestão de horário de agendamento. Achados relevantes ao planejar:

- `pipeline.generate_weekly_intelligence` (Celery Beat, toda segunda 07h) tem as queries de busca Exa E o `segment` salvo **hardcoded como "Construção civil"** (`app/tasks/pipeline.py` linhas ~927-975). Para qualquer outro segmento de cliente (ex.: "Moda e vestuário", testado nesta sessão), essa fonte simplesmente **nunca gera dado nenhum** — confirmado: zero linhas em `weekly_context` fora de construção civil.
- O Agente Scout (Epic 22) só sugere mudança de segmento (`suggested_segment`) — nunca aplica automaticamente, por decisão de design (Decisão #2 do Epic 22, correta e intencional). O usuário precisa aceitar explicitamente via `POST /onboarding/scout/accept`.
- Scout roda **uma única vez**, na conexão OAuth — não há re-análise periódica para acompanhar evolução da conta.
- Scout não analisa a distribuição de formato de posts (feed x carrossel x reels x story) do cliente — só conteúdo/estilo.
- Não existe hoje nenhuma análise de contas concorrentes/outros perfis do Instagram do mesmo nicho — a única fonte "externa" é busca web geral via Exa (e só para construção civil).
- As únicas fontes que melhoram com o tempo (`get_strategy_recommendation`, `get_best_posting_time` — fonte "histórico") usam exclusivamente o histórico do próprio cliente, não comparação de mercado.

**Por que é grande:** generalizar a inteligência semanal por segmento (queries dinâmicas + `segment` real por cliente) toca o pipeline de Celery Beat, o schema de `weekly_context`, e potencialmente decisões de custo/rate-limit da Exa (hoje 1 execução/semana fixa vs. N segmentos). Também envolve decidir se vale investir em re-análise periódica do Scout e/ou análise de concorrentes — que é escopo novo, não extensão simples.

**Ação recomendada:** tratar como candidato a epic novo — passar por `@pm *create-epic` com research de `@architect` sobre custo/viabilidade de Exa por segmento antes de detalhar stories.

### 5. Também descoberto nesta sessão (bookkeeping, não bloqueante)
O banco de produção tem **vários clientes de teste acumulados** (`bilhoteiro4`, `Probe User`, `Meta App Reviewer`, múltiplos "Matheus Fontanella Augusto" duplicados, etc. — visto ao investigar o Scout). Não é urgente, mas vale uma limpeza em algum momento antes de escalar para clientes reais.

---

## 🚨 Incidentes desta sessão (contexto para não repetir)

### Incidente 1 — Migração do Epic 19 nunca aplicou em produção (backend)
**Sintoma:** dashboard parou de carregar posts (`GET /content-requests` retornando 500) logo após o "deploy confirmado saudável" do Epic 19.
**Causa raiz:** os dois arquivos de migração novos (`a1b2c3d4e5f6` "scheduling" e `b2c3d4e5f6a7` "suggested_time") reutilizaram **IDs de revisão que já existiam** desde abril/maio (`a1b2c3d4e5f6` = "add caption_edited", `b2c3d4e5f6a7` = "create weekly_context"). Isso criou um ciclo no grafo do Alembic — `alembic upgrade head` falhava silenciosamente em todo deploy, e a coluna `scheduled_for` nunca existiu de fato no banco.
**Fix:** IDs renomeados para `c5581ee0ad14`/`18ec2311ad3b` (únicos), migração aplicada manualmente em produção e verificada. Commit `e5b95a9`.
**Lição:** `/health` não pega esse tipo de falha (não toca a tabela afetada) — confirmar deploy com uma query real, não só o health check, quando a mudança envolve schema.

### Incidente 2 — Frontend do Epic 19 nunca chegou em produção (branch errada)
**Sintoma:** mesmo após o fix do backend, o app continuava sem a seção "Posts agendados" e sem o botão "Agendar".
**Causa raiz:** a branch local `main` do repo `autopost-frontend` estava com upstream configurado para `origin/master` (branch secundária que o Vercel não observa). Todos os pushes desta sessão foram para `master`; `origin/main` (o branch padrão do GitHub, servido pelo Vercel) ficou 2 commits atrás.
**Fix:** fast-forward de `origin/main` para o commit correto (`f335e6e`) + correção do tracking local (`git branch --set-upstream-to=origin/main main`) para não repetir.
**Lição:** ao pushar, confirmar `git branch -vv` de vez em quando neste repo específico — o mismatch pode voltar se alguém clonar de novo sem configurar o upstream certo.

### Incidente 3 — Post travado em "publishing" (efeito colateral do redeploy)
Um post ficou preso em `status=publishing` porque o worker Celery foi reiniciado (pelo push do Incidente 1) no meio da tarefa de publicação. Resetado manualmente para `awaiting_approval` via SQL direto; usuário reaprovou e publicou normalmente. Sem ação pendente — mas serve de alerta: **pushes no backend reiniciam o worker e podem interromper publicações em andamento.**

### Incidente 4 (menor) — Horário sugerido de agendamento sem base real
Usuário notou que a modal de agendar sugeria um horário específico mesmo sem ter posts publicados com métricas nem pesquisa Exa para o segmento. Investigado e confirmado: o valor vinha da própria IA de copywriting "inventando" um horário plausível (campo `copy_result.suggested_time`), apresentado sem aviso de que não era uma análise real. Corrigido: `GET /insights/best-posting-time` agora retorna `fonte="sem_dados"` com mensagem honesta quando não há histórico nem Exa; a modal só mostra "Sugerido: X" quando a fonte é real. Commits `ab27738` (backend) + `1c0d728` (frontend), já em produção.

---

## ✅ Estado dos repositórios (2026-07-26, fim de sessão)

| Repo | Branch | Estado |
|------|--------|--------|
| autopost-docs (docs) | master | pendente commit deste handoff |
| autopost-backend (backend) | main | ✅ sincronizado com origin/main — `ab27738`, deploy saudável |
| autopost-frontend (frontend) | main | ✅ sincronizado com origin/main (corrigido do mismatch) — `1c0d728`, deploy saudável e validado pelo usuário |

## 📦 Epic 19 — status final

| Story | Status | Gate |
|---|---|---|
| 19.1 (modelo + endpoint agendar) | Done | PASS (96) |
| 19.2 (executor Beat + cancelar/reagendar) | Done | PASS (95) |
| 19.3 (sugestão de horário) | Done | PASS (96) — refinado nesta sessão (fonte="sem_dados") |
| 19.4 (frontend — botão Agendar) | Done | PASS (95) — validado em produção real |
| 19.5 (frontend — tela de agendados) | Done | PASS (96) — validado em produção real |

**Próxima decisão de prioridade fica com o usuário.** Candidatos: item 4 acima (planejamento da inteligência de mercado multi-nicho, escopo grande) depois do beta test, ou seguir com backlog de valor (item 3).
