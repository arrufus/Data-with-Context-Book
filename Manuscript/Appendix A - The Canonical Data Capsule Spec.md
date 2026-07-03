# Appendix A — The Canonical Data Capsule Specification

*The full, annotated capsule/data-contract spec introduced in Chapters 5–6, with every field documented, its mapping to the Open Data Contract Standard (ODCS), and a JSON Schema you can validate against in CI.*

A Data Capsule is an **ODCS-compatible superset**: where ODCS gives a standard envelope for schema, quality, and service levels, the capsule insists that *semantics and policy are first-class and co-versioned*, and that the whole is bound to the data and deployed as one unit. Express your capsules as ODCS wherever the standard reaches; use the extensions below where it does not.

---

## A.1 The annotated spec

Every field below is documented inline. Fields marked **(required)** must be present for a capsule to pass validation; **(recommended)** should be present for any product beyond Level 1; the rest are situational.

```yaml
# ─────────────────────────────────────────────────────────────
# IDENTITY & OWNERSHIP
# ─────────────────────────────────────────────────────────────
data_product: client_holdings          # (required) stable product name, kebab/snake
version: 4.1.0                          # (required) SEMANTIC version — see A.3
kind: dataset                          # dataset | training_dataset | document_capsule | stream
owners:
  domain: wealth_client                # (required) owning business domain
  product_owner: maya.osei@…           # (required) ACCOUNTABLE person, not a team
  steward: client-data-stewards@…      # (recommended) executes governance checks
  contact: "#wealth-client-data; pager: wealth-client"   # a channel that is answered

# ─────────────────────────────────────────────────────────────
# 1. DATA PAYLOAD  +  2. STRUCTURAL METADATA
# ─────────────────────────────────────────────────────────────
datasets:
  - name: fact_client_holding          # (required) physical dataset name
    format: iceberg                    # (required) iceberg | delta | hudi | parquet
    location: s3://…/wealth/client_holding   # where the payload lives
    grain: "one row per client per holding per valuation_date"   # (required) THE grain
    classification: confidential       # (required) public|internal|confidential|restricted
    schema:                            # (required) structural metadata
      primary_key: [client_id, holding_id, valuation_date]
      fields:
        - name: client_id
          type: string                 # (required) type per SQL/Arrow
          description: "MDM golden-record client id"   # (required) what it IS
          classification: pii_pseudonymous            # field-level sensitivity
          constraints: [not_null]
        - name: em_exposure_pct
          type: decimal(5,2)
          description: >
            EM percentage. Meaning is DIVISION-DEPENDENT — see semantics.
          constraints: [not_null, "minimum: 0", "maximum: 100"]

# ─────────────────────────────────────────────────────────────
# 3. SEMANTIC METADATA  (the capsule's headline extension over a bare schema)
# ─────────────────────────────────────────────────────────────
semantics:
  em_exposure_pct:
    definition_ref: "glossary://wealth/emerging_markets"   # bound, versioned meaning
    rules:                             # machine-applicable business rules
      - "division == institutional: frontier markets INCLUDED in EM"
      - "division == retail: frontier markets EXCLUDED from EM"
    controlled_vocabulary: null        # or a ref, for enum fields
    note: "Consumers MUST normalise before cross-division comparison."

# ─────────────────────────────────────────────────────────────
# 4. QUALITY EXPECTATIONS  (executable, deployment-gating)
# ─────────────────────────────────────────────────────────────
quality:
  suite_ref: "tests/fact_client_holding/@v4"   # where the runnable tests live
  expectations:
    - "risk_score between 1 and 10"
    - "completeness(risk_review_date) >= 0.99"
    - "em_exposure_pct within 3 sigma of trailing 90d BY division"
    - "referential: client_id exists in dim_client"

# ─────────────────────────────────────────────────────────────
# 5. POLICY METADATA  (travels and enforces)
# ─────────────────────────────────────────────────────────────
policy:
  retention: P7Y                       # ISO-8601 duration
  legal_basis: "contract; legitimate_interest (suitability monitoring)"
  cde: true                            # is this a Critical Data Element?
  approved_uses: [internal_suitability_monitoring, adviser_copilot_retrieval]
  prohibited_uses: [automated_client_facing_advice_without_human_review]
  masking:
    - { field: client_id, method: hash_sha256, applies_to: [non_production] }

# ─────────────────────────────────────────────────────────────
# 6. PROVENANCE & LINEAGE ANCHORS
# ─────────────────────────────────────────────────────────────
lineage:
  openlineage_namespace: meridian.production
  dataset_name: wealth.client_holding
  upstream: [custodian_feed_a, custodian_feed_b, mdm.dim_client, risk.assessment]

# ─────────────────────────────────────────────────────────────
# 7. OPERATIONAL CONTRACT
# ─────────────────────────────────────────────────────────────
contracts:
  freshness_sla: 4_hours               # (required for L2+)
  completeness_sla: 99.9_percent
  availability_sla: 99.9_percent
  breaking_change_policy: semver_major
  deprecation_notice: 90_days
  consumers:                           # (recommended) known, contracted consumers
    - { team: adviser_copilot, contact: copilot-team@… }
    - { team: reg_reporting,   contact: reg-data@… }

# ─────────────────────────────────────────────────────────────
# LIFECYCLE  &  (where applicable) CERTIFICATION
# ─────────────────────────────────────────────────────────────
lifecycle:
  status: published                    # draft|review|published|deprecated|retired
certification_ref: certs/client_holdings.yaml   # -> Appendix C
```

### Kind-specific extensions

- **`kind: training_dataset`** adds a `snapshot` block (pinned source + immutable snapshot id), a per-feature `lawful_basis` and `permits_automated_decision` flag, a `labels` block (target, provenance, known bias), and `quality_baselines` for drift monitoring. (Chapter 7.)
- **`kind: document_capsule`** replaces `schema` with `source` (uri + content hash), a `lifecycle` block carrying `effective_from`/`effective_to`/`superseded_by`, and a `retrieval` block (chunking strategy, embedding-model version, index). (Chapter 9.)
- **`kind: stream`** adds registry `compatibility_mode` (backward|forward|full) and a topic/offset lineage anchor. (Chapter 7.)

---

## A.2 ODCS mapping

| Capsule field | ODCS location |
|---------------|---------------|
| `data_product`, `version` | `id`, `version` |
| `owners.*` | `team`, `roles`, `support` |
| `datasets[].schema` | `schema` (objects → properties) |
| `datasets[].grain` | schema object description / custom property |
| `semantics.*` | property `businessName` + custom semantic properties |
| `quality.*` | `quality` (rules, dimensions, thresholds) |
| `policy.classification`, `.masking` | `roles`, custom classification properties |
| `policy.retention`, `.legal_basis` | `slaProperties`, custom properties |
| `contracts.*_sla` | `slaProperties` |
| `lineage.*` | custom properties + OpenLineage identifiers |
| `lifecycle.status` | `status` |

The two places the capsule *extends* ODCS are `semantics` (bound, rule-bearing meaning) and `policy` (approved/prohibited uses as first-class, enforceable fields). Everything else round-trips to a standard ODCS document.

---

## A.3 Versioning rules (data semver)

- **Major** — a breaking change: a column removed/retyped, a key change, **or a semantic redefinition** (redefining what a field *means* is major even if the schema is untouched). Requires consumer sign-off + the deprecation notice.
- **Minor** — additive: a new nullable column, a new quality expectation, a widened-compatible type. Old consumers keep working.
- **Patch** — a correction that does not change the interface (a fixed description, a corrected lineage ref).

---

## A.4 JSON Schema (validation)

Drop this in `schemas/capsule.schema.json` and validate every spec in CI (`capsule-cli validate` wraps it). Abbreviated to the required core:

```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "title": "DataCapsule",
  "type": "object",
  "required": ["data_product", "version", "kind", "owners", "datasets"],
  "properties": {
    "data_product": { "type": "string", "pattern": "^[a-z0-9_]+$" },
    "version": { "type": "string", "pattern": "^\\d+\\.\\d+\\.\\d+$" },
    "kind": { "enum": ["dataset","training_dataset","document_capsule","stream"] },
    "owners": {
      "type": "object",
      "required": ["domain", "product_owner"],
      "properties": {
        "domain": { "type": "string" },
        "product_owner": { "type": "string", "format": "email",
                           "pattern": "^[^@\\s]+@[^@\\s]+$" }
      }
    },
    "datasets": {
      "type": "array", "minItems": 1,
      "items": {
        "type": "object",
        "required": ["name", "format", "grain", "classification", "schema"],
        "properties": {
          "format": { "enum": ["iceberg","delta","hudi","parquet"] },
          "classification": {
            "enum": ["public","internal","confidential","restricted"] },
          "schema": {
            "type": "object",
            "required": ["primary_key", "fields"],
            "properties": {
              "fields": {
                "type": "array", "minItems": 1,
                "items": {
                  "type": "object",
                  "required": ["name", "type", "description"],
                  "properties": {
                    "description": { "type": "string", "minLength": 1 }
                  }
                }
              }
            }
          }
        }
      }
    },
    "lifecycle": {
      "type": "object",
      "properties": {
        "status": {
          "enum": ["draft","review","published","deprecated","retired",
                   "superseded","withdrawn"]
        }
      }
    }
  }
}
```

The lifecycle enum carries the *union* of the table-capsule states (`draft…retired`) and the document-capsule states (`superseded`, `withdrawn`, from Chapter 9), so a document capsule validates against the same schema; a stricter deployment can make the enum kind-aware with an `if`/`then` branch on `kind: document_capsule`. Note also the small but load-bearing constraints: `product_owner` must match an email pattern (a *person*, not a team — `format: email` alone is annotation-only under most validators, so the `pattern` makes the floor actually bear load), every field's `description` must be non-empty (no undocumented columns), and every dataset must declare a `grain` and a `classification`. These are the machine-checkable floor beneath "no orphan, no undocumented, no unclassified data." Content validation beyond the schema — that `owner` is not the string `"TBD"`, that a semantic rule exists for a division-dependent field — runs as additional CI checks (Chapter 10), because a schema can enforce *presence* but not *meaning*.
