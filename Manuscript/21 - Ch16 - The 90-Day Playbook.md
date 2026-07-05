# Chapter 16 — The 90-Day Playbook: Shipping One Context Data Product

> **Where you are:** Part IV — *Package*, the hands-on spine. Chapter 15 defined the context data product. This chapter builds one, in ninety days, on Meridian's suitability domain — taking a single domain from **Level 1 to Level 3**. It is the most operational chapter in the book: a dated plan you can run on Monday.

---

## Where do we actually start?

Every team that buys the vision of context data products asks the same question next: *where do we actually start?* The vision is compelling, the architecture diagrams are clean, and six months later most of those teams are still at slide decks — because nobody told them where to begin cutting. The pattern is familiar and fatal: conference inspiration leads to an architecture diagram, which leads to a pilot proposal, which leads to tool-procurement debates, which leads to quiet abandonment. Semantic initiatives are especially prone to this, because they feel *foundational*, and foundational work is the easiest work in the world to defer.

This chapter exists to break that cycle. It is a phased plan to deliver a *single* context data product in ninety days — long enough to prove value, short enough to force the choices that make semantic work actually ship. The ninety-day constraint is not arbitrary; it is a forcing function. It makes you choose one domain, one consumer, one proof point, and it makes ambition the enemy rather than the goal. We run the whole plan on Meridian's client-suitability domain, because that domain has been the book's worked example since Chapter 5 and because — conveniently — it is exactly the kind of domain the playbook is designed for.

Before the timeline, name the three failure modes that derail most attempts, because the plan is built to avoid each one.

The first is **tool-first thinking**: procuring a knowledge-graph database before defining what knowledge matters. Teams spend months evaluating Neptune versus Neo4j versus TigerGraph while the actual modelling work — the hard, human, interpretive work of deciding what entities exist and how they relate — never begins.

The second is **ontology perfectionism**: attempting an enterprise-wide ontology before delivering one useful product. An enterprise ontology is a multi-year programme; a context data product for a single domain can be built in weeks. The ambition is understandable and it is fatal.

The third is **no consumer pull**: building a semantic layer nobody asked for, with no agent, copilot, or analytics use case waiting on the other end. Without a consumer pulling for the product, the pilot is an academic exercise, and academic exercises lose their funding.

The principle that defeats all three, and that underpins the entire plan: **start with a consumer problem, not a producer ambition.** Meridian has one waiting — the adviser copilot that got the Adeyemi question wrong.

### The 90 days, week by week

"A plan you can run on Monday" needs dates and owners, so here is the timeline as a table:

| Weeks | Phase | Lead | Effort |
|-------|-------|------|--------|
| 1–2 | Domain selection, brief | Product owner (Maya) | light |
| 3–5 | MVO: entities, relationships, rules | Modeller + 2–3 SMEs | heavy |
| 6–8 | MVO review & sign-off; controlled vocabularies | Modeller + owner | medium |
| 6–9 | Bootstrap graph from catalogue/lineage/glossary | Engineer | heavy |
| 9 | First AI consumer querying the graph | Engineer + ML | medium |
| 10–11 | Assemble 5 layers; semantic API; extend contract | Engineer + modeller | medium |
| 11–12 | Wire into catalogue; three usage patterns | Engineer | medium |
| 13 | Measure (4 metrics); value summary | Owner + analyst | light |

### Who runs it, and what it costs

The pilot team is small and the infrastructure cost is close to zero — the answer a sponsor is hoping for. Four part-to-full-time people: a **product owner** (accountable, ~0.3 FTE), an **engineer** (build, ~0.8), an **analyst/modeller** (the ontology and semantic work, ~0.6), and a **part-time steward** (governance checks, ~0.2). Infrastructure — Neo4j Community, an existing lakehouse, an open-source semantic layer — is near-zero at pilot scale; the recommendation to *not* procure a managed graph platform until value is proven is what keeps it that way. The number a sponsor asks for is therefore "a fraction of four people for a quarter, and essentially no new licences," against the value the day-90 summary will quantify.

## Days 1–15: Choosing the right domain

The single most consequential decision is the first one: which domain to pilot. Get it wrong and the pilot either drowns in complexity or succeeds at something nobody cares about. Five criteria reliably predict a good starting point.

First, the domain should have an **existing data product**, even a basic one — the goal is to *enrich*, not build from scratch. Meridian has `client_holdings`, governed and active. Second, there should be a **live or near-term AI use case** waiting for better context: the adviser copilot, which needs to interpret risk scores and resolve region-code ambiguity. Third, **domain ownership should already be established** — a named owner who can make semantic decisions, because someone has to decide what "active client" means and whether frontier markets count as EM. Maya owns the domain. Fourth, and paradoxically, **domains with known terminology disputes are better candidates**: if people have already argued about what a term means, the pain is felt and the value of resolving it is obvious. Meridian has argued about EM classification since before the Okonkwo breach. Fifth, **data volume should be manageable** — pick something mid-tier where the team can move fast without cross-divisional negotiations eating the timeline.

For financial-services firms, three domains reliably meet these criteria: customer analytics (high visibility, well-understood entities, clear AI use cases), regulatory compliance and suitability (where ambiguity is expensive and context gaps have audit consequences), and product reference data (stable entities with high reuse). Meridian's suitability domain sits squarely in the second, and it satisfies all five criteria without strain.

**Deliverable (day 15):** a one-page domain-selection brief — the domain, the consumer use case, the existing data product being enriched, and the success metric. One page, because if it takes more the domain is too big.

### The deliverable, filled in

The day-15 brief is a named deliverable, so here it is completed for Meridian — the whole thing fits on a page, which is the discipline:

```
DOMAIN SELECTION BRIEF — Client Suitability
Domain:            Wealth client suitability & holdings
Consumer use case: Adviser copilot — resolve EM-classification and
                   risk-band ambiguity in suitability answers
Existing product:  client_holdings (capsule, Level 1, governed)
Domain owner:      Maya Osei (accountable, funded)
Known dispute:     "EM" frontier-market inclusion differs retail vs institutional
Success metric:    Ambiguity-resolution rate > 90%; copilot suitability
                   error rate down measurably vs raw-data baseline
Out of scope:      Custodian ops, fees, settlement (adjacent, not needed)
```

## Days 15–40: The minimum viable ontology

The instinct now is to model everything. Resist it. A **minimum viable ontology** for a single-domain pilot covers just four things: five to ten core entities relevant to the use case; the relationships between them with *explicit cardinality and directionality*; the business rules that govern interpretation; and controlled vocabularies for the fields where ambiguity causes real problems.

For Meridian's suitability domain, the MVO defines five entities — **Client, Portfolio, Holding, Asset Class, Risk Profile** — with the relationships among them and, crucially, the explicit rules that resolve the ambiguities the copilot keeps hitting: *risk tolerance is assessed annually, scored 1 (conservative) to 10 (aggressive); region code "EM" includes frontier markets for the institutional division but excludes them for retail.* That is not a comprehensive enterprise ontology, and it is not trying to be. It is a precise, bounded model that resolves the specific ambiguities an AI agent encounters when answering a suitability question. The Adeyemi trust's frontier-market question is answered by a single rule in this MVO.

The good news is that most of this knowledge already exists in the organisation — it is simply not formalised — and four extraction techniques accelerate the work. **Mine the business glossary** for entity and term definitions, even incomplete ones. **Interview two or three domain experts** using a structured template rather than open-ended workshops — for each candidate entity and term, ask the same four questions (*What is it? How is it identified? What does it relate to, and with what multiplicity? What rule or exception governs how it is interpreted?*), because the template forces the specificity ("EM excludes frontier for retail") that an open workshop talks around. **Reverse-engineer the logical data model**, because the relationships are already implicit in the existing schemas. And **review support tickets, analyst questions, and Slack threads** for recurring "what does X mean?" signals — those signals are the highest-priority candidates for ontological definition, because they are exactly the ambiguities that are already costing time. For Meridian, the EM-classification argument is all over the support history; it self-selects as the first rule to formalise.

On tooling, start lightweight. If the team has semantic-web skills, OWL or RDF is natural; if not, well-structured JSON-LD or even YAML will serve for the pilot. The only hard requirement is that the ontology be machine-readable and loadable into a graph later. **Do not procure a graph database at this stage** — that is tool-first thinking, and it is a trap. And for financial services specifically, the **Financial Industry Business Ontology (FIBO)** — an EDM Council / OMG reference ontology of over three thousand entities covering instruments, legal entities, and contracts — is a powerful accelerator. The point is *not* to adopt FIBO wholesale (that is ontology perfectionism) but to use its patterns and definitions as a starting vocabulary the team adapts to its own context. Meridian borrows FIBO's treatment of instruments and asset classes and adapts it; it does not swallow FIBO whole.

**Deliverable (day 40):** a documented ontology covering the core entities, relationships, and business rules, reviewed and signed off by the domain owner. Maya's signature is the gate — an ontology no owner has endorsed is an academic artefact.

## Days 40–65: Bootstrapping the knowledge graph

The key insight for this phase reframes the whole task: the organisation does *not* need to build a knowledge graph from zero. It already has one — scattered across the data catalogue, the lineage tools, the glossary, and the analysts' heads. The work is assembly, not creation.

The sequence is clear. **Export entity and relationship metadata** from the existing catalogue. **Map those catalogue entries to the ontology entities** from the previous phase — this is the manual, high-value work that cannot be automated away, and it is where the real intellectual effort of the phase goes. **Enrich with lineage metadata** showing where each element originated and what transformations shaped it — Meridian already emits this as OpenLineage into the standards-based graph of Chapter 13, so much of it is a matter of connecting rather than capturing. **Add confidence scoring** — freshness indicators, completeness metrics, known quality issues — which is the active-metadata trust signal of Chapter 12, now attached to graph nodes. And **layer in usage intelligence** — common query patterns, documented use cases, known limitations.

For a single-domain pilot, a property graph in Neo4j Community Edition is more than sufficient; a well-structured set of JSON documents would also work. The knowledge graph is a *logical construct first, a technology choice second* — evaluate managed graph platforms only after the pilot proves value. Meridian resists the urge to procure and stands up a Community-Edition graph for the suitability domain.

The harder challenge in this phase is cultural, not technical, and it has to be met head-on because Maya will hear it from her own domain experts: *"Why should I invest time enriching metadata when my pipeline already delivers clean data?"* The answer is the one this book keeps arriving at: **clean data an AI agent cannot interpret is a liability, not an asset.** The industry evidence is on the table — the projection that organisations will abandon most AI projects unsupported by AI-ready data. Semantic enrichment is not additional bureaucracy; it is what protects the domain's *existing* investment from becoming invisible to the systems that increasingly need to consume it. The clean holdings pipeline Maya's team is proud of is worth *more* after enrichment, not less, because now the copilot can actually use it.

**Deliverable (day 65):** a queryable knowledge graph for the domain, populated from existing metadata, with *at least one AI agent or analytics consumer successfully using it.* The consumer requirement is non-negotiable — it is what keeps the pilot honest and defeats the no-consumer-pull failure mode. For Meridian, the copilot is querying the graph by day 65.

### What actually went wrong (because it always does)

The plan above reads frictionless, and no real 90 days is. Two setbacks inside Meridian's pilot are worth telling, because the *recovery* is the transferable part.

**The SME who wouldn't sign the rule.** Around week 5, the institutional and retail desks could not agree to *write down* the frontier-market rule — not because they disagreed about the fact, but because each worried that formalising "their" definition would be read as conceding the other's was equally valid. The modelling stalled for a week on what was really a political, not a semantic, problem. The recovery: Maya, as accountable owner, reframed it — the ontology would record *both* division-specific rules explicitly, not adjudicate between them, because the whole point was that the copilot must apply the *right* one per division. Once "we are documenting that they differ" replaced "we are deciding who's right," the sign-off came in a day. The lesson: ambiguity disputes are often ownership disputes in disguise, and a funded owner with the authority to reframe is what unsticks them — foreshadowing Chapter 21.

**The week lost to the graph-tool debate.** Around week 7, the engineering team spent the better part of a week arguing Neo4j versus a managed graph platform versus a document store — the exact *tool-first* failure mode the playbook warns against, committed by the very team running the playbook. The recovery was a rule imposed from outside the debate: stand up Neo4j Community *this week*, prove value, and revisit the platform question only if the pilot succeeds. The debate evaporated once "decide the vendor" was replaced by "the graph is a logical construct first." The lesson is the playbook's own: the failure modes are not things that happen to other people; they are the default gravity, and staying out of them takes active discipline even when you know they exist.

## Days 65–80: Assembling the context data product

With the ontology defined and the graph populated, assembly brings the five layers of Chapter 15 together — base data product, ontology, semantic layer, knowledge graph, usage intelligence — into one coherent product. Composition is the easy half; the requirements for *exposure* are the three Chapter 15 set out: the semantic API, catalogue discoverability, and the context-extended contract.

Assembly week is where those requirements stop being requirements and become tickets. The semantic API is stood up in front of the graph; the catalogue entry is generated from the product descriptor rather than hand-written; the contract's new semantic section — the EM definition, the confidence commitments, the permitted and prohibited uses — is reviewed by the same two desks that signed the EM rule. By day 80 the copilot is calling the API in staging and the contract diff is in review. The work here is sequencing and sign-off, not invention: everything being assembled was designed in the phases before it.

What "done" looks like: the copilot queries the suitability domain and receives a contextually enriched answer, with ambiguous terms — frontier-market inclusion, risk-band interpretation — resolved *programmatically*, not by a human. The product carries lineage, confidence scoring, and at least three documented usage patterns, and the Adeyemi question of Chapter 15 returns a correct, defensible answer because the rule that resolves it is in the product and served through the API.

## Days 80–90: Measuring what matters

The final phase is measurement and expansion planning, and the measurement is what converts a successful pilot into a funded programme. Four metrics prove value.

**Time-to-answer**: how long does an analyst or agent take to get a trustworthy answer from this domain, versus before the context product existed? **Ambiguity resolution rate**: how many "what does X mean?" queries are now resolved by the product itself rather than by a human? **Agent accuracy**: for the copilot, are error and hallucination rates measurably lower against the context product than against the raw data product? **Reuse rate**: are other teams beginning to pull from this product without being asked — the surest sign it is a real product and not a pilot?

### Measuring it, with instruments and numbers

The four metrics only persuade if you show *how* they are captured and *what* they moved, so here is the instrumentation and Meridian's before/after.

- **Time-to-answer** — captured from query logs (timestamp in, trustworthy answer out). *Before:* a suitability question with an EM-classification wrinkle took an adviser a half-day of back-and-forth with the desk. *After:* seconds, from the copilot.
- **Ambiguity-resolution rate** — captured by tagging support tickets and copilot fallbacks as "definitional." *Before:* ~35% of suitability queries needed a human to resolve "what does this mean." *After:* under 8% — the product resolves the rest itself.
- **Agent accuracy** — captured with a golden-set eval harness of suitability questions with known-correct answers, run against the raw product and the context product. *Before:* the raw-data baseline erred on the division-dependent cases (the Okonkwo class). *After:* those errors substantially eliminated, because the rule is applied, not inferred.
- **Reuse rate** — captured from the semantic API's caller log. *After:* a second team (client reporting) began pulling the governed `em_exposure` definition without being asked — the surest sign it is a real product.

Package these into a **one-page value summary** for leadership — the day-90 deliverable — honest about what was hard (the two setbacks above) because credibility funds the next domain. Then identify the next two domains and propose a **lightweight context standard**: the minimum semantic enrichment every product should carry going forward (an owner, agreed definitions for its key terms, a machine-readable ambiguity note), which is the seed of scaling from one product to a discipline.

### Scaling to domain two and three

The value summary funds the next domains, and the second is faster than the first because the *pattern* is reusable even though the *content* is not. Reusable across domains: the phase structure, the brief and interview templates, the graph scaffolding, the semantic-API shape, the CI wiring, and the context standard. Per-domain and non-transferable: the ontology (each domain's entities and rules are its own), the specific business rules, and the SME relationships. Meridian's second domain — regulatory suitability reporting — reused every scaffold and spent its real effort only on the new ontology and rules, hitting Level 3 in noticeably fewer than 90 days. The scaling law of the whole book applies: prove the pattern once, then the marginal domain is mostly assembly, and the discipline becomes self-propagating rather than a series of heroic pilots.

## One domain, one consumer, one proof point

Step back and see what the ninety days did. It took Meridian's suitability domain from a governed-but-mute data product (Level 1) to a modelled ontology (Level 2) to a populated knowledge graph with a live AI consumer (Level 3) — the exact climb the maturity ladder describes, achieved deliberately and provably in a quarter. It did so by cutting one precise slice: one domain, one consumer, one proof point. Context data products are not a rip-and-replace initiative; they are an enrichment layer on top of the federated architecture the organisation already spent years building, and the ninety-day constraint is what forces the pragmatism that makes them ship.

The competitive window is open but narrowing. Metadata-maturity surveys consistently place only a minority of organisations at the top levels, while agentic AI is moving from pilot to production fast. The firms that ship structured, machine-readable context into their products now will set the terms for how AI agents consume their data; the ones waiting for the perfect enterprise ontology will still be workshopping definitions while their competitors' agents are already acting. Ninety days, one domain, one consumer. Start cutting.

The playbook gives you the *plan*. What it necessarily compresses is the *craft* of the two hardest phases — building an ontology that an agent can actually reason over, and assembling a semantic layer and knowledge graph that serve humans and machines from the same governed meaning. Those phases are where a pilot either produces a genuine context product or an elegant academic artefact, and they deserve a chapter of their own. That is Chapter 17.

## Further reading

- On the domain-ownership operating model the playbook assumes, *Data Mesh* (Zhamak Dehghani) — the operating-model dependency Chapter 21 confronts.
- On the accelerator ontology, the Financial Industry Business Ontology (FIBO, EDM Council) — borrow patterns, not the whole model.
- On the tooling named here, the Neo4j (Community Edition) and dbt MetricFlow / semantic-layer documentation (the craft of these is Chapter 17).

> **Chapter summary.** Most semantic initiatives die at slide decks; the ninety-day plan is a forcing function against three failure modes — tool-first thinking, ontology perfectionism, and no consumer pull — under one principle: start with a consumer problem. The phases: **Days 1–15** choose the domain by five criteria (existing product, live AI use case, established ownership, known terminology disputes, manageable volume); **Days 15–40** build a minimum viable ontology (5–10 entities, relationships, business rules, controlled vocabularies), extracted from existing sources, accelerated by FIBO, without procuring a graph DB; **Days 40–65** bootstrap the knowledge graph from existing metadata with a live consumer; **Days 65–80** assemble and expose the five-layer product via a semantic API with a context-extended contract; **Days 80–90** measure (time-to-answer, ambiguity resolution, agent accuracy, reuse) and plan expansion. Meridian's suitability domain climbs Level 1 → 3 in a quarter. One domain, one consumer, one proof point.

> **Try this.** Write the one-page domain-selection brief for your own first context data product today: the domain, the consumer with a live AI use case, the existing product you would enrich, the named owner, and the success metric. If you cannot fill in the consumer or the owner, you have found why a pilot would stall — fix that before writing any ontology.
