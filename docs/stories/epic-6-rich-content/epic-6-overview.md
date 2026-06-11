# Epic 6 — Conteúdo Rico: Múltiplas Fotos, Antes/Depois e Carrossel

**Status:** Draft  
**Criado em:** 2026-04-24  
**Criado por:** @pm (Morgan)  
**Origem:** Ideas session 23/04 — caso real Espectra (construção civil)

---

## Epic Goal

Permitir que o usuário envie 2 ou mais fotos e o AutoPost componha automaticamente um **antes/depois** ou **carrossel sequencial**, gerando legenda adequada ao formato e publicando no Instagram — aproveitando a narrativa visual natural que obras de construção civil geram.

---

## Contexto do Sistema Existente

- **Stack atual:** FastAPI + Celery (Railway) · Next.js (Vercel) · Supabase · Cloudflare R2 · Meta Graph API
- **Pipeline atual:** 1 foto → analyze_photo → generate_copy → prepare_design → aprovação → publish_post
- **Menu de Intenção (Story 5.3):** campo `content_type` já existe no modelo — vai ser expandido com `before_after` e `carousel`
- **Publicador (Story 2.4):** publica single post hoje; precisa suportar carousel container da Meta API
- **Upload (dashboard):** FAB aceita 1 foto hoje; frontend precisa aceitar múltiplas

---

## Enhancement Details

### O que está sendo adicionado

1. **Upload múltiplo** — frontend aceita 2–10 fotos em sequência
2. **Tipos de conteúdo novos** — `before_after` (2 fotos) e `carousel` (2–10 fotos) no Menu de Intenção
3. **Pipeline adaptado** — Analista processa cada foto, Copywriter gera legenda contextualizada ao formato
4. **Publicação carrossel** — Publicador cria container carrossel na Meta API antes de publicar

### Como integra ao sistema existente

- Modelo `ContentRequest` ganha campo `photo_urls: list[str]` (multi-foto)
- Migration adiciona coluna JSONB `photo_urls` mantendo `photo_url` por retrocompatibilidade
- Pipeline Celery existente: `analyze_photo` é executado N vezes (1 por foto)
- `content_type` já existe — apenas novos valores `before_after` / `carousel`
- Meta Graph API: carousel = múltiplos `/media` containers + 1 container principal

---

## Stories

### Story 6.1 — Upload Múltiplo + Modelo (Backend)
**Executor:** `@dev` | **Quality Gate:** `@architect`

- Migração: adiciona `photo_urls JSONB` em `content_requests` (retrocompat com `photo_url`)
- `POST /content-requests` aceita múltiplas imagens (multipart)
- Upload cada foto para R2, armazena lista de URLs no modelo
- `content_type` aceita `before_after` e `carousel` além dos existentes
- **Complexity:** 3 pts | **Risk:** MÉDIO (migração + mudança no endpoint principal)

**Quality Gates:**
- Pre-Commit: schema validation, retrocompat check
- Pre-PR: migration safety, endpoint contract validation

---

### Story 6.2 — Pipeline Antes/Depois e Carrossel (Backend Agentes)
**Executor:** `@dev` | **Quality Gate:** `@architect`

- `analyze_photo`: loop sobre `photo_urls`, gera análise por foto
- `generate_copy`: recebe análises e `content_type`, gera legenda específica ao formato
  - `before_after`: narrativa comparativa (mudança/progresso)
  - `carousel`: legenda principal + descrição de sequência
- `prepare_design`: gera uma imagem por foto (Designer itera sobre a lista)
- `publish_post`: se `len(photo_urls) > 1` → fluxo carrossel Meta API:
  1. Upload cada imagem → container individual
  2. Polling `status_code=FINISHED` em cada container
  3. Criar carousel container com lista de IDs
  4. Publicar carousel container
- **Complexity:** 5 pts | **Risk:** ALTO (mudança no pipeline principal + nova lógica Meta API)

**Quality Gates:**
- Pre-Commit: pipeline regression tests, Meta API mock coverage
- Pre-PR: full pipeline integration test com 2 fotos

---

### Story 6.3 — Frontend Multi-Foto (Frontend)
**Executor:** `@dev` | **Quality Gate:** `@architect`

- `PhotoUploadFAB`: aceita múltipla seleção (mobile: galeria multi-select; desktop: file picker multi)
- Preview de fotos selecionadas antes de enviar (thumbnails em linha, botão remover)
- `POST /content-requests` com `FormData` de N fotos
- Menu de Intenção exibe opções `Antes/Depois` e `Carrossel` quando 2+ fotos selecionadas
- `PostCard` no dashboard: exibe thumbnails múltiplos quando `photo_urls` tem 2+ itens
- **Complexity:** 3 pts | **Risk:** MÉDIO (mudança na UI do fluxo principal)

**Quality Gates:**
- Pre-Commit: TypeScript types corretos, build limpo
- Pre-PR: teste de upload com 2 fotos no ambiente de staging

---

## Sequência de Implementação

```
6.1 (Backend modelo + upload)
  ↓
6.2 (Pipeline + Meta API carrossel)
  ↓
6.3 (Frontend multi-foto)
```

6.1 bloqueia 6.2 e 6.3. 6.2 e 6.3 podem ser desenvolvidas em paralelo após 6.1.

---

## Compatibility Requirements

- [ ] `POST /content-requests` continua funcionando com 1 foto (retrocompat)
- [ ] `photo_url` (singular) mantido no modelo — nullable quando `photo_urls` em uso
- [ ] Pipeline de single-photo não é alterado para `content_type = post`
- [ ] Tokens e autenticação Meta não mudam

---

## Risk Mitigation

| Risco | Probabilidade | Mitigação |
|---|---|---|
| Meta API carousel limite (10 fotos max) | Baixa | Validar no frontend (máx 10) |
| Pipeline single-photo regride | Média | Testes de regressão obrigatórios em 6.2 |
| Migration quebra prod | Baixa | `photo_urls` nullable, `photo_url` preservado |
| Timeout no polling de N containers | Média | Timeout proporcional: `N * 60s` |

**Rollback:** `photo_urls` nullable — reverter migration é seguro. Pipeline condicional por `content_type`.

---

## Definition of Done

- [ ] Upload de 2 fotos gera carrossel publicado no Instagram
- [ ] Upload de 2 fotos com intenção `before_after` gera legenda narrativa comparativa
- [ ] Upload de 1 foto continua funcionando exatamente como antes (regressão zero)
- [ ] 90+ testes passando (inclui novos testes de pipeline multi-foto)
- [ ] Deploy Railway + Vercel sem downtime

---

## Handoff para @sm

> "Por favor, crie as stories detalhadas para este epic brownfield. Considerações chave:
>
> - Sistema existente: FastAPI + Celery + Next.js + Meta Graph API
> - Ponto crítico de integração: pipeline Celery existente em `app/tasks/pipeline.py`
> - Padrão de compatibilidade obrigatório: `POST /content-requests` com 1 foto deve continuar funcionando
> - Migration deve ser backward-compatible: `photo_urls JSONB nullable`
> - Meta API carousel: ver [documentação oficial](https://developers.facebook.com/docs/instagram-api/guides/content-publishing) — seção Carousel Posts
> - Cada story deve incluir verificação que o fluxo single-photo (Epic 1–5) continua intacto
>
> Sequência: 6.1 → (6.2 || 6.3) → deploy"
