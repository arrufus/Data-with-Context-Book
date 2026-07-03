# Expansion Recommendations — Outlines of Additional Content

*Recommendations only. No draft files have been changed.*

## 1. The gap, quantified

| | Words |
|---|---|
| Current manuscript (21 chapters + epilogue + part overviews) | ~55,500 |
| Typical data-engineering book (reviewer benchmark) | 120,000–130,000 |
| Gap to close | ~65,000–70,000 |

Chapters currently average ~2,500 words; the blueprint itself targeted 4,000–5,500. The prose is strong and argument-complete — the thinness is **depth per chapter**, not missing argument. The reviewer's "light and scant" impression comes from four specific absences:

1. **Almost no runnable artefacts.** One capsule YAML (Ch 5), a handful of short snippets. Practitioner books of this genre are judged on specs, DDL, pipelines, repo layouts, and checklists — the blueprint's own positioning ("field manual… patterns, specs, repo layouts, CI/CD gates, certification checklists").
2. **One case study, one industry.** Meridian is excellent but every example is wealth management. Readers in retail, healthcare, SaaS, manufacturing, and public sector need at least sketch-level translation.
3. **No appendices yet.** A–E are planned but unwritten — they are the "practitioner payload" the blueprint says the audience will judge the book on.
4. **A visible topical gap for 2026: GenAI/RAG data engineering.** The book readies data for "AI agents" but never covers unstructured data, chunking, embeddings, vector stores, or retrieval pipelines — the workloads most readers are actually building. One new chapter fixes this.

## 2. Recommended expansion budget

| Component | Added words |
|---|---|
| Part I (Ch 1–4), ~2,200 each | ~8,800 |
| Part II (Ch 5–8), ~2,300 each | ~9,200 |
| **New Ch 8A — Capsules for Unstructured Data and RAG** | ~4,500 |
| Part III (Ch 9–13), ~2,200 each | ~11,000 |
| Part IV (Ch 14–16), ~2,500 each | ~7,500 |
| Part V (Ch 17–21), ~2,300 each | ~11,500 |
| Epilogue | ~1,000 |
| Appendices A–E (currently unwritten) | ~12,000 |
| Front matter (preface, how-to-read-this-book, audience map) | ~2,000 |
| **Total added** | **~67,500** |
| **Projected final length** | **~123,000** |

## 3. Cross-cutting additions (apply to every chapter)

- **"Meridian artefact" box per chapter** — the actual deliverable Meridian produced at that stage (a contract file, a CI config, an ontology fragment, a certification record). Turns narrative into field manual. (~300–500 words each; counted in per-chapter budgets below.)
- **"Beyond Meridian" sidebar per part** — the same pattern in one non-financial setting (hospital, e-commerce, manufacturer) to widen the audience.
- **Diagrams** — the blueprint's gap #6 is still open: capsule anatomy, hot/cold metadata split, six-layer toolchain flow, metadata-integrator architecture, five-layer context product, six evidence surfaces, maturity ladder, certification lifecycle, operating-model triumvirate. Figure captions and in-text walk-throughs add real words and real value.
- **"Further reading / references" section per chapter** — the source articles were citation-rich; the book has almost no citations. Restores authority and adds ~100–150 words per chapter.
- **End-of-part checklists** — one page per part: "you are done with Bind when…" (feeds Appendix D).

---

## 4. Per-chapter outlines of additional content

### Part I — Why the Old Model Breaks

**Ch 1 — How Data Engineering Got Here** *(2,508 → ~4,700)*
- *The eras in the room today*: a section mapping each era to the systems readers still run (the mainframe, the Netezza appliance, the Hadoop cluster in run-off, the half-migrated lakehouse) — sediment as an architecture-assessment lens.
- *A worked genealogy of one metric*: trace "active customer" through warehouse → Hadoop → modern stack, showing concretely where its meaning fragmented at each hop. Sets up the Okonkwo pattern early.
- *The modern-data-stack integration crisis*, expanded: name the seams (ingest/transform/orchestrate/catalogue), with a diagram of where context is dropped at each seam.
- *Timeline figure + era table* (era, constraint solved, problem handed forward, surviving artefacts).
- *What didn't happen*: brief treatment of attempts that tried to keep meaning attached (MDM, EII, semantic web 1.0) and why each stalled — pre-empts "haven't we tried this?"

**Ch 2 — The Lake Was a Mental-Model Error** *(2,845 → ~5,000)*
- *Two anonymised failure case studies* (one lake, one lakehouse migration-that-changed-nothing), each with a cost line — the chapter asserts the 85% figure but never shows a failure in texture.
- *The economics section, quantified*: a worked TCO sketch — storage vs compute vs the human cost of discovery/reconciliation — supporting "storage is cheap, complexity is expensive."
- *The lake-to-product migration path*: expand "what fixing it looks like" into a staged sequence with an example first-contract for a real table, linking forward to Ch 5.
- *Objections and answers*: "we need raw data for data science," "products slow us down," "we already have a catalogue" — one honest paragraph each.
- *Meridian artefact*: the one-page product charter for `client_holdings` (owner, purpose, consumers, SLA) referenced in later chapters.

**Ch 3 — How Data and Metadata Drifted Apart** *(2,618 → ~4,600)*
- *A drift anatomy, step by step*: one column followed over 18 months across catalogue, dbt doc, BI model, and wiki — dated table of what each store said vs what production did. Makes "drift surface" measurable.
- *Why catalogues underdeliver — the vendor-era history*: brief, fair treatment of catalogue generations (Excel → wiki → enterprise catalogue → "active" catalogues), what each fixed and re-created.
- *The latency ledger*: typical staleness rates by metadata type (schema, lineage, ownership, glossary) with reasoning — gives the latency argument numbers.
- *Counter-examples where separation is fine*: scratch data, exploratory notebooks — sharpening the claim so it doesn't read as absolutist.
- *Diagram*: the three historical "pull-apart" moves (RDBMS export, Hive metastore, tool-private stores) as one figure.

**Ch 4 — The New Failure Mode** *(3,095 → ~5,300)*
- *Three failure archetypes, told fully*: (1) the training-data contradiction (learned bias), (2) the RAG hallucination from stale context, (3) the agent action on a mis-defined field. Currently the chapter asserts the failure mode; it should show all three shapes readers will recognise. This also plants the GenAI thread the new Ch 8A pays off.
- *The Meridian dossier*: a fuller introduction — org chart, systems estate diagram, the two AI systems' architectures, the cast (Maya, Tom, Helena, Nadia) in one reference spread readers can flip back to.
- *The regulatory landscape section, expanded into a table*: EU AI Act, FCA/PRA, OSFI E-23, MAS, NIST AI RMF, ISO 42001 — instrument, scope, the data-layer questions each asks, dates. (Feeds Ch 19.)
- *A self-assessment teaser*: the ten questions of the Part V audit, asked now, scored 0–2 each — gives the reader a personal stake and foreshadows Appendix D.
- *Diagram*: the four moves × five-level ladder as one map, referenced from every later chapter.

### Part II — Bind

**Ch 5 — Data Capsules** *(3,759 → ~6,000)*
- *The capsule spec vs open standards*: map the seven components onto **ODCS (Open Data Contract Standard)** and data-contract tooling — show the same capsule expressed as an ODCS document and state the book's position (capsule = ODCS-compatible superset). The blueprint promised ODCS in Appendix A; introducing it here makes the spec credible.
- *Versioning semantics*: what semver means for data (major = breaking schema/definition change, minor = additive, patch = correction), with a worked change history for `client_holdings` 3.2 → 4.1 and the consumer-notification rules each bump triggers.
- *Capsule lifecycle*: draft → review → published → deprecated → retired, with the gates at each transition.
- *A second, contrasting capsule example*: a small reference-data capsule (asset-class taxonomy) — showing the discipline scaled *down*, so readers don't infer capsules are only for big fact tables.
- *Anti-patterns*: the "documentation capsule" (YAML nobody enforces), the "kitchen-sink capsule," the "orphan capsule" (no consumer).
- *Diagram*: capsule anatomy (seven components), and the binding-vs-colocation figure.

**Ch 6 — Capsules on the Lakehouse** *(3,662 → ~5,700)*
- *Table maintenance as capsule operations*: compaction, snapshot expiry, vacuum, manifests — what each does to the metadata the capsule depends on, and the retention policy that keeps time travel long enough for audit (links to Ch 7's training snapshots).
- *Catalog choice section, expanded into a comparison*: Hive Metastore vs Glue vs Unity vs Nessie vs Polaris vs Iceberg REST — governance features, tag support, access control, multi-engine behaviour (one table, ~400 words of commentary).
- *Performance and cost of the discipline*: what column stats, constraints, and pre-commit validators cost at 50 TB; when to sample; how to keep CI table-deployment fast.
- *The CI pipeline itself*: the actual GitHub-Actions/pipeline YAML that validates a table spec, runs the quality suite, and promotes the snapshot — the chapter describes this flow but never shows it.
- *Migration case narrative*: Meridian's first wrap-in-place migration told as a two-week diary (what broke, the consumer that bypassed the table, how they deprecated raw-file access).
- *Streaming-adjacent note*: where the schema registry sits relative to the table format (bridges to Ch 7).

**Ch 7 — Capsules for AI/ML and Data in Motion** *(3,154 → ~5,300)*
- *A full training-data capsule spec*: the YAML for `client_suitability_training` — snapshot pin, feature list with lawful-basis manifest, label provenance block, distribution baselines. This is the artefact the Maya story turns on; the reader should see it.
- *The model-registry link, concretely*: an MLflow-style registration record referencing capsule versions; the query that answered the committee's question.
- *Feature-store worked example*: one feature group (e.g. `client_activity`) defined with policy tags, gating tests, and online/offline contract — in Feast-like syntax.
- *Streaming section, deepened*: schema-registry compatibility modes (backward/forward/full) and what each means for capsule evolution; exactly-once and late-data implications for quality gates; a Kafka topic-as-capsule spec.
- *Drift monitoring in practice*: the actual comparison job — training-capsule baselines vs live inference distributions, thresholds, and the alert-to-owner routing.
- *Meridian artefact*: the lawful-basis manifest (a 73-row excerpt reduced to 8 illustrative rows).

**Ch 8 — Modelling for Context** *(3,010 → ~5,300)*
- *A worked dimensional model for the suitability domain*: the actual star — `fact_suitability_assessment`, dims for Client, Portfolio, Instrument, Risk Profile — with DDL and grain statements. The chapter teaches choices but never shows a finished model.
- *A worked Data Vault fragment*: hub_client, link_client_portfolio, sat_client_risk — 20 lines of DDL plus the 3NF integration view over it, making the "hybrid" concrete.
- *SCD mechanics*: a Type-2 MERGE pattern on Iceberg/Delta with `effective_from/to`, and the point-in-time join the copilot uses — connects directly to Ch 7's temporal-correctness needs.
- *Survivorship worked example*: the CRM-vs-custodian domicile conflict resolved attribute-by-attribute in a survivorship table, signed by Maya (the "Try this" made real).
- *Wide-table design rules*: column budget, nesting, update cadence, the serving-table contract; when `Gold.Serving` diverges from `Gold.Canonical` and how drift between them is prevented.
- *Naming and conventions standard*: a half-page house style (feeds Appendix B).

**NEW — Ch 8A (or Part IV insert): Capsules for Unstructured Data and RAG** *(~4,500 new)*
The book's most consequential content gap. AI consumers in 2026 predominantly consume *documents and embeddings*, and none of the current chapters govern them.
- *Why unstructured data breaks every rule so far*: no schema, no grain, no obvious owner; the "document swamp" as the new data lake.
- *The document capsule*: source, version, effective date, classification, permitted uses, and chunk lineage bound to each document; what co-versioning means when the payload is a PDF.
- *Chunking and embedding as transformations with lineage*: chunk → embedding → index entry as OpenLineage-tracked steps; embedding-model version as part of the capsule (re-embedding = a version event).
- *The vector store as a governed serving layer*: metadata filters as policy enforcement (division, entitlement, retention) at retrieval time; why an un-filtered index is an uncontrolled consumer.
- *Freshness and revocation*: propagating a document correction or a right-to-erasure through chunks, embeddings, and caches — the hardest lineage problem in RAG.
- *Evaluation as quality gates*: retrieval-relevance and groundedness checks as the Great-Expectations equivalent for RAG.
- *Meridian thread*: the copilot's research-notes retrieval — a stale, superseded product note surfaces in an answer; fixed by document capsules with `superseded_by` and effective dating.

### Part III — Automate

**Ch 9 — Metadata as Code** *(2,612 → ~4,800)*
- *The repository layout*: the actual tree — `contracts/`, `table-specs/`, `tests/`, `policies/`, `glossary/`, `pipelines/` — with what lives where and why (seeds Appendix B).
- *A PR walkthrough, end to end*: the `em_exposure_pct` definition change as a pull request — the diff, the CI checks that fire, the reviewers CODEOWNERS pulls in, the consumer notification on merge. The chapter's central promise, shown once, concretely.
- *The CI pipeline stages*: lint → schema validation → contract compatibility → quality tests → impact analysis → deploy, with the actual pipeline config (~40 lines).
- *Reconciliation and drift correction*: what "GitOps for governance" does when the live estate diverges (detect, alert, auto-revert vs ticket) — the Terraform-plan analogy completed.
- *Adoption sequencing for a brownfield estate*: which metadata to codify first; templates and scaffolding (`new-capsule.sh` foreshadowed).
- *Honest costs*: review latency, YAML fatigue, the governance-team skill shift — and mitigations.

**Ch 10 — The Toolchain That Makes It Real** *(2,279 → ~4,500)*
- *An end-to-end trace of one change*: the custodian rename replayed through the assembled stack — which event fires where, in sequence, with the artefacts at each hop (the six-layer diagram, animated in prose).
- *Standards coverage table*: OpenLineage / dbt artefacts / AsyncAPI / Great Expectations × what each captures, where it runs, maturity, gaps.
- *What the standards don't cover*: business glossary exchange, policy language portability, BI-tool lineage — and the pragmatic bridges.
- *Platform matrix*: Snowflake, Databricks/Unity, BigQuery, Fabric/Purview, plus open-source (DataHub, OpenMetadata, Marquez) — native hooks for tags, masking, lineage, contracts (~500 words + table).
- *Testing the metadata itself*: unit tests for policies, contract tests between specs, smoke tests post-deploy.
- *Meridian artefact*: the stack diagram with named products in each slot, and the "swap rules" (what changing any one slot costs).

**Ch 11 — Active Metadata** *(2,485 → ~4,600)*
- *Architecture of the active layer*: event bus, rules engine, action executors, audit log — a reference architecture with a build-vs-buy discussion (DataHub Actions, warehouse-native, or custom on Kafka).
- *The four behaviours, each with its playbook*: for auto-stop, trust scoring, agent context, and auto-classification — trigger, decision logic, action, human-escalation path, and the failure mode of each (~250 words each).
- *Trust-score design*: the actual formula (freshness, quality pass rate, ownership currency, incident recency), thresholds, and how scores decay and recover — currently asserted, not specified.
- *Alert economics*: precision/recall of automated actions, alert-fatigue budgets, tiering actions by blast radius (observe → warn → quarantine → halt).
- *The agent-context API*: the request/response an agent makes before answering (a JSON example of certified-context retrieval) — the book's most important integration, currently described only in prose.
- *Runbook*: the first 30 days of turning on active metadata without flooding the team.

**Ch 12 — Standards-Based Metadata Infrastructure** *(2,252 → ~4,400)*
- *A worked DCAT + PROV record*: the actual Turtle/JSON-LD for `client_holdings` and one job run — readers meet SHACL but never see the data model it validates.
- *The API surface*: register, query lineage, impact, trust — endpoint sketches with example payloads; how agents and CI consume them.
- *Sizing and scaling notes*: triples per dataset, event volumes at Meridian scale, store options (Jena/Fuseki, GraphDB, Neptune) and the property-graph projection pattern.
- *Bitemporal modelling, shown*: the two-timeline record for one ownership change, and the SPARQL for "who owned this last March."
- *Integration inventory*: the adapters Meridian actually built (mainframe extract, BI-tool scraper) — the unglamorous 20% the standards miss.
- *Build sequence*: 90-day phasing for the integrator itself (capture first, graph second, SHACL third).

**Ch 13 — Rollout** *(2,437 → ~4,600)*
- *The business case, written out*: Phase 0's pain audit turned into a one-page cost model (incident hours × rate, audit-prep weeks, analyst search time) with Meridian's numbers — the funding artefact the chapter says exists but never shows.
- *RACI for the rollout*: producer teams, platform, governance, stewards, domain owners across the four phases.
- *The steward feedback loop, end to end*: a Collibra enrichment travelling back to Git as a PR — screens/steps, and the sync architecture (event vs nightly) with conflict rules.
- *A quarter-by-quarter rollout calendar*: pilot → domain 2–3 → division → enterprise, with the maturity-score distribution targets per quarter.
- *Change-management section*: training, office hours, the "capsule champion" pattern, and what to do with the team that refuses.
- *Two mini-case vignettes*: a rollout that stalled at Phase 2 (why) and one that scaled (what differed) — generalising beyond Meridian.

### Part IV — Package

**Ch 14 — From Data Products to Context Data Products** *(2,230 → ~4,800)*
- *The five-layer anatomy as a spec*: the blueprint explicitly flags this gap ("an explicit five-layer anatomy diagram + spec"). A one-page machine-readable product descriptor covering all five layers, plus the diagram.
- *The context-extended contract, shown*: Meridian's suitability contract with the semantic obligations section written out (definitions promised, confidence commitments, permitted/prohibited uses) — extends Ch 5's YAML.
- *The semantic API, concretely*: one request/response pair for the Adeyemi question — what the copilot sends, what an enriched answer (data + definitions + confidence + caveats) looks like on the wire.
- *Consumer tiers*: how the same product serves an agent, an analyst, and a regulatory report differently from the same layers.
- *What this costs and who staffs it*: the incremental effort over a plain data product, honestly.
- *Beyond finance sidebar*: the same five layers for a hospital's clinical-terminology product or a retailer's product catalogue.

**Ch 15 — The 90-Day Playbook** *(2,626 → ~5,000)*
- *The templates, included*: the day-15 domain-selection brief (filled in for Meridian), the SME interview template, the day-90 one-page value summary — each named as a deliverable but not shown.
- *A week-by-week Gantt/table* of the 90 days with owners and effort (this is the "dated plan you can run on Monday"; give it dates).
- *Staffing and budget*: the pilot team (product owner, engineer, analyst/modeller, part-time steward), day rates, infra cost ≈ near-zero — the number a sponsor asks for.
- *What went wrong at Meridian*: two honest setbacks inside the 90 days (an SME who wouldn't agree the EM rule; a week lost to graph-tool debate) and the recovery — the playbook currently reads frictionless.
- *The measurement section, instrumented*: how each of the four metrics is actually captured (query logs, ticket tags, eval harness for agent accuracy), with Meridian's before/after numbers.
- *Scaling to domain 2 and 3*: what is reusable (patterns, scaffolds, standard) vs per-domain (ontology, rules), and the "lightweight context standard" one-pager.

**Ch 16 — Ontologies, Semantic Layers, and Knowledge Graphs** *(2,323 → ~4,800)*
- *The MVO, written out*: Meridian's five entities in JSON-LD/YAML (~60 lines) with both load-bearing rules encoded — the chapter's central artefact, currently invisible.
- *A semantic-layer definition in code*: `em_exposure` in MetricFlow/LookML-style syntax, parameterised by division; plus the governance workflow for changing a metric definition.
- *Three graph queries that earn the graph*: suitability traversal, blast-radius, "which rule applied on date X" — in Cypher or SPARQL with plain-English readings.
- *FIBO in practice*: a concrete borrow — FIBO's instrument-classification pattern adapted to Meridian's taxonomy, shown side by side (~300 words).
- *Ontology change management*: versioning meaning, deprecating an entity, migrating consumers when a rule changes — the drift problem at the semantic level.
- *Common modelling mistakes gallery*: attribute-as-entity, relationship without cardinality, enum-as-free-text, rule-in-prose — each with the fix.

### Part V — Prove

**Ch 17 — AI-Readiness Is an Evidence Problem** *(1,901 → ~4,200)*
- *One evidence artefact per surface*: the actual query + response for each of the six surfaces at Meridian (ownership record, semantic lookup, quality-contract status, lineage receipt, lawful-basis answer, consumer register) — six boxed examples, the chapter's promise made tangible.
- *The evidence-on-demand test protocol*: a table of the ten auditor questions, the artefact that answers each, and the maximum acceptable retrieval time.
- *Mapping surfaces → regulations*: which surface satisfies which EU AI Act article / BCBS 239 principle / model-risk expectation (table; pairs with Ch 4's landscape).
- *Degrees of evidence*: reconstruction vs monitoring vs bound evidence — a maturity mini-ladder for each surface.
- *A counter-story*: an organisation that passed an audit with heroic reconstruction and what it cost — the honest alternative path.

**Ch 18 — AI-Ready Is a Certification, Not a State** *(2,155 → ~4,500)*
- *A complete certification record*: Meridian's suitability product certified at Level 4 for copilot retrieval — the full YAML/document (dimensions, evidence links, sign-offs, triggers, expiry). The book's capstone artefact; must be shown.
- *The recertification workflow*: the trigger taxonomy (quality failure, lineage break, contract change, regulation change, time), and what an automated downgrade looks like to consumers mid-flight.
- *The certifier's worksheet*: per-dimension scoring rubric with pass evidence (seeds Appendix C).
- *A second product for contrast*: the marketing next-best-action product certified only to Level 2 — showing proportionality in practice.
- *Certification economics*: cost per product per level; the portfolio view (how many products at which level is healthy).
- *Failure story*: a certification programme that became the gate the chapter warns about, and the redesign.

**Ch 19 — The Assessment in Practice** *(2,777 → ~4,900)*
- *Remediation runbooks per pair*: Tom's, Nadia's, and the churn-model teams' actual 90-day fix plans — the chapter diagnoses brilliantly but never prescribes (quality-as-code migration steps; the training-time join implementation; the ownership re-chartering).
- *The assessment instrument itself*: a scoring sheet — six surfaces × the three-question test × 0–2 scoring — with Meridian's before/after scores (seeds Appendix D).
- *The auditor's-eye vignette*: one scene written from Nadia's/the regulator's side of the table — what a reviewer looks for, what raises flags — the corpus's "What Your Auditors Will Ask by 2027" material, currently underused.
- *A fourth vignette (consumption surface)*: the unregistered-agent incident — an internal team pointing an LLM at a product nobody knew about — since privacy/consumption currently gets summary treatment while the other pairs get full stories.
- *Sector translations*: the six surfaces asked in a hospital and in a SaaS company (half a page each).

**Ch 20 — Why This Needs a New Operating Model** *(2,526 → ~4,700)*
- *The operating-model blueprint*: Meridian's target structure drawn out — domain product teams, platform team, governance function, the triumvirate per product — with role definitions (the data-product-manager job description, ~half page, is a gift to readers).
- *Funding models*: cost-centre vs product-funded vs chargeback; how Meridian funds Maya's roadmap; why annual project funding kills products.
- *Federated computational governance, made concrete*: which decisions are global (identity, classification taxonomy, contract standard) vs domain-local (definitions, rules) — a decision-rights table.
- *The transition plan*: 18 months from central to federated, with the two hardest handoffs (quality policing → contracts; catalogue curation → source-sync) told as Meridian episodes for Tom and Helena — completing their character arcs.
- *Team Topologies mapping*: stream-aligned domain teams, the platform as enabling/platform team, governance as complicated-subsystem — with the interaction modes.
- *Metrics of the operating model itself*: product NPS, contract-violation rate, escape-hatch usage, time-to-new-product.

**Ch 21 — ROI and the Two Disciplines** *(1,843 → ~4,100)*
- *A worked ROI model*: Meridian's numbers — discipline cost (roles, infra, ~£X) vs avoided cost (incident reduction, audit prep, the paused-model counterfactual, regulatory-penalty exposure) over three years, with the honest uncertainty ranges. The finance-director chapter needs a spreadsheet-shaped argument (pairs with Ch 13's business case).
- *Measurement framework*: leading/lagging indicator sets for the FOR-AI discipline (currently only IN-data-management gets metrics).
- *The cost of the discipline itself, itemised*: what capsules/automation/certification add to a team's run rate — the number sceptics ask for and the chapter avoids.
- *Two build-vs-buy analyses*: active-metadata layer and metadata integrator — licence vs build costs at three org sizes.
- *A CFO conversation vignette*: Maya defending the programme at budget season — dramatises the chapter the way Part V's other chapters dramatise theirs.

**Epilogue** *(1,577 → ~2,600)*
- *A day-in-the-life, 2028*: the orchestrator-engineer's morning — reviewing agent-proposed pipeline changes against contracts — making the role shift concrete.
- *A skills map*: what to learn/deprioritise for the transition (judgement, modelling, contracts up; hand-built ETL down).
- *A short "what would falsify this book" section* — intellectual honesty as a closing note.

---

## 5. Appendices (unwritten — highest-leverage ~12,000 words)

- **A. The canonical Data Capsule / data-contract spec** (~3,000): the full annotated YAML, every field documented, ODCS mapping table, JSON-Schema for validation.
- **B. Metadata-as-Code repo layout** (~2,000): full tree, CI pipeline configs, PR template, CODEOWNERS, scaffolding script.
- **C. AI-readiness certification checklist** (~2,500): 5 dimensions × evidence items × automatable-vs-attested, plus the blank certification record.
- **D. Maturity self-assessment** (~2,000): per-asset 0–4 scoring sheet + the six-surface assessment instrument from Ch 19, with interpretation bands.
- **E. Glossary** (~2,500): ~60 terms, each with a one-line definition and chapter reference.

## 6. Front matter (~2,000 words)

Preface (who this is for, the Meridian conceit, how the corpus became a book), a "how to read this book" map (fast path for executives: Ch 2, 4, 14, 17–18, 20–21; build path for engineers: Parts II–IV + appendices), and conventions (YAML style, tool-neutrality statement).

## 7. Suggested sequencing of the expansion work

1. **Appendices A–E** — highest value-per-word, and writing them will surface artefacts (spec fields, checklist items) the chapter expansions should reference.
2. **New Ch 8A (RAG/unstructured)** — closes the topical gap reviewers in 2026 will flag hardest.
3. **Artefact passes on Part II then Part IV** — the "field manual" chapters where missing code is most conspicuous.
4. **Part V evidence/certification artefacts** (Ch 17–18 boxes, Ch 19 runbooks).
5. **Part I and III depth passes**, diagrams, references, front matter last.
