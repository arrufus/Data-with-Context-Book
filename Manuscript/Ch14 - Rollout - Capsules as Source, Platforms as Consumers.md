# Chapter 14 — Rollout: Capsules as Source, Platforms as Consumers

> **Where you are:** Part III — *Automate*, closing chapter. The pieces exist — the capsule (Part II), metadata as code, the toolchain, active metadata, the standards-based integrator. This chapter turns them into an adoption you can actually run, without trying to do it all at once. It also settles the political question that decides whether any of it survives contact with the organisation: what is the source of truth? Maturity: this is the chapter that operationalises the climb from **Level 0 to Level 3**, asset by asset.

---

## The question that decides everything

Maya has a capsule. Helena has a catalogue she spent four hundred thousand pounds and eighteen months building. Tom has a data-quality platform. Meridian, like every enterprise of its size, has already bought governance tools — Collibra or Atlan or Alation or Purview, sometimes more than one. So the moment the capsule discipline meets the real organisation, a political question surfaces that no architecture diagram can dodge: **when the capsule and the platform disagree about who owns a dataset or what a column means, which one is right?**

Get the answer wrong and the whole of Parts II and III unravels into the exact mess they were meant to cure. If the platform is the source of truth, you are back to a separate store of meaning, maintained by hand, drifting from the data — Chapter 3's disease, now wearing a six-figure licence. If the capsule and the platform are *both* sources of truth, you have dual maintenance, two systems to keep in sync, and a guaranteed drift the first time someone updates one and forgets the other. There is only one arrangement that holds, and stating it plainly is the spine of this chapter:

**Data Capsules are the authoritative source of metadata. Governance platforms are net consumers.**

The capsule's metadata is embedded in table properties, schemas, column comments, quality tests, and lineage events — living with the data it describes, versioned together, deployed together. Collibra, Atlan, Alation, and Purview *index, enrich, visualise, and operationalise* that metadata. They provide discovery, stewardship workflows, business-glossary management — genuinely valuable things. But they do not *own* the technical metadata. If the platform and the capsule disagree, you fix the capsule, and the platform refreshes from the source. This is not a dismissal of the platforms Helena and Tom run; it is a division of labour that makes both more valuable, because the platform becomes a reliable window onto the truth rather than a separate system drifting away from it. Helena's catalogue, reframed this way, finally stops going stale — not because her team works harder, but because it now reflects a source rather than trying to *be* one.

Hold that principle; everything in the rollout serves it. Now, how do you actually get there, across a real estate, without a two-year programme that collapses under its own ambition?

## Phase 0: assess and prioritise

You do not start by building infrastructure. You start by finding where the pain is worst and choosing a pilot, because the entire rollout philosophy is *prove value early, expand on evidence* — the opposite of the big-bang metadata project that dies of scope creep and stakeholder fatigue.

Begin with a pain audit. Count the incidents caused by undocumented schema changes — Meridian's 7 a.m. zeros are one data point in a long list. Measure how often analysts ask "what does this column mean?" Assess the compliance exposure: could you answer a GDPR Article 30 request, or a regulator's lineage demand, *today*? Then inventory the tooling: which governance platforms are deployed, what metadata lives *only* in those platforms and is not bound to the data (that is your drift risk), and where meaning is purely tribal, existing only in people's heads. Finally, prioritise for high-impact, low-effort wins: a critical table that causes frequent confusion, a pipeline that has broken twice from undocumented changes, a dataset under regulatory scrutiny. Pick three to five pilot assets.

For Meridian the choice was obvious: `client_holdings` is high-value, has already failed visibly, and sits under suitability regulation. The deliverable of Phase 0 is a ranked list of pilot candidates with justification, documented pain points with estimated cost, and a map of where the governance platforms are used and where the metadata gaps are. That map is also the business case for everything that follows.

### The business case, written out

Phase 0's pain audit is not just prioritisation; it is the *funding artefact*. "Governance is worth it" loses to a spreadsheet unless it *is* a spreadsheet, so here is the one-page cost model it produces, for the wealth division, in round numbers:

| Current annual cost of the status quo | Estimate |
|---|---|
| Reconciliation / discovery time (30 engineers × ⅓ week × cost) | ~£900k |
| Data-quality incidents (≈40/yr × downtime + recovery) | ~£600k |
| Audit / regulatory evidence prep (weeks of manual reconstruction) | ~£250k |
| The paused-model counterfactual (one stalled AI initiative)\* | ~£500k+ |
| **Total recurring drag** | **~£2.25m/yr** |

\*The paused-model line is an annualised *expectation* — the probability-weighted cost of a stalled initiative — not a recurring invoice; a sceptical CFO is right to treat it separately from the three lines above it, which recur every year regardless.

| Cost of the discipline | Estimate |
|---|---|
| Roles (product owners, part-time stewards, platform effort) | ~£600k/yr |
| Infrastructure (graph, active layer, CI) | ~£150k/yr |
| **Total** | **~£750k/yr** |

The shape, not the precision, is the argument: the recurring drag of the ungoverned estate dwarfs the cost of the discipline, and most of the drag is invisible because it is paid in a line no vendor invoices (the reconciliation tax of Chapter 2). Phase 0 turns that invisible cost into a number a sponsor can approve, and the leading indicators below turn it into a number that visibly falls. Chapter 22 develops the full three-year ROI model; Phase 0's job is to produce enough of it to fund Phase 1.

## The four phases of adoption

<!-- FIG 14.1: four-phase rollout against capsule completeness — Phase 1 (structural) → 2 (quality/contracts) → 3 (policy/lineage) → 4 (scale/sustain), each raising per-asset completeness 1→4; a parallel track shows the domain climbing the book ladder toward Level 3. -->

From the pilot, the rollout proceeds in four phases that map onto the per-asset **capsule-completeness** score (0–4) developed later in this chapter — a measure of how much of the binding discipline each dataset carries. That score is an engineering measure, distinct from the book's maturity ladder; reaching completeness 3 across a domain is what lets the domain reach ladder Level 3, where an AI consumer can safely depend on it.

**Phase 1 — Structural foundation (completeness 1).** The goal is schemas versioned with data and basic table-level metadata captured consistently. If you are on Iceberg, Delta, or Hudi, schemas are already versioned in the table's metadata layer — the key is ensuring nobody bypasses it by writing raw Parquet directly, and for streaming, enforcing the schema registry: no schema, no publish. Establish a minimum set of table properties on every asset — `owner`, `domain`, `classification`, `last_reviewed` — living in table properties, not a wiki. Then *gate deployments*: CI/CD blocks any table missing the required properties, and — critically — validates the *content*, not just the presence, of those properties:

```python
REQUIRED = ['owner', 'domain', 'classification']
def validate_capsule(props: dict):
    missing = [p for p in REQUIRED if p not in props]
    if missing: raise ValueError(f"Missing: {missing}")
    if props.get('owner') in ('TBD', 'unknown', ''):
        raise ValueError("Owner cannot be a placeholder")
```

That placeholder check matters more than it looks: an `owner` field reading `TBD` is governance theatre, and a gate that accepts it is theatre too. Finally, configure the governance platform to *pull* these properties from the lakehouse catalogue, so that when someone searches `client_holdings` in Collibra or Atlan, they see the same owner and classification embedded in the table itself. Success means every pilot table is compliant and the platform matches the capsule source.

**Phase 2 — Quality and contracts (completeness 2).** Embed quality expectations and formalise producer–consumer agreements. Quality rules belong in code, versioned beside the table definition — dbt tests, Great Expectations, or Soda — and referenced from the capsule. Tests must pass before a version reaches production; you do not discover quality issues after consumers are affected. For high-dependency tables, add an explicit data contract naming the schema, the SLA, the registered consumers, and the breaking-change process (RFC required, notice period to consumers). Then push the test *results* to the governance platform — Atlan ingests dbt test results natively, Collibra's quality module links rules to assets, Alation shows trust flags — so the platform *displays* quality status while the capsule and CI/CD *enforce* it. Note the consistent division of labour: the platform shows, the capsule enforces.

**Phase 3 — Policy, lineage, and governance (completeness 3).** Classification moves to the source — sensitivity labels in table and column properties, driving native policy enforcement (Snowflake tag-based masking, Unity Catalog ABAC) so the policy follows the data with no per-query configuration. Lineage is captured by emitting OpenLineage events from orchestration and transformation, flowing to a lineage backend and the standards-based graph of Chapter 13, linked to capsule versions. The governance platforms consume this too: all five major ones ingest OpenLineage (natively or via a bridge like DataHub or Marquez) and sync classifications from the lakehouse. The platform visualises lineage and classification; the capsule remains the source. At the end of Phase 3 an asset has reached capsule-completeness 3 — policy and lineage embedded, platform synced — which is what lets its domain reach ladder Level 3, the level at which an AI consumer can safely depend on it.

**Phase 4 — Scale and sustain (completeness 4, estate-wide).** Capsule standards become the default for all new tables: CI/CD validates the full checklist — schema, properties, quality tests, classification, ownership, lineage anchor — and incomplete capsules do not deploy. Make compliance *easier than the old way* by providing scaffolding (`new-capsule.sh --name … --domain … --owner …` generating the model, the schema with required properties, the contract, and the test suite), because a standard that is harder than the shortcut loses to the shortcut. Move platform sync from manual connector runs to automation: event-driven on every deployment, scheduled nightly reconciliation, and streaming lineage in real time.

Two Phase-4 patterns are worth dwelling on because they are what make the source-of-truth principle survive real organisational life:

**The feedback loop.** Business users *do* enrich metadata in the governance platform — linking assets to glossary terms, adding certifications, flagging issues — and that context is valuable and must not be lost. The pattern that preserves it without violating the source-of-truth rule: when a steward enriches metadata in the platform, propagate the enrichment back to the capsule as a Git pull request, reviewed and merged, after which the platform refreshes from the updated capsule. The capsule stays authoritative; the business still contributes. This is the answer to the obvious objection — "but our stewards work in Collibra" — and it is what lets Helena's team keep working the way they work while the capsule stays the source.

**Drift detection.** A scheduled job compares platform metadata against the capsule source and alerts on any discrepancy, and the remediation is always the same one sentence: *fix the capsule, then the platform refreshes.* Never the reverse.

```python
def detect_drift(capsule: dict, platform: dict) -> list:
    return [{'field': k, 'capsule': capsule.get(k), 'platform': platform.get(k)}
            for k in ('owner', 'domain', 'classification')
            if capsule.get(k) != platform.get(k)]
```

### Who does what: a rollout RACI

Adoption stalls most often on unclear responsibility, so name it. Across the four phases, the roles divide like this:

| Activity | Producer team | Platform | Governance | Domain owner |
|----------|:---:|:---:|:---:|:---:|
| Write capsule specs | **R** | C | C | **A** |
| CI gates & reconciliation | C | **R/A** | C | I |
| Quality suites | **R** | C | C | **A** |
| Classification & policy | C | R | **A** | C |
| Platform sync (capsule→consumer) | I | **R/A** | C | I |
| Sign-off for AI use | I | I | C | **A** |

(R = responsible, A = accountable, C = consulted, I = informed.) The pattern that matters: the **domain owner is accountable** for the product's specs, quality, and AI-use sign-off; the **platform is accountable** for the machinery (CI, reconciliation, sync); **governance is accountable** for policy. Blur these — as the failed rollouts in a moment do — and the work falls between stools.

### The steward feedback loop, end to end

<!-- FIG 14.2: the steward feedback loop — steward enriches in the governance platform → overnight job opens a PR against the capsule's glossary/contract → owner reviews and merges → platform refreshes from the updated capsule; the capsule stays authoritative, the enrichment is preserved. -->

The Phase-4 feedback loop is where the source-of-truth rule survives real organisational life, so trace it concretely. A steward, working in Collibra (where stewards work), adds a business-glossary link and a certification note to a product. An overnight job detects the enrichment and opens a **pull request against the capsule's glossary and contract files**, attributing it to the steward. Maya reviews and merges it; the capsule version bumps; the platform then **refreshes from the updated capsule**, so Collibra now shows the enrichment *because it flowed back to the source and forward again*, not because it was edited in place. Conflicts (steward and capsule disagree) resolve one way only: the PR makes the disagreement visible and a human decides, in Git, with history. The architecture is event-driven for speed and nightly-reconciled as a backstop. This is the mechanism that lets Helena's team keep working in the catalogue while the capsule stays authoritative — the answer to "but our stewards live in the platform."

### A rollout calendar

Adoption is a behaviour-change programme, so it has a calendar, not just a set of phases. Meridian's, quarter by quarter, with maturity-distribution targets:

- **Q1 — Pilot.** `client_holdings` and 2–3 wealth products to Level 3. Target: pilot cohort at L3, MTTR baseline captured.
- **Q2 — Domain 2–3.** Extend to the two domains the pilot's value summary justified. Target: 20% of tier-1 products at L2+.
- **Q3 — Division.** Roll across the wealth division; scaffolding and standards now reusable. Target: 50% of tier-1 at L2+, legacy (L0–1) count falling.
- **Q4 — Enterprise.** Second division begins; the context standard becomes default for new products. Target: all *new* products born at L1+, no L0 tier-1 products.

The calendar is what turns "capsules are the source of truth" from a principle into a dated programme a steering committee can track — and the falling legacy count is the number that proves it is working.

## The anti-patterns that kill rollouts

Enough rollouts have failed in known ways that the failure modes can be named in advance. Each is a temptation that feels reasonable in the moment and is corrosive over time.

The **platform-as-truth trap**: metadata lives only in Collibra or Atlan while the table properties sit empty — Chapter 3's separation, re-bought. *Mitigation: the capsule is authoritative; the platform indexes.*

**Dual maintenance**: teams update metadata in the platform *and* the capsule separately, guaranteeing drift. *Mitigation: single source — the capsule — with automated sync.*

**Governance theatre**: the properties exist but contain `TBD`. *Mitigation: validate content, not just presence* — the placeholder check from Phase 1.

**Over-engineering**: months building infrastructure before a single capsule ships. *Mitigation: add table properties today; ship value this week.*

**Boiling the ocean**: trying to retrofit every table at once. *Mitigation: phase by criticality and track progress.*

Read the five together and they share one root: impatience with the discipline of *source of truth, one asset at a time*. Every anti-pattern is a shortcut around that discipline, and every shortcut leads back to drift.

## Measuring the climb

Because the rollout is asset-by-asset, you can measure it asset-by-asset, and you should, because the measurement is what justifies the next phase's funding. Score each asset 0 to 4 on **capsule completeness**: **0** no metadata, **1** basic properties, **2** quality expectations enforced, **3** policy and lineage embedded and platform synced, **4** full capsule discipline. This is deliberately an *engineering* score, not the book's maturity ladder — a product can be a complete capsule (4 here) while still sitting at Level 1 on the ladder, because the ladder measures *meaning* (glossary, ontology, graph, a live AI consumer) and this score measures *binding*. The two climb together: Chapter 4's ladder tells you what the domain understands; this score tells you what the platform enforces. Track the distribution by domain, watch the legacy (completeness 0–1) count fall quarter over quarter, and celebrate the milestones, because adoption is a behaviour-change programme as much as a technical one and behaviour change runs on visible progress.

Pair the maturity score with leading and lagging indicators. *Leading*: capsule completeness rate, CI/CD gate pass rate, platform-to-capsule sync success rate. *Lagging*: incident MTTR, data-quality incident rate, compliance-audit prep time, analyst onboarding time. The leading indicators tell you the discipline is taking hold; the lagging ones tell the business it was worth it. When Meridian can show the wealth division's MTTR falling and audit prep compressing from weeks to days, the case for Phase 2 in the next division needs no selling.

## Change management, and two rollouts

The technical rollout is the easy half; the change-management half is where programmes actually live or die, and it deserves more than a footnote. Three practices carry it. **Training and office hours** — regular, low-ceremony sessions where teams bring their real capsules and get help, so the discipline is learned by doing not by decree. **The capsule champion pattern** — one enthusiast per domain, seeded early, who becomes the local expert and the social proof that this is normal, not imposed. And **a plan for the team that refuses** — because there is always one, and the answer is neither to force nor to exempt them but to let the *value* of the adopting teams (faster delivery, fewer incidents, products other teams pull from) make the holdout's position untenable, while leadership holds the line that new products are born governed. Adoption follows demonstrated utility; it cannot be mandated ahead of it.

Two short cases, generalised from patterns across the industry, show the difference between a rollout that scaled and one that stalled.

**The stall (Phase 2, forever).** A firm stood up the CI gates, wrote specs for its pilot products, and got them to Level 1 — and then stopped. Quality suites (Phase 2) never got written for anything beyond the pilot, because the domain teams treated capsules as a platform-team deliverable, not their own. The RACI above was never established: the platform "owned" the capsules, which meant the domains did not, which meant the specs went stale and the catalogue drifted back to being the de facto truth. Eighteen months in, they had a governed pilot and an ungoverned everything-else, and the programme quietly lost its funding. The failure was ownership, not technology — the exact operating-model failure Chapter 21 anatomises.

**The scale (past the pilot).** A second firm did one thing differently: it made domain product owners *accountable* (the RACI's A) from Phase 1, funded a part-time steward per domain, and required every *new* product to be born as a capsule. The pilot proved value; the value summary funded the next two domains; the scaffolding made the second domain faster than the first; and by the fourth quarter the discipline was self-propagating, because a new product was easier to build the governed way than the old way. The difference between the two was not tooling — they ran similar stacks. It was that the second treated product ownership as the work and the platform as the enabler, and the first treated the platform as the work and ownership as something to back-fill later. The platform can be procured; the ownership has to be built, and nobody budgets for the building — which is Chapter 21's subject, met here for the first time. That single distinction, more than any technology in Part III, decides whether the rollout takes.

## What "automate" has delivered, and what comes next

Step all the way back. Part III set out to take the hand-bound capsule of Part II and make it something the platform maintains, enforces, and acts on at scale. It has. Metadata is code, versioned and tested (Chapter 10). The toolchain captures lineage, contracts, and quality automatically (Chapter 11). Active metadata moves that context to the control plane, stopping breaks and feeding agents live (Chapter 12). A standards-based graph connects it across every tool (Chapter 13). And this chapter has made it adoptable — capsules as the source, platforms as consumers, rolled out phase by phase from Level 0 to Level 3 without trying to do it all at once. Meridian's client domain now has governed, active, machine-consumable context, maintained automatically, with the governance platforms serving as windows onto the truth rather than rivals to it.

That is *Bind* and *Automate* complete: context attached to data, and kept correct and active by the platform. But there is a consumer this has all been for, and it needs more than governed tables and active metadata — it needs the context *packaged* into something it can reason over. An AI agent does not consume a well-governed table the way an analyst does; it needs the meaning assembled into a product, with an ontology, a semantic layer, a knowledge graph, and usage intelligence wrapped around the data. Governed context is necessary; it is not yet a *product an agent can use*. Turning it into one is the third move of the discipline — **Package** — and it is where Part IV begins, taking Meridian's client domain the final step from a governed data product to a *context* data product.

## Further reading

- On the operating-model dependency this chapter sets up, *Data Mesh* (Zhamak Dehghani) for domain ownership and federated computational governance (developed in Chapter 21).
- On the governance platforms positioned here as consumers, the Collibra, Atlan, Alation, and Microsoft Purview documentation — and their OpenLineage / dbt-test ingestion capabilities specifically.
- On change management for data programmes, the practitioner literature on data-product champions and the "make the governed path the fast path" pattern used throughout Part III.

> **Chapter summary.** The political question that decides everything is the source of truth, and there is one stable answer: **capsules are authoritative; governance platforms (Collibra, Atlan, Alation, Purview) are consumers** that index, enrich, and visualise but do not own the metadata. Rollout starts with **Phase 0** (pain audit, pick 3–5 pilots), then four phases mapping to per-asset capsule-completeness 1–4: structural foundation, quality and contracts, policy and lineage, scale and sustain — each gated by CI/CD, with the platform displaying what the capsule enforces, and the domain reaching ladder Level 3 as its assets reach completeness 3. Phase 4's feedback loop (steward enrichments flow back to the capsule via PR) and drift detection ("fix the capsule, the platform refreshes") preserve the source-of-truth rule under real organisational life. Avoid the five anti-patterns — platform-as-truth, dual maintenance, governance theatre, over-engineering, boiling the ocean — and measure the climb with a per-asset 0–4 score. This completes *Automate*; next is *Package*.

> **Try this.** For one governance platform you already run, find a single dataset and compare its owner and classification *in the platform* against the same fields *in the table's own properties*. If they differ — or if the table has none — you have found a live instance of the platform-as-truth trap, and the first asset for a capsules-as-source rollout.
