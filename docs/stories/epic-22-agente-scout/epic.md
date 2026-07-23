# Epic 22 — Agente Scout: Análise de Perfil e Auto-Calibração

**Status:** Ready — epic aprovado pelo usuário; 5 stories validadas pelo @po (GO), aguardando go de implementação
**Data:** 2026-07-22
**Prioridade:** Alta
**Origem:** Visão recorrente do usuário ("agente bisbilhoteiro") + avaliação técnica @aios-master 2026-07-22
**Dependência de entrada:** Story 12.4 (tratamento de erro no OAuth) — o fluxo de conexão precisa produzir token de forma confiável, o que a 12.4 acabou de garantir.

---

## Objetivo

Depois que o cliente conecta o Instagram, o **Scout** acessa o perfil real dele, analisa os posts existentes e **auto-calibra os demais agentes** (copywriter, temas) para o nicho e a função reais do cliente — em vez de depender exclusivamente da lista fixa de segmentos coletada no onboarding conversacional.

O resultado: um cliente cujo perfil real é "marcenaria de móveis planejados sob medida" deixa de ser tratado genericamente como "comércio" ou "outro", e passa a receber temas, gatilhos, tom e horários calibrados ao que ele realmente faz e para quem.

---

## Problema

Hoje o `brand_profile` — a **única superfície de calibração de todo o sistema de conteúdo** — é preenchido de duas formas, ambas com o mesmo limite:

1. **Onboarding conversacional** (`app/agents/onboarding.py`): o Claude força o segmento para uma lista fixa — `construção civil, arquitetura, saúde, dentista, advogado, contador, comércio, outro`.
2. **Formulário direto** (`POST /onboarding/setup`): o usuário escolhe entre as mesmas opções.

O `segment` resultante é a **chave de roteamento de praticamente tudo a jusante**:

- `app/data/theme_library.py` — `THEME_LIBRARY` é indexado por segmento
- `app/agents/copywriter.py` — `_VIRAL_TRIGGERS[segment]` e `_DEFAULT_TIMES[segment]`
- `app/agents/theme_generator.py` — seleciona temas pelo segmento

Consequência: quando o segmento real do cliente não está na lista (ou cai em "outro"), **todo o conteúdo gerado fica genérico** — temas, gatilhos virais, horários e tom deixam de ser específicos. O cliente declara o que acha que é; o sistema nunca olha o que ele **de fato** publica.

Ao mesmo tempo, no momento em que o cliente conecta o Instagram, o AutoPost recebe um token (`instagram_business_basic`) que **já permite ler os próprios posts do cliente** — uma fonte de verdade que hoje é ignorada.

---

## Solução

Novo agente **Scout** (`app/agents/scout.py`), disparado **de forma assíncrona** logo após a conexão OAuth bem-sucedida:

```
Conexão OAuth bem-sucedida (meta_callback salva token + instagram_business_id)
  ↓
Enfileira tarefa Celery run_scout_analysis(client_id)  ← não bloqueia o onboarding
  ↓
Scout busca os N posts mais recentes via graph.instagram.com/me/media
  (caption, media_type, media_url, timestamp, like_count, comments_count)
  ↓
Análise combinada:
   • Visual  — reusa a capacidade de visão do analyst.py sobre uma amostra de imagens
   • Textual — captions + bio → nicho real, tom, temas recorrentes, público
  ↓
Síntese (Claude Sonnet — per ADR de roteamento) → ScoutReport:
   { refined_segment, inferred_tone, visual_style, recurring_topics[],
     audience_notes, confidence }
  ↓
Merge ADITIVO no brand_profile (enriquece, não sobrescreve o que o usuário declarou)
  ↓
Copywriter e theme_generator passam a ler os insights automaticamente
```

**Princípios de design (decisões do @architect):**

- **Não bloqueante:** o Scout roda em background (Celery). O onboarding termina na hora; a calibração chega depois. O usuário nunca espera pela análise.
- **Enriquecer, não sobrescrever:** valores declarados pelo usuário (`segment`, `tone`) são preservados. O Scout escreve num sub-objeto `scout_insights` e, se detectar divergência forte de segmento, **sugere** a correção para o usuário aceitar — nunca troca em silêncio.
- **Degradação graciosa:** conta nova com 0 posts, conta privada, ou falha de API → Scout é no-op e o sistema segue com o `brand_profile` declarado. Jamais quebra o onboarding.
- **Roteamento de modelos:** Sonnet para a síntese/raciocínio (baixo volume, alta qualidade — per `adr_llm_model_routing`); Gemini Flash/Haiku para a análise por imagem (volume). Reusa `_resolve_analyst_provider`.
- **Custo limitado:** amostra limitada de posts (ex.: últimos 12), resultado cacheado, execução única por conexão.

---

## Entregas

| Story | Título | Esforço | Camada |
|-------|--------|---------|--------|
| 22.1 | Leitura de mídia do Instagram (tool backend) | Médio | Backend |
| 22.2 | Agente Scout — síntese de perfil (visão + texto → ScoutReport) | Grande | Backend |
| 22.3 | Persistência, merge no brand_profile e disparo pós-OAuth | Médio | Backend + DB |
| 22.4 | Consumo a jusante — copywriter e theme_generator lêem os insights | Médio | Backend |
| 22.5 | Frontend — status "Analisando seu perfil" e sugestão de refino | Médio | Frontend |

---

## Arquitetura

### Novo agente e tools
- `app/agents/scout.py` — orquestra análise visual + textual e produz o `ScoutReport`
- `app/tools/instagram_media.py` — `fetch_recent_media(ig_id, token, limit)` via `graph.instagram.com/me/media`
- Tarefa Celery `run_scout_analysis(client_id)` — execução assíncrona

### Reuso (REUSE > ADAPT > CREATE)
- **Visão:** `app/agents/analyst.py` já tem Claude Haiku + Gemini Flash com visão, download/compressão de imagem e `_resolve_analyst_provider` — reusado para analisar os posts do cliente.
- **Token/IDs:** `meta_access_token` + `instagram_business_id` já persistidos no `Client` pela conexão OAuth.
- **Superfície de calibração:** `brand_profile` (JSONB no `Client`) já é lido por todos os agentes a jusante — o Scout só precisa enriquecê-lo.

### Mudanças no modelo (`app/models/client.py`)
- `scout_report: JSONB` — relatório bruto do Scout (auditoria/debug)
- `scout_status: str` — `pending | running | done | skipped | failed`
- Migração Alembic correspondente

### Ponto de disparo
- `app/api/meta.py:meta_callback` — no ramo de sucesso (após `db.commit()`), enfileira `run_scout_analysis`

### Sem mudanças em
- Pipeline de foto/vídeo existente (análise, designer, publisher — intactos)
- Escopos OAuth (o `instagram_business_basic` atual já cobre a leitura de mídia própria)
- Fluxo de aprovação e publicação

---

## Decisões que precisam ser travadas antes da implementação

1. **[Story 22.1] Campos reais do `/me/media` sob `instagram_business_basic`** — validar exatamente quais campos e métricas o escopo atual devolve (caption, media_url, like_count, comments_count e, principalmente, se a **bio** do perfil é acessível). Alguns campos de insight podem exigir escopo adicional — bloquear premissas até confirmar contra a API real.
2. **[Story 22.3] Auto-aplicar vs sugerir** — confirmar a regra de merge: campos aditivos (temas recorrentes, estilo visual, público) entram automaticamente; troca de `segment` divergente é **sugestão** que o usuário aceita no frontend. Travar o comportamento exato antes de codar.

---

## Escopo

### IN
- Ler os próprios posts do cliente (conta conectada)
- Análise visual + textual → nicho real, tom, temas recorrentes, estilo, público
- Enriquecer o `brand_profile` de forma aditiva
- Alimentar copywriter e theme_generator com os insights
- Status no frontend e sugestão opcional de refino de segmento

### OUT (futuro / outros epics)
- Análise de concorrentes (é o Epic 13 — EXA Intelligence)
- Re-análise periódica/contínua (MVP é one-shot na conexão)
- Análise de contas que não sejam a do próprio cliente
- Geração dinâmica de temas para nichos fora da `THEME_LIBRARY` (candidato a epic próprio)

---

## Definition of Done do Epic

- [ ] Ao conectar o Instagram, o Scout roda em background sem atrasar o onboarding
- [ ] Um cliente com posts reais tem o `brand_profile` enriquecido com insights derivados dos posts
- [ ] Copywriter e theme_generator usam os insights quando presentes, com fallback ao segmento declarado
- [ ] Conta vazia/privada/erro de API → Scout no-op, onboarding e geração seguem normalmente
- [ ] Frontend exibe o status da análise e a sugestão de refino (quando houver)
- [ ] Sem regressão no pipeline de conteúdo existente
- [ ] Testes cobrindo: caminho feliz, conta vazia, falha de API, merge aditivo

---

## Riscos

| Risco | Mitigação |
|-------|-----------|
| Campos do `/me/media` limitados pelo escopo `instagram_business_basic` | Validar contra a API real na Story 22.1 **antes** de desenhar o ScoutReport |
| Conta nova/vazia (provável no beta com usuário real) | Fallback obrigatório — Scout no-op, sistema usa brand_profile declarado |
| Custo/latência de analisar muitas imagens | Amostra limitada (ex.: 12 posts) + execução assíncrona + cache do resultado |
| Trocar o segmento do usuário sem consentimento surpreende/irrita | Aditivo por padrão; troca de segmento é sugestão explícita, nunca silenciosa |

---

## Dependências

- **Story 12.4** (tratamento de erro OAuth) — conexão confiável como pré-requisito ✅ (recém-implementada)
- Reusa: `analyst.py` (visão), `brand_profile`, token/IDs de `meta_oauth`, infra Celery existente
