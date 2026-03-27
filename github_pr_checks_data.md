# GitHub PR Checks for the CDC Lakehouse Assignment

Use the following checks as **required status checks** on the repository branch protection rules.

## Recommended Required Checks

### 1. `lint-python-or-sql`
Purpose:
- enforce formatting and baseline code quality
- catch obvious issues before review

Suggested commands for a Python-based stack:
```bash
ruff check .
black --check .
```

If SQL models are first-class in the repo, also run:
```bash
sqlfluff lint .
```

### 2. `test-pipeline`
Purpose:
- run unit and integration tests for CDC logic, validations, and warehouse modeling

Suggested command:
```bash
pytest -q
```

### 3. `validate-models`
Purpose:
- validate transformations and warehouse models

Examples:
```bash
dbt deps
dbt parse
dbt test
```

If not using dbt, run your equivalent model validation/test command.

### 4. `schema-contract-check`
Purpose:
- ensure schema compatibility logic works and breaking changes fail safely

Suggested command:
```bash
python scripts/check_schema_contracts.py
```

Or equivalent test target.

### 5. `data-quality-check`
Purpose:
- enforce system and business validation parity

Suggested command:
```bash
python scripts/run_data_quality_checks.py
```

Examples of checks:
- primary key uniqueness
- not-null checks
- referential integrity
- domain / enum validation
- business rule assertions

### 6. `catalog-check`
Purpose:
- verify lake and warehouse datasets are published in the catalog metadata

Suggested command:
```bash
python scripts/validate_catalog.py
```

### 7. `security-and-secrets-scan`
Purpose:
- catch vulnerable dependencies and committed secrets

Suggested tools:
```bash
pip-audit
bandit -r .
gitleaks detect --no-banner --redact
```

---

## Nice-to-Have Checks

### `e2e-local-pipeline`
- bootstrap local source + lake + warehouse
- run CDC ingestion
- apply updates and deletes
- verify lake history and warehouse snapshot
- simulate incompatible schema change and verify stop-the-line behavior

This is the best end-to-end signal, but it may take longer than the baseline checks.

### `docs-check`
- validate README and design docs are present and not stale

### `container-build`
- build the local pipeline image if the repo is containerized

---

## Branch Protection Recommendation

Set the following as required before merge:

- `lint-python-or-sql`
- `test-pipeline`
- `validate-models`
- `schema-contract-check`
- `data-quality-check`
- `catalog-check`
- `security-and-secrets-scan`

Optionally require:
- at least 1 reviewer approval
- conversation resolution
- up-to-date branch before merge
- squash merge or linear history

---

## Reviewer Checklist for PR Template

Add these items to the PR template so reviewers and candidates explain the important decisions.

### Required PR Description Sections

- Summary of what was implemented
- Source schema design
- CDC strategy
- Lake modeling
- Warehouse modeling
- Schema change safety
- Validation parity
- Catalog exposure
- Validation steps run locally
- Known limitations / next steps
- Responsible AI usage disclosure

### Author Checklist

- [ ] Linting passes locally
- [ ] Tests pass locally
- [ ] Model validation/tests pass
- [ ] Schema compatibility checks pass
- [ ] Data quality validations pass
- [ ] Catalog metadata validation passes
- [ ] README/setup steps were tested from a clean state
- [ ] End-to-end CDC flow was validated locally
- [ ] AI usage disclosed in PR description if used

---

## Suggested Tooling

### Python
- `ruff`
- `black`
- `pytest`
- `bandit`
- `pip-audit`

### SQL / Modeling
- `dbt`
- `sqlfluff`

### Data Quality
- `dbt test`
- Great Expectations, Soda, or equivalent
- custom SQL assertions if preferred

### Secrets / Security
- `gitleaks`

---

## Minimum Practical Baseline

If you want the leanest useful setup, start with these 5 blocking checks:

1. Lint and format checks
2. Pipeline tests
3. Model validation/tests
4. Schema contract compatibility check
5. Data quality validation

That is the smallest set that still gives good automatic PR review signal for this assignment.
