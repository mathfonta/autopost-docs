# 🔄 Handoff de Sessão — AutoPost

> **Propósito:** ponto de retomada do projeto. Ao dar `resume` ou `*retomar`, o agente lê ESTE arquivo + o doc de implementação (`docs/plano-mestre-2026-07.md`) para saber onde paramos.
> **Como manter:** este arquivo é sobrescrito ao FECHAR cada sessão de trabalho, refletindo as pendências reais do momento. Não é histórico — é sempre o estado atual.

**Última atualização:** 2026-07-25 · @devops (Gage)

---

## 📍 Onde paramos

Sessão de 2026-07-25 fechou a **Story 6.3** (PostCard multi-foto), fez o **beta test checklist**, e planejou + implementou **Epic 19 completo (Agendamento Inteligente) até a Story 19.3** — epic, architecture review, 5 stories criadas e validadas, 3 implementadas (19.1, 19.2, 19.3) com gate PASS, deployado em produção. `/health` confirmado ok após deploy (api/database/redis todos ok).

---

## 🔴 Pendências abertas

### 1. 🎯 Epic 19 — continuar com 19.4 e 19.5 (frontend)
Backend completo e no ar (agendar, cancelar, reagendar, executor Beat, sugestão de horário). Faltam:
- **19.4** — botão "📅 Agendar" na tela de aprovação, horário sugerido editável (reusa `copy_result.suggested_time` como default + `GET /insights/best-posting-time`)
- **19.5** — tela "Posts agendados" (listar/cancelar/reagendar). Tem uma decisão de UX em aberto (AC5): lista fica no dashboard ou no histórico — default sugerido: dashboard.
- Ambas dependem de 19.1 (pronta). Ordem: 19.4 → 19.5.

### 2. Story 6.3 — checagem visual pendente (MANUAL-001, não bloqueante)
PostCard multi-foto foi implementado e tem gate PASS, mas o código nunca foi visto rodando em navegador real (só validado por tsc/build). Recomendado conferir visualmente antes do beta test.

### 3. Meta App Review / InstagramAnalyticsCard (bloqueado no usuário)
`InstagramAnalyticsCard` desabilitado em `frontend/app/dashboard/page.tsx:445`. Gargalo: o usuário ainda precisa **gravar os vídeos de demonstração** exigidos pelo App Review. Não é tarefa de código.

### 4. Beta test com colega da imobiliária
Planejado para a semana de 2026-07-27 (o usuário disse "semana que vem" em 2026-07-25). Checklist pronto em `docs/beta-onboarding-checklist.md` — passo crítico: adicionar o colega como Tester no painel Meta antes de enviar.

### 5. Backlog de valor (não priorizado)
- **0.3 Analytics PostHog** — só 2/5 eventos emitem (`post_created`, `post_published`); faltam `post_approved`, `upgrade_clicked`, `churn`.
- **Epic 25 — Modelos LLM** — código ainda em `claude-sonnet-4-6` / `claude-haiku-4-5`; Sonnet 5 / Opus 4.8 disponíveis.
- **Thumbnail de vídeo no PostCard** — posts de vídeo mostram tela preta no card, sem frame/preview. Precisa extrair frame (ffmpeg) + salvar R2 + expor no schema — vira story própria.

---

## ✅ Estado dos repositórios (2026-07-25)

| Repo | Branch | Estado |
|------|--------|--------|
| autopost-docs (docs) | master | ✅ sincronizado com origin |
| autopost-backend (backend) | main | ✅ sincronizado com origin — **deploy em produção confirmado saudável** (`/health`: api/database/redis ok) |
| autopost-frontend (frontend) | master | ✅ sincronizado com origin |

## 📦 Epic 19 — status das stories

| Story | Status | Gate |
|---|---|---|
| 19.1 (modelo + endpoint agendar) | Done | PASS (96) |
| 19.2 (executor Beat + cancelar/reagendar) | Done | PASS (95) |
| 19.3 (sugestão de horário) | Done | PASS (96, após fix de bug de comparação de segmento) |
| 19.4 (frontend — botão Agendar) | Ready (não iniciada) | — |
| 19.5 (frontend — tela de agendados) | Ready (não iniciada) | — |
