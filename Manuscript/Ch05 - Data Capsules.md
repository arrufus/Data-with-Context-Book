# Chapter 5 — Data Capsules: Data and Metadata as One Atomic Unit

> **Where you are:** Part II — *Bind*. This is the first move of the discipline: attaching context to data so the two cannot drift apart. Maturity target for this chapter: **Level 0 → Level 1**.

---

## The answer that was almost right

On a Tuesday in March, an adviser at Meridian Financial asked the firm's new relationship-manager copilot a routine question: *"Is the Okonkwo family portfolio still suitable given their risk profile?"* The copilot answered in two seconds. It said yes. The portfolio was 18% weighted to emerging-market equities, the client's risk tolerance was assessed at 4 out of 10 — moderate-conservative — and the model judged the allocation within tolerance.

The answer was wrong, and it was wrong for a reason that had nothing to do with the model.

The copilot had read a field called `em_exposure_pct` from the client-holdings dataset and compared it against a suitability threshold. What it did not know — what was written nowhere it could read — was that the institutional side of Meridian classifies frontier markets *inside* emerging markets, while the retail side excludes them. The Okonkwo account was a retail account whose holdings had been enriched, two pipelines upstream, using the institutional definition. The real frontier-inclusive exposure was 26%. For a 4-out-of-10 client, that was outside tolerance, and a human adviser who knew the family would have caught it. The copilot did not know the family. It knew the column. And the column's meaning had quietly changed identity somewhere between the custodian feed and the dashboard.

Nobody had done anything wrong, exactly. The engineer who wrote the enrichment job used the definition their team used. The data passed every quality check it was asked to pass — it was complete, fresh, correctly typed, and reconciled to the custodian to the penny. Maya, the data product owner for Meridian's client domain, could show you a green dashboard for that feed going back eighteen months. The data was, by every measure her team tracked, good.

It was good data that could not explain itself. And an AI consumer that reads columns at machine speed, thousands of times an hour, with no institutional memory and no instinct for when something smells wrong, is exactly the kind of consumer that turns "good data that can't explain itself" into a suitability breach with a regulator's name on it.

This chapter is about the discipline that would have prevented that answer. Not a better model. Not a cleaner pipeline. A different unit of design.

## The problem is structural, not careless

It is tempting to read the Okonkwo story as a failure of communication — if only the two teams had agreed on what "emerging markets" meant, if only someone had written it down. But people *had* written it down. The retail definition lived in a glossary in Confluence. The institutional definition lived in a different team's Confluence space. The reconciliation rule lived in someone's head and, partially, in a comment inside a dbt model. The data itself — the actual numbers flowing through the platform — carried none of this. It carried values. The meaning of those values lived everywhere except next to them.

This is not a Meridian problem. It is the default condition of nearly every data platform built in the last twenty years, and it is an artefact of how those platforms evolved rather than a choice anyone made.

Data lives in one place: databases, object stores, files, queues. Metadata — the schema, the definitions, the quality rules, the ownership, the lineage, the policy — lives somewhere else: in catalogues, wikis, slide decks, ETL repositories, the tribal memory of whoever has been on the team longest. The two are connected by convention and good intentions, which is to say they are connected by nothing enforceable. So they drift. A column is renamed in production but not in the catalogue. A definition is agreed in a meeting but never propagated to the dataset. A retention policy is set in a governance forum and never reaches the table it governs. Each drift is small. Together they are the reason your engineers spend their afternoons in Slack asking "what does this column actually mean?" and your governance team spends its quarters reconstructing documentation that was out of date the day it was published.

At small scale, this is tolerable. The team is small enough that the tribal knowledge holds. At enterprise scale — under regulatory scrutiny, with AI models training on the data and AI agents reasoning over it in production — it stops being tolerable and becomes a liability you are carrying on the books whether or not you have named it.

The fix is not more documentation. More documentation, disconnected from the data, just gives you more things to keep in sync and more ways to be wrong. The fix is to change what travels. To make context an intrinsic property of the data rather than an external commentary on it.

## What a Data Capsule is

A **Data Capsule** is a logical unit in which the data payload and the context that makes it interpretable are **bound together and versioned together**, so that they cannot move or change independently without someone noticing. The idea is older than any name for it and, once you see it, faintly obvious: data should never travel alone.

<!-- FIG 5.1: capsule anatomy — seven components (payload, structural, semantic, quality, policy, provenance, contract) bound and co-versioned around one data payload. -->

A complete capsule binds seven components.

**1. Data payload.** The actual records, rows, and events — whatever you are storing. The holdings, the positions, the transactions. This is the part everyone already has.

**2. Structural metadata.** Schema, data types, primary keys, nullability, constraints — the details that tell you what *shape* the data has. That `client_id` is a non-null string, that `risk_score` is an integer between 1 and 10, that the grain of the table is one row per holding per valuation date.

**3. Semantic metadata.** Business definitions, units, domain vocabulary — the details that tell you what the data *means*. What `em_exposure_pct` actually counts, and crucially, *under which definition*. Why `revenue_adj` differs from `revenue_raw`. What a null in `risk_review_date` signifies. This is the layer whose absence cost Meridian the Okonkwo answer.

**4. Quality expectations.** Validation rules, expected distributions, thresholds, anomaly signals — what counts as "good data" versus "investigate this immediately." Not as documentation, but as executable expectations: `risk_score` must be between 1 and 10; completeness on `risk_review_date` must exceed 99% or training is blocked.

**5. Policy metadata.** Sensitivity classification, masking rules, retention requirements, geographic restrictions, approved uses. Can this dataset be used to train a client-facing model? Can it cross a geographic boundary? May it be shown to all employees, or only to the client's own adviser? For a wealth manager, these are not afterthoughts; they are the difference between a compliant system and an enforcement action.

**6. Provenance and lineage anchors.** Where did this come from, what transformations produced it, how does it connect to other datasets — anchored by stable identifiers that let you trace data from farm to table. When a regulator asks "where did this number come from?", this is the component that lets you answer with evidence instead of a forensic project.

**7. Operational contract.** Freshness SLAs, completeness guarantees, ownership, and a contact who answers the phone. Who owns this, how fresh is it promised to be, and what is the process when it breaks.

A useful test for whether you actually have a capsule, rather than a dataset with some documentation nearby: **if the data and any of these components can change independently without the change being detected, you do not have an atomic unit.** You have data and metadata that happen, for now, to agree.

### Binding does not mean colocation

A common misreading is that a capsule requires stuffing all seven components physically inside every file. It does not, and for large datasets it should not. What the capsule requires is three properties, and physical colocation is only one way — often not the best way — to achieve them.

- **Strong binding.** Stable identifiers and enforced references between the data and its context, so that you can always get from one to the other and the link cannot silently break.
- **Co-versioning.** When the data changes, its metadata changes in the same act. Version 4 of the schema, the quality suite that goes with version 4, and the contract that version 4 promises all move as one.
- **Deployability as one unit.** The whole thing ships through CI/CD as a single artefact, not assembled by hand from parts that were deployed separately and hopefully match.

In practice this means some metadata travels *in* the data — the schema and statistics that a query engine needs immediately — while richer, heavier metadata is *referenced* by stable identifier and versioned alongside. We will make that hot-versus-cold distinction concrete in the next chapter. The principle to hold onto here is that binding is about enforced, co-versioned reference, not about cramming everything into one place.

## Why this is feasible today

If this sounds like it requires inventing new infrastructure, it does not. Most of what a capsule needs already exists in systems you are probably already running. The discipline is mostly a matter of using these capabilities deliberately, together, instead of accidentally and apart.

**Self-describing file formats** already embed metadata with data. Parquet, ORC, Avro, and Arrow carry their schema and column statistics inside the file. Copy a Parquet file to a new system and the schema travels with it — no catalogue lookup required. Scientific formats like HDF5, NetCDF, and FITS have done this for decades, because the people using them learned early that shipping data without its context produces irreproducible results, which in science is the only kind of result that doesn't count.

**Open table formats** take the idea up a level and make an entire table behave like a capsule. Apache Iceberg defines a table as data files plus manifest and metadata files that track schema evolution, partitioning, snapshots, and statistics — reference the table and you get all of it together. Delta Lake represents a table as data files plus a transaction log recording every change and schema version, which is why time travel and ACID guarantees work at all: data and metadata evolve in the same log. Apache Hudi maintains a commit timeline tracking every file version and operation as part of the table's identity.

**Schema registries and data contracts** push binding into streaming. A registry binds schemas to Kafka topics and enforces compatibility, so producers and consumers agree on structure through an enforced intermediary rather than a shared hope. Data contracts formalise structure and expectations as machine-readable specifications.

**"Metadata as code" tooling** co-locates the softer components with the data's transformation logic. dbt keeps SQL models, tests, documentation, and tags in the same repository, so deploying a model deploys its tests and docs too. Great Expectations and Soda let you define quality expectations as code that lives beside the pipeline. OpenLineage provides a standard for emitting lineage events about jobs and datasets.

None of this is exotic. The technology to bind data and metadata is production-ready and, in most enterprises, already partly deployed. What is usually missing is not capability but the architectural decision to treat "data plus context, atomic" as the default unit rather than the exception — and the organisational alignment to make that decision stick. We will spend Part III on the alignment. The rest of this chapter is about the decision.

## The Okonkwo account, recast as a capsule

Return to the suitability error and watch what each capsule component would have changed.

Had `em_exposure_pct` carried **semantic metadata** binding it to a specific, named definition — `frontier_markets: excluded; basis: retail_classification_v2` — the enrichment job that applied the institutional definition would have been writing into a field whose contract said otherwise. That is a detectable contradiction, not a silent reinterpretation.

Had the dataset carried **quality expectations** tied to fitness for purpose, the mismatch between the retail account's expected exposure distribution and the institutional-enriched values would have been a candidate anomaly rather than a green tick.

Had it carried **provenance anchors**, the question "which definition produced this number?" would have been a lineage query returning the institutional enrichment job, not a two-day investigation across two teams' Confluence spaces.

And had it carried an **operational contract** with a named owner, the answer to "who approved using the institutional definition on retail accounts?" would have been a person with a remit, rather than a chain of *not me*.

The model was never the problem. The data was never *dirty*. The data simply could not produce, on demand, the evidence about its own meaning that an AI consumer needed in order to use it safely. Binding that evidence to the data is the whole of the discipline.

## The capsule as code

Here is what Meridian's client-holdings dataset looks like when its context is bound to it and expressed as a single, version-controlled definition. This is the artefact that lives in Git, travels through CI/CD, and serves as the source of truth from which catalogues and dashboards are merely views.

**Listing 5.1 — the `client_holdings` capsule** (the source of truth Chapter 6 deploys from).

```yaml
data_product: client_holdings
version: 4.1.0
owners:
  domain: wealth_client
  product_owner: maya.osei@meridian.example
  steward: client-data-stewards@meridian.example
  contact: "#wealth-client-data (Slack), pager: wealth-client"

datasets:
  - name: fact_client_holding
    format: iceberg
    location: s3://meridian-lakehouse/curated/wealth/client_holding
    grain: "one row per client per holding per valuation_date"
    classification: confidential

    schema:
      primary_key: [client_id, holding_id, valuation_date]
      fields:
        - name: client_id
          type: string
          description: "Meridian client identifier (golden record from MDM)"
          classification: pii_pseudonymous
        - name: division
          type: string
          description: "Servicing division: retail | institutional"
          constraints: [not_null, "in: [retail, institutional]"]
        - name: holding_id
          type: string
          description: "Instrument identifier, FIGI where available"
        - name: asset_class
          type: string
          description: "Asset class per Meridian taxonomy v3 (see semantics.asset_class)"
        - name: em_exposure_pct
          type: decimal(5,2)
          description: >
            Percentage of this holding classified as emerging-market.
            Definition is DIVISION-DEPENDENT — see semantics below.
          constraints: [not_null, "minimum: 0", "maximum: 100"]
        - name: risk_score
          type: integer
          description: "Client risk tolerance, 1 (conservative) to 10 (aggressive)"
          constraints: [not_null, "minimum: 1", "maximum: 10"]
        - name: risk_review_date
          type: date
          description: "Date risk tolerance was last assessed; null = never assessed"
        - name: valuation_date
          type: date
          description: "As-of date for the holding valuation (UTC)"

semantics:
  asset_class:
    glossary_ref: "glossary://wealth/asset_class@v3"
  em_exposure_pct:
    definition_ref: "glossary://wealth/emerging_markets"
    rules:
      - "division == institutional: frontier markets INCLUDED in EM"
      - "division == retail: frontier markets EXCLUDED from EM"
    note: >
      This field's meaning depends on `division`. Consumers MUST NOT
      compare em_exposure_pct across divisions without normalising.

quality:
  suite_ref: "tests/fact_client_holding/@v4"
  expectations:
    - "risk_score between 1 and 10"
    - "completeness(risk_review_date) >= 0.99"
    - "em_exposure_pct distribution within 3 sigma of trailing 90d by division"
    - "referential: client_id exists in dim_client"

policy:
  retention: P7Y
  legal_basis: "contract; legitimate_interest (suitability monitoring)"
  approved_uses:
    - internal_suitability_monitoring
    - adviser_copilot_retrieval   # with division-aware normalisation
  prohibited_uses:
    - automated_client_facing_advice_without_human_review
  masking:
    - field: client_id
      method: hash_sha256
      applies_to: [non_production, analytics_read_only]

contracts:
  freshness_sla: 4_hours
  completeness_sla: 99.9_percent
  breaking_change_policy: semver_major
  deprecation_notice: 90_days

lineage:
  openlineage_namespace: meridian.production
  dataset_name: wealth.client_holding
  upstream: [custodian_feed_a, custodian_feed_b, mdm.dim_client, risk.assessment]
```

Read that definition and notice what it is. It is not documentation *about* the data; it is a specification that the data is deployed *from* and validated *against*. The semantic rule about frontier markets is not a note someone might find — it is a machine-readable warning bound to the field, sitting in the path of any consumer or any CI check. The quality expectations are not aspirations — they gate promotion. The approved and prohibited uses are not a governance slide — they are the policy the platform can enforce. The owner is not a name on a wiki — it is a person with a contact and a pager.

This single artefact *is* the capsule description: data, semantics, policy, contract, and lineage identifiers, defined and versioned together. Everything in the rest of Part II is about where this lives and how it runs.

### The capsule and the Open Data Contract Standard

A fair question at this point: is the capsule a bespoke format you now have to invent and maintain, or does it align with something the industry already agrees on? It aligns — and stating the alignment is what makes the spec credible rather than idiosyncratic. The **Open Data Contract Standard (ODCS)**, maintained by the Bitol project under the Linux Foundation AI & Data umbrella and originally contributed by PayPal, is a mature YAML specification for data contracts, and the seven capsule components map onto it almost one-to-one:

| Capsule component | ODCS section |
|-------------------|--------------|
| Structural metadata | `schema` (objects, properties, types, keys) |
| Semantic metadata | `description`, property `businessName`/`meaning`, custom semantic properties |
| Quality expectations | `quality` (rules, dimensions, thresholds) |
| Policy metadata | `roles`, `slaProperties`, custom classification/retention properties |
| Provenance / lineage | referenced via custom properties + OpenLineage identifiers |
| Operational contract | `slaProperties`, `support`, `team` |
| Data payload | the physical dataset the contract governs |

The book's position is simple: **a Data Capsule is an ODCS-compatible superset.** Where ODCS gives you a standard, tool-supported envelope for schema, quality, and SLAs, the capsule adds the insistence that *semantics and policy are first-class and co-versioned*, and that the whole thing is *bound to the data and deployed as one unit* rather than filed in a contract registry beside it. In practice, express your capsules as ODCS documents wherever the standard reaches, extend it with the semantic and policy richness the standard leaves open, and you get both interoperability and the full discipline. Appendix A gives the complete annotated spec with the ODCS mapping field by field. The point for now: adopting capsules does not mean abandoning standards; it means using a standard and refusing to stop where the standard currently stops.

### Versioning: what semver means for data

The `version: 4.1.0` at the top of the capsule is not decoration — it is a promise, and the promise only means something if everyone agrees what a version bump *signals*. Borrow semantic versioning from software, adapted to data:

- **Major** (`3.x → 4.0`): a *breaking* change. A column removed or retyped, a primary-key change, or — the one teams forget — a *semantic* change: redefining what `em_exposure_pct` counts is a breaking change even though the schema is untouched, because every consumer's interpretation just broke. Major bumps require consumer sign-off and the 90-day notice the contract promises.
- **Minor** (`4.0 → 4.1`): an *additive* change. A new nullable column, a new quality expectation, a widened-but-compatible type. Consumers on the old version keep working.
- **Patch** (`4.1.0 → 4.1.1`): a *correction* that changes nothing about the interface — a fixed description, a corrected lineage reference, a tightened test that was always meant to hold.

Watch it work across one real change history for `client_holdings`:

| Version | Change | Bump | Consumer action |
|---------|--------|------|-----------------|
| 3.2.0 | baseline | — | — |
| 3.3.0 | add nullable `esg_score` column | minor | none required |
| 4.0.0 | **redefine `em_exposure_pct` to be division-aware** (the Okonkwo fix) | **major** | 90-day notice; copilot must adopt division-aware normalisation |
| 4.1.0 | add `risk_review_date` completeness expectation to quality suite | minor | none required |
| 4.1.1 | correct a wrong glossary reference on `asset_class` | patch | none |

The 4.0.0 row is the important one: *the schema did not change, and it was still a breaking release,* because meaning changed. A versioning scheme that only tracks structure would have shipped the single most consequential change in the table's history as invisible. Capsule versioning tracks meaning as a first-class reason to bump, which is exactly what a semantic consumer like the copilot needs.

### The capsule lifecycle

A capsule is not born certified and does not live forever; it moves through a lifecycle, with a gate at each transition.

- **Draft** — the capsule exists in a branch; schema and semantics are being worked out. No consumer depends on it. *Gate to next:* owner review and a passing test suite.
- **Review** — proposed for publication; the domain owner (Maya) and affected consumers review the contract, especially the semantics and policy. *Gate:* sign-off recorded in the PR.
- **Published** — live, contracted, discoverable, consumable. Changes now follow the semver rules above. *Gate to next:* a decision to retire, triggered by falling usage or replacement.
- **Deprecated** — still available but flagged; consumers are on notice to migrate, within the deprecation window the contract states (90 days at Meridian). *Gate:* all registered consumers migrated or acknowledged.
- **Retired** — removed from production; the capsule and its history are archived for audit but no longer served.

The lifecycle matters because it is the answer to the lake's "permanent hoarding" from Chapter 2: a product that is *evaluated and deprecated when usage falls* rather than accumulated forever. It is also what makes the "who can I still depend on?" question answerable — a consumer knows that a Published capsule carries the full guarantee and a Deprecated one is on a clock.

## What you get the moment data and metadata are bound

Binding is not an aesthetic preference. Six things change the day the context is bound, and they are what you will be asked to show for the effort.

**Ambiguity drops, because meaning travels with the data.** Consumers derive structure, meaning, and usage rules from the artefact itself. The "what timezone are these timestamps in?" and "is this the CRM customer ID or the billing one?" questions stop arriving in your Slack, because the answer is bound to the field. Reliance on tribal knowledge — the single most fragile dependency in any data team — falls.

**Contracts become enforceable instead of aspirational.** When schema, expectations, and policy are bound to the dataset, breaking changes get caught in CI/CD rather than in a production incident. "We have a convention" is replaced by "we have a test that enforces this," which is the only kind of convention that survives contact with a deadline and a reorg.

**Governance moves into the fabric.** Policy metadata travels with the data — classification, masking, retention, geographic restriction, approved use. A processing engine can read those rules from the dataset and enforce masking automatically, rather than relying on every consumer to remember the policy and apply it correctly.

**Lineage and auditability become reliable.** Provenance is captured at production time and anchored in the dataset, so the regulator's "where did this come from?" is answerable from the data rather than from a heroic reconstruction. This is the difference between an audit you pass and an audit that pauses your model — a distinction we will return to in Part V, because it is the one that decides whether your AI initiative ships.

**The platform becomes resilient to its own tooling.** If the catalogue is down or the documentation portal is stale, the dataset remains interpretable, because its essential context lives with it. The catalogue becomes an index over authoritative metadata rather than a fragile single point of truth.

**And the data becomes AI-ready in the only sense that matters.** Tools and agents can inspect schema, constraints, and policy directly from the capsule — to propose mappings, check compatibility, generate tests, or detect a policy violation *before* execution. AI-ready data, as we will argue at length, is not clean data. It is data that can produce evidence about itself on demand. A capsule is the unit that can.

## A smaller capsule, so the discipline doesn't look heavy

The `client_holdings` capsule is a large fact table, and a reader could reasonably conclude that capsules are only worth it for big, high-traffic datasets. They are not, and the corrective is to see the discipline scaled *down* to something small — a piece of reference data — where the payoff is just as real and the artefact is a fraction of the size.

Meridian's asset-class taxonomy is exactly the kind of small, stable, high-reuse dataset that quietly underpins everything and is almost never governed. It is a lookup: a few dozen rows mapping an asset-class code to a name, a parent class, and — critically — the emerging-market flag that the whole suitability question turns on. As a capsule it is tiny:

```yaml
data_product: asset_class_taxonomy
version: 3.0.0
owners:
  domain: reference_data
  product_owner: ref-data-owner@meridian.example
datasets:
  - name: dim_asset_class
    format: iceberg
    grain: "one row per asset_class_code (current version)"
    classification: internal
    schema:
      primary_key: [asset_class_code]
      fields:
        - { name: asset_class_code, type: string, description: "Meridian taxonomy code, e.g. EQ-EM (equity) or FI-EM (fixed income)" }
        - { name: name, type: string, description: "Human-readable class name" }
        - { name: parent_code, type: string, description: "Parent class; null at root" }
        - { name: is_emerging_market, type: boolean, description: "EM flag — see semantics" }
semantics:
  is_emerging_market:
    definition_ref: "glossary://wealth/emerging_markets"
    note: >
      Base classification. Division-dependent frontier-market inclusion is
      applied downstream in client_holdings, NOT here. This flag is the
      institutional (frontier-inclusive) base; retail derives from it.
quality:
  expectations:
    - "asset_class_code unique and not null"
    - "parent_code references an existing asset_class_code or is null"
    - "no orphan cycles in parent hierarchy"
contracts:
  freshness_sla: 24_hours
  breaking_change_policy: semver_major
```

Notice how much this small capsule earns. It pins down, in one governed place, the *base* EM classification that `client_holdings` depends on — and it records, in its semantics, the exact relationship (this is the institutional base; retail derives from it) that, left unstated, produced the Okonkwo error. Reference data is where meaning is *defined* before it is *used*, so a governed reference capsule prevents drift at the source rather than catching it downstream. The lesson: capsules are not a heavyweight treatment reserved for big tables; they are the unit of governed meaning at any size, and the small ones are often the highest-leverage because everything else depends on them.

## Three capsule anti-patterns

Each of these anti-patterns is a plausible-looking failure that a well-intentioned team will drift into.

**The documentation capsule.** A YAML file exists, it has all seven sections, it looks like a capsule — and *nothing enforces it.* The quality expectations do not gate deployment; the semantics are not read by any consumer; the version is bumped by hand when someone remembers. This is the catalogue disease of Chapter 3 in a new file format: a description beside the data, drifting from it. The tell is that you could change the data without the capsule failing anything. A capsule that cannot fail a build is documentation wearing a capsule's clothes.

**The kitchen-sink capsule.** Every conceivable piece of metadata is stuffed in — every downstream note, every historical caveat, every tangential policy — until the capsule is so large that no one maintains it and its signal drowns in its own completeness. The hot/cold distinction from the next chapter is the discipline that prevents this: embed what a consumer needs *now*, reference the rest. A capsule is a contract, not an archive.

**The orphan capsule.** A beautifully specified capsule with no named owner and no registered consumer — governance performed for its own sake. With no consumer, no one notices when it drifts; with no owner, no one can fix it if they did. This is the failure Chapter 20 returns to as the ownership surface: a perfect description of a thing nobody answers for is still an orphan. Every capsule should be able to name, from day one, the person who owns it and at least one consumer who would notice if it broke.

The three share a root: a capsule is only real when it is *enforced, bounded, and owned.* A YAML file is necessary and nowhere near sufficient.

## Why it is still hard

A discipline sold as effortless gets abandoned the first time it costs something. Here is what makes capsules genuinely hard.

**Legacy and heterogeneity.** Meridian, like any firm its age, runs a mix of old relational systems, a mainframe in the corner that processes settlements, petabytes of CSV and Excel in object storage, and BI tools with business logic baked into them. You will not retrofit atomic units across all of it, and you should not try. The realistic move is to start with new pipelines and the handful of datasets where ambiguity already hurts, and to define migration patterns rather than attempt heroic rewrites.

**Tooling fragmentation.** Iceberg, Snowflake, and BigQuery describe schema, policy, and lineage in incompatible ways. Binding across them requires deliberate translation layers and some compromise. This is real work, and Chapter 13 is about doing it with open standards so the work is done once rather than per-tool.

**Some metadata is more sensitive than the data.** Internal topology, contractual constraints, and the structure of your controls can be more revealing than the records themselves. Binding metadata to data does not mean exposing all of it to everyone; it means layered metadata, encryption, and role-aware views. The capsule is bound, not naked.

**Performance and practicality.** Embedding every component inside every file bloats the files. The pragmatic design — hot metadata embedded, cold metadata referenced — is the subject of the next chapter, and it is what keeps the discipline affordable at scale.

**Organisational boundaries.** Data engineering, governance, security, and legal use different tools and answer to different incentives. Adopting capsules surfaces questions that the old, drift-tolerant world let everyone avoid: who defines policy, who approves a schema change, who owns the dataset when it spans three teams. These are operating-model problems, not technical ones, and pretending otherwise is how good architectures die in committee. Part V is where we confront them directly.

## The shift, and where it goes next

The mental shift the rest of this book asks you to make is small to state and large to live: **stop treating context as commentary on the data and start treating it as a property of the data.** A column without a bound definition is not a documented column with the documentation temporarily missing — it is an undefined column that will, sooner or later, be interpreted by a consumer who guesses. When that consumer is an AI agent answering a suitability question at machine speed, the guess becomes a decision before anyone reviews it.

Meridian's data was good and still produced a wrong, consequential answer, because good is the wrong target. The target is data that can explain and defend itself — data with its context bound, co-versioned, and deployable as one unit. That is a Data Capsule, and naming it is the first move.

In maturity terms, defining your first capsule — taking one important dataset and binding agreed definitions, quality expectations, policy, and ownership to it — moves that dataset from **Level 0 (Flat: technical metadata only)** to **Level 1 (Defined: business meaning agreed and linked)**. It is one dataset, not the enterprise. That is deliberate. Capsules are adopted one high-value dataset at a time, and the next chapter shows where that first capsule actually lives: not in a catalogue, not in a wiki, but in the table format your data already sits on.

## Further reading

- The *Open Data Contract Standard* (ODCS), maintained by the Bitol project under the Linux Foundation AI & Data — the specification the capsule maps onto; pin to ODCS v3.x for durability. Appendix A gives the field-by-field mapping.
- On self-describing formats, the Apache Parquet, ORC, and Avro specifications; on open table formats, the Apache Iceberg, Delta Lake, and Apache Hudi documentation (all developed in Chapter 6).
- On semantic versioning as adapted here, the *Semantic Versioning* specification (semver.org) — read alongside this chapter's argument that a semantic redefinition is a major change even when the schema is untouched.

> **Chapter summary.** Data and metadata drift apart by default, an artefact of how platforms evolved rather than anyone's carelessness — and at AI scale, that drift turns good data into confident wrong answers. A **Data Capsule** binds seven components (payload, structural, semantic, quality, policy, provenance, contract) into a single, co-versioned, CI/CD-deployable unit. Binding means enforced reference and co-versioning, not physical colocation. The technology already exists; what is usually missing is the decision to make "data plus context, atomic" the default. Defining your first capsule takes one dataset from Level 0 to Level 1.

> **Try this.** Take the one dataset whose meaning your team argues about most. Write its capsule definition — the seven components, in YAML, in a repo. You will discover, in the act of writing the semantic and policy sections, exactly which of your "agreed" definitions are not actually agreed. That discovery is the point.
