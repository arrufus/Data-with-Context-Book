# Appendix C — AI-Readiness Certification Checklist

*The certifier's working instrument from Chapter 19: the five readiness dimensions decomposed into evidence items, each marked automatable or attested, plus the blank certification record to fill in.*

Certification is a **derived property** of a data product, assembled from evidence that already exists, combining machine-readable checks with named human sign-off. It is *use-case specific*: certify a product for a *use*, at a *level*, on *evidence*. This checklist is what a certifier walks. Mark each item **✓ pass**, **~ partial**, or **✗ fail**; a dimension passes when its required items pass. "Auto" items the platform can confirm; "Attest" items require a named human to affirm.

---

## C.1 The five dimensions, decomposed

### 1. Meaning readiness
*Does the product carry enough business meaning to be used correctly?*

| Item | Auto / Attest | Evidence |
|------|:---:|----------|
| Business glossary terms bound to fields | Auto | glossary refs resolve |
| Entity & metric definitions present | Auto | semantic layer / ontology present |
| Calculation logic documented & governed | Auto | `governed_by` refs resolve |
| Division-/context-dependent rules encoded | Auto | rules present for flagged fields |
| **Documented meaning is *correct*** | **Attest** | owner affirmation |

### 2. Quality readiness
*Is the data reliable enough for the intended use?*

| Item | Auto / Attest | Evidence |
|------|:---:|----------|
| Quality contracted to the *use* (not generic score) | Auto | contract quality section |
| Completeness / accuracy thresholds defined & passing | Auto | gate results |
| Freshness within SLA | Auto | freshness monitor |
| Reconciliation results available | Auto | reconciliation job output |
| Known limitations documented | Attest | owner note |

### 3. Governance readiness
*Are ownership, policy, and accountability clear?*

| Item | Auto / Attest | Evidence |
|------|:---:|----------|
| Named product owner + steward (person, not team) | Auto | owners block |
| Privacy classification & CDE mapping | Auto | policy block |
| Retention & access policies defined | Auto | policy / platform config |
| Lawful basis per feature (for AI decision use) | Auto | lawful-basis manifest |
| **Policy-sensitive AI use approved** | **Attest** | risk/compliance sign-off |

### 4. Consumption readiness
*Can AI systems consume the product safely and consistently?*

| Item | Auto / Attest | Evidence |
|------|:---:|----------|
| Semantic-layer mappings / governed endpoints | Auto | semantic API present |
| Grain & aggregation rules stated | Auto | spec grain + semantic defs |
| Prohibited uses declared & enforced | Auto | policy + retrieval filter |
| Versioned data contract published | Auto | contract version |
| **Technical access pattern validated** | **Attest** | AI/ML team sign-off |

### 5. Operational readiness
*Can the org monitor, support, and change the product safely over time?*

| Item | Auto / Attest | Evidence |
|------|:---:|----------|
| End-to-end lineage complete | Auto | lineage graph coverage |
| Monitoring & alerting active | Auto | active-metadata config |
| Incident process & owner escalation defined | Attest | runbook exists |
| Version management & deprecation policy | Auto | contract policy |
| Recertification triggers set | Auto | cert `triggers` block |

**Rule of thumb:** three dimensions (quality, consumption, operational) are largely automatable; two (meaning, governance) hinge on human attestation. If no one will attest the meaning and the use, the product is not ready — whatever the automated checks say.

---

## C.2 Certification levels

| Level | Label | Bar |
|:---:|-------|-----|
| 0 | Not assessed | exists, unreviewed for AI use |
| 1 | Discoverable | catalogued, basic ownership |
| 2 | Contextualised | meaning, lineage, quality metadata present |
| 3 | Controlled | access, usage policy, contract enforced |
| 4 | Certified | approved for specific AI uses, with evidence + sign-off |
| 5 | Continuously assured | monitored, triggered recertification, telemetry |

Certify to the level the *use's stakes* require — most products need only be discoverable; a client-facing AI use needs 4–5. A healthy estate is a pyramid, not a plateau at the top.

---

## C.3 The blank certification record

Copy, fill, and store at `certs/<product>.yaml` (versioned with the product; queryable by agents before consumption).

```yaml
certification:
  product:            # product name
  product_version:    # semver
  level:              # 0–5
  approved_uses:
    - use:            # named AI use case
      controls: []    # e.g. [human_review_required, division_aware_normalisation]
  prohibited_uses: []
  evidence:
    meaning:      { status: , ref: , attested_by: }
    quality:      { status: , ref: , automated: true }
    governance:   { status: , owner: , cde_mapped: }
    consumption:  { status: , semantic_api: , prohibited_uses_enforced: }
    operational:  { status: , lineage: , monitoring: }
  signoffs:
    - { role: product_owner,  name: , date: }
    - { role: risk_compliance, name: , date: }   # for policy-sensitive uses
  triggers:
    - quality_gate_failure
    - lineage_break
    - contract_major_version
    - regulatory_change
    - periodic: P6M
  expires:            # date
```

---

## C.4 Anti-theatre guardrails

Certification dies when it becomes the gate it was meant to replace. Enforce these:

- **Derive, don't author.** Auto items must be pulled from existing tooling (catalogue, observability, lineage, CI), never re-entered by hand.
- **Surface, don't ceremony.** Certification status shows in the marketplace and the agent-context API automatically; a *meeting* is required only for the attested items on high-stakes products.
- **Faster than the workaround.** If certifying is slower than shipping uncertified, teams route around it and the risk goes underground. Speed is a safety feature.
- **Withdraw itself.** A triggered reassessment that downgrades a product must propagate to consumers *mid-flight* (suppress from certified workflows, lower the level in the API) — a certification that cannot revoke itself is a false assurance.
