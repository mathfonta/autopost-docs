# Git Authentication — Configuração Automática por Sessão

## Propósito

Configurar autenticação GitHub automaticamente no início de cada sessão,
permitindo que @devops execute `git push` sem intervenção manual.

> **Uso em repos privados:** essencial para projetos com código proprietário.

---

## Configuração por projeto

1. Criar arquivo `{PROJECT_ROOT}\.github-token` com o PAT do GitHub
2. Adicionar ao `.gitignore` do projeto: `.github-token`
3. O agente lê o token e configura as credenciais no sandbox

---

## Protocolo de execução

### Passo 1 — Localizar o token

Leia o arquivo `.github-token` na raiz do projeto atual.

- Se não existir → avisar discretamente e seguir sem push automático
- Se contiver o placeholder `ghp_COLE_SEU_TOKEN_AQUI` → mesma ação
- Se contiver token válido (começa com `ghp_`) → executar Passo 2

### Passo 2 — Configurar git no sandbox

Execute via bash (substituindo `{TOKEN}` pelo conteúdo do arquivo, **sem exibir o token no chat**):

```bash
git config --global user.email "matheusfontanellaaugusto@gmail.com"
git config --global user.name "mathfonta"
git config --global credential.helper store
echo "https://{TOKEN}@github.com" > ~/.git-credentials
chmod 600 ~/.git-credentials
```

### Passo 3 — Confirmar silenciosamente

Adicionar ao resumo de sessão apenas:
```
🔑 Git auth: configurado ✅
```

Ou se não configurado:
```
🔑 Git auth: .github-token não encontrado — push manual necessário
```

**Nunca exibir o token ou parte dele no chat.**

---

## Segurança

- O token fica apenas em memória do sandbox durante a sessão
- `~/.git-credentials` existe apenas no sandbox Linux isolado
- O arquivo `.github-token` nunca sai do computador (está no .gitignore)
- Em caso de comprometimento: revogar o token em https://github.com/settings/tokens

## Como gerar o token

1. Acesse: https://github.com/settings/tokens
2. *Generate new token (classic)*
3. Scope mínimo: `repo`
4. Cole o token gerado no arquivo `.github-token` na raiz do projeto
