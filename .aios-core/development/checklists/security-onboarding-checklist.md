# Security Checklist — Onboarding (Primeiro Audit de um Projeto)

**Agente:** @security (Sage)
**Uso:** Executar UMA vez ao começar a auditar um projeto pela primeira vez.
**Objetivo:** Estabelecer a baseline de segurança e popular o MEMORY.md inicial.

---

## Fase 1 — Mapeamento de Superfície

- [ ] Listar todos os endpoints da API (ler `app/api/` ou equivalent)
- [ ] Listar todas as tabelas do banco de dados (ler migrations ou schema)
- [ ] Listar todas as integrações externas (APIs de terceiros, webhooks, queues)
- [ ] Listar todos os serviços de infraestrutura (hosting, storage, CDN, cache)
- [ ] Listar todos os dados sensíveis processados (PII, tokens, secrets)
- [ ] Identificar quem são os usuários do sistema e seus níveis de acesso

## Fase 2 — Inventário de Secrets

- [ ] Listar todas as variáveis de ambiente sensíveis esperadas
- [ ] Verificar que `.env` está no `.gitignore`
- [ ] Verificar que não há secrets em arquivos de configuração commitados
- [ ] Documentar onde cada secret é usado no código

## Fase 3 — Modelo de Ameaça Inicial

- [ ] Identificar os 3 assets mais críticos do sistema (o que seria mais catastrófico expor?)
- [ ] Para cada asset: qual é o vetor de ataque mais provável?
- [ ] Documentar no MEMORY.md como "Known Risk Areas (baseline)"

## Fase 4 — Primeira Execução dos Checks

- [ ] Executar `*secrets-check` — encontrar e documentar qualquer exposição
- [ ] Executar `*rls-review` — verificar todas as tabelas
- [ ] Executar `*audit infra` — verificar configurações de infraestrutura

## Fase 5 — Baseline MEMORY.md

Preencher as seções do MEMORY.md:
- [ ] Known Risk Areas table com severidade inicial
- [ ] Padrões de verificação específicos deste projeto
- [ ] Primeira entrada no Audit History

## Fase 6 — Relatório de Onboarding

Criar `docs/qa/security-reports/security-onboarding-{YYYY-MM-DD}.md` com:
- [ ] Superfície de ataque mapeada
- [ ] Top 5 riscos identificados com severidade
- [ ] Roadmap de hardening sugerido (priorizado por risco)
- [ ] Stories de remediação criadas para CRITICAL e HIGH findings

---

## AutoPost — Onboarding Já Realizado

> **Status:** Onboarding baseline documentado em 2026-04-25 via ADS @security.
> Known Risk Areas já populadas no MEMORY.md inicial.
> Próximo passo: executar `*audit full` para validar baseline com verificações reais.
