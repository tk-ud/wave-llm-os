# Draft: Structural Near-Neighbor Search

## Status

This document defines the canonical near-neighbor search model for the Quaternion Exploration Core.

Near-neighbor search is structural.

It does not use external embedding vectors as semantic authority.

The ordered index arrays already are the semantic vectors.

For large-scale search, a derived vector index may be used as an acceleration layer.

The derived vector index is not semantic authority.

---

# 1. Core Principle

Canonical semantic hierarchy:

```text
token
→ vocabulary
→ grammar
→ grammar_relation
```

Each layer is represented by ordered bigint index arrays:

```text
vocabulary.token_array        bigint[] of token.token_index
grammar.vocabulary_array      bigint[] of vocabulary.vocabulary_index
grammar_relation.grammar_array bigint[] of grammar.grammar_index
phase_relation_candidate.grammar_array bigint[] of grammar.grammar_index
```

These arrays are not merely storage fields.

They are the structural semantic vectors of the system.

UUIDs are internal row identities only.

They are not semantic reference keys.

The meaning of a candidate is preserved by:

- which indexes are present
- their order
- their repetition
- their gaps
- their overlap with other arrays
- their observed coherence pressure
- their relation pressure
- their promotion/adoption history

---

# 2. Dense Vector Authority Is Rejected

`pgvector` or equivalent dense vector search must not become semantic authority.

Reason:

```text
Dense vectorization can erase structural meaning when treated as the source of truth.
```

The canonical core must preserve the hierarchy:

```text
token index array
→ vocabulary index array
→ grammar index array
→ grammar relation index array
```

External embedding vectors may collapse order, gaps, position, relation path, and table identity into a derived distance.

That conflicts with the canonical premise that meaning is connection form.

Canonical rule:

```text
Do not make dense embedding vectors semantic authority.
```

---

# 3. Derived Vector Index Is Allowed

Large-scale search may require stronger approximate-nearest-neighbor indexes.

A derived vector index is allowed if it is generated from canonical index arrays and aggregate fields.

Allowed role:

```text
candidate retrieval accelerator
```

Forbidden role:

```text
semantic source of truth
```

The source of truth remains:

```text
index array
+ position/link table
+ logs.current pressure
+ logs.diff history
+ status/adoption state
```

A derived vector may be stored in a read-model table or generated index table.

It must be reproducible from canonical index-array state.

If the derived vector disagrees with structural verification, structural verification wins.

---

# 4. Minimal Structure Is the Strongest Source of Truth

The minimal source structure is:

```text
index array + position + logs.current pressure + logs.diff history
```

This is sufficient because the arrays already encode the semantic path.

A separate vector layer is not required to define meaning.

The index array is the vector.

The link table is the positional index.

`logs.current` is the observed pressure field.

`logs.diff` is the mutation/adoption evidence field.

For scale, a derived vector index may accelerate first-pass retrieval, but it must be followed by structural scoring.

---

# 5. Canonical Near-Neighbor Inputs

Near-neighbor search may use:

```text
array overlap
ordered subsequence overlap
position distance
gap distance
prefix / suffix continuity
end_of_sentence_flag
status / draft_flag
relation_weight
logs.current coherence pressure
logs.current relation pressure
logs.diff promotion/adoption history
decoherence_bank pressure
derived vector index result, as candidate retrieval only
```

It must not require:

```text
embedding model output as meaning authority
dense semantic vector distance as final truth
spectral basis distance
wave_signature distance
```

---

# 6. Layered Near-Neighbor Functions

Near-neighbor search is layer-specific.

```text
core_find_near_vocabulary(...)
core_find_near_grammar(...)
core_find_near_grammar_relation(...)
core_find_near_phase_relation_candidate(...)
core_fill_missing_by_reverse_hierarchy(...)
```

Each function should preserve multiple candidates.

It must not collapse too early.

The recommended shape is two-stage search:

```text
1. retrieve candidate bundle quickly
2. verify and rank candidates structurally
```

The retrieval stage may use a derived vector index at scale.

The verification/ranking stage must use canonical index arrays, link positions, and aggregate pressure.

---

# 7. Vocabulary Near-Neighbor Search

Vocabulary near-neighbor search operates on `token_array bigint[]`.

Signals:

```text
token_index overlap
ordered token_index overlap
position distance
raw_text exact / partial match if present
split_flag / split_kind compatibility
logs.current pressure if available
derived vocabulary index as retrieval accelerator
```

The primary signal is structural token-index-array similarity.

Raw text similarity is secondary evidence only.

Derived vector retrieval is allowed only to gather a candidate bundle.

---

# 8. Grammar Near-Neighbor Search

Grammar near-neighbor search operates on `vocabulary_array bigint[]`.

Signals:

```text
vocabulary_index overlap
ordered vocabulary_index overlap
position distance
gap distance
end_of_sentence_flag compatibility
logs.current coherence_count
logs.current near_hit_count
logs.current partial_hit_count
logs.current end_of_sentence_score
derived grammar index as retrieval accelerator
```

Grammar search must not start from raw tokens if a grammar context is already available.

Derived vector retrieval must be followed by structural verification.

---

# 9. Grammar Relation Near-Neighbor Search

Grammar relation near-neighbor search operates on `grammar_array bigint[]`.

Signals:

```text
grammar_index overlap
ordered grammar_index overlap
relation path continuity
prefix / suffix continuity
relation_weight
logs.current relation_pressure
phase candidate pressure
logs.diff promotion history
derived relation index as retrieval accelerator
```

This is the main search space for relation completion and Phase promotion.

Derived vector retrieval must not replace grammar-index-array scoring.

---

# 10. Phase Candidate Near-Neighbor Search

Phase candidate search also operates on `grammar_array bigint[]`.

It compares `phase_relation_candidate.grammar_array` against existing `grammar_relation.grammar_array` and other Phase candidates.

Signals:

```text
grammar_array overlap
ordered grammar path similarity
source_current evidence
score
pressure
evidence_json selection counts
derived phase candidate index as retrieval accelerator
```

Promotion is allowed only when near-neighbor selection repeatedly identifies the candidate as useful and structural verification confirms the relation path.

---

# 11. Reverse Hierarchical Missing-Slot Completion

When a candidate contains a missing slot, completion proceeds from the highest available structure downward.

Order:

```text
grammar_relation / grammar_array
→ grammar
→ vocabulary
→ token
```

Rules:

- if grammar-array context exists, search missing grammar_index first
- if grammar cannot be resolved, search vocabulary_index candidates inside nearby grammar candidates
- if vocabulary cannot be resolved, search token_index candidates last
- do not start from raw token similarity when higher-level relation context exists

This preserves structural meaning.

---

# 12. Derived Index Table Pattern

A derived index table may be used for large-scale candidate retrieval.

Example shape:

```sql
create table structural_vector_index (
  index_uuid uuid primary key default gen_random_uuid(),

  target_table text not null,
  target_index bigint not null,

  index_kind text not null,
  -- vocabulary | grammar | grammar_relation | phase_relation_candidate

  source_hash text not null,

  derived_vector vector null,

  metadata_json jsonb null,

  created_at timestamptz not null default now(),
  updated_at timestamptz not null default now(),

  unique (target_table, target_index, index_kind)
);
```

Rules:

```text
source_hash must be derived from canonical index-array fields.
derived_vector is invalid if source_hash is stale.
structural verification must run after vector retrieval.
```

The derived vector may encode structural features such as array overlap signatures, positional features, relation path features, and aggregate pressure.

It must not be treated as independent meaning.

---

# 13. Example Scoring Shape

The exact formula is implementation-specific, but the final score should be structural.

Example grammar score:

```text
grammar_near_score =
  overlap_score
+ ordered_overlap_score
+ position_score
+ current_coherence_pressure
+ eos_compatibility_bonus
+ relation_context_bonus
+ derived_retrieval_prior
```

Example relation score:

```text
relation_near_score =
  grammar_array_overlap_score
+ grammar_order_score
+ relation_weight
+ current_relation_pressure
+ phase_candidate_pressure
+ promotion_history_bonus
+ derived_retrieval_prior
```

The derived retrieval prior can influence candidate ordering, but it must not override structural contradiction.

The score is not semantic truth.

It is candidate ordering evidence.

---

# 14. PostgreSQL Requirements

Canonical requirements:

```text
bigint[] index arrays
GIN indexes over arrays
link tables with position
JSONB for current/evidence fields
B-tree indexes for status/flags/weights
```

Optional raw-text helper:

```text
pg_trgm for raw_text similarity
```

Scale-oriented derived index option:

```text
pgvector as an acceleration index for derived structural vectors
```

`pg_trgm` may support raw text fallback, but it is not semantic authority.

`pgvector` may support large-scale candidate retrieval, but it is not semantic authority.

---

# 15. Non-Goals

Structural near-neighbor search is not:

- embedding search as meaning source
- dense vector similarity as final truth
- spectral search
- wave signature search
- raw text fuzzy matching as primary semantic authority
- early collapse to one nearest item

It is candidate selection over observed hierarchical index arrays.

At scale, derived vector indexes may accelerate candidate retrieval, but final ranking remains structural.

---

# 16. Short Form

```text
The index array is the semantic vector.
The hierarchy is the meaning.
The link table is the position index.
logs.current is the pressure field.
logs.diff is the mutation evidence.

pgvector may accelerate search over derived structural indexes.
It must not become semantic authority.

Vector retrieval proposes candidates.
Structural verification decides candidates.
```
