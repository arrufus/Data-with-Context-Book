# Part I — Why the Old Model Breaks

Part I is the diagnosis. Before the book prescribes a cure, four chapters make a single argument: the foundation most data platforms stand on was built for a consumer — the human analyst — who is no longer the one that matters most, and AI is the consumer that exposes the flaw. This part is deliberately diagnostic. The discipline itself begins in Part II.

| Ch | Title | Move in the argument |
|----|-------|----------------------|
| 1 | How Data Engineering Got Here, and Where It Broke | History as setup: each era separated data from its meaning a little further |
| 2 | The Lake Was a Mental-Model Error | The most consequential inherited assumption — hoard now, find meaning later |
| 3 | How Data and Metadata Drifted Apart | The mechanism: metadata kept in a system of its own guarantees drift, and the catalogue shares the disease |
| 4 | The New Failure Mode: AI Dies at the Data Layer | The stakes and the map — evidence not quality; the four moves; the maturity ladder |

**The arc.** Chapter 1 frames the whole history of data engineering as a triumph of *moving* data and a quiet failure to keep its *meaning* attached — a failure humans silently absorbed. Chapter 2 shows the lake hardening that failure into a philosophy. Chapter 3 names the precise mechanism — metadata living in a system of its own — and explains why the catalogue, the obvious cure, fails on *latency*. Chapter 4 reveals what removes the slack — AI, a consumer with no judgement and no memory — reframes the problem as **evidence, not quality**, and lays out the book's spine.

**The hinge is Chapter 4.** It introduces the three things the rest of the book runs on:

- **The running case study — Meridian Financial**, a wealth manager whose adviser copilot and suitability model are both about to fail in instructive ways.
- **The four moves** — **Bind** (Part II) → **Automate** (Part III) → **Package** (Part IV) → **Prove** (Part V).
- **The maturity ladder** — Level 0 (Flat) to Level 4 (Intelligent) — and **the deadline**: the EU AI Act's high-risk obligations, applying 2 August 2026.

**Into Part II.** Chapter 4 introduces Meridian and foreshadows two failures it deliberately does *not* tell in full — the Okonkwo suitability error (told fresh in Chapter 5) and the model-risk pause (told fresh in Chapter 7) — so Part II's vignettes still land at full force. The "Where you are" callouts, chapter summaries, and "Try this" exercises run consistently from here to the end of the book.
