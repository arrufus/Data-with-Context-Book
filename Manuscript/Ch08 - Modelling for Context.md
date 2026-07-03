# Chapter 8 — Modelling for Context

> **Where you are:** Part II — *Bind*, closing chapter. Chapters 5–7 bound context to data. This chapter addresses the structure that makes context possible to bind in the first place — and sets up the move from *Bind* to *Automate* and *Package*. Maturity note: rigorous modelling is the bridge from **Level 1 (Defined)** toward **Level 2 (Modelled)**.

---

## A definition needs something to attach to

A capsule binds the statement *"`em_exposure_pct` excludes frontier markets for retail clients"* to a field. But that binding is only as stable as the field. If `em_exposure_pct` is computed three different ways in three source-shaped tables, recombined by a fragile join, and surfaced under slightly different names in two marts, then the definition has nothing solid to attach to. You can bind a precise meaning to a column whose underlying structure is incoherent, and all you have done is document the incoherence precisely.

This is the prerequisite Part II has been quietly leaning on. Context binds to structure. Where the structure is sound — clear grain, conformed entities, explicit keys, history handled deliberately — a definition has a stable home and a capsule has something worth governing. Where the structure is a swamp of raw tables, cryptic columns, and ambiguous relationships, no amount of binding will save you, because the thing you are binding to keeps shifting underfoot. So the last move of *Bind* is, perhaps counter-intuitively, about modelling: the discipline of giving data a shape that meaning can hold onto.

For most of the last decade this was an unfashionable thing to say. "Schema-on-read" promised a world without upfront structure — dump everything raw, impose meaning later, move fast. The reality of delivering reliable data products swung the pendulum back, and the arrival of AI agents has driven it back harder still. Agents fail without rigorous semantic definitions. They cannot infer your business from cryptic column names and hope; they reason over what is explicitly modelled, and they expose the gaps at machine speed. Modelling, the supposedly ancient craft, turns out to be the thing standing between a high-value data product and a data swamp with marketing.

## The model is the user interface

Start with the point that reframes everything else: **a data model is the user interface for the data.** A data product is only as good as its consumption experience, and without a clear model, consumers wade through raw tables, cryptic names, and ambiguous relationships, reconstructing meaning every time from scratch. A well-designed dimensional or semantic model abstracts that complexity away and lets a consumer — an analyst, and now an agent — query with confidence rather than guesswork.

This used to be a statement about human ergonomics. It is now a statement about whether your AI works at all. When Meridian's adviser copilot asks for a client's emerging-market exposure, it is not browsing; it is issuing a query against a model and trusting the answer. If the model presents one conformed `client` entity with one governed definition of EM exposure, the copilot gets a trustworthy answer. If it presents six source-shaped tables and leaves the joining and the definitional reconciliation to the caller, the copilot will join wrong and reconcile wrong — confidently, and thousands of times an hour. The Okonkwo error from Chapter 5 was, underneath the missing semantic metadata, also a *modelling* failure: two divisional definitions of the same concept were allowed to coexist in the structure with nothing forcing a reconciliation. Good modelling is what makes the model a translation engine instead of a trap.

Two more consequences follow, and they are worth stating because they are the arguments you will use to fund the work.

**Quality and governance become design-time properties, not afterthoughts.** Defining primary keys, foreign keys, and constraints during design enforces integrity at the source rather than patching it downstream. Shared modelling standards are what let independent products — a "Client 360" and a "Suitability" product — interoperate instead of contradicting each other. As governance shifts to the data layer (because agents can expose sensitive data faster than any dashboard ever could), the model becomes the binding contract that enforces meaning, security, and quality *before* data leaves the platform. A well-modelled product, in the phrase that has been making the rounds, "comes with instructions" — and those instructions are the semantics.

**Rigour increases agility rather than reducing it.** This is the counter-intuitive part. When business logic lives in a stable model rather than scattered across hundreds of ad-hoc BI reports and SQL scripts, change becomes tractable: redefine "churn" or "active client" once, in the model, and the change propagates downstream. The alternative — bespoke logic re-implemented in every report — is what actually slows you down, because every change becomes an archaeology project across artefacts nobody fully inventoried. Teams that model upfront shrink their review cycles for new sources from weeks to days, because they are assembling standardised, modular products rather than hand-coding each pipeline. You can build a shed without a blueprint. You cannot build a skyscraper without one, and you certainly cannot build a safe environment for AI agents without one.

## Where the modelling actually happens: the medallion layers

Most modern lakehouse platforms — Meridian's included — organise data into the medallion architecture: Bronze, Silver, Gold. The concept is simple to state and the source of endless confusion in practice, because the real question is not "which three layers" but *"what modelling approach belongs in each?"* The answer is not one-size-fits-all, but the patterns are well established, and getting them right is what makes a capsule's context stable.

### Bronze: keep it raw

Bronze is the landing zone and the audit trail. Keep data in its original source format, adding only technical metadata — ingestion timestamps, source-system identifiers, batch IDs — using append-only patterns to preserve full lineage. The governing instinct is *preservation, not transformation*. Store in flexible, schema-evolution-friendly formats like Parquet or Delta, and resist the temptation to apply business logic here. That restraint is what keeps Bronze replayable, explainable, and useful for forensics — and forensics, as Chapter 7's audit story showed, is not hypothetical for a regulated firm.

In capsule terms, Bronze carries mostly structural and provenance metadata: what arrived, from where, when. The semantic and policy richness comes later. Bronze's job is to be an honest, immutable record of what the source actually sent, so that when the institutional-versus-retail definition question arises, you can replay from the raw truth rather than from a transformed copy that already baked in someone's assumption.

### Silver: the integration and truth layer

Silver is where integration happens, and it is the layer where the modelling choice matters most — because Silver is where you build the single, reusable version of the truth that downstream products and capsules will attach their meaning to. Two approaches dominate.

For most domains, a **normalised, 3NF-style approach** is the right default: cleanse, deduplicate, and standardise; establish common business keys and conformed entities (Client, Portfolio, Holding); implement slowly-changing-dimension logic where history matters; apply business rules and quality checks. It earns its place by being easy to explain, easy to test, and easy to reuse across multiple products. Clean, reusable entities can feed many Gold models without remodelling each time. This is why so many lakehouse teams quietly run "3NF-on-Delta" in Silver — it is the least surprising model, and least-surprising is a virtue in a layer everything else depends on.

For **complex, multi-source, heavily regulated** domains, **Data Vault 2.0** is worth its additional cost: hubs for business keys, links for relationships, satellites for history. It decouples identity from relationships from context-and-history, which makes it remarkably tolerant of source churn and excellent for traceability — and it maps cleanly onto active metadata and lineage work because the relationships are *explicit*. Meridian's client domain is a good candidate: a single client identity is assembled from a CRM, two custodians, a billing system, and an MDM golden record; auditability is non-negotiable; and the firm knows from experience that source systems get replaced. A quick decision rule: **few sources, need clarity → 3NF; many sources, need audit → Data Vault; unsure → model in 3NF and add Data-Vault-style satellites for the history-heavy entities.**

Purism is not required. A workable hybrid is Data-Vault-style ingestion (hubs, links, satellites) for the ugly multi-source domains, with 3NF-style integration views *over* the vault that present clean entities to downstream teams. That way you get the vault's auditability without forcing every consumer — or every AI agent — to learn Data Vault to ask a question.

Whichever you choose, Silver is where the hard reconciliation decisions get made and *recorded*, and several of them are exactly the decisions whose absence caused Chapter 5's failure:

- **History management.** 3NF stores current state; analytics and audit need history. Implement SCD (usually Type 2) with `effective_from`, `effective_to`, and `is_current`, but be selective — full history on everything balloons storage. Prioritise client, product, pricing, and reference data.
- **Keys.** Keep both surrogate keys (for internal joins) and natural business keys (for lineage and reconciliation). Always both: the surrogate for flexibility when source keys change, the natural key for traceability.
- **One truth from many sources.** When the same entity arrives from different sources with conflicting values, define a system-of-record hierarchy, carry a `source_system_id`, and document survivorship rules explicitly. *Which source wins when the CRM and the custodian disagree on a client's domicile?* Decide it with the domain owner now, write it down, and — in a regulated firm — make it part of your data controls. This is the modelling-layer answer to the Okonkwo problem: do not let two definitions coexist unreconciled; choose, record the choice, and bind it.
- **Pragmatic normalisation.** Aim for 3NF but allow sensible violations. If common queries join more than four or five tables, you have over-normalised; keep frequently-used attributes denormalised and do not create separate tables for tiny lookups. Balance joins against storage by *actual query patterns*, not theoretical purity.
- **CDC at scale.** Detect changes efficiently with hash/checksum columns, apply them with Delta or Iceberg `MERGE`, and partition by ingestion or effective date so merges stay fast. This is what keeps Silver current without full reloads.
- **Quality without blocking.** Lakehouses do not enforce foreign keys the way databases do, so enforce referential integrity in transformation code — but grade violations as critical versus warning and let processing continue where appropriate. Not every violation should stop the pipeline; production needs tolerance.

The Silver layer should be stable and reusable — the single source of truth and the contract for downstream teams. Get Silver right and everything above it falls into place. Rush it or over-engineer it and you will refactor for months.

### Gold: optimise for consumption

By the time data reaches Gold it should already be clean, conformed, and history-aware, so the Gold modelling decision is less about correctness and more about *how consumers consume*. Gold tends to split into two families, and the modern twist is that one of those consumers is now a machine.

Use **dimensional / star-schema models** (Kimball) when the consumers are BI analysts, reporting, finance, and operations — when you need clear business grain ("Holding," "Transaction," "Suitability Assessment"), conformed dimensions across reports, and low-friction self-service in tools like Power BI, Tableau, or Looker. Dimensional models expose the business language — facts for events and measures, dimensions for who/what/where/when — which remains the lowest-friction way for non-engineers to explore data, and they map cleanly onto semantic layers and catalog tools. This is why Kimball is still alive decades on.

Use **denormalised wide tables** (the "one big table" pattern) when the consumers are ML workloads, agents, notebooks, or data-product APIs that want very few joins and fast, predictable responses — a `client_360` or a `suitability_serving` table that pre-joins what a caller would otherwise have to assemble. Wide tables optimise for *cost of use* rather than cost of storage: you pre-join so the consumer does not have to, which is exactly what a feature store, a reverse-ETL push, or an AI agent prefers.

The decision rule is short: **humans exploring → star schema; machines and pipelines consuming → wide table; mixed audience → star as canonical, wide as a materialised convenience.** A pragmatic production split keeps a `Gold.Canonical` (dimensional, semantic-layer-friendly, used by BI), a `Gold.Serving` (denormalised, task-specific, used by ML, apps, and the copilot), and a short-lived `Gold.Sandbox` for experiments — so the source of truth stays clean while teams still get the "just give me one table" experience. Gold should change faster than Silver, because business questions change faster than source systems do.

## The suitability domain, modelled in code

The chapter has taught the *choices*; a reader building this needs to see the *models*. Here is Meridian's suitability domain across the layers, in enough code to lift.

**Gold, dimensional (the canonical star).** The consumption model the copilot and the reports read is a star with a clear grain:

```sql
-- grain: one row per suitability assessment, per client, per assessment date
CREATE TABLE gold.fact_suitability_assessment (
    assessment_sk     BIGINT,         -- surrogate
    client_sk         BIGINT,         -- FK -> dim_client
    portfolio_sk      BIGINT,         -- FK -> dim_portfolio
    risk_profile_sk   BIGINT,         -- FK -> dim_risk_profile
    assessment_date   DATE   NOT NULL,
    em_exposure_pct   DECIMAL(5,2),   -- division-normalised at build
    breach_flag       BOOLEAN,
    breach_margin_pct DECIMAL(5,2)
) USING iceberg PARTITIONED BY (months(assessment_date));

-- dim_client is SCD2: one row per client per validity period
CREATE TABLE gold.dim_client (
    client_sk       BIGINT,           -- surrogate (join key)
    client_id       STRING,           -- natural/business key (lineage)
    division        STRING,           -- retail | institutional
    domicile        STRING,
    effective_from  TIMESTAMP,
    effective_to    TIMESTAMP,        -- null-high for current
    is_current      BOOLEAN
) USING iceberg;
```

Two disciplines from earlier in the chapter are visible in the DDL: every entity keeps *both* a surrogate (`client_sk`, for joins) and the natural key (`client_id`, for lineage and reconciliation); and `dim_client` is Type-2, carrying validity periods so a point-in-time join is possible.

**Silver, Data Vault (multi-source integration).** Because client identity is assembled from a CRM, two custodians, and MDM, the Silver layer uses Data Vault, and a fragment makes the "hybrid" concrete:

```sql
CREATE TABLE silver.hub_client (
    client_hk    BINARY,              -- hash of business key
    client_id    STRING,              -- business key
    load_ts      TIMESTAMP, source_system STRING);

CREATE TABLE silver.link_client_portfolio (
    link_hk BINARY, client_hk BINARY, portfolio_hk BINARY,
    load_ts TIMESTAMP, source_system STRING);

CREATE TABLE silver.sat_client_risk (      -- context + history
    client_hk BINARY, risk_score INT, risk_review_date DATE,
    load_ts TIMESTAMP, hash_diff BINARY,   -- change detection
    source_system STRING);

-- 3NF integration VIEW over the vault: clean entities for downstream teams
CREATE VIEW silver.client AS
SELECT h.client_id, s.risk_score, s.risk_review_date, s.source_system
FROM silver.hub_client h
JOIN silver.sat_client_risk s ON h.client_hk = s.client_hk
WHERE s.load_ts = (SELECT max(load_ts) FROM silver.sat_client_risk x
                   WHERE x.client_hk = h.client_hk);   -- current context
```

The vault absorbs source churn and audit history; the 3NF view over it means no downstream consumer — or agent — has to learn Data Vault to ask a question. That is the hybrid the chapter recommended, in twenty lines.

**The SCD Type-2 MERGE.** History management is where 3NF-in-Silver actually costs effort, so here is the idempotent pattern that maintains `dim_client`:

```sql
MERGE INTO gold.dim_client t
USING staged_client s ON t.client_id = s.client_id AND t.is_current
WHEN MATCHED AND t.hash_diff <> s.hash_diff THEN      -- attributes changed
  UPDATE SET t.effective_to = current_timestamp(), t.is_current = false
WHEN NOT MATCHED THEN
  INSERT (client_sk, client_id, division, domicile,
          effective_from, effective_to, is_current)
  VALUES (s.client_sk, s.client_id, s.division, s.domicile,
          current_timestamp(), null, true);
-- second pass inserts the new current row for changed clients
```

This closes the old row and opens a new one on change, which is exactly what lets the copilot do the *point-in-time* join Chapter 7's temporal-correctness discussion demanded — reading a client's division *as it was on the assessment date*, not as it is today.

**Survivorship, made real.** The chapter said "decide which source wins, attribute by attribute, and write it down." Here is the artefact — Meridian's survivorship table for the client entity, signed by Maya:

| Attribute | System of record | Rule when sources conflict |
|-----------|------------------|----------------------------|
| `client_id` | MDM | MDM golden record is authoritative |
| `domicile` | Custodian | Custodian of record; CRM ignored |
| `email` | CRM | Most-recently-updated CRM value |
| `division` | Onboarding | Set at onboarding; change requires approval |
| `risk_score` | Risk system | Latest completed assessment only |

That table is the "Try this" of the original chapter, completed: the CRM-vs-custodian domicile conflict that could have produced another Okonkwo is *resolved in advance*, recorded, owned, and — because it lives in the capsule's semantics — enforceable rather than remembered.

## Modelling for the agent, specifically

Everything above is sound modelling practice that predates the current AI moment. What the AI moment changes is the *consumer's tolerance for ambiguity*, and that has two concrete implications for how Meridian models.

First, the **serving layer becomes a first-class product, not an afterthought.** When the adviser copilot consumes `suitability_serving`, that wide table is the interface the agent reasons over. It needs to carry — or bind, via its capsule — the governed definitions, the division-aware normalisation, and the prohibited-use flags, because the agent will not go and read a Confluence page to disambiguate. The model is the only context the agent has. A wide serving table without bound semantics is precisely the trap that produces confident wrong answers.

Second, **the semantic layer is where the model stops being a set of tables and starts being a set of business concepts** — and it is the natural seam between Part II and Part III. A semantic layer (dbt's MetricFlow, LookML, or a dedicated tool) sits above Gold and maps physical tables to business concepts: "active client," "emerging-market exposure," "suitability breach," each with one governed definition and one calculation. Modelled well, it serves humans and agents from the same trusted concepts, so an analyst's "EM exposure" and the copilot's "EM exposure" are guaranteed to be the same thing. This is the foreshadow of the **context data product** that Part III builds: a base data product, plus an ontology of entities and relationships, plus a semantic layer of governed concepts, plus the knowledge graph and usage intelligence that make it consumable by reasoning systems. Modelling is where that stack gets its foundation. You cannot package context (Part III) that you have not first structured (Part II).

## Designing the wide serving table

The serving table the agent reads deserves its own design rules, because a wide table done carelessly is how a governed star quietly forks into an ungoverned duplicate. Meridian's rules for `Gold.Serving`:

- **Column budget.** A serving table pre-joins what a caller would assemble, but not everything — a `suitability_serving` table with four hundred columns is a maintenance liability nobody reads. Cap it at the columns the consumer actually queries, and add on evidence of use, not on speculation.
- **Derive, don't redefine.** Every metric in the serving table is *derived from the semantic layer's* governed definition, never re-implemented in the serving SQL. This is the guard against the metric-level drift the chapter warns about: `em_exposure` in `Gold.Serving` is the *same* calculation as everywhere else, materialised, not a second version.
- **Update cadence and freshness contract.** The serving table states its own freshness SLA, and it is refreshed *from* the canonical models, so `Gold.Serving` can never be fresher-but-wrong than `Gold.Canonical`.
- **Prevent canonical/serving drift.** A scheduled check reconciles a sample of serving-table values against the canonical star; a mismatch is an incident, not a rounding tolerance. The serving table is a *materialised convenience over the source of truth*, and the reconciliation is what keeps "convenience" from becoming "third conflicting definition."

## A house style for naming

Naming is not cosmetic — it is the first layer of semantics an agent or analyst meets, and inconsistent naming scales inconsistency as fast as AI can read it. Meridian's half-page standard, offered as a starting point (and feeding Appendix B):

- **Layers by prefix/schema:** `bronze.`, `silver.`, `gold.` (canonical), `gold_serving.` — the layer is legible from the name.
- **Entity tables** singular business nouns: `client`, `portfolio`, `holding`. **Facts** `fact_<event>`; **dimensions** `dim_<entity>`; **vault** `hub_/link_/sat_`.
- **Keys:** `<entity>_sk` for surrogates, `<entity>_id` for natural keys — always both, never conflated.
- **Booleans** prefixed `is_`/`has_`; **dates** suffixed `_date`, **timestamps** `_ts` (and stated timezone in the description — UTC by default).
- **Division-dependent or otherwise ambiguous fields** carry a semantic reference in the capsule, and the ambiguity is named in the column description, never left to the reader.

The rule behind the rules: a name should tell a consumer what layer it is in, what grain it has, and where its meaning is defined — so that the model is legible before anyone opens the documentation, because the agent will not open the documentation.

## Three rules that survive contact with production

After enough platforms, three principles earn the status of non-negotiable.

**Start simple.** Begin with ten to fifteen core entities, model those well, then expand. Do not try to model the enterprise on day one — the 90-day discipline of Part III exists precisely to resist that ambition. For Meridian's client domain, the starting set is small and obvious: Client, Portfolio, Holding, Asset Class, Risk Profile, Suitability Assessment. Five or six entities, modelled cleanly, are worth more than fifty modelled vaguely.

**Build reusable patterns.** Template your SCD types and merge logic so the twentieth source onboards like the second. Reusable patterns are what convert modelling from heroics into engineering.

**Monitor performance early.** Profile Silver-to-Gold transformations before you have terabytes, because performance problems are far harder to fix after the fact — and Silver-to-Gold is usually where the bottleneck hides, especially once analysts start asking for "just one more attribute."

The thread through all three is judgement. Perfection can be the enemy of good, but *good-enough-without-thought* is the enemy of maintainability — and in a regulated firm building for AI consumers, maintainability is the property that decides whether you can still answer the auditor's question in two years.

## Closing Part II

Part II has made one move: *Bind*. Chapter 5 named the unit — the Data Capsule, data and metadata as one co-versioned, deployable whole. Chapter 6 gave it a home in the lakehouse table and the patterns that keep it honest. Chapter 7 carried it into the high-stakes world of training data, features, and streams, and showed the payoff at its sharpest: a model that survives its audit because its data can produce evidence about itself. This chapter supplied the prerequisite underneath all three — the structure that gives meaning something stable to bind to, modelled deliberately for a world where the consumer may be an agent.

Meridian now has its first capsule: a well-modelled client-holdings product, its context bound, its meaning stable, its lineage anchored. That is one dataset at Level 1, reaching toward Level 2. But a single hand-bound capsule is a craft object, and craft does not scale. The moment Meridian has fifty capsules, the binding it did by hand has to become something the platform does automatically — versioned, tested, event-driven, enforced. Keeping context correct by hand across a growing estate is exactly the manual-governance trap that fails at scale. That is the second move of the discipline, and the subject of Part III: **Automate**. We turn the capsule from something you maintain into something the platform maintains for you — metadata as code, and metadata that acts.

> **Chapter summary.** Context binds to structure, so modelling is the prerequisite for everything in Part II. The model is the user interface — now for agents as much as analysts — and rigour increases agility rather than reducing it. Across the medallion layers: Bronze preserves raw truth; Silver builds the integrated single source of truth (3NF by default, Data Vault for multi-source regulated domains, hybrids welcome) and is where the hard reconciliation decisions get recorded; Gold optimises for consumption (star schemas for humans, wide tables for machines). Model the serving and semantic layers as first-class products, because they are the only context an AI agent gets. Start with ten to fifteen entities, build reusable patterns, profile early.

> **Try this.** For your first capsule's domain, write down the system-of-record hierarchy and survivorship rules for its central entity — which source wins, attribute by attribute, when they conflict. If you cannot answer that in an afternoon, you have found why the domain's definitions keep drifting. Recording that decision is the modelling work that makes the capsule's semantics stable.
