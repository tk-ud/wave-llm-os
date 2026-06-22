# Supporting Note: Structural Near-Neighbor Search

Canonical authority: `DRAFT_CANONICAL_CORE.md`.

---

# Rule

Near-neighbor search is structural over index arrays.

```text
index array = semantic vector
hierarchy   = meaning
```

Primary structures:

```text
vocabulary.token_array
grammar.vocabulary_array
grammar_relation.grammar_array
phase_relation_candidate.grammar_array
```

All are `bigint[]` index arrays.

---

# Scoring Inputs

Use:

```text
array overlap
ordered overlap
position / gap distance
prefix / suffix continuity
logs.current pressure
logs.diff history
decoherence_bank pressure
status / draft_flag / relation_weight
```

---

# pgvector

Allowed:

```text
pgvector as derived retrieval accelerator
```

Rejected:

```text
pgvector as semantic authority
```

Flow:

```text
pgvector retrieves candidates
structural verification decides
```

---

# Missing Slot Completion

```text
grammar_array
→ grammar_index search
→ vocabulary_index search
→ token_index search
```

Do not start from raw tokens when higher-level context exists.
