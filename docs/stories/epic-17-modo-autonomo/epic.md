# Epic 17 — Modo Autônomo: Carrossel Temático

**Status:** Done
**Data de início:** 2026-06-10
**Prioridade:** Média
**Origem:** Análise arquitetural @aios-master 2026-06-10 — documento de visão de produto
**Dependência de entrada:** Epic 15 concluído; Epic 16 em paralelo

---

## Objetivo

Permitir que o usuário gere um carrossel completo sem enviar nenhuma foto. O usuário informa apenas o nicho e seleciona um tema pré-definido ("5 erros que encarecem uma reforma", "Antes de assinar um contrato com pedreiro..."). O sistema gera: estrutura de slides, copy de cada card e a legenda — o usuário apenas revisa e aprova.

---

## Problema

Hoje o pipeline inteiro começa com uma foto enviada pelo usuário. Isso limita a frequência de publicação: o usuário só posta quando tem uma foto nova disponível.

O documento de visão identificou que carrosséis temáticos (educativos, de dicas, de erros do nicho) podem ser gerados completamente pela IA, sem dependência de mídia do usuário. Esse formato tem alta taxa de salvamento e posiciona o cliente como especialista.

---

## Solução

Novo tipo de conteúdo: `autonomous_carousel`. Pipeline alternativo:

```
Usuário escolhe tema da biblioteca
  ↓
Backend gera estrutura de slides (título + texto por card)
  ↓
Designer gera imagem para cada card (texto sobre fundo de marca)
  ↓
Copywriter gera legenda principal
  ↓
Usuário aprova + publica
```

Sem upload de foto. Sem análise de imagem. O conteúdo parte do tema e do brand_profile do cliente.

---

## Entregas

| Story | Título | Esforço |
|-------|--------|---------|
| 17.1 | Biblioteca de Temas por Nicho (Backend) | Médio |
| 17.2 | Pipeline Autônomo sem Upload (Backend) | Grande |
| 17.3 | Interface de Seleção de Tema (Frontend) | Médio |
| 17.4 | Fluxo de Aprovação Autônomo (Frontend) | Médio |

---

## Arquitetura

### Novo `content_type`
- `autonomous_carousel` — processado por pipeline separado, sem upload de foto

### Novos agentes/funções
- `ThemeGenerator` — usa brand_profile + theme_id para gerar estrutura de slides via Claude
- `SlideDesigner` — gera imagem de cada card (texto sobre fundo, cores da marca)

### Sem mudanças em
- Pipeline de foto existente (Stories 1–14 preservadas integralmente)
- Modelo `ContentRequest` — adiciona campos opcionais apenas

---

## Definition of Done do Epic

- [ ] Usuário consegue selecionar um tema e receber carrossel gerado sem enviar foto
- [ ] Carrossel publicado com sucesso no Instagram via Meta API
- [ ] Pipeline de foto existente sem regressão
- [ ] Deploy Railway + Vercel sem downtime

---

## Risco Principal

**Geração de imagens de slide**: o `designer.py` atual processa fotos enviadas pelo usuário com Cloudflare Images. Para carrosséis autônomos, precisa gerar imagens com texto sobre fundo. Avaliar na Story 17.2 se usar: (a) geração de imagem via API (DALL-E/Ideogram), (b) template HTML→screenshot via Playwright, ou (c) Pillow (Python) com template de texto.

**Decisão arquitetural necessária na 17.2** — bloquear implementação até decisão aprovada.
