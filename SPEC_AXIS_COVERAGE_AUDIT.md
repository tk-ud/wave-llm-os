# Specification Axis Coverage Audit

This audit compares the former monolithic core on `main` against the split PR branch by four axes:

```text
functional axis
judgement axis
prohibition axis
behavior axis
```

Branch: `fix-aggregate-current-split`

Base: `main`

---

# Functional Axis

Functional axis means runtime functions and table families.

Covered by:

```text
SPEC_REPLY_PIPELINE.md
SPEC_CRON_PIPELINE.md
SPEC_SEMANTIC_TABLES.md
SPEC_LOG_AGGREGATE_ARCHIVE.md
SPEC_CORE_STATE_AND_SCHEDULER.md
SPEC_REMOTE_TRUST.md
SPEC_SCALE_AND_COST_MODEL.md
```

Coverage:

```text
reply path
cron path
semantic table family
log / aggregate / archive family
core state and scheduler family
remote event intake
mastication jobs
storage and cost model
```

Result:

```text
functional_axis: pass
```

---

# Judgement Axis

Judgement axis means rules that decide promotion, rejection, fallback, scoring, verification, and mutation permission.

Covered by:

```text
SPEC_OPERATION_GATE.md
SPEC_SCORING_AND_THRESHOLDS.md
SPEC_SEARCH_AND_VERIFICATION.md
SPEC_REPLY_PIPELINE.md
SPEC_CRON_PIPELINE.md
SPEC_LOG_AGGREGATE_ARCHIVE.md
```

Coverage:

```text
core_can_execute(operation_key)
freeze behavior
promotion evidence
phase relation thresholds
phase_score
draft_antipattern_score
moth_eaten_score
input grammar / grammar_relation diff
input subtraction
mirror-output evidence
logs.diff decision evidence
```

Result:

```text
judgement_axis: pass
```

---

# Prohibition Axis

Prohibition axis means explicit non-canonical patterns and forbidden interpretations.

Covered by:

```text
SPEC_PROHIBITED_CANONICAL_PATTERNS.md
SPEC_REFERENCE_MODEL.md
SPEC_OPERATION_GATE.md
SPEC_CANONICAL_CORE_RESUME.md
```

Coverage:

```text
no uuid[] semantic arrays
no status enum authority
no adopt.* operation keys
no constraint_rule
no adoption_audit
no web-result-only semantic table
no decoder_trace
no loop_guard
no direct remote semantic mutation
no semantic authority in logs / archives / notes
```

Result:

```text
prohibition_axis: pass
```

---

# Behavior Axis

Behavior axis means abstract runtime behavior names and their projections.

Covered by:

```text
SPEC_RUNTIME_BEHAVIOR_MODEL.md
SPEC_CANONICAL_CORE_RESUME.md
SPEC_REPLY_PIPELINE.md
SPEC_CRON_PIPELINE.md
SPEC_LOG_AGGREGATE_ARCHIVE.md
SPEC_SEARCH_AND_VERIFICATION.md
```

Coverage:

```text
wave
coherence
decoherence
pressure
Phase
Sleep
collapse
abstract runtime behavior -> concrete PostgreSQL projection
database rows are projections, not behavior authority
```

Result:

```text
behavior_axis: pass
```

---

# Overall Result

```text
axis_audit_status: pass
functional_axis: pass
judgement_axis: pass
prohibition_axis: pass
behavior_axis: pass
```

No material semantic compression is detected by the four-axis comparison.

DDL remains delegated to mapped NOTE files.
