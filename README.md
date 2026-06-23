# Wave LLMOS

Wave LLMOS treats observations, vocabulary, grammar, relations, residuals, pressure, and evidence as inspectable PostgreSQL projections of abstract runtime behavior.

Read direction:

```text
abstract runtime behavior
→ concrete inspectable PostgreSQL projection
```

DB rows are projections, not semantic authority by themselves.

---

# Canonical Specifications

The former monolithic core has been split into routing-level core and domain-specific canonical specs.

```text
SPEC_CANONICAL_CORE_RESUME.md
SPEC_REFERENCE_MODEL.md
SPEC_LOG_AGGREGATE_ARCHIVE.md
SPEC_REPLY_PIPELINE.md
SPEC_CRON_PIPELINE.md
SPEC_SEMANTIC_TABLES.md
SPEC_SEARCH_AND_VERIFICATION.md
SPEC_SCALE_AND_COST_MODEL.md
SPEC_CANONICALIZATION_AUDIT.md
```

## `SPEC_CANONICAL_CORE_RESUME.md`

Routing-level core.

Defines invariants, document routing, and section-level runtime resumes.

## `SPEC_REFERENCE_MODEL.md`

Defines UUID / index / `bigint[]` semantic reference rules.

## `SPEC_LOG_AGGREGATE_ARCHIVE.md`

Defines logs, `aggregate.current`, `logs.current`, archive registry, retention, and rollup policy.

## `SPEC_REPLY_PIPELINE.md`

Defines reply-time input, indexing, search, decoder projection, collapse, and evidence logging.

## `SPEC_CRON_PIPELINE.md`

Defines current refresh, Phase, Sleep, and archive rolloff.

## `SPEC_SEMANTIC_TABLES.md`

Defines the canonical semantic table family.

## `SPEC_SEARCH_AND_VERIFICATION.md`

Defines structural search, near-neighbor search, verification, and mirror-output evidence.

## `SPEC_SCALE_AND_COST_MODEL.md`

Defines storage scale, learning-volume assumptions, and per-reply cost shape.

## `SPEC_CANONICALIZATION_AUDIT.md`

Records the audit from the old monolithic core into distributed canonical specs.

---

# Notes

`NOTE_*` files are explanatory unless promoted into a `SPEC_*` file.

If a `NOTE_*` file conflicts with a `SPEC_*` file, the `SPEC_*` file wins.

Legacy material remains in `legacy_design/` as historical context.
