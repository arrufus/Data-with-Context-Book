# Chapter 13 — Rollout: Capsules as Source, Platforms as Consumers

> **Where you are:** Part III — *Automate*, closing chapter. The pieces exist — the capsule (Part II), metadata as code, the toolchain, active metadata, the standards-based integrator. This chapter turns them into an adoption you can actually run, without boiling the ocean. It also settles the political question that decides whether any of it survives contact with the organisation: what is the source of truth? Maturity: this is the chapter that operationalises the climb from **Level 0 to Level 3**, asset by asset.

---

## The question that decides everything

Maya has a capsule. Helena has a catalogue she spent four hundred thousand pounds and eighteen months building. Tom has a data-quality platform. Meridian, like every enterprise of its size, has already bought governance tools — Collibra or Atlan or Alation or Purview, sometimes more than one. So the moment the capsule discipline meets the real organisation, a political question surfaces that no architecture diagram can dodge: **when the capsule and the platform disagree about who owns a dataset or what a column means, which one is right?**

Get the answer wrong and the whole of Parts II and III unravels into the exact mess they were meant to cure. If the platform is the source of truth, you are back to a separate store of meaning, maintained by hand, drifting from the data — Chapter 3's disease, now wearing a six-figure licence. If the capsule and the platform are *both* sources of truth, you have dual maintenance, two systems to keep in sync, and a guaranteed drift the first time someone updates one and forgets the other. There is only one arrangement that holds, and stating it plainly is the spine of this chapter:

**Data Capsules are the authoritative source of metadata. Governance platforms are net consumers.**

The capsule's metadata is embedded in table properties, schemas, column comments, quality tests, and lineage events — living with the data it describes, versioned together, deployed together. Collibra, Atlan, Alation, Ataccama, and Purview *index, enrich, visualise, and operationalise* that metadata. They provide discovery, stewardship workflows, business-glossary management — genuinely valuable things. But they do not *own* the technical metadata. If the platform and the capsule disagree, you fix the capsule, and the platform refreshes from the source. This is not a dismissal of the platforms Helena and Tom run; it is a division of labour that makes both more valuable, because the platform becomes a reliable window onto the truth rather than a separate system drifting away from it. Helena's catalogue, reframed this way, finally stops going stale — not because her team works harder, but because it now reflects a source rather than trying to *be* one.

Hold that principle; everything in the rollout serves it. Now, how do you actually get there, across a real estate, without a two-year programme that collapses under its own ambition?

## Phase 0: assess and prioritise

You do not start by building infrastructure. You start by finding where the pain is worst and choosing a pilot, because the entire rollout philosophy is *prove value early, expand on evidence* — the opposite of the big-bang metadata project that dies of scope creep and stakeholder fatigue.

Begin with a pain audit. Count the incidents caused by undocumented schema changes — Meridian's 7 a.m. zeros are one data point in a long list. Measure how often analysts ask "what does this column mean?" Assess the compliance exposure: could you answer a GDPR Article 30 request, or a regulator's lineage demand, *today*? Then inventory the tooling: which governance platforms are deployed, what metadata lives *only* in those platforms and is not bound to the data (that is your drift risk), and where meaning is purely tribal, existing only in people's heads. Finally, prioritise for high-impact, low-effort wins: a critical table that causes frequent confusion, a pipeline that has broken twice from undocumented changes, a dataset under regulatory scrutiny. Pick three to five pilot assets.

For Meridian the choice writes itself: `client_holdings` is high-value, has already failed visibly, and sits under suitability regulation. The deliverable of Phase 0 is a ranked list of pilot candidates with justification, documented pain points with estimated cost, and a map of where the governance platforms are used and where the metadata gaps are. That map is also the business case for everything that follows.

## The four phases of adoption

From the pilot, the rollout proceeds in four phases that map cleanly onto a per-asset maturity score — the same Level 0-to-4 ladder the whole book climbs, now applied one dataset at a time.

**Phase 1 — Structural foundation (Level 1).** The goal is schemas versioned with data and basic table-level metadata captured consistently. If you are on Iceberg, Delta, or Hudi, schemas are already versioned in the table's metadata layer — the key is ensuring nobody bypasses it by writing raw Parquet directly, and for streaming, enforcing the schema registry: no schema, no publish. Establish a minimum set of table properties on every asset — `owner`, `domain`, `classification`, `last_reviewed` — living in table properties, not a wiki. Then *gate deployments*: CI/CD blocks any table missing the required properties, and — critically — validates the *content*, not just the presence, of those properties:

```python
REQUIRED = ['owner', 'domain', 'classification']
def validate_capsule(props: dict):
    missing = [p for p in REQUIRED if p not in props]
    if missing: raise ValueError(f"Missing: {missing}")
    if props.get('owner') in ('TBD', 'unknown', ''):
        raise ValueError("Owner cannot be a placeholder")
```

That placeholder check matters more than it looks: an `owner` field reading `TBD` is governance theatre, and a gate that accepts it is theatre too. Finally, configure the governance platform to *pull* these properties from the lakehouse catalogue, so that when someone searches `client_holdings` in Collibra or Atlan, they see the same owner and classification embedded in the table itself. Success means every pilot table is compliant and the platform matches the capsule source.

**Phase 2 — Quality and contracts (Level 2).** Embed quality expectations and formalise producer–consumer agreements. Quality rules belong in code, versioned beside the table definition — dbt tests, Great Expectations, or Soda — and referenced from the capsule. Tests must pass before a version reaches production; you do not discover quality issues after consumers are affected. For high-dependency tables, add an explicit data contract naming the schema, the SLA, the registered consumers, and the breaking-change process (RFC required, notice period to consumers). Then push the test *results* to the governance platform — Atlan ingests dbt test results natively, Collibra's quality module links rules to assets, Alation shows trust flags — so the platform *displays* quality status while the capsule and CI/CD *enforce* it. Note the consistent division of labour: the platform shows, the capsule enforces.

**Phase 3 — Policy, lineage, and governance (Level 3).** Classification moves to the source — sensitivity labels in table and column properties, driving native policy enforcement (Snowflake tag-based masking, Unity Catalog ABAC) so the policy follows the data with no per-query configuration. Lineage is captured by emitting OpenLineage events from orchestration and transformation, flowing to a lineage backend and the standards-based graph of Chapter 12, linked to capsule versions. The governance platforms consume this too: all five major ones ingest OpenLineage (natively or via a bridge like DataHub or Marquez) and sync classifications from the lakehouse. The platform visualises lineage and classification; the capsule remains the source. At the end of Phase 3 an asset has reached Level 3 — policy and lineage embedded, platform synced — which, not coincidentally, is the level at which an AI consumer can safely depend on it.

**Phase 4 — Scale and sustain (Level 4).** Capsule standards become the default for all new tables: CI/CD validates the full checklist — schema, properties, quality tests, classification, ownership, lineage anchor — and incomplete capsules do not deploy. Make compliance *easier than the old way* by providing scaffolding (`new-capsule.sh --name … --domain … --owner …` generating the model, the schema with required properties, the contract, and the test suite), because a standard that is harder than the shortcut loses to the shortcut. Move platform sync from manual connector runs to automation: event-driven on every deployment, scheduled nightly reconciliation, and streaming lineage in real time.

Two Phase-4 patterns are worth dwelling on because they are what make the source-of-truth principle survive real organisational life:

**The feedback loop.** Business users *do* enrich metadata in the governance platform — linking assets to glossary terms, adding certifications, flagging issues — and that context is valuable and must not be lost. The pattern that preserves it without violating the source-of-truth rule: when a steward enriches metadata in the platform, propagate the enrichment back to the capsule as a Git pull request, reviewed and merged, after which the platform refreshes from the updated capsule. The capsule stays authoritative; the business still contributes. This is the answer to the obvious objection — "but our stewards work in Collibra" — and it is what lets Helena's team keep working the way they work while the capsule stays the source.

**Drift detection.** A scheduled job compares platform metadata against the capsule source and alerts on any discrepancy, and the remediation is always the same one sentence: *fix the capsule, then the platform refreshes.* Never the reverse.

```python
def detect_drift(capsule: dict, platform: dict) -> list:
    return [{'field': k, 'capsule': capsule.get(k), 'platform': platform.get(k)}
            for k in ('owner', 'domain', 'classification')
            if capsule.get(k) != platform.get(k)]
```

## The anti-patterns that kill rollouts

Enough rollouts have failed in known ways that the failure modes can be named in advance. Each is a temptation that feels reasonable in the moment and is corrosive over time.

The **platform-as-truth trap**: metadata lives only in Collibra or Atlan while the table properties sit empty — the disease of Chapter 3 with a licence fee. *Mitigation: the capsule is authoritative; the platform indexes.*

**Dual maintenance**: teams update metadata in the platform *and* the capsule separately, guaranteeing drift. *Mitigation: single source — the capsule — with automated sync.*

**Governance theatre**: the properties exist but contain `TBD`. *Mitigation: validate content, not just presence* — the placeholder check from Phase 1.

**Over-engineering**: months building infrastructure before a single capsule ships. *Mitigation: add table properties today; ship value this week.*

**Boiling the ocean**: trying to retrofit every table at once. *Mitigation: phase by criticality and track progress.*

Read the five together and they share one root: impatience with the discipline of *source of truth, one asset at a time*. Every anti-pattern is a shortcut around that discipline, and every shortcut leads back to drift.

## Measuring the climb

Because the rollout is asset-by-asset, you can measure it asset-by-asset, and you should, because the measurement is what justifies the next phase's funding. Score each asset 0 to 4: **0** no metadata, **1** basic properties, **2** quality expectations enforced, **3** policy and lineage embedded and platform synced, **4** full capsule discipline. This is the maturity ladder of Chapter 4, instantiated per dataset. Track the distribution by domain, watch the legacy (Level 0–1) count fall quarter over quarter, and celebrate the milestones, because adoption is a behaviour-change programme as much as a technical one and behaviour change runs on visible progress.

Pair the maturity score with leading and lagging indicators. *Leading*: capsule completeness rate, CI/CD gate pass rate, platform-to-capsule sync success rate. *Lagging*: incident MTTR, data-quality incident rate, compliance-audit prep time, analyst onboarding time. The leading indicators tell you the discipline is taking hold; the lagging ones tell the business it was worth it. When Meridian can show the wealth division's MTTR falling and audit prep compressing from weeks to days, the case for Phase 2 in the next division makes itself.

## What "automate" has delivered, and what comes next

Step all the way back. Part III set out to take the hand-bound capsule of Part II and make it something the platform maintains, enforces, and acts on at scale. It has. Metadata is code, versioned and tested (Chapter 9). The toolchain captures lineage, contracts, and quality automatically (Chapter 10). Active metadata moves that context to the control plane, stopping breaks and feeding agents live (Chapter 11). A standards-based graph connects it across every tool (Chapter 12). And this chapter has made it adoptable — capsules as the source, platforms as consumers, rolled out phase by phase from Level 0 to Level 3 without boiling the ocean. Meridian's client domain now has governed, active, machine-consumable context, maintained automatically, with the governance platforms serving as windows onto the truth rather than rivals to it.

That is *Bind* and *Automate* complete: context attached to data, and kept correct and active by the platform. But there is a consumer this has all been for, and it needs more than governed tables and active metadata — it needs the context *packaged* into something it can reason over. An AI agent does not consume a well-governed table the way an analyst does; it needs the meaning assembled into a product, with an ontology, a semantic layer, a knowledge graph, and usage intelligence wrapped around the data. Governed context is necessary; it is not yet a *product an agent can use*. Turning it into one is the third move of the discipline — **Package** — and it is where Part IV begins, taking Meridian's client domain the final step from a governed data product to a *context* data product.

> **Chapter summary.** The political question that decides everything is the source of truth, and there is one stable answer: **capsules are authoritative; governance platforms (Collibra, Atlan, Alation, Ataccama, Purview) are consumers** that index, enrich, and visualise but do not own the metadata. Rollout starts with **Phase 0** (pain audit, pick 3–5 pilots), then four phases mapping to maturity Levels 1–4: structural foundation, quality and contracts, policy and lineage, scale and sustain — each gated by CI/CD, with the platform displaying what the capsule enforces. Phase 4's feedback loop (steward enrichments flow back to the capsule via PR) and drift detection ("fix the capsule, the platform refreshes") preserve the source-of-truth rule under real organisational life. Avoid the five anti-patterns — platform-as-truth, dual maintenance, governance theatre, over-engineering, boiling the ocean — and measure the climb with a per-asset 0–4 score. This completes *Automate*; next is *Package*.

> **Try this.** For one governance platform you already run, find a single dataset and compare its owner and classification *in the platform* against the same fields *in the table's own properties*. If they differ — or if the table has none — you have found a live instance of the platform-as-truth trap, and the first asset for a capsules-as-source rollout.
