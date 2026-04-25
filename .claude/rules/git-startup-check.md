# Git Startup Check — Regra Obrigatória de Sessão

## Propósito

Garantir que toda sessão começa com visibilidade completa do estado git dos repositórios do projeto. Evita código "fantasma" — modificações sem story, sem commit, sem rastreabilidade.

---

## Quando executar

**SEMPRE** — em qualquer uma destas situações:
- Início de nova sessão de trabalho
- Ativação de qualquer agente (`@dev`, `@qa`, `@devops`, `@security`, etc.)
- Antes de implementar qualquer feature ou fix
- Antes de encerrar a sessão

---

## Protocolo de verificação (executar via Bash)

```bash
# Backend
cd "c:\Projetos\autopost\backend"
git status --short
git log --oneline origin/main..HEAD 2>/dev/null || git log --oneline -3

# Frontend
cd "c:\Projetos\autopost\frontend"
git status --short
git log --oneline origin/main..HEAD 2>/dev/null || git log --oneline -3
```

---

## Formato de reporte obrigatório

Após executar as verificações, sempre reportar no seguinte formato:

```
📊 Git Status — [DATA]

backend/ (mathfonta/autopost-backend)
  Branch: main | [N commits à frente do origin]
  Modificados sem commit: [lista ou "LIMPO"]
  Commits não pushed: [lista ou "NENHUM"]

frontend/ (mathfonta/autopost-frontend)
  Branch: main | [N commits à frente do origin]
  Modificados sem commit: [lista ou "LIMPO"]
  Commits não pushed: [lista ou "NENHUM"]

⚠️ Atenção: [qualquer anomalia detectada]
✅ AIOX root: autopost-aiox repo [SINCRONIZADO / PENDENTE]
```

---

## Regras de bloqueio

| Situação | Ação |
|----------|------|
| Arquivos modificados sem story associada | ALERTAR — criar story antes de commitar |
| Commits locais há mais de 1 sessão sem push | ALERTAR — acionar @devops |
| Branch divergida do origin | ALERTAR — investigar antes de continuar |
| AIOX root sem commit recente | ALERTAR — commitar docs/stories |

---

## Repositórios do projeto AutoPost

| Repositório | Path local | Remote |
|-------------|-----------|--------|
| autopost-backend | `c:\Projetos\autopost\backend\` | `github.com/mathfonta/autopost-backend` |
| autopost-frontend | `c:\Projetos\autopost\frontend\` | `github.com/mathfonta/autopost-frontend` |
| autopost-aiox | `c:\Projetos\autopost\` (raiz) | `github.com/mathfonta/autopost-aiox` *(a criar)* |

---

## Integração com PROTOCOLO-SESSAO.md

Esta verificação git é executada **após** a leitura do Segundo Cérebro (Status + APRENDIZADOS + Ideas), como Passo 4 do protocolo de sessão:

```
1. Ler Status do Projeto.md
2. Ler APRENDIZADOS.md
3. Ler Ideas.md
4. [NOVO] Executar Git Startup Check ← esta regra
5. Confirmar ao usuário com o resumo completo
```

---

## Para projetos futuros (quando iniciados)

Ao iniciar trabalho em qualquer projeto novo ou retomado:
1. Verificar se tem `.git` na raiz e nas subpastas
2. Verificar se tem remote configurado (`git remote -v`)
3. Verificar último commit (`git log --oneline -3`)
4. Reportar estado antes de qualquer ação

Projetos iniciados mas não ativos: verificar ao menos o `git status` para garantir que não há código não salvo.
