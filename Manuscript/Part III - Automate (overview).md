# Part III — Automate

*The second move: keep bound context correct, current, and acting — at scale, by the platform rather than by hand.*

Part II bound context to one dataset by hand. Part III makes the binding a property of the platform: declarative, versioned, tested, event-driven, and enforced. It continues the Meridian case study, opening with the 7 a.m. dashboards-read-zero incident (a custodian column rename cascading through fourteen tables) and ending with a governed, active, AI-consumable client domain.

| Ch | Title | What it adds | Maturity | Core source |
|----|-------|-------------|----------|-------------|
| 9 | Metadata as Code: GitOps for Governance | The principle: metadata as declarative, versioned, CI/CD-reconciled code; four principles | Make L1 repeatable | *Metadata as Code* Pt 1 |
| 10 | The Toolchain That Makes It Real | The stack: OpenLineage, dbt artefacts, AsyncAPI, Great Expectations + platform-native hooks | L1 → 2 | *Metadata as Code* Pt 2 |
| 11 | Active Metadata: The Platform's Nervous System | Metadata moves to the control plane; auto-stop, trust scoring, live agent context | L2 → 3 | *Active Metadata* + *MaC* Pt 3 |
| 12 | Standards-Based Metadata Infrastructure | The shared substrate: OpenLineage + W3C DCAT/PROV + SHACL + knowledge graph | Enables L3 across tools | *Building Standards-Based Metadata Infrastructure* |
| 13 | Rollout: Capsules as Source, Platforms as Consumers | The adoption discipline + the source-of-truth rule; phased L0→L3 | Operationalises L0→3 | *Data Capsules* Pt 4 + *MaC* Pt 3 roadmap |

**The arc.** Ch 9 reframes metadata from byproduct to product, governed like code (the four principles: declarative, validated, event-driven, policy-as-code). Ch 10 assembles the open-standard + platform-native toolchain that captures lineage, contracts, and quality automatically. Ch 11 is the turn: metadata stops describing and starts *acting* — stopping broken pipelines, scoring trust live, and feeding the adviser copilot certified context before it answers (the threshold of Level 3 and a trustworthy AI consumer). Ch 12 builds the standards-based graph that lets active metadata act across a sprawling, multi-tool estate. Ch 13 settles the political question — **capsules are the source of truth; governance platforms are consumers** — and lays out the phased, anti-pattern-aware rollout from Level 0 to Level 3.

**Continuity with Parts I–II.** The 7 a.m. incident is the Automate-side counterpart to Part II's Okonkwo error. Maya (client-domain product owner) carries through; **Helena** (catalogue programme) and **Tom** (data-quality programme) are introduced by role here and set up for their Part V vignettes. Meridian is established as Iceberg-primary with a Snowflake footprint, so platform-native examples stay consistent. The Bind→Automate→Package→Prove spine and the Level 0–4 ladder are referenced throughout; Ch 13 explicitly maps its four phases to the ladder.

**Handoff.** Part III ends *Bind* + *Automate*: context attached and kept active. But an AI agent needs context *packaged* — ontology, semantic layer, knowledge graph, usage intelligence — into a product it can reason over. That is **Part IV — Package**.

**Status:** First draft. Chapters ~3,200–3,800 words. Next candidates: Part IV (Package), or a voice/expansion pass across Parts I–III.
