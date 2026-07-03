# Part III — Automate

*The second move: keep bound context correct, current, and acting — at scale, by the platform rather than by hand.*

Part II bound context to one dataset by hand. Part III makes the binding a property of the platform: declarative, versioned, tested, event-driven, and enforced. It continues the Meridian case study, opening with the 7 a.m. dashboards-read-zero incident — a custodian column rename cascading through fourteen tables — and ending with a governed, active client domain that carries everything Level 3 (packaged in Part IV) will run on. Part III hardens Levels 1–2 rather than crossing a new rung; the crossing into Level 3 is Part IV's.

| Ch | Title | What it adds | Maturity |
|----|-------|--------------|----------|
| 10 | Metadata as Code: GitOps for Governance | Metadata as declarative, versioned, CI/CD-reconciled code; the four principles | Make L1–2 repeatable |
| 11 | The Toolchain That Makes It Real | OpenLineage, dbt artefacts, AsyncAPI, Great Expectations + platform-native hooks | Harden L1–2 |
| 12 | Active Metadata: The Platform's Nervous System | Metadata moves to the control plane; auto-stop, trust scoring, live agent context | Makes L2 active; readies L3 |
| 13 | Standards-Based Metadata Infrastructure | The shared substrate: OpenLineage + W3C DCAT/PROV + SHACL + a knowledge graph | Enables L3 across tools |
| 14 | Rollout: Capsules as Source, Platforms as Consumers | The adoption discipline and the source-of-truth rule; phased rollout | Operationalises the climb |

**The arc.** Chapter 10 reframes metadata from byproduct to product, governed like code (declarative, validated, event-driven, policy-as-code). Chapter 11 assembles the open-standard and platform-native toolchain that captures lineage, contracts, and quality automatically. Chapter 12 is the turn: metadata stops describing and starts *acting* — stopping broken pipelines, scoring trust live, and feeding the adviser copilot certified field-level context before it answers, which readies the estate for the Level 3 crossing that Part IV completes. Chapter 13 builds the standards-based graph that lets active metadata act across a sprawling, multi-tool estate. Chapter 14 settles the political question — **capsules are the source of truth; governance platforms are consumers** — and lays out the phased, anti-pattern-aware rollout.

**Continuity.** The 7 a.m. incident is the Automate-side counterpart to Part II's Okonkwo error. Maya carries through; **Helena** (catalogue programme) and **Tom** (data-quality programme) are introduced by role here and set up for their Part V vignettes. Meridian is established as Iceberg-primary with a Snowflake footprint, so platform-native examples stay consistent throughout.

**Handoff.** Part III ends *Bind* and *Automate*: context attached and kept active. But an AI agent needs context *packaged* — ontology, semantic layer, knowledge graph, usage intelligence — into a product it can reason over. That is **Part IV — Package**.
