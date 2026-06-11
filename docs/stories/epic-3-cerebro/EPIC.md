# Epic 3 — Segundo Cérebro Global

**Status:** Ready  
**Criado:** 2026-04-16  
**Owner:** @pm (Morgan)

---

## Objetivo

Criar uma camada de aprendizado que transcende projetos individuais. O conhecimento acumulado no AutoPost (padrões que funcionam, lições técnicas, decisões de arquitetura) é promovido para um cérebro global acessível em qualquer sessão Claude Code — de qualquer projeto futuro.

## Problema que resolve

Hoje cada projeto começa do zero. Quando o AutoPost aprende que "foto com rosto = 3x mais curtidas às 15h", esse conhecimento morre quando você abre um novo projeto. O cerebro global garante que cada novo projeto comece no nível de expertise do projeto anterior.

## Valor esperado

```
Projeto 1 (AutoPost):   Aprende do zero → acumula padrões
Projeto 2 (futuro):     Começa com padrões do projeto 1 → 30% melhor desde o início
Projeto 3 (futuro):     Começa com padrões de P1+P2 → Expert desde o dia 1
```

## Arquitetura

```
~/.cerebro-global/                    ← acessível em qualquer sessão
├── VISAO.md                          ← seus objetivos de longo prazo
├── PADROES_GLOBAIS.md                ← o que funciona em TODOS os projetos
├── APRENDIZADOS.md                   ← lições técnicas e estratégicas
└── projetos/
    ├── autopost.md                   ← resumo do aprendizado do AutoPost
    └── [novo-projeto].md             ← criado quando novo projeto inicia

C:\projetos\{projeto}\.cerebro-{projeto}/   ← por projeto (Story 2.8)
├── PADROES.md
├── INSIGHTS.md
└── historico/
```

## Stories

| Story | Título | Depende de | Pontos |
|-------|--------|-----------|--------|
| 3.1 | Estrutura Global + Bootstrap | — | 2 |
| 3.2 | Motor de Promoção de Padrões | 2.8, 4.1 | 3 |
| 3.3 | Integração AIOS Agents | 3.1 | 2 |

**Total: 7 pontos**

---

## Change Log

| Data | Autor | O que mudou |
|------|-------|-------------|
| 2026-04-16 | @pm | Epic criado |
