# Migration Register: Legacy Design

Canonical authority: `SPEC_CANONICAL_CORE_RESUME.md` and routed `SPEC_*` files.

This file tracks migration status only.

---

# Migrated

```text
observation-derived meaning
persistent DB semantic memory
input_observation with source_kind metadata
remote_node_trust linked to core_state.distributed_sync.enabled
logs.coherence
logs.current
logs.diff
core_state
core_operation_policy
scheduler_job / scheduler_job_run
phase_relation_candidate
Phase Attention as scheduled aggregate-weighted relation candidate generation
structural near-neighbor search
canonical index arrays
logs.promotion_audit view
logs.deletion_audit view
draft_flag as canonical lifecycle truth
decoherence_bank as JSONB three-layer fallback store
```

---

# Reinterpreted

```text
fallback hierarchy → decoherence notify / offline blend / question notify
external observation → source_kind input
web search result → input_observation(source_kind='web')
legacy mastication → token-table mastication / normalized aggregate growth
remote sync → core-gated remote input observation
loop guard → grammar-array / Phase Attention / near-neighbor / pressure handling
pgvector → derived retrieval accelerator only
wave_signature → optional evidence only
UI action → manual_operation / operator_action processing classification
adoption wording → operation-gated promotion
```

---

# Rejected

```text
wave_geometry as primary semantic authority
psi / spectral matrix as meaning authority
Euler wave state as semantic substrate
SQLite canonical target
semantic_snapshot / restore
adoption_audit table
adopt.* promotion operation keys
status enum table as lifecycle authority
constraint_rule table
constraint layer as independent authority
decoder_trace table
loop_guard table
dense vector search as semantic authority
UUID semantic references
uuid[] canonical columns
web-result-only semantic table
trusted remote direct semantic mutation
public UI surface in canonical spec
```

---

# Absorbed

```text
decoder/collapse constraints → current core invariants
adoption audit → logs.promotion_audit view
explicit deletion audit → logs.deletion_audit view
web search result handling → source_kind input pipeline
loop-like repetition → unresolved exploration pressure
input mirroring failure → relation-growth / Phase Attention metrics
count-only promotion wording → pressure / surfacing / verification / operation gate
```

---

# Pending

```text
none
```
