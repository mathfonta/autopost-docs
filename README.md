# AIOX Pessoal — mathfonta

Base pessoal do framework AIOS, acumulando melhorias entre projetos.

## O que é este repo

Este repositório é o **ponto de partida para novos projetos** com o framework AIOS.
Cada projeto copia ou clona daqui e herda todas as melhorias acumuladas.

**Não é um projeto de produto.** É um framework evolutivo.

## Como usar em um novo projeto

```bash
# 1. Clonar como base
git clone https://github.com/mathfonta/equipe-aiox.git meu-novo-projeto

# 2. Remover o histórico e iniciar fresh
cd meu-novo-projeto
rm -rf .git
git init
git remote add origin https://github.com/mathfonta/meu-novo-projeto.git

# 3. Configurar por projeto (substituir placeholders)
# - .claude/CLAUDE.md → {OBSIDIAN_VAULT}, {PROJECT_ROOT}
# - .claude/rules/git-auth.md → caminho do .github-token
# - .claude/rules/git-startup-check.md → paths dos repos

# 4. Criar .github-token (não commitado)
echo "ghp_SEU_TOKEN_AQUI" > .github-token

# 5. Primeiro commit
git add .
git commit -m "chore: initialize project — Synkra AIOS"
```

## O que já está incluído (vs vanilla AIOS)

Ver [IMPROVEMENTS.md](IMPROVEMENTS.md) para o changelog completo.

Resumo:
- **@security agent** — Sage, com tasks de audit, threat modeling, secrets check, RLS review
- **Protocolo git privado** — autenticação automática por sessão
- **Git startup check** — visibilidade de estado ao iniciar sessão
- **Workflow hotfix** — protocolo formal para bugs críticos em produção
- **Workflows adicionais** — epic-orchestration, qa-loop, spec-pipeline
- **Tool response filtering** — regra para filtrar respostas de tools

## Estrutura

```
equipe-aiox/
├── .aios-core/          # Framework AIOS (L1-L2: não modificar; L3: configurar)
├── .claude/
│   ├── CLAUDE.md        # Regras do projeto (editar por projeto)
│   ├── rules/           # Regras contextuais (maioria genérica, algumas configuram por projeto)
│   └── settings.json    # Permissões Claude Code
├── docs/
│   ├── architecture/    # Documentação de arquitetura
│   ├── qa/              # QA gates e security reports
│   └── stories/         # Stories do projeto (criadas durante desenvolvimento)
├── AGENTS.md            # Instruções para Codex CLI
├── IMPROVEMENTS.md      # Changelog de melhorias vs vanilla AIOS
└── README.md            # Este arquivo
```

## Filosofia

> Cada projeto ensina algo. Este repo garante que o próximo começa mais inteligente.

Quando descobrir algo não óbvio (protocolo, regra, task), adicionar aqui.
Quando começar um novo projeto, puxar daqui primeiro.
