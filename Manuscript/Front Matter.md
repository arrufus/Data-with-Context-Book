# Data with Context

### Engineering Data for the AI Era

*Bind · Automate · Package · Prove*

---

## Preface

This book began as an argument I kept having in different rooms. In one room, an AI team insisted their models were failing because the data was dirty. In another, a data-quality team showed me an award-winning programme and could not understand why the models it fed kept getting paused. In a third, a governance lead described a £400,000 catalogue that could not answer the one question the AI programme actually needed answered. Each room had a piece of the same problem, and none had the whole. It is this: for thirty years we built extraordinary machinery for *moving* data and almost none for keeping its *meaning* attached — and we got away with it because a human being was always in the loop to supply the missing meaning. Then a consumer arrived that cannot: the AI model that trains on our contradictions, and the AI agent that reads our columns at machine speed and acts on whatever it finds. The slack is gone, and the gap between data and its context — chronic and survivable for decades — has become the binding constraint on whether AI delivers any value at all.

The book's answer is a discipline in four moves. **Bind** context to data so the two cannot drift apart. **Automate** that binding so the platform maintains it rather than a heroic human. **Package** the bound, active context into products an AI agent can actually reason over. And **Prove** — turn that context into evidence a data product can produce about itself, on demand, for any consumer, human, machine, or regulator. Bind, automate, package, prove: the spine of everything that follows.

**Who this is for.** Practitioners, mostly — the data, analytics, and platform engineers and the architects who build these systems, and the data leaders who fund and organise them. It assumes you know what a table, a pipeline, and a model are, and it does not assume you have solved governance, because almost nobody has. If you are building AI on top of enterprise data in 2026 and you have felt the ground move under a project that "should have worked," this book is a map of why, and a plan.

**A word on Meridian.** Rather than scatter disconnected examples, the book follows one fictional organisation — **Meridian Financial**, a mid-sized wealth and asset manager — through the whole arc of the problem and its cure. You will watch its adviser copilot fail in a way no quality dashboard predicted, and its people — Maya, Tom, Helena, Nadia — meet questions their well-run functions could not answer. Meridian is financial services because that is where regulation makes the stakes sharpest and the evidence demands hardest, but the discipline is not finance-specific, and the book translates it to hospitals, retailers, and SaaS firms where it matters. Meridian is not special; the firms reading this will recognise their own estate in it.

**On the tools.** Every tool named in this book — Iceberg, dbt, OpenLineage, Snowflake, DataHub, Neo4j, and the rest — is *illustrative*. The patterns are the point; the logos are interchangeable, and the book says so repeatedly because vendor-neutrality is not a disclaimer here but a design principle: a discipline built on open standards and durable patterns outlives any tool you happen to run today.

This book is an argument responsive to evidence, not a creed. The epilogue names what would show it wrong. Read it that way — as a wager that a consumer has arrived that cannot supply its own context, and that binding meaning to data is therefore no longer optional. If you build with AI, you are about to find out whether that wager is right, on your own estate, whether or not you meant to test it.

---

## How to read this book

The book is built to be read cover to cover, but it is long enough that two shorter paths are worth naming.

**The executive / decision-maker path.** If you set strategy and fund the work but will not personally write a capsule spec, read: **Chapter 2** (why the lake mental-model is the constraint), **Chapter 4** (the failure mode, the four moves, the maturity ladder, and the regulatory clock), **Chapter 15** (what a context data product is and why a governed table is not enough), **Chapters 18–19** (the evidence reframe and certification), and **Chapters 21–22** (why this needs a new operating model, and what it costs and returns). That is the argument, the stakes, the assurance model, and the business case — roughly a third of the book, and enough to decide and to fund.

**The builder / engineer path.** If you will implement this, read **Parts II–IV in full** (Chapters 5–17, including Chapter 9 on RAG and unstructured data) plus the **appendices**, which turn the argument into artefacts you can lift: the capsule spec (A), the repository layout (B), the certification checklist (C), and the assessment instrument (D). Parts I and V give you the *why* and the *proof*; Parts II–IV and the appendices give you the *how*.

**Everyone** should read the **Epilogue**, because it is short and it is about where your job is going.

Each chapter opens with a *"Where you are"* note locating it on the four-move spine and the maturity ladder, and closes with a *"Chapter summary"* and a *"Try this"* — a single concrete exercise you can run on your own estate to feel the gap the chapter describes. The "Try this" boxes, read in sequence, are themselves a rough diagnostic of where your organisation stands.

---

## Conventions

- **Code and specs** are illustrative and abbreviated for readability; they show the *shape* of a real artefact, not a copy-paste-ready production file. YAML is the book's default for specs (schema, contracts, capsules, certification) because it is what the tooling ecosystem has converged on; the choice is not load-bearing.
- **Tool neutrality.** Named products are examples of a pattern, never endorsements. Where the book says "Snowflake-style" or "Feast-like," read "the pattern these tools exemplify."
- **The maturity ladder** (Level 0 Flat → 4 Intelligent) is the book-wide progress measure; the **certification model** (Level 0 Not-assessed → 5 Continuously-assured) in Chapter 19 is a finer-grained, per-product-per-use instrument. They are related but distinct, and the book flags which it means.
- **Currency and figures.** Monetary illustrations use pounds sterling because Meridian is UK-based; treat all figures as round, illustrative orders of magnitude, not benchmarks.
- **"He/she/they"** for the recurring cast follows the characters introduced in the Meridian dossier (Chapter 4); everyone else is "they."

---

*The four moves begin in Part II. Part I is the diagnosis — why the old model breaks, and why a better lake, a better catalogue, or more data-quality tooling will not fix it.*
