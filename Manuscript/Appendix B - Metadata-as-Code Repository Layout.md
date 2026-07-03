# Appendix B — Metadata-as-Code Repository Layout

*The full repository structure, CI pipeline, PR template, CODEOWNERS, and scaffolding introduced in Chapters 9–10 and 13. Lift and adapt.*

The principle (Chapter 9): the definition in Git is the source of truth; the live platform is a deployment of it. Everything below serves that principle — one repository where a product's contract, schema, tests, policy, and glossary definition live together, move together, and are reconciled by CI.

---

## B.1 The tree

```
meridian-data-platform/
├── contracts/                    # data contracts (ODCS), one per product
│   └── wealth/
│       └── client_holdings.yaml
├── table-specs/                  # capsule/table definitions → DDL (Appendix A)
│   └── wealth/
│       └── fact_client_holding.yaml
├── tests/                        # quality suites (dbt / Great Expectations)
│   └── fact_client_holding/
│       ├── schema.yml
│       └── expectations.py
├── policies/                     # policy-as-code: classification, masking, access
│   ├── pii.yaml
│   └── retention.yaml
├── glossary/                     # business definitions & rules (bound meaning)
│   └── wealth/
│       └── emerging_markets.yaml
├── ontologies/                   # domain ontologies (Part IV)
│   └── wealth/
│       └── suitability.yaml
├── semantic/                     # semantic-layer metric definitions (Part IV)
│   └── wealth/
│       └── em_exposure.yaml
├── certs/                        # certification records (Appendix C)
│   └── client_holdings.yaml
├── pipelines/                    # transformation code (dbt models, jobs)
│   └── wealth/
├── schemas/                      # JSON Schemas for validation (Appendix A)
│   └── capsule.schema.json
├── scripts/
│   └── new-capsule.sh            # scaffolding (B.5)
├── CODEOWNERS                    # who must review which paths (B.3)
└── .github/workflows/
    └── deploy-capsule.yml        # the CI (B.2)
```

The layout encodes the discipline: a product's contract, schema, tests, policy, glossary term, ontology, and certification are all reachable from its name, and a change to any of them shows up alongside the others in one pull request. Nothing about the product lives *only* in a wiki or a catalogue.

---

## B.2 The CI pipeline

```yaml
# .github/workflows/deploy-capsule.yml
name: deploy-capsule
on:
  pull_request:
    paths: ["table-specs/**", "contracts/**", "tests/**",
            "policies/**", "glossary/**", "semantic/**", "certs/**"]
jobs:
  reconcile:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      # 1. LINT & SCHEMA-VALIDATE the spec (Appendix A JSON Schema)
      - run: capsule-cli validate table-specs/**/*.yaml
      # content checks beyond schema: owner != TBD, semantic rule present for
      # division-dependent fields, every field has a non-empty description
      - run: capsule-cli lint --strict

      # 2. PLAN: diff spec vs live table; classify the change (major/minor/patch)
      - run: capsule-cli plan --out change_type.txt

      # 3. CONTRACT COMPATIBILITY: block breaking change without sign-off + notice
      - run: capsule-cli check-consumers --require-signoff-on major

      # 4. QUALITY: run the bound suite against a sampled staging snapshot
      - run: dbt build --select state:modified+
      - run: great_expectations checkpoint run fact_client_holding

      # 5. IMPACT ANALYSIS via the lineage graph
      - run: capsule-cli impact --format pr-comment

      # 6. POLICY TESTS: assert masking works for each role (Chapter 10)
      - run: capsule-cli test-policies policies/

      # 7. DEPLOY (main only): apply DDL, promote snapshot, emit lineage, notify
      - if: github.ref == 'refs/heads/main'
        run: capsule-cli apply && capsule-cli publish && capsule-cli notify-consumers
```

Each numbered stage is a guarantee from the book: validation (no malformed or undocumented spec), semver classification (computed, not asserted), consumer protection (the Okonkwo guard), quality gating, blast-radius surfacing, policy verification, and a single deploy path from `main`. The only route to production runs through all seven.

---

## B.3 CODEOWNERS

```
# Domain owners must review changes to their products
/table-specs/wealth/     @maya-osei @wealth-data-eng
/contracts/wealth/       @maya-osei
/glossary/wealth/        @maya-osei @wealth-sme
/semantic/wealth/        @maya-osei @wealth-analytics

# Governance owns policy and classification standards
/policies/               @governance-team
/certs/                  @governance-team @risk-compliance

# Platform owns the machinery
/.github/workflows/      @platform-team
/schemas/                @platform-team
```

CODEOWNERS is the mechanism behind "the right reviewer is pulled in automatically." A change to `glossary/wealth/emerging_markets.yaml` cannot merge without Maya and a wealth SME — which is exactly the human check that was missing when the Okonkwo redefinition slipped through as an enrichment-job edit.

---

## B.4 Pull-request template

```markdown
<!-- .github/pull_request_template.md -->
## What changes
- [ ] Product(s) affected:
- [ ] Change type (CI will confirm): major / minor / patch

## If major (breaking)
- [ ] Consumers notified (list): 
- [ ] 90-day notice started / waiver approved by owner
- [ ] Migration path documented

## Meaning & policy
- [ ] Semantic definitions updated where meaning changed
- [ ] Classification / masking reviewed
- [ ] Certification impact assessed (see certs/)

## Evidence
- [ ] Quality suite passing
- [ ] Impact analysis reviewed (CI comment)
```

The template makes the discipline unavoidable at the moment of change: you cannot open a breaking PR without confronting the consumers, the notice, and the migration path.

---

## B.5 Scaffolding — make the governed path the fast path

```bash
#!/usr/bin/env bash
# scripts/new-capsule.sh  —  generate a compliant capsule skeleton
# usage: ./new-capsule.sh --name monthly_revenue --domain finance --owner alice@…
set -euo pipefail
name="$2"; domain="$4"; owner="$6"
mkdir -p "table-specs/$domain" "tests/$name" "contracts/$domain"
cat > "table-specs/$domain/$name.yaml" <<EOF
data_product: $name
version: 0.1.0
kind: dataset
owners: { domain: $domain, product_owner: $owner }
datasets:
  - name: $name
    format: iceberg
    grain: "TODO — state the grain"
    classification: internal
    schema: { primary_key: [TODO], fields: [] }
lifecycle: { status: draft }
EOF
cat > "contracts/$domain/$name.yaml" <<EOF
contract_version: "0.1"
table: $domain.$name
owner: $owner
sla: { freshness: "TODO", completeness: "TODO" }
consumers: []
EOF
echo "Scaffolded $name. Fill the TODOs, then open a PR."
```

The scaffolder is not a convenience; it is a governance control. When the governed path (`new-capsule.sh` + a PR) is *easier* than the ungovernanced shortcut (a console `CREATE TABLE`), the shortcut loses — which is the only way adoption sticks. Every TODO it leaves is a decision the discipline requires you to make consciously: the grain, the keys, the SLA, the consumers.

---

## B.6 Naming house style

- **Layers by schema/prefix:** `bronze.`, `silver.`, `gold.` (canonical), `gold_serving.`
- **Entities** singular nouns (`client`); **facts** `fact_<event>`; **dims** `dim_<entity>`; **vault** `hub_`/`link_`/`sat_`.
- **Keys:** `<entity>_sk` (surrogate) and `<entity>_id` (natural) — always both.
- **Booleans** `is_`/`has_`; **dates** `_date`; **timestamps** `_ts` (UTC unless stated).
- **Ambiguous fields** carry a semantic reference and name the ambiguity in the description.

A name should tell a consumer its layer, its grain, and where its meaning is defined — because the agent will not open the documentation.
