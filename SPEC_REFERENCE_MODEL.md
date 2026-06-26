# Specification: Reference Model

## Authority

This file is the canonical authority for semantic identity and reference rules.

It canonizes the implementation-relevant parts of `NOTE_INDEX_ARRAY_CANONICAL.md` and the reference rules previously embedded in `SPEC_CANONICAL_CORE.md`.

`NOTE_INDEX_ARRAY_CANONICAL.md` remains explanatory background after this file exists.

---

# Core Rule

```text
UUID  = internal row / event identity only
index = semantic reference key
array = bigint[] index array only
```

Semantic references must use numeric index values.

Semantic arrays must be `bigint[]` arrays of index values.

UUID values must not be used as semantic reference arrays.

Evidence UUIDs may appear in JSONB evidence only.

---

# Identity Layers

Wave separates row identity from semantic reference identity.

```text
row identity       = uuid
semantic identity  = generated bigint index
semantic structure = ordered bigint[] index array
```

A UUID identifies an internal row or event.

An index identifies a semantic object in the canonical reference graph.

An array stores ordered semantic structure.

---

# Canonical Reference Chain

Canonical semantic references follow this hierarchy:

```text
token.token_index
→ vocabulary.token_array bigint[]
→ vocabulary.vocabulary_index
→ grammar.vocabulary_array bigint[]
→ grammar.grammar_index
→ grammar_relation.grammar_array bigint[]
→ grammar_relation.grammar_relation_index
→ phase_relation_candidate.grammar_array bigint[]
```

The array is the inspectable semantic structure.

The generated index is the stable reference key for the row produced from that structure.

---

# Allowed Semantic Array Columns

Canonical semantic array columns include:

```text
vocabulary.token_array

grammar.vocabulary_array

grammar_relation.grammar_array

grammar_relation_link.position ordered reference

phase_relation_candidate.grammar_array

phase_relation_candidate.relation_array

logs.coherence.target_path

logs.coherence.input_grammar_path

logs.coherence.candidate_path

logs.diff.target_path

logs.current.target_path

aggregate.current.target_path

decoherence_bank.target_path
```

All semantic array columns are `bigint[]` unless explicitly defined as a scalar index or ordered link table.

No canonical semantic `uuid[]` array is defined.

---

# Hashes

Hashes identify structural equality or deduplication keys.

```text
source_hash       = observation source identity
vocabulary_hash   = token-array / vocabulary-shape deduplication
grammar_hash      = vocabulary-array / grammar-shape deduplication
relation_hash     = grammar-array / relation-shape deduplication
residual_hash     = unresolved residual-shape deduplication
```

A hash is not a semantic reference key.

A hash may decide whether a structure already exists.

The generated index remains the semantic reference key after the row exists.

---

# Reference Use by Table Family

## Semantic Tables

Semantic tables expose generated index keys.

```text
token.token_index
vocabulary.vocabulary_index
grammar.grammar_index
grammar_relation.grammar_relation_index
decoherence_bank.decoherence_index
phase_relation_candidate.phase_relation_candidate_index
```

## Logs

Logs must identify targets with:

```text
target_table
target_index
target_path
```

`target_index` is used when the target is a single known semantic row.

`target_path` is used when the target is a structural path or compound semantic reference.

## Aggregate Identity Projection

`aggregate.current` and `logs.current` may use `target_identity_key` to enforce current-surface uniqueness across both single semantic targets and compound semantic paths.

`target_identity_key` is an implementation identity projection only.

It is not a semantic reference key.

Canonical semantic references remain `target_index` or `target_path`.

If `target_identity_key` is implemented as an encoded index or stable target-path hash, the hash rule still applies: the hash may support equality or uniqueness, but it must not replace the generated index or bigint[] path as semantic reference authority.

## Archive Registry

Archive metadata may store row-index ranges, timestamps, hashes, and storage manifests.

Archive registry rows do not become semantic references.

---

# Decoder and Collapse Boundary

Decoder output may use UUIDs in internal evidence JSON for traceability.

Collapsed semantic references must use index or index-array form.

Public or downstream semantic references must not depend on UUID identity.

---

# Prohibited Patterns

```text
uuid[] semantic arrays
UUID as semantic path component
hash as semantic reference key
status enum as semantic lifecycle authority
archive row as semantic authority
log row as semantic authority
```

These patterns are non-canonical.

---

# Migration Rule

If a legacy note or draft uses UUIDs as semantic reference identifiers, reinterpret the UUID as row identity only.

Migrate semantic references to generated index keys.

If a legacy structure stores a semantic path as UUIDs, create or resolve the corresponding index path before using it as a canonical semantic path.

---

# Short Form

```text
UUIDs identify rows.
Indexes identify semantic objects.
Arrays store semantic structure.
Hashes deduplicate structures.
Logs record evidence.
Archives record moved history.
Only indexes and bigint[] paths are canonical semantic references.
```
