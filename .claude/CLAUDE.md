# Synkra AIOS Development Rules for Claude Code

You are working with Synkra AIOS, an AI-Orchestrated System for Full Stack Development.

---

## Segundo Cérebro — Obsidian (Pull Model)

**Regras de carregamento (não automático — pull on demand):**

| Gatilho | Ação | Custo |
|---------|------|-------|
| "carrega status do [projeto]" | Lê `{OBSIDIAN_VAULT}\📊 Status.md` | ~80 tokens |
| "tem ideias novas" | Lê `{OBSIDIAN_VAULT}\💡 Ideas\Novas.md` | ~200 tokens |
| `*cerebro` | Lê `{OBSIDIAN_VAULT}\🧠 Cerebro\APRENDIZADOS.md` | ~300 tokens |
| `*cerebro-update "texto"` | Appenda em `APRENDIZADOS.md` | escrita |
| `*ideia-salvar` | Cria `ideia-{slug}.md` + atualiza `Novas.md` | escrita |

> **Configurar por projeto:** substituir `{OBSIDIAN_VAULT}` pelo path real no início do projeto.
> Exemplo: `G:\Meu Drive\OBSidian\NomeDoProjeto\`

**Nunca carregar automaticamente no início de sessão.**
Carregar somente quando o usuário acionar um gatilho acima.

---

## Git Authentication (Repos Privados)

**Protocolo automático no início de sessão:**

Ver `.claude/rules/git-auth.md` para o protocolo completo.

> **Configurar por projeto:** o token fica em `{PROJECT_ROOT}\.github-token` (no .gitignore).
> Substituir `{PROJECT_ROOT}` pelo path local do projeto.

---

## Git Startup Check

**Ver:** `.claude/rules/git-startup-check.md`

Executar ao inicio de sessão para garantir visibilidade do estado dos repositórios do projeto.

---

<!-- AIOS-MANAGED-START: core-framework -->
## Core Framework Understanding

Synkra AIOS is a meta-framework that orchestrates AI agents to handle complex development workflows. Always recognize and work within this architecture.
<!-- AIOS-MANAGED-END: core-framework -->

<!-- AIOS-MANAGED-START: constitution -->
## Constitution

O AIOS possui uma **Constitution formal** com princípios inegociáveis e gates automáticos.

**Documento completo:** `.aios-core/constitution.md`

**Princípios fundamentais:**

| Artigo | Princípio | Severidade |
|--------|-----------|------------|
| I | CLI First | NON-NEGOTIABLE |
| II | Agent Authority | NON-NEGOTIABLE |
| III | Story-Driven Development | MUST |
| IV | No Invention | MUST |
| V | Quality First | MUST |
| VI | Absolute Imports | SHOULD |

**Gates automáticos bloqueiam violações.** Consulte a Constitution para detalhes completos.
<!-- AIOS-MANAGED-END: constitution -->

<!-- AIOS-MANAGED-START: sistema-de-agentes -->
## Sistema de Agentes

### Ativação de Agentes
Use `@agent-name` ou `/AIOS:agents:agent-name`:

| Agente | Persona | Escopo Principal |
|--------|---------|------------------|
| `@dev` | Dex | Implementação de código |
| `@qa` | Quinn | Testes e qualidade |
| `@architect` | Aria | Arquitetura e design técnico |
| `@pm` | Morgan | Product Management |
| `@po` | Pax | Product Owner, stories/epics |
| `@sm` | River | Scrum Master |
| `@analyst` | Alex | Pesquisa e análise |
| `@data-engineer` | Dara | Database design |
| `@ux-design-expert` | Uma | UX/UI design |
| `@devops` | Gage | CI/CD, git push (EXCLUSIVO) |
| `@security` | Sage | Auditoria de segurança e threat modeling |

### Comandos de Agentes
Use prefixo `*` para comandos:
- `*help` - Mostrar comandos disponíveis
- `*create-story` - Criar story de desenvolvimento
- `*task {name}` - Executar task específica
- `*exit` - Sair do modo agente
<!-- AIOS-MANAGED-END: sistema-de-agentes -->

<!-- AIOS-MANAGED-START: agent-system -->
## Agent System

### Agent Activation
- Agents are activated with @agent-name syntax: @dev, @qa, @architect, @pm, @po, @sm, @analyst
- The master agent is activated with @aios-master
- Agent commands use the * prefix: *help, *create-story, *task, *exit

### Agent Context
When an agent is active:
- Follow that agent's specific persona and expertise
- Use the agent's designated workflow patterns
- Maintain the agent's perspective throughout the interaction
<!-- AIOS-MANAGED-END: agent-system -->

## Development Methodology

### Story-Driven Development
1. **Work from stories** - All development starts with a story in `docs/stories/`
2. **Update progress** - Mark checkboxes as tasks complete: [ ] → [x]
3. **Track changes** - Maintain the File List section in the story
4. **Follow criteria** - Implement exactly what the acceptance criteria specify

### Code Standards
- Write clean, self-documenting code
- Follow existing patterns in the codebase
- Include comprehensive error handling
- Add unit tests for all new functionality
- Use TypeScript/JavaScript best practices

### Testing Requirements
- Run all tests before marking tasks complete
- Ensure linting passes: `npm run lint`
- Verify type checking: `npm run typecheck`
- Add tests for new features
- Test edge cases and error scenarios

<!-- AIOS-MANAGED-START: framework-structure -->
## AIOS Framework Structure

```
.aios-core/
├── development/
│   ├── agents/         # Agent persona definitions
│   ├── tasks/          # Executable task workflows
│   ├── workflows/      # Multi-step workflow definitions
│   ├── templates/      # Document and code templates
│   └── checklists/     # Validation and review checklists
├── core/               # Framework core (L1 — não modificar)
└── data/               # Project config (L3 — mutável)

docs/
├── stories/            # Development stories (numbered)
├── prd/                # Product requirement documents
├── architecture/       # System architecture documentation
└── qa/                 # QA gates and security reports
```
<!-- AIOS-MANAGED-END: framework-structure -->

<!-- AIOS-MANAGED-START: framework-boundary -->
## Framework vs Project Boundary

O AIOS usa um modelo de 4 camadas (L1-L4):

| Camada | Mutabilidade | Paths |
|--------|-------------|-------|
| **L1** Framework Core | NEVER modify | `.aios-core/core/`, `.aios-core/constitution.md` |
| **L2** Framework Templates | NEVER modify | `.aios-core/development/tasks/`, `workflows/`, `templates/` |
| **L3** Project Config | Mutable | `.aios-core/data/`, `core-config.yaml` |
| **L4** Project Runtime | ALWAYS modify | `docs/stories/`, `packages/`, `tests/` |
<!-- AIOS-MANAGED-END: framework-boundary -->

<!-- AIOS-MANAGED-START: rules-system -->
## Rules System

| Rule File | Description |
|-----------|-------------|
| `agent-authority.md` | Agent delegation matrix and exclusive operations |
| `agent-handoff.md` | Agent switch compaction protocol |
| `coderabbit-integration.md` | Automated code review integration |
| `git-auth.md` | Git authentication for private repos (configurar path por projeto) |
| `git-startup-check.md` | Git status verification at session start (configurar repos por projeto) |
| `ids-principles.md` | Incremental Development System principles |
| `mcp-usage.md` | MCP server usage rules |
| `story-lifecycle.md` | Story status transitions and quality gates |
| `tool-response-filtering.md` | Tool response filtering rules |
| `workflow-execution.md` | 4 primary workflows |
<!-- AIOS-MANAGED-END: rules-system -->

## Git & GitHub Integration

### Commit Conventions
- Use conventional commits: `feat:`, `fix:`, `docs:`, `chore:`, etc.
- Reference story ID: `feat: implement feature [Story X.Y]`
- Keep commits atomic and focused

### GitHub CLI Usage
- Ensure authenticated: `gh auth status`
- Use for PR creation: `gh pr create`

## Claude Code Specific Configuration

### Tool Usage Guidelines
- Always use the Grep tool for searching, never `grep` or `rg` in bash
- Use the Task tool for complex multi-step operations
- Batch file reads/writes when processing multiple files
- Prefer editing existing files over creating new ones

### Session Management
- Track story progress throughout the session
- Update checkboxes immediately after completing tasks
- Save important state before long-running operations

---
*Synkra AIOS Personal Base — mathfonta*
