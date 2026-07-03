# Review — *Data with Context* (Part I + book-level structure)

**Reviewed:** Book Blueprint, Front Matter, all five Part overviews, Chapters 1–4 in full depth, and the opening of Chapter 5 (continuity check only).
**Not yet reviewed:** Chapters 5–21 in depth, Epilogue, Appendices A–E. Suggested batching order for the rest: Part II (Ch 5–8A) → Part III (Ch 9–13) → Part IV (Ch 14–16) → Part V (Ch 17–21) → Epilogue → Appendices.
**Review date:** 2026-07-03

---

## 1. Executive Assessment

This is a complete-manuscript review scoped to Part I plus book-level structure. The draft is a practitioner-focused technical book with a genuinely coherent through-line — the Bind → Automate → Package → Prove spine, the Level 0–4 maturity ladder, and the Meridian case study give it connective tissue most stitched-from-articles books never achieve. Part I does its assigned job well: each chapter has a distinct role (history → philosophy → mechanism → stakes and map), and each one earns its place.

**Overall strength:** high. The argument is credible for the stated audience (data/analytics/platform engineers and architects), the technical claims are largely sound, and the writing is far above the AI-generated baseline — it has cadence, opinion, and lived texture ("The files go in beautifully. Nobody can find anything." is a line a human editor would keep).

**Main strengths:** the chapter-role discipline; the concrete artefacts (the product charter in Ch 2, the drift table in Ch 3, the regulatory table and ten questions in Ch 4); the honest handling of composite anecdotes; the "Try this" boxes, which are the best furniture in the book.

**Main weaknesses, in priority order:**

1. **Statistical framing and sourcing.** The 60% Gartner figure is misframed as an unconditional probability in Ch 4's opening; the 85% figure is attributed more narrowly (data lakes) than its source supports; several load-bearing numbers (40% catalogue staleness, one issue per ten tables) carry no source type. Experienced readers — the exact audience — will catch these.
2. **Framework-number drift.** The book's own central frameworks are inconsistent across files: "six evidence surfaces" are listed as five items in Ch 4, six different items in the Blueprint, and a *third* different six in the Part V overview. The move↔ladder mapping also disagrees between Ch 4's map table, Ch 4's prose, and the part overviews. For a book whose thesis is that meaning drifts when it lives in multiple unbound places, this is an irony the author will want to fix before a reviewer names it.
3. **Rhetorical tic density.** The prose reads human at paragraph level but accumulates recognisable patterns at chapter level: "it is worth…" (12+ instances in four chapters), "not because X but because Y," "Here is…" openings, and a balanced aphorism closing nearly every section. One aphorism per section lands; three reads as generated.
4. **Cross-file duplication.** The Preface and Ch 4 share near-verbatim passages; Ch 3 and Ch 5 share a near-identical "data lives here, metadata lives there" passage; the Okonkwo story is pre-told twice before its "fresh" telling in Ch 5.

**Does it read as human-authored?** Mostly yes — better than the vast majority of AI-assisted drafts. The risk is not any single sentence but pattern-recognition across chapters. A tic-thinning pass (Section 6) would take it the rest of the way.

**Single biggest improvement:** fix the framework-number inconsistencies and the 60%/85% framing. Those are the two things that can cost the book credibility with exactly the readers it targets.

## 2. Manuscript Structure Review

| Area | Assessment | Recommendation | Justification | Priority |
|------|------------|----------------|---------------|----------|
| Book thesis | Clear, stated identically in Blueprint, Preface, and Ch 4; the four-move spine genuinely fuses the source articles | None needed on substance | The connective tissue the Blueprint said was missing has been built | — |
| Part structure | Diagnosis (I) → three constructive moves (II–IV) → assurance (V) is sound and matches how practitioners adopt | Keep | Mirrors real adoption sequence: convince → build → operationalise → defend | — |
| Evidence-surface framework consistency | Ch 4 lists **five** items ("meaning, ownership, history, control, and consumption") while calling them "the six evidence surfaces of Chapter 17"; Blueprint says six (ownership, meaning, history/lineage, control, consumption, **operations**); Part V overview says six **different** ones (ownership, metadata, quality, lineage, privacy, consumption) | Pick one canonical list of six, state it in Ch 4, and reconcile the Blueprint, Part V overview, and (when reviewed) Ch 17 against it | The book's central Part V framework must not itself exhibit definition drift; a reviewer or careful reader will spot this immediately | **Critical** |
| Move ↔ ladder mapping | Ch 4 map table says Bind = L1→2, but Ch 4 prose says "Part II moves a product from Level 0 toward Level 2"; Part II overview says Ch 5 is L0→1; Part III overview gives Ch 11 L2→3 while Part IV overview gives Ch 14 L2→3 — two parts claim the same rung | Draw the canonical mapping once (which chapter crosses which rung), then correct Ch 4's table, its prose, and all four part-overview tables to agree | The ladder is the book's "progress bar"; if it doesn't reconcile, the reader loses the ability to locate themselves — the exact feature it exists to provide | **Critical** |
| Part overview files | All five contain internal production notes ("Status: First draft. Next candidates: …") and blueprint references ("the connective chapter the blueprint flagged") | Decide: (a) reader-facing part openers — rewrite in book voice, cut status notes; or (b) internal scaffolding — move to a tracking doc outside `Manuscript/` | As they stand they will leak drafting apparatus into any compiled manuscript | Important |
| Chapter sequencing (Part I) | 1→2→3→4 ordering is right; each chapter ends by explicitly handing to the next | Keep | The handoffs are done well and never feel mechanical | — |
| Repetition across chapters | The "human absorbed the drift / AI removes the slack" idea appears at full length in Ch 1 (twice), Ch 2, Ch 3, and Ch 4, several times in near-identical wording ("reads at machine speed, with no judgement and no memory") | Say it fully once (Ch 1's inflection section is the natural home); thereafter echo in a clause, not a paragraph, and vary the wording | Deliberate motif is good; verbatim recurrence reads as template-filling and dulls the Ch 4 payoff | Important |
| Cross-part duplication | Ch 3 "The separation nobody chose" and Ch 5 "The problem is structural, not careless" contain near-identical passages (the data-lives-here/metadata-lives-there inventory; the "what does this column actually mean?" Slack line appears in both) | In Ch 5, compress to one sentence plus a back-reference ("Chapter 3 traced the mechanism; here is what it costs one account") | Part II should feel like it's building on Part I, not re-arguing it | Important |
| Terminology consistency | Mixed UK/US spelling: Ch 1 uses "catalog" (outside proper nouns) where Ch 2–4 use "catalogue"; Ch 3 and Ch 5 use "artifact" where Front Matter and Ch 2 use "artefact" | Standardise on UK spelling throughout (the book is UK-voiced: "per cent", £, FCA); keep US spelling only inside proper nouns (AWS Glue Data Catalog) | Inconsistent spelling is the kind of surface noise that undermines a book about consistency | Important |
| Ch 8A placement | "Ch08A" filename signals a late insertion | Renumber before production (making it Ch 9 and shifting, or folding into Ch 7/8); update the many forward references ("Chapter 8A") | "8A" in a published book reads as a patch; fine for drafting only | Optional |
| Conclusion strength | Not yet reviewed (Epilogue queued) | — | — | — |

## 3. Expert Review

| Location | Issue | Recommendation | Suggested Edit | Justification | Priority |
|----------|-------|----------------|----------------|---------------|----------|
| Ch 4, "A sixty per cent chance it won't work," opening two paragraphs | The Gartner prediction is conditional — organisations will abandon ~60% of AI projects *that are unsupported by AI-ready data* — but the opening frames it as an unconditional probability: "There is roughly a sixty per cent chance it will not work" | Reframe the hook so the condition carries the drama instead of being flattened out of it | E.g.: "Whether it works is now, according to Gartner, mostly a bet on something that has nothing to do with the model: through 2026, organisations will abandon roughly sixty per cent of AI projects that are not supported by AI-ready data. The question is which side of that line your data puts you on." | Expert accuracy; the book's target readers quote this stat correctly and will notice the slip — in the chapter that introduces the book's central evidence standard | **Critical** |
| Ch 2, "A warning that read as prophecy" | "The failure rate of data-lake initiatives has been put, consistently, at around 85%" — Heudecker's 2017 revision ("closer to 85 percent") was about *big-data projects* broadly, not data-lake initiatives specifically, and the figure is now nearly a decade old | Attribute precisely and date it; let the lake inherit the number honestly | E.g.: "In 2017 Gartner's Nick Heudecker revised the firm's estimate of big-data project failure upward from 60% to 'closer to 85 percent' — and the data lake was the flagship big-data project of the era. The figure is old now, but nothing since suggests the lake's odds improved…" | Industry credibility; misattributed statistics are the first thing a hostile reviewer checks | **Critical** |
| Ch 2, "The first product, concretely" (product charter) | Timeline problem: the charter — which Meridian wrote "before it wrote a line of pipeline code" — already contains the EM division-dependence warning. But the Okonkwo error (Ch 5) happens *because* that meaning was unstated. If the charter predates the pipelines, the book's central error shouldn't have occurred | Make the charter explicitly retrospective: the artefact Meridian wrote *after* the Okonkwo incident, or frame it as "what the charter should have looked like" | E.g. change the intro to: "…here is the one-page product charter Meridian eventually wrote for client_holdings — after the suitability error you will read about in Chapter 5. One line in it is there because of that error." | Narrative continuity; as written, an attentive reader will conclude either the charter or the Okonkwo story can't be true | Important |
| Ch 1, "What didn't happen: three attempts" | The prior-art survey (MDM, EII, semantic web) skips the two attempts the target reader knows best: **data contracts** and **data mesh** — both current, both explicitly attempts to attach meaning/ownership to data. Practitioners will ask "isn't this just data contracts?" 200 pages before the book answers | Add a short fourth entry acknowledging both as the nearest recent attempts, stating in one or two sentences how the capsule differs (contracts bind the *interface promise*; capsules bind the full context; mesh is the operating model this discipline needs — Ch 20), with forward pointers to Ch 5 and Ch 20 | New paragraph after the semantic-web entry: "**Data contracts and data mesh (2019–)** are the nearest living attempts, and the book builds on both rather than replacing them: a contract binds the promise, a capsule binds the full context that makes the promise checkable (Chapter 5), and mesh describes the operating model without which neither survives (Chapter 20)." | Completeness and reader trust; pre-empting the obvious objection in Part I keeps the sceptical practitioner reading | Important |
| Ch 4, whole chapter | The strongest counterargument is never addressed in Part I: "frontier models keep getting better at inferring context — won't the agent just read the wiki, the catalogue, and the dbt docs itself?" The Preface says the Epilogue names falsifiers, but the objection will occur to the reader *here* | Add a short objection-and-answer (3–4 paragraphs or an objection box, mirroring Ch 2's "Objections, answered honestly"): a model can read all five stores of meaning from Ch 3's drift table — and gets five contradictory answers with no way to adjudicate authority or currency; inference over contradictions is the failure mode, not the cure | Anchor it to the Ch 3 drift table: "By month 18, an agent that reads everything reads one production value and four stale descriptions of it. More reading is not more truth." | Conceptual strength; this is the objection every AI-optimist stakeholder will raise, and the book has a good answer it isn't giving yet | Important |
| Ch 3, "The latency ledger" | Half-life estimates ("Half-life measured in weeks… days… one reorg") are presented with empirical confidence but are authorial heuristics | Own them as such in one framing sentence | E.g. before the list: "The half-lives below are field estimates from practice, not measurements — but any practitioner can check them against their own estate." | Technical precision; labelling heuristics as heuristics *increases* credibility with this audience | Important |
| Ch 2, "The economics, on the back of an envelope" | "mature estates still see one issue per ten tables per year" — uncited; the widely circulated observability-vendor benchmark is closer to one incident per 15 tables per year | Either soften to an order-of-magnitude ("an issue per every ten-to-twenty tables per year, on industry observability benchmarks") or attribute to the source type (data-observability vendor incident surveys) | — | Expert accuracy; the envelope math is the passage most likely to be re-used in readers' business cases, so its inputs need to survive scrutiny | Important |
| Ch 3, "…why scale changed it" | "a carefully built catalogue is 40% out of date within months" — a specific number with no source, used as load-bearing evidence for the latency thesis (and echoed in the ledger section) | Attribute the type of source (catalogue-vendor freshness studies / industry surveys), or soften to "a large fraction… within months" and let the drift table carry the argument | — | Claims discipline; the drift table is strong enough that the unsourced stat is a liability, not a support | Important |
| Ch 2, "The economics…" | "data professionals routinely report spending on the order of a third to 40% of their week on data-quality and discovery work" — directionally supported by multiple industry surveys but unattributed | Name the source type: "industry surveys of data teams (state-of-data-engineering and observability studies) consistently put it at a third to 40%" | — | Credibility; "routinely report" invites "reported where?" | Important |
| Ch 4, regulatory table + "deadline" section | Substantively accurate (Aug 2026 phasing, OSFI E-23 2027, BCBS 239, penalties order-of-magnitude), but "becomes broadly operational on 2 August 2026" compresses a phased schedule (prohibitions Feb 2025; GPAI Aug 2025; Annex III high-risk Aug 2026; some embedded-product cases 2027) | One clause of precision in the prose; the table's "phasing from Aug 2026" hedge is already right | E.g.: "The EU AI Act's obligations phase in from 2025, but 2 August 2026 is the date that matters here: it is when the high-risk obligations — credit scoring, eligibility, the exact territory Meridian's suitability model lives in — apply." | Technical precision on the claim the whole urgency argument hangs on | Important |
| Ch 1, big-data era section | "Alongside it came NoSQL databases — Bigtable, Cassandra, HBase" — Bigtable was Google-internal (a 2006 paper, not an adoptable product until Cloud Bigtable in 2015); listing it as capability "in everyone's hands" is loose | "…NoSQL databases — Cassandra and HBase, both descendants of Google's Bigtable paper —" | As given | Technical precision; cheap fix | Optional |
| Ch 4, "Ten questions, asked now" | Good instrument, but the scoring anchor ("A mature product scores in the high teens") is asserted without the 0/1/2 rubric being totalled for the reader | Add "(maximum 20)" after the scoring scheme | "…0 (no answer), 1 (answer in a week), 2 (answer from a query, in the room) — a maximum of 20 —" | Reader usability of the instrument | Optional |

## 4. Editorial Review

| Location | Issue | Recommendation | Suggested Edit | Justification | Priority |
|----------|-------|----------------|----------------|---------------|----------|
| All four chapters | The "it is worth…" construction appears 12+ times across Part I ("worth understanding", "worth stating as plainly as possible", "worth slowing down", "worth naming", "worth watching", "worth unpacking", "worth being honest about", "worth one reference spread", "worth stating as starkly as possible", "worth scoring") | Keep at most one per chapter; elsewhere, cut the throat-clearing and make the point directly | "The 85% figure is easy to quote and hard to feel, so it is worth slowing down on two composite failures" → "The 85% figure is easy to quote and hard to feel. Two composite failures give it texture." | Humanisation; this is the single most identifiable pattern in the draft, and it's pure preamble — the sentences lose nothing when it goes | **Critical** |
| All four chapters | Section-closing aphorism density: nearly every section ends on a balanced epigram ("You can only pay for them later, with interest." / "The files go in beautifully. Nobody can find anything." / "Drift is the *default behaviour* of the arrangement." / "The structure of the cure follows the structure of the disease.") — individually excellent, collectively formulaic | Ration to roughly one per section; where two adjacent sections both end on epigrams, let one end on plain information instead | Keep the best (the filing-system line, "Slowness is what you have now; you are just not counting it") and flatten the runners-up | Humanisation; the reader starts hearing the drumbeat by Ch 3, and the genuinely great lines lose their force | Important |
| Ch 1 §"The inflection" (twice, incl. chapter summary), Ch 3 closing ¶, Ch 4 §agent-action archetype, Ch 5 opening | The "reads at machine speed / no institutional memory / no instinct for when a number smells wrong" formulation recurs near-verbatim four-plus times across Part I and into Ch 5 | Keep the full formulation once (Ch 1's inflection section, its first and best statement); afterwards vary or truncate — each later instance can carry a different concrete absence | See Passage 4 for a worked variation | Style; motifs should rhyme, not repeat | Important |
| Preface ¶4 vs Ch 4 "Meet Meridian" ¶2 | Near-verbatim duplication: "Meridian is not special. Its situation — good people, real data, genuine investment…" appears in both | Keep the full version in Ch 4 (where Meridian is properly introduced); in the Preface, cut to one clause or drop the sentence | — | Structure; a reader hitting the same sentence twice in 60 pages assumes the book wasn't read through by its author | Important |
| Preface ¶4 | The Preface pre-tells both Meridian set-pieces (the confident wrong copilot answer; the model paused because data "could not testify about itself") that Ch 4 previews again and Ch 5/7 tell "fresh" — three tellings before the scene plays | In the Preface, gesture without narrating: name the *kind* of failure, not the specific incidents | "You will watch its adviser copilot fail in a way no quality dashboard predicted, and its people meet questions their well-run functions could not answer." | Flow; foreshadowing works once — the third telling makes Ch 5's cold open feel re-heated | Important |
| Ch 1, "One job, four solutions…" opening | The book's first chapter opens on abstraction ("Strip away fifty years of acronyms…") where Ch 2 and Ch 4 open on concrete scenes — the weakest of the four openings, in the position that matters most | Consider opening Ch 1 with the sediment image made concrete (Meridian's estate: the mainframe, the warehouse, the half-migrated lakehouse, and two AI systems reaching across all of it), then dropping into the history | Move the "Meridian runs something from every row" material forward as a cold open, or write a two-paragraph scene of the estate | Reader engagement at the single highest-stakes page of the manuscript | Important |
| Ch 3 §"The cure that became the disease" | "the reason is the key that unlocks the rest of this book" — cliché, and "unlock" is on every AI-tell watchlist | "…because the reason reframes what a remedy has to be." | As given | Style/humanisation | Important |
| Recurring, all chapters | The "not because X, but because Y" construction appears 6+ times ("not because the data was bad but because…", "not because anyone was negligent but…", "not because it was poorly built but because…") | Keep where the contrast is the actual point (Ch 4's "not dirty… what it lacked was bound context" earns it); elsewhere recast as plain assertion | "It failed not because it was poorly built but because it shared the disease's architecture" → "It was well built. It shared the disease's architecture, and that decided the outcome." | Humanisation; the symmetry is the tell | Important |
| Ch 2 §"Where lake thinking erodes credibility" and elsewhere | Em-dash density is very high (several sentences run two or three pairs); combined with the balanced-clause habit it produces the over-polished cadence | In the voice pass, convert a share of em-dash asides to commas, parentheses, or separate sentences — target roughly half the current density | — | Style; rhythm variation is the cheapest humanisation available | Optional |
| Ch 4 §"Here is what the AI hype glosses over"; Ch 3 §"Here is a sentence that sounds like…" | "Here is…" as a section-opening gambit recurs; the Ch 3 instance works (it sets up a real reversal), the Ch 4 instance is filler | Keep Ch 3; in Ch 4 open on the substance: "Machine learning pipelines are absurdly sensitive to the coherence of their inputs." | As given | Style | Optional |
| Number style across Part I | "sixty per cent" (Ch 4 prose) vs "85%" (Ch 2) vs "40%" (Ch 3) — mixed spelled-out/numeral styling for statistics | Pick one convention (recommend numerals for all percentages and statistics) and apply in the copy-edit pass | — | Consistency; publishers will force a choice anyway | Optional |

## 5. Markdown and Formatting Review

| Location | Issue | Recommendation | Suggested Markdown Edit | Justification | Priority |
|----------|-------|----------------|-------------------------|---------------|----------|
| Ch 1 and Ch 3 have "## Further reading"; Ch 2 and Ch 4 have none | Inconsistent chapter furniture — the two chapters making the heaviest statistical claims (2 and 4) are the two without a references section | Add Further reading to Ch 2 (Gartner swamp/failure analyses; data-product literature) and Ch 4 (Gartner AI-readiness prediction; EU AI Act primary text; NIST AI RMF; ISO/IEC 42001) — or drop the section from all chapters and centralise references | `## Further reading` block matching Ch 1/Ch 3 format | Publication consistency; also the natural home for the source types Section 7 asks for | Important |
| Part I contains zero diagrams; Blueprint §7 promises six diagram families | The three highest-value visuals for Part I are missing: (1) the drift mechanism (Ch 3) — five stores of meaning diverging from production over time; (2) the four-moves × five-levels map (Ch 4) — currently a table doing a diagram's job; (3) the era-sediment stack (Ch 1) | Commission/sketch these three first; the Ch 3 drift table is already 90% of a diagram's content | Placeholder comment convention now (`<!-- FIG 3.1: drift mechanism — five stores diverging -->`) so production can find insertion points | Comprehension; Ch 4's map is the book's navigation aid and deserves better than a 4-row table | Important |
| All five Part overview files | Trailing "**Status:** First draft…" production notes inside manuscript files | Move status/next-step notes to a tracking file outside `Manuscript/` (see Section 2) | Delete the `**Status:**` blocks from manuscript copies | Any Pandoc/production compile of the folder will include them | Important |
| Ch 1 "catalog" ×4 (prose), "catalogue" ×1 (table); Ch 3/Ch 5 "artifact" vs Ch 2/Front Matter "artefact" | Spelling inconsistency (also flagged in Section 2; listed here because it's a find-and-replace pass) | Global pass: `catalog` → `catalogue`, `artifact` → `artefact`, excluding proper nouns (AWS Glue Data Catalog, Unity Catalog) and code identifiers | — | Copy-edit hygiene | Important |
| Ch 2, product charter code block | Unlanguaged fence | Use ` ```text ` (and audit later chapters for `yaml` where applicable) | ` ```text ` | Some renderers style bare fences unpredictably | Optional |
| Ch 4, `### Ten questions, asked now` sits under `## The ladder you are climbing` | The self-assessment isn't ladder content; heading hierarchy buries one of the chapter's best devices | Promote to `##`, or move to sit directly before "The deadline…" | `## Ten questions, asked now` | Heading logic + findability in a compiled TOC | Optional |
| Blockquote furniture (`> **Where you are:**`, `> **Chapter summary.**`, `> **Try this.**`) | Consistently applied — good. Note only that all three render as identical blockquotes | For production, tag them distinctly (e.g., Pandoc fenced divs or a consistent bold-label convention, already present) so the designer can style summary vs exercise differently | — | Conversion readiness; no action needed at draft stage | Optional |
| File naming: `Ch08A - …` | Sorts correctly but signals insertion (see Section 2) | Renumber at production | — | Book production | Optional |

## 6. AI-Generated Language Watchlist

The draft's baseline is well above AI-typical — most sentences would pass any sniff test individually. The exposure is *pattern frequency*. The entries below are representative instances of each pattern, not an exhaustive list; treat each as a class to sweep for.

| Location | Original Wording | Why It Sounds Weak or AI-Generated | Better Alternative |
|----------|------------------|-------------------------------------|--------------------|
| Ch 2, "A warning that read as prophecy" | "so it is worth stating as plainly as possible" | Throat-clearing preamble; twinned almost exactly with Ch 4's "worth stating as starkly as possible" — the repetition across chapters is the tell | Delete the frame; state the thing: "The distinction — technology versus organisation — is the whole of this chapter. **The lake solved a storage problem. The swamp is an organisational failure.**" |
| Ch 4, "The reframe" | "it is worth stating as starkly as possible" | Same construction, same position (setting up the bolded thesis), two chapters apart | "This points to the single most important idea in the book: **AI-readiness is not a data quality problem. It is an evidence problem.**" |
| Ch 3, "The cure that became the disease" | "the reason is the key that unlocks the rest of this book" | "Key that unlocks" is stock thought-leadership metaphor; "unlock" specifically is a flagged AI-tell word | "…because the reason dictates what any real remedy has to look like." |
| Ch 1, "One job, four solutions…" | "It is a genuinely impressive lineage, and it is worth understanding" | "Genuinely" as an empty intensifier + the "worth" tic in the book's second paragraph | "The lineage is impressive, and the platform you inherited is a sediment of it — which is why the history matters." |
| Ch 4, "AI does not work around problems…" | "Here is what the AI hype glosses over: machine learning pipelines are absurdly sensitive…" | "Here is what X glosses over" is a formulaic reveal-frame (contrast Ch 3's "Here is a sentence that sounds like an accusation but is not," which earns the device with an actual reversal) | Open on the claim: "Machine learning pipelines are absurdly sensitive to the coherence of their inputs." |
| Ch 3, chapter summary | "It failed not because it was poorly built but because it shared the disease's architecture" | Fourth-plus instance of the "not because X but because Y" symmetry in three chapters | "The catalogue was well built. It shared the disease's architecture, and architecture decided." |
| Ch 1/Ch 3/Ch 4 (recurring) | "reads at machine speed, with no judgement and no memory" | Verbatim motif recurrence — reads as a template variable by the fourth appearance | Vary per instance: "a consumer that never asks whether a number smells wrong"; "no memory of last quarter's caveats" |
| Ch 2, "Why the correction is urgent now" | "Three pressures have converged to change the economics" | "X forces/pressures have converged" is a stock consulting frame; mild here, but it heads a triad of bolded parallel paragraphs — the full pattern | Keep the triad (it's earned) but roughen the frame: "Three things changed, roughly at once." |

**Also sweep for:** "the whole of it is this:" (Preface); em-dash pairs at 2–3 per sentence (all chapters); triads of bolded parallel paragraphs as the default expository move (Ch 2 ×2, Ch 3 ×2, Ch 4 ×3 — vary at least one per chapter into prose or a table).

**Explicitly keep (human, distinctive, working):** "Sit with how strange that is." (Ch 2); "The files go in beautifully. Nobody can find anything." (Ch 2); "almost embarrassing in its consistency" (Ch 3); "wearing better clothes" (Ch 2); "with almost tedious reliability" (Ch 4); "Slowness is what you have *now*; you are just not counting it." (Ch 2); "Half-life: one reorg." (Ch 3).

## 7. Claims, Assumptions, and Evidence Check

| Claim or Assumption | Concern | Suggested Treatment | Priority |
|---------------------|---------|---------------------|----------|
| Ch 4: "roughly a sixty per cent chance it will not work" | Misframes Gartner's conditional prediction (abandonment of projects *unsupported by AI-ready data*, through 2026) as an unconditional failure probability | Reword the hook (see Section 3, row 1); keep the correctly worded version that follows | **Critical** |
| Ch 2: data-lake failure "consistently… around 85%" | Heudecker's 85% (2017) covered big-data projects broadly; attribution to "data-lake initiatives" specifically overreaches, and the figure's age is unacknowledged | Attribute precisely, date it, and bridge honestly to the lake (see Section 3, row 2) | **Critical** |
| Ch 4: "sixty-three per cent of organisations either do not have, or are not sure they have, the right data management practices for AI" | Matches the Gartner survey language; needs only a year to be audit-proof | Add the survey year in text or Further reading | Important |
| Ch 3: catalogue "40% out of date within months" | No source; load-bearing for the latency thesis; repeated in substance in the ledger section | Attribute source type or soften to qualitative; let the drift table carry the weight | Important |
| Ch 2: "a third to 40% of their week" on quality/discovery work | Directionally consistent with multiple industry surveys, but unattributed | Name the source type (industry surveys of data practitioners); no invented citation | Important |
| Ch 2: "one issue per ten tables per year" | Circulating observability benchmark is nearer 1 per 15 tables/year | Round honestly ("an issue for every ten to twenty tables") or attribute the source type | Important |
| Ch 2: £900k/year reconciliation-tax arithmetic | Checks out (30 × £90k × ⅓ = £900k); depends on the two inputs above | Fix the inputs; the arithmetic can stay | — |
| Ch 4: EU AI Act "broadly operational on 2 August 2026"; penalties "tens of millions of euros or a percentage of global turnover" | Substantively right (Annex III high-risk: Aug 2026; up to €35m/7%); "broadly operational" compresses the phased schedule | One clause of precision in prose; table hedge already correct | Important |
| Ch 4: OSFI E-23 "Effective 2027"; BCBS 239, MAS, NIST, ISO/IEC 42001 rows | Accurate as summarised | None | — |
| Ch 3: latency-ledger half-lives ("weeks… days… one reorg") | Authorial heuristics presented with empirical confidence | Label as field estimates (see Section 3) | Important |
| Ch 1: Gartner 2014 "data swamp" warning; GFS 2003 / MapReduce 2004 / Hadoop 2006 / Kafka / Airflow 2015; Codd 1970 | All check out | None | — |
| Ch 1: "around 2016, gave the field its name" | Defensible ("data engineer" circulated from ~2011 at Facebook; mainstream recognition mid-2010s) | Optionally hedge: "by the mid-2010s" | Optional |
| Ch 2: insurer and bank composite failures | Handled honestly — explicitly labelled composite/anonymised. This is the right pattern; maintain it wherever vignettes look factual | None; replicate the labelling discipline in later parts | — |
| Part V overview: data mesh "stalls (82%)" | Out of this review's depth scope, but flagging early: an 82% figure for mesh failure will need the same sourcing discipline as the 85% lake figure when Ch 20 is reviewed | Pre-check the source before Part V review | Important |

## 8. Missing Content or Underdeveloped Ideas

| Missing or Weak Area | Why It Matters | Suggested Addition | Where It Should Go | Priority |
|----------------------|----------------|--------------------|--------------------|----------|
| The "better models will supply their own context" counterargument | It's the objection the book's most influential sceptic (the AI-optimist exec) will raise, and Part I never engages it; deferring to the Epilogue leaves it live for 20 chapters | 3–4 paragraphs or an objection box: an agent reading all five drifted stores gets five answers and no authority signal; inference over contradiction is the failure mode | Ch 4, after "Why your current data-quality setup will not save you" (or as a fourth objection in a Ch 2-style objections section) | Important |
| Data contracts / data mesh as prior art | Target readers live in this discourse now; silence in Part I reads as either ignorance or evasion | One-paragraph fourth entry in the "attempts" survey with forward pointers to Ch 5 and Ch 20 | Ch 1, "What didn't happen" | Important |
| Diagrams (drift mechanism, moves × levels map, era sediment) | Part I argues visually-shaped ideas entirely in prose and tables; Blueprint §7 already commits to these | Three figures; the Ch 3 drift table is nearly a storyboard for the first | Ch 3, Ch 4, Ch 1 respectively | Important |
| Ch 2 / Ch 4 Further reading sections | The two most statistic-heavy chapters have no references apparatus; also the natural home for the source types Section 7 demands | Match Ch 1/Ch 3 format | Ch 2, Ch 4 | Important |
| Charter timeline clarification | Resolves the Ch 2 vs Ch 5 continuity contradiction | One sentence making the charter retrospective | Ch 2, "The first product, concretely" | Important |
| A canonical statement of the six evidence surfaces | Ch 4 references "the six evidence surfaces of Chapter 17" but lists five; three files disagree on the six | State the canonical six once in Ch 4 (even as a footnote-level aside), then reconcile everywhere | Ch 4, "The reframe" | **Critical** |
| Maximum score on the ten-question instrument | Small usability gap in the book's first self-assessment | "(maximum 20)" | Ch 4, "Ten questions, asked now" | Optional |

## 9. Structural Recommendations

**Part I works as a unit.** The four-chapter arc — history, philosophy, mechanism, stakes-and-map — is the right decomposition, the handoffs between chapters are explicit without being mechanical, and no chapter fails to earn its place. Nothing needs reordering, merging, or cutting at chapter level.

**Chapter 4 is carrying too much, and the fix is trimming rather than splitting.** It runs ~4,800 words against a 3,500 target and stacks nine jobs: the hook, the compounding-error argument, three failure archetypes, the Meridian introduction *and* dossier, the quality-won't-save-you argument, the evidence reframe, the four moves, the ladder, the map table, the ten questions, and the regulatory clock. All of it belongs in the book; not all of it needs full height here. The cheapest reductions: compress "Meet Meridian" now that the dossier exists (the two overlap — the "Meridian is not special" paragraph and the estate description appear in both forms), and tighten the four-moves section, whose content the map table then largely repeats. The archetypes, the reframe, the ladder, the ten questions, and the regulatory table should keep their full footprint — they are the chapter's payload.

**Deduplicate along the Preface → Ch 4 → Ch 5 axis.** The same material currently plays three times: Meridian's two set-piece failures and the "Meridian is not special" framing. Recommended allocation: the Preface gestures (one clause), Ch 4 previews (the archetype paragraphs, which are doing real analytical work), Ch 5 and Ch 7 perform. Similarly, Ch 5's "The problem is structural, not careless" re-argues Ch 3's mechanism nearly verbatim; in a book read front-to-back that section should shrink to a sentence and a back-reference, keeping only what is new (the Okonkwo-specific texture).

**Reconcile the two navigation systems before drafting further.** The move↔ladder map is the book's stated method for locating every chapter, and right now Ch 4's table, Ch 4's prose, and the four part overviews give overlapping but non-identical mappings (who owns L2→3? does Bind start at L0 or L1?). Because every subsequent chapter header promises to announce its rungs, this must be settled once, canonically, and propagated — it will only get more expensive to fix.

**Decide the status of the part-overview files.** They are currently internal briefing documents (status notes, blueprint references, drafting options) sitting inside the manuscript folder. Either promote them to reader-facing part openers — one to two pages, in book voice, doing the "Where you are" work at part scale — or move them out of `Manuscript/`. The reader-facing option is worth considering seriously: the overviews' chapter tables are close to what a good part opener needs.

**Ch 1's opening is the manuscript's weakest high-stakes page.** Chapters 2 and 4 open on scenes; Chapter 1 — the first page after the preface — opens on a thesis. The raw material for a concrete open already exists in the chapter (the Meridian estate as geological cross-section); moving it forward would make the book's first move a picture rather than a claim.

## 10. Recommended Rewrite Samples

```
Passage 1

Location: Ch04, "A sixty per cent chance it won't work", paragraphs 1–2

Original: "You have the budget. You hired the ML engineers. The GPUs are
humming. The AI project is finally real. / There is roughly a sixty per cent
chance it will not work — and it will not be the models that kill it."

Recommended rewrite: "You have the budget. You hired the ML engineers. The
GPUs are humming. The AI project is finally real. / Whether it survives is
now, according to Gartner, mostly a bet on something that has nothing to do
with the models: through 2026, organisations will abandon roughly sixty per
cent of AI projects that are not supported by AI-ready data. The question is
which side of that line your data puts you on — and most teams find out in
month three, when everything quietly stalls."

Why this works better: keeps the hook's rhythm and stakes while stating the
prediction as the conditional it is. The drama actually improves: "which side
of the line are you on" is a sharper personal stake than a coin-flip
statistic, and it survives fact-checking by the book's most statistically
literate readers.
```

```
Passage 2

Location: Ch02, "A warning that read as prophecy", second paragraph

Original: "The failure rate of data-lake initiatives has been put,
consistently, at around 85% — Gartner's Nick Heudecker revised an earlier 60%
estimate *upward* to 'closer to 85 percent,' noting pointedly that the
problem was never the technology."

Recommended rewrite: "In 2017, Gartner's Nick Heudecker revised the firm's
estimate of big-data project failure *upward* — from 60% to 'closer to 85
percent' — noting pointedly that the problem was never the technology. The
data lake was the flagship big-data project of that era, and nothing in the
decade since suggests its odds improved; what changed is only that the
failures got larger and moved to the cloud."

Why this works better: attributes the number to what it actually measured,
dates it, and then earns the application to lakes with an argument instead of
an assertion. The honest version is also more damning — "nothing since
suggests the odds improved" invites the reader to check, which is exactly the
posture the book's Prove thesis demands of data. The book should hold its own
evidence to its own standard.
```

```
Passage 3

Location: Ch03, "The cure that became the disease", first paragraph

Original: "…all of them valuable, all of them necessary, and all of them, on
their own, increasingly insufficient. It is worth understanding why the
obvious cure underdelivered, because the reason is the key that unlocks the
rest of this book."

Recommended rewrite: "…all of them valuable, all of them necessary, and all
of them, on their own, increasingly insufficient. Why the obvious cure
underdelivered is not a side question. The reason dictates what any real
remedy has to look like — and it rules out most of what the industry is
currently selling."

Why this works better: removes the "worth understanding" preamble and the
"key that unlocks" cliché, and replaces a promise about the book with a
claim about the world — which is more confident, more human, and gives the
reader a concrete expectation (the remedy is constrained) rather than a
meta-comment (this part matters).
```

```
Passage 4

Location: Ch03, closing section "Where this leaves us", final paragraph
(third appearance of the machine-speed motif, after Ch01's two)

Original: "The answer is the consumer — the arrival of AI, which reads at
machine speed, carries no institutional memory, and turns every undocumented
assumption into a confident error."

Recommended rewrite (vary, don't repeat): "The answer is the consumer. AI
was never in the meeting where 'active' got redefined. It never met the
analyst who knew which feed to distrust. It has the data, and only the data
— and it turns every undocumented assumption into a confident error."

Why this works better: the motif's *idea* must recur — it is the book's
spine — but its *sentence* should not. Ch01 owns the full formulation;
each later instance can carry a different concrete absence (the meeting,
the analyst), which accumulates evidence instead of accumulating echo. The
same treatment applies to the Ch04 agent-action variant and the Ch05
opening's recurrence.
```

## 11. Reader Experience Review

The practitioner reader this book targets will find Part I unusually respectful of their time. The chapters are honest about being an on-ramp ("compress hard — least original," says the Blueprint, and the compression shows in a good way), and the recurring furniture — "Where you are," chapter summaries, "Try this" — gives the book a usable interface rather than mere decoration. The "Try this" boxes are the standout: each one is genuinely runnable this week, on the reader's own estate, and the front matter's observation that they form a cumulative diagnostic is the kind of design that builds trust.

Where the reader may lose patience: the third and fourth full restatement of the human-absorbed-the-drift thesis. By mid-Chapter 3 the reader has it; by Chapter 4 the verbatim recurrence of the machine-speed formulation starts to register as padding even though the chapters are individually tight. The fix is variation and compression, not cutting the motif.

Where they may question credibility: the statistics. This audience quotes the Gartner 60% prediction correctly at work; the misframed version in Ch 4's opening, the over-attributed 85%, and the unsourced 40%/one-in-ten figures are precisely the places a senior engineer's trust will wobble — and Part I is where that trust is being built. The vignettes have the opposite property: the composite-and-labelled-as-such insurer and bank failures, the drift table, and the latency ledger all *feel* like lived experience and will land hard with anyone who has run an estate.

The chapter-to-chapter pull is strong. Each chapter ends by opening the next one's question, and Ch 4's closing — the copilot "about to give a confident, wrong answer about the Okonkwo family portfolio" — is a genuine page-turner into Part II (and the Ch 5 cold open, checked for this review, pays it off). The reader has a reason to continue at every boundary. Whether the final chapters deliver on Part I's promise is outside this review's depth scope, but the part overviews suggest the payoff structure (vignettes seeded here landing in Part V) is deliberate and tracked.

One format note: the executive path in "How to read this book" routes decision-makers through Ch 2 and Ch 4 — the two chapters with the statistical issues flagged above and no Further reading apparatus. If executives are a named audience, those two chapters' evidence hygiene matters double.

## 12. Final Prioritised Edit Plan

**Critical Fixes**

1. Reframe Ch 4's opening so the Gartner 60% prediction reads as the conditional it is (Section 3, row 1; Passage 1).
2. Correct the 85% attribution in Ch 2 — big-data projects, 2017, bridged honestly to lakes (Section 3, row 2; Passage 2).
3. Canonicalise the six evidence surfaces: one list, stated in Ch 4, reconciled against the Blueprint and Part V overview (and Ch 17 when reviewed).
4. Reconcile the move↔ladder mapping across Ch 4's map table, Ch 4's prose, and all four part overviews; settle who owns each rung before further drafting.
5. Run the "it is worth…" sweep — the single highest-leverage humanisation fix (Section 4, row 1).

**Important Improvements**

6. Deduplicate the Preface → Ch 4 → Ch 5 axis: "Meridian is not special," the two pre-told set-pieces, and Ch 5's re-argument of Ch 3's mechanism.
7. Add the "better models will infer context" counterargument to Ch 4.
8. Add data contracts and data mesh to Ch 1's prior-art survey with forward pointers.
9. Fix the Ch 2 charter timeline so it doesn't contradict the Okonkwo story.
10. Source-type or soften the unsourced statistics (40% catalogue staleness, one-issue-per-ten-tables, a-third-to-40% of the week); label the latency-ledger half-lives as field estimates.
11. Add Further reading to Ch 2 and Ch 4; add the survey year to the 63% claim; add one clause of EU AI Act phasing precision.
12. Vary the machine-speed motif after its first full statement; ration section-closing aphorisms to ~1 per section; recast the weaker "not because X but because Y" instances.
13. Decide the part-overview files' status (reader-facing openers vs internal docs) and strip status notes from `Manuscript/` either way.
14. Trim Ch 4 (~4,800 → nearer 4,000 words): compress "Meet Meridian" against the dossier; tighten the four-moves prose against the map table.
15. Standardise UK spelling (catalogue, artefact) outside proper nouns.
16. Insert figure placeholders for the three Part I diagrams (drift mechanism, moves × levels map, era sediment).
17. Consider a concrete cold open for Ch 1 (the Meridian estate as sediment).

**Optional Polish**

18. Thin em-dash density by roughly half in the voice pass.
19. Standardise number style for statistics (recommend numerals).
20. Promote "Ten questions, asked now" to a `##` heading; add "(maximum 20)" to its scoring rubric.
21. Language-tag the Ch 2 charter code fence; audit later chapters' fences.
22. Precision touch-ups: Bigtable phrasing in Ch 1; "by the mid-2010s" hedge for the data-engineer role date.
23. Renumber Ch 8A before production and update forward references.
24. Pre-check the 82% data-mesh figure (Part V overview) before the Ch 20 review reaches it.

---

*Next review batch on request: Part II (Ch 5–8A) in full depth — including whether the capsule spec survives expert scrutiny and whether Ch 5's Okonkwo telling stays fresh after the Part I trims above.*

