# Part II — Bind

*The first move of the discipline: attach context to data so the two cannot drift apart.*

Part II argues that the right unit of design is not the dataset but the **Data Capsule** — data plus its context, bound and co-versioned. It runs the Meridian Financial case study — a wealth manager, the client-suitability domain, an adviser copilot in production — through five chapters, taking one product from Level 0 (Flat) to Level 2 (Modelled) across the structured *and* unstructured halves of the estate.

| Ch | Title | What it does | Maturity |
|----|-------|--------------|----------|
| 5 | Data Capsules: Data and Metadata as One Atomic Unit | Defines the unit and binds the drifting definition | Level 0 → 1 |
| 6 | Capsules on the Lakehouse: Delta, Iceberg, Hudi | Gives the capsule a home on real table formats | Consolidate L1 |
| 7 | Capsules for AI/ML and Data in Motion | Raises the stakes — training sets, feature stores, streaming | L1 pays off |
| 8 | Modelling for Context | Models the structure meaning can bind to | L1 → 2 |
| 9 | Capsules for Unstructured Data and RAG | Extends *Bind* to documents and embeddings — the half the agent reads most | Binds L1 for unstructured |

**The Meridian thread.** A suitability error — the Okonkwo account — opens Chapter 5: good data that could not explain itself produced a confident wrong answer because a definition (`em_exposure_pct`, division-dependent) had silently drifted. Each chapter advances the fix: bind the definition (5), deploy it on the table format (6), extend it to the model that must survive a model-risk audit (7), model the structure that makes the definition stable (8), and carry the discipline into documents and the vector index the copilot retrieves from (9). Part II ends with *Bind* complete across the whole estate, setting up **Part III — Automate**, where hand-binding becomes platform-enforced metadata-as-code.

**Recurring cast.** Maya (data product owner, client domain) is introduced here; Tom, Helena, and Nadia appear from Part III onward by role, and their full stories land in Part V (*Prove*).
