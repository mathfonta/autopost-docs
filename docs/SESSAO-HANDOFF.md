# 🔄 Handoff de Sessão — AutoPost

> **Propósito:** ponto de retomada do projeto. Ao dar `resume` ou `*retomar`, o agente lê ESTE arquivo + o doc de implementação (`docs/plano-mestre-2026-07.md`) para saber onde paramos.
> **Como manter:** este arquivo é sobrescrito ao FECHAR cada sessão de trabalho, refletindo as pendências reais do momento. Não é histórico — é sempre o estado atual.

**Última atualização:** 2026-07-24 · @aios-master (Orion)

---

## 📍 Onde paramos

Sessão de 2026-07-24 fechou a **Story 12.4** (erro de OAuth para conta Instagram não-profissional, gate PASS validado manualmente), fez uma **auditoria completa do projeto**, corrigiu **23 status de story desatualizados** e **reconciliou o plano mestre** com o estado real do código.

---

## 🔴 Pendências abertas

### 1. Story 6.3 — Frontend Multi-Foto (PAUSADA, aguardando decisão do usuário)
- **Descoberta importante:** o carrossel de fotos **já funciona** por outro fluxo (`frontend/components/dashboard/UploadScreen.tsx` modo `multi-image` + `app/dashboard/page.tsx`). A story foi escrita para uma arquitetura (`PhotoPreviewQueue`, `IntentMenu` com `photoCount`) que o time não seguiu.
- **Gap real que falta:** (a) `PostCard.tsx` não exibe múltiplas fotos — sempre mostra só `photo_url` (AC6); (b) "Antes/Depois" não existe como tipo de post (`lib/post-types.tsx` só tem `feed_photo`, `carousel`, `reels`, `story`).
- **Bloqueio:** usuário quer conferir o PostCard/carrossel atual em produção (`autopost.app.br`) antes de decidir escopo — só fechar o gap do PostCard, ou também criar Antes/Depois.

### 2. Commit + push pendente no repo `docs` (autopost-docs)
Mudanças locais NÃO commitadas neste repo:
- 23 arquivos de story com Status → `Done` (bookkeeping da auditoria)
- Story 4.4: checkbox do `FRONTEND_URL` marcado
- `plano-mestre-2026-07.md`: seção "Reconciliação 2026-07-24"
- este `SESSAO-HANDOFF.md`
- **Ação:** @dev commita, @devops faz push — quando o usuário aprovar.

### 3. Meta App Review / InstagramAnalyticsCard (bloqueado no usuário)
- `InstagramAnalyticsCard` desabilitado em `frontend/app/dashboard/page.tsx:445` (permissão `instagram_manage_insights` não aprovada).
- **Gargalo atual:** o usuário ainda precisa **gravar os vídeos de demonstração** exigidos pelo App Review. Não é tarefa de código.

### 4. 🎯 Fila de Produção (aprovada pelo usuário para desenvolver)
Ordem de prioridade — próximos alvos de produto:

1. **Epic 19 — Agendamento Inteligente** ⬅️ colocado na fila em 2026-07-25
   - Na tela de aprovação, além de "Publicar agora", oferecer "📅 Agendar — melhor horário" com sugestão baseada em dados (histórico do usuário → Meta `audience_online` → Exa semanal → fallback 9h/12h/19h).
   - Cenário-gatilho do usuário: criar post à noite, mas o ideal é postar de manhã.
   - Fontes 1/3/4 já existem; NÃO depende da aprovação da Meta. Esforço: M (3–4 stories).
   - Especificado em `docs/plano-mestre-2026-07.md` (Epic 19) e Obsidian `🗺️ Roadmap — Ideias e Pendências.md`.
   - **Próximo passo:** `@pm *create-epic` para formalizar em epic + stories.

### 5. Backlog de valor (ainda não priorizado)
- **0.3 Analytics PostHog** — só 2/5 eventos emitem (`post_created`, `post_published`); faltam `post_approved`, `upgrade_clicked`, `churn`. Infra já pronta.
- **Epic 25 — Modelos LLM** — código ainda em `claude-sonnet-4-6` / `claude-haiku-4-5`; Sonnet 5 / Opus 4.8 disponíveis. Revisitar à luz do ADR de roteamento LLM.
- **Thumbnail de vídeo no PostCard** — descoberto em 2026-07-25 (usuário notou ao revisar a 6.3): posts de vídeo (Reels/Story) mostram uma tela preta com ícone de play no card, sem nenhum frame/preview do vídeo. Diferente do gap do multi-foto, aqui o dado nem existe — `ContentRequest` não tem campo de thumbnail/poster. Precisa: (a) backend extrair um frame do vídeo (ffmpeg) e salvar no R2, (b) expor no schema, (c) frontend trocar a tela preta pela imagem. Não é 1-linha — vira uma story pequena própria.

---

## ✅ Estado dos repositórios (2026-07-24)

| Repo | Branch | Estado |
|------|--------|--------|
| autopost-docs (docs) | master | ⚠️ mudanças locais não commitadas (ver pendência 2) |
| autopost-backend (backend) | main | ✅ sincronizado com origin |
| autopost-frontend (frontend) | master | ✅ sincronizado com origin (5 commits pushados nesta sessão) |
