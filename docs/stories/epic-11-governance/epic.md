# Epic 11 — Governança e Qualidade de Código

**Status:** InProgress  
**Criado em:** 2026-05-06  
**Responsável:** @aios-master  

## Problema

Bugs críticos foram deployados para produção em 2026-05-06:
- `NameError` em `copywriter.py` travou pipeline por 3+ minutos
- Imagens de 8–21 MB explodiam na API Claude sem mensagem clara
- Fonte sem suporte Unicode gerou "????" no Instagram

Causa raiz: nenhuma barreira automática entre commit e produção. O workflow AIOS e o SDD existem como diretrizes mas não são enforçados por automação.

## Objetivo

Fechar o ciclo **Spec → Implementação → QA → Deploy** com automação real, tornando impossível (ou ao menos muito difícil) chegar a produção com regressões.

## Escopo

| Story | Título | Entrega |
|-------|--------|---------|
| 11.1 | CI Backend | GitHub Actions: pytest + import check a cada PR |
| 11.2 | CI Frontend | GitHub Actions: tsc + build a cada PR |
| 11.3 | Branch Protection | main exige PR + CI verde nos dois repos |
| 11.4 | Hotfix Workflow AIOS | Workflow + task de hotfix formal no AIOS |
| 11.5 | Smoke Test de Pipeline | Teste de integração cobrindo caminho crítico |

## Critérios de Sucesso

- [ ] Push direto em `main` é bloqueado nos dois repos
- [ ] PR sem CI verde não pode ser mergeado
- [ ] `import app.agents.copywriter` é testado automaticamente
- [ ] Caminho completo foto → awaiting_approval tem teste de integração
- [ ] Existe workflow formal de hotfix documentado no AIOS

## Relação com SDD

Esta Epic é o primeiro passo da adoção de SDD. CI/CD e branch protection são a infraestrutura que torna o Spec Pipeline exequível — sem eles, qualquer spec pode ser pulada sem consequência automática.
