# Appendix E — Glossary

*Roughly sixty terms as the book uses them, each with a one-line definition and the chapter where it is developed. Where the book's usage is more specific than common usage, the definition reflects the book's.*

---

**Active metadata** — Metadata that acts rather than merely describes: event-driven, in the platform's control plane, stopping breaks and feeding agents in real time. (Ch 12)

**Agent (AI agent)** — An autonomous system that reasons, and increasingly *acts*, over data at machine speed with no human in the loop; the consumer that cannot supply its own context. (Ch 4)

**Agent-context API** — The endpoint an agent calls *before* answering, returning certification, trust, semantics, and entitlement for a product. (Ch 12)

**Agentic data engineering** — Data engineering in which agents build, monitor, and self-heal pipelines; the human role shifts to orchestration. (Epilogue)

**AI-ready** — Not "clean," but able to produce defensible *evidence* about itself on demand, for a specific use. (Ch 18)

**AsyncAPI** — Open standard for describing and versioning event-stream contracts; the streaming analogue of OpenAPI. (Ch 11)

**Automate** — The second move: making bound context versioned, tested, event-driven, and platform-maintained. (Part III)

**Bind** — The first move: attaching context to data as one co-versioned unit so the two cannot drift apart. (Part II)

**Bitemporal versioning** — Storing both *valid time* (true in the world) and *transaction time* (when recorded), so the past can be reconstructed as it was then understood. (Ch 13)

**Bronze / Silver / Gold** — The medallion layers: raw landing, integrated single-source-of-truth, consumption-optimised. (Ch 8)

**Capsule** — See *Data Capsule*.

**Capsule completeness score** — A per-asset 0–4 *engineering* score of how much of the binding discipline a product carries (no metadata → full capsule discipline); distinct from the book's maturity ladder, which measures meaning. (Ch 14)

**Certification** — A derived, use-case-specific assurance that a product is safe for a given AI use, at a level, on evidence, with named sign-off. (Ch 19, App C)

**CODEOWNERS** — The file that auto-assigns reviewers to changes by repository path; the mechanism behind the Okonkwo human check. (Ch 10, App B)

**Cold metadata** — Rich, occasionally-consulted metadata referenced by stable identifier rather than embedded. (Ch 6)

**Confidence context** — A machine-readable signal of how much to trust a product element (freshness, completeness, known issues). (Ch 15)

**Context data product** — A data product packaged with its meaning — ontology, semantic layer, knowledge graph, usage intelligence — so an agent can reason over it. (Ch 15)

**Controlled vocabulary** — An explicit, governed enumeration for a field whose free-text values an agent could misread. (Ch 17)

**Critical Data Element (CDE)** — A data element whose accuracy is material to a regulated outcome; flagged for heightened governance. (Ch 19)

**Data Capsule** — A logical unit binding a data payload with seven components of context (structural, semantic, quality, policy, provenance, contract) so they co-version and deploy as one. (Ch 5)

**Data contract** — The promise a product makes to consumers: schema, semantics, quality, freshness, and change policy; the capsule's outer envelope. (Ch 5–6)

**Data lake** — Centralised raw storage; the architecture whose "hoard now, mean later" mental model this book identifies as the constraint. (Ch 2)

**Data mesh** — A decentralised operating model (domain ownership, data as product, self-serve platform, federated governance); fails when treated as architecture, not operating model. (Ch 21)

**Data product** — Not a dataset with a README, but a contractual commitment: an interface, a named owner, a guarantee. (Ch 2)

**Data Product Manager** — The accountable owner of a data product, with roadmap, decision rights, and budget; the role most orgs have not staffed. (Ch 21)

**Data swamp** — A data lake into which data is poured with no metadata, ownership, or agreement on meaning; an organisational failure, not a storage one. (Ch 2)

**Data Vault 2.0** — A Silver-layer modelling style (hubs, links, satellites) for multi-source, audit-heavy integration. (Ch 8)

**DCAT** — W3C Data Catalog Vocabulary; the standard for describing datasets in the metadata graph. (Ch 13)

**Document capsule** — A capsule for unstructured data: source, content hash, lifecycle, permitted uses, and chunk/embedding lineage. (Ch 9)

**Document swamp** — The unstructured counterpart of the data swamp: a vector index into which documents are poured with no schema, grain, ownership, or lifecycle, retrieved from by agents at machine speed. (Ch 9)

**Drift** — The silent divergence of data from the separate stores of its meaning; the default behaviour of unbound metadata. (Ch 3)

**Embedding** — A vector representation of a text chunk; a derived artefact whose model version is part of the capsule (re-embedding is a version event). (Ch 9)

**Evidence (six surfaces)** — Ownership, metadata, quality, lineage, privacy, consumption: the six questions an AI-ready product must answer on demand. (Ch 18)

**Federated computational governance** — The mesh principle that decides what is governed globally (interoperability rules) versus locally (meaning, lifecycle). (Ch 21)

**FIBO** — Financial Industry Business Ontology; a reference ontology to *borrow patterns from*, not adopt wholesale. (Ch 16–17)

**Golden record** — The single authoritative version of a master entity (e.g. a client), produced by MDM survivorship rules. (Ch 8)

**Great Expectations** — Open framework for expressing data-quality rules as executable, deployment-gating assertions. (Ch 11)

**Groundedness** — Whether a generated answer actually follows from its retrieved sources; a RAG quality gate. (Ch 9)

**Guided access** — Consumption in which the product supplies the semantics, endpoints, grain, and prohibited uses an AI consumer needs, rather than leaving them to a skilled human; what a context data product provides. (Ch 15, 19)

**Hot metadata** — Metadata needed for query or immediate governance, embedded in the table/capsule. (Ch 6)

**Knowledge graph** — The populated, connected instance of an ontology, enriched with lineage and confidence; answers relationship questions natively. (Ch 13, 17)

**Lakehouse** — Object storage plus an open table format plus compute; the natural home for a capsule. (Ch 6)

**Lineage** — The traceable provenance of data; the *receipt* to quality's *claim*. Schema-level (diagram) vs instance-level (queryable) matters. (Ch 13, 20)

**Maturity ladder** — The book-wide 0–4 scale (Flat → Intelligent) measuring the four-move climb. (Ch 4, App D)

**MDM (Master Data Management)** — The discipline of a single golden record for core entities; governs a few entities, not all meaning. (Ch 1, 8)

**Medallion architecture** — See *Bronze / Silver / Gold*.

**Metadata as Code** — Managing metadata as declarative, version-controlled specs reconciled by CI/CD; GitOps for governance. (Ch 10)

**Minimum Viable Ontology (MVO)** — A bounded ontology (5–10 entities, relationships, rules) for a single domain and consumer. (Ch 16–17)

**Model card** — Documentation of a model's use, performance, and limits; incomplete without a reference to the training-data capsule. (Ch 7)

**ODCS (Open Data Contract Standard)** — The Bitol/LF AI & Data YAML standard for data contracts; the capsule is an ODCS-compatible superset. (Ch 5, App A)

**Ontology** — The formal model of a domain: what entities exist and how they relate, with cardinality and rules. (Ch 17)

**OpenLineage** — Open standard ("HTTP for lineage") for emitting job→dataset lineage events across tools. (Ch 11, 13)

**Package** — The third move: assembling bound, active context into a product an agent can reason over. (Part IV)

**Policy as code** — Access, masking, and quality policy defined alongside data definitions and enforced by the platform. (Ch 10)

**Prove** — The fourth move: turning packaged context into evidence and certification that survive an audit. (Part V)

**PROV-O** — W3C provenance ontology; describes which activity used and generated which dataset. (Ch 13)

**RAG (retrieval-augmented generation)** — Answering by retrieving documents/chunks and grounding a generated answer in them; the dominant enterprise-AI pattern, and a governance frontier. (Ch 9)

**Reconciliation (GitOps)** — The loop that detects and corrects drift between the Git spec and the live estate. (Ch 10)

**Semantic layer** — The governed mapping from business concepts to physical calculation; guarantees one definition per metric across all consumers. (Ch 17)

**Semantic versioning (for data)** — Major (breaking, incl. semantic redefinition) / minor (additive) / patch (correction). (Ch 5, App A)

**SHACL** — Shapes Constraint Language; validates metadata against rules at write time (e.g. every dataset must have an owner). (Ch 13)

**Slowly Changing Dimension (SCD)** — A pattern (usually Type 2) for tracking entity history with validity periods; enables point-in-time joins. (Ch 8)

**Survivorship** — The rules deciding which source wins, attribute by attribute, when sources conflict. (Ch 8)

**Trust score** — A dynamic, exposed measure of a product's current reliability (freshness, quality, ownership, incident recency) that gates consumption. (Ch 12)

**Usage intelligence** — The accumulated knowledge of how a product is used well: query patterns, limitations, examples. (Ch 15)

**Vector store / index** — The serving layer for embeddings; governed via metadata filters that enforce currency, entitlement, and policy at retrieval. (Ch 9)

**Write–audit–publish** — A pattern that holds data in an audit zone until it passes contract and quality checks before publishing. (Ch 12)
