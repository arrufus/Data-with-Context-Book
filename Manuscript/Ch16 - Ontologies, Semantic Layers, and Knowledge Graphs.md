# Chapter 16 — Ontologies, Semantic Layers, and Knowledge Graphs in Practice

> **Where you are:** Part IV — *Package*, closing chapter. The playbook gave you the plan; this chapter is the craft beneath its two hardest phases — building meaning an agent can actually reason over. Get this right and you have a context data product; get it wrong and you have an elegant artefact nobody queries. It closes the design half of the book and hands off to *Prove*.

---

## Three words teams use interchangeably and shouldn't

The playbook moved briskly through "build a minimum viable ontology" and "bootstrap a knowledge graph," and it moved through a semantic layer without dwelling. In a ninety-day plan that pace is correct. But those three things — ontology, semantic layer, knowledge graph — are where a context-data-product initiative actually succeeds or fails, and teams routinely conflate them, which is the first thing to fix. They are not synonyms, and they are not interchangeable. They are three distinct layers of meaning that do different jobs, and an agent needs all three.

An **ontology** is the *model of the domain*: what kinds of things exist and how they relate. It is a schema for meaning — Client, Portfolio, Holding, Asset Class, Risk Profile, and the relationships and rules among them — independent of any particular row of data. It answers *what things are and how they connect in principle.*

A **semantic layer** is the *mapping from business concepts to physical computation*: it says how "EM exposure" or "suitability breach" is actually calculated from the columns in the warehouse, and it guarantees one governed definition per concept. It answers *how a business concept is computed, consistently, for everyone.*

A **knowledge graph** is the *populated, connected instance*: the ontology's entities instantiated with real data, linked, and enriched with lineage and confidence. It answers *how these specific things actually connect, right now, and how much to trust each connection.*

The relationship is layered and worth holding precisely: the ontology is the schema of meaning; the semantic layer binds that meaning to physical calculation; the knowledge graph is the meaning made concrete and traversable. An agent reasoning about the Adeyemi trust's suitability needs the ontology (to know a Client *has* a Risk Profile and *holds* Holdings with Asset Classes), the semantic layer (to compute EM exposure under the retail definition), and the knowledge graph (to traverse from this client to these holdings to the applicable rule). Miss any one and the reasoning breaks. This chapter takes them in turn.

## Building the ontology well

The playbook's rule — five to ten entities, not the enterprise — is the single most important discipline, and it deserves reinforcing because the pull toward comprehensiveness is relentless. Start from the *consumer's questions*, not the domain's totality. Meridian's ontology exists to answer suitability questions the copilot receives; every entity and rule earns its place by being needed for one of those questions. Client, Portfolio, Holding, Asset Class, Risk Profile are in because a suitability judgement requires them. Custodian, settlement instruction, and fee schedule are out — real, related, and irrelevant to *this* consumer. The bounded ontology is not a compromise; it is the design.

Three elements of ontology craft repay attention.

**Relationships need cardinality and directionality, not just existence.** "A Client has a Portfolio" is underspecified. Can a client have several portfolios (a retail account and a SIPP)? Does a holding belong to exactly one portfolio? The agent reasons over these multiplicities — a suitability calculation that assumes one portfolio per client will misread a client who has three. Specify them explicitly: a Client has *one or more* Portfolios; a Portfolio contains *zero or more* Holdings; a Holding has *exactly one* Asset Class. Directionality matters too — the relationship from Client to Risk Profile ("is assessed as") is not the same as the reverse.

**Business rules are where the value concentrates.** The entities and relationships are the skeleton; the rules are the reason the ontology resolves ambiguity. Meridian's two load-bearing rules — *risk tolerance scored 1–10, reassessed annually* and *EM includes frontier for institutional, excludes it for retail* — are what turn a diagram into a decision engine. When Maya's team builds the ontology, most of their real effort goes into surfacing, agreeing, and encoding rules like these, because each one is a place the copilot would otherwise guess. The Okonkwo breach was a missing rule; the Adeyemi answer is a present one.

**Controlled vocabularies close the ambiguous fields.** Where a field's values carry meaning that an agent could misread — region codes, product classifications, risk bands — an explicit controlled vocabulary removes the guesswork. "EM," "DM," "FM" are not free text an agent should interpret; they are a governed enumeration with defined membership that, at Meridian, varies by division. The vocabulary is where that variation is pinned down.

On acceleration and restraint together: **FIBO** is a genuine gift for financial services — three thousand-plus entities of ready patterns for instruments, legal entities, and contracts — but the discipline is to *borrow patterns, not swallow the model.* Meridian adapts FIBO's asset-class treatment to its own taxonomy and leaves the other 2,990 entities alone. Adopting FIBO wholesale is the ontology-perfectionism failure mode wearing a standards badge; the reference ontology is a vocabulary to draw from, not a project to complete. And throughout, extract from what exists — the glossary, the SME interviews, the logical data model, the "what does X mean?" support threads — rather than inventing from a blank page. The knowledge is already in the building; the ontology formalises it.

### The MVO, written out

The chapter's central artefact has been invisible; here it is. Meridian's minimum viable ontology for suitability, in YAML/JSON-LD-style form — five entities, their relationships with cardinality, and the two load-bearing rules:

```yaml
ontology: wealth/suitability
version: 2.0
entities:
  Client:      { keys: [client_id], attrs: [division, risk_profile_ref] }
  Portfolio:   { keys: [portfolio_id], attrs: [client_ref] }
  Holding:     { keys: [holding_id], attrs: [portfolio_ref, asset_class_ref, value] }
  AssetClass:  { keys: [asset_class_code], attrs: [is_emerging_market, is_frontier] }
  RiskProfile: { keys: [profile_id], attrs: [score, assessed_date] }
relationships:
  - { from: Client, to: Portfolio, verb: holds, card: "1..*" }
  - { from: Portfolio, to: Holding, verb: contains, card: "0..*" }
  - { from: Holding, to: AssetClass, verb: classified_as, card: "1..1" }
  - { from: Client, to: RiskProfile, verb: assessed_as, card: "1..1" }
rules:
  em_classification:
    when: "division == institutional"
    then: "EM includes frontier (is_frontier -> counts as EM)"
    else: "EM excludes frontier (retail)"
  risk_tolerance:
    scale: "1 (conservative) .. 10 (aggressive)"
    reassessed: annually
controlled_vocabularies:
  region_code: [DM, EM, FM]
```

Everything the Adeyemi answer needed is here: the entities, the cardinalities (a Client `holds` one-or-more Portfolios — the multiplicity that a naive model gets wrong), and the `em_classification` rule that a machine can apply per division. This is what "sign-off by the domain owner" signs off — a precise, bounded, machine-readable model, not a diagram.

### Borrowing from FIBO, not adopting it

The discipline of "borrow patterns, not the model" is clearest shown side by side. FIBO models a financial instrument with a rich class hierarchy — `FinancialInstrument → DebtInstrument → Bond`, with properties for issuer, maturity, and classification. Meridian *borrows the pattern* — the idea of an instrument classified into an asset-class taxonomy with an issuer and a market classification — and *adapts it* to its own five-entity model, mapping FIBO's classification pattern onto its `AssetClass` entity and its EM/frontier flags. It does not import FIBO's three thousand classes, ninety-nine per cent of which are irrelevant to a suitability question. The borrow is a paragraph of adaptation; adopting FIBO wholesale would be the multi-year ontology-perfectionism project the playbook exists to prevent. FIBO is a vocabulary to draw from, and the drawing is deliberate and small.

### A gallery of modelling mistakes

Four mistakes recur often enough to name, each with its fix, because recognising them is half of ontology craft:

- **Attribute-as-entity.** Modelling `risk_score` as its own entity rather than an attribute of `RiskProfile` — bloating the model and complicating every query. *Fix:* an entity is something with an identity and a lifecycle; a value that only describes something else is an attribute.
- **Relationship without cardinality.** "Client has Portfolio" with no multiplicity, so a query assumes one portfolio per client and misreads the client with three. *Fix:* every relationship states cardinality and direction.
- **Enum-as-free-text.** Leaving `region_code` as free text an agent must interpret, rather than a controlled vocabulary with defined membership. *Fix:* enumerate the ambiguous fields.
- **Rule-in-prose.** The EM rule written as a glossary sentence rather than an encoded, machine-applicable rule — readable, unenforceable, and exactly how Okonkwo happened. *Fix:* rules are code the platform applies, not prose a human is meant to remember.

## Building the semantic layer

If the ontology says *what* "EM exposure" means, the semantic layer says *how it is computed* — and, critically, guarantees that everyone computes it the same way. This is the layer that most directly attacks the failure that has haunted the book since Chapter 5: the analyst's "EM exposure" and the copilot's "EM exposure" being *different numbers* because they were calculated from different columns with different assumptions.

A semantic layer (dbt's MetricFlow, LookML, or a dedicated tool) sits above the Gold tables and maps physical columns to governed business concepts with their calculation logic. One definition of "active client." One definition of "suitability breach." One calculation of "EM exposure," parameterised by division per the ontology's rule. Every consumer — the copilot, an analyst in Tableau, a regulatory report, a downstream team's own product — draws the concept from the same governed definition. This is the modelling-as-UI argument of Chapter 8, now formalised as a product layer and made non-optional: the semantic layer is the single translation engine between the messy physical schema and the clean business concepts, and it serves humans and agents identically.

The practical payoff is that the semantic layer is what makes the context data product *trustworthy across consumers*. Without it, each consumer re-implements the calculation and the definitions silently diverge — the drift of Chapter 3, resurfacing at the metric level. With it, a change to how "suitability breach" is defined happens once, in the layer, and propagates to everyone. For a regulated firm, this is not merely convenient; it is the difference between being able to tell an auditor "every consumer uses the same certified definition" and having to reconcile five different numbers after the fact.

### The semantic layer, in code

"One governed definition per concept" is abstract until you see the concept defined once, in code, parameterised by the ontology's rule. Here is `em_exposure` in a MetricFlow/LookML-style semantic definition:

```yaml
metric:
  name: em_exposure
  description: "Emerging-market exposure %, division-aware per ontology rule"
  type: ratio
  numerator:
    measure: holding_value
    filter: >
      asset_class.is_emerging_market
      OR (client.division == 'institutional' AND asset_class.is_frontier)
  denominator:
    measure: holding_value
  governed_by: ontologies/wealth/suitability@2.0#em_classification
```

The `filter` *is* the ontology's EM rule, expressed once, in the one place every consumer draws from. When an analyst charts EM exposure in Tableau, when the copilot answers a suitability question, when the regulatory report aggregates it — all three resolve to *this* definition, so they cannot disagree. And `governed_by` binds the metric back to the ontology rule, so changing the rule is a governed, traceable act, not a silent edit in one of five places. This single definition is the antidote to the metric-level drift the chapter warns about: the analyst's "EM exposure" and the copilot's are guaranteed identical because they are the same code.

## Building the knowledge graph

The liberating insight from the playbook bears repeating because it dissolves the most common blocker: you do not build a knowledge graph from zero. You already have one, scattered across the catalogue, the lineage tools, the glossary, and people's heads. The work is *assembly*.

The high-value, irreducibly manual step is **mapping catalogue entries to ontology entities** — deciding that this catalogued table *is* the Client entity, that this column *is* the risk-profile assessment. That mapping cannot be automated because it is exactly the interpretive judgement the ontology was built to capture; it is where domain knowledge enters the graph. Around it, the enrichment is largely connection rather than creation: lineage flows in from the OpenLineage events Meridian already emits into the Chapter 12 graph; confidence scoring comes from the active-metadata trust signals of Chapter 11; usage intelligence accumulates from observed query patterns. The knowledge graph is where Parts II, III, and IV visibly converge — the capsule's bindings, the active layer's signals, and the ontology's meaning, joined into one traversable structure.

On tooling, the playbook's discipline is the right one and worth restating as a principle: **the knowledge graph is a logical construct first and a technology choice second.** A property graph in Neo4j Community Edition or a well-structured set of JSON documents is more than enough for a single-domain pilot. Procuring a managed graph platform before the pilot proves value is tool-first thinking — the failure mode that has teams comparing graph databases for months while no meaning gets modelled. Meridian stands up the smallest thing that works and evaluates managed platforms only after the suitability product earns the investment.

### Three queries that earn the graph

A knowledge graph justifies itself only if it answers questions a table cannot answer cheaply. Three do, and they are worth seeing in Cypher:

```cypher
// 1. Suitability traversal — the Adeyemi question, as a single hop-chain
MATCH (c:Client {client_id:'C-88213'})-[:HOLDS]->(:Portfolio)
      -[:CONTAINS]->(h:Holding)-[:CLASSIFIED_AS]->(a:AssetClass)
WHERE a.is_emerging_market OR (c.division='institutional' AND a.is_frontier)
RETURN c.division, sum(h.value) AS em_value;

// 2. Blast radius — what breaks if we redefine an asset class?
MATCH (a:AssetClass {code:'EQ-EM'})<-[:CLASSIFIED_AS]-(:Holding)
      <-[:CONTAINS]-(:Portfolio)<-[:HOLDS]-(c:Client)
RETURN count(DISTINCT c) AS clients_affected;

// 3. Which rule applied on a past date? (bitemporal, cf. Ch 12)
MATCH (r:Rule {name:'em_classification'})
WHERE r.valid_from <= date('2026-03-15') < r.valid_to
RETURN r.definition;
```

Each is a traversal that would be a brutal multi-join across silos in a relational world and is a single readable query here — the suitability reasoning the agent needs, the impact analysis the change process needs, and the point-in-time rule reconstruction the auditor needs. This is why the knowledge graph is a layer and not a luxury: the questions that matter to an agent and an auditor are *relationship* questions, and a graph answers relationship questions natively.

### Managing meaning as it changes

Ontologies and semantic definitions drift like everything else, so meaning needs change management too — the drift problem at the semantic level. Meridian versions its ontology (the `@2.0` in the specs) and treats a rule change as a governed release: redefining `em_classification` is a major version, requires the owner's sign-off, and — because the semantic layer's `governed_by` binds metrics to the rule — automatically flags every metric and every context product that depends on it. Deprecating an entity or migrating consumers when a rule changes follows the same capsule lifecycle from Chapter 5: draft, review, publish, deprecate, with notice to consumers. The point is that the meaning layers are not write-once artefacts; they are living products under the same version discipline as the data, because a semantic definition that drifts silently reintroduces the exact problem the semantic layer was built to solve.

## The three failure modes, revisited as craft

Chapter 15 named the three failure modes as things to avoid; here they recur as things the *craft* actively guards against, because each maps to one of the three layers.

**Ontology perfectionism** is a failure of the *ontology* layer — modelling the enterprise instead of the consumer's questions. The guard is the five-to-ten-entity bound and the consumer-first discipline: every entity justified by a question the copilot must answer.

**Tool-first thinking** is a failure of the *knowledge graph* layer — procuring technology before modelling meaning. The guard is "logical construct first": model the ontology and map the entities on paper or in YAML before any graph database is chosen.

**No consumer pull** is a failure that spans all three layers — building meaning nobody asked for. The guard is the live-consumer requirement baked into the playbook's day-65 deliverable: the copilot must be querying the graph, or the work has no target and the semantic layer has no one to serve.

Read together, the three layers and the three failure modes make the same point from two directions: meaning is built *for a consumer*, *bounded to a domain*, and *modelled before it is stored*. Violate any of those and you get the elegant artefact nobody queries.

## The operating-model shadow, and the handoff to Prove

One theme has been deferred through all of Part IV and can be deferred no longer, though its full treatment belongs to Part V. Everything in this chapter — an agreed ontology, a governed semantic layer, a maintained knowledge graph — requires *ownership*. Someone has to decide that "active client" means this and not that, has to sign off the EM rule, has to keep the semantic layer's definitions current as the business changes. The single discriminator between organisations that make semantic products work and those that produce costly relabelling exercises is whether domains genuinely operate as *product organisations*: identified consumers, articulated value, contracts, roadmaps, lifecycle management, funded and named ownership. "Data as a product" is not a storage concern; it is an operating-model commitment, and it is the gating capability. Meridian's suitability product works because Maya owns it with a real remit — not because the graph technology is good. This is the thread that Part V picks up when it turns to why data mesh stalls and why platform teams become bottlenecks: the packaging of Part IV is only durable on an operating model that funds and empowers domain ownership.

With that flagged, stand back and see where the book has arrived. Meridian's suitability domain is now a genuine context data product: governed data (Part II), kept active (Part III), and packaged with an ontology, a semantic layer, and a knowledge graph an agent can reason over (Part IV). The copilot answers the Adeyemi question correctly, resolving the frontier-market ambiguity from a rule rather than a guess. This is the complete *design* answer to the question the book posed in Chapter 4: what does AI need from data? It needs context — bound, active, and packaged.

But a designed context data product and a *trusted* one are different things. When the copilot gives a client-facing suitability answer, when Meridian's suitability model faces its model-risk committee, when a regulator asks whether the data behind an automated decision was fit for the purpose — the question is no longer *"is the context well-designed?"* but *"can you prove it is safe and appropriate for this use?"* That is the assurance question, and it is the last move of the discipline. If context data products answer the design question, **certification answers the assurance question.** Part V — *Prove* — is where the meaning Meridian has so carefully packaged is turned into evidence that survives an audit.

> **Chapter summary.** Ontology, semantic layer, and knowledge graph are three distinct layers of meaning, not synonyms: the ontology models the domain (entities, relationships, rules), the semantic layer maps business concepts to governed physical calculation, and the knowledge graph is the populated, connected, confidence-scored instance. An agent needs all three. Ontology craft: bound to the consumer's questions (5–10 entities), relationships with cardinality and directionality, business rules where the value concentrates, controlled vocabularies for ambiguous fields, FIBO for patterns not wholesale. The semantic layer guarantees one governed definition per concept across all consumers, killing metric-level drift. The knowledge graph is assembled from existing metadata (the manual, high-value step is mapping entries to ontology entities), logical construct before technology choice. The three failure modes map to the three layers. All of it rests on funded domain ownership — the handoff to Part V, where design becomes provable assurance.

> **Try this.** For one business metric your organisation argues about, write its single governed definition and calculation as it would appear in a semantic layer — then count how many places it is currently computed differently. Each divergence is a number an agent (or an auditor) could be given wrongly, and the reason the semantic layer is not optional.
