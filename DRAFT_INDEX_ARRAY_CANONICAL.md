# Draft: Canonical Index Arrays

## Status

This note corrects the canonical array representation used by the Quaternion Exploration Core.

Semantic arrays must store indexes, not UUIDs.

UUIDs are internal row identifiers only.

Indexes are the only semantic reference keys.

---

# 1. Absolute Rule

```text
semantic reference = index only
semantic array     = index array only
UUID               = internal row identity / event identity only
```

Do not use UUIDs as semantic references.

Do not mix UUID and index references in the same semantic link.

Do not define a semantic array as `uuid[]`.

Do not define UUID-array columns in the canonical schema.

If evidence needs to mention multiple UUID identities, store them inside `evidence_json` with explicit naming.

---

# 2. Canonical Arrays

```text
token_array       = bigint[] of token.token_index
vocabulary_array  = bigint[] of vocabulary.vocabulary_index
grammar_array     = bigint[] of grammar.grammar_index
```

The array is the structural semantic vector.

The array elements are indexes.

---

# 3. UUID Rule

UUID columns may exist only as internal row identity or event/audit identity.

Examples:

```text
token.token_uuid
vocabulary.vocabulary_uuid
grammar.grammar_uuid
grammar_relation.grammar_relation_uuid
logs.current.current_uuid
```

They must not be used for:

- semantic arrays
- semantic near-neighbor search
- semantic relation paths
- semantic link position scoring
- Phase candidate output
- missing-slot completion
- canonical UUID-array columns

---

# 4. Index Rule

Index columns are the canonical semantic keys.

Examples:

```text
token.token_index
vocabulary.vocabulary_index
grammar.grammar_index
grammar_relation.grammar_relation_index
```

They are used for:

- arrays
- semantic links
- near-neighbor search
- hierarchy traversal
- Phase output
- reverse missing-slot completion
- derived structural vector indexes

---

# 5. Hierarchy

```text
token.token_index
→ vocabulary.token_array bigint[]
→ grammar.vocabulary_array bigint[]
→ grammar_relation.grammar_array bigint[]
→ phase_relation_candidate.grammar_array bigint[]
```

A decided grammar array determines lower layers by index traversal:

```text
grammar_index
→ grammar.vocabulary_array
→ vocabulary.token_array
→ token.raw
```

---

# 6. Link Tables

Semantic link tables must use indexes as their semantic keys.

Example:

```sql
create table grammar_relation_link (
  grammar_relation_index bigint not null references grammar_relation(grammar_relation_index),
  grammar_index bigint not null references grammar(grammar_index),
  position integer not null,

  primary key (grammar_relation_index, position)
);
```

No UUID column is required in semantic link tables.

If an implementation keeps UUIDs for internal tooling, they must not be treated as semantic references and must not appear in canonical examples.

---

# 7. Logs

Logs may identify events and records with UUIDs.

But semantic targets use indexes when the target belongs to a semantic table.

Canonical target form:

```text
target_table text
target_index bigint
```

Not:

```text
target_table text
target_uuid uuid
```

For non-semantic operational events, UUID event IDs may still be used as event identity.

Evidence that mentions multiple UUID identities should be placed inside `evidence_json`.

It must not become a canonical `uuid[]` column.

---

# 8. Decoherence Bank

Residual arrays use indexes:

```text
decoherence_bank.token_array bigint[]
decoherence_bank.vocabulary_array bigint[]
decoherence_bank.grammar_array bigint[]
```

The bank may still have its own `decoherence_uuid` as row identity.

That UUID is not a semantic reference.

---

# 9. Phase Candidate

Phase output is an index array:

```text
phase_relation_candidate.grammar_array bigint[]
```

The candidate row may have a UUID identity.

The semantic candidate is the `grammar_array` of grammar indexes.

Evidence record identities should be stored in `evidence_json`, not in a UUID-array column.

---

# 10. Derived Vector Index

Derived vector indexes must be generated from index arrays and aggregate fields.

Source of truth:

```text
index array
+ position
+ logs.current pressure
+ logs.diff history
```

The derived vector is only an acceleration index.

---

# 11. Correction Rule

Any existing draft text that describes semantic arrays as `uuid[]` is superseded by this rule.

Read those as `bigint[]` index arrays.

Any existing draft text that uses `target_uuid` for semantic table targets is superseded by this rule.

Read it as `target_index`.

Any evidence UUID list must be stored in JSONB evidence, not as a UUID-array column.

---

# 12. Short Form

```text
UUID is identity.
Index is meaning reference.
Array is index array.
Semantic reference never uses UUID.
Canonical schema does not define uuid[] columns.
Evidence UUID lists live inside JSONB evidence only.
```
