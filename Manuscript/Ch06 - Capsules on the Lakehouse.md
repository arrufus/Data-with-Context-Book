# Chapter 6 — Capsules on the Lakehouse: Delta, Iceberg, and Hudi

> **Where you are:** Part II — *Bind*. Chapter 5 defined the capsule. This chapter gives it a home. Maturity target: consolidating **Level 1**, and making it durable enough to build on.

---

## "Where does this thing actually live?"

When Maya took the client-holdings capsule from Chapter 5 to her platform team, the first question was not whether the idea was good. The idea was obviously good — nobody at Meridian wanted another Okonkwo. The first question was the practical one that kills good ideas when it has no answer: *where does this thing actually live?* A YAML file in Git is a description. The data is fifty terabytes of holdings sitting in object storage. If the capsule is one thing and the data is another, you have just invented a new document to keep in sync with the data — the exact problem capsules exist to abolish.

The honest answer is that you do not need to build anywhere for capsules to live, because the place already exists and most teams are already standing on it. For the overwhelming majority of modern data platforms, the natural home for a Data Capsule is the **lakehouse table format**: Delta Lake, Apache Iceberg, or Apache Hudi. These formats have quietly become the point where data files and metadata already converge. They were built to track schema, manage versions, and store table-level properties — which is to say they were built, without using the word, to be capsules. You do not need to invent a metadata layer. You need to use the one you already have, deliberately.

This chapter is about doing exactly that: mapping the seven capsule components onto lakehouse primitives, the patterns that hold across all three formats, the format-specific mechanics, and how to migrate the naked Parquet you already have into capsules without halting the business to do it.

## Why the lakehouse is the natural home

The platform landscape has been converging for years, and most organisations now arrive at the same shape: object storage at the bottom, an open table format in the middle, one or more compute engines on top. Whether you call the result a lakehouse, a data-mesh node, or just "the platform," the table format has become the nucleus that binds everything together. Meridian's is no different — S3 underneath, Iceberg in the middle, Spark and Trino and a Flink job or two on top.

This convergence is the good news. Instead of bolting a separate metadata system onto the side of your platform, you can lean on what the table format already provides. The table's transaction log, its schema definition, its partition spec, and its table properties form a natural anchor point. Everything else — catalogs, documentation portals, lineage graphs — can *reference that anchor* rather than competing to be the source of truth.

All three formats share a philosophy that aligns precisely with the capsule concept: **a table is an atomic object consisting of data files, metadata, and commit history.** They differ in emphasis. Delta Lake is transaction-log centric, with deep roots in the Databricks and Spark ecosystem; its log records every change, making schema evolution and metadata history straightforward to track. Iceberg is specification-first and engine-agnostic, with mature schema evolution and broad support across Spark, Trino, Flink, Dremio, and DuckDB. Hudi was built around change data capture and incremental processing; its commit timeline tracks every write, which suits streaming and CDC-heavy workloads. Three different bets, one shared property: data and metadata move together, atomically, in a versioned commit. That property is the whole foundation.

## Mapping the seven components to lakehouse primitives

The mapping from capsule components to table-format constructs is remarkably consistent across Delta, Iceberg, and Hudi. Hold the seven components from Chapter 5 in mind and watch where each one lands.

- **Structural metadata** → the table's schema, partition spec, sort order, and constraints. This is native; the format exists to track it.
- **Semantic metadata** → column comments / field documentation for the inline definitions, plus table properties that reference external vocabulary (`glossary.term = wealth/asset_class@v3`) for the richer ones.
- **Quality expectations** → references in table properties to a test suite (`quality.suite = tests/fact_client_holding/`), with the suite itself in the repo and run by CI before promotion.
- **Policy metadata** → classification, legal basis, retention, and masking expressed as table properties and, where the catalog supports it, as enforced tags and masking policies.
- **Provenance and lineage anchors** → the version, snapshot, or commit ID of each write, emitted as OpenLineage events that connect jobs to specific table versions.
- **Operational contract** → owner, domain, SLA targets, and contact stored as table properties, queryable by your monitoring system so alerts route to the right place.
- **Data payload** → the data files themselves, governed by all of the above in the same commit.

The key realisation is that you are not adding a parallel structure to the table. You are *populating fields the table format already gives you* and treating them as load-bearing rather than decorative.

### Hot metadata and cold metadata

Chapter 5 promised to make the binding-without-colocation idea concrete, and this is where it becomes concrete. Not all metadata should travel inside the table, and trying to make it do so is how you bloat your files and slow your queries. Split it.

**Hot metadata** is anything needed for query execution or an immediate governance decision: the schema, column statistics, primary classification, partition information. This belongs directly in the table definition and properties, because a query planner and a masking policy need it *now*, in the same place as the data.

**Cold metadata** is the rich, heavy, occasionally-consulted material: detailed lineage graphs, full documentation, historical audit reports, cost analyses. This can live in external systems and be *referenced* from the table by name, ID, and version. You do not need the six-month audit history embedded in every Parquet footer; you need a stable identifier that retrieves it when asked.

This split is the practical resolution of the colocation question. Hot metadata is colocated because it must be. Cold metadata is bound by stable, co-versioned reference because colocating it would cost more than it is worth. Both are part of the capsule. Neither is allowed to drift — the reference is enforced, not hoped.

## Three patterns that hold across all three formats

Before the format-specific mechanics, three architectural patterns apply regardless of whether you choose Delta, Iceberg, or Hudi. If you internalise only these three, you have most of the discipline.

### Pattern 1: The table is an atomic object

All three formats treat a table as a single, versioned unit. When you write, the commit does not merely add data files — it updates metadata atomically in the same commit. Change a schema, add a column comment, modify a table property, and that change is recorded in the same commit history as the data change. There is no separate versioning for data and for metadata. They move together. This atomicity is not a convenience; it *is* the capsule's co-versioning guarantee, provided by the format for free. Your job is to not undermine it.

### Pattern 2: Table definitions are code

The fastest way to break the capsule model is to manage tables through a web console or ad-hoc SQL run from someone's laptop. The moment a human changes a table by hand, the Git definition and the live table diverge, and you are back in drift. So the table definition — the `CREATE` and `ALTER` statements, the properties, the references to contracts and tests — lives in Git alongside the transformation code, and the live table is treated as a *deployment artifact* produced from it.

How you implement this varies. Some teams use migration tools like Flyway or Alembic adapted for data platforms. Others keep "table spec" YAML — exactly the capsule definition from Chapter 5 — that CI/CD translates into DDL. The mechanism matters less than the principle: **the definition in Git is the source of truth; the table in the lakehouse is a deployment of it.** This also buys you code review for data structure. When someone wants to add a column, change a constraint, or alter a classification, that change goes through the same pull-request process as application code — reviewed, tested, traceable, and revertible. At Meridian, the institutional-versus-retail definition change that caused the Okonkwo error would have arrived as a pull request against `em_exposure_pct`, where a reviewer who understood both divisions would have seen it before it ever reached production.

### Pattern 3: Catalogs are indexes, not sources of truth

Unity Catalog, AWS Glue, Apache Nessie, Polaris, Collibra, Atlan — these are valuable for discovery and access control, and you should use them. But they must function as an *index over* capsules, not as the authoritative source of table metadata. The real metadata lives in the table format, in object storage, next to the data. The catalog provides a searchable, queryable window onto that reality.

The test of whether you have this right is what happens during an argument. If the catalog says one thing and the table's own metadata says another, **the table wins**, and the catalog needs refreshing or repairing. A team that resolves that disagreement the other way — trusting the catalog over the data — has rebuilt the drift problem with a more expensive tool. This single principle, applied consistently, is what turns a catalog from a documentation graveyard into a live index of the truth.

## The same capsule on each format

To make the patterns concrete, here is Meridian's `fact_client_holding` capsule expressed on each of the three formats. The capsule definition from Chapter 5 is the source of truth; what follows is how CI deploys it.

### On Delta Lake

Delta stores its metadata in a transaction log — JSON files with periodic Parquet checkpoints — that records every change. The current state of the table is the result of replaying that log, which means schema evolution, constraint additions, and property changes are all first-class, versioned events. Delta supports `CHECK` and `NOT NULL` constraints, treats column comments as first-class metadata, and lets table properties store arbitrary key-value pairs. On Databricks with Unity Catalog you also get classification tags, row-level security, and column masking integrated into the table.

```sql
CREATE TABLE wealth.fact_client_holding (
    client_id        STRING NOT NULL COMMENT 'Meridian client id (MDM golden record); PII pseudonymous',
    division         STRING NOT NULL COMMENT 'Servicing division: retail | institutional',
    holding_id       STRING NOT NULL COMMENT 'Instrument id, FIGI where available',
    asset_class      STRING COMMENT 'Asset class per Meridian taxonomy v3 (glossary: wealth/asset_class@v3)',
    em_exposure_pct  DECIMAL(5,2) COMMENT 'EM percent; DIVISION-DEPENDENT definition — see contract',
    risk_score       INT COMMENT 'Client risk tolerance 1 (conservative) to 10 (aggressive)',
    risk_review_date DATE COMMENT 'Last risk assessment date; null = never assessed',
    valuation_date   DATE NOT NULL COMMENT 'As-of valuation date (UTC)'
)
USING DELTA
PARTITIONED BY (valuation_date)
TBLPROPERTIES (
    'owner'                = 'wealth-client-domain',
    'product_owner'        = 'maya.osei@meridian.example',
    'domain'               = 'wealth_client',
    'classification'       = 'confidential',
    'sla.freshness_hours'  = '4',
    'contract.path'        = 'contracts/fact_client_holding.yaml',
    'quality.suite'        = 'tests/fact_client_holding/',
    'semantics.em_def'     = 'glossary://wealth/emerging_markets',
    'glossary.term'        = 'client_holding'
);
ALTER TABLE wealth.fact_client_holding
  ADD CONSTRAINT risk_score_range CHECK (risk_score BETWEEN 1 AND 10);
```

The `contract.path` and `quality.suite` properties point at files in the same repository. CI validates that those files exist and that the tests pass before promoting the change. The `CHECK` constraint turns the capsule's "risk_score between 1 and 10" expectation from a written rule into an enforced one.

**Gotcha worth knowing:** Delta's tight Spark/Databricks integration is both its strength and its limit. If a data scientist queries this table from DuckDB or Trino, they may not see every column comment or property. So for Meridian, the rule is that any semantic information that *must* reach every consumer — like the division-dependent warning on `em_exposure_pct` — is duplicated in the contract file (which every consumer can read) rather than living only in a Delta comment that some engines drop.

### On Apache Iceberg

Iceberg is specification-first and engine-agnostic, which is why Meridian standardised on it: in a firm running Spark, Trino, and Flink against the same tables, having all engines read and write with full metadata support is worth a great deal. An Iceberg table is metadata files, manifest lists, and manifests describing current state and full history; each write creates a snapshot you can time-travel to. Schema evolution is mature — add, rename, reorder, even hide columns without rewriting data — and field-level documentation lives in the schema itself.

```yaml
# table-specs/wealth/fact_client_holding.yaml  (CI translates to DDL)
table: wealth.fact_client_holding
format: iceberg
schema:
  - { name: client_id,        type: string,         required: true,  doc: "MDM golden record; PII pseudonymous" }
  - { name: division,         type: string,         required: true,  doc: "retail | institutional" }
  - { name: holding_id,       type: string,         required: true,  doc: "Instrument id, FIGI where available" }
  - { name: asset_class,      type: string,                          doc: "Meridian taxonomy v3" }
  - { name: em_exposure_pct,  type: decimal(5,2),                    doc: "EM percent; DIVISION-DEPENDENT — see contract" }
  - { name: risk_score,       type: int,                             doc: "Risk tolerance 1..10" }
  - { name: risk_review_date, type: date,                            doc: "Last assessment; null = never" }
  - { name: valuation_date,   type: date,           required: true,  doc: "As-of valuation date (UTC)" }
partition_spec:
  - { field: valuation_date, transform: month }
properties:
  owner: wealth-client-domain
  domain: wealth_client
  classification: confidential
  sla.freshness_hours: "4"
  contract.path: contracts/fact_client_holding.yaml
  quality.suite: tests/fact_client_holding/
  semantics.em_def: "glossary://wealth/emerging_markets"
```

Here the snapshot ID is the lineage linchpin: OpenLineage records which job produced which snapshot, and quality runs gate snapshot promotion — tests must pass before a new snapshot becomes the current version consumers see. **The gotcha for Iceberg is catalog choice.** Hive Metastore, Glue, Nessie, and Polaris differ in their support for tags and access control, so pick the catalog whose governance features you actually need, because the table format alone will not provide column masking.

### On Apache Hudi

Hudi earns its place when change data capture or incremental processing is central — which, for Meridian, describes the intraday position feed rather than the daily holdings snapshot. Hudi's commit timeline is a sequence of "instants" recording writes and table services; record keys and precombine fields define how upserts and deduplication behave.

```python
hudi_options = {
    'hoodie.table.name': 'fact_position_intraday',
    'hoodie.datasource.write.recordkey.field': 'client_id,holding_id',
    'hoodie.datasource.write.precombine.field': 'event_ts',
    'hoodie.datasource.write.partitionpath.field': 'valuation_date',
    'hoodie.datasource.write.table.type': 'MERGE_ON_READ',
    # capsule metadata
    'hoodie.meta.owner': 'wealth-client-domain',
    'hoodie.meta.classification': 'confidential',
    'hoodie.meta.sla_freshness_minutes': '15',
    'hoodie.meta.contract_path': 'contracts/fact_position_intraday.yaml',
    # quality integration: reject the commit if validation fails
    'hoodie.precommit.validators': 'com.meridian.quality.PositionValidator',
}
(df.write.format('hudi').options(**hudi_options).mode('append')
   .save('s3://meridian-lakehouse/curated/wealth/position_intraday'))
```

The pre-commit validator is the capsule's quality gate, enforced at the version boundary: if checks fail, the write is rejected and no bad data enters the table. The Hudi capsule is best understood not as a static table but as *a stream of validated state transitions* — each commit a checked step forward. The gotcha is operational: background table services like compaction and clustering can appear as large writes even when no new data arrives, so your freshness and volume monitoring must account for them.

## Deploying a capsule: the CI pipeline

Pattern 2 says "the definition in Git is the source of truth; the table is a deployment of it," and that sentence hides the mechanism that makes it real: a pipeline that turns a table-spec into a live table only if the change is safe. It is worth seeing, because the whole discipline rests on this pipeline existing and being un-bypassable. Here is the shape of Meridian's, deploying the `client_holdings` capsule:

```yaml
# .github/workflows/deploy-capsule.yml (abridged)
on:
  pull_request:
    paths: ["table-specs/**", "contracts/**", "tests/**"]
jobs:
  validate-and-deploy:
    steps:
      - uses: actions/checkout@v4

      # 1. Lint & validate the spec against the capsule/ODCS schema
      - run: capsule-cli validate table-specs/wealth/fact_client_holding.yaml

      # 2. Diff against the live table; classify the change (semver)
      - run: capsule-cli plan --spec table-specs/wealth/fact_client_holding.yaml
        # emits: change_type = major | minor | patch

      # 3. Contract compatibility: would this break a registered consumer?
      - run: capsule-cli check-consumers --spec ... --contract contracts/fact_client_holding.yaml
        # fails the build if a breaking change lacks consumer sign-off + notice

      # 4. Run the bound quality suite against a staging snapshot
      - run: dbt build --select fact_client_holding
      - run: great_expectations checkpoint run fact_client_holding

      # 5. Impact analysis via OpenLineage graph (what's downstream?)
      - run: capsule-cli impact --dataset wealth.fact_client_holding

      # 6. On merge to main only: apply DDL, promote snapshot, emit lineage
      - if: github.ref == 'refs/heads/main'
        run: capsule-cli apply --spec ... && capsule-cli publish
```

Read the stages in order and each is one of the guarantees the book keeps promising, made mechanical. Step 1 rejects a malformed capsule. Step 2 is where the semver classification of Chapter 5 is *computed*, not asserted — the tool diffs the spec against reality and decides whether this is major, minor, or patch. Step 3 is the guard that would have stopped the Okonkwo change: a redefinition of `em_exposure_pct` is a major change, and a major change that lacks consumer sign-off and the 90-day notice *fails the build here*, before it can reach production. Steps 4–5 gate on quality and surface the blast radius. Step 6 applies the change only from `main`, so the only path to production is through the pipeline — which is what makes "no console edits" enforceable rather than aspirational. This single file is why Pattern 2 is a rule and not a hope.

## Cross-cutting: contracts, testing, lineage

Whichever format you choose, three concerns span all of them and deserve to be designed once.

**The data contract is the outer envelope.** It is the promise the table makes to its consumers: what schema it will hold, what values are valid, how fresh it will be, and what changes are allowed without notice. The contract deploys *with* the table as one unit — modify the table definition and the contract is reviewed and updated in the same pull request, with CI enforcing that the two stay in sync. The format of the contract (YAML, JSON Schema, a structured document) matters far less than the discipline of co-location and co-versioning.

**Testing is a deployment gate, not documentation.** Quality expectations are executable. Whether you use dbt tests, Great Expectations, Soda, or custom validators, the pattern is identical: tests live beside the table definition, run in CI/CD, and block promotion on failure. Observability then extends past pass/fail to freshness (time since last successful write), volume (record counts, file sizes), distribution (null rates, value ranges), and access patterns — most of which the table format exposes as statistics you can read without scanning the data.

**Lineage connects versions to the jobs that made them.** Every format provides versioning; the work is linking each version to its producing job. OpenLineage has become the standard for this. When a Spark job writes an Iceberg snapshot, OpenLineage records the job, its inputs, and the output snapshot; stitch those events together and you have a provenance graph. Dataset identity here means the fully qualified table name *plus* the snapshot, version, or commit ID — so "where did this number come from?" traces from a query result back through snapshots to source jobs to upstream datasets. We will build this into a standards-based platform in Chapter 12; for now, note that the table format gives you the anchor and OpenLineage gives you the edges.

## Table maintenance is capsule maintenance

There is an operational dimension the capsule concept quietly depends on and that teams new to table formats routinely neglect: the background maintenance the format performs is not housekeeping *beside* the capsule — it *is* capsule maintenance, and getting it wrong quietly erodes the guarantees the capsule makes.

**Compaction** rewrites many small files into fewer large ones. It changes no data and no schema, but it produces a new snapshot and a large write, so your freshness and volume monitoring must know to expect it or it will read as an anomaly. **Snapshot expiry** and **vacuum** delete old snapshots and unreferenced files to control storage — and this is where it bites the capsule, because *snapshot history is the substrate of time travel, and time travel is how you answer "what did this data look like on the day the model trained."* Set the retention window too short and you have optimised away the very evidence Chapter 7 and Part V depend on. Meridian's rule is explicit: snapshot retention on any capsule feeding a model or a regulated report is set to *at least* the audit horizon — years, not days — even though it costs storage, because the alternative is a paused model and no receipt. **Manifest and metadata files** are the table's own record of itself; a corrupted or mispruned manifest is a corrupted capsule, so these are backed up and monitored like the data they describe.

The principle: the capsule's promises — reproducibility, time travel, auditability — are only as durable as the maintenance policy underneath them. A retention window is not an infrastructure setting; it is a *governance decision about how long your data can testify about its own past,* and it should be made by the owner with the audit horizon in view, not defaulted by whoever set up the table.

## Migrating what you already have

Meridian did not get to start clean, and neither will you. The realistic starting point is naked Parquet or CSV in object storage, loosely tracked in Glue, with schema drift happening silently and metadata living in wikis and people's heads. The path from there to capsules is incremental and does not require halting the business.

1. **Pick one dataset.** Important enough to matter, bounded enough to manage. A fact table that several teams depend on is ideal — for Meridian, client holdings was the obvious first choice, because it was the one that had already produced a visible failure.
2. **Wrap the existing files.** Delta, Iceberg, and Hudi can all import existing Parquet without rewriting it. The format's metadata layer now tracks schema and partitioning over data that has not moved.
3. **Move metadata into the table.** Take the schema knowledge that lived in Glue or nowhere and express it explicitly: column comments, ownership and classification properties, the references to contract and tests.
4. **Introduce contracts and tests.** Define the basic promise — schema, owner, freshness — and write the few critical tests: uniqueness, non-null keys, referential integrity, the division-aware distribution check. Wire them into the pipeline.
5. **Enforce and expand.** Once the first capsule works, deprecate direct access to the old file layout so consumers query the table, not the raw files. Then pick the next dataset and repeat.

During migration you will have capsules and non-capsules side by side, and that is fine — not every dataset needs the treatment at once. A workable criterion for what gets promoted first: **if a dataset is consumed by more than one team, feeds a regulated report, or would cause real pain if it broke unexpectedly, it should become a capsule.** Internal scratch tables and exploratory datasets can wait.

## A migration, as a two-week diary

The five-step migration reads cleanly, so it is worth telling how Meridian's first one — wrapping the legacy `client_holdings` extract — actually went, because the friction is the instructive part and the recommendations promised the texture.

*Week one, days 1–2.* Maya's team registered the existing Parquet as an Iceberg table in place — no data rewrite, minutes of work — and it felt anticlimactic, which is correct: the point of wrap-in-place is that nothing moves. *Days 3–4.* They moved the schema knowledge out of Glue and people's heads into the table spec, adding column comments and the ownership and classification properties. This surfaced the first surprise: two columns nobody could define, which turned out to be dead fields no consumer read — deleted, not documented. *Day 5.* Wrote the first four quality tests (uniqueness, non-null keys, the division-aware distribution check, referential integrity to `dim_client`) and immediately caught a real problem — the distribution check failed on exactly the institutional-basis rows that had caused Okonkwo, which was the moment the discipline paid for itself in front of the team.

*Week two, days 6–7.* Wired the tests and the contract into CI (the pipeline above). *Day 8* was the hard one: a downstream analytics team was reading the *raw Parquet files directly*, bypassing the table entirely, and their weekly report broke when file layout changed under compaction. This is the migration's real work — not the wrap, but the *deprecation of direct file access*. They gave the team two weeks' notice, a view that matched the old shape, and a deadline. *Days 9–10.* Published the capsule, pointed the copilot and the reports at the table, and left the raw path readable-but-deprecated with a logged warning on access, to catch stragglers.

The lesson Meridian drew, and the one to carry: the technical migration is a day; the *consumer* migration is the fortnight. Wrapping data in a table format is easy. Getting every consumer to depend on the *governed table* rather than the raw files underneath it is the actual change, and it is a change to people's habits, not to storage.

## Choosing a catalog (it matters more than you expect)

Chapter's earlier Iceberg note flagged that catalog choice matters; here is the comparison, because for an Iceberg or multi-engine shop the catalog decides which governance features the capsule can actually enforce. The table format gives you schema and snapshots; the *catalog* is what gives you tags, masking, and access control.

| Catalog | Multi-engine | Tags / classification | Column masking / ABAC | Notes |
|---------|:---:|:---:|:---:|-------|
| Hive Metastore | ✓ | limited | ✗ | Ubiquitous, minimal governance |
| AWS Glue | ✓ | ✓ (Lake Formation) | ✓ (Lake Formation) | Good on AWS; ties you to it |
| Unity Catalog | Spark-centric | ✓ | ✓ | Rich governance; Databricks-centric |
| Nessie | ✓ | via properties | ✗ | Git-like branching for data |
| Polaris | ✓ | ✓ | emerging | Open Iceberg REST catalog |
| Iceberg REST + engine | ✓ | via properties | engine-dependent | Maximum portability, least built-in governance |

The rule of thumb: **choose the catalog for the governance features you need to enforce, not the format for them** — because the format is largely interchangeable and the catalog is where masking and access control actually live. Meridian runs Iceberg with a catalog that supports tag-based masking, because `client_id` masking is a policy the platform must enforce, not a convention the capsule merely declares.

## What the discipline costs at 50 TB

Honesty about cost keeps the discipline credible. Column statistics, constraints, and pre-commit validators are not free at scale. On a 50 TB capsule, computing full column statistics on every write is expensive, so Meridian samples for distribution checks and computes exact stats only on partitions touched by the write. Pre-commit validators add latency to each commit, which matters for the intraday Hudi feed and barely registers for the daily holdings snapshot — so the *depth* of validation is tiered to the write cadence. And CI table-deployment must stay fast enough that engineers do not route around it, which means running the heavy quality suite against a *sampled* staging snapshot in the PR and the full suite on a schedule. The governing principle mirrors the hot/cold split: spend validation effort in proportion to blast radius and cadence, so the discipline stays affordable and therefore stays used.

## Choosing between the three

Teams burn surprising amounts of time on this decision, so let it be brief. You can implement capsules on any of the three formats; the patterns matter far more than the logo.

- **Lean Delta** if you are standardised on Databricks and Spark. Unity Catalog's tagging, lineage, and access control integrate directly, the ecosystem is mature, and it is the path of least resistance in a Spark-centric shop.
- **Lean Iceberg** if you run multiple query engines or value vendor-neutral, specification-first design. All engines read and write the same tables with full metadata support, which is why Meridian chose it.
- **Lean Hudi** if change data capture or incremental processing is central. The commit timeline and incremental queries are built in rather than bolted on, and the same table can take streaming writes and serve batch reads.

The common thread survives whichever you pick: define tables as code, store metadata in the table rather than only in the catalog, co-locate contracts and tests, version everything together. You can even mix formats across an organisation — the capsule abstraction sits above the table format. What matters is that each table is self-describing, governed, and trustworthy.

## The shift, restated

The mental move this chapter asks for is to stop treating `CREATE TABLE` as a one-time setup command and start treating it as **the primary place your Data Capsule lives.** The table definition in Git — schema, comments, properties, references to contracts and tests — is the source of truth. The catalog is an index. The documentation portal is a view. The governance dashboard is a report. Internalise that ordering and implementation follows almost mechanically: pick an important table, define it completely, deploy it through CI/CD, monitor it, iterate.

The payoff compounds. Each capsule is one fewer dataset that can surprise you, one fewer "I thought someone else owned that," one fewer compliance gap to explain to an auditor — and one more piece of your estate that is ready for whatever comes next. For Meridian, "whatever comes next" was not hypothetical. It was an adviser copilot in production and a credit-risk model heading for a model-risk committee, both of which would consume these tables not as dashboards-for-humans but as machine-read inputs with regulatory consequences. Batch tables sitting quietly in object storage are only half the picture. The next chapter takes the capsule discipline into the half where the stakes are highest: training data, feature stores, and data in motion.

> **Chapter summary.** The lakehouse table format is the natural home for a capsule because data and metadata already converge there and move atomically in versioned commits. Map the seven components onto the format's schema, properties, comments, snapshots, and constraints; split hot metadata (embedded) from cold metadata (referenced). Three patterns hold across Delta, Iceberg, and Hudi: the table is an atomic object, table definitions are code, and catalogs are indexes rather than sources of truth. Migrate incrementally, one high-value dataset at a time. The format choice matters less than the discipline.

> **Try this.** Take the capsule you defined at the end of Chapter 5 and deploy it as a real table on your platform's format. Then deliberately make a breaking change by hand in the console — and watch your CI fail to detect it. That gap is why Pattern 2 exists.
