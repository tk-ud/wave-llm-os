# Specification: Semantic Tables

This file is the canonical authority for semantic table meanings.

SQL shape and DDL detail are preserved in `NOTE_SQL_TOKENIZATION_IMPLEMENTATION.md`.

The note is an implementation projection; this SPEC owns semantic authority.

Canonical table family:

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

## input_observation

Shared intake table for user, web, file, remote_event, manual, and system sources.

Source differences are metadata, not separate semantic authority.

## token

Canonical token-level seed table.

Tokenization is indexing and segmentation, not semantic interpretation.

## vocabulary

Stores ordered token-index arrays.

`vocabulary.token_array` is semantic structure.

`vocabulary_hash` deduplicates stable token sequences or raw text.

## grammar

Stores ordered vocabulary-index arrays.

`grammar.vocabulary_array` is semantic structure.

## grammar_relation

Canonical coherence relation layer.

Stores ordered semantic continuity between grammar candidates.

Relation is not optional metadata.

## grammar_relation_link

Optional ordered relation link projection.

Preserves position and role metadata for grammar_relation paths.

## phase_relation_candidate

Stores scheduled draft relation candidates produced by Phase.

Phase outputs grammar_index arrays only.

Promotion remains operation-gated.

## decoherence_bank

Stores unresolved, no-hit, low-hit, partial-hit, unused, unstable, moth-eaten, mirror-output, and manual-delete-candidate evidence.

It is searchable fallback structure, not trash.

It preserves vocabulary, grammar, and relation context together through layer_bundle_json.

## structural_vector_index

Optional retrieval accelerator.

It is not semantic authority.

Semantic references are defined by `SPEC_REFERENCE_MODEL.md`.

Log and aggregate tables are defined by `SPEC_LOG_AGGREGATE_ARCHIVE.md`.
