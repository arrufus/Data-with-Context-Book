# Part II — Bind

*The first move of the discipline: attach context to data so the two cannot drift apart.*

Part II argues that the right unit of design is not the dataset but the **Data Capsule** — data plus its context, bound and co-versioned. It runs the Meridian Financial case study (a wealth manager, client-suitability domain, an adviser copilot in production) through four chapters.

| Ch | Title | Move | Maturity | Core source |
|----|-------|------|----------|-------------|
| 5 | Data Capsules: Data and Metadata as One Atomic Unit | Define the unit | Level 0 → 1 | *Data Capsules Pt 1* |
| 6 | Capsules on the Lakehouse: Delta, Iceberg, Hudi | Give it a home | Consolidate L1 | *Implementing Data Capsules on a Lakehouse* |
| 7 | Capsules for AI/ML and Data in Motion | Raise the stakes | L1 pays off | *Data Capsules for AI/ML and Real-Time Systems* |
| 8 | Modelling for Context | Structure meaning can bind to | L1 → 2 | *Medallion Modelling* + *The Backbone of Value* |

**The Meridian thread.** A suitability error (the Okonkwo account) opens Chapter 5 — good data that could not explain itself produced a confident wrong answer because a definition (`em_exposure_pct`, division-dependent) had silently drifted. Each chapter advances the fix: bind the definition (5), deploy it on the table format (6), extend it to the model that survives a model-risk audit (7), and model the structure that makes the definition stable in the first place (8). Part II ends with one governed capsule at Level 1–2, setting up **Part III — Automate**, where hand-binding becomes platform-enforced metadata-as-code.

**Recurring cast (introduced here, expanded in Part V):** Maya (data product owner, client domain). Tom, Helena, and Nadia join in the *Prove* chapters.

**Status:** First draft. Chapter targets ~4,000–5,000 words each. Next: review/voice pass, then either continue to Part III or draft the Ch 4 thesis-fusion chapter that governs the whole arc.
