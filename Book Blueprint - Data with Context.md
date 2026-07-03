# Data with Context — Book Blueprint

*Working blueprint for turning the Articles & Blogs corpus into a single, coherent book.*
*Spine chosen: **Data with Context**. Primary reader: **practitioners** (data, analytics, and platform engineers; architects).*

---

## 1. The premise

In the AI era, data without bound, machine-readable context is a liability, not an asset. Clean data that an AI agent cannot interpret is worse than useless — it produces confident, wrong answers at machine speed. The discipline this book teaches is **shipping context *with* data**: binding meaning, quality, policy, lineage, and contracts to the data itself, automating that binding, packaging it for consumers, and proving it with evidence.

This is the thread that already runs through the strongest, most original pieces in the corpus (Data Capsules, Metadata as Code, Active Metadata, Context Data Products, AI-readiness as evidence). Everything else — the history, the lake-to-product shift, the changing craft — is the on-ramp that earns the reader's trust before the original payload lands.

## 2. The unifying thesis (the connective tissue the articles lack)

The articles each argue one idea well, but stand alone. The book's job is to fuse them into **one lifecycle**. Proposed frame: **Context-Bound Data**, delivered in four moves.

| Move | Question it answers | Core chapters | Source IP |
|------|--------------------|--------------|-----------|
| **1. Bind** | How do we attach context to data so the two can't drift apart? | Data Capsules | Capsules Pt 1–3 |
| **2. Automate** | How do we keep context correct, versioned, and reactive at scale? | Metadata as Code, Active Metadata | MaC Pt 1–3, Active Metadata, Metadata Integrator |
| **3. Package** | How do we expose context so humans *and* AI agents can consume it? | Context Data Products | 90-Day Playbook, Stop Building Data Lakes |
| **4. Prove** | How do we show a product is safe and ready for a given AI use? | AI-Readiness Certification / Evidence | Evidence Problem, Certification, AI-Readiness series |

A one-line spine for the reader: **Bind → Automate → Package → Prove.** Every part should restate where the reader is on that arc.

## 3. The maturity spine (the book's progress bar)

Promote the 90-Day Playbook's five-level ladder into the book's organising backbone. Each part moves the reader up a rung, so progress feels concrete.

- **Level 0 — Flat:** data products carry only technical metadata (schema, owner, SLA).
- **Level 1 — Defined:** business glossary linked; key terms agreed.
- **Level 2 — Modelled:** an ontology exists for at least one domain.
- **Level 3 — Connected:** a knowledge graph ties entities, lineage, and confidence; one AI consumer is live.
- **Level 4 — Intelligent:** usage intelligence, dynamic confidence scoring, and semantic query APIs operate; context standards enforced.

The book takes a team from Level 0 to Level 3 for one domain, then shows how to scale to Level 4.

## 4. The running case study (the strongest missing element)

The articles use scattered vignettes (a UK bank's fraud model, an insurer's pricing model, an asset manager's suitability case). Unify them into **one fictional organisation carried through every chapter** so each technique compounds visibly.

Proposal: **"Meridian Financial,"** a mid-size financial-services firm, and **one domain** — *client suitability / holdings* (Client, Portfolio, Holding, Asset Class, Risk Profile). Thread it: model the capsule (Bind) → put its metadata under CI/CD with quality gates (Automate) → publish it as a context data product with a semantic API (Package) → certify it for a relationship-manager copilot and survive a model-risk review (Prove). FIBO can supply the starting vocabulary. The recurring characters from the AI-Readiness series (Maya, Tom, Helena, Nadia) can become Meridian colleagues, turning five separate vignettes into one narrative.

## 5. Positioning

- **What it is:** the practitioner's field manual for the context discipline — patterns, specs, repo layouts, CI/CD gates, certification checklists.
- **What it is not:** a tool tutorial or a vendor survey. Tools (Delta/Iceberg/Hudi, dbt, OpenLineage, DataHub, Collibra/Atlan/Purview) appear as *illustrative*, swappable implementations of durable patterns.
- **Why it's differentiated:** most data-engineering books stop at "build reliable pipelines." This one starts where they stop and argues the next foundation: data that can explain and defend itself to AI consumers.
- **Estimated length:** ~85,000–95,000 words; ~20 chapters across 5 parts + appendices, ~4,000 words/chapter.

---

## 6. Chapter-by-chapter outline

Legend — **Source:** existing article(s) to adapt. **Gap:** new material to write. **Words:** rough target.

### Part I — Why the old model breaks *(on-ramp; compress hard — least original)*

**Ch 1 — How data engineering got here, and where it broke**
Goal: a fast, opinionated history that lands on the AI inflection — relational → warehouse → big data → cloud/real-time → lakehouse → the context wall. Don't be a survey; every era ends with the unsolved problem it handed forward.
Source: *Evolution of Data Engineering Pt 1 & 2* (trim ~60%). Words: ~4,500.

**Ch 2 — The lake was a mental-model error**
Goal: reframe the reader from "store everything, extract meaning later" to "meaning comes from intent." Data-lake/lakehouse failure rates; lake vs product table.
Source: *Stop Building Data Lakes*, *Lakehouse Is a Swamp*. Words: ~4,000.

**Ch 3 — How data and metadata drifted apart**
Goal: the root-cause chapter. Why every platform generation re-split data from its context (RDBMS exports, Hive/Glue, ETL/BI silos), and why that's now unaffordable.
Source: historical-norm sections of *Data Capsules Pt 1*, *Active Metadata*. Words: ~3,500.

**Ch 4 — The new failure mode: AI dies at the data layer**
Goal: the stakes. 60% of AI projects abandoned for non-AI-ready data; introduce the thesis that this is an *evidence* problem, not a quality problem. Open the Meridian story.
Source: *60% of AI Projects Will Fail*, opening of *Evidence Problem*. Words: ~3,500.
**Gap:** the thesis-fusion section (Bind→Automate→Package→Prove) lands here as the book's spine.

### Part II — Bind: the Data Capsule *(Level 0 → 1)*

**Ch 5 — Data Capsules: data and metadata as one atomic unit**
Goal: define the capsule (payload + structural, semantic, quality, policy, provenance, contract metadata); strong binding, co-versioning, deploy-as-one-unit. The canonical YAML capsule spec.
Source: *Data Capsules Pt 1*. Words: ~4,500.

**Ch 6 — Capsules on the lakehouse: Delta, Iceberg, Hudi**
Goal: the practical home — table formats as the binding point; per-format patterns; hot vs cold metadata; managing capsules through CI/CD, not console edits.
Source: *Implementing Data Capsules on a Lakehouse* (Pt 2). Words: ~5,000.

**Ch 7 — Capsules for AI/ML and data in motion**
Goal: extend the discipline to training sets, feature stores, and streaming — provenance, reproducibility, model governance, schema registries.
Source: *Data Capsules for AI/ML and Real-Time Systems* (Pt 3). Words: ~4,500.

**Ch 8 — Modelling for context**
Goal: structure is the prerequisite for meaning. Medallion modelling choices (Bronze raw, Silver 3NF/Data Vault, Gold dimensional/semantic); modelling as the UI for data products and AI agents.
Source: *Data Modelling for a Medallion Architecture*, *The Backbone of Value*. Words: ~4,500.

### Part III — Automate: Metadata as Code & Active Metadata *(Level 1 → 2)*

**Ch 9 — Metadata as Code: GitOps for governance**
Goal: declarative, version-controlled, tested, CI/CD-integrated metadata; the four principles (declarative, validated, event-driven, policy-as-code).
Source: *Metadata as Code Pt 1*. Words: ~4,000.

**Ch 10 — The toolchain that makes it real**
Goal: open standards as the substrate — OpenLineage, dbt artifacts, AsyncAPI, Great Expectations; platform-native hooks (Snowflake tags, Unity Catalog, BigQuery policy tags). Patterns, not endorsements.
Source: *Metadata as Code Pt 2*. Words: ~4,500.

**Ch 11 — Active Metadata: the platform's nervous system**
Goal: metadata that *acts* — schema-change detection, impact analysis, quality gates, auto-classification, SLA alerts. Catalogue as index over the source of truth, not the source itself.
Source: *Active Metadata*, *Metadata as Code Pt 3* (activation patterns). Words: ~4,000.

**Ch 12 — Standards-based metadata infrastructure**
Goal: making metadata interoperable across a sprawling stack — OpenLineage (runtime) + W3C DCAT/PROV (semantics) + a knowledge graph (storage). The "where did this come from / what breaks if I change it" platform.
Source: *Designing an Enterprise Metadata Integrator* / *Building Standards-Based Metadata Infrastructure* (consolidate the two). Words: ~4,000.

**Ch 13 — Rollout: capsules as source, platforms as consumers**
Goal: the adoption roadmap — incremental, pilot-first; how Collibra/Atlan/Alation/Purview become consumers of capsule metadata, not competing truths. Phase 0 pain audit → phased expansion.
Source: *Data Capsules Pt 4 (Condensed Outline)*, *Metadata as Code Pt 3* (roadmap). Words: ~4,000.

### Part IV — Package: Context Data Products *(Level 2 → 3)*

**Ch 14 — From data products to context data products**
Goal: the product contract (owner, SLA, versioned schema) plus the context layers (ontology, semantic layer, knowledge graph, usage intelligence). Why AI consumers need *guided* access, not open access.
Source: *Stop Building Data Lakes* (product framing) + context-product framing across the AI-readiness pieces. Words: ~4,000.
**Gap:** an explicit five-layer anatomy diagram + spec.

**Ch 15 — The 90-day playbook: shipping one context data product**
Goal: the hands-on spine of Part IV — domain selection, minimum viable ontology, bootstrapping a knowledge graph from existing metadata, assembly, measurement. Run it on Meridian's suitability domain.
Source: *Building Your First Context Data Product (90-Day Playbook)*. Words: ~5,000.

**Ch 16 — Ontologies, semantic layers, and knowledge graphs in practice**
Goal: the modelling craft for meaning — entities/relationships/business rules; FIBO as accelerator; lightweight tooling (OWL/RDF vs JSON-LD/YAML); when to introduce a graph DB.
Source: MVO + bootstrapping sections of the *90-Day Playbook*; *Data Mesh* semantic threads. Words: ~4,000.

### Part V — Prove: AI-Readiness as Evidence *(Level 3 → 4)*

**Ch 17 — AI-readiness is an evidence problem, not a quality problem**
Goal: the reframe — a product is AI-ready when it can produce defensible evidence on demand. The six surfaces of evidence (ownership, metadata/meaning, quality, lineage, privacy, consumption).
Source: *AI-Readiness Is an Evidence Problem*. Words: ~4,000.

**Ch 18 — AI-ready is a certification, not a state**
Goal: readiness is conditional (ready for *which* use, *which* audience, *which* controls). The five certification dimensions (meaning, quality, governance, consumption, operational readiness).
Source: *AI-Ready Data Is Not a State. It Is a Certification.* Words: ~4,000.

**Ch 19 — The assessment in practice**
Goal: the field chapter, vignette-driven (the corpus's best writing). Quality ≠ readiness; catalogue ≠ metadata; the auditor's questions (EU AI Act, FCA/PRA, OSFI E-23, MAS); claim vs receipt (lineage); ownership vs stewardship.
Source: *AI-Readiness Articles 1–5*. Words: ~5,500.

**Ch 20 — Why this needs a new operating model**
Goal: the discipline fails without ownership and governance maturity. Why data mesh stalls (operating model, not architecture); how platform teams become the bottleneck; federated computational governance as the gating capability.
Source: *Data Mesh Promised a Revolution*, *Platform Teams Are the New Bottleneck*. Words: ~4,500.

**Ch 21 — ROI and the two disciplines** *(optional / could fold into Ch 20)*
Goal: measuring value honestly; distinguishing *data management FOR AI* from *AI IN data management* so budgets target the right problem.
Source: *ROI Reality*, *FOR AI vs IN Data Management*. Words: ~3,500.

**Epilogue — Where the craft goes**
Goal: the horizon — agentic data engineering, the AI-copilot stack, the death of hand-built ETL; close on the long arc (the *Five Orders of AI* as a far-future coda).
Source: *Death of Traditional ETL*, *New Data Engineer's Stack*, *Five Orders of AI*. Words: ~3,500.

### Appendices (practitioner payload — mostly new)
- **A. The canonical Data Capsule / data-contract spec** (full annotated YAML). *Gap, seeded from Capsules Pt 1 + ODCS.*
- **B. A Metadata-as-Code repo layout** (folders, CI gates, PR workflow). *Gap.*
- **C. The AI-readiness certification checklist** (5 dimensions × evidence items). *Gap, seeded from Certification + Evidence pieces.*
- **D. The maturity self-assessment** (Level 0–4 scoring sheet). *Source: 90-Day Playbook.*
- **E. Glossary** (capsule, active metadata, context data product, certification, evidence surface…). *Gap.*

---

## 7. Gaps to write fresh (what makes it a *book*, not a stitched anthology)

1. **The thesis-fusion chapter (Ch 4).** The single most important new writing: weld four parallel ideas into one lifecycle so the reader carries one mental model, not four.
2. **The running Meridian case study.** Continuity artifacts in every chapter — the same domain, advancing one maturity level at a time.
3. **Hands-on appendices.** Runnable specs, repo layouts, checklists. The chosen audience will judge the book on these.
4. **Cross-references and a maturity progress bar.** Each chapter opens with "where you are on Bind→Automate→Package→Prove and which Level you're moving to."
5. **A consistent voice pass.** The AI-Readiness series is vignette-driven and excellent; the Evolution/2-Day pieces are survey-toned and citation-heavy. Normalise toward the former.
6. **Diagrams.** Capsule anatomy, five-layer context product, evidence surfaces, the standards-based metadata platform, the maturity ladder.

## 8. Cleanup before drafting

Consolidate near-duplicates to one canonical source per idea so the mapping above stays clean (details in the inventory below). Net: ~36 files → ~30 distinct pieces + 1 idea bank, with ~5 true duplicates to archive.

## 9. Suggested sequencing

1. Approve/adjust this structure and the Bind→Automate→Package→Prove spine.
2. Lock the running case study (org, domain, characters).
3. Archive duplicates; designate the canonical file per chapter.
4. Draft Part II first (Bind) — it's the most original and most complete in the source material; momentum + proof of the format.
5. Write the Ch 4 thesis-fusion chapter early (it governs everything downstream), even though it sits in Part I.
6. Build the appendices in parallel with the parts that seed them.

---

## 10. Dedup + file inventory

**Status key:** *Distinct* = unique source · *DUP* = redundant copy to archive · *Idea bank* = backlog material.

| # | File | Cluster | Maps to | Status |
|---|------|---------|---------|--------|
| 1 | The Evolution of Data Engineering — Part 1 (incl. Pt 2 inline) | Foundations | Ch 1 | Distinct |
| 2 | 2-Day Data Architecture Training Course | Foundations | Ch 1, 8, 12 (reference) | Distinct |
| 3 | The Five Orders of AI — From Lightbulb to Cosmic Mind | Futurist | Epilogue | Distinct (outlier) |
| 4 | Stop Building Data Lakes. Start Building Data Products. | Data products | Ch 2, 14 | Distinct |
| 5 | Why Your Data Lakehouse Is Actually Just a Data Swamp… | Data products | Ch 2 | Distinct |
| 6 | Data Capsules — Why Data and Metadata Must Ship Together | Bind (Pt 1) | Ch 3, 5 | Distinct (canonical) |
| 7 | Data Capsules — Ship Data and Metadata as a Single Unit. | Bind (Pt 1) | Ch 5 | **DUP of #6** |
| 8 | Implementing Data Capsules on a Lakehouse – Delta, Iceberg, Hudi | Bind (Pt 2) | Ch 6 | Distinct (canonical) |
| 9 | implementing-data-capsules-on-lakehouse.md | Bind (Pt 2) | Ch 6 | **DUP of #8** |
| 10 | Data Capsules for AI ML and Real-Time Systems | Bind (Pt 3) | Ch 7 | Distinct |
| 11 | Data Capsules Part 4 – Condensed Outline | Bind/rollout (Pt 4) | Ch 13 | Distinct |
| 12 | Data Modelling for a Medallion Architecture | Modelling | Ch 8 | Distinct |
| 13 | The Backbone of Value — Why Data Modelling Matters… | Modelling | Ch 8 | Distinct |
| 14 | Metadata as Code — Part 1 | Automate | Ch 9 | Distinct |
| 15 | Metadata as Code — Part 2 | Automate | Ch 10 | Distinct |
| 16 | Metadata as Code — Part 3 | Automate | Ch 11, 13 | Distinct (canonical) |
| 17 | metadata-as-code-part-3-revised.md | Automate | Ch 11, 13 | **DUP/variant of #16** |
| 18 | Active Metadata — The Quiet Revolution… | Automate | Ch 3, 11 | Distinct |
| 19 | Designing an Enterprise Metadata Integrator | Automate | Ch 12 | Distinct (canonical pair) |
| 20 | Building Standards-Based Metadata Infrastructure… | Automate | Ch 12 | **Near-DUP of #19** — consolidate |
| 21 | Building Your First Context Data Product, A 90-Day Playbook | Package | Ch 15, 16; maturity spine | Distinct (keystone) |
| 22 | AI-Ready Data Is Not a State. It Is a Certification. | Prove | Ch 18 | Distinct |
| 23 | AI-Readiness Is an Evidence Problem… (root) | Prove | Ch 4, 17 | Distinct (canonical) |
| 24 | …/AI-Readiness Is an Evidence Problem… (in subfolder) | Prove | Ch 17 | **DUP of #23** |
| 25 | AI-Readiness Article 1 — Your DQ Programme Is Making AI Worse | Prove | Ch 19 | Distinct |
| 26 | AI-Readiness Article 2 — Your Catalogue Is Not Your Metadata | Prove | Ch 19 | Distinct |
| 27 | AI-Readiness Article 3 — What Your Auditors Will Ask by 2027 | Prove | Ch 19 | Distinct |
| 28 | AI-Readiness Article 4 — Your Data Is Fine. Prove It. | Prove | Ch 19 | Distinct |
| 29 | AI-Readiness Article 5 — Nobody Owns Your Data Product | Prove | Ch 19, 20 | Distinct |
| 30 | Data Mesh Promised a Revolution… 82%… | Operating model | Ch 20 | Distinct |
| 31 | Platform Teams Are Becoming the New Bottleneck… | Operating model | Ch 20 | Distinct |
| 32 | 60% of AI Projects Will Fail This Year… | AI-readiness hook | Ch 4 | Distinct |
| 33 | The ROI Reality of AI in Data Management… | AI value | Ch 21 | Distinct |
| 34 | Data Management FOR AI vs. AI IN Data Management… | AI framing | Ch 21 | Distinct |
| 35 | The Death of Traditional ETL — AI Agents Rewriting DE | Craft | Epilogue | Distinct |
| 36 | The New Data Engineer's Stack — Airflow + dbt + AI Copilot | Craft | Epilogue | Distinct |
| 37 | data-engineer-stack-airflow-dbt-ai-copilot.md | Idea bank | (backlog) | **Idea bank**, not a dup — mine for future chapters |

*Note: #37 is a "Writing Ideas + Draft Package" containing additional blog-post concepts — keep it as a backlog source, not a chapter.*

---

*Next: confirm the structure and the running case study, then draft Part II (Bind) as the first vertical slice.*
