# Epic 16 — Camada de Intenção de Marketing

**Status:** Draft
**Data de início:** 2026-06-10
**Prioridade:** Média-Alta
**Origem:** Análise arquitetural @aios-master 2026-06-10 — documento de visão de produto
**Dependência de entrada:** Epic 15 concluído

---

## Objetivo

Adicionar uma camada de **intenção de marketing** antes da seleção de formato e estratégia. O usuário escolhe o objetivo de negócio ("Quero gerar orçamentos") e o sistema recomenda automaticamente o formato e a estratégia mais adequados — sem eliminar o controle manual.

---

## Problema

Hoje o fluxo é: **Formato** (Feed/Carrossel/Reels/Story) → **Estratégia** (Prova Social, Bastidores...).

O usuário pensa em objetivos de negócio, não em formatos de Instagram. Ele quer saber se vai conseguir cliente, não se deve usar "Carousel" ou "Feed Photo". A escolha do formato cria fricção desnecessária para quem não tem intimidade com marketing digital.

---

## Solução

Inserir uma tela opcional antes do `ContentTypeBar`:

**"O que você quer com este post?"**
- Gerar orçamentos
- Ganhar novos seguidores
- Aumentar engajamento
- Construir autoridade
- Manter perfil ativo

O sistema mapeia a intenção → recomenda formato + estratégia → o usuário pode aceitar a recomendação ou navegar manualmente como antes.

A intenção também é passada ao `copywriter.py` como contexto extra, permitindo que a copy reflita o objetivo além da estratégia visual.

---

## Entregas

| Story | Título | Esforço |
|-------|--------|---------|
| 16.1 | Mapeamento Intenção → Estratégia (Config) | Pequeno |
| 16.2 | Tela de Seleção de Intenção (Frontend) | Médio |
| 16.3 | Intenção no Pipeline do Copywriter (Backend) | Médio |

---

## Fluxo Novo (Happy Path)

```
Dashboard
  ↓
[novo] IntentSelector — "O que você quer com este post?"
  ↓
ContentTypeBar — formato recomendado pré-selecionado (editável)
  ↓
SubStrategySelector — estratégia recomendada destacada (editável)
  ↓
Upload → Generate → Approval
```

O usuário pode pular o IntentSelector clicando "Escolher manualmente" — comportamento atual preservado.

---

## Definition of Done do Epic

- [ ] Tela de seleção de intenção funciona e recomenda formato + estratégia
- [ ] Intenção é passada ao copywriter e reflete no copy gerado
- [ ] Fluxo manual (sem intenção) funciona sem regressão
- [ ] Deploy Railway + Vercel sem downtime
