# Chapter 15 — From Data Products to Context Data Products

> **Where you are:** Part IV — *Package*, the third move of the discipline. Parts II and III bound context to data and kept it active. This part assembles that context into a *product an AI agent can reason over*. Maturity target: **Level 2 → Level 3** — a domain with a modelled ontology, a populated knowledge graph, and a live AI consumer.

---

## A governed table an agent still cannot use

By the end of Part III, Meridian's client domain is in genuinely good shape. The `client_holdings` capsule is bound, versioned, and governed. Active metadata stops broken pipelines, scores trust in real time, and hands the adviser copilot certified, fresh context before it answers. On paper, this is a well-run data product, and for a human analyst it is more than enough — an analyst who queries it knows how to join, knows which caveats apply, and carries the domain in their head.

Then an adviser asks the copilot a real question. *"The Adeyemi trust is a retail account with a risk profile of 3. Is its 22% allocation to emerging-market debt suitable, and does that figure include the frontier-market exposure?"* And the well-governed table, for all its bindings and active metadata, cannot answer well — because the question is not really about the *data*. It is about *meaning*. What counts as emerging-market debt for a retail account? Does "EM" include frontier here or not? What is the relationship between a risk profile of 3 and an acceptable allocation? Is this figure current as of a valuation the copilot can trust? The answer lives in the *relationships between concepts* — Client to Risk Profile to Holding to Asset Class — and in the *business rules* that govern how those concepts are interpreted for this division. None of that is in the table. The table has values. The meaning that turns values into a defensible suitability judgement is somewhere else, or nowhere.

Be precise about what Part III already gave the copilot, because it was real and it matters. Chapter 12's context API serves facts *about a field* — this field is division-dependent, this feed is fresh, this use is permitted. Field-level context prevents field-level mistakes; it is exactly why the copilot no longer misreads `em_exposure_pct` in isolation, and it is not what is failing here. The Adeyemi question is not about a field. It requires walking a chain of concepts — this client, her division, her risk profile, the band that profile permits, the holdings and their classifications under the division's rule — and no amount of per-field context supplies the chain. What is missing is not facts but *structure*: the relationships and rules that connect facts into a judgement. Field-level context was the achievement of Part III; concept-level structure is the work of Part IV. That is why this is a new move, not a re-solving of a solved problem.

This is the gap Part IV closes. A data product delivers trustworthy *data*. An AI agent reasoning about suitability needs trustworthy *meaning* — the entities, the relationships, the rules, the confidence, and the usage constraints, assembled into a form it can query and reason over. The thing that assembles them is a **context data product**, and building Meridian's is the work of this part. This chapter defines what a context data product is and what it is made of; the next shows how to build one in ninety days; the one after that goes deep on the craft of the meaning layers. We start with the distinction that motivates all three.

## What a data product gives you, and where it stops

Recall the data-product idea from Chapter 2, because the context data product is built directly on top of it. A data product is not a dataset with a README; it is a contractual commitment — an explicit versioned interface, a named owner, and an SLA-backed guarantee. That was the correction to lake thinking: stop hoarding inventory, start serving products with purpose and promise. Everything in Parts II and III made that real and enforceable: the capsule bound the context, metadata-as-code versioned it, active metadata kept it honest.

For a human consumer, a good data product is sufficient, because the human supplies the missing meaning themselves. They know that "EM exposure" means something different for retail and institutional accounts; they know a risk profile of 3 is conservative; they know to check the valuation date. The data product hands them clean, trustworthy data and they do the reasoning. This was fine for thirty years, for the reason Chapter 1 laboured: the consumer carried the context.

The AI consumer does not. It reads the interface at face value and reasons over exactly what is there — no more. Hand it a governed table and it will confidently compute a suitability answer using whatever definition of "EM" it can infer from a column name, which is precisely how the Okonkwo error happened in Chapter 5 and how the Adeyemi question goes wrong now. The data product's guarantee covers the *data* — schema, freshness, quality, ownership. It does not cover the *meaning*, because meaning was always assumed to live in the consumer's head. For an AI consumer, that assumption is the failure. A data product is necessary and no longer sufficient.

## What a context data product adds

A **context data product** is a data product with its meaning packaged alongside its data, in a machine-readable, queryable form — so that a consumer that carries no context of its own can still reason correctly. If the data product answers *"here is trustworthy data,"* the context data product answers *"here is trustworthy data, plus the meaning, relationships, confidence, and usage rules you need to reason about it correctly."*

Concretely, it packages four kinds of context that a bare data product leaves implicit:

- **Semantic context** — what the entities and metrics *mean*, the definitions and calculation logic, the controlled vocabularies, the business rules. What "EM exposure" counts, under which divisional definition; what a risk profile of 3 permits.
- **Temporal context** — as of when each thing is true, so an agent knows whether a valuation, a risk assessment, or a definition is current or historical.
- **Usage context** — the documented query patterns, the known limitations, the prohibited uses. How the product is meant to be consumed, and how it is not.
- **Confidence context** — how much to trust each element: freshness, completeness, known quality issues, an explicit signal of reliability rather than an implicit assumption of correctness.

These four kinds live across the five layers described next: semantic context in the ontology and semantic layer, usage context in the usage-intelligence layer, confidence context in the knowledge graph's enrichment, and temporal context split between each product's `as_of` and the bitemporal store of Chapter 13. Notice that they are not new inventions; they are the capsule's semantic, policy, and provenance components (Part II) plus active metadata's trust signals (Part III), now *assembled and exposed* as a coherent product surface an agent can call. The context data product is not a replacement for anything the book has built. It is the *packaging* of it — which is exactly why "Package" is the third move and not the first. You cannot package context you have not bound and kept active.

Put it one way: a context data product makes *guided* access the default. Traditional consumption assumes a skilled human who understands the joins, the filters, and the caveats and applies judgement. AI consumption requires the product to supply that guidance itself — the semantic mappings, the governed endpoints, the grain and aggregation rules, the prohibited uses — because the agent will not, and cannot, supply it. AI needs guided access, not open access. The context data product is how you guide it.

## The anatomy: five layers

A complete context data product is built from five layers, stacked from the data upward. This anatomy is the reference model for the rest of Part IV, so walk through each layer as it looks for Meridian's suitability domain.

<!-- FIG 15.1: five-layer context data product anatomy — (1) base data product → (2) ontology → (3) semantic layer → (4) knowledge graph → (5) usage intelligence; the stack is exposed through a semantic API, a catalogue entry, and a context-extended contract. This is the book's central diagram. -->

**Layer 1 — The base data product.** The existing governed dataset: `client_holdings` and its neighbours, the capsule-bound, quality-gated, actively-governed tables from Parts II and III. This is the foundation, and it must already be solid; a context data product built on ungoverned data is a semantic layer over a swamp. Meridian has this.

**Layer 2 — The ontology layer.** The formal model of the domain: what entities exist and how they relate. For suitability: Client, Portfolio, Holding, Asset Class, Risk Profile — with the relationships between them (a Client holds a Portfolio, a Portfolio contains Holdings, a Holding has an Asset Class) expressed with explicit cardinality and direction, and the *business rules* that govern interpretation (risk tolerance is scored 1–10 and reassessed annually; "EM" includes frontier markets for institutional accounts and excludes them for retail). This layer is where the Adeyemi question gets its footing — the ambiguity about frontier markets is *resolved* here, as a rule, not left to inference.

**Layer 3 — The semantic layer.** The business-friendly mappings from physical tables to business concepts, with calculation logic and domain terminology. Where the ontology says *what* "EM exposure" means, the semantic layer says *how it is computed* from the physical columns, and guarantees that every consumer — the copilot, an analyst in Tableau, a regulatory report — gets the *same* governed definition. This is the modelling-as-UI point from Chapter 8, now formalised as a product layer: one definition of "suitability breach," served identically to humans and agents.

**Layer 4 — The knowledge graph.** The queryable connections: the entities from the ontology, instantiated and linked, enriched with lineage (where each element came from) and confidence metadata (how fresh, how complete, how trustworthy). This is what lets an agent *traverse* meaning — from the Adeyemi trust to its risk profile to its holdings to their asset classes to the applicable EM rule — in a single reasoning step rather than a pile of joins. It builds directly on the standards-based metadata graph of Chapter 13.

**Layer 5 — Usage intelligence.** The documented query patterns, the known limitations, the successful use cases. The accumulated knowledge of *how this product is actually used well* — which questions it answers reliably, which it does not, where the edge cases lie. This is the layer that turns a static product into a learning one, and it is what makes the second and third consumers faster to onboard than the first.

Five layers, from governed data at the bottom to accumulated usage wisdom at the top. The lower layers Meridian largely has; the upper three are what Part IV builds.

### The five layers as one descriptor

The blueprint flagged this explicitly: the five layers need a spec, not just a diagram. Here is the machine-readable product descriptor that ties them together for Meridian's suitability context product — a single document a catalogue and an agent can both read:

```yaml
context_data_product: client_suitability
version: 1.2.0
layers:
  base_data_product:
    capsule: client_holdings@4.1.0        # Part II foundation
  ontology:
    ref: ontologies/wealth/suitability@2.0    # entities, relationships, rules
    entities: [Client, Portfolio, Holding, AssetClass, RiskProfile]
  semantic_layer:
    metrics: [em_exposure, suitability_breach, risk_capacity]
    definitions_ref: semantic/wealth/@1.4
  knowledge_graph:
    endpoint: "graph://meridian/wealth/suitability"
    enrichment: [lineage, confidence_scoring]
  usage_intelligence:
    documented_patterns: 5
    known_limitations_ref: docs/suitability_limits.md
exposure:
  semantic_api: "https://api.meridian/context/wealth.client_suitability"
  catalogue_urn: "urn:cdp:wealth:client_suitability"
  contract_ref: contracts/context/client_suitability.yaml
certification: { level: 4, ref: certs/client_suitability.yaml }   # Part V
```

Read top to bottom and the anatomy is legible as a stack: a governed capsule at the base (Part II), an ontology and semantic layer of meaning above it (Chapter 17), a knowledge graph that connects and scores (Chapter 13), and usage intelligence on top — all exposed through one API and one catalogue entry, and (looking ahead) carrying a certification. This descriptor is what makes the "five layers" concrete: not a picture, but a document that the platform assembles, versions, and serves. Everything in Chapters 16 and 17 is about producing the ontology, semantic, and graph layers this descriptor references.

## Exposure: a product is only as good as its interface

Composition is half the job; exposure is the other half, and it is where many semantic initiatives quietly fail — they model beautifully and expose nothing an agent can call. A context data product has to be *reachable* in the way its consumers actually work.

Three exposure requirements matter. First, a **semantic query API** that AI agents and analytics tools can call programmatically — the copilot does not browse a catalogue, it calls an endpoint and expects a contextually enriched response. Second, **discoverability in the enterprise catalogue** with the product's full contextual metadata attached, not buried in a separate semantic system nobody finds; Helena's catalogue (reframed in Chapter 14 as a consumer of the source of truth) is where a new consumer discovers the product exists. Third, and this is the thread that ties Part IV back to the whole book, the **data contract is extended to include semantic obligations** — the ontological definitions, the confidence-scoring commitments, the usage documentation — alongside the traditional schema and quality terms. The contract that Chapter 6 introduced as the capsule's outer envelope now carries meaning as a first-class, promised, versioned obligation.

### The contract, extended; the API, in use

Two artefacts make "exposure" concrete. First, the **context-extended contract** — the capsule's contract from Chapter 6, now carrying *semantic obligations* alongside schema and quality:

```yaml
# contracts/context/client_suitability.yaml (semantic section)
semantic_obligations:
  definitions:
    - metric: em_exposure
      promise: "division-aware; retail excludes frontier, institutional includes"
    - metric: suitability_breach
      promise: "allocation outside risk-profile band per policy WM-2024-03"
  confidence:
    freshness: "< 4h"
    completeness: "> 99.9%"
    confidence_signal: "returned with every response"
  permitted_uses: [adviser_copilot_retrieval]
  prohibited_uses: [unsupervised_client_facing_advice]
```

The product no longer merely promises *fresh, complete data*; it promises *this definition of EM exposure, this confidence signal, these permitted uses* — meaning as a first-class, versioned contractual obligation.

Second, the **semantic API in use** — the Adeyemi question of the chapter's opening, answered on the wire. The copilot translates the adviser's natural-language question into a *structured* call — natural-language understanding stays in the agent; governed meaning stays in the product — and asks:

```json
// request — a structured intent, not free text
POST /context/wealth.client_suitability/query
{ "intent": "suitability_assessment", "client_id": "C-88213",
  "metric": "em_exposure", "caller_division": "retail" }

// response — data PLUS the context needed to reason correctly
{ "answer_context": {
    "em_exposure_pct": 22.0,
    "em_definition_applied": "retail (frontier excluded)",
    "frontier_inclusive_pct": 29.5,          // the number the copilot must weigh
    "risk_profile": 3,
    "suitable_band_max_pct": 20.0,
    "suitability_breach": true,
    "confidence": 0.97, "as_of": "2026-06-30T08:00:00Z" },
  "caveats": ["client-facing advice requires human review (prohibited_use)"] }
```

Notice what the response carries beyond the number: *which* EM definition was applied, the frontier-inclusive figure the copilot must also consider, the breach verdict computed from the governed rule, a confidence signal, and the prohibited-use caveat. The agent does not infer any of this — it is served. The Adeyemi ambiguity is resolved *programmatically*, from the ontology's rule, exactly as promised. A governed table returns a value; a context data product returns a value *plus the meaning, currency, and constraints needed to act on it safely.* That is the exchange the whole part exists to make possible.

The response settles one architectural question worth stating plainly: where do the *values* come from — the knowledge graph or the lakehouse? The division of labour is deliberate. The knowledge graph carries *meaning and relationships* — the entities, the links, the rules, the confidence signals. The *values* (`em_exposure_pct` and the holdings figures) are computed by the semantic layer against the governed lakehouse at query time, which is why the response carries a fresh `as_of`. The API composes the two. The graph is not a second analytical store holding a copy of the holdings; bulk-loading fact data into it is the most common misimplementation, and the one to avoid.

## What "done" looks like, and where it points

For Meridian's suitability domain, done means this: the adviser copilot can query the domain and get a contextually enriched answer to the Adeyemi question — resolving the frontier-market ambiguity *programmatically*, from the ontology's rule, without a human interpreting anything. The product carries lineage, confidence scoring, and a set of documented usage patterns. The ambiguity that caused the Okonkwo breach in Chapter 5 is now impossible to hit blind, because the meaning that resolves it travels with the product and is served through the API the agent calls.

That is the *design* answer to the question this book has circled since Chapter 4: what does AI need from data? It needs context — bound (Part II), active (Part III), and now packaged (Part IV) into a product it can reason over. But designing a context data product and *proving it is safe to use* for a given purpose are two different things. The Adeyemi answer is only as good as the assurance that the product is fit for client-facing suitability advice, under the right controls, with the right evidence — and that assurance question is what Part V answers with certification. For now, Part IV has two chapters left: the ninety-day method for actually building the thing without stalling in slide decks, and the deeper craft of the meaning layers that most teams get wrong.

## One product, three consumers

A context data product is not built once per consumer; it serves several from the same five layers, differently. The same `client_suitability` product serves: an **AI agent** (the copilot), which calls the semantic API and needs the definitions, confidence, and prohibited-use caveats returned inline because it carries no context of its own; an **analyst** in Tableau, who reaches the *semantic layer* directly and gets the same governed `em_exposure` metric, so their number matches the copilot's by construction; and a **regulatory report**, which reads the *base capsule* with full lineage and the point-in-time history, and does not want the semantic conveniences, only the auditable facts. One product, three interfaces onto the same governed meaning — which is exactly why the semantic layer's "one definition, served identically" guarantee (Chapter 17) matters: it is what keeps the agent's answer, the analyst's dashboard, and the regulator's report from being three different numbers.

## What it costs, and who staffs it

The increment is real, and it is mostly *people*, not infrastructure. A context data product costs more than a plain data product, and over a governed capsule the added effort is the ontology and semantic modelling (a modeller or analyst-engineer, weeks per domain — Chapter 16's 90 days), the knowledge-graph assembly (largely connecting existing metadata, not new capture), and the ongoing ownership of the meaning as the business changes. The infrastructure increment — a graph store, a semantic-layer tool, a semantic API — is real but modest and largely open-source at pilot scale. The honest framing for a sponsor: the base data product is the expensive foundation and you have already paid for it (Parts II–III); the context layer is an *enrichment* on top, weeks not years per domain, and it is what turns a foundation an agent cannot use into a product an agent can.

> **Beyond finance.** The five layers are not a wealth-management artefact; they generalise. A **hospital** builds a context data product over a clinical-terminology dataset: the base is the governed patient-observations table; the ontology models Patient, Encounter, Observation, and the rules that govern them (what "abnormal" means for this assay, which reference range applies by age); the semantic layer defines "readmission" once; the graph connects observations to encounters with provenance; and an agent answering a clinician's question is served the governed definition and the caveat rather than inferring from a column called `val_flag`. A **retailer** does the same over its product catalogue — Product, Category, Attribute, the rules that decide what "in stock" and "comparable item" mean — so a shopping agent recommends from governed meaning rather than a mislabelled feed. A **SaaS company** builds one over its usage and billing data — Account, Subscription, Feature, Event, and the rules that decide what "active account," "seat," and "churn" mean across plan types — so a support or renewals agent reasons from one governed definition of churn rather than the three that live in three dashboards. Different domain, identical anatomy: base product, ontology, semantic layer, knowledge graph, usage intelligence, exposed through an API a reasoning consumer can call.

## Further reading

- On the data-product foundation, *Data Mesh* (Zhamak Dehghani) for data-as-a-product and the product-contract idea this chapter extends.
- On the semantic layer, the dbt MetricFlow / Semantic Layer and LookML documentation (developed in Chapter 17).
- On the domain ontology, the Financial Industry Business Ontology (FIBO, EDM Council / OMG) as an accelerator — borrow patterns, not the whole model (Chapters 16–17).

> **Chapter summary.** A governed data product delivers trustworthy *data* but leaves *meaning* to the consumer — fine for a human who carries context, fatal for an AI agent that does not. A **context data product** packages meaning alongside data in machine-readable, queryable form: semantic, temporal, usage, and confidence context assembled from the capsule and active-metadata foundations. Its anatomy is five layers — base data product, ontology, semantic layer, knowledge graph, usage intelligence — and it must be *exposed* through a semantic query API, discoverable in the catalogue, with the data contract extended to carry semantic obligations. "Done" is an agent resolving domain ambiguity programmatically. This is the *design* answer to what AI needs from data; Part V supplies the *assurance* answer.

> **Try this.** Take the last question an AI agent got confidently wrong in your organisation and locate where the correct answer actually lived: in a column, or in the *relationship between concepts and a business rule*? If the latter, you have found why a governed table was not enough — and the first entity and rule for your ontology layer.
