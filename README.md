# Wave LLMOS

Wave LLMOS is now defined by the PostgreSQL-backed canonical core specification.

Current implementation authority:

```text
SPEC_CANONICAL_CORE.md
```

Legacy wave-geometry / psi / spectral-band design notes are archived under:

```text
legacy_design/
```

They are historical context only.

---

# Current Core

The current core treats meaning as persistent connection structure, not as a spectral wave state, dense vector, or model weight.

```text
meaning = connection form
```

The database is semantic memory.

The decoder does not originate meaning.

Mutation is operation-gated.

---

# Canonical Reference Rule

```text
UUID  = internal row / event identity only
index = semantic reference key
array = bigint[] index array only
```

Semantic references use indexes.

Canonical schema does not define `uuid[]` semantic arrays.

---

# Main Specification Files

```text
SPEC_CANONICAL_CORE.md                  canonical implementation authority
NOTE_SQL_TOKENIZATION_IMPLEMENTATION.md PostgreSQL implementation sketch
NOTE_INDEX_ARRAY_CANONICAL.md           index-array reference rule
NOTE_PHASE_RELATION_CANDIDATE.md        Phase candidate generation
NOTE_NEAR_NEIGHBOR_SEARCH.md            structural near-neighbor search
MIGRATION_LEGACY_REGISTER.md            migration / rejection register
NOTE_QUATERNION_PHILOSOPHY.md           short philosophy note
```

---

# Canonical Runtime Surfaces

```text
input_observation
core_state
core_operation_policy
logs.coherence
logs.current
logs.diff
scheduler_job
scheduler_job_run
phase_relation_candidate
structural_vector_index
remote_node_trust
```

---

# Explicit Rejections

The current spec rejects these as canonical authority:

```text
WaveCore.psi primary semantic state
wave_geometry primary semantic authority
spectral band basis as meaning authority
SQLite canonical schema
constraint_rule table
adoption_audit table
decoder_trace table
loop_guard table
semantic UUID references
uuid[] canonical columns
```

---

# Legacy Design Archive

Archived legacy documents:

```text
legacy_design/README_LEGACY_WAVE_GEOMETRY.md
legacy_design/todo_legacy_manifest_wavecore.md
```

These files are not implementation authority.

If legacy files conflict with `SPEC_CANONICAL_CORE.md`, the canonical core spec wins.
