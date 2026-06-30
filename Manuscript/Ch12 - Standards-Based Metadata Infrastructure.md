# Chapter 12 — Standards-Based Metadata Infrastructure

> **Where you are:** Part III — *Automate*. Chapter 11 made metadata act. But active metadata across a sprawling estate only works if every tool can speak a common language and share a common graph. This chapter builds that substrate — the metadata integrator — using open standards and a knowledge graph. It is the spine beneath the nervous system.

---

## When every tool speaks its own language

Active metadata made a promise in the previous chapter: detect a schema change anywhere, trace its blast radius everywhere, feed an agent live context before it answers. That promise has a hidden prerequisite. It only holds if the lineage, the definitions, and the quality signals from *every* tool in the estate can be joined into one coherent picture. And in most enterprises they cannot, because every tool speaks its own dialect.

Meridian's estate is the ordinary kind of sprawling. Orchestrators — Airflow for batch, something else for the streams. Transformation engines — dbt, Spark, Flink. Storage — the Iceberg lakehouse, a Snowflake footprint, the settlement mainframe nobody migrates. BI — Tableau in one division, Power BI in another. Each of these stores metadata differently, and none of them speaks the same language as the others. So Meridian lives through the scenarios every data team knows: a dashboard breaks and it takes the better part of two days and forty-seven Slack messages to find which upstream pipeline changed; the firm spends a fortune a year on catalogue tools and analysts still cannot find the *real* client table; a SOC 2 auditor asks for lineage documentation and the team spends weeks building spreadsheets and interviewing people.

When metadata lives in disconnected silos, you get duplicated catalogues that disagree about what is authoritative, lineage that dies at every tool boundary, governance held together by spreadsheets, slow incident response, and risk from changes nobody saw coming. For a mid-sized data organisation this fragmentation quietly costs seven figures a year in wasted engineering time, compliance overhead, and decisions made on data nobody fully trusted. The active-metadata behaviours of Chapter 11 are impossible on this foundation, because there is no shared picture for them to act on. Before metadata can act, it has to *connect*. This chapter is about building the thing that connects it.

## What you are actually building

The architecture has a simple core: instead of every tool maintaining its own island of metadata, they all report to a single platform that captures what happened, validates it, connects it, and answers questions about it. For an architect, it is a metadata router with a semantic brain and graph storage. For everyone else, it is a central hub where all your data tools report what they are doing, and anyone — a person or an agent — can ask "where did this come from?" or "what depends on this?" and get a straight answer.

The substrate does four jobs, and the layering follows them:

1. **Capture what happened.** Tools emit standardised lineage and quality events as they run.
2. **Translate and enrich.** Those events are normalised into a shared semantic model and validated against governance rules.
3. **Persist and connect.** The normalised metadata lands in a knowledge graph, where relationships become first-class and traversable.
4. **Serve and act.** Search, lineage visualisation, impact analysis, and trust scores are exposed via APIs — the same APIs the active-metadata layer and the AI agents consume.

The reason this works where catalogue-buying did not is the combination at its heart: **open standards plus a graph.** Standards make the metadata portable and federable rather than locked to a vendor; the graph makes the relationships — which are the whole point of metadata — queryable as relationships rather than reconstructed by hand. Take the standards first.

## The standards that make metadata speak one language

Four open standards, layered, give the platform a common tongue.

**OpenLineage for runtime lineage.** We met it in Chapter 10 as the universal lineage protocol; here it is the capture layer. Airflow, Spark, dbt, Flink, Kafka, and forty-plus other systems can emit OpenLineage events natively, so the lineage that used to die at each tool boundary now flows into one place in one format. This is the single most important integration decision, because lineage is the metadata that crosses the most boundaries and suffers the most from fragmentation.

**W3C semantic standards for meaning.** Captured events are normalised onto established vocabularies: **DCAT** (the Data Catalog Vocabulary) for describing datasets, **PROV-O** for describing provenance — what activity used what and generated what — and **SKOS** for controlled vocabularies and business glossaries. These are not bespoke schemas; they are W3C standards with formal semantics, which matters for two reasons. First, they are precise enough to reason over. Second, auditors recognise them, which turns a compliance conversation from "trust our internal model" into "here is a W3C-standard provenance record."

**SHACL for validation.** The Shapes Constraint Language lets you declare rules that metadata must satisfy — and reject writes that violate them at the API boundary. Meridian can declare that every dataset must have an owner, full stop:

```turtle
:DatasetShape a sh:NodeShape ;
  sh:targetClass dcat:Dataset ;
  sh:property [ sh:path dcat:contactPoint ;
                sh:minCount 1 ;
                sh:message "Every dataset must have an owner." ] .
```

A pipeline that tries to register an ownerless dataset is refused, with a helpful error, before the orphan exists. Compliance stops being a spreadsheet the governance team maintains and becomes a *property of the platform* — a dataset cannot enter production out of compliance, because the gate is in the write path. This is the policy-as-code principle of Chapter 9, enforced at the metadata layer itself.

**A knowledge graph for storage.** The normalised metadata persists as RDF triples in a graph store. This is the choice that turns hard questions into easy queries. "Show me everything three hops upstream of this report" is a brutal join across silos in a relational world; in the graph it is a trivial traversal. When Meridian's Q4 client-outflows dashboard misbehaves, finding every dataset and job that fed it is one SPARQL query against the provenance graph, not a two-day investigation:

```sparql
SELECT ?upstream ?job WHERE {
  :report_q4_outflows prov:wasDerivedFrom+ ?upstream .
  ?job prov:generated ?upstream .
}
```

The `+` is doing heroic work there: it traverses the lineage to *arbitrary depth*, which is exactly the query that was impossible when lineage died at each tool boundary.

## The capabilities this unlocks

With capture, semantics, validation, and graph storage in place, a set of operational capabilities falls out — and each one is a direct answer to a fragmentation pain from the top of the chapter.

**Register and discover.** Pipelines declare datasets — owner, domain, schema, SLA — through a simple API, and an analyst finds the authoritative client table in thirty seconds instead of filing a ticket and waiting two days.

**Capture lineage automatically.** As pipelines run, they emit events that the platform turns into provenance triples, so "where did this come from?" is always answerable and always current — the antidote to the forty-seven-Slack-message incident.

**Trace impact before changes.** Before deploying a schema change, an impact query returns the downstream jobs that will break, the datasets affected, the dashboards at risk, and the owners to notify. Meridian runs this on `fact_client_holding` before any change ships, so the 7 a.m. zeros become a pre-deployment warning rather than a post-deployment fire.

**Enforce governance with SHACL.** Rules are enforced at write time, so every dataset meets its governance requirements before it enters production, by construction.

**Query trust metrics.** The platform tracks freshness, quality, and ownership and exposes a trust score, so consumers — human and agent — know what to rely on before they build on it. This is the data behind Chapter 11's dynamic trust scoring.

## Two design decisions worth understanding

Most of this architecture is assembly, but two choices are load-bearing enough to explain, because they are what make the platform answer the questions that matter to a regulated firm.

**Bitemporal versioning.** The platform stores two timelines for every fact: *valid time* (when was this true in the real world?) and *transaction time* (when did we record it?). This sounds academic until an auditor asks the question that ends careers: *"Who owned this dataset when the incident occurred last quarter, and what did its quality look like on the day the model trained?"* A system that stores only the current state cannot answer; it knows who owns the dataset *now*. A bitemporal store reconstructs the past as it was understood at the time — the owner of record on that Tuesday in March, the quality score as it then stood. Part V will show this exact question deciding whether a model survives its audit; the bitemporal store is what lets the answer be "here it is" rather than "I'll come back to you."

**Canonical identity.** Different tools label the same dataset differently — the custodian's `positions`, the lakehouse's `fact_position`, the BI tool's `Positions (Prod)`. Left alone, this ID fragmentation silently shreds metadata reliability: the graph thinks one dataset is three, and lineage never connects. The platform enforces a canonical URN format at the API boundary — `urn:dataset:wealth:client_holding` — rejects non-canonical IDs, and keeps a crosswalk for legacy identifiers. Unglamorous, and the difference between a graph that connects and a graph that lies.

## Honest trade-offs

No architecture is free, and three trade-offs deserve naming so the platform is adopted with eyes open.

**Performance at scale.** Deep graph traversals — five or more hops — can slow to multiple seconds. The mitigations are standard: precompute transitive closures for common paths, cap query depth, cache popular queries with a short TTL, and project to a property graph for interactive UI work. Real, manageable, but to be designed for rather than discovered.

**Standards maturity.** Not every tool emits OpenLineage, and the W3C specs continue to evolve. Bridge the gaps with adapters for legacy systems, adopt new spec features only after two stable implementations exist, and keep a compatibility matrix. The standards are mature enough to build on; they are not uniform enough to assume.

**Adoption.** The deepest risk is technical success without organisational uptake — a beautifully connected graph nobody queries. The mitigation is the same one that recurs through this entire book: launch with one visible, high-pain use case ("resolve incidents faster in the wealth division"), bake metadata capture into CI/CD so it is default rather than optional, and let demonstrated utility drive the rollout. Adoption follows value; it cannot be mandated ahead of it.

## From cataloguing to infrastructure

The phrase that captures what this chapter builds is the one to end on: the goal is to cross from *data cataloguing* to *metadata as infrastructure*. A catalogue describes your data. This platform holds your organisation accountable to it — answering, on demand and with evidence, what your lineage looked like last March, who changed a schema and when, and which datasets violate your governance rules. That is a different category of thing from a searchable inventory. It is runtime infrastructure that pipelines, agents, and products declare and consume, the way they declare and consume compute and storage.

But — and this is the note that closes every chapter of Part III, because it is the truest thing in it — technology gives the structure; organisational commitment gives it life. DCAT and SHACL do not enforce themselves, and lineage does not stay accurate without engineers who care about precision. The teams that succeed invest as much in education as in infrastructure, because a platform that nobody understands is a platform that nobody feeds. Meridian now has the substrate: a connected, standards-based, queryable, bitemporal metadata graph that lets active metadata act across the whole estate rather than within one tool. What it does not yet have is a disciplined way to *roll this out* — to take the capsule, the code, the active layer, and the integrator and adopt them across a real organisation without trying to boil the ocean, and without falling into the trap of letting the shiny new platform become yet another competing source of truth. That discipline — capsules as the source, platforms as consumers, adopted phase by phase — is the final chapter of Part III, and it is where the whole *Automate* move becomes an operating reality rather than an architecture diagram.

> **Chapter summary.** Active metadata needs a shared substrate, because in most estates every tool speaks its own dialect and metadata dies at tool boundaries — a seven-figure annual cost in lost time and compliance overhead. The fix is a **standards-based metadata integrator**: capture with **OpenLineage**, give meaning with **W3C DCAT/PROV-O/SKOS**, validate with **SHACL** at write time, and store in a **knowledge graph** where lineage becomes a one-query traversal. It unlocks register/discover, automatic lineage, pre-deployment impact analysis, enforced governance, and trust scoring. Two design decisions carry the weight: **bitemporal versioning** (answering "who owned this and how good was it *then*") and **canonical identity** (so the graph connects rather than fragments). Trade-offs in performance, standards maturity, and adoption are manageable. The move is from cataloguing to *metadata as infrastructure* — but it lives only with organisational commitment.

> **Try this.** Take your last cross-tool incident and count the boundaries the investigation had to cross — orchestrator to warehouse to BI tool, each with its own metadata. Every boundary was a place lineage died. The number of boundaries is the number of integrations a standards-based graph would have collapsed into a single query.
