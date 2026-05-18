# IMPROVEMENTS — O que foi adicionado ao vanilla AIOS

Changelog das melhorias acumuladas neste framework pessoal.
Cada entrada tem o projeto de origem e o motivo da adição.

---

## Agentes

### @security (Sage) — Auditoria de Segurança
**Origem:** AutoPost (Epic 7, 2026-04)
**Motivo:** Necessidade de auditoria formal de segurança em repos com dados sensíveis e APIs externas.
**O que faz:**
- Threat modeling de features antes do desenvolvimento
- Audit de dependências (npm audit + pip-audit)
- Verificação de secrets no codebase
- RLS review para projetos Supabase
- Security report gerado como artefato formal

**Tasks incluídas:**
- `.aios-core/development/tasks/security-audit.md`
- `.aios-core/development/tasks/security-scan.md`
- `.aios-core/development/tasks/security-secrets-check.md`
- `.aios-core/development/tasks/security-threat-model.md`
- `.aios-core/development/tasks/security-rls-review.md`
- `.aios-core/development/tasks/security-python-dep-audit.md`

---

## Protocolos e Rules

### git-auth.md — Autenticação Automática para Repos Privados
**Origem:** AutoPost (2026-04)
**Motivo:** Repos privados exigem autenticação GitHub em cada sessão do sandbox Claude Code.
**O que faz:** Lê `.github-token` na raiz do projeto e configura credenciais no sandbox silenciosamente.
**Como usar:** Configurar path do token por projeto. Token fica no `.gitignore`.

### git-startup-check.md — Visibilidade de Estado ao Iniciar Sessão
**Origem:** AutoPost (2026-04)
**Motivo:** Evitar código "fantasma" — modificações sem story, sem commit, sem rastreabilidade.
**O que faz:** Verifica git status de todos os repos do projeto e reporta ao usuário antes de qualquer ação.
**Como usar:** Configurar paths dos repos no protocolo por projeto.

### tool-response-filtering.md — Filtragem de Respostas de Tools
**Origem:** AutoPost (2026-04)
**Motivo:** Respostas muito longas de tools podem saturar o contexto desnecessariamente.
**O que faz:** Define regras para filtrar e sumarizar respostas de tools antes de processar.

---

## Workflows

### hotfix.yaml — Protocolo Formal de Hotfix
**Origem:** AutoPost (Epic 11, 2026-05)
**Motivo:** Necessidade de processo estruturado para bugs críticos em produção sem quebrar o fluxo normal.
**O que faz:** Workflow de hotfix com spec, implementação e rollback documentados.
**Task associada:** `.aios-core/development/tasks/hotfix-spec.md`

### epic-orchestration.yaml — Orquestração de Épicos
**Origem:** AutoPost
**Motivo:** Coordenar múltiplas stories dentro de um épico com rastreabilidade.

### qa-loop.yaml — Ciclo Iterativo de QA
**Origem:** AutoPost
**Motivo:** Permitir iterações QA-dev sem sair do fluxo principal.

### spec-pipeline.yaml — Pipeline de Especificação
**Origem:** AutoPost
**Motivo:** Formalizar a criação de specs antes da implementação em features complexas.

---

## Como contribuir com este changelog

Ao adicionar algo novo ao framework (task, rule, workflow, agente), adicionar uma entrada aqui com:
- **Nome** — o que foi adicionado
- **Origem** — qual projeto originou a necessidade
- **Motivo** — por que foi adicionado (o problema que resolve)
- **O que faz** — descrição concisa
- **Como usar** — instruções de configuração se necessário
