# Chapter 8A — Capsules for Unstructured Data and RAG

> **Where you are:** Part II — *Bind*, extension chapter. Chapters 5–8 bound context to structured data — tables, features, streams. But the data most AI consumes in 2026 is neither structured nor tabular; it is *documents and embeddings*. This chapter extends the capsule discipline to unstructured data and the retrieval-augmented systems that consume it. It is the archetype Chapter 4 named the "RAG hallucination," addressed. Maturity note: it brings the same Level 1 binding to the half of the estate the earlier chapters left ungoverned.

---

## The answer from the withdrawn note

Six months after the Okonkwo error, Meridian's adviser copilot gave another confident, wrong answer — and this time no table was to blame. An adviser asked, "What's our house view on emerging-market debt for a conservative client right now?" The copilot retrieved a research note, summarised it fluently, and recommended a modest allocation. The note it retrieved had been *superseded three months earlier* — the house view had since turned cautious on EM debt after a sovereign downgrade — but the withdrawn note was still sitting in the vector index, still semantically similar to the query, still retrievable, with nothing recording that it had been withdrawn. The copilot did not know the note was stale. It knew the note was *relevant*. Relevance and currency are different properties, and the index tracked only the first.

This is the RAG hallucination archetype from Chapter 4, in the flesh, and it is worth dwelling on why the whole apparatus of Part II so far would not have caught it. `client_holdings` was a capsule. The suitability model trained on a capsule. The position stream was a capsule. And none of it touched the research note, because the research note is not a row in a table — it is a PDF, chunked into passages, embedded into vectors, and stored in an index that no capsule governed. The most consequential data Meridian's copilot consumes is *unstructured*, and the discipline of the last four chapters, for all its rigour, stopped at the edge of the structured estate.

That edge is where most enterprises now live. The copilots, assistants, and agents being built in 2026 are overwhelmingly retrieval-augmented: they answer by retrieving documents — policies, research notes, contracts, manuals, tickets, emails — and grounding a generated answer in them. The data they consume is text, and the pipeline that prepares it (chunk, embed, index) is a data pipeline like any other, with all the same ways to drift, mislead, and leak. If Part II governs only the tabular half of the estate, it has left the AI-facing half — the half the agent actually reads — completely ungoverned. This chapter closes that gap.

## Why unstructured data breaks every rule so far

Everything in Chapters 5–8 rested on properties that a document simply does not have, and naming the mismatches is the fastest way to see why unstructured data needs its own treatment rather than a hand-wave.

**No schema.** A table has columns with types; a document has prose. There is no `em_exposure_pct` field to bind a definition to — the meaning is diffuse, spread across paragraphs, implicit. The structural-metadata component of the capsule, which for a table is the schema, has to be reconceived for a document, because there is no shape to enforce.

**No grain.** A table has a grain — one row per client per holding per date. A document has no natural unit until you *impose* one by chunking it, and the chunk boundaries are a modelling decision that changes what the AI can retrieve and reason over. The grain of unstructured data is not given; it is manufactured, and manufacturing it well is part of the discipline.

**No obvious owner.** A holdings table has a domain and a product owner. A research note has an author, a publishing desk, a compliance reviewer, and a shelf life — and in most organisations, once published, it has no *data* owner at all. It drifts into a shared drive and is forgotten, which is precisely how a superseded note stays retrievable.

**No update semantics.** When a table row changes, the capsule co-versions and the change is detectable. When a document is superseded, corrected, or withdrawn, what happens to the chunks derived from it, the embeddings derived from the chunks, and the cached answers derived from the embeddings? In most RAG systems: *nothing*. The correction never propagates, because nothing binds the derivatives to the source.

Put these together and you get the phenomenon that is to the AI era what the data swamp was to the analytics era: the **document swamp**. A vast, growing, unstructured store — SharePoint, Confluence, shared drives, ticket systems — into which documents are poured with no schema, no grain, no ownership, and no lifecycle, and out of which an AI agent now retrieves at machine speed to ground consequential answers. The lake accepted any data without oversight and became a swamp; the vector index accepts any document without oversight and becomes the same thing, except that its consumer is an agent that acts on what it finds. Chapter 2's warning was about analysts starting from scratch; this version is about agents answering from withdrawn guidance.

## The document capsule

The capsule discipline generalises, and the generalisation is the fix. A **document capsule** binds a source document together with the context that makes it safely consumable, co-versioned and lifecycle-managed, exactly as a table capsule does — with the seven components reconceived for unstructured content.

```yaml
document_capsule: house_view_em_debt
version: 4.0.0
source:
  uri: s3://meridian-research/house-views/em-debt-2026-01.pdf
  content_hash: sha256:9f2a...        # binds THIS payload; a new PDF = new version
  format: pdf
  authored_by: research-emd@meridian.example
lifecycle:
  status: superseded                  # draft | published | superseded | withdrawn
  effective_from: "2026-01-15"
  effective_to: "2026-04-02"          # the day the cautious view replaced it
  superseded_by: house_view_em_debt@5.0.0
semantics:
  topic: "emerging-market debt house view"
  entities: [asset_class:EQ-EM, risk_appetite]
  summary: "Constructive on EM debt for moderate+ risk; see v5 for revised view"
classification: internal
permitted_uses:
  - adviser_copilot_retrieval           # ONLY when status == published
prohibited_uses:
  - client_facing_verbatim_quotation
retrieval:
  chunking: { strategy: heading_aware, target_tokens: 400, overlap: 40 }
  embedding_model: "text-embed-3-large@2025-11"
  index: research_emd_v4
lineage:
  chunks: 18
  openlineage_namespace: meridian.rag
```

Read what this binds that a bare document in a drive does not. The **content hash** binds *this specific PDF* to the capsule, so replacing the file is a version event, not a silent swap — co-versioning for unstructured payloads means the hash changes when the bytes change. The **lifecycle block** is the fix for the opening disaster: `status: superseded`, `effective_to`, and `superseded_by` are machine-readable facts a retrieval system can *filter on*, so a withdrawn note can be excluded from retrieval the instant it is withdrawn. The **permitted_uses** state that copilot retrieval is allowed *only when published* — policy travelling with the document. And the **retrieval block** records the chunking strategy and, crucially, the *embedding-model version*, because — as the next section argues — the embedding is a derived artefact whose provenance matters as much as the data's.

The document capsule does for the research note what `client_holdings` did for the holdings: it makes the document a governed, owned, lifecycle-managed product rather than a file in a swamp. And it gives the retrieval system the one fact it was missing — that relevance is not currency, and here is how to tell them apart.

## Chunking and embedding are transformations with lineage

The step most RAG systems treat as invisible plumbing — turn documents into chunks, chunks into embeddings, embeddings into index entries — is in fact a *data pipeline*, with inputs, transformations, and outputs, and it deserves lineage exactly as a Spark job does. Making that pipeline legible is what lets you answer, when an agent gives a bad answer, *which chunk of which version of which document, embedded by which model, produced it.*

Model the pipeline as three tracked transformations, each emitting OpenLineage events:

- **Document → chunks.** Chunking is a modelling decision, not a mechanical split. Heading-aware chunking preserves the document's structure so a chunk carries its section's context; naive fixed-size chunking severs a sentence from the heading that gave it meaning. Each chunk inherits its parent capsule's identity and lifecycle — a chunk of a superseded note is itself superseded — so the chunk carries `parent: house_view_em_debt@4.0.0` and can be filtered on it.
- **Chunks → embeddings.** Each chunk is embedded by a specific model version, and *the embedding-model version is part of the capsule*, because embeddings from different models are not comparable and a model upgrade silently changes retrieval behaviour. This is the subtle one: **re-embedding a corpus with a new model is a version event**, like reprocessing a table with new logic, and it must be recorded, because a query answered against a v2-embedded index is not answered against the same knowledge as a v1-embedded index even if the documents are identical.
- **Embeddings → index entries.** Each vector lands in the index with its metadata — the parent capsule id, lifecycle status, effective dates, classification, division scope — attached. This metadata is not decoration; it is the enforcement surface the next section is built on.

The payoff of tracking this chain is the same as for structured lineage, sharpened by the stakes: when the copilot grounds an answer in a chunk, the full provenance — this chunk, of this document version, embedded by this model, effective on these dates — is a query, not an archaeology. "Which answers were grounded in the withdrawn EM-debt note?" becomes a graph traversal, exactly as "which models trained on this Kafka topic?" did in Chapter 7. Retrieval-augmented generation without this lineage is generation you cannot audit, and generation you cannot audit is the RAG archetype waiting to fire.

## The vector store is a governed serving layer

Here is the reframe that changes how you build RAG: **the vector index is not a search cache; it is a serving layer, and an ungoverned serving layer is an uncontrolled consumer.** Everything Chapter 8 said about the serving table applies to the index, plus a new enforcement mechanism unique to retrieval.

That mechanism is the **metadata filter**. Every vector in a well-built index carries the metadata the chunk inherited from its capsule, and the retrieval query filters on that metadata *before* similarity ranking. This is where policy stops being a document and becomes runtime enforcement:

```python
results = index.search(
    query_embedding,
    filter={
        "lifecycle_status": "published",        # exclude superseded/withdrawn
        "effective_from": {"<=": today},
        "effective_to":   {">=": today},        # currency, not just relevance
        "classification": {"in": caller.clearance},
        "division_scope": {"in": [caller.division, "all"]},
    },
    top_k=8,
)
```

Read the filter and every clause is a control that the opening disaster lacked. `lifecycle_status: published` is the single line that would have kept the withdrawn note out of the copilot's answer. The effective-date range enforces *currency* — the property the naive index conflated with relevance. The classification and division filters enforce *entitlement*: a retail adviser's copilot cannot retrieve institutional-only research, and a user without clearance cannot retrieve confidential chunks, no matter how semantically similar they are to the query. **An index without these filters is an uncontrolled consumer** — it will serve any chunk to anyone who asks a similar-enough question, which for a regulated firm is a data-leak and a mis-advice incident with the same root cause: retrieval that ranks on similarity alone.

The discipline, then, is that a vector index is provisioned and governed like any other serving product: it has an owner, a contract (what it will and will not return, its freshness, its filters), and a capsule per source document behind it. Retrieval is *guided access*, in exactly the sense Chapter 4 and Part IV mean it — the index guides the agent to current, permitted, in-scope chunks, because the agent cannot supply that judgement itself.

## Freshness and revocation: the hardest lineage problem in RAG

The document capsule's lifecycle block promises that a correction propagates; delivering on that promise is genuinely hard, and it is worth being honest that this is the frontier where RAG governance is least mature. The difficulty is the *derivation chain*. A source document is corrected or withdrawn. But the document has been chunked, the chunks embedded, the embeddings indexed, and answers may have been cached. A correction at the source has to propagate through all four derivative layers, and most systems propagate it through none.

Two cases, both of which Meridian has to handle:

**Supersession.** The EM-debt house view is replaced. The moment the new version is published, the old capsule flips to `status: superseded` — and because that status is a metadata field on every chunk and every vector derived from it, the retrieval filter (`lifecycle_status: published`) *immediately* stops the old chunks from being retrieved, without re-embedding anything. This is the elegant case: because lifecycle status was bound as filterable metadata, revocation is a metadata update, not a re-indexing project. The withdrawn note becomes unretrievable the instant it is withdrawn.

**Erasure (right to be forgotten).** A client exercises a GDPR erasure right, and their personal data must be removed from everywhere — including any chunk of any document that mentions them, the embeddings of those chunks, and any cache. This is the hard case, because erasure means *deletion*, not filtering, and it must reach every derivative. The document capsule's lineage — which chunks came from which document, which vectors from which chunks — is what makes this tractable: erasure becomes a traversal that finds and deletes every derived artefact, with the capsule as the anchor. Without that lineage, erasure in a RAG system is a hope, and "we think we removed it" is not an answer a regulator accepts. Caches are the trap here: an answer generated last week and cached may contain the very data just erased, so caches must carry TTLs short enough, or invalidation hooks precise enough, that erased content cannot survive in them.

The principle underneath both: **in a RAG system, a correction is only as propagable as the lineage from source to served answer.** The document capsule exists to make that lineage explicit, so that supersession is a filter update and erasure is a bounded traversal, rather than either being a manual sweep of a swamp.

## Evaluation is the quality gate for RAG

Structured capsules gate on quality expectations — uniqueness, ranges, distributions. Unstructured capsules need an equivalent, and it is *evaluation*: automated checks that the retrieval-and-generation behaves, run as gates in CI and in production, the Great-Expectations equivalent for RAG. Two families matter.

**Retrieval-quality gates.** Does the system retrieve the *right* chunks for a known set of questions? Meridian maintains a golden set of representative queries with the chunks that *should* ground their answers, and measures retrieval precision and recall against it on every change to the corpus, the chunking, or the embedding model. A change that drops retrieval quality below threshold fails the build — the same discipline as a distribution check failing a table deployment. Crucially, this catches the silent regression that re-embedding can cause: swap the embedding model and the golden-set score moves, which is how you learn that a "routine" model upgrade changed what your copilot can find.

**Groundedness gates.** Does the generated answer actually follow from the retrieved chunks, or has the model confabulated beyond them? Groundedness (or faithfulness) checks — increasingly automatable with model-graded evaluation — flag answers that assert more than their sources support. For a regulated adviser copilot, an answer that is fluent but not grounded in a current, permitted source is precisely the failure mode to block, and a groundedness gate is the mechanism. Paired with the currency filter, it closes the loop: retrieve only current, permitted chunks, and answer only what those chunks support.

These evaluations are not a one-off acceptance test; they are continuous, versioned alongside the corpus and the retrieval config, and wired into the same CI and active-metadata machinery as everything else in the book. A RAG system without evaluation gates is a RAG system whose quality is asserted, not evidenced — which, by the argument of Part V, means it is not AI-ready however good its demos look.

## Where this leaves Meridian, and Part II

Run the opening scene again with the discipline in place. The EM-debt house view is a document capsule; when the cautious view replaces it, the old capsule flips to `superseded` and every chunk derived from it inherits the status. The adviser asks the copilot for the house view; the retrieval filter excludes superseded and out-of-effective-range chunks, so the withdrawn note is never retrieved; the copilot grounds its answer in the *current* view, and the groundedness gate confirms the answer follows from it. No withdrawn guidance, no confident wrong answer, no mis-advice with a compliance tail. The fix was not a better model. It was, once again, binding context to data — this time to the document, the chunk, and the vector — so the AI consumes current, permitted, evidenced context rather than whatever was merely similar.

This completes the *Bind* move for the whole estate, structured and unstructured alike. Part II has now given Meridian capsules for tables (Chapter 6), for training data and features and streams (Chapter 7), for the models that make meaning stable (Chapter 8), and for documents and embeddings (this chapter) — the full surface an AI consumer actually reads. What none of it has yet solved is *scale*: all of this binding has been described as something a team does deliberately, dataset by dataset, document corpus by document corpus. The moment Meridian has hundreds of capsules across both halves of the estate, the binding has to become something the platform maintains automatically — versioned, tested, event-driven, enforced. That is the second move, and Part III begins it.

> **Chapter summary.** The data most AI consumes in 2026 is unstructured — documents and embeddings — and Chapters 5–8 left it ungoverned, producing the "document swamp": a vector index into which content is poured with no schema, grain, ownership, or lifecycle, and out of which an agent retrieves consequential answers. Unstructured data breaks the capsule's assumptions (no schema, no grain, no owner, no update semantics), so the capsule is reconceived as a **document capsule** binding source, content hash, lifecycle (published/superseded/withdrawn with effective dates and `superseded_by`), semantics, permitted uses, and the chunking/embedding config. Chunking and embedding are *transformations with lineage* (and re-embedding is a version event); the **vector index is a governed serving layer** whose metadata filters enforce currency, entitlement, and policy at retrieval time — an unfiltered index is an uncontrolled consumer. Freshness and erasure propagate only as far as the lineage allows, so the capsule anchors both. **Evaluation** (retrieval-quality and groundedness gates) is the quality gate for RAG. This completes *Bind* for the whole estate.

> **Try this.** Point at the corpus your most important AI assistant retrieves from and ask: could a superseded or withdrawn document still be retrieved today, and would you know which answers it had grounded? If the index has no lifecycle metadata to filter on, you are one supersession away from the opening scene — and the document capsule's lifecycle block is the first thing to add.
