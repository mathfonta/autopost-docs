# Epic 5 — Fluxo de Aprovação Avançado

**Status:** InProgress  
**Origem:** Ideas (1) — 23/04/2026 — caso real Espectra  
**Objetivo:** Dar ao usuário mais controle no momento da aprovação — editar, tentar novamente e escolher o tipo de conteúdo antes de publicar.

---

## Contexto

O Epic 4 entregou o fluxo básico: foto → pipeline → prévia → aprovar/rejeitar.  
O Epic 5 enriquece esse fluxo com três melhorias de alto impacto e baixa complexidade:

1. **Edição Inline** — editar a legenda antes de aprovar, sem sair da tela
2. **Retry** — tentar nova versão sem reiniciar o fluxo
3. **Menu de Intenção** — escolher o tipo de conteúdo antes de disparar o pipeline

---

## Stories

| Story | Título | Estimativa | Depende de |
|-------|--------|-----------|------------|
| 5.1 | Edição Inline da Legenda | 2 pts | Story 4.2 |
| 5.2 | Retry — Nova Versão sem Reiniciar | 3 pts | Story 5.1 |
| 5.3 | Menu de Intenção antes do Upload | 3 pts | Story 4.2 |

---

## Arquitetura

**Frontend:** Next.js 14 (Vercel) — `frontend/components/dashboard/`  
**Backend:** FastAPI (Railway) — novos endpoints em `app/api/content.py`  
**Pipeline:** Celery — `app/tasks/pipeline.py`

---

## Definition of Done do Epic

- [ ] Usuário pode editar a legenda antes de aprovar
- [ ] Usuário pode gerar nova versão sem sair da tela de aprovação
- [ ] Usuário escolhe o tipo de conteúdo antes de enviar a foto
- [ ] Todos os testes passando (unit + integração)
- [ ] Deploy em Vercel + Railway
