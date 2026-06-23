# Canonicalization Audit

This file records the migration from the previous monolithic core into distributed canonical specifications.

Branch: `fix-aggregate-current-split`

Base: `main`

---

# Audit Result

```text
status: pass_after_third_main_comparison
reason: canonical meaning, processing rules, routing, non-semantic runtime table families, and explicit prohibited patterns from main are represented in split SPEC files; SQL / DDL detail is preserved through mapped NOTE files
```

The PR-side split is semantically equivalent at the specification-authority layer.

DDL and SQL implementation detail may remain in NOTE files when the corresponding SPEC owns the meaning and authority.

---

# Canonicalized Files

```text
SPEC_CANONICAL_CORE_RESUME.md
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

---

# Coverage Map

## Core interpretation rules

Target: `SPEC_CANONICAL_CORE_RESUME.md`

Covered:

```text
repository-defined meanings win over common implementation patterns
current / promotion / verification / scheduler / draft / coherence / decoherence are routed SPEC-defined terms
canonical concepts must not be split just because common DB / queue / workflow patterns suggest it
```

## Prohibited canonical patterns

Target: `SPEC_PROHIBITED_CANONICAL_PATTERNS.md`

Covered:

```text
no canonical constraint_rule table
no canonical adoption_audit table
no canonical web-result-only semantic table
no canonical decoder_trace table
no canonical loop_guard table
no canonical status enum table
no adopt.* operation keys
promotion/deletion audit are logs.diff views or projections
web results are input_observation(source_kind='web')
looping is unresolved exploration pressure
```

## Reference model

Target: `SPEC_REFERENCE_MODEL.md`

Covered:

```text
UUID as row/event identity
index as semantic reference key
bigint[] as semantic structure
no uuid[] semantic arrays
hashes as deduplication, not authority
semantic target path convention
```

## Semantic tables

Target: `SPEC_SEMANTIC_TABLES.md`

SQL projection: `NOTE_SQL_TOKENIZATION_IMPLEMENTATION.md`

Covered:

```text
input_observation
token
vocabulary
grammar
grammar_relation
grammar_relation_link
phase_relation_candidate
decoherence_bank
structural_vector_index
source_kind list
layer_bundle_json shape
residual_kind meaning
```

## Core state and scheduler

Target: `SPEC_CORE_STATE_AND_SCHEDULER.md`

Covered:

```text
core_state
core_operation_policy
core_notify_queue
scheduler_job
scheduler_job_run
mastication_job
remote_event_inbox
remote_event_quarantine
scheduler job namespace separation
mastication output must obey input subtraction
remote inbox / quarantine are not semantic authority
```

## Logs / aggregate / archive

Target: `SPEC_LOG_AGGREGATE_ARCHIVE.md`

Covered:

```text
logs.coherence
logs.diff
logs.current as aggregate history
aggregate.current as hot pressure surface
archive.registry
promotion audit view meaning
deletion audit view meaning
retention / rollup policy
```

## Operation gate

Target: `SPEC_OPERATION_GATE.md`

Covered:

```text
core_can_execute(operation_key)
freeze behavior
canonical operation keys
promotion evidence rules
delete rules
sleep.decohere_structure rules
phase_relation default policy thresholds
```

## Scoring and thresholds

Target: `SPEC_SCORING_AND_THRESHOLDS.md`

Covered:

```text
phase_score formula
draft_antipattern_score formula
moth_eaten_score formula
moth-eaten default decision threshold
```

## Remote trust

Target: `SPEC_REMOTE_TRUST.md`

Covered:

```text
remote trust global switch
remote_node_trust fields
trust_level values
remote event acceptance rule
trusted nodes cannot mutate semantic tables directly
remote trust changes pass operation gate
```

## Reply path

Target: `SPEC_REPLY_PIPELINE.md`

Covered:

```text
reply generation is synchronous core work
not Phase Attention
near-neighbor retrieval
input grammar / grammar_relation diff verification
xi coherence hit / yj residual bank / zk decoder flow
immediate evidence-driven promotion
human review is not ordinary reply-time promotion
freeze / policy / contradiction failure behavior
decoherence fallback runtime
decoder / collapse boundary
```

## Cron path

Target: `SPEC_CRON_PIPELINE.md`

Covered:

```text
scheduler job kind namespace
current_refresh
sleep_consolidation
phase_candidate_generation
archive_rolloff
Phase as scheduled aggregate-weighted relation candidate generation
Draft as anti-pattern pressure
Sleep consolidation boundary
```

## Search / verification

Target: `SPEC_SEARCH_AND_VERIFICATION.md`

Covered:

```text
index-array search
near-neighbor search
pgvector as acceleration only
mandatory input grammar / grammar_relation diff
input subtraction
mirror-output evidence
fallback verification
operation-gated promotion
```

## Scale / cost

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

# Main Comparison Notes

The second comparison found and resolved:

```text
1. Canonical interpretation rules were made explicit in SPEC_CANONICAL_CORE_RESUME.md.
2. Non-semantic runtime table family was routed to SPEC_CORE_STATE_AND_SCHEDULER.md.
```

The third comparison found and resolved:

```text
1. Explicit negative / prohibited canonical table rules were routed to SPEC_PROHIBITED_CANONICAL_PATTERNS.md.
2. Prohibited decoder/status/adoption/web-result patterns were made explicit after core split.
```

---

# Remaining Non-blocking Notes

`NOTE_SQL_TOKENIZATION_IMPLEMENTATION.md` still contains historical wording that points to the former monolithic core.

This is not an authority conflict because `NOTE_SQL_IMPLEMENTATION_MAP.md` and `SPEC_CANONICAL_CORE_RESUME.md` state that SPEC files now own authority.

---

# Final Result

```text
merge_status: allowed_after_review
compression_status: no material semantic compression detected after third main comparison
DDL_status: delegated_to_notes
```
