# 🔄 Handoff de Sessão — AutoPost

> **Propósito:** ponto de retomada do projeto. Ao dar `resume` ou `*retomar`, o agente lê ESTE arquivo + o doc de implementação (`docs/plano-mestre-2026-07.md`) para saber onde paramos.
> **Como manter:** este arquivo é sobrescrito ao FECHAR cada sessão de trabalho, refletindo as pendências reais do momento. Não é histórico — é sempre o estado atual.

**Última atualização:** 2026-07-26 · @devops (Gage)

---

## 📍 Onde paramos

**Epic 19 (Agendamento Inteligente) está completo: 5/5 stories Done, todos os gates PASS.** Cliente agora consegue agendar um post na aprovação (com horário sugerido por dados reais), ver a fila de agendados no dashboard, cancelar e reagendar. Deploy em produção confirmado saudável (`/health`: api/database/redis ok) após os dois blocos de push desta sessão.

---

## 🔴 Pendências abertas

### 1. Story 6.3 — checagem visual pendente (MANUAL-001, não bloqueante)
PostCard multi-foto tem gate PASS, mas nunca foi visto rodando em navegador real (só validado por tsc/build). Recomendado conferir antes do beta test.

### 2. Story 19.4/19.5 — checagem visual pendente (não bloqueante)
Mesma categoria: fluxo de agendar/cancelar/reagendar tem gate PASS por leitura de código + typecheck/build, mas nunca foi visto rodando de verdade. Recomendado conferir antes do beta test: agendar um post, ver a sugestão de horário, cancelar, reagendar.

### 3. Meta App Review / InstagramAnalyticsCard (bloqueado no usuário)
`InstagramAnalyticsCard` desabilitado em `frontend/app/dashboard/page.tsx:445`. Gargalo: o usuário ainda precisa **gravar os vídeos de demonstração** exigidos pelo App Review. Não é tarefa de código.

### 4. Beta test com colega da imobiliária
Planejado para a semana de 2026-07-27. Checklist pronto em `docs/beta-onboarding-checklist.md` — passo crítico: adicionar o colega como Tester no painel Meta antes de enviar. **Agora com Epic 19 pronto, vale considerar mostrar o agendamento a ele também.**

### 5. Backlog de valor (não priorizado)
- **0.3 Analytics PostHog** — só 2/5 eventos emitem (`post_created`, `post_published`); faltam `post_approved`, `upgrade_clicked`, `churn`. (Nota: `post_scheduled` foi adicionado nesta sessão, Story 19.4.)
- **Epic 25 — Modelos LLM** — código ainda em `claude-sonnet-4-6` / `claude-haiku-4-5`; Sonnet 5 / Opus 4.8 disponíveis.
- **Thumbnail de vídeo no PostCard** — posts de vídeo mostram tela preta no card, sem frame/preview. Precisa extrair frame (ffmpeg) + salvar R2 + expor no schema — vira story própria.
- **Epic 23/24 (Score de engajamento, Mix editorial)** — mencionados no plano mestre, não iniciados. Naturais candidatos a próximo epic, já que compartilham a mesma fonte de dados de `publish_result.metrics` usada pela Story 19.3.

---

## ✅ Estado dos repositórios (2026-07-26)

| Repo | Branch | Estado |
|------|--------|--------|
| autopost-docs (docs) | master | ✅ sincronizado com origin |
| autopost-backend (backend) | main | ✅ sincronizado com origin — deploy confirmado saudável |
| autopost-frontend (frontend) | master | ✅ sincronizado com origin — deploy confirmado saudável |

## 📦 Epic 19 — status final

| Story | Status | Gate |
|---|---|---|
| 19.1 (modelo + endpoint agendar) | Done | PASS (96) |
| 19.2 (executor Beat + cancelar/reagendar) | Done | PASS (95) |
| 19.3 (sugestão de horário) | Done | PASS (96, após fix de comparação de segmento) |
| 19.4 (frontend — botão Agendar) | Done | PASS (95) |
| 19.5 (frontend — tela de agendados) | Done | PASS (96, após fix de item stale na UI) |

**Nenhum epic novo na fila de produção no momento.** Próxima decisão de prioridade fica com o usuário — candidatos naturais: checagem visual das stories pendentes de teste manual (item 1/2 acima), ou um epic novo (23/24, ver backlog).
