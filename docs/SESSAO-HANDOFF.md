# 🔄 Handoff de Sessão — AutoPost

> **Propósito:** ponto de retomada do projeto. Ao dar `resume` ou `*retomar`, o agente lê ESTE arquivo + o doc de implementação (`docs/plano-mestre-2026-07.md`) para saber onde paramos.
> **Como manter:** este arquivo é sobrescrito ao FECHAR cada sessão de trabalho, refletindo as pendências reais do momento. Não é histórico — é sempre o estado atual.

**Última atualização:** 2026-07-26 · sessão de resposta a incidente pós-Epic 19 + Epic 25 (modelos LLM) + Epic 27 (inteligência multi-nicho) via Spec Pipeline + fechamento da Story 6.3

---

## 📍 O que finalizamos nesta sessão

1. **Epic 19 (Agendamento Inteligente)** — 5/5 stories Done, validado em produção real pelo usuário (agendar, sugestão, editar, cancelar, reagendar, tudo funcionando).
2. **Dois incidentes de deploy corrigidos** (quebraram produção depois do Epic 19, ambos silenciosos ao `/health`): colisão de revision ID no Alembic (migração nunca aplicava) + branch do Vercel desalinhada (frontend nunca chegava em produção). Ver detalhes em [[project_epic19_deploy_incidents]] (memória) — vale reler antes do próximo deploy que toque schema ou em caso de comportamento estranho pós-deploy.
3. **Horário sugerido de agendamento corrigido** — não inventa mais uma recomendação sem dado real; mostra mensagem honesta quando não há histórico nem pesquisa Exa.
4. **Thumbnail de vídeo implementado** — posts de reels/story agora mostram um frame real do vídeo em vez de tela preta, nos 3 lugares onde apareciam (card de aprovação, publicações recentes, agendados).
5. **Bookkeeping de stories** — Story 4.4 corrigida para Done (estava presa em "Ready for Review" por esquecimento, mas está em produção e uso contínuo há meses).
6. **Roteiro completo do beta test pronto** — `docs/roteiro-beta-colega.md`: mensagem para enviar, explicação do app, passo a passo de instalação (PWA, iOS/Android), motivo do tester Meta, roteiro de feedback, e cronograma separado para admin (você) e para o colega.
7. **Epic 25 (Modelos LLM) Done** — `claude-sonnet-4-6` → `claude-sonnet-5` em 5 arquivos do backend. Gate PASS (100). Ver `docs/stories/epic-25-modelos-llm/`.
8. **Epic 27 (Inteligência de mercado multi-nicho) Done** — 3/3 stories, gates PASS (100, 100, 95). `/insights/weekly` não serve mais dado de "Construção civil" para clientes de outro segmento; queries/prompts do produtor generalizados via Gemini Flash; fan-out por segmento distinto ativo com guardrail de custo e isolamento de falha. Passou pelo Spec Pipeline completo (6 fases) antes de virar stories. Ver `docs/stories/epic-27-intel-multinicho/`.
9. **Story 6.3 (PostCard multi-foto) fechada** — usuário confirmou visualmente que a faixa de thumbnails + carrossel em tela cheia funcionam corretamente. Gate CONCERNS → **PASS (98)**. Essa era a única pendência real de QA em aberto no projeto — agora não há nenhuma.

Todos os commits desta sessão estão pushed e os repos (docs/backend) sincronizados com o remoto. Frontend não teve mudança de código nesta sessão.

---

## 🔴 Pendências para a próxima sessão

### 1. Enviar o roteiro do beta test para o colega — usuário vai fazer contato esta semana
`docs/roteiro-beta-colega.md` está pronto. Siga a Parte 1 do cronograma "Admin" dentro do documento (pegar @ do Instagram do colega, adicionar como Tester na Meta, confirmar app no ar). **Ação do usuário, não bloqueada em código.**

### 2. Meta App Review / InstagramAnalyticsCard (bloqueado no usuário)
Ainda precisa gravar os vídeos de demonstração exigidos pelo App Review. Não é tarefa de código.

### 3. Epic 28 — análise de concorrentes + re-análise do Scout (planejamento futuro, não urgente)
Escopo que ficou de fora do Epic 27 no split do `@architect`: análise de concorrentes por segmento (capacidade nova, maior risco/incerteza — precisa de Research própria sobre como extrair sinal via Exa) e re-análise periódica do Agente Scout.

**Avaliação de prioridade feita nesta sessão (2026-07-26):** não é urgente. Razões — (a) FR-7 é o item de maior incerteza de todo o planejamento original, exigiria uma Fase 3 (Research) própria antes de qualquer story; (b) o beta test (pendência 1) validaria o Epic 27 com clientes reais de outros segmentos primeiro, o que alimentaria essa Research com casos reais em vez de hipotéticos; (c) não há bug nem bloqueio de negócio empurrando isso agora — é diferencial competitivo futuro. Recomendado retomar como um ciclo de planejamento dedicado (sessão inteira: Research + Spec Pipeline), idealmente depois que o beta test já estiver rodando.

### 4. Gate operacional do Epic 27 — antes de habilitar Exa multi-segmento em produção
Epic 27 está Done no código, mas `EXA_PROVIDER=exa` com fan-out por segmento ainda não deve ser ligado sem: (a) confirmar o N real de segmentos distintos ativos em produção (Supabase estava sem token de acesso durante o planejamento — CON-3), e (b) setar `MAX_SEGMENTS_PER_RUN` explicitamente no Railway (guardrail de custo). Detalhes em `docs/stories/epic-27-intel-multinicho/epic.md`.

### 5. Backlog de valor (não priorizado)
- Analytics PostHog incompleto (faltam `upgrade_clicked`, `churn` — `post_approved` já existe, achado ao investigar nesta sessão)
- Epic 23/24 (Score de engajamento, Mix editorial) — não iniciados
- Limpeza de clientes de teste acumulados no banco de produção

### 6. Nota informativa (não bloqueante) — 3 gates antigos com CONCERNS de baixa severidade
`docs/qa/gates/2.3-agente-designer.yml`, `2.6-api-conteudo.yml`, `2.7-oauth-meta.yml` (todos de abril/2026, severidade "low", código estável em produção desde então). Não requer ação — só registrado para não ser confundido com pendência nova numa auditoria futura.

---

## ✅ Estado dos repositórios (2026-07-26, fim de sessão)

| Repo | Branch | Estado |
|------|--------|--------|
| autopost-docs (docs) | master | commit deste handoff + fechamento Story 6.3 pendente de push (será feito ao fechar) |
| autopost-backend (backend) | main | ✅ sincronizado — `1768e54`, Epic 27 completo |
| autopost-frontend (frontend) | main | ✅ sincronizado — `aa181a2`, deploy saudável, validado pelo usuário |

## 🚨 Incidentes desta sessão (resumo — detalhes completos em [[project_epic19_deploy_incidents]])

1. **Migração Alembic nunca aplicou** — colisão de revision ID entre migrações novas e antigas. Corrigido, aplicado, verificado.
2. **Frontend nunca chegou em produção** — branch local `main` rastreando `origin/master` em vez de `origin/main`. Corrigido, tracking arrumado.
3. **Post travado em "publishing"** — efeito colateral do redeploy interrompendo o worker no meio de uma publicação. Resetado manualmente, sem ação pendente.
4. **Horário sugerido inventado sem dado real** — corrigido para mensagem honesta.

## 📦 Epic 19 — status final

| Story | Status | Gate |
|---|---|---|
| 19.1 (modelo + endpoint agendar) | Done | PASS (96) |
| 19.2 (executor Beat + cancelar/reagendar) | Done | PASS (95) |
| 19.3 (sugestão de horário) | Done | PASS (96) — refinado nesta sessão |
| 19.4 (frontend — botão Agendar) | Done | PASS (95) — validado em produção real |
| 19.5 (frontend — tela de agendados) | Done | PASS (96) — validado em produção real |

## 📦 Epic 25 — status final

| Story | Status | Gate |
|---|---|---|
| 25.1 (model IDs Sonnet 4.6 → 5) | Done | PASS (100) |

## 📦 Epic 27 — status final

| Story | Status | Gate |
|---|---|---|
| 27.1 (fix `/weekly` + matching normalizado) | Done | PASS (100) |
| 27.2 (queries/prompts por segmento — produtor) | Done | PASS (100) |
| 27.3 (fan-out por segmento + guardrail + dedup) | Done | PASS (95) |

## 📦 Story 6.3 — fechada nesta sessão

| Story | Status | Gate |
|---|---|---|
| 6.3 (PostCard multi-foto) | Done | PASS (98) — MANUAL-001 resolvido, checagem visual confirmada pelo usuário |

Suíte de testes backend: **416 testes, 0 falhas** (era 398 no início da sessão).

---

## 🎯 Estado geral do projeto ao fechar esta sessão

**Nenhuma pendência real de QA em aberto.** Todos os gates de stories ativas estão PASS. As pendências restantes são: uma ação do usuário (beta test, contato esta semana), um bloqueio do lado do usuário (gravar vídeos do Meta App Review), um gate operacional pré-produção do Epic 27 (não bloqueia nada, só precisa ser feito antes de ligar Exa multi-segmento), e itens de planejamento futuro não urgentes (Epic 28, backlog de valor).

**Próxima sessão:** não há decisão de prioridade pendente — o caminho natural é aguardar o retorno do beta test do colega, e/ou atacar itens pequenos do backlog de valor (Analytics PostHog, 2 eventos) se houver tempo de execução disponível sem planejamento prévio necessário.
