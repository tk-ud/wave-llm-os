# Specification: Scoring and Thresholds

This file is the canonical authority for inspectable score formulas and policy thresholds.

Scores are inspectable aggregate calculations, not neural parameters.

Implementations may tune policy values through core_operation_policy, but must preserve input evidence and output score components.

## Phase score

```text
phase_score = clamp01(
  0.25 * recurrence_score
+ 0.20 * coherence_score
+ 0.20 * relation_density_score
+ 0.15 * scope_crossing_score
+ 0.10 * decoder_success_score
+ 0.10 * output_delta_score
- 0.20 * contradiction_penalty
- 0.20 * mirror_output_penalty
- 0.15 * draft_antipattern_penalty
)
```

## Draft anti-pattern score

```text
draft_antipattern_score = clamp01(
  0.35 * similarity_to_unresolved_draft
+ 0.25 * mirror_output_rate
+ 0.20 * contradiction_rate
+ 0.20 * unresolved_slot_rate
- 0.30 * fresh_coherence_score
)
```

Candidate generation rule:

```text
new candidate
+ draft_antipattern_score >= 0.60
+ fresh_coherence_score < 0.30
-> reduce phase_score or reject candidate
```

## Moth-eaten score

Moth-eaten detection is based on repeated missing or replaced slots inside an otherwise active parent structure.

```text
missing_rate = missing_slot_count / greatest(parent_structure_usage_count, 1)
replaced_rate = replaced_slot_count / greatest(parent_structure_usage_count, 1)
slot_missing_or_replaced_rate = (missing_slot_count + replaced_slot_count) / greatest(parent_structure_usage_count, 1)
```

```text
moth_eaten_score = clamp01(
  0.50 * missing_rate
+ 0.35 * replaced_rate
+ 0.15 * residual_rate
)
```

Default moth-eaten decision:

```text
parent_structure_usage_count >= 5
and (missing_slot_count + replaced_slot_count) >= 3
and slot_missing_or_replaced_rate >= 0.30
-> moth_eaten evidence
-> sleep.decohere_structure may send the unstable slot or attachment to decoherence_bank
```

Moth-eaten evidence is scheduled decoherence evidence, not immediate deletion.
