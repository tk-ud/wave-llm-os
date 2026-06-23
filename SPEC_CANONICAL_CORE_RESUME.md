# Specification: Canonical Core Resume

## Authority

This file replaces the previous monolithic `SPEC_CANONICAL_CORE.md` as the routing-level canonical core.

It defines canonical routing, invariants, and section-level flow.

Detailed authority lives in distributed `SPEC_*` files.

Implementation SQL and DDL detail may live in mapped `NOTE_*` files.

---

# Core Invariants

```text
UUID  = internal row / event identity only
index = semantic reference key
array = bigint[] index array only
```

`draft_flag` remains the canonical lifecycle truth.

Operation-capable semantic mutation must pass the operation gate.

Logs, aggregate tables, archive registries, and implementation notes are not semantic authority.

---

# Document Routing

```text
Reference model         -> SPEC_REFERENCE_MODEL.md
Log / aggregate/archive -> SPEC_LOG_AGGREGATE_ARCHIVE.md
Reply pipeline          -> SPEC_REPLY_PIPELINE.md
Cron pipeline           -> SPEC_CRON_PIPELINE.md
Semantic tables         -> SPEC_SEMANTIC_TABLES.md
Search / verification   -> SPEC_SEARCH_AND_VERIFICATION.md
Scale / cost model      -> SPEC_SCALE_AND_COST_MODEL.md
Canonicalization audit  -> SPEC_CANONICALIZATION_AUDIT.md
SQL implementation map  -> NOTE_SQL_IMPLEMENTATION_MAP.md
```

---

# SQL / NOTE Wiring

```text
SPEC_REFERENCE_MODEL.md
-> NOTE_INDEX_ARRAY_CANONICAL.md

SPEC_SEMANTIC_TABLES.md
-> NOTE_SQL_TOKENIZATION_IMPLEMENTATION.md

SPEC_REPLY_PIPELINE.md
-> NOTE_SQL_TOKENIZATION_IMPLEMENTATION.md
-> NOTE_NEAR_NEIGHBOR_SEARCH.md

SPEC_CRON_PIPELINE.md
-> NOTE_PHASE_RELATION_CANDIDATE.md

SPEC_SEARCH_AND_VERIFICATION.md
-> NOTE_NEAR_NEIGHBOR_SEARCH.md

SPEC_SCALE_AND_COST_MODEL.md
-> NOTE_QUATERNION_PHILOSOPHY.md

SPEC_LOG_AGGREGATE_ARCHIVE.md
-> NOTE_SQL_TOKENIZATION_IMPLEMENTATION.md
-> SPEC_CANONICAL_CORE_AMENDMENT_001_AGGREGATE_CURRENT.md
```

`SPEC_*` files own meaning and processing authority.

`NOTE_*` files may preserve SQL, DDL, implementation sketches, and explanatory detail.

If a NOTE conflicts with a SPEC, the SPEC wins.

---

# Reply Path Resume

Reply Path handles input-time semantic search and output collapse.

```text
input_observation
-> token
-> vocabulary
-> grammar
-> coherence search
-> structural verification
-> relation or residual handling
-> decoder projection
-> collapse
-> evidence logs
```

Detailed authority: `SPEC_REPLY_PIPELINE.md`.

---

# Cron Path Resume

Cron Path handles scheduled maintenance, pressure refresh, relation candidate generation, Sleep consolidation, and archive rolloff.

```text
current_refresh
-> aggregate.current upsert
-> optional logs.current snapshot
-> phase_candidate_generation
-> sleep_consolidation
-> archive_rolloff
```

Detailed authority: `SPEC_CRON_PIPELINE.md`.

---

# Semantic Table Resume

Canonical semantic table family:

```text
input_observation
token
vocabulary
grammar
grammar_relation
grammar_relation_link
decoherence_bank
phase_relation_candidate
structural_vector_index
```

Detailed authority: `SPEC_SEMANTIC_TABLES.md`.

---

# Log / Aggregate / Archive Resume

```text
logs.coherence     = append-only observation evidence
logs.diff          = mutation / decision evidence
logs.current       = aggregate snapshot history / time-series evidence
aggregate.current  = hot aggregate read model / pressure surface
archive.registry   = archive batch registry and restore manifest
```

Detailed authority: `SPEC_LOG_AGGREGATE_ARCHIVE.md`.

---

# Search / Verification Resume

Canonical search uses semantic index arrays first.

Near-neighbor or vector search may accelerate retrieval, but verification must return to structural references.

Mirror-output, residual, fallback, and promotion evidence must be logged.

Detailed authority: `SPEC_SEARCH_AND_VERIFICATION.md`.

---

# Scale / Cost Resume

Wave does not require unbounded hot logs.

Hot-path intelligence depends on:

```text
current semantic structures
aggregate.current pressure
searchable fallback structures
recent decision evidence
operation-gated promotion results
```

Canonical semantic growth is expected to become sublinear under repeated observations because existing vocabulary, grammar, and relation structures are reinforced.

Detailed authority: `SPEC_SCALE_AND_COST_MODEL.md`.

---

# Notes After Canonicalization

`NOTE_*` files are implementation or explanatory notes unless explicitly promoted into a `SPEC_*` file.

If a `NOTE_*` file conflicts with a `SPEC_*` file, the `SPEC_*` file wins.

`NOTE_AGENT_METAPHOR_MAPPING.md` and `NOTE_QUATERNION_PHILOSOPHY.md` remain explanatory interpretation notes.

---

# Deletion Rule

The previous monolithic `SPEC_CANONICAL_CORE.md` may be removed after:

```text
1. distributed specs are created
2. canonicalization audit is recorded
3. this section-level resume exists
```

Those conditions are satisfied by this file and `SPEC_CANONICALIZATION_AUDIT.md`.
