# Investigação de Código Fantasma — Backend AutoPost

> **Executor:** @devops (Gage)
> **Data:** 2026-07-08
> **Origem:** encontrado no fechamento (`*close-story`) da Story 21.1, Epic 21 — Copy-Chief
> **Ação tomada:** `git stash` (não commitado, não descartado)

---

## 1. O que foi encontrado

No fechamento da Story 21.1, a verificação de DoD do @po acusou `backend/app/agents/copywriter.py` como modificado (`git status`), quando a story não deveria ter tocado nenhum arquivo de produção. Investigação revelou que **não era 1 mudança pequena — eram 5 mudanças diferentes empacotadas juntas**, sem story, sem branch própria, sentadas no working tree de `main` (não commitadas, não pushadas):

| # | Mudança | Arquivo(s) | Natureza |
|---|---|---|---|
| 1 | `strip_json_fences()` extraído para `app/core/ai_parsing.py` (novo arquivo) | `analyst.py`, `copywriter.py`, `theme_generator.py` | DRY — elimina duplicação de 3 blocos idênticos |
| 2 | **`ANALYST_PROVIDER`** — novo provider de visão: `_call_gemini_vision()` + `_resolve_analyst_provider()`, Gemini como alternativa ao Claude Haiku na análise de foto | `analyst.py`, `config.py` | Feature nova, não documentada em nenhuma story |
| 3 | `_wait_for_container_finished()` — deduplica o polling de status da Meta Graph API (4 funções de `publisher.py` repetiam o mesmo loop de 12-30 tentativas) | `publisher.py` | Refactor |
| 4 | Cliente Anthropic síncrono → assíncrono | `theme_generator.py` | Fix (bug: bloqueava o event loop) |
| 5 | **`COPY_PROVIDER` default: `"claude"` → `"gemini"`** | `config.py` | ⚠️ Muda comportamento de produção — não é refactor |

## 2. Verificação — impacto real (testado, não presumido)

Rodei a suite de testes com e sem o diff (`git stash` / `git stash pop`) para isolar causa e efeito:

| Cenário | Resultado |
|---|---|
| `main` como está commitado (sem o diff) | 9 falhas — todas em `test_analyst_agent.py`, `httpx.ConnectError`. **Pré-existente, anterior a esta sessão, sem relação com o Epic 21 ou com o diff investigado.** |
| Com o diff pendente aplicado | **28 falhas** — as 9 anteriores + **19 novas**, cobrindo 100% de `test_copywriter_agent.py` (14 casos) e `test_copywriter_strategy.py` (5 casos), que passavam antes do diff |

**Causa raiz confirmada:** a mudança #5 (`COPY_PROVIDER` default → `"gemini"`) desvia `generate_copy_with_ai()` para `_call_gemini_for_copy()`. Os testes mockam apenas o cliente Anthropic (`app.agents.copywriter.anthropic.AsyncAnthropic`) — com o path do Gemini ativo por default, o mock não intercepta, e o teste tenta uma chamada de rede real. Sem `GEMINI_API_KEY` no `.env` local (confirmado ausente), a chamada falha com `httpx.ConnectError`.

**Por que isso importava para o Epic 21:** a Story 21.2 vai editar `copywriter.py` e depende de `test_copywriter_agent.py` + `test_copywriter_strategy.py` passando como critério de regressão (AC8 da 21.2). Se este diff tivesse sido commitado sem correção, a 21.2 teria herdado 19 testes quebrados que não têm nada a ver com a mudança de prompts.

## 3. Ação tomada

```
git stash push -u -m "WIP: ANALYST_PROVIDER (Gemini vision) + COPY_PROVIDER default->gemini +
strip_json_fences DRY + publisher polling dedup — NAO commitar sem story formal, GEMINI_API_KEY
e correcao dos mocks de teste (quebra 19 testes: test_copywriter_agent.py + test_copywriter_strategy.py).
Encontrado no fechamento da Story 21.1 (Epic 21), 2026-07-08."
```

**Confirmado após o stash:**
- `git status --short` → limpo
- `test_copywriter_agent.py` + `test_copywriter_strategy.py` → **26/26 passando**
- `copywriter.py` liberado, sem contaminação, para a Story 21.2

**O trabalho não foi perdido** — está em `git stash list` (`stash@{0}`), recuperável com `git stash pop` quando alguém formalizar isso como story.

## 4. Recomendação — como resgatar este trabalho no futuro

Quando alguém quiser retomar (ex: para avaliar Gemini como provider de visão, que é uma ideia com mérito — mais barato que Claude Haiku):

1. `git stash pop` numa branch própria (`feature/analyst-provider-gemini` — não em `main` direto, ver achado 5.3 abaixo)
2. Separar em commits/stories distintas: (a) o DRY refactor `strip_json_fences` é seguro e pequeno, pode ir sozinho; (b) `ANALYST_PROVIDER` + `_wait_for_container_finished` são features/fixes que merecem story própria com testes atualizados; (c) a troca de `COPY_PROVIDER` default é uma **decisão de produto**, não deveria estar embutida silenciosamente num refactor — se a intenção é migrar o default para Gemini, isso merece o mesmo rigor do Epic 25 do Plano Mestre (avaliação A/B antes de trocar default)
3. Configurar `GEMINI_API_KEY` no `.env` local antes de rodar os testes
4. Atualizar os mocks de `test_copywriter_agent.py`/`test_copywriter_strategy.py`/`test_analyst_agent.py` para cobrir ambos os providers, não só Claude

## 5. Achados secundários (fora do escopo desta investigação — registrados para o backlog)

### 5.1 `test_publisher_agent.py` não coleta — quebrado em produção desde 13/06
`ImportError: cannot import name 'publish_to_facebook'`. Causa: commit `0406dd8` ("migrate to Instagram API without Facebook Login", já em `origin/main` desde 2026-06-13) removeu a função sem atualizar o teste correspondente. Gate de qualidade (`*pre-push`) não foi rodado antes desse push, ou os testes de publisher não estavam no escopo verificado.

### 5.2 `test_analyst_agent.py` — 9 falhas pré-existentes em produção
`httpx.ConnectError` em 9 dos 44 testes, presente no `main` committed, sem relação com o diff investigado ou com o Epic 21. Não investigado a fundo aqui (fora do escopo desta tarefa) — registrado para o @qa ou @dev avaliar.

### 5.3 Não existe branch `develop` neste repositório
`git branch -a` retorna só `main` e `origin/main`. A regra de proteção documentada em `.claude/rules/devops-git-rules.md` ("master só recebe via PR, trabalho acontece em develop") nunca foi de fato implementada neste repo — os últimos 15+ commits (incluindo mudanças de Meta App Review e migração de API) foram todos direto em `main`, sem PR.

---

## 6. Itens de backlog sugeridos (não criados como stories — aguardando priorização)

| Item | Prioridade sugerida | Por quê |
|---|---|---|
| Corrigir `test_publisher_agent.py` (import quebrado) | Alta | CI silenciosamente não cobre o publisher há ~1 mês |
| Investigar as 9 falhas de `test_analyst_agent.py` | Média | Pré-existente, mas mascara regressões reais no analyst |
| Criar branch `develop` + instalar hook `pre-push` | Média | Governança — regra documentada não está em vigor |
| Avaliar `ANALYST_PROVIDER` (Gemini vision) como story formal | Baixa/Backlog | Ideia com mérito de custo, mas precisa de A/B como qualquer troca de modelo (Epic 25 do Plano Mestre) |
