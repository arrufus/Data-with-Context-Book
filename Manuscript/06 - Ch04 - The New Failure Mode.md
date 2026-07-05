# Chapter 4 — The New Failure Mode: AI Dies at the Data Layer

> **Where you are:** Part I — *Why the old model breaks*, final chapter. Chapters 1–3 diagnosed the disease: data engineering separated data from its meaning, the lake hardened that into a philosophy, and the drift was survivable only because humans absorbed it. This chapter shows what happens when the human is removed from the loop — and then draws the map the rest of the book follows. It introduces the running case study, the four moves of the discipline, and the maturity ladder you will climb.

---

## A 60% chance it won't work — on one condition

You have the budget. You hired the ML engineers. The GPUs are humming. The AI project is finally real.

Whether it survives is now, according to Gartner, mostly a bet on something that has nothing to do with the models. Through 2026, the firm projects, organisations will abandon roughly 60% of AI projects that are *not supported by AI-ready data*. Read the condition carefully, because it is the whole argument: what decides these projects is not the model architecture, the compute, or the talent — it is whether the data underneath was built for this kind of work. The question is which side of that line your data puts you on, and most teams find out in month three, when the pilot that should have worked quietly stalls. The pattern repeats across industries with almost tedious reliability, and it is almost never the technology that failed.

This is the failure mode Part I has been building toward. Chapters 1 through 3 explained, in three different registers, how data came to be separated from its meaning. This chapter explains why that separation, which was a chronic and survivable condition for thirty years, has suddenly become an acute and project-killing one. The short version: the consumer changed, and the new consumer has none of the forgiveness the old one had.

## AI does not work around problems. It learns from them.

Machine learning pipelines are absurdly sensitive to the quality and coherence of their inputs. Feed a model inconsistent formats, stale records, or duplicated entries and it does not fail in an obvious, investigable way. It produces answers that look right and are not. It drifts. It hallucinates. It learns the contradictions in the data as if they were signal.

Most enterprise data was not built to withstand this, because it was built for a different consumer. It was built for reports, for compliance, for humans who could spot when something looked off. A strange revenue number? Someone investigated. A missing customer field? Someone worked around it. There are finance teams that operated for months on a dataset with duplicate client records and never noticed, because the analysts instinctively filtered the noise when building their quarterly decks. A human brain does that automatically, without being asked, without even registering that it is doing it. A model cannot. The human safeguard that Chapter 1 identified as the thing silently holding the whole edifice together — judgement in the loop, filling in what the data could not say for itself — is exactly what an AI consumer lacks.

And the distinction is sharper than "AI needs cleaner data," because the feedback loops are different in kind. In traditional BI, a bad number surfaces in a dashboard and someone raises a ticket; the error is visible and contained. In an ML pipeline, a bad number trains the model to produce more bad numbers, and the error is invisible and compounding. The model does not flag the contradiction it was fed. It internalises it and serves it back, confidently, thousands of times an hour. That is what it means to say AI does not work around problems in the data — it learns from them. The slack is gone, and with it the margin that made decades of data drift survivable.

## Three shapes of the failure

"AI dies at the data layer" is a slogan until you can recognise the specific shapes it takes, because they look different in production and a team that has seen only one will be blindsided by the others. There are three archetypes, and every reader building with AI in 2026 will meet at least one.

**The training-data contradiction — a model that learns a lie.** A model trains on data carrying an undocumented contradiction — two definitions of "active," a feature computed two ways, a label set with a systematic bias — and it does exactly what it is designed to do: it learns the pattern, contradiction included. Nothing errors. The model deploys, performs plausibly in aggregate, and quietly makes a fraction of its decisions on a fault line no one flagged. Meridian's version: a suitability model trained on holdings where EM exposure was computed under mixed divisional definitions learns a relationship between "EM exposure" and "suitable" that is wrong for half its population, and the error is invisible until a specific client — a retail account misread as institutional — surfaces it. The failure is *silent, aggregate-passing, and baked into the weights*, which is the worst place for an error to live because retraining is the only fix.

**The RAG hallucination — an answer from stale or mis-scoped context.** A retrieval-augmented system — the pattern behind most enterprise copilots — retrieves documents or chunks and generates an answer grounded in them. When the retrieved context is stale, superseded, mis-scoped, or wrongly permissioned, the model grounds its answer in the wrong evidence and presents it with full fluency. Meridian's adviser copilot retrieves a product research note to answer a suitability question — but the note was superseded three months ago, and nothing in the vector index recorded that it had been. The answer is confident, well-written, and based on withdrawn guidance. This is the archetype the current chapters barely touch and Chapter 9 is added to address, because in 2026 the data most AI consumes is not tables but *documents and embeddings*, and none of the failure modes above are structural until you govern unstructured context too.

**The agent action — a decision taken on a mis-defined field.** The most consequential archetype, because the AI does not merely answer; it *acts*. An agent reads a field, reasons over it, and takes an action — approves, prices, routes, blocks — at machine speed, with no human between the reading and the consequence. When the field's meaning has drifted, the action is wrong and it is already done. Meridian's copilot, asked whether a portfolio is suitable, reads `em_exposure_pct` under the wrong divisional definition and tells an adviser a breach is within tolerance — and the adviser, trusting it, acts. The Okonkwo scene that opens Part II is exactly this archetype. As agents move from answering to acting — the trajectory of the epilogue — this becomes the dominant risk, because the loop that used to contain errors (a human reading a dashboard, raising a ticket) is gone.

Three shapes, one root. In each, the data was not *dirty* — it was complete, typed, fresh, and passing every check. What it lacked was *bound, machine-readable context*: the definition, the currency, the scope, the lawful basis that a human consumer would have supplied and a machine consumer cannot. The rest of the book is organised around producing exactly that context, and each of these three archetypes is defused by a specific later part — the training contradiction by binding and modelling (Parts II, IV), the RAG hallucination by governing unstructured context (Chapter 9), the agent action by evidence and certification at the point of consumption (Part V).

## Meet Meridian

To keep this concrete rather than abstract, the book follows one organisation through the whole arc of the problem and its cure. **Meridian Financial** is a mid-sized wealth and asset manager — regulated enough for mistakes to be expensive, large enough to have a sprawling estate, and ambitious enough to be putting AI into production right now. The full profile is in the dossier below; what matters at this point is that it has just placed two AI systems on the critical path — an adviser copilot answering relationship managers' questions about client portfolios, and a suitability-and-eligibility model heading for model-risk review — on top of a foundation built, like everyone's, for a human consumer.

Meridian is not special. Its situation is the situation of nearly every enterprise putting AI into production: good people, real data, genuine investment — and a foundation built for a human consumer now being asked to serve a machine one. Over the chapters ahead you will watch its copilot give a confident, wrong answer about a client's portfolio because a definition drifted two pipelines upstream, and watch its model get paused by a committee not because the data was bad but because the data could not produce evidence about itself. Both failures are versions of the 60% statistic, made specific. Both are entirely preventable, and the rest of the book is, in effect, the story of Meridian preventing them.

### The Meridian dossier

Because Meridian recurs in every chapter, here is one reference spread you can flip back to — the estate, the systems, the AI, and the people.

**The firm.** A mid-sized wealth and asset manager, UK-headquartered and regulated by the FCA and PRA. Two divisions: **Retail** (mass-affluent clients, standardised products) and **Institutional** (private and corporate clients, bespoke mandates). The two divisions classify some things differently — most consequentially, whether frontier markets count as "emerging" — and that divergence is the seed of the book's running error.

**The estate (the sediment of Chapter 1, in one firm).** A settlement **mainframe** nobody migrates; a decade-old cloud **data warehouse** of curated marts; an **Iceberg lakehouse** half the domains have moved to; source systems including two **custodians**, a **CRM**, a **billing** system, and market-data feeds; a **feature store** and a **vector index** feeding the AI layer; governance tooling (a **catalogue**, a **quality platform**) that this book will reframe from source-of-truth to consumer.

**The two AI systems on the critical path.**
- The **adviser copilot** — a retrieval-augmented assistant that answers relationship managers' questions about client portfolios (suitability, exposure, performance). It reads both structured holdings data and unstructured research notes. It is the *RAG* and *agent-action* archetypes waiting to happen.
- The **suitability-and-eligibility model** — a supervised model scoring whether a portfolio suits a client and whether a client qualifies for a product, heading toward model-risk review. It is the *training-contradiction* archetype waiting to happen.

**The cast.**
- **Maya Osei** — data product owner for the Client domain; owns `client_holdings` and the suitability context. The book's protagonist; her funded authority (Chapter 21) is why the discipline holds.
- **Tom** — head of the data-quality programme; award-winning, and about to discover his instrument was built for the wrong consumer (Chapter 20).
- **Helena** — ran the £400k enterprise-catalogue programme; about to be asked a question her catalogue cannot answer (Chapters 12, 20).
- **Nadia** — model-validation lead; the person in the room when the awkward, joined question of quality-and-lineage lands (Chapter 20).

Keep this spread bookmarked. Every technical move in the book lands on one of these systems, and every human failure lands on one of these people.

## Why your current data-quality setup will not save you

The reflexive response to all of this is reassuring and wrong: *we have data quality tools, we run checks, we have a governance team.* These things cost real money and effort, and they are not nothing. But they are not enough for AI, for three reasons that map precisely onto the diagnosis of the previous chapters.

**They are reactive.** You find problems after they have spread. By the time someone notices, the model has been training on bad data for weeks, and — because of the compounding loop above — the damage is not a single wrong number but a model that has learned to produce wrong numbers. The quality regime built to catch errors in a dashboard is structurally too slow for a pipeline that learns.

**They are siloed.** Marketing's "active customer" is not Sales' "active customer" is not Finance's, and in many organisations those three definitions live in three different systems maintained by teams that never speak. This is the metadata drift of Chapter 3, viewed from the consumer's side: the model is learning from contradictions nobody ever agreed on. A Gartner survey of data-management leaders (third quarter of 2024) found that 63% of organisations either do not have, or are not sure they have, the right data management practices for AI — which is to say, most organisations cannot even confirm they are ready, let alone demonstrate it.

**They have no teeth.** Best practices exist; documentation exists; conventions exist. But when someone breaks an implicit agreement with a downstream team, nothing happens — no alert, no blocked deploy, no accountability. Policies without enforcement are suggestions, and suggestions do not survive the pressure of a quarterly release cycle. This is the passivity of Chapter 3's catalogue, restated as a quality problem: the rules describe what should be true; nothing makes them true.

Put the three together and you have the precise reason a conventional data-quality programme — even a good, award-winning one — does not produce AI-readiness. It catches the wrong things, too late, with no power to stop them. It is built around the assumption that a human will be there to act on what it finds. Remove the human, add a consumer that reads at machine speed, and the whole apparatus is revealed as necessary but radically insufficient.

### "Won't better models just read the context themselves?"

There is one objection that every AI-optimist stakeholder raises, and it deserves a straight answer here rather than a deferral to the epilogue, because it will be raised at the next steering committee: *frontier models keep getting better at inference — won't the agent simply read the wiki, the catalogue, and the dbt docs and work the meaning out for itself?*

Return to Chapter 3's drift table. By month 18, an agent that reads *everything* reads one production value and four stale, mutually contradictory descriptions of it: the dbt doc says one thing, the two BI measures a second and third, the glossary a fourth, and the steward who knew the truth has left. A more capable model reads all five faster and phrases the answer more fluently. It does not read them more *truly*, because nothing in the five records which one is authoritative or current. Inference over contradiction is not the cure for drift; it is the mechanism that turns drift into a confident, well-written, wrong answer. More reading is not more truth.

What the agent needs is not more capability but a single bound, current, authoritative source of meaning to read *from* — which is exactly what the rest of this book builds, and exactly what no amount of model progress supplies on its own. The better the model, in fact, the more urgent the problem: a fluent reasoner over contradictory context produces more convincing errors, not fewer.

## The reframe: an evidence problem, not a quality problem

This points to the single most important idea in the book — the one the whole constructive half is built to deliver.

**AI-readiness is not a data quality problem. It is an evidence problem.**

The data that fails an AI initiative is frequently not dirty. It is complete, accurate, fresh, and statistically well-behaved; it passes every quality gate it is asked to pass. What it cannot do is produce, on demand, defensible answers to the questions an AI workload — and the regulator, the auditor, and the model-risk committee standing behind it — actually ask. Not the evidence engineers consume (schema, row counts, freshness), but evidence across what Chapter 18 will formalise as the *six surfaces*: **ownership** (who is accountable, by name, with a budget), **meaning** (what a field denotes, and under whose definition), **quality** (fitness for the specific use, not an average score), **lineage** (where a value came from, traced end to end), **privacy** (the lawful basis for using it in an automated decision), and **consumption** (who depends on it downstream, including which AI systems). What does this field mean, and under whose definition? Who is accountable for it, with a budget and a phone number? What did its quality look like on the specific day a model trained on it? What is the lawful basis for using it in an automated decision? A data product is AI-ready when, for any of its consumers — human, machine, or regulator — it can answer those six questions with evidence rather than a shrug. (These are the same six surfaces the regulatory table below arrives at from the opposite direction, and that Part V turns into a working certification.)

This reframe matters because it redirects all the effort. Most enterprises respond to AI-readiness anxiety by investing harder in data quality — cleaner pipelines, more tests, another catalogue — and then discover that the same effort which produces a clean monthly report does not produce a model that survives its first audit. The problem was never the cleanliness of the data. It was that the data could not speak about itself. We will develop this idea in full in Part V, where it becomes a practical certification model. For now, hold it as the hinge: the goal is not cleaner data. It is data that carries enough bound, active, machine-readable context to produce evidence about itself the moment any consumer asks.

## The four moves

If the diagnosis of Part I is correct — data separated from its meaning, the separation now unaffordable because the consumer changed, the remedy data that carries its own context and can prove things about itself — then the shape of the solution follows directly. Four moves, one per remaining part of the book, each answering one part of the diagnosis. (The map table below gives the mechanics; the paragraphs here give the logic.)

**Bind.** Attach context to data so the two cannot drift apart — the direct answer to Chapter 3's mechanism. Metadata stops living in a separate system and becomes a property of the data, created, versioned, and deployed with it. The unit is the **Data Capsule**: data plus its structural, semantic, quality, policy, provenance, and contractual context, co-versioned into one deployable thing. **Part II.**

**Automate.** Hand-binding does not scale past the first few datasets, and bound-but-passive context still cannot keep pace with a fast-moving estate. So the second move makes the context *run* — declarative, tested, event-driven — moving governance from the documentation layer to the control plane: a schema change halts a pipeline before bad data spreads; a sensitive field is classified and policy applied before a steward meets. This is **metadata as code** and **active metadata**, the answer to Chapter 3's latency problem. **Part III.**

**Package.** Bound, active context still has to be *exposed* in a form an AI consumer can use. A **context data product** wraps the dataset with the ontology, semantic layer, knowledge graph, and usage intelligence that let an agent reason over it rather than guess — Chapter 2's mental-model correction made real. **Part IV.**

**Prove.** Finally, the organisation has to *demonstrate* a product is safe for a given AI use — to itself, its auditors, its regulators. This is **AI-readiness as evidence and certification**, which turns "we think the data is fine" into "here is the evidence, on demand, for this use." It answers the evidence reframe directly. **Part V.**

Bind, automate, package, prove — the spine of everything that follows. Every chapter from here says which move it is making and why.

## The ladder you are climbing

Because four abstract moves can feel like a lot to hold at once, it helps to have a concrete sense of progress — a way to locate where a given data product is today and where the next move takes it. Throughout the book we use a five-rung maturity ladder for exactly this.

- **Level 0 — Flat.** A data product exists but carries only technical metadata: schema, ownership tags, an SLA. This is where most organisations are, and it is where most of Meridian started.
- **Level 1 — Defined.** A business glossary is linked to the product; its key terms have agreed definitions. Meaning exists and is attached.
- **Level 2 — Modelled.** An ontology exists for at least one domain; entities and their relationships are formally documented.
- **Level 3 — Connected.** A knowledge graph ties entities, lineage, and confidence together, and at least one AI consumer is actively using it.
- **Level 4 — Intelligent.** Usage intelligence, dynamic confidence scoring, and semantic query APIs are operating; context standards are enforced across domains.

Most organisations, honestly assessed, sit at Level 0 or 1. That is not a failure; it is a starting point. The work of this book is to take one data product from Level 0 to Level 3 deliberately and provably, and then to show how that scales. Part II moves a product from Level 0 to Level 2 by binding and modelling its context. Part III makes that context active and enforced — hardening those levels rather than adding a rung; there is no ontology, graph, or live consumer yet, but everything Level 3 will run on is now in place. Part IV packages it — ontology, semantic layer, knowledge graph — and lights up the first AI consumer, crossing into Level 3. Part V proves it and points the way to Level 4. The ladder is how you will know the moves are working: not by faith, but by a product that can demonstrably do more than it could a chapter ago.

### The map: four moves against five levels

The four moves and the five-level ladder are two views of the same journey, and it helps to hold them as a single map. Read the moves along the top and the levels up the side, and each part of the book is the cell where a move lifts a product to the next level:

<!-- FIG 4.1: the map — four moves (Bind, Automate, Package, Prove) across five levels (Flat → Defined → Modelled → Connected → Intelligent); shade the cell where each part lifts a product to the next rung. This is the book's navigation aid; it deserves a diagram, not just the table below. -->

| Move | Part | Does what | Takes a product to |
|------|------|-----------|--------------------|
| **Bind** | II | Attaches context to data as one co-versioned unit (the capsule); models it | Level 0 → 2 (Flat → Modelled) |
| **Automate** | III | Makes the bound context versioned, tested, event-driven, active | Hardens Levels 1–2 (durable, enforced, active) |
| **Package** | IV | Assembles context into a product an agent can reason over | Level 2 → 3 (Modelled → Connected) |
| **Prove** | V | Turns packaged context into certified, audit-surviving evidence | Level 3 → 4 (Connected → Intelligent) |

Every chapter from here announces, in its header, which move it is making and which rungs it is crossing — so that at any point you can locate a technique on this map and know what it is *for*. Bind, automate, package, prove; Flat, Defined, Modelled, Connected, Intelligent. Two axes, one climb.

## Ten questions, asked now

Part V builds a full certification model, but its heart is ten questions an auditor (or an AI workload) will eventually ask of any data product feeding a high-stakes AI use. Score your own most important product against them *today* — 0 (no answer), 1 (answer in a week), 2 (answer from a query, in the room), for a maximum of 20 — because the gap you find now is the backlog the rest of the book clears, and the exercise gives the argument a personal stake.

1. Who *owns* this product — a named person with budget and decision rights?
2. Is each field's *meaning* queryable as code, under a stated definition?
3. Is its *quality* contracted to your specific use, not just an average score?
4. Can you *trace* one value end-to-end, with business meaning intact?
5. Can you retrieve its quality and lineage *as they stood on a past date*?
6. Is the *lawful basis* for each use enforced at runtime, not filed in a policy?
7. Do you know every *consumer* downstream, including AI ones?
8. Will a *breaking change* notify those consumers before it ships?
9. Is the product *certified* for this use, by someone who signed off?
10. Would degradation be *detected and acted on* automatically, or discovered in an incident?

A mature product scores in the high teens out of 20. Most, honestly assessed, score below ten. That number is not a verdict; it is a starting line, and every part of this book raises it. (These ten questions are a fast preview; they map onto the six evidence surfaces of Chapter 18, which Appendix D turns into the fuller scored instrument and Chapter 20 runs in practice.)

## The deadline that removes the option to wait

There is one more reason this is urgent now rather than eventually, and it is not competitive pressure, though that is real. It is regulation, and it has a date on it. The EU AI Act's obligations phase in over several years — prohibitions from February 2025, general-purpose-model rules from August 2025 — but 2 August 2026 is the date that matters here: it is when the high-risk obligations apply, for AI systems in credit scoring, eligibility, hiring, and medical diagnostics, the exact territory Meridian's suitability model lives in. Those systems must meet comprehensive obligations around data governance, transparency, traceability, and human oversight, with penalties scaling into the tens of millions of euros or a percentage of global turnover. The UK takes a different route, with the FCA, PRA, and ICO leaning on existing rulebooks rather than a single statute, and other jurisdictions are converging on the same questions from their own directions. The instruments look nothing alike. Read enough of them and the same handful of questions keep surfacing, and they are questions about your data layer, not your models: where did the data come from, was it fit for the purpose, can you trace what the model did, who is accountable, and how would you know if any of it had quietly shifted.

The instruments differ in letter and agree in substance, which is easiest to see in one table. Different jurisdictions, different legal forms — and the same handful of data-layer questions underneath.

| Instrument | Jurisdiction / scope | Data-layer questions it forces | Timing |
|------------|----------------------|-------------------------------|--------|
| **EU AI Act** | EU; high-risk AI systems | Training-data governance, traceability, transparency, human oversight, record-keeping | Main obligations phasing from Aug 2026 |
| **FCA / PRA** (UK) | UK financial services | Governance, accountability (SM&CR), model risk, data quality for regulated decisions | In force, applied via existing rulebooks |
| **BCBS 239** | Global systemically important banks | Risk-data aggregation, lineage, accuracy, completeness | In force (long-standing) |
| **OSFI E-23** | Canada; federally regulated financial institutions | Model risk management across the model lifecycle, incl. data | Effective 2027 |
| **MAS FEAT / guidance** | Singapore financial sector | Fairness, ethics, accountability, transparency of data and models | In force (guidance) |
| **NIST AI RMF** | US; voluntary framework | Data provenance, validity, documentation, measurement | Published; widely referenced |
| **ISO/IEC 42001** | International; AI management systems | Auditable AI governance, incl. data controls | Published; certification emerging |

Read down the "questions" column and the point of the whole book is visible in a regulatory register: *where did the data come from, was it fit for the purpose, can you trace what the model did, who is accountable, and how would you know if it had shifted.* These are, near enough, the six evidence surfaces of Chapter 18 — ownership, meaning, quality, lineage, privacy, and consumption — arriving as law. What the regulators have added is a deadline. The evidence these regimes demand either exists when you are asked for it or it does not, because — as Meridian's model-risk committee will demonstrate in Chapter 7 — you cannot assemble it retrospectively. Gartner's observation that most governance initiatives fail for want of a crisis catalyst has, for AI-readiness, found its catalyst. The crisis is dated, and the date is close.

## Turn the page

Part I set out to do one thing: convince you that the old model breaks, and that it breaks for a reason more specific than "data is hard." The reason is that data engineering spent decades separating data from its meaning, relied on human judgement to bridge the gap, and has now handed the data to a consumer that supplies no judgement of its own — at the precise moment when regulators have begun demanding evidence that the gap is closed. A better lake will not fix it. A better catalogue will not fix it. More data-quality tooling will not fix it, because the problem is not quality; it is evidence, and evidence requires context that is bound to the data, kept active, packaged for consumption, and provable on demand.

That is the diagnosis. The remaining four parts are the cure, and they begin with the first move. Meridian's adviser copilot is about to give a confident, wrong answer about the Okonkwo family portfolio — and the fix starts not with the model, not with the pipeline, but with changing the fundamental unit of design. Turn the page to Part II, and we begin to *Bind*.

## Further reading

- Gartner, *Lack of AI-Ready Data Puts AI Projects at Risk* (press release, 26 February 2025) — the source of the "abandon 60% of AI projects unsupported by AI-ready data through 2026" prediction and the Q3-2024 survey behind the 63% figure.
- The EU AI Act (Regulation (EU) 2024/1689) — primary text; see Annex III for the high-risk categories and Article 113 for the phased application dates, including 2 August 2026 for high-risk systems.
- NIST *AI Risk Management Framework* (AI RMF 1.0, 2023) — the US voluntary framework on data provenance, validity, and documentation referenced in the regulatory table.
- ISO/IEC 42001:2023, *AI management systems* — the standard behind the emerging AI-governance certification market.

> **Chapter summary.** AI fails at the data layer — Gartner expects organisations to abandon roughly 60% of AI projects unsupported by AI-ready data through 2026 — because ML learns from the contradictions, duplicates, and stale records that human analysts used to silently filter; the new consumer has none of the old one's forgiveness, and a more capable model reasoning over contradictory context produces more confident errors, not fewer. Conventional data quality cannot save it: it is reactive, siloed, and toothless. The decisive reframe is that **AI-readiness is an evidence problem, not a quality problem** — data is ready when it can produce defensible evidence, on demand, across six surfaces: ownership, meaning, quality, lineage, privacy, and consumption. The cure is four moves — **Bind** (Part II), **Automate** (Part III), **Package** (Part IV), **Prove** (Part V) — tracked against a five-level maturity ladder from Flat to Intelligent. The EU AI Act's high-risk obligations, applying 2 August 2026, remove the option to wait, because the required evidence cannot be assembled retrospectively.

> **Try this.** Take the data product feeding your most important AI initiative and ask the Part V questions of it now, before a regulator does: what does each field mean and under whose definition, who is accountable by name, what was its quality on the day the model trained, and what is the lawful basis for each use. The questions you cannot answer today are the ones that will pause your model — and they are the backlog the rest of this book clears.
