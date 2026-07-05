# Chapter 11 — The Toolchain That Makes It Real

> **Where you are:** Part III — *Automate*. Chapter 10 made the case for Metadata as Code as a principle. This chapter is the workshop: the open standards and platform features that turn the principle into a running stack. Maturity target: **Level 1 → 2** — consolidating repeatable binding into a working, event-ready stack. A note before we start — every tool named here is *illustrative*. The patterns are the point; the logos are interchangeable.

---

## From principle to plumbing

A principle that cannot be implemented is a slogan, and "treat metadata as code" would be a slogan if there were no concrete machinery to capture lineage, validate contracts, enforce quality, and apply policy automatically. There is, and most of it is open, mature, and probably already partly installed in your platform. This chapter walks Meridian through assembling it.

The architecture has two halves. The first is a foundation of **open standards** that keep metadata portable and prevent any single vendor from owning your governance. The second is a set of **platform-native features** that give those standards teeth inside whatever warehouse or lakehouse you actually run on. The art is in combining them: standards for portability, platform features for enforcement, version control underneath both so every change is traceable and reversible. We take the foundation first.

## The open standards that do the load-bearing work

A small set of open standards, working together, covers the essential metadata surfaces: lineage, transformation, contracts (in motion and at rest), and quality.

### OpenLineage: the universal lineage language

OpenLineage is an open framework for collecting lineage. At its heart is an extensible specification that any tool can speak — think of it as *HTTP for data lineage*, a standard protocol so that Airflow, Spark, dbt, Flink, and dozens of other systems can all report what they did in the same language. When Meridian's nightly job that builds `fact_client_holding` completes, it emits an event that names the job, its inputs (the custodian feeds, `dim_client`, the risk assessment), its outputs (the holdings table and the snapshot it produced), and facets carrying schema and data-quality detail:

```json
{
  "eventType": "COMPLETE",
  "job": { "namespace": "meridian.wealth", "name": "build_client_holding" },
  "inputs": [
    { "namespace": "custodian.a", "name": "positions",
      "facets": { "schema": { "fields": [ {"name":"account_status","type":"string"} ] } } }
  ],
  "outputs": [
    { "namespace": "iceberg.wealth", "name": "fact_client_holding",
      "facets": { "stats": { "rowCount": 4193322 } } } ]
}
```

Why this matters for Meridian: the 7 a.m. zeros of the previous chapter took two hours to trace because lineage was scattered and stale. With OpenLineage events emitted automatically by every job, the dependency from `account_status` to fourteen downstream tables is captured continuously and consistently, in one language, the moment each job runs. Root-cause analysis stops being archaeology and becomes a query — and as we will see in Chapter 12, the same events let the platform *block* the break before it happens, not just explain it afterward.

### dbt artefacts: the transformation metadata goldmine

dbt generates artefacts with every invocation — `manifest.json` (the full project DAG and metadata for every model, test, source, and snapshot), `catalog.json` (the runtime schema the database actually produced), `semantic_manifest.json` (the metrics layer). These are not a side effect to be ignored; they are a goldmine of governance metadata produced for free on every run. When Meridian's analysts annotate a dbt model with owner, criticality, PII level, business definitions, and tests, those annotations become machine-readable artefacts that documentation, lineage, and quality monitoring can all consume without anyone re-entering anything:

```yaml
models:
  - name: fact_client_holding
    description: "Client holdings with suitability context, refreshed 4-hourly"
    meta:
      owner: maya.osei@meridian.example
      business_critical: true
      pii_level: "pseudonymous"
    columns:
      - name: client_id
        description: "MDM golden-record client identifier"
        tests: [unique, not_null]
      - name: em_exposure_pct
        description: "EM percent; DIVISION-DEPENDENT definition — see contract"
        tests:
          - dbt_expectations.expect_column_values_to_be_between:
              min_value: 0
              max_value: 100
```

The discipline is to treat `schema.yml` not as optional documentation but as part of the deliverable — tests as executable documentation, definitions as governed metadata. After a run, the artefacts drive the catalogue, the lineage graph, and the quality dashboard automatically. This is the single highest-leverage starting point for most teams, because the tool is already there and the metadata is already half-written.

### AsyncAPI: contracts for data in motion

OpenLineage and dbt cover data at rest and in transformation, but Meridian's intraday position feed is data in *motion*, and motion needs its own contract. Just as OpenAPI standardised REST contracts, **AsyncAPI** standardises event streams — a way to describe, document, and version the asynchronous messages a system exchanges. An AsyncAPI spec for Meridian's position events declares the channel, the message payload, the required fields, and their semantics, and — crucially — it lives in the repository and runs in CI:

```yaml
asyncapi: '3.0.0'
info: { title: Position Events, version: '2.1.0' }
channels:
  position/updated:
    address: 'wealth.position.updated'
    messages:
      positionUpdated:
        payload:
          type: object
          properties:
            clientId: { type: string, format: uuid }
            holdingId: { type: string }
            eventTs: { type: string, format: date-time }
          required: [clientId, holdingId, eventTs]
```

Wired into CI/CD, the spec is validated and linted on every change, and consumer-compatibility tests run against it, so a producer cannot ship a breaking change to the position stream without the pipeline catching it. The streaming equivalent of the column-rename disaster — a required field removed from an event — is stopped in CI with a clear migration path, rather than discovered three weeks later when the suitability model's features quietly degrade.

### Great Expectations: quality as executable documentation

The fourth standard makes quality *executable*. Great Expectations expresses data-quality rules as verifiable assertions that read clearly to technical and non-technical people alike — "this table should have between X and Y rows," "this column must be unique," "this field must match this format for compliance." Meridian's expectations for the holdings data are not prose in a quality policy; they are code that runs:

```python
ExpectationConfiguration(
    expectation_type="expect_column_values_to_be_between",
    kwargs={"column": "risk_score", "min_value": 1, "max_value": 10,
            "meta": {"business_rule": "Risk tolerance scale", "criticality": "high"}})
```

These can live as a standalone suite or, more conveniently for a dbt shop, as `dbt-expectations` tests inside the model spec, so quality rules sit beside the model they govern and run on every build. In CI/CD, a failed checkpoint raises an error and the deployment stops. Quality stops being a dashboard someone checks and becomes a gate nothing gets through — the difference, again, between metadata that describes and metadata that enforces.

### ODCS: the contract at rest

The four above govern lineage, transformation, streams, and quality; the *at-rest* contract — the promise a stored data product makes about its schema, semantics, and SLAs — has its own standard, and the book met it in Chapter 5. The **Open Data Contract Standard (ODCS)**, maintained by the Bitol project under the Linux Foundation AI & Data umbrella, is the YAML envelope the capsule is a superset of. In this toolchain it plays the same role the others do: the contract spec lives in Git (the `contracts/` path of Chapter 10's repository), is validated and compatibility-checked in CI, and travels with the data product it governs. It is listed here for completeness because a stack that governs data in motion but leaves data at rest to convention has a hole exactly where regulated products live.

Take these together and the value compounds: OpenLineage answers *what transformed what*, dbt artefacts capture *how it was transformed and what it means*, AsyncAPI governs *data in motion*, Great Expectations enforces *whether it is good enough*, and ODCS is the contract the product ships under *at rest*. All of them are open, all keep your metadata portable, and all produce machine-readable artefacts that the rest of the stack can consume. Portability is not a purity concern; it is what lets Meridian change a warehouse vendor in three years without re-implementing its entire governance layer.

### What each standard covers — and what none of them does

Five standards, five surfaces; it helps to see the coverage as a grid, because the gaps are as instructive as the coverage.

| Standard | Captures | Where it runs | Maturity |
|----------|----------|---------------|----------|
| OpenLineage | job→dataset lineage, run facets | orchestrators, engines | high, dozens of integrations |
| dbt artefacts | transformation graph, tests, docs, meta | every dbt run | high (dbt shops) |
| AsyncAPI | event/stream contracts, payload schemas | CI, gateways, registries | growing |
| Great Expectations | executable quality assertions | CI, pipelines | high |
| ODCS (data contracts at rest) | capsule/contract specs: schema, semantics, SLAs | Git, CI-validated | growing (Bitol / LF AI & Data) |

### Where the standards stop

Several important surfaces are covered by *none* of these, and a team that assumes the standards are complete will be surprised. **Business-glossary exchange** — moving governed term definitions between systems — has no dominant open standard; you bridge it with custom properties and, at the semantic layer, the vocabularies of Chapter 13. **Policy-language portability** — expressing "mask this for that role" once and enforcing it across Snowflake, Iceberg, and BigQuery — is not standardised; each platform has its own dialect, which is why Chapter 13's integrator has to translate. **BI-tool lineage** — the last hop from a warehouse table to the dashboard or copilot that consumes it — is frequently missing, because BI tools are inconsistent about emitting it; the pragmatic bridge is the external-lineage APIs below. Naming these gaps is not a criticism of the standards; it is the map of where you will still have to build glue, and knowing it up front is what keeps the "assembled stack" from having silent holes.

## Platform-native features: where the standards get teeth

Open standards keep metadata portable, but enforcement happens inside the platform your data actually lives on. The major platforms now ship first-class governance features that turn a classification tag into an automatically applied control. Meridian runs Iceberg as its primary lakehouse with a Snowflake footprint for certain analytics marts, so it uses a mix of these; the patterns generalise across vendors.

**Tag-based policy automation (Snowflake-style).** You define governance tags with allowed values, define masking policies that vary the returned value by role, then *bind the policy to the tag*. From then on, any column that receives that tag is masked automatically — now or in the future, without anyone touching the policy again. At Meridian, tagging `client_email` as `EMAIL` means an analyst sees `joh***@domain.com` while a compliance admin sees the full value, and a new email column added next quarter inherits the protection the instant it is tagged. The automation payoff is the point: classify once, enforce forever.

```sql
ALTER TAG governance.pii_type SET MASKING POLICY pii_email_mask WHEN 'EMAIL';
ALTER TABLE wealth.dim_client MODIFY COLUMN client_email
  SET TAG governance.pii_type = 'EMAIL';
```

**External lineage integration (Unity Catalog-style).** Captured lineage usually dies at the platform boundary — it knows the warehouse tables but not the Tableau dashboard or the copilot that consumes them. Unity Catalog and equivalents let you add external lineage programmatically, linking a governed table to the BI report or AI agent downstream of it, so the lineage graph runs end to end from custodian feed to the answer an adviser sees. This matters enormously for impact analysis: when Meridian changes `fact_client_holding`, it can enumerate not just the affected tables but the affected *dashboards and the copilot*.

**Column-level policy tags (BigQuery-style).** A taxonomy of sensitivity levels, applied as policy tags to specific columns, with IAM bindings that grant only the right group fine-grained access. The pattern is the same as tag-based masking, applied to access rather than display: classify the column, and access control follows automatically.

**Event-driven metadata automation (DataHub Actions-style).** This is the most powerful pattern and the bridge to the next chapter. An event-driven framework watches the metadata graph for changes and *reacts*. When a schema change is detected on a Kafka stream of metadata events, it can classify new PII columns, apply governance tags, run a Great Expectations suite, and — for high-risk types like national-identity numbers — open a Jira ticket and post to Slack, all automatically:

```yaml
filter:
  conditions:
    - type: "dataset_schema_change"
action:
  type: "python"
  config:
    script: |
      def handle_schema_change(event):
          pii = classify_pii_columns(event.entityUrn)
          for col, pii_type in pii.items():
              apply_tag(event.entityUrn, col, f"PII_{pii_type}")
          run_great_expectations_suite(event.entityUrn)
          if any(t in ('SSN','PASSPORT') for t in pii.values()):
              create_jira_ticket(...); notify_slack("#wealth-governance", ...)
```

Notice what just happened: a new column appeared, and the platform classified it, protected it, tested it, and queued it for review — with no human in the loop until the moment human judgement is actually needed. That is the whole subject of Chapter 12.

### The platform matrix

Which native hook lives where is the practical question when you assemble the stack, so here is the matrix across the platforms Meridian and its peers actually run, including the open-source metadata backends:

| Platform | Tags / classification | Column masking / access | Lineage | Contracts |
|----------|:---:|:---:|:---:|:---:|
| Snowflake | object tags | tag-based masking, row policies | Access History, external | dbt/GX in CI |
| Databricks / Unity | tags | ABAC, column masks | built-in + external API | dbt/GX in CI |
| BigQuery | policy tags | column-level ACL, DLP | Data Catalog lineage | dbt/GX in CI |
| Microsoft Fabric / Purview | sensitivity labels | Purview policies | Purview lineage | via ADF / connectors |
| DataHub (OSS) | tags, terms | — (indexes external) | OpenLineage ingest | contract assertions |
| OpenMetadata (OSS) | tags, glossary | — | OpenLineage ingest | test integration |
| Marquez (OSS) | — | — | OpenLineage reference store | — |

The reading is not "pick the platform with the most ticks"; it is "know which slot each control lives in, because your capsule declares the policy and the *platform* enforces it." Meridian runs Iceberg with a governance-capable catalogue for enforcement and DataHub as the open, cross-platform index — a deliberate split so no single vendor owns the metadata.

### Testing the metadata itself

One discipline is easy to forget: if metadata is code, it should be *tested* like code, not just used to test data. Meridian runs three kinds of metadata test. **Unit tests on policies** — assert that the PII masking policy actually redacts for an analyst role and reveals for a compliance role, run in CI whenever `policies/` changes, so a policy that silently stops masking fails the build. **Contract tests between specs** — assert that a consumer's expected schema is still satisfied by a producer's contract, so an incompatibility surfaces at the seam before deployment. **Smoke tests post-deploy** — after a capsule ships, confirm the live table's tags, masking, and lineage match what the spec declared (the reconciliation check of Chapter 10, run as a gate). Testing the metadata closes the loop the whole part is built on: metadata that governs data is itself governed, versioned, and verified, rather than trusted.

## Wiring it together

These pieces are not a pile of tools; they assemble into a coherent flow, and seeing the shape keeps the parts from feeling like a shopping list. Picture six layers, with metadata flowing through and feeding back:

<!-- FIG 11.1: six-layer metadata flow — (1) sources + version control → (2) CI/CD validation → (3) active-metadata processing → (4) metadata stores/catalogues → (5) governance + API layer → (6) consumption; metadata flows through and feeds back laterally. -->

1. **Sources and version control** — Git repositories holding schemas, dbt models, contracts, and capsule specs, alongside the data platforms themselves.
2. **CI/CD and validation** — the pipeline that lints, tests, and validates every metadata change before it deploys (where the four standards' checks run).
3. **Active metadata processing** — where lineage events, quality results, and schema-change events are captured and acted on (Chapter 12).
4. **Metadata stores and catalogues** — open-standards-based storage that persists the assembled metadata (Chapter 13).
5. **Governance and API layer** — where policies, access controls, and compliance rules are enforced and exposed via APIs.
6. **Consumption** — discovery, analytics, BI, the copilot, and external integrations like Slack and Jira.

The flow is not strictly one-directional; assets move laterally and feed back, stewards' enrichments flowing back to source (a pattern Chapter 14 makes precise). The four insights that hold the architecture together are worth committing to memory: **open standards** ensure portability and prevent lock-in; **platform features** provide deep enforcement; **event-driven automation** delivers real-time governance without manual intervention; and **version control** underneath all of it makes every change traceable and reversible. Together they turn metadata from a documentation burden into an active participant in how the platform runs.

## One change, traced through the whole stack

The best way to see the assembled stack is to follow a single change through it, because the value is not in any one component but in the *sequence* they fire in. Take the change that opened Chapter 10 — a custodian renames `acct_status` to `account_status` — and trace it through Meridian's toolchain, now assembled:

1. **The custodian's stream** carries the change; the **AsyncAPI contract** in the schema registry is set full-compatible, so the registry *rejects* the incompatible field rename at publish — and the break is stopped at the source before a single downstream row is affected. (In the batch case, the rename arrives in a file and is caught one step later.)
2. Had it slipped past the registry, the **dbt contract** on the affected model fails at build: `contract: {enforced: true}` turns the schema drift into a hard CI failure, not a silent absorption.
3. **OpenLineage** events, emitted by the affected jobs, let **impact analysis** enumerate the fourteen downstream tables, three dashboards, and the copilot that depend on the field — the blast radius that took two hours to trace by hand is now a query.
4. **Great Expectations** would have caught the *symptom* (a spike in nulls) at the next run had the structural guards failed — defence in depth.
5. The **active-metadata layer** (Chapter 12) routes the blocked-deployment alert to Maya with the impact report attached, and the **catalogue** (DataHub) refreshes from the corrected spec so discovery stays true.

Five components, one sequence, and the 7 a.m. zeros never happen — the change is caught at step 1 or 2, and even a total failure of the structural guards is backstopped at step 4 and surfaced at step 5. That is what "assembled" means: not five tools installed, but five guarantees firing in order, each catching what the last might miss.

## What Meridian now has

By assembling this stack, Meridian's `client_holdings` capsule from Part II gains an automated nervous system. OpenLineage events trace its provenance continuously. Its dbt artefacts feed the catalogue and quality monitors without manual entry. Its streaming inputs are governed by an AsyncAPI contract that fails CI on a breaking change. Its quality rules are Great Expectations tests that gate every deployment. Its `confidential` classification drives masking automatically through platform-native tags. And a schema change anywhere in its lineage triggers classification, testing, and notification without a human initiating any of it.

The tools are open, mature, and largely already present — that is the chapter's quiet argument. What was missing was the decision to wire them together around the capsule as the unit of governance. With the stack assembled, the third principle of Metadata as Code — event-driven automation — is ready to be taken to its conclusion. Capturing lineage and gating quality is necessary, but it is still, in a sense, passive: metadata that is correct and waiting to be read. The next chapter activates it. It shows metadata moving from the documentation layer to the *control plane* of the platform — detecting, deciding, and acting in real time, including supplying the live context that Meridian's adviser copilot needs before it dares to answer.

## Further reading

- The canonical documentation for the standards this chapter assembles: OpenLineage (openlineage.io), dbt (artefacts, contracts, and tests), AsyncAPI, Great Expectations, and the *Open Data Contract Standard* (Bitol / LF AI & Data).
- On platform-native enforcement, the Snowflake object-tagging and masking-policy docs, Databricks Unity Catalog lineage and ABAC docs, and BigQuery policy-tags / DLP docs — verify current capabilities, which move quickly.
- On the open-source metadata backends indexed here, the DataHub, OpenMetadata, and Marquez project documentation.

> **Chapter summary.** Metadata as Code is implemented from two halves: open standards for portability and platform features for enforcement. Four standards do the load-bearing work — **OpenLineage** (lineage as a universal protocol), **dbt artefacts** (transformation metadata for free on every run), **AsyncAPI** (contracts for data in motion), and **Great Expectations** (quality as executable, deployment-gating tests). Platform-native features give them teeth: tag-based masking, external lineage, column-level policy tags, and event-driven automation frameworks that classify, protect, test, and notify on schema change. Wired through a six-layer flow with version control underneath, the stack turns metadata into an active participant. The tools are open and mostly already present; the missing step was assembling them around the capsule.

> **Try this.** If you run dbt, open one model's `schema.yml` and count how much governance metadata is already there versus how much could be — owner, definitions, tests, classification. The gap is governance you can capture this week, for free, from a tool you already run. Start there before buying anything.
