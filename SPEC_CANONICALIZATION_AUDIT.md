# Canonicalization Audit

This file records the migration from the previous monolithic core into distributed canonical specifications.

Branch: `fix-aggregate-current-split`

---

# Canonicalized Files

```text
SPEC_REFERENCE_MODEL.md
SPEC_LOG_AGGREGATE_ARCHIVE.md
SPEC_REPLY_PIPELINE.md
SPEC_CRON_PIPELINE.md
SPEC_SEMANTIC_TABLES.md
SPEC_SEARCH_AND_VERIFICATION.md
SPEC_SCALE_AND_COST_MODEL.md
```

---

# Source Coverage Map

## Core reference rules

Source: `SPEC_CANONICAL_CORE.md`, `NOTE_INDEX_ARRAY_CANONICAL.md`

Target: `SPEC_REFERENCE_MODEL.md`

Covered:

```text
UUID as row/event identity
index as semantic reference key
bigint[] as semantic structure
no uuid[] semantic arrays
hashes as deduplication, not authority
```

## Logs / aggregate / archive

Source: `SPEC_CANONICAL_CORE.md`, aggregate-current amendment

Target: `SPEC_LOG_AGGREGATE_ARCHIVE.md`

Covered:

```text
logs.coherence
logs.diff
logs.current as aggregate history
aggregate.current as hot pressure surface
archive.registry
retention / rollup policy
```

## Reply path

Source: `SPEC_CANONICAL_CORE.md`, `NOTE_SQL_TOKENIZATION_IMPLEMENTATION.md`, `NOTE_NEAR_NEIGHBOR_SEARCH.md`

Target: `SPEC_REPLY_PIPELINE.md`

Covered:

```text
input_observation
token
vocabulary
grammar
coherence search
structural verification
decoder projection
collapse
evidence logging
```

## Cron path

Source: `SPEC_CANONICAL_CORE.md`, `NOTE_PHASE_RELATION_CANDIDATE.md`

Target: `SPEC_CRON_PIPELINE.md`

Covered:

```text
current_refresh
aggregate.current update
logs.current snapshot
phase_candidate_generation
sleep_consolidation
archive_rolloff
```

## Semantic tables

Source: `SPEC_CANONICAL_CORE.md`, `NOTE_SQL_TOKENIZATION_IMPLEMENTATION.md`

Target: `SPEC_SEMANTIC_TABLES.md`

Covered:

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

## Search / verification

Source: `SPEC_CANONICAL_CORE.md`, `NOTE_NEAR_NEIGHBOR_SEARCH.md`

Target: `SPEC_SEARCH_AND_VERIFICATION.md`

Covered:

```text
index-array search
near-neighbor search
pgvector as acceleration only
structural verification
input subtraction
mirror-output evidence
fallback verification
operation-gated promotion
```

## Scale / cost

Source: README discussion and design notes

Target: `SPEC_SCALE_AND_COST_MODEL.md`

Covered:

```text
raw observation growth
sublinear canonical semantic growth
relation/residual growth
hot log bounds
storage-scale intelligence
per-reply search cost shape
```

---

# Compression Audit

No source concept is intentionally dropped.

Some local SQL column detail from the previous monolithic core is summarized in the first canonicalization pass to keep the new specs small.

Those details are preserved by table family and field-role mapping rather than copied as one monolithic schema block.

Known compression points to expand later if needed:

```text
full SQL DDL for every semantic table
full SQL DDL for logs.coherence and logs.diff
full SQL DDL for aggregate.current and logs.current
full scheduler_job / scheduler_job_run columns
full remote event intake / quarantine columns
```

These are not considered semantic loss because their table roles and authority destinations are preserved.

---

# Audit Result

```text
status: pass_with_compressed_ddl
reason: semantic authority and routing preserved; some DDL details intentionally delegated for later expansion
old core may be replaced by a section-level resume after this audit
```
