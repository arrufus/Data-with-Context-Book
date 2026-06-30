# Chapter 7 — Capsules for AI/ML and Data in Motion

> **Where you are:** Part II — *Bind*, final chapter. Chapters 5 and 6 bound context to tables at rest. This chapter extends the discipline to the workloads where getting it wrong costs the most: training data, feature stores, streaming, and model governance. Maturity note: this is where Level 1 binding starts to pay for itself in audit survival.

---

## The question with no answer

Maya's worst meeting happened in a glass-walled room on the fourth floor, and it lasted eleven minutes. Meridian's first production model that touched clients — a suitability-and-eligibility scorer her team had supplied half the features for — was up for review by the model-risk committee. She had her dashboards. She had her quality metrics. She had diagrams. Then the head of model risk read a question off her notes, slowly, in the voice of someone who already knew the answer would not come: *"Maya, of the seventy-three features this model consumes, which were sourced under a lawful basis that includes automated decision-making, and which were not?"*

Maya knew which catalog to look in. She knew the consent register existed somewhere in the privacy team's SharePoint. She knew her engineers could probably trace it given a week or two. What she could not do was answer the question in the room, or know whether the answer today matched the answer six months ago when the model was trained. She said, "I'll come back to you on that." The committee paused the model. Six months of engineering, three vendors, two regulatory submissions, and a head of data's reputation — paused. Not because the data was bad. Because the data could not produce evidence about itself.

That is the difference this chapter is about. The data feeding Meridian's model was, by every measure the team tracked, fine: complete, accurate, fresh, statistically well-behaved. It passed every quality gate it was asked to pass. What it could not do was answer, on demand, the questions an AI workload and its regulators actually ask — about meaning, lineage, lawful basis, and what the data looked like on the specific day a model trained on it. Batch tables at rest, governed as capsules, are necessary but not sufficient. The capsule discipline has to extend into the places where ML data actually lives: training snapshots, feature stores, and streams in motion. Those workloads have different constraints and higher consequences, and most MLOps tooling does not cover the gap.

## The gap MLOps leaves open

Ask an ML team basic questions about a production model — what data trained it, what version of that data, what quality checks it passed, which features were included and what transformations produced them — and you tend to get answers that are vague, partial, or require archaeology through old notebooks and pipeline logs. This is not a failure of the ML engineers. It is a failure of the data infrastructure beneath them.

Feature-engineering logic scatters across notebooks, SQL scripts, Spark jobs, and ad-hoc Python. There is rarely a single source of truth for what a feature means or how it is computed. Reproducibility becomes aspirational: "just re-run the training notebook" does not work when the upstream tables have changed, the feature definitions have drifted, or someone quietly fixed a transformation bug three months ago. When a model misbehaves in production, debugging means tracing back through data that may no longer exist in its original form. When a regulator asks about bias, the team scrambles to reconstruct what data influenced what decisions.

Tools like MLflow, Weights & Biases, Kubeflow, and SageMaker do an excellent job — on the *model* side. They track experiments, model versions, hyperparameters, deployment metadata. But they track the model side of the equation. The data side stays underserved, and the data side is where Maya's eleven-minute meeting went wrong. Data Capsules fill that gap by bringing the same versioning, lineage, and governance discipline to training data and features that Chapters 5 and 6 brought to analytical tables.

## Training data as a capsule

A training dataset is not a table you query once and forget. It is a critical input to a system that will make decisions affecting clients, revenue, and risk, and it deserves the full capsule treatment — all seven components, with the emphasis shifted to reflect the stakes.

- **Structural metadata** documents not just column names but what each field represents and how it relates to the prediction target.
- **Semantic metadata** pins down feature meaning precisely: does `client_tenure_months` count from account opening or first funded trade? What does a null mean? These details decide whether someone can correctly reuse the feature two years later.
- **Quality expectations** become distribution thresholds, missing-value limits, and label-accuracy requirements — and ideally checks that *fail before training proceeds* rather than notes that are read afterward.
- **Policy metadata** carries PII classification, consent basis, and approved use. *This is the component that would have saved Maya's meeting.* Had the lawful basis for each feature — including whether it covered automated decision-making — been bound to the feature and versioned with it, "which of the seventy-three" would have been a query, not a postponement.
- **Lineage** records which source tables, transformations, and filters produced the training set, so unexpected behaviour traces to a root cause.
- **Operational contract** names the owner, the refresh process, and how changes are requested.

The discipline that makes all of this real is one rule: **training-data capsules are immutable snapshots tied to model versions.** When you train model v2.3, you record that it was trained on `client_suitability_training` snapshot `2026-03-15T00:00:00Z`. That snapshot is frozen. If anyone later asks what data produced that model, you retrieve exactly that version — not whatever the table looks like today. Delta, Iceberg, and Hudi all support this snapshotting natively; the capsule discipline simply *requires that you use it*, every training run recording the specific snapshot it consumed, that reference becoming a permanent part of the model's record.

## Feature stores through the capsule lens

Feature stores — Feast, Tecton, Databricks Feature Store, Vertex AI Feature Store — exist precisely because teams recognised that feature engineering was a mess. They centralise feature definitions, serve features consistently for training and inference, and add some discoverability. In effect they are *proto-capsules*: they already bundle data with metadata — feature schemas, descriptions, entity definitions, freshness. But most implementations stop short of the full model, and the parts they skip are the parts that bite.

**Policy metadata is usually missing.** A feature store tracks what features exist but rarely whether a feature contains PII, what consent basis applies, or which models are approved to consume it. At Meridian, a feature like `client_postcode_district` looks innocuous until it is combined with others in a way that could proxy for a protected characteristic in an eligibility decision. That policy context has to travel with the feature.

**Lineage to raw sources is usually shallow.** The store documents the feature, but often not the full path from raw data through intermediate transformations. When a custodian changes the semantics of a source field, you need to know which features — and therefore which models — are affected, not merely that the features exist.

**Quality expectations beyond schema are usually absent.** A feature can have the right type and the wrong distribution. If `trades_per_quarter` suddenly shows 80% zeros when it historically showed 20%, that is a data-quality event that should block training; most feature stores will not catch it.

Bringing capsule discipline to a feature store means treating each feature group as a full capsule: defining quality tests that *gate feature availability* so a degraded feature is quarantined rather than silently served; attaching policy tags that *control consumption* so `income_band` requires additional approval before use in a credit decision; and tracing lineage from raw data through every transformation so a source change immediately identifies the affected features and models. This also resolves the online/offline consistency problem that haunts feature stores: the capsule definition becomes the contract both serving paths must honour. If the batch (offline) and real-time (online) paths diverge in schema, semantics, or quality, you do not have a reliable feature — you have two different things wearing the same name.

## Model governance and the data side of MLOps

Model cards have become the standard way to document a model — intended use, performance, limitations, ethical considerations. But model cards typically treat training data as a black box. "Trained on client transaction data" is not enough for serious governance, and it is certainly not enough for the model-risk committee that paused Maya's model.

The capsule completes the picture: the model card documents the model; the capsule documents what fed it. The connection should be explicit and permanent. When a model is registered, the registration includes references to the training-data capsule versions — not "client_transactions table" but "`client_transactions`, snapshot `2026-03-15`, schema version 4, quality suite v2.1 passed, lawful-basis manifest v7." That single discipline buys three capabilities Meridian did not previously have.

**Audit trails that actually work.** When a regulator asks what data trained the suitability model, you answer with a capsule reference that retrieves the exact data with full lineage, quality results, and policy documentation. No forensic reconstruction. The eleven-minute meeting becomes a query against a frozen, evidenced artifact.

**Drift detection with meaningful baselines.** Production inference data is compared against the training capsule's recorded distributions. The capsule's quality expectations — "`portfolio_value` between 0 and 50m, mean around 480k" — are the baseline for detecting when live data diverges from training conditions, rather than a number someone has to remember.

**Defensible retraining decisions.** When an upstream capsule changes — new schema, new source, new quality rule — every model that consumes it can be flagged for review automatically. You discover the problem when the *data* changes, not three months later when the model's predictions have quietly degraded on a feature that no longer means what it meant at training time.

## Lineage for streaming and ML pipelines

Lineage is easy when data sits in tables and jobs run on schedules. It is hard when data flows continuously and models train on snapshots of a moving target — which is exactly Meridian's intraday position pipeline. Consider the chain behind a single suitability feature:

Raw position events land in a Kafka topic. A CDC job captures custodian database changes into another topic. A Flink job joins and transforms these streams into a feature table (the Hudi capsule from Chapter 6). A batch job snapshots that feature table for model training. The trained model deploys and serves predictions to the adviser copilot.

Every step has its own tooling and its own lineage mechanism, and the value of capsule-anchored lineage is *connecting them into one chain*: raw topic → CDC stream → feature table → training snapshot → model version. When a prediction is challenged or performance degrades, you trace backward through the whole chain. This is not just operational convenience; it is a compliance requirement for high-stakes ML. The EU AI Act demands transparency about training data for high-risk systems; financial regulators want audit trails for decisions that affect clients. With capsule-anchored lineage, "which models were trained on data derived from this Kafka topic?" is a graph query. Without it, you are back to archaeology — which is to say, back to "I'll come back to you on that."

Streaming reframes what the capsule *is*, slightly. A Hudi capsule fed by a stream is not a static table but a sequence of validated state transitions, each commit a checked step. The capsule's quality gate runs at the commit boundary (the pre-commit validator from Chapter 6), so the stream cannot advance into an invalid state. Data in motion is still data with context bound to it; the binding just happens per-commit rather than per-batch.

## Doing this without strangling experimentation

A fair objection: ML development is exploratory. Data scientists iterate fast through feature ideas, architectures, and hyperparameters. Impose heavyweight metadata on every experimental dataset and you smother the very iteration that makes ML work. The resolution is to **tier the capsule requirements to the stakes.**

Exploratory work in a development environment might require only structural metadata and basic lineage — enough to know what a thing is and where it came from, no more. As a dataset matures toward production training, progressively enforce semantic documentation, quality gates, and policy classification. Reserve the *full* capsule treatment for datasets that will train production models. At Meridian, the rule is explicit and risk-tiered: a model that influences a client-facing suitability or eligibility decision gets full capsule rigour from the first training run; an internal next-best-action recommender for the marketing team can operate with lighter metadata until it earns more. Right-sizing governance to actual risk is not a compromise of the discipline — it is the discipline applied with judgement.

Three ML-specific subtleties deserve a flag, because they do not arise as sharply in batch analytics:

- **Temporal complexity.** A feature like `client_lifetime_value` computed over rolling windows produces different values for the same entity on different dates; point-in-time labels may be revised later. ML capsules must record not just what the data contains but *when it was valid* — computation timestamps, label observation dates, the temporal semantics of each feature — or you can retrieve the data without knowing whether it is appropriate for a given training task.
- **Label provenance.** In supervised learning, labels are data too, and often the worst-behaved data. They come from human annotators (with disagreement), production systems (with noise), or proxy metrics (with systematic bias). Capsules should document how labels were generated, the annotation process, and the estimated accuracy — and version label corrections like any other data change.
- **Cross-functional ownership.** Capsule discipline across ML requires data engineering, ML engineering, and governance to agree on shared vocabulary and ownership boundaries before tooling can help. Who defines a feature's semantics — the team that produces it or the team that consumes it? Who approves a training set's policy classification? These questions need answers, and they are operating-model questions, which is why they recur in Part V.

## Honest limits

Capsules are not a panacea, and pretending otherwise invites the backlash that kills disciplines. Four limits are worth stating plainly.

**Metadata completeness is aspirational.** You will never have perfect metadata for every dataset. Legacy training data lacks lineage; old feature definitions are ambiguous. The goal is progressive improvement prioritised by risk — high-stakes models first — not immediate perfection.

**Tooling fragmentation is real.** Feature stores, model registries, lineage systems, and catalogs each have their own metadata models, so capsules often mean building integration layers between tools never designed to cooperate. That is genuine, ongoing overhead. The return is governance capability you cannot otherwise get, but the cost should not be hand-waved. (Chapter 12 is largely about reducing this cost with open standards.)

**Versioning has a price.** Every immutable snapshot consumes storage; every lineage edge consumes graph capacity; every quality check consumes compute. At the scale of a feature store serving thousands of features across hundreds of models, this adds up. Most organisations find it acceptable — storage is cheap against regulatory risk and debugging time — but capacity planning should account for it rather than discover it.

**Human discipline is the hardest part.** Tools enforce schemas and run checks; they cannot force anyone to write a meaningful semantic description or think carefully about policy implications. The value of a capsule depends entirely on the quality of the metadata humans put in it, which makes adoption a cultural problem as much as a technical one — teams need to *experience* the payoff before they invest in the inputs.

That last point names the central adoption trap, and it has a known antidote: connect metadata to a tangible win early. Show how lineage cut a production incident from hours to minutes. Show how a quality gate caught a bad feature before it reached training. When a team feels the value firsthand — when someone avoids their own version of the eleven-minute meeting — metadata quality improves on its own. Capsule discipline imposed as bureaucracy produces fields filled in with minimal effort: technically complete, practically useless. Capsule discipline pulled by demonstrated value produces metadata people actually maintain.

## Where this leaves Meridian

Six months after the model was paused, Maya's team retrained the suitability scorer on capsule-governed training data. Each feature carried its lawful basis, versioned and bound. The training set was a frozen snapshot referenced from the model registry, with its quality suite and a lawful-basis manifest attached. When the model-risk committee reconvened and asked the same question — which features were sourced under a basis covering automated decision-making — Maya answered in the room, from a query, with evidence. The model was approved.

Nothing about the model had changed. What changed was that the data could now produce evidence about itself: meaning, lineage, lawful basis, and the exact state it was in on the day the model trained. That is the entire payoff of *Bind*, made visible at the moment it matters most.

This closes Part II. The capsule is defined (Chapter 5), it has a home in the lakehouse and the stream (Chapter 6), and it extends to the ML workloads where the stakes are highest (Chapter 7). But there is a quieter prerequisite running underneath all three chapters that we have leaned on without examining: none of this binding produces *meaning* unless the data underneath is *structured* to carry meaning in the first place. A capsule can bind a definition to a column, but if the column sits in a tangle of unmodelled, conflicting, source-shaped tables, the definition has nothing stable to attach to. Before we automate context in Part III, the next chapter addresses the structure that makes context possible at all: modelling for a world where the consumer is as likely to be an AI agent as an analyst.

> **Chapter summary.** MLOps tooling governs the model side and leaves the data side exposed — which is how good data still fails an audit. Extend capsules to ML: training data as immutable snapshots tied to model versions; feature groups as full capsules with policy, deep lineage, and distribution gates; model registrations that reference training-capsule versions for working audit trails, meaningful drift baselines, and defensible retraining. Capsule-anchored lineage connects the streaming-to-training chain into one graph query. Tier the rigour to risk so experimentation survives, and respect the honest limits — completeness is aspirational, tooling is fragmented, versioning costs, and human discipline is the real bottleneck.

> **Try this.** Pick one production (or soon-to-be-production) model. Try to answer, today, from a query rather than from people: what exact data version trained it, what quality it passed, and the lawful basis of each feature. The size of the gap between "I can query that" and "I'll come back to you" is your ML capsule backlog.
