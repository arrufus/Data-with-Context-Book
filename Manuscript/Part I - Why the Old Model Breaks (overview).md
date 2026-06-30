# Part I — Why the Old Model Breaks

*The on-ramp. Four chapters that diagnose the disease before the rest of the book prescribes the cure.*

Part I makes one argument: the foundation most data platforms stand on was built for a consumer — the human analyst — who is no longer the one that matters most, and AI is the consumer that exposes the flaw. It is deliberately diagnostic; the discipline itself begins in Part II.

| Ch | Title | Move in the argument | Core source |
|----|-------|----------------------|-------------|
| 1 | How Data Engineering Got Here, and Where It Broke | History as setup: each era separated data from meaning further | *Evolution of Data Engineering* Pt 1 & 2 |
| 2 | The Lake Was a Mental-Model Error | The most consequential inherited assumption: hoard now, mean later | *Stop Building Data Lakes* + *Lakehouse Is a Swamp* |
| 3 | How Data and Metadata Drifted Apart | The mechanism: metadata kept in a *separate system* guarantees drift; the catalogue shares the disease | *Data Capsules* Pt 1 (historical norm) + *Active Metadata* |
| 4 | The New Failure Mode: AI Dies at the Data Layer | The stakes + the map: evidence not quality; the four moves; the maturity ladder | *60% of AI Projects Will Fail* + *Evidence Problem* (opening) |

**The arc.** Chapter 1 frames the whole history of data engineering as a triumph of *moving* data and a quiet failure to keep its *meaning* attached — a failure humans silently absorbed. Chapter 2 shows the lake hardening that failure into a philosophy. Chapter 3 names the precise mechanism (metadata living in a system of its own) and explains why the catalogue, the obvious cure, fails on *latency*. Chapter 4 reveals what removes the slack — AI, a consumer with no judgement and no memory — reframes the problem as **evidence, not quality**, and lays out the book's spine.

**The hinge (Ch 4).** This is the connective chapter the blueprint flagged as key new writing. It introduces:
- **The running case study — Meridian Financial** (wealth manager; adviser copilot + suitability model), handing off into Part II.
- **The four moves** — **Bind** (II) → **Automate** (III) → **Package** (IV) → **Prove** (V).
- **The maturity ladder** — Level 0 (Flat) → 4 (Intelligent).
- **The deadline** — EU AI Act, 2 August 2026, as the crisis catalyst.

**Continuity with Part II.** Ch 4 introduces Meridian and foreshadows two failures it deliberately does *not* tell in full — the Okonkwo suitability error (told fresh in Ch 5) and the model-risk pause (told fresh in Ch 7) — so Part II's vignettes still land. Voice, "Where you are" callouts, and chapter-end summary + "Try this" boxes match Part II.

**Status:** First draft. Next candidates: a voice/expansion pass across Parts I–II, or forward to **Part III — Automate**.
