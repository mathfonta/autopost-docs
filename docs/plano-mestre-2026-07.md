# 🗺️ Plano Mestre AutoPost — Diagnóstico e Melhorias

> **Data:** 2026-07-07 · **Autor:** @aios-master (Orion) · **Status:** PLANEJADO — nada executado
> **Propósito:** documento de referência para implementação posterior via SDC (`@pm *create-epic` → `@sm *draft` → `@po *validate` → `@dev *develop` → `@qa *qa-gate` → `@devops *push`)

---

## 1. Resumo Executivo

O AutoPost está **feature-complete para MVP** (Epics 1–18 Done): pipeline foto/vídeo 100% funcional em produção, modo autônomo, camada de intenção de marketing, plano de ataque v2, recomendações data-driven e slide_designer v2. O gargalo atual é **lançamento, não desenvolvimento**.

**Bloqueio central de negócio:** Meta App Review (`instagram_manage_insights` + `instagram_manage_comments`). Sem aprovação, clientes novos não conectam Instagram e o `InstagramAnalyticsCard` permanece desabilitado.

**Posicionamento recomendado:** não é "ferramenta de post" (Predis/Ocoya, US$ 15–30/mês) — é **"social media autônomo que aprende com o nicho"**. Diferenciais defensáveis: nicho construção civil BR, aprendizado acumulado cross-cliente (cérebro local + global), sequência editorial estratégica, modo autônomo com intenção de marketing. Pricing pode ficar acima dos ~US$ 27 do Predis.

### Stack em produção (verificado 2026-07-07)
| Camada | Tecnologia |
|--------|-----------|
| Backend | FastAPI + Celery + Redis → Railway (api + worker + beat) |
| Frontend | **Next.js 16.2.4 + React 19.2.4 + Tailwind 4** → Vercel (`autopost.app.br`) |
| Banco | Supabase (PostgreSQL + RLS) |
| Storage | Cloudflare R2 |
| LLM | Claude `claude-sonnet-4-6` (copy/cérebro/onboarding), `claude-haiku-4-5` (análise imagem), Gemini 2.5 Flash (alternativo + transcrição) |
| Busca | Exa API (tendências + hashtags, semanal) |
| Analytics | PostHog (chaves configuradas; eventos NÃO implementados) |

> ⚠️ O Obsidian (`📊 Status do Projeto.md`) estava parado em 2026-05-15 e citava "Next.js 14". Atualizado nesta sessão.

---

## 2. Gargalos Identificados

### 2.1 Código — Backend

| # | Gargalo | Evidência | Risco |
|---|---------|-----------|-------|
| B1 | God-file de pipeline | `backend/app/tasks/pipeline.py` (38KB): 10 tasks Celery + helpers async + weekly intelligence + extração de hashtags no mesmo arquivo | Manutenção cara, testes acoplados, merge conflicts |
| B2 | Prompts embutidos em código | `backend/app/agents/copywriter.py` (34KB): `_SYSTEM_PROMPT`, `STRATEGY_PROMPTS`, `CONTENT_TYPE_PROMPTS`, `INTENT_CONTEXT_PROMPTS` hardcoded | Iterar copy exige deploy; A/B de prompt impossível |
| B3 | Ponte sync/async frágil | `_run_sync(coro)` envolve todo acesso async dentro de tasks Celery síncronas | Bugs sutis de event loop |
| B4 | Duplicação Redis | `core/redis.py` E `core/redis_client.py` coexistem | Confusão de imports |
| B5 | Modelos LLM defasados | `claude-sonnet-4-6` e `claude-haiku-4-5` em uso; Sonnet 5 / Opus 4.8 disponíveis | Qualidade de copy e visão abaixo do possível |
| B6 | Lixo na raiz do repo | `openapi_tmp2.json`, `ruvector.db`, `icone autopost.png` | Higiene |

### 2.2 Código — Frontend

| # | Gargalo | Evidência | Risco |
|---|---------|-----------|-------|
| F1 | Componente monolítico | `frontend/app/dashboard/page.tsx` (32KB) concentra estado + orquestração + UI | Toda mudança de UX passa por ele |
| F2 | Componente monolítico | `frontend/components/dashboard/UploadScreen.tsx` (23KB) | Idem |
| F3 | Zero testes de frontend | Nenhum runner configurado (backend tem 27 módulos de teste) | Regressões silenciosas de UX |

### 2.3 Processos

| # | Gargalo | Detalhe |
|---|---------|---------|
| P1 | Segundo Cérebro desatualizado | Status parado ~8 semanas apesar da regra de enforcement |
| P2 | Meta App Review sem rastreio | O maior bloqueador do negócio não tem story/epic com checklist e prazo |
| P3 | Hooks fantasma | `.claude/hooks/README.md` descreve 6 hooks Python (read-protection, sql-governance etc.) que **não existem** — só há 2 `.cjs` reais |
| P4 | Analytics de uso ausente | 5 eventos ("obrigatório antes de escalar" desde maio): `post_created`, `post_approved`, `post_published`, `upgrade_clicked`, `churn` |

---

## 3. Benchmark Competitivo e Tendências 2026

### 3.1 Concorrentes (US$ 15–40/mês)

| Produto | Preço | Força | AutoPost tem? |
|---------|-------|-------|---------------|
| Predis.ai | ~$27 | Prompt → carrossel/vídeo/imagem + caption; foco e-commerce | ✅ Equivalente |
| FeedHive | ~$19 | **Predição de engajamento pré-publicação**, post recycling, posting condicional | ❌ Gap — e temos o dado do cérebro para fazer melhor |
| Ocoya | ~$15 | Copy em 26 idiomas, calendário integrado | ⚠️ Sem calendário visual |
| Buffer/Publer/SocialBee | $10–29 | Multi-rede (TikTok, LinkedIn, YouTube) | ❌ Só IG/FB — aceitável para o nicho atual |

### 3.2 Tendências que devem virar produto

1. **Mix ideal 2026:** 60–70% Reels, 20–30% carrosséis, ≤10% estáticos → o plano de ataque deve *impor* esse mix.
2. **Carrosséis:** engajamento 0,72% (+30,9% vs 2025) → reforça modo autônomo de carrossel.
3. **DM shares pesam 3–5× mais que likes** no algoritmo de Reels → copy precisa de CTA de compartilhamento ("manda pra quem tá construindo"), não só "comente".
4. **Autenticidade > produção:** Instagram despriorizando conteúdo over-produced/IA genérico. Foto real de obra = vantagem estrutural do AutoPost; copy deve soar humana.
5. **Brasil:** 3º maior mercado do Instagram (134M usuários).

Fontes: Apaya, TheStacc, Zapier, Sprout Social, Searchlab, TrueFuture Media, Manychat (pesquisa 2026-07-07).

---

## 4. Plano por Fases

```
FASE 0 (esta semana)  → 0.1 Meta Review · 0.2 Obsidian · 0.3 Analytics
FASE 1 (quick wins)   → Epic 21 Copy-Chief · Epic 25 Modelos LLM
FASE 2 (produto)      → Epic 19 Agendamento · Epic 23 Score · Epic 24 Mix
FASE 3 (paralelo)     → Epic 22 Refatoração (por story, sem parar produto)
FASE 4 (pós-launch)   → Epic 20 Biblioteca de fotos · Epic 26 Boost pago
```

---

### 🔴 FASE 0 — Destravar o negócio

#### 0.1 Meta App Review (story formal)
- **Dono:** @devops + @pm
- **Escopo:** checklist de submissão, screencast das permissões em uso, justificativas (`instagram_manage_insights`, `instagram_manage_comments`), acompanhamento de prazo
- **Referência:** `docs/meta-app-review-resubmissao.md`
- **AC:** submissão confirmada com Submission ID registrado no Obsidian; card de acompanhamento com data-limite

#### 0.2 Atualização do Segundo Cérebro
- **Dono:** @aios-master
- **Escopo:** Status do Projeto (stack Next 16, epics 15–18, este plano), Roadmap, registro do diagnóstico
- **Status:** ✅ feito nesta sessão (2026-07-07)

#### 0.3 Analytics de uso — 5 eventos PostHog
- **Dono:** @dev
- **Escopo IN:** `post_created`, `post_approved`, `post_published`, `upgrade_clicked`, `churn` — backend emite, frontend complementa; dashboard PostHog básico
- **Escopo OUT:** funis avançados, session replay
- **Por quê:** pré-requisito para escalar e para calibrar Epics 19/23
- **Esforço:** S (1–2 stories)

---

### 🟠 FASE 1 — Quick wins de qualidade

#### Epic 21 — Copy-Chief: Elevação de Prompts
- **Origem:** ideia registrada no Obsidian 2026-06-17 (análise completa pronta: `ideia-copy-chief-elevacao-prompts`)
- **O que é:** usar o agente copy-chief (Obras SaaS) para auditar e elevar os prompts do `copywriter.py`. Metodologia: diagnóstico de consciência (Eugene Schwartz), auditoria Hopkins (mínimo 85/100), 30 Sugarman triggers. O copy-chief NÃO substitui o worker — melhora os prompts em sessão Claude Code; Railway continua igual.
- **Impacto estimado:** hook +60–70%; variação real entre as 3 cópias (personas distintas)
- **Custo de infra:** ZERO
- **Incluir na auditoria:**
  - CTA de **DM share** nos Reels (algoritmo pesa 3–5×)
  - Tom "humano/autêntico" (tendência anti-IA-genérica)
  - Variação estrutural real entre as 3 variantes de caption
- **Stories sugeridas:**
  1. 21.1 — Copiar `copy-chief.md` para `.claude/agents/` do AutoPost + rodar `audit-copy` sobre `_SYSTEM_PROMPT` e `STRATEGY_PROMPTS`
  2. 21.2 — Aplicar prompts elevados + A/B manual (10 posts antes/depois) + registrar resultado no cérebro
- **Esforço:** S (1–2 sessões)

#### Epic 25 — Atualização de Modelos LLM
- **O que é:** migrar `claude-sonnet-4-6` → `claude-sonnet-5` (copy, onboarding, cérebro); avaliar `claude-haiku-4-5` → Sonnet 5 apenas onde visão importa; manter Gemini 2.5 Flash como provider de volume (conforme ADR de roteamento LLM).
- **Arquivos afetados:** `agents/copywriter.py:25`, `agents/analyst.py:29`, `agents/onboarding.py:21`, `agents/theme_generator.py:91`, `cerebro/analyzer.py:16`, `cerebro/promoter.py:18`
- **Regras:**
  - Rodar avaliação A/B de copy ANTES de trocar default (Sonnet 5 vs Sonnet 4.6 vs Gemini)
  - Modelo deve virar config (`config.py` / env var), não constante espalhada
  - Medir custo por post antes/depois
- **Stories sugeridas:**
  1. 25.1 — Centralizar model IDs em `config.py` com env override
  2. 25.2 — A/B Sonnet 5 na copy + análise de custo + decisão registrada como ADR
- **Esforço:** S

---

### 🟡 FASE 2 — Produto orientado a mercado

#### Epic 19 — Agendamento Inteligente
*(já especificado no Roadmap Obsidian — promover a epic formal)*
- **O que é:** na aprovação, além de "Publicar agora", o sistema sugere o melhor horário com explicação: *"📅 Agendar — hoje às 19h · Seus posts de quarta às 19h têm 2,3× mais engajamento"*
- **Fontes do horário (ordem de prioridade):**
  1. Histórico do próprio usuário (posts × horário × engajamento — dado já existe)
  2. `insights/audience_online` Meta Graph API (quando permissão aprovada)
  3. Exa semanal — melhores horários do nicho (já roda toda segunda)
  4. Fallback estático: 9h / 12h / 19h (construção civil)
- **Stack:**
  - Backend: campo `scheduled_for` no `ContentRequest` + Celery Beat verifica a cada minuto
  - Backend: `GET /insights/best-posting-time?format={format}` → `{horario, fonte, confianca}`
  - Frontend: `ApprovalScreen` ganha botão secundário de agendamento com horário sugerido + razão inline
- **Dependências:** 0.3 (analytics ajuda a calibrar); Meta Review melhora a fonte 2 mas NÃO bloqueia (fontes 1/3/4 já disponíveis)
- **Esforço:** M (3–4 stories)

#### Epic 23 — Score de Engajamento Pré-Publicação
*(resposta direta ao FeedHive — com vantagem de dados que ele não tem)*
- **O que é:** na tela de aprovação, previsão de performance: *"⚡ Posts assim renderam 2,3× mais engajamento"* — baseado em padrões do cérebro + histórico do cliente + formato + horário
- **v1 (heurística):** regressão simples sobre features já coletadas (formato, estratégia, tema, horário, dia, hashtags) × engajamento histórico. Sem ML pesado.
- **v2 (futura):** incorporar métricas Meta pós-aprovação da permissão de insights
- **Diferencial competitivo:** o score melhora com cada cliente novo (efeito de rede do cérebro global — FeedHive não tem isso)
- **UI:** badge de score (baixo/médio/alto) + 1 frase de explicação + sugestão de melhoria quando score baixo ("adicione CTA de compartilhamento")
- **Dependências:** 0.3 analytics; se junto com Epic 19, compartilham o endpoint de insights
- **Esforço:** M

#### Epic 24 — Mix Editorial Guiado por Algoritmo
- **O que é:** o plano de ataque passa a balancear automaticamente 60–70% Reels / 20–30% carrossel / ≤10% estático, com indicador visual do mix da semana no dashboard
- **UI:** barra de mix semanal (tipo "anel de atividade") + recomendação do próximo formato: *"Essa semana você só postou estáticos — o próximo deveria ser um Reel"*
- **Backend:** regra de mix no gerador de recomendações (Epic 18.2 já criou a base data-driven)
- **Esforço:** S–M

---

### 🟢 FASE 3 — Restruturação técnica (paralelo, por story)

#### Epic 22 — Refatoração Estrutural

| Story | Escopo | Regra de segurança |
|-------|--------|--------------------|
| 22.1 | Dividir `pipeline.py` → `pipeline_content.py` (analyze/copy/design/publish), `pipeline_metrics.py` (metrics + cérebro), `pipeline_weekly.py` (Exa intelligence) | **Manter os nomes das tasks Celery** (`pipeline.analyze_photo` etc.) → zero mudança de deploy/beat |
| 22.2 | Extrair prompts → `app/prompts/` (arquivos versionados, carregados por provider; opcional: tabela Supabase com fallback local) | Testes de snapshot dos prompts montados; habilita A/B sem deploy |
| 22.3 | Unificar `core/redis.py` + `core/redis_client.py` | Grep de todos os imports antes |
| 22.4 | Frontend: quebrar `dashboard/page.tsx` em subcomponentes por seção + hook `useContentFlow` (máquina de estados upload→geração→aprovação) | Sem mudança visual — refactor puro |
| 22.5 | Frontend: Vitest + Testing Library + smoke tests dos fluxos críticos (upload, aprovação, histórico) | CI roda os testes nos PRs |
| 22.6 | Higiene: remover `openapi_tmp2.json`, `ruvector.db` da raiz; mover ícone p/ `frontend/public/`; atualizar `.gitignore` | — |

- **Esforço total:** M–L, mas cada story é independente e pequena

---

### 🔵 FASE 4 — Pós-lançamento

#### Epic 20 — Varredura de Biblioteca de Fotos
*(já esboçado no Roadmap — 3 fases evolutivas)*

| Fase | Comportamento | Entrega |
|------|--------------|---------|
| 1. Sugestão | Agente varre pasta (Google Drive ou galeria via PWA/File System Access API), sugere post por foto adequada; aprovação individual | Primeira |
| 2. Lote | Agente monta semana inteira (foto+copy+estratégia+horário); aprovação em lote | ~3 meses depois |
| 3. Autônomo | Publica sozinho conforme calendário dentro de parâmetros aprovados | Futuro |

- **Dependência:** Epic 19 (agendamento) facilita a Fase 1
- **Esforço:** L

#### Epic 26 — Boost Pago (Meta Ads)
- **O que é:** botão "🚀 Impulsionar" no post aprovado/publicado → cria boost via Meta Ads (MCP `ads_boost_ig_post` já disponível no ambiente; em produção, via Marketing API)
- **Por quê:** nenhuma ferramenta SMB BR do segmento faz boost integrado ao fluxo de aprovação — upsell natural (fee sobre gasto de mídia ou plano premium)
- **Pré-requisitos:** Meta Review aprovado; clientes com contas de anúncio; modelagem de cobrança
- **Esforço:** L

---

## 5. UX / Layout / Interação
*(validar com @ux-design-expert — alimenta Epics 19/23/24)*

| # | Melhoria | Detalhe |
|---|----------|---------|
| U1 | **Aprovação como centro de decisão** | Preview + score previsto (E23) + horário sugerido (E19) + edição inline de caption → decisão em um toque |
| U2 | **Calendário visual de conteúdo** | Visão semana: agendados + sugestões do plano de ataque + mix Reels/carrossel — gap vs Ocoya/FeedHive |
| U3 | **Indicador de mix semanal** | Anel/barra 60-30-10 no dashboard (E24) |
| U4 | **Onboarding "resultado em 48h"** | Medir com eventos do 0.3; reduzir passos até o primeiro post publicado |
| U5 | **TWA Google Play** | Empacotar PWA como app Android (1–2 semanas) — confiança do público-alvo |
| U6 | **Refactor do dashboard** | Pré-requisito técnico das anteriores (story 22.4) |

---

## 6. AIOS — Agentes, Skills, Hooks, MCP

| # | Item | Proposta | Dono |
|---|------|----------|------|
| A1 | Hooks fantasma | Corrigir `.claude/hooks/README.md`: implementar os 6 hooks Python descritos OU reescrever o README para refletir os 2 `.cjs` reais | @aios-master |
| A2 | Skills de marketing | Formalizar as 3 skills do Obsidian (🔬 Copy Viral GPT-10X, Carrossel Social Media, Stories Viral) como tasks/skills versionadas no repo | @aios-master |
| A3 | Agente @copy-chief | Tornar permanente em `.claude/agents/` — auditoria contínua de prompts a cada mudança no copywriter (liga com Epic 21) | @aios-master |
| A4 | MCP Meta Ads | Base do Epic 26 (boost); usar `ads_insights_industry_benchmark` para calibrar expectativas de score (E23) | @devops |
| A5 | MCP Canva | Avaliar como gerador alternativo de templates para o slide_designer (comparar com pipeline atual) | @architect |
| A6 | ruflo memory | Reforçar STORE de padrões de copy que performam (namespace `autopost`) — hoje subutilizado | todos |
| A7 | AIOS upstream | PR do `hotfix.yaml` + `hotfix-spec.md` para SynkraAI/aiox-core (Opção A do Obsidian) | @devops |

---

## 7. Riscos e Mitigações

| Risco | Prob. | Impacto | Mitigação |
|-------|-------|---------|-----------|
| Meta Review negado/atrasado | Média | Alto | Fontes alternativas de dados (histórico próprio + Exa) já cobrem E19/E23 v1; resubmissão com screencast melhorado |
| Troca de modelo piora copy no nicho | Baixa | Médio | A/B obrigatório antes de trocar default (25.2) |
| Refatoração quebra pipeline em produção | Baixa | Alto | Manter nomes de tasks Celery; smoke tests já existem; uma story por vez |
| Score de engajamento impreciso no início | Alta | Baixo | Comunicar como "estimativa"; melhora com volume; fallback: ocultar score com <10 posts |
| Dispersão (muitos epics abertos) | Média | Médio | Regra: máx. 1 epic de produto + 1 story de refatoração em paralelo |

---

## 8. Ordem de Execução Recomendada

| Ordem | Item | Esforço | Impacto | Gate |
|-------|------|---------|---------|------|
| 1 | 0.1 Meta Review | S | 🔴 Crítico | — |
| 2 | 0.3 Analytics 5 eventos | S | 🔴 Crítico | — |
| 3 | Epic 21 Copy-Chief | S | Alto | A/B 10 posts |
| 4 | Epic 25 Modelos LLM | S | Médio-Alto | A/B custo/qualidade |
| 5 | Epic 19 Agendamento | M | Alto | — |
| 6 | Epic 23 Score | M | Alto (diferencial) | ≥10 posts de histórico |
| 7 | Epic 24 Mix Editorial | S–M | Médio | — |
| 8 | Epic 22 Refatoração | M–L | Técnico | 1 story por vez, em paralelo |
| 9 | Epic 20 Biblioteca | L | Alto (retenção) | Pós-launch |
| 10 | Epic 26 Boost | L | Alto (receita) | Meta Review + clientes ativos |

**Próximo comando sugerido:** `@pm *create-epic` para Epic 19 e Epic 21.

---

## Change Log

| Data | Autor | Mudança |
|------|-------|---------|
| 2026-07-07 | @aios-master (Orion) | Documento criado a partir do diagnóstico completo da sessão |
