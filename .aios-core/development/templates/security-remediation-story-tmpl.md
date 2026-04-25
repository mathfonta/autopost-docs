# Story {epic}.{n} — [SECURITY] {title}

**Status:** Draft
**Epic:** {epic} — Security Remediations
**Story:** {epic}.{n}
**Estimativa:** {1-3} pontos
**Severity:** {CRITICAL | HIGH | MEDIUM}
**Finding ID:** {SEC-XXX}
**Source audit:** `docs/qa/security-reports/security-report-{date}.md`
**Created by:** @security (Sage) — autoridade aprovada por Matheus 2026-04-25

---

## Descrição

**Vulnerabilidade identificada:** {description of the finding}

**Impacto potencial:** {what could happen if exploited}

**Evidência:** {file:line or config reference where the issue was found}

---

## Critérios de Aceitação

- [ ] AC1: {specific fix that can be verified}
- [ ] AC2: {test or verification step}
- [ ] AC3: @security `*audit {scope}` confirms finding is resolved in next audit

---

## Escopo

**IN:**
- {specific change required}

**OUT:**
- Other security improvements not related to this specific finding

---

## Tasks

- [ ] {specific implementation step}
- [ ] Teste unitário ou de integração cobrindo o cenário corrigido
- [ ] @security re-verifica o finding após implementação

---

## Dev Notes

- **OWASP Category:** {A01-A10}
- **Attack vector:** {how the vulnerability could be exploited}
- **Recommended fix:** {concrete technical recommendation}

---

## Change Log

| Data | Agente | Ação |
|------|--------|------|
| {date} | @security | Story criada automaticamente a partir do finding {SEC-XXX} |
