---
name: rag-pgvector
description: "Yörünge's Android knowledge RAG pipeline — document ingestion and chunking, embeddings via app/services/vector_service.py, pgvector similarity search, hybrid retrieval, reranking, citation-grounded answers in the rag_qa agent, and retrieval evaluation. Use when working on knowledge.py, vector_service.py, rag_qa.py, embeddings, or indexing jobs."
---

# Android Knowledge RAG (pgvector)

Purpose: answer Kotlin/Compose/Android questions from **current** official documentation, so the mentor never advises a deprecated API. Freshness is the product feature; treat stale or unattributed answers as failures.

## Ingestion

- Sources: Android Developers docs, Kotlin docs, Jetpack release notes. Store `source_url`, `title`, `section_path`, `fetched_at` on every chunk — answers must be citable.
- Chunk on structure (heading → subsection), not a fixed character count. Target ~500–800 tokens with ~15% overlap, and never split a code block across chunks.
- Prepend the heading trail to the chunk text before embedding (`"Compose > Side-effects > LaunchedEffect\n\n..."`). It measurably improves retrieval on short chunks.
- Re-ingestion is idempotent: `UNIQUE (source_url, chunk_index)` with an upsert, plus a content hash so unchanged chunks skip re-embedding.
- Indexing runs as an Arq job, never in a request.

## Embeddings

- One model, pinned in `app/config.py`, with its dimension pinned to the `Vector(N)` column. Changing the model means a migration and a full re-index — there is no partial mix.
- Batch embed (e.g. 96 inputs per call) with retry + backoff; a partial batch failure must not leave half-indexed documents committed.
- Store the model name on each chunk so a future migration can find what needs re-embedding.

## Search

```sql
CREATE INDEX ON knowledge_chunks USING hnsw (embedding vector_cosine_ops);
```

```python
stmt = (
    select(KnowledgeChunk, KnowledgeChunk.embedding.cosine_distance(qvec).label("distance"))
    .order_by("distance")
    .limit(top_k)
)
```

- Cosine distance, matching the index opclass. Mixing `<->` (L2) with a cosine index silently degrades results.
- Retrieve wide (top_k ≈ 20), then rerank down to 4–6 chunks for the prompt. Stuffing 20 chunks costs tokens and lowers answer quality.
- Hybrid retrieval: combine vector search with Postgres full-text (`tsvector`) for exact API names like `rememberCoroutineScope` — pure embeddings miss rare identifiers.
- Apply a distance threshold. If nothing clears it, the agent says it doesn't have a grounded answer; it does not improvise.

## Answer generation (`rag_qa` agent)

- The prompt instructs: answer **only** from the provided context, cite `source_url` per claim, and state explicitly when the docs are silent.
- Return citations as structured data so the frontend can render them as links, not as text glued into the answer body.
- Never let the model paper over an empty retrieval — no context means "bilmiyorum, dokümanlarda bulamadım", not a plausible guess.

## Evaluation

Keep `backend/tests/rag/golden_set.yaml`: question → expected source URL(s). CI asserts recall@5 does not regress. Any change to chunking, the embedding model, top_k, or the prompt must be run against it and the numbers reported in the PR.

## Cost

Embedding calls are the cheap part; re-indexing everything on each deploy is not. Gate re-index behind an explicit command/job, never on application startup.
