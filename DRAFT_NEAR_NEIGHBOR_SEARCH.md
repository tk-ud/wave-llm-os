# Draft: Structural Near-Neighbor Search

## Status

This document defines the canonical near-neighbor search model for the Quaternion Exploration Core.

Near-neighbor search is structural.

It does not use external embedding vectors as semantic authority.

The ordered arrays already are the semantic vectors.

---

# 1. Core Principle

Canonical semantic hierarchy:

```text
token
→ vocabulary
→ grammar
→ grammar_relation
```

Each layer is represented by ordered UUID arrays:

```text
vocabulary.token_array
grammar.vocabulary_array
grammar_relation.grammar_array
phase_relation_candidate.grammar_array
```

These arrays are not merely storage fields.

They are the structural semantic vectors of the system.

The meaning of a candidate is preserved by:

- which elements are present
- their order
- their repetition
- their gaps
- their overlap with other arrays
- their observed coherence pressure
- their relation pressure
- their promotion/adoption history

---

# 2. Rejection of External Vector Authority

`pgvector` or equivalent dense embedding search is not canonical.

Reason:

```text
Dense vectorization can erase structural meaning.
```

The canonical core must preserve the hierarchy:

```text
token array
→ vocabulary array
→ grammar array
→ grammar relation array
```

External embedding vectors may collapse order, gaps, position, relation path, and table identity into a derived distance.

That conflicts with the canonical premise that meaning is connection form.

Canonical rule:

```text
Do not make dense embedding vectors semantic authority.
```

---

# 3. Minimal Structure Is the Strongest Structure

The minimal structure is:

```text
UUID array + position + logs.current pressure + logs.diff history
```

This is sufficient because the arrays already encode the semantic path.

A separate vector layer is not required to make the system meaningful.

The array is the vector.

The link table is the positional index.

`logs.current` is the observed pressure field.

`logs.diff` is the mutation/adoption evidence field.

---

# 4. Canonical Near-Neighbor Inputs

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
```

It must not require:

```text
pgvector
embedding model output
dense semantic vector distance
spectral basis distance
wave_signature distance
```

---

# 5. Layered Near-Neighbor Functions

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

---

# 6. Vocabulary Near-Neighbor Search

Vocabulary near-neighbor search operates on `token_array`.

Signals:

```text
token overlap
ordered token overlap
position distance
raw_text exact / partial match if present
split_flag / split_kind compatibility
logs.current pressure if available
```

The primary signal is structural token-array similarity.

Raw text similarity is secondary evidence only.

---

# 7. Grammar Near-Neighbor Search

Grammar near-neighbor search operates on `vocabulary_array`.

Signals:

```text
vocabulary overlap
ordered vocabulary overlap
position distance
gap distance
end_of_sentence_flag compatibility
logs.current coherence_count
logs.current near_hit_count
logs.current partial_hit_count
logs.current end_of_sentence_score
```

Grammar search must not start from raw tokens if a grammar context is already available.

---

# 8. Grammar Relation Near-Neighbor Search

Grammar relation near-neighbor search operates on `grammar_array`.

Signals:

```text
grammar overlap
ordered grammar overlap
relation path continuity
prefix / suffix continuity
relation_weight
logs.current relation_pressure
phase candidate pressure
logs.diff promotion history
```

This is the main search space for relation completion and Phase promotion.

---

# 9. Phase Candidate Near-Neighbor Search

Phase candidate search also operates on `grammar_array`.

It compares `phase_relation_candidate.grammar_array` against existing `grammar_relation.grammar_array` and other Phase candidates.

Signals:

```text
grammar_array overlap
ordered grammar path similarity
source_current_array evidence
score
pressure
evidence_json selection counts
```

Promotion is allowed only when near-neighbor selection repeatedly identifies the candidate as useful.

---

# 10. Reverse Hierarchical Missing-Slot Completion

When a candidate contains a missing slot, completion proceeds from the highest available structure downward.

Order:

```text
grammar_relation / grammar_array
→ grammar
→ vocabulary
→ token
```

Rules:

- if grammar-array context exists, search missing grammar first
- if grammar cannot be resolved, search vocabulary candidates inside nearby grammar candidates
- if vocabulary cannot be resolved, search token candidates last
- do not start from raw token similarity when higher-level relation context exists

This preserves structural meaning.

---

# 11. Example Scoring Shape

The exact formula is implementation-specific, but the score should be structural.

Example grammar score:

```text
grammar_near_score =
  overlap_score
+ ordered_overlap_score
+ position_score
+ current_coherence_pressure
+ eos_compatibility_bonus
+ relation_context_bonus
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
```

The score is not semantic truth.

It is candidate ordering evidence.

---

# 12. PostgreSQL Requirements

Canonical requirements:

```text
uuid[] arrays
GIN indexes over arrays
link tables with position
JSONB for current/evidence fields
B-tree indexes for status/flags/weights
```

Optional raw-text helper:

```text
pg_trgm for raw_text similarity
```

`pg_trgm` may support raw text fallback, but it is not semantic authority.

`pgvector` is rejected as canonical near-neighbor authority.

---

# 13. Non-Goals

Structural near-neighbor search is not:

- embedding search
- dense vector similarity
- spectral search
- wave signature search
- raw text fuzzy matching as primary semantic authority
- early collapse to one nearest item

It is candidate selection over observed hierarchical arrays.

---

# 14. Short Form

```text
The array is the vector.
The hierarchy is the meaning.
The link table is the position index.
logs.current is the pressure field.
logs.diff is the mutation evidence.

Dense embedding vectors are not canonical because they erase structural meaning.
```
