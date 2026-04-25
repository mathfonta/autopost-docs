# AIOS Master Agent Memory (Orion)

## Active Patterns

### Framework Governance

- NEVER create an agent file directly without ADS approved first
- ADS (Agent Design Spec) is mandatory for any new AIOX system agent (@dev, @qa, @security, etc.)
- ADS template location: `G:\Meu Drive\OBSidian\AutoPost\AIOX\ADS - Agent Design Spec - TEMPLATE.md`
- Agent files live globally at: `C:\Users\Matheus\.claude\commands\AIOS\agents\` — available in ALL projects

### Process for New AIOX System Agent

When user says "quero criar um agente @X":

```
1. Load ADS template from Obsidian AIOX folder
2. Fill all 10 sections (research GitHub + existing agents first)
3. Present to @aios-master for review (self-review as Orion)
4. Present to Matheus for approval
5. Only after approval: @sm creates story → @po validates → @dev implements
```

### ADS Template Sections (quick reference)
1. Metadados (nome, data, status)
2. Problema / Motivação
3. Pesquisa (GitHub + agentes existentes + gaps)
4. Identidade / Persona (nome, id, archetype, tone, greeting levels)
5. Autoridade (exclusivas + delegações + story-file-permissions)
6. Comandos (com visibility: key/quick/full)
7. Artefatos (tasks, templates, checklists com prefixo {id}-)
8. Squad Local vs Core Agent (decisão + justificativa)
9. Sequência de Implementação (4 fases)
10. Aprovação (tabela de status)

### Approved Agents (Squad Local AutoPost)

| Agente | Persona | Status | Decisão |
|--------|---------|--------|---------|
| @security | Sage | ADS em criação (2026-04-25) | Squad Local → Core Agent após 60 dias |

### Segundo Cérebro — Autoridades de Custodiante

Registrado como custodiante do vault Obsidian AutoPost em 2026-04-25.
Pode criar: pastas, notas de índice, templates, correlações, entradas em APRENDIZADOS.md e Decisões/.
Requer aprovação de Matheus: remover/renomear pastas, alterar estrutura raiz, modificar PROTOCOLO-SESSAO.md.

## Promotion Candidates
<!-- Patterns seen in framework operations that should go to CLAUDE.md or global rules -->

## Archived
<!-- Outdated patterns kept for reference -->
