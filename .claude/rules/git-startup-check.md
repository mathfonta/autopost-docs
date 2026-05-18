# Git Startup Check — Regra Obrigatória de Sessão

## Propósito

Garantir que toda sessão começa com visibilidade completa do estado git dos repositórios do projeto.
Evita código "fantasma" — modificações sem story, sem commit, sem rastreabilidade.

---

## Quando executar

**SEMPRE** em qualquer destas situações:
- Início de nova sessão de trabalho
- Ativação de qualquer agente (`@dev`, `@qa`, `@devops`, `@security`, etc.)
- Antes de implementar qualquer feature ou fix
- Antes de encerrar a sessão

---

## Protocolo de verificação

### Passo 1 — Identificar repositórios do projeto

Cada projeto define seus próprios repos. Verificar:

```bash
# Repo principal (AIOX framework/docs)
git status --short
git log --oneline -3

# Repos adicionais (backend, frontend, etc.) — se existirem no projeto
# Configurar os paths abaixo ao iniciar um novo projeto:
# cd "{BACKEND_PATH}" && git status --short && git log --oneline -3
# cd "{FRONTEND_PATH}" && git status --short && git log --oneline -3
```

> **Configurar por projeto:** substituir `{BACKEND_PATH}` e `{FRONTEND_PATH}` pelos paths
> reais dos repos de código. Se o projeto for monorepo, verificar apenas a raiz.

### Passo 2 — Reportar estado

```
📊 Git Status — [DATA]

[nome-do-repo] ([github.com/mathfonta/repo-name])
  Branch: main | [N commits à frente do origin | SINCRONIZADO]
  Modificados sem commit: [lista ou "LIMPO"]
  Commits não pushed: [lista ou "NENHUM"]

⚠️ Atenção: [qualquer anomalia detectada]
```

---

## Regras de bloqueio

| Situação | Ação |
|----------|------|
| Arquivos modificados sem story associada | ALERTAR — criar story antes de commitar |
| Commits locais há mais de 1 sessão sem push | ALERTAR — acionar @devops |
| Branch divergida do origin | ALERTAR — investigar antes de continuar |

---

## Configuração para novo projeto

Ao iniciar trabalho em um novo projeto:
1. Verificar se tem `.git` na raiz e nas subpastas relevantes
2. Verificar se tem remote configurado (`git remote -v`)
3. Verificar último commit (`git log --oneline -3`)
4. Anotar os paths dos repos em um comentário neste arquivo ou no CLAUDE.md do projeto
5. Reportar estado antes de qualquer ação

---

## Integração com protocolo de sessão

Esta verificação é executada após a configuração de autenticação Git (`git-auth.md`):

```
1. [Opcional] Carregar Segundo Cérebro (Status + APRENDIZADOS + Ideas)
2. Executar git-auth.md (configurar credenciais)
3. Executar Git Startup Check ← esta regra
4. Confirmar ao usuário com o resumo completo
```
