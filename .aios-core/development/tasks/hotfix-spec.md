# Task: Hotfix Spec — Documentação de Bug Crítico

**Quando usar:** antes de qualquer implementação de hotfix  
**Tempo esperado:** 5 minutos  
**Output:** arquivo `docs/qa/hotfixes/hotfix-{YYYY-MM-DD}-{slug}.md`

---

## Instrução de Execução

Antes de escrever uma linha de código, crie o arquivo de spec abaixo.
Preencha todos os 5 campos obrigatórios. Se não conseguir preencher algum,
é sinal de que ainda não entendeu o problema — investigue mais antes de codar.

---

## Template

```markdown
# Hotfix Spec — {título curto}

**Data:** {YYYY-MM-DD}  
**Autor:** {agente ou desenvolvedor}  
**Status:** OPEN | RESOLVED  
**Resolvido em:** {data ou —}

## 1. Sintoma (o que o usuário vê)

{Descreva exatamente o que aparece para o usuário. Mensagem de erro,
comportamento incorreto, o que ficou parado. Seja concreto.}

## 2. Causa Raiz (por que acontece)

{Arquivo, linha, função. Explique a causa técnica real — não o sintoma.
"Não sei" não é aceito. Investigue até saber.}

Exemplo: `app/agents/copywriter.py:264 — _VOICE_TONE_MAP definido após
_STATIC_LIBRARY que já o referenciava, causando NameError no import.`

## 3. O que vai mudar (escopo da correção)

{Lista dos arquivos e linhas que serão modificados. Nada além disso.
Se precisar mudar mais, considere se é realmente um hotfix ou um SDC.}

- [ ] `arquivo.py:linha` — descrição da mudança

## 4. Como testar (smoke test)

{Passos exatos para confirmar que o bug foi corrigido e não regrediu.
Deve ser executável em menos de 5 minutos.}

1. {passo 1}
2. {passo 2}
3. Resultado esperado: {o que deve acontecer}

## 5. Rollback (se der errado)

{Como reverter se o hotfix piorar a situação.}

Exemplo: `git revert {hash}` + `git push origin main`
Ou: rollback no Railway para o deploy anterior.
```

---

## Regras

- Spec criado ANTES do primeiro commit de código
- Arquivado em `docs/qa/hotfixes/` e comitado junto ao fix
- Campo "Status" atualizado para RESOLVED após deploy confirmado
- Se causa raiz não couber no hotfix (ex: refactor maior necessário),
  criar story de follow-up e documentar como tech debt
