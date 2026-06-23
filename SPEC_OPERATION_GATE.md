# Specification: Operation Gate

This file is the canonical authority for mutation-capable operation gating.

All semantic mutation must call:

```text
core_can_execute(operation_key)
```

Freeze blocks semantic mutation.

Freeze allows read-only lookup, decoder projection, invariant check, and output collapse.

Canonical operation keys:

```text
promote.coherence_hit
promote.decoherence_hit
promote.phase_relation
sleep.decohere_structure
delete.semantic_structure
delete.decoherence_entry
reject.candidate
quarantine.remote_event
```

No `adopt.*` operation key is canonical.

Promotion is represented by `promote.*` operations that set or reinforce `draft_flag = false`.

Deletion is explicit and represented by `delete.*` operations.

Near-neighbor similarity alone is never sufficient promotion evidence.

## promote.coherence_hit

Allowed only when freeze is false, structural verification passes, input relation diff is pass or compatible_delta, contradiction is false, mirror output is false, and policy result is allowed.

Effect: target draft_flag becomes false and logs.diff records previous and next lifecycle flags.

## promote.decoherence_hit

Allowed only when freeze is false, source_path is decoherence_bank_fallback, structural verification passes, input relation diff is pass or compatible_delta, contradiction is false, and policy result is allowed.

A decoherence-bank candidate may cohere again only after verification against current input grammar.

## promote.phase_relation

Allowed only when freeze is false, phase_score and pressure meet policy thresholds, evidence_count is sufficient, scheduled structural verification passes, contradiction_count is within policy, mirror output is reduced, and policy result is allowed.

Default policy:

```text
min_phase_score = 0.70
min_pressure = 0.50
min_evidence_count = 3
max_contradiction_count = 0
```

## sleep.decohere_structure

Allowed only when freeze is false, target_table is vocabulary, grammar, grammar_relation, or phase_relation_candidate, reason is unused, unstable, moth_eaten, unresolved, or mirror_output, and policy result is allowed.

Sleep never hard-deletes semantic structures.

## delete.semantic_structure

Allowed only for explicit manual or operator delete when freeze is false and policy result is allowed.

Effect: target deleted_flag becomes true. Ordinary search ignores deleted structures unless explicit maintenance analysis requests them.

## delete.decoherence_entry

Allowed only for explicit manual or operator delete when freeze is false and policy result is allowed.

Effect: decoherence_bank deleted_flag becomes true.
