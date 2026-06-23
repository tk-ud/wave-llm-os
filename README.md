# Wave LLMOS

Wave LLMOS treats observations, vocabulary, grammar, relations, residuals, pressure, and evidence as inspectable PostgreSQL projections of abstract runtime behavior.

```text
abstract runtime behavior
-> concrete inspectable PostgreSQL projection
```

DB rows are projections, not semantic authority by themselves.

---

# Canonical Specifications

```text
SPEC_CANONICAL_CORE_RESUME.md
SPEC_RUNTIME_BEHAVIOR_MODEL.md
SPEC_REFERENCE_MODEL.md
SPEC_SEMANTIC_TABLES.md
SPEC_LOG_AGGREGATE_ARCHIVE.md
SPEC_OPERATION_GATE.md
SPEC_SCORING_AND_THRESHOLDS.md
SPEC_REMOTE_TRUST.md
SPEC_CORE_STATE_AND_SCHEDULER.md
SPEC_PROHIBITED_CANONICAL_PATTERNS.md
SPEC_REPLY_PIPELINE.md
SPEC_CRON_PIPELINE.md
SPEC_SEARCH_AND_VERIFICATION.md
SPEC_SCALE_AND_COST_MODEL.md
NOTE_SQL_IMPLEMENTATION_MAP.md
```

`SPEC_*` files own semantic and processing authority.

`NOTE_*` files are explanatory or implementation projections unless promoted into a `SPEC_*` file.

If a `NOTE_*` file conflicts with a `SPEC_*` file, the `SPEC_*` file wins.

Legacy material remains in `legacy_design/` as historical context.
