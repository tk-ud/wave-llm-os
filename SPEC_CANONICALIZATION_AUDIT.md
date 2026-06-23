# Canonicalization Audit

This file records the migration from the previous monolithic core into distributed canonical specifications.

Branch: `fix-aggregate-current-split`

Base: `main`

---

# Audit Result

```text
status: fail_requires_expansion
reason: PR-side split preserves routing intent, but several canonical details from main are compressed or missing
```

The split direction is valid.

The current PR-side specifications are not yet semantically equivalent to the monolithic core on `main`.

---

# Preserved Without Material Compression

```text
SPEC_REFERENCE_MODEL.md
```

Preserved:

```text
UUID as row/event identity
index as semantic reference key
bigint[] as semantic structure
no uuid[] semantic arrays
hashes as deduplication, not semantic authority
semantic target path convention
```

---

# Compressed / Missing Areas

## Semantic table DDL

Source details from main:

```text
input_observation full DDL and source_kind list
token full DDL
vocabulary full DDL
grammar full DDL
grammar_relation full DDL
grammar_relation_link full DDL
phase_relation_candidate full DDL
decoherence_bank full DDL
layer_bundle_json shape
residual_kind list
```

Current target:

```text
SPEC_SEMANTIC_TABLES.md only lists the table family
```

Required action:

```text
Move full SQL shape into NOTE_SQL_TOKENIZATION_IMPLEMENTATION.md or a SQL implementation note.
Keep semantic authority in SPEC_SEMANTIC_TABLES.md.
```

## Operation gate

Source details from main:

```text
core_can_execute(operation_key)
freeze behavior
canonical operation keys
promotion evidence rules
sleep.decohere_structure rules
delete.semantic_structure rules
delete.decoherence_entry rules
policy defaults for phase_relation
```

Current target:

```text
Only general operation-gated mutation wording is preserved
```

Required action:

```text
Create SPEC_OPERATION_GATE.md or expand SPEC_REPLY_PIPELINE.md / SPEC_CRON_PIPELINE.md.
```

## Scoring and thresholds

Source details from main:

```text
phase_score formula
draft_antipattern_score formula
moth_eaten_score formula
moth-eaten default decision threshold
```

Current target:

```text
SPEC_SCALE_AND_COST_MODEL.md mentions cost shape but does not preserve score formulas
```

Required action:

```text
Create SPEC_SCORING_AND_THRESHOLDS.md or move formulas into SPEC_SCALE_AND_COST_MODEL.md.
```

## Reply path details

Source details from main:

```text
reply generation is synchronous core work
not Phase Attention
near-neighbor retrieval
input grammar / grammar_relation diff verification
xi coherence hit / yj residual bank / zk decoder flow
immediate evidence-driven promotion
human review is not ordinary reply-time promotion
freeze / policy / contradiction failure behavior
```

Current target:

```text
SPEC_REPLY_PIPELINE.md is a very compressed 11-line skeleton
```

Required action:

```text
Expand SPEC_REPLY_PIPELINE.md with the full reply-path rules from main.
```

## Input grammar / relation diff

Source details from main:

```text
mandatory diff before collapse / promotion / reinforcement / decoherence hit promotion
shared grammar indexes
missing grammar slots
extra relation path elements
replaced vocabulary slots
replaced grammar slots
order shifts
scope boundary mismatch
terminal flag mismatch
required decision pattern
```

Current target:

```text
SPEC_SEARCH_AND_VERIFICATION.md preserves the category but not the decision detail
```

Required action:

```text
Expand SPEC_SEARCH_AND_VERIFICATION.md.
```

## Corpus output / input subtraction

Source details from main:

```text
candidate output - input grammar
empty or reorder-only delta = mirror_output evidence
store output as difference, relation, or residual, not raw echo
```

Current target:

```text
Partially preserved as mirror-output category only
```

Required action:

```text
Expand SPEC_SEARCH_AND_VERIFICATION.md or SPEC_REPLY_PIPELINE.md.
```

## Decoherence runtime

Source details from main:

```text
active search first
decoherence fallback search after active miss
verification before reuse
promote.decoherence_hit path
sleep cannot hard-delete
manual/operator delete only
```

Current target:

```text
Partially preserved, but not full runtime path
```

Required action:

```text
Expand SPEC_REPLY_PIPELINE.md and SPEC_CRON_PIPELINE.md.
```

## Phase details

Source details from main:

```text
cron-like scheduled aggregate-weighted relation candidate generation
Draft as anti-pattern pressure
may read decoherence_bank as Draft source
children remain draft_flag=true
Phase outputs grammar_index arrays only
not synchronous reply path
promotion only through phase_score and operation gate
```

Current target:

```text
SPEC_CRON_PIPELINE.md preserves a compressed version only
```

Required action:

```text
Expand SPEC_CRON_PIPELINE.md.
```

## Decoder / collapse boundary

Source details from main:

```text
decoder does not originate meaning
no independent constraint table
no decoder_trace table
no loop_guard table
looping is unresolved exploration pressure
repeated grammar path handled by grammar_array, Phase, near-neighbor, decoherence, pressure, diff evidence
```

Current target:

```text
Partially preserved in SPEC_REPLY_PIPELINE.md and SPEC_REFERENCE_MODEL.md
```

Required action:

```text
Expand SPEC_REPLY_PIPELINE.md.
```

## Remote trust / remote event intake

Source details from main:

```text
remote_node_trust DDL
core_state.distributed_sync.enabled
trust_level values
remote event acceptance rule
trusted nodes cannot mutate semantic tables directly
remote trust changes pass operation gate
```

Current target:

```text
Missing from split specs
```

Required action:

```text
Create SPEC_REMOTE_TRUST.md or add to SPEC_CRON_PIPELINE.md.
```

---

# Conclusion

```text
PR branch is structurally correct but not yet information-equivalent to main.
```

Do not merge until compressed sections are expanded or explicitly delegated to SQL/implementation notes.

Recommended next action:

```text
1. Add DDL / SQL detail to NOTE_SQL_TOKENIZATION_IMPLEMENTATION.md or mapped SQL notes
2. Add SPEC_OPERATION_GATE.md
3. Add SPEC_SCORING_AND_THRESHOLDS.md
4. Expand SPEC_REPLY_PIPELINE.md
5. Expand SPEC_CRON_PIPELINE.md
6. Expand SPEC_SEARCH_AND_VERIFICATION.md
7. Add SPEC_REMOTE_TRUST.md
8. Re-run audit and change status only when coverage is equivalent
```
