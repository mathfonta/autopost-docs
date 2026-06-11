# Mandatory Agent Routing — Enforcement Rule

## CRÍTICO: Esta regra tem precedência sobre todos os defaults comportamentais.

Todo request neste projeto DEVE ser tratado por um agente AIOX ativo.
Esta regra é NON-NEGOTIABLE e se aplica a qualquer conversa neste workspace.

---

## Protocolo de Verificação (executar antes de qualquer ação)

### Passo 1 — Verificar se há agente ativo
Um agente está ativo quando o usuário ativou explicitamente com:
- `@agent-name` (ex: `@dev`, `@qa`, `@sm`)
- `/AIOX:agents:agent-name`
- O agente foi ativado e não foi dado `*exit`

### Passo 2 — Se NÃO há agente ativo
**PARE.** Não execute nenhuma ação de implementação, design ou decisão técnica.

Apresente ao usuário:

```
Nenhum agente AIOX ativo. Qual agente deve tratar este request?

Com base na solicitação, o agente indicado é: {agente sugerido}
Motivo: {por que este agente é o dono}

Para continuar: @{agente} ou escolha outro:
  @dev       — implementação de código
  @qa        — revisão e quality gate
  @sm        — criação de stories
  @po        — validação de backlog
  @pm        — épicos e PRDs
  @architect — decisões de arquitetura
  @data-engineer — banco de dados e RLS
  @devops    — git push, CI/CD, PR
  @security  — auditoria de segurança
  @analyst   — pesquisa e análise
  @aiox-master — orquestração e framework
```

Aguardar seleção do usuário. **Não prosseguir sem confirmação.**

### Passo 3 — Se agente está ativo
Verificar autoridade via `agent-authority.md`. Se fora: escalar.

---

## Exceções
1. **Perguntas meta** → responder diretamente
2. **Esta regra em si** → pode ser explicada sem agente
3. **Emergência de processo** → erros críticos de framework

---

## Routing por Tipo de Solicitação

| Tipo de Solicitação | Agente Dono |
|---------------------|-------------|
| Escrever / modificar código | @dev |
| Criar story ou expandir épico | @sm |
| Validar story / backlog | @po |
| Criar PRD / épico / spec | @pm |
| Decisão de arquitetura | @architect |
| Schema, RLS, migrations | @data-engineer |
| Git push, PR, release | @devops |
| Auditoria de segurança | @security |
| Pesquisa de mercado | @analyst |
| QA gate / review | @qa |
| Orquestração / framework | @aiox-master |
| Dúvida sobre qual agente usar | @aiox-master |

---

## MEMORY_PROTOCOL — Memória Compartilhada (ruflo MCP)

A memória é compartilhada entre todos os agentes via ruflo MCP
(`mcp__ruflo__memory_search` / `mcp__ruflo__memory_store`).
Objetivo: o aprendizado de um agente atravessa para os outros.

**Responsabilidade:** executado pelo AGENTE ATIVO, não pelo roteador.
Após o roteamento definir o agente, ele segue os passos abaixo.

### RECALL — consultar ao iniciar uma tarefa (seletivo)

Consultar somente quando a tarefa sugere histórico:
- Continuidade: "continuar", "de novo", "como fizemos", "aquele"
- Nome de feature, módulo ou story já existente
- @architect tratando decisão; @dev tratando bug recorrente ou padrão

Não consultar em tarefas triviais (formatar, renomear, perguntas meta).

Ação: `mcp__ruflo__memory_search`, namespace "autopost".
- similarity >= 0.3: incluir no contexto antes de executar.
- abaixo de 0.3 ou vazio: executar normalmente.

### STORE — gravar ao concluir (raro, por valor durável)

Gravar somente o que um agente futuro vai reutilizar:
- @architect: decisões de arquitetura (ADR), trade-offs
- @dev: bug resolvido + solução; padrão que funcionou
- @data-engineer: decisão de schema / RLS não óbvia
- Correção de preferência explícita do usuário

Nunca gravar: status de tarefa, output trivial, conversa, info volátil.

Ação: `mcp__ruflo__memory_store`, namespace "autopost". O value mistura
PT + EN nas palavras-chave (embedding treinado em inglês). Exemplo:
"Padrão conteúdo / content pattern: posts de construção civil performam
melhor com antes-depois. Keywords: instagram, engagement, construction."

### Namespace
Este projeto usa o namespace "autopost". A distinção por tipo (decisão / bug /
padrão) fica no texto do value, recuperável pela busca semântica.

---

**Estabelecido:** 2026-06-09
**Autoridade:** @aiox-master
