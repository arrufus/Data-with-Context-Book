# Appendix D — Maturity Self-Assessment

*Two instruments you can run on your own estate: the per-asset maturity ladder (0–4) that measures the four-move climb, and the six-surface evidence instrument from Chapter 19 that measures whether a product can testify. Together they turn "AI-ready" from a feeling into a pair of numbers you can track over a rollout.*

---

## D.1 The per-asset maturity ladder

Score each data product on the book-wide ladder. Each level *includes* the levels below it.

| Level | Label | A product is here when… | Reached by |
|:---:|-------|-------------------------|-----------|
| 0 | **Flat** | it carries only technical metadata: schema, ownership tag, SLA | (starting point) |
| 1 | **Defined** | a glossary is linked; its key terms have agreed, bound definitions | Bind (Ch 5) |
| 2 | **Modelled** | an ontology exists for its domain; entities & relationships documented | Bind + model (Ch 8, 16) |
| 3 | **Connected** | a knowledge graph ties entities, lineage & confidence; ≥1 AI consumer live | Automate + Package (Ch 11, 15) |
| 4 | **Intelligent** | usage intelligence, dynamic confidence scoring, semantic APIs; context standards enforced | Prove + scale (Ch 18, 20) |

**How to use it.** Score every tier-1 product, tabulate the distribution, and track it quarterly. The number that matters most is the *falling count of Level 0–1 products* — that is the rollout working. Targets by quarter (from Chapter 13): pilot cohort to L3 in Q1; 20% of tier-1 at L2+ by Q2; 50% by Q3; all *new* products born at L1+ by Q4.

**Interpretation bands (portfolio view):**
- *Most of the estate at 0–1* — normal starting point; you have a discovery-and-trust problem and are not AI-ready. Pick one high-value product and run the 90-day playbook (Ch 15).
- *A pilot cohort at 3, the rest at 0–1* — healthy early rollout. The risk now is stalling (Ch 13's "stall" case); make domain owners accountable and fund the next domains from the pilot's value summary.
- *Tier-1 broadly at 2–3, apex at 4* — mature. Focus shifts to enforcing context standards on new products and to certification (Appendix C).

---

## D.2 The six-surface evidence instrument

The maturity ladder measures *capability*; the evidence instrument measures whether a product can *testify* — the Chapter 17/19 test. Score each surface **0 / 1 / 2**:

- **0 — reconstruction only:** you *could* produce the answer with time and people. Fails the retrieval-time test.
- **1 — monitoring:** you have the *current* state on a dashboard, but not the point-in-time past.
- **2 — bound evidence:** the answer is a queryable, versioned, point-in-time property of the product, retrievable within budget for any date.

| # | Surface | The question | Budget | Score (0/1/2) |
|:-:|---------|--------------|:------:|:---:|
| 1 | Ownership | Who is accountable, with authority (a name, not a team)? | seconds | ▢ |
| 2 | Metadata | What does each field mean, as queryable code? | seconds | ▢ |
| 3 | Quality | Is quality contracted to *this* use, tested at every layer? | seconds | ▢ |
| 4 | Lineage | Can you trace one value end-to-end, *on a past date*? | minutes | ▢ |
| 5 | Privacy | Is lawful basis / consent enforced at runtime, per feature? | seconds | ▢ |
| 6 | Consumption | Do you know every consumer downstream, incl. AI ones? | seconds | ▢ |
| | | | **Total /12** | ▢ |

**Interpretation bands (per product):**
- **0–4 — not AI-ready.** The product cannot testify; a green dashboard is hiding this. It will pause a model or fail an audit. This is where most products start (Meridian's `client_holdings` began at 3).
- **5–8 — partial.** Some surfaces bound, others reconstruction-only. Identify the 0-scoring surfaces first — they are the ones an auditor or agent will hit.
- **9–11 — strong.** Nearly all surfaces bound; close the remaining gap before certifying for a high-stakes AI use.
- **12 — can testify.** Every surface returns within budget for any date. This is the bar for Level-4 certification of a client-facing AI use.

**The link to certification.** The six surfaces map onto the five certification dimensions of Appendix C: ownership + governance readiness; metadata + meaning readiness; quality + quality readiness; lineage + operational readiness; privacy + governance readiness; consumption + consumption readiness. A product scoring 12 here has, in effect, assembled most of the evidence a Level-4 certification requires.

---

## D.3 A worked example (Meridian `client_holdings`)

| Surface | Before | After | What closed the gap |
|---------|:---:|:---:|---------------------|
| Ownership | 1 | 2 | named owner with decision rights + pager (Ch 20) |
| Metadata | 0 | 2 | semantics bound as code; glossary refs (Ch 5) |
| Quality | 1 | 2 | contracted, per-use, gating tests (Ch 6, 10) |
| Lineage | 0 | 2 | OpenLineage + bitemporal graph (Ch 12) |
| Privacy | 0 | 2 | lawful-basis manifest, runtime masking (Ch 7, 10) |
| Consumption | 1 | 2 | registered, contracted consumers (Ch 13) |
| **Total** | **3** | **12** | |

Three to twelve is the whole book on one product: not cleaner data, but data that can now explain and defend itself.

---

## D.4 How to run the assessment

1. **Pick the products that feed AI** — start where the stakes are, not with the whole estate.
2. **Score both instruments** — maturity (capability) and evidence (testimony). They answer different questions; a product can be capable (L3) and still score poorly on evidence if its lineage is a diagram not a graph.
3. **Attack the 0s first** — a single 0-scoring evidence surface is what pauses a model; it dominates the average.
4. **Re-score quarterly** — the trend is the point. Falling Level-0/1 counts and rising evidence totals are what you show leadership to fund the next domain (Ch 21).
5. **Be honest.** The instrument is worthless if scored generously; a "1 we call a 2" is exactly the monitoring-mistaken-for-evidence error that pauses models in Chapter 19.
