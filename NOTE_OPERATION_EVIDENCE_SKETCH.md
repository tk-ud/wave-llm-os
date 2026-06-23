# Supporting Note: Operation Evidence Sketch

Canonical authority: `SPEC_CANONICAL_CORE.md`.

This file is an implementation sketch only.

It does not add canonical tables.
It clarifies the minimum evidence that implementation code should preserve when executing core promotion, Phase promotion, and Draft / decoherence handling.

---

# Purpose

The current core separates:

```text
reply-time core promotion
scheduled Phase promotion
sleep / cron consolidation
draft anti-pattern pressure
decoherence_bank fallback search
```

This note sketches the evidence each path should emit so that `logs.coherence`, `logs.current`, and `logs.diff` remain queryable and reproducible.

---

# Evidence Surfaces

Canonical log roles remain:

```text
logs.coherence = append-only observation evidence
logs.current   = scheduled aggregate read model
logs.diff      = mutation / decision evidence
```

Implementation should avoid hiding decisions inside decoder-only runtime state.

Every promotion-capable path should leave enough evidence for later Phase Attention, Sleep, and UI inspection.

---

# Reply-Time Coherence Evidence

For `promote.coherence_hit`, implementation should preserve evidence such as:

```text
operation_key = promote.coherence_hit
source_path = core_reply_path
input_observation_index
input_grammar_index or input_grammar_path
candidate_table
candidate_index or candidate_path
near_neighbor_score
structural_verification_result
input_relation_diff_result
coherence_hit_kind
unabsorbed_residual_count
mirror_output_flag
output_delta_kind
decoder_usage_flag
contradiction_flag
policy_result
```

The important invariant is:

```text
near-neighbor retrieval alone is not promotion evidence.
```

Promotion evidence requires structural verification and input grammar / grammar_relation diff verification.

---

# Decoherence Fallback Evidence

For `promote.decoherence_hit`, implementation should preserve evidence such as:

```text
operation_key = promote.decoherence_hit
source_path = decoherence_bank_fallback
input_observation_index
input_grammar_index or input_grammar_path
decoherence_bank_index or fallback_candidate_path
fallback_hit_score
structural_verification_result
input_relation_diff_result
coherence_hit_kind
residual_origin
moth_eaten_flag
mirror_output_flag
policy_result
```

A decoherence-bank candidate may cohere again only after verification against the current input grammar.

```text
decoherence_bank candidate
→ structural verification passes
→ input grammar / grammar_relation diff verification passes
→ xi coherence hit
→ promote.decoherence_hit
```

---

# Phase Promotion Evidence

For `promote.phase_relation`, implementation should preserve evidence such as:

```text
operation_key = promote.phase_relation
source_path = scheduled_phase
phase_relation_candidate_index
candidate_grammar_array
candidate_relation_array
phase_score
pressure
evidence_count
near_neighbor_stability
scheduled_structural_verification_result
scope_crossing_stability
mirror_output_reduction
contradiction_count
policy_result
```

Phase promotion is scheduled and evidence-driven.
It must not define reply-time adoption semantics.

---

# Draft Anti-Pattern Evidence

Draft is not the primary promotion target.
Draft is an unconfirmed candidate filter and anti-pattern evidence surface.

Implementation should expose enough Draft evidence for Phase scoring:

```text
draft_candidate_path
draft_source_kind
similarity_to_new_candidate
fresh_coherence_evidence_count
mirror_output_count
contradiction_count
unresolved_slot_count
last_seen_at
```

Candidate generation rule:

```text
new candidate
+ similar draft anti-pattern
+ no fresh coherence evidence
→ lower phase_score
→ keep draft / residual / rejected
```

The implementation should not hard-code one universal penalty.
It should store the evidence needed for an inspectable penalty calculation.

---

# Sleep / Cron Evidence

Sleep may send unused, unstable, moth-eaten, or unresolved structures to `decoherence_bank`.

Sleep should preserve evidence such as:

```text
operation_key = sleep.decohere_structure
source_path = sleep_consolidation
target_table
target_index
parent_structure_index or parent_relation_path
usage_window
parent_structure_usage_count
missing_slot_count
replaced_slot_count
slot_missing_or_replaced_rate
moth_eaten_flag
reason
```

Sleep must not hard-delete semantic structures.
Hard deletion remains explicit UI action only.

---

# Minimal Implementation Boundary

A minimal implementation should avoid describing ordinary learning as:

```text
decoherence_bank → draft → adopted
```

That path is historical or diagnostic wording only.

A safer minimal implementation boundary is:

```text
no-hit / low-hit residual
→ decoherence_bank insert/update
→ scheduled aggregate refresh
→ fallback search / Phase candidate surfacing
→ structural verification
→ core_can_execute(operation_key)
→ logs.diff
```

Short form:

```text
Count creates pressure.
Pressure surfaces candidates.
Verification creates promotion evidence.
Operation gate permits mutation.
```
