# Chapter 9 — Metadata as Code: GitOps for Governance

> **Where you are:** Part III — *Automate*, the second move of the discipline. Part II bound context to data by hand, one capsule at a time. This chapter makes the binding *systematic*: declarative, version-controlled, tested, and deployed like software. Maturity target: making **Level 1** durable and repeatable across many datasets, the foundation for **Level 2**.

---

## Seven in the morning, and every dashboard reads zero

It is seven in the morning at Meridian, ninety minutes before the markets open, and every relationship-manager dashboard in the retail division is showing zero. Zero assets under management. Zero client holdings. Zero everywhere a number should be. The overnight client-reporting run completed without error — green ticks the whole way down — and produced a portfolio of nothing. Maya's Slack is already a wall of red: advisers who have client meetings at nine, a desk head asking whether the firm has lost its data, a compliance officer asking, more quietly, whether anyone outside the building can see this.

The cause, when it is finally traced two hours later, is mundane to the point of insult. A custodian feed renamed a column — `acct_status` became `account_status` — in a routine upstream release. One enrichment job depended on the old name, silently returned nulls, and the nulls propagated through fourteen downstream transformations into every Gold table the dashboards read. Nobody did anything wrong, in the now-familiar sense. The custodian's change was sensible on their side. Meridian's pipeline did exactly what it was told. The metadata that *should* have caught the mismatch — the knowledge that fourteen things depended on that column under its old name — existed, but it existed in a lineage diagram drawn months ago, in two engineers' heads, and in a catalogue that updates nightly and so still showed yesterday's reality. None of it was in the path of the change. None of it could *act*.

This scenario plays out in data teams every week, and it is not a horror story so much as a diagnosis. Part II gave Meridian a way to bind context to a dataset — the capsule. But Maya bound the `client_holdings` capsule *by hand*: she wrote its definition, she reviewed it, she deployed it. That works for one capsule. It does not work for fifty, and it does nothing at all at 7 a.m. when a column rename is racing through the estate faster than any human can trace it. The binding has to stop being a craft act and become a property of the platform — declarative, versioned, tested, and enforced automatically. That is what this part of the book is about, and it begins with treating metadata the way the industry already learned to treat infrastructure: as code.

## Why manual governance breaks down

The 7 a.m. failure is a specific instance of a general condition: the velocity and volume of change in a modern data estate have outrun the human-scale processes meant to govern it. It is worth being precise about the ways manual governance breaks, because each one is a thing Metadata as Code is designed to fix.

Consider the shape of Meridian's estate, which is unremarkable for a firm its size. Fifty-plus data sources — custodians, the CRM, the billing system, the settlement mainframe, market-data feeds, SaaS platforms. Hundreds of transformation steps across dbt, Spark, and Flink. Multiple storage layers: the Iceberg lakehouse, a Snowflake footprint for certain analytics marts, a feature store for the models. Dozens of consuming applications, from BI dashboards to the adviser copilot to regulatory reporting. What began, years ago, as a few tidy ETL jobs is now an intricate web of transformations, dependencies, and integrations spanning multiple clouds, tools, and teams. Manual governance does not scale into that web, and it fails in five recognisable ways.

**Schema evolution chaos.** A single column rename in a source cascades through twenty downstream transformations, breaks fifteen reports and three models, and the damage is done before anyone notices. The 7 a.m. zeros are exactly this.

**Documentation drift.** The carefully built catalogue is forty per cent out of date within months — not through negligence but because schemas at scale change faster than humans can document them. This is Chapter 3's latency problem, restated operationally.

**Policy enforcement gaps.** Beautiful governance policies live in slide decks, but enforcing them consistently across Iceberg, Snowflake, and the streaming platform requires armies of analysts manually applying tags, permissions, and checks. The policy exists; nothing makes it true.

**Incident response delays.** When something breaks, understanding the blast radius takes hours or days, because lineage is scattered across tools, tribal memory, and stale documentation. Two hours to trace a column rename is the optimistic case.

**Compliance theatre.** A regulatory audit becomes a scramble to assemble evidence, because the governance processes were never automated, auditable, or version-controlled. The evidence has to be reconstructed because it was never captured.

Underneath all five is a single root cause, and naming it is the pivot of this chapter: **we have been treating metadata as a byproduct instead of a product.** Schema, ownership, definitions, quality rules, lineage, policy — all of it has been generated as exhaust from the real work and stored, if at all, in whatever passive system was nearest to hand. Metadata that is a byproduct drifts, because nothing is responsible for keeping it true. Metadata that is a *product* — versioned, tested, owned, deployed — cannot, because the same machinery that keeps software honest keeps it honest too.

## What Metadata as Code means

**Metadata as Code** means managing metadata through declarative specifications — YAML or JSON — that are version-controlled in Git and applied and reconciled by automated pipelines. It is the application of GitOps principles, already standard in infrastructure, to data governance. The capsule definition Maya wrote in Chapter 5 was, though we did not dwell on the point, exactly such a specification. Part III is about taking that specification and running it like code: branching it, reviewing it, testing it, deploying it through CI/CD, and reconciling the live estate against it continuously.

The analogy that unlocks the idea is Infrastructure as Code. A decade ago, infrastructure was configured by hand — someone logged into a console and clicked, and the configuration that resulted lived nowhere except in the running system and the operator's memory. Terraform and its kin changed that: infrastructure became a declarative specification in Git, and the running system became a deployment of that specification, with drift between the two detected and corrected automatically. The benefits were not subtle — version control, peer review, testing, rollback, reproducibility — and they transformed operations from a craft into an engineering discipline. Metadata as Code brings precisely those benefits to governance:

- **Version control.** Every metadata change is tracked, reviewable, and revertible. The `acct_status` rename would have arrived as a commit, with a diff, against a definition that declared fourteen dependents.
- **Testing.** Metadata definitions are validated, linted, and tested before deployment, the way code is.
- **CI/CD integration.** A schema change triggers automated validation and impact analysis before it ships, not after it breaks production.
- **Collaboration.** Teams review and approve metadata changes through pull requests, so a change to what `em_exposure_pct` means is a reviewed event, not a silent one.

Here is what a fragment looks like — a data product defined as code, the same shape as Meridian's `client_holdings` capsule, now living in a repository where CI/CD reconciles it against the live platform:

```yaml
version: v1
kind: DataProduct
metadata:
  name: client-holdings
  namespace: wealth
spec:
  owner: maya.osei@meridian.example
  description: "Client portfolio holdings with suitability context"
  sla:
    freshness: "< 4 hours"
    availability: "> 99.9%"
  tables:
    - name: fact_client_holding
      tests:
        - unique: [client_id, holding_id, valuation_date]
        - not_null: [client_id, division, risk_score]
        - accepted_values:
            column: division
            values: [retail, institutional]
  tags:
    - classification: confidential
    - retention: "7_years"
    - business_critical: "true"
```

This is not documentation that describes the data product. It is the specification the data product is *deployed from and validated against*. Change the file, open a pull request, and CI runs the tests, checks the schema against downstream consumers, and blocks the merge if something would break. The metadata has teeth because it is code, and code has a CI pipeline standing behind it.

## The repository, and a change in flight

"Metadata as code" is only real if there is a *repository* it lives in, so here is the shape of Meridian's — the tree that turns the abstraction into somewhere an engineer commits on Monday (and the seed of Appendix B):

```
meridian-data-platform/
├── contracts/            # data contracts (ODCS), one per product
│   └── wealth/fact_client_holding.yaml
├── table-specs/          # capsule/table definitions -> DDL
│   └── wealth/fact_client_holding.yaml
├── tests/                # quality suites (dbt / Great Expectations)
│   └── fact_client_holding/
├── policies/             # policy-as-code: classification, masking, access
│   └── pii.yaml
├── glossary/             # business definitions, division rules
│   └── wealth/emerging_markets.yaml
├── pipelines/            # transformation code (dbt models, jobs)
├── CODEOWNERS            # who must review changes to each path
└── .github/workflows/    # the CI that reconciles all of it
```

The layout encodes the discipline: the contract, the schema, the tests, the policy, and the glossary definition for a product all live in the same repository and move together, so a change to one is visible alongside the others. `CODEOWNERS` is quietly load-bearing — it is what pulls the right reviewer into a change automatically.

Now watch the change that caused the Okonkwo error travel through this repository *as a pull request* instead of as a silent production edit. An engineer proposes redefining `em_exposure_pct` to be division-aware. They edit three files: the glossary definition (`glossary/wealth/emerging_markets.yaml`), the table spec, and the contract. Opening the PR fires the CI, and `CODEOWNERS` — because the change touches `glossary/wealth/` and `contracts/wealth/` — automatically requests review from Maya (the product owner) and the reviewer who understands both divisions. The CI classifies the change as **major** (a semantic redefinition), so it demands the 90-day consumer-notice check; it runs impact analysis and lists the copilot and three reports as affected; it runs the quality suite. The reviewer who understands retail-versus-institutional sees the diff *before* it reaches production — the exact human check that was missing when the change slipped through as an enrichment-job edit. On merge, the contract version bumps, the notice fires to the registered consumers, and the change deploys. The whole Okonkwo failure, replayed, is caught at the pull request. That is the chapter's central promise, shown once, concretely.

### The CI stages

The pipeline that stands behind the PR runs a fixed sequence of stages, and it is worth naming them because each is a guarantee:

```
lint → schema-validate → contract-compatibility → quality-tests →
impact-analysis → (on main) deploy + notify-consumers
```

Lint rejects malformed specs; schema-validate checks the spec against the capsule/ODCS schema; contract-compatibility computes the semver class and blocks a breaking change lacking sign-off and notice; quality-tests gate on the bound suite; impact-analysis enumerates downstream consumers from the lineage graph; and only from `main` does the change apply and the consumer notification fire. Every stage is a place a bad change stops, and the only path to production runs through all of them.

## The four principles

Effective Metadata as Code rests on four principles. They are worth stating plainly because every chapter in Part III is an elaboration of one or more of them.

**1. Declarative specifications.** Express governance *intent* in human-readable, version-controlled configuration rather than imperative scripts. You declare the desired state — these owners, these tests, these classifications, this SLA — and let the tooling reconcile the live system to it. You do not write a procedure that applies a tag; you declare that the column is `confidential` and let the platform enforce what that means. Declarative beats imperative for the same reason it won in infrastructure: the desired state is reviewable, diffable, and reproducible, and reconciliation is the machine's job.

**2. Version-controlled and automatically validated.** Every metadata change flows through standard software development workflow: a feature branch, a pull request, automated tests, a deployment pipeline. This ends the era of uncontrolled changes to the data estate. A schema change, a new classification, a redefined business term — each requires documented justification, peer review, and deployment through CI/CD, with full traceability and rollback. The `acct_status` rename, in this world, cannot reach production without passing a compatibility check against everything that depends on it.

**3. Event-driven automation.** Metadata changes propagate automatically through event-driven mechanisms — webhooks, Kafka streams, message queues. When a schema evolves, downstream consumers receive structured notifications and respond: pipelines pause for validation, consuming applications trigger compatibility checks, quality monitors activate new tests. This is the principle that closes the latency gap of Chapter 3, and it is the bridge to active metadata in Chapter 11 — metadata that does not wait to be read but acts the moment something changes.

**4. Policy as code.** Access control, masking, and quality policies are defined *alongside* the data definitions, not in a separate governance tool that drifts from them. This keeps policy synchronised with the data it governs, because the two live in the same repository and deploy in the same act. Meridian's rule that `client_id` is masked for non-production roles is not a setting in a console someone might change; it is a line in the capsule spec, reviewed and versioned with everything else.

## Why now, specifically

Teams have managed metadata by hand for years; the reasonable question is why the manual approach has tipped from inconvenient to untenable now. Several forces have converged.

**Lakehouse and streaming have exploded the rate of change.** A single organisation now runs real-time pipelines processing millions of events per second, Iceberg and Delta tables with frequent schema evolution, dbt transformations spawning hundreds of derived datasets daily, and feature stores with strict freshness requirements. The velocity and volume of change have simply overwhelmed human-scale governance.

**Platform APIs have matured.** Modern platforms now ship first-class metadata APIs that make programmatic governance possible: Snowflake's object tagging, Databricks Unity Catalog's lineage APIs, BigQuery's column-level policy tags, Confluent's schema registry. Metadata has gone from read-only documentation to a programmable asset, which is what makes "as code" feasible rather than aspirational. Chapter 10 is largely a tour of these.

**Compliance now demands provable, auditable governance at scale.** GDPR, the EU AI Act, and the financial-services regimes Meridian lives under require demonstrable lineage, access evidence, quality attestation, and retention enforcement. Manual processes leave audit gaps that translate into fines. Governance that is version-controlled and automated produces the evidence by construction.

**Data teams have absorbed DevOps culture.** dbt generates machine-readable artefacts describing transformations and tests; Airflow and Prefect expose programmatic interfaces; Great Expectations makes quality testable; Terraform manages resources declaratively. The tools and the muscle memory for treating data work like software engineering are already in the building. Metadata as Code is the natural extension.

**And the cost of manual governance has become quantifiable.** Data-downtime incidents, teams losing a third of their week hunting and cleaning datasets, audit preparation measured in months — these are now line items, and the ROI case for automation has become hard to argue against. The 7 a.m. zeros had a cost: advisers' time, a compliance scare, two hours of senior engineers' morning. Multiply by a year of such incidents and the business case for automation writes itself.

## Reconciliation: what GitOps does when reality drifts

The Infrastructure-as-Code analogy has one more part the principles imply but do not spell out: *reconciliation*. Terraform does not only apply changes; it continuously compares the declared state in Git against the live infrastructure and flags — or corrects — drift. Metadata as Code needs the same loop, because a live table *can* still be changed out of band (a console edit, a hotfix, a permission granted directly), and when it is, the Git definition and reality diverge exactly as they did in the bad old world.

So Meridian runs a scheduled reconciliation: a job compares each live table's actual schema, properties, and policy tags against the capsule spec in Git, and on any discrepancy it does one of two things depending on blast radius. For low-risk drift (a missing description, a stale property), it *auto-reverts* to the declared state — the Git definition wins, silently. For high-risk drift (a schema change, a classification downgrade, a policy removed), it *does not* auto-revert; it raises an alert and opens a ticket to the owner, because an unexplained schema change might be a legitimate emergency fix that needs to be reconciled *into* Git rather than stamped out. The rule mirrors Chapter 13's source-of-truth principle: the Git spec is authoritative, and reconciliation is how that authority is enforced continuously rather than assumed. Drift becomes a detected, routed event, not a silent divergence discovered in an incident.

## Adopting this on a brownfield estate

Nobody starts clean, so the realistic question is sequencing. You do not codify the whole estate at once; you codify the metadata that hurts most, first. Meridian's order: **schema and ownership first** (the cheapest to capture and the most immediately useful — every table gets a spec with an owner, gated in CI), **then quality** (the bound test suites for tier-1 products), **then policy** (classification and masking as code), **then semantics and lineage** (the glossary definitions and OpenLineage wiring). Scaffolding makes compliance easier than the old way — a `new-capsule.sh` generator produces the spec, the contract skeleton, and the test stub, so the governed path is the *fast* path, not the slow one. The sequencing principle is the same as everywhere in the book: highest pain, lowest effort, first — and make the right thing the easy thing, or the shortcut wins.

## The honest costs

Metadata as Code is not free, and pretending otherwise is how it gets abandoned at the first friction. Three costs are real. **Review latency:** routing metadata changes through pull requests adds hours or a day to changes that used to be instant console edits — worth it for the traceability, but a genuine change to cadence that teams must be prepared for. **YAML fatigue:** engineers who wanted to write transformations now also write specs, and without good scaffolding and templates the ceremony breeds resentment; the mitigation is generators and sane defaults so the spec is mostly filled in. **The governance-team skill shift:** stewards who curated a catalogue by hand now review pull requests and write policy-as-code — a real reskilling from documentation to engineering, which some will welcome and some will resist (a thread Chapter 20 picks up as Tom's and Helena's arcs). None of these is a reason not to do it; each is a reason to do it deliberately, with tooling and change management, rather than by decree.

## What this changes for Meridian

With Metadata as Code, Maya's `client_holdings` capsule stops being a thing she maintains and becomes a thing the platform maintains. Its definition lives in a Git repository alongside the transformation code and the test suites. A change to it — a new field, a redefined term, a tightened quality threshold — is a pull request, reviewed by the people who should review it, validated by CI before it can merge. When a custodian renames a column, the change meets a compatibility check that knows what depends on the old name, and the build fails *before* the nulls reach production rather than after. The fourteen downstream dependents are no longer knowledge in two engineers' heads; they are edges in a dependency graph the pipeline consults automatically.

And crucially, this is how the capsule discipline *scales*. Binding one capsule by hand proved the idea (Part II). Metadata as Code is what lets Meridian have a hundred capsules without a hundred times the manual effort, because the binding is now declarative, templated, and enforced by CI/CD. Helena, who runs Meridian's catalogue programme, no longer maintains the catalogue by hand against a moving estate; the catalogue becomes a consumer of these specifications, refreshed from the source of truth rather than curated in parallel with it — a relationship we will make precise in Chapter 13. Tom, who runs the data-quality programme, no longer chases quality after the fact; his expectations become tests in the repository that gate deployment. Governance stops being a meeting and starts becoming a property of the pipeline.

That is the shift, and it sets up the rest of Part III. This chapter argued the principle: treat metadata as a versioned, tested, declarative product, governed like code. The next chapter makes it concrete, walking through the open standards and platform features — OpenLineage, dbt artefacts, AsyncAPI, Great Expectations, and the native governance hooks of the major platforms — that turn the principle into a working stack. Then Chapter 11 brings the third principle, event-driven automation, to its full conclusion: metadata that does not merely sit in Git correctly, but flows through the platform and *acts*.

> **Chapter summary.** Binding context by hand does not scale and does nothing during a fast-moving incident. Manual governance breaks in five ways — schema chaos, documentation drift, policy gaps, slow incident response, compliance theatre — all rooted in treating metadata as a byproduct rather than a product. **Metadata as Code** fixes this by managing metadata as declarative, version-controlled specifications reconciled by CI/CD — GitOps for governance — bringing version control, testing, CI/CD, and peer review to metadata. Its four principles: declarative specs, version-controlled and validated, event-driven automation, and policy as code. The convergence of lakehouse velocity, mature platform APIs, compliance pressure, and DevOps culture makes the shift urgent now. For Meridian, it turns the hand-bound capsule into a platform-maintained, scalable product.

> **Try this.** Take your most-broken pipeline's last incident and ask: was the metadata that would have caught it (the dependency, the schema contract, the quality rule) anywhere in the *path* of the change, or only in a diagram, a wiki, or someone's head? If the latter, you have found a candidate for the first entry in your metadata-as-code repository.
