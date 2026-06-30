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

## What this changes for Meridian

With Metadata as Code, Maya's `client_holdings` capsule stops being a thing she maintains and becomes a thing the platform maintains. Its definition lives in a Git repository alongside the transformation code and the test suites. A change to it — a new field, a redefined term, a tightened quality threshold — is a pull request, reviewed by the people who should review it, validated by CI before it can merge. When a custodian renames a column, the change meets a compatibility check that knows what depends on the old name, and the build fails *before* the nulls reach production rather than after. The fourteen downstream dependents are no longer knowledge in two engineers' heads; they are edges in a dependency graph the pipeline consults automatically.

And crucially, this is how the capsule discipline *scales*. Binding one capsule by hand proved the idea (Part II). Metadata as Code is what lets Meridian have a hundred capsules without a hundred times the manual effort, because the binding is now declarative, templated, and enforced by CI/CD. Helena, who runs Meridian's catalogue programme, no longer maintains the catalogue by hand against a moving estate; the catalogue becomes a consumer of these specifications, refreshed from the source of truth rather than curated in parallel with it — a relationship we will make precise in Chapter 13. Tom, who runs the data-quality programme, no longer chases quality after the fact; his expectations become tests in the repository that gate deployment. Governance stops being a meeting and starts becoming a property of the pipeline.

That is the shift, and it sets up the rest of Part III. This chapter argued the principle: treat metadata as a versioned, tested, declarative product, governed like code. The next chapter makes it concrete, walking through the open standards and platform features — OpenLineage, dbt artefacts, AsyncAPI, Great Expectations, and the native governance hooks of the major platforms — that turn the principle into a working stack. Then Chapter 11 brings the third principle, event-driven automation, to its full conclusion: metadata that does not merely sit in Git correctly, but flows through the platform and *acts*.

> **Chapter summary.** Binding context by hand does not scale and does nothing during a fast-moving incident. Manual governance breaks in five ways — schema chaos, documentation drift, policy gaps, slow incident response, compliance theatre — all rooted in treating metadata as a byproduct rather than a product. **Metadata as Code** fixes this by managing metadata as declarative, version-controlled specifications reconciled by CI/CD — GitOps for governance — bringing version control, testing, CI/CD, and peer review to metadata. Its four principles: declarative specs, version-controlled and validated, event-driven automation, and policy as code. The convergence of lakehouse velocity, mature platform APIs, compliance pressure, and DevOps culture makes the shift urgent now. For Meridian, it turns the hand-bound capsule into a platform-maintained, scalable product.

> **Try this.** Take your most-broken pipeline's last incident and ask: was the metadata that would have caught it (the dependency, the schema contract, the quality rule) anywhere in the *path* of the change, or only in a diagram, a wiki, or someone's head? If the latter, you have found a candidate for the first entry in your metadata-as-code repository.
