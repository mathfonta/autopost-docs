# security-python-dep-audit

**Task ID:** `security-python-dep-audit`  
**Version:** 1.0.0  
**Status:** Active

---

## Purpose

Executa auditoria de dependências Python no projeto, usando `pip-audit` (PyPA) como ferramenta primária. Detecta vulnerabilidades em pacotes do `requirements.txt` ou `pyproject.toml`, categoriza por severidade e gera relatório consolidado.

**Estratégia:** Automação total, graceful skip se pip-audit indisponível — nunca bloqueia o workflow por falha de instalação.

---

## Execution Modes

### 1. YOLO Mode - Fast, Autonomous (0-1 prompts)
- Autonomous decision making with logging
- **Best for:** CI/CD pipelines, time-sensitive work

### 2. Interactive Mode - Balanced (5-10 prompts) **[DEFAULT]**
- Explicit checkpoints
- **Best for:** Manual runs, audit review sessions

**Parameter:** `mode` (optional, default: `interactive`)

---

## Task Definition (AIOS Task Format V1.0)

```yaml
task: securityPythonDepAudit()
responsável: Sage (Sentinel)
responsavel_type: Agente
atomic_layer: Strategy

Entrada:
- campo: target_dir
  tipo: string
  origem: User Input
  obrigatório: true
  validação: Path with requirements.txt or pyproject.toml

- campo: fail_on
  tipo: string
  origem: config
  obrigatório: false
  padrão: CRITICAL
  validação: CRITICAL|HIGH|MEDIUM|LOW

Saída:
- campo: audit_report
  tipo: object
  destino: File (docs/qa/security-reports/dep-audit-YYYY-MM-DD.md)
  persistido: true

- campo: vulnerabilities
  tipo: array
  destino: Memory
  persistido: false

- campo: gate_result
  tipo: string (FAIL|CONCERNS|PASS|SKIPPED)
  destino: Memory
  persistido: false
```

---

## Pre-Conditions

```yaml
pre-conditions:
  - [ ] requirements.txt or pyproject.toml exists in target_dir
    tipo: pre-condition
    blocker: true
    validação: |
      Check for requirements.txt or pyproject.toml in target directory
    error_message: "Pre-condition failed: No Python project file found at target_dir"
```

---

## Step-by-Step Execution

### Step 1: Detect Python Project File

**Purpose:** Determine the pip-audit target file

**Actions:**
```bash
cd ${target_dir}

if [ -f requirements.txt ]; then
  AUDIT_TARGET="--requirement requirements.txt"
  echo "✓ Found: requirements.txt"
elif [ -f pyproject.toml ]; then
  AUDIT_TARGET=""
  echo "✓ Found: pyproject.toml (scanning current project)"
else
  echo "SKIP: No Python project file found"
  exit 0  # Graceful skip
fi
```

---

### Step 2: Ensure pip-audit is Available

**Purpose:** Install pip-audit if missing, with graceful fallback

**Actions:**
```bash
if ! command -v pip-audit &> /dev/null; then
  echo "Installing pip-audit..."
  pip install pip-audit --quiet 2>&1 || {
    echo "WARNING: pip-audit installation failed. Skipping Python dep audit."
    echo "To fix: pip install pip-audit"
    exit 0  # Graceful skip — never blocks workflow
  }
fi

pip-audit --version
```

---

### Step 3: Run pip-audit

**Purpose:** Execute dependency vulnerability scan in JSON format

**Commands:**
```bash
# JSON output for programmatic parsing
pip-audit \
  ${AUDIT_TARGET} \
  --format json \
  --progress-spinner off \
  --output pip-audit-results.json \
  2>&1 || true  # Exit code 1 = vulnerabilities found (expected)

# Human-readable output for logs
pip-audit \
  ${AUDIT_TARGET} \
  --progress-spinner off \
  2>&1 | tee pip-audit-output.txt || true

echo "✓ pip-audit scan completed"
```

---

### Step 4: Parse and Categorize Results

**Purpose:** Extract vulnerabilities and classify by severity

**Parse Logic:**
```python
import json

with open('pip-audit-results.json') as f:
    results = json.load(f)

# pip-audit JSON structure: list of {name, version, vulns: [{id, fix_versions, aliases, description}]}
vulnerabilities = []
counts = {'CRITICAL': 0, 'HIGH': 0, 'MEDIUM': 0, 'LOW': 0}

for dep in results:
    for vuln in dep.get('vulns', []):
        severity = classify_severity(vuln)
        vulnerabilities.append({
            'package': dep['name'],
            'version': dep['version'],
            'id': vuln['id'],
            'aliases': vuln.get('aliases', []),
            'fix_versions': vuln.get('fix_versions', []),
            'description': vuln.get('description', ''),
            'severity': severity
        })
        counts[severity] += 1

def classify_severity(vuln):
    # pip-audit GHSA IDs — conservative default MEDIUM unless CVE data available
    # With --aliases flag, CVE IDs enable better CVSS-based classification
    # Without CVSS data: HIGH if fix available, MEDIUM otherwise
    if vuln.get('fix_versions'):
        return 'HIGH'
    return 'MEDIUM'

gate_result = (
    'FAIL' if counts['CRITICAL'] > 0 else
    'CONCERNS' if counts['HIGH'] > 0 else
    'PASS'
)
```

**Note on Severity:** pip-audit v2.x does not return CVSS scores natively. Use `pip-audit --aliases` to get CVE IDs for cross-referencing the NVD database. For CI gates, `--fail-on CRITICAL` uses pip-audit's own severity classification.

---

### Step 5: Generate Report

**Purpose:** Create consolidated report using the standard template

**Actions:**
1. Load `security-dep-audit-report-tmpl.md`
2. Fill Python audit section with findings
3. Save to `docs/qa/security-reports/dep-audit-{YYYY-MM-DD}.md`
4. Update `@security` MEMORY.md with audit entry

**Gate Decision Logic:**
```
CRITICAL found   → gate = FAIL   (block CI, create remediation story)
HIGH found       → gate = CONCERNS (warn, document as debt)
MEDIUM/LOW only  → gate = PASS   (log in MEMORY.md)
No vulnerabilities → gate = PASS (clean)
pip-audit unavailable → gate = SKIPPED (warn, do not block)
```

---

## Post-Conditions

```yaml
post-conditions:
  - [ ] Audit executed or gracefully skipped; report generated
    tipo: post-condition
    blocker: true
    validação: |
      Verify docs/qa/security-reports/dep-audit-YYYY-MM-DD.md exists
    error_message: "Post-condition failed: Audit report not generated"
```

---

## Tools

```yaml
Tools:
- pip-audit:
    version: latest (PyPA)
    install: pip install pip-audit
    used_for: Python dependency vulnerability scanning via PyPI Advisory Database
    cost: $0
    note: Maintained by PyPA. Preferred over 'safety' (requires paid token for full DB).

- python-pip:
    version: built-in
    used_for: Package management and pip-audit installation
    cost: $0
```

---

## Error Handling

| Error | Recovery |
|-------|----------|
| pip-audit not installable | Graceful skip — WARN, do not block |
| requirements.txt missing | Graceful skip — SKIPPED result |
| CRITICAL vulnerabilities found | gate = FAIL, create remediation story via @security |
| Network unreachable (PyPI DB) | Partial results — WARN, proceed with available data |

---

## Integration with *dep-audit Command

This task is called by `*dep-audit` in the @security agent:

```
*dep-audit
  → Detect project type (Python / Node.js / both)
  → Python: security-python-dep-audit.md
  → Node.js: npm audit (security-scan.md Step 2)
  → Consolidate → security-dep-audit-report-tmpl.md
  → Save → docs/qa/security-reports/dep-audit-YYYY-MM-DD.md
  → Update MEMORY.md audit history
```

---

## GitHub Actions Integration

```yaml
# backend/.github/workflows/security.yml
- run: pip install pip-audit
- run: pip-audit --requirement requirements.txt --fail-on CRITICAL
```

---

## Success Criteria

- ✅ pip-audit executes or gracefully skips with warning
- ✅ Report generated in `docs/qa/security-reports/`
- ✅ Gate decision reflects actual findings
- ✅ Zero manual intervention required
- ✅ Works in GitHub Actions (CI/CD)

---

## Notes

- **pip-audit vs safety:** pip-audit is PyPA-maintained and free. `safety` requires a paid token for full vulnerability database access — only use as fallback if explicitly configured.
- **Severity nuance:** pip-audit severity classification differs from CVSS. Use `pip-audit --aliases` to get CVE IDs for more precise CVSS scoring.
- **CI usage:** `pip-audit --requirement requirements.txt --fail-on CRITICAL` — exits with code 1 on CRITICAL findings.

---

## Metadata

```yaml
story: STORY-7.2
version: 1.0.0
dependencies:
  - security-dep-audit-report-tmpl.md
tags:
  - security
  - python
  - pip-audit
  - dependency-audit
created_at: 2026-04-25
ids_decision: CREATE
ids_justification: |
  No Python dependency audit task existed. security-scan.md covers only Node.js/npm.
  New capability for Python/FastAPI backend (AutoPost backend on Railway).
  evaluated_patterns: [security-scan.md (npm only)]
  rejection_reason: security-scan.md uses npm audit, ESLint, secretlint — incompatible with Python
  new_capability: Python dependency vulnerability scanning via pip-audit (PyPA)
```
