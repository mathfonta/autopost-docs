# Dependency Audit Report — {PROJECT_NAME}

**Data:** {YYYY-MM-DD}  
**Executado por:** @security (Sage)  
**Story/Contexto:** {STORY_ID | Manual}  
**Projeto:** {PROJECT_NAME}

---

## Resumo Executivo

| Ferramenta | Escopo | CRITICAL | HIGH | MEDIUM | LOW | Status |
|-----------|--------|----------|------|--------|-----|--------|
| pip-audit | Python (requirements.txt) | {C} | {H} | {M} | {L} | {PASS\|FAIL\|CONCERNS\|SKIPPED} |
| npm audit | Node.js (package.json) | {C} | {H} | {M} | {L} | {PASS\|FAIL\|CONCERNS\|SKIPPED} |
| **TOTAL** | | **{C}** | **{H}** | **{M}** | **{L}** | **{OVERALL}** |

**Decisão de Gate:** `{FAIL | CONCERNS | PASS | SKIPPED}`

---

## 1. Dependências Python (pip-audit)

> **Arquivo escaneado:** `{requirements.txt | pyproject.toml | SKIPPED}`  
> **pip-audit version:** {VERSION}

{IF vulnerabilities found}

### Vulnerabilidades Encontradas

| Pacote | Versão | ID | Aliases (CVE) | Fix Disponível | Severidade | Descrição |
|--------|--------|----|---------------|----------------|-----------|-----------|
| {package} | {version} | {GHSA-xxxx} | CVE-xxxx-xxxx | {fix_version \| Não} | {CRITICAL\|HIGH\|MEDIUM\|LOW} | {description} |

### Recomendações Python

- [ ] **IMEDIATO:** `pip install --upgrade {package}=={fix_version}` para pacotes CRITICAL
- [ ] Atualizar `requirements.txt` com versões fixas
- [ ] Re-executar `pip-audit` após atualização para confirmar remedição
- [ ] Se sem fix disponível: criar story de remediação via `@security *create-remediation`

{ELSE}
✅ Nenhuma vulnerabilidade encontrada nas dependências Python.
{END IF}

{IF pip-audit SKIPPED}
⚠️ pip-audit não disponível — scan pulado. Para executar manualmente:
```bash
pip install pip-audit
pip-audit --requirement requirements.txt
```
{END IF}

---

## 2. Dependências Node.js (npm audit)

> **Arquivo escaneado:** `package.json`  
> **npm version:** {VERSION}

{IF vulnerabilities found}

### Vulnerabilidades Encontradas

| Pacote | Severidade | CVE/GHSA | Caminho | Fix | Ação |
|--------|-----------|----------|---------|-----|------|
| {package} | {CRITICAL\|HIGH\|MEDIUM\|LOW} | {ID} | {via chain} | {npm audit fix} | {descrição} |

### Recomendações Node.js

- [ ] **IMEDIATO:** `npm audit fix --force` para CRITICAL
- [ ] Revisar breaking changes antes de atualizar dependências major
- [ ] Re-executar após atualização para confirmar

{ELSE}
✅ Nenhuma vulnerabilidade encontrada nas dependências Node.js.
{END IF}

{IF npm SKIPPED}
ℹ️ npm audit não executado (sem package.json no escopo ou skipped).
{END IF}

---

## 3. Decisão de Gate

**Status Final:** `{FAIL | CONCERNS | PASS}`

### Lógica de Decisão

```
CRITICAL encontrado em qualquer ferramenta → FAIL  (bloquear CI, criar story de remediação)
HIGH encontrado                             → CONCERNS (documentar como dívida técnica)
Somente MEDIUM/LOW                         → PASS (registrar em MEMORY.md)
Nenhuma vulnerabilidade                    → PASS (clean)
Todas as ferramentas skipped               → SKIPPED (aviso — não bloqueia)
```

{IF FAIL}
### Ações Obrigatórias (BLOQUEIAM merge/deploy)

- [ ] Corrigir todas as vulnerabilidades CRITICAL antes de merge
- [ ] Rodar `@security *create-remediation {finding-id}` para cada CRITICAL
- [ ] Re-executar `*dep-audit` após remediação para confirmar PASS
{END IF}

{IF CONCERNS}
### Ações Recomendadas (antes de produção)

- [ ] Endereçar vulnerabilidades HIGH em próxima sprint
- [ ] Registrar como tech debt no backlog via `@po *backlog-add`
{END IF}

---

## 4. Próximos Passos

### Imediato (bloqueia merge se FAIL)
{immediate_actions_list}

### Curto Prazo (antes de produção)
{short_term_actions_list}

### Longo Prazo (dívida técnica)
{long_term_actions_list}

---

## Histórico de Auditorias

| Data | Ferramenta | C | H | M | L | Status | Notas |
|------|-----------|---|---|---|---|--------|-------|
| {YYYY-MM-DD} | pip-audit + npm audit | {C} | {H} | {M} | {L} | {STATUS} | {notas} |

---

**Versões das Ferramentas:**
- pip-audit: {version}
- npm: {version}
- Python: {version}
- Node.js: {version}

**Relatório gerado em:** {ISO-8601 timestamp}  
**Agente:** @security (Sage — Sentinel)  
**Próxima auditoria recomendada:** {YYYY-MM-DD + 30 dias}
