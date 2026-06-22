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

# Phase Attention

Phase Attention is scheduled aggregate-weighted relation candidate generation.

It reads normalized vocabulary, grammar, relation aggregates, and `logs.current` pressure.

It generates draft `phase_relation_candidate.grammar_array` paths for later evidence, review, promotion, or decoder support.

It is not the synchronous reply path.

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

# Document Agenda

Read documents by responsibility, not by file order.

The canonical runtime flow must be understood as one connected machine:

```text
observation
→ token / vocabulary / grammar mastication
→ coherence / decoherence
→ grammar_relation
→ logs.current
→ Phase Attention
→ phase_relation_candidate
→ promotion gate
→ decoder / collapse
```

Supporting notes may explain parts of this machine, but they do not replace the canonical flow.

## `SPEC_CANONICAL_CORE.md`

Role:

```text
Single implementation authority.
Defines canonical tables, reference rules, logs, operation gate, Phase, decoder/collapse boundary.
```

Use this file to answer:

```text
What is canonical?
Which tables exist?
Which references are allowed?
Which mutations require operation gate?
What wins on conflict?
```

Do not use this file as:

```text
A complete algorithm implementation manual.
A legacy philosophy explanation.
A replacement for supporting notes.
```

## `NOTE_SQL_TOKENIZATION_IMPLEMENTATION.md`

Role:

```text
PostgreSQL implementation sketch.
Shows table-shape examples and storage conventions.
```

Use this file to answer:

```text
How might the canonical tables be represented in SQL?
Which columns are expected in sketches?
How do input_observation, semantic arrays, logs, and Phase candidates map to storage?
```

Do not use this file as:

```text
The source of truth when it conflicts with SPEC_CANONICAL_CORE.md.
A full migration script.
A full runtime algorithm.
```

## `NOTE_INDEX_ARRAY_CANONICAL.md`

Role:

```text
Reference rule note.
Explains why semantic references use bigint index arrays and not UUID arrays.
```

Use this file to answer:

```text
What is identity?
What is semantic reference?
Which arrays are canonical?
Why is uuid[] rejected for semantic paths?
```

Do not use this file as:

```text
A relation-growth algorithm.
A Phase Attention specification.
A decoder specification.
```

## `NOTE_PHASE_RELATION_CANDIDATE.md`

Role:

```text
Phase Attention and phase_relation_candidate supporting specification.
Explains scheduled aggregate-weighted relation candidate generation.
```

Use this file to answer:

```text
What is Phase Attention?
What does Phase read?
What does Phase write?
How does logs.current pressure drive grammar_array expansion?
What is a phase_relation_candidate?
How are draft candidates promoted?
What is mirror output?
```

Do not use this file as:

```text
The synchronous reply path.
A Transformer Attention description.
A raw-input reasoning algorithm.
The canonical authority over SPEC_CANONICAL_CORE.md.
```

## `NOTE_NEAR_NEIGHBOR_SEARCH.md`

Role:

```text
Structural near-neighbor search note.
Explains search over index arrays and derived-vector acceleration boundaries.
```

Use this file to answer:

```text
How are similar grammar_array paths searched?
What does structural verification mean?
How can pgvector be used without becoming semantic authority?
How does repeated grammar return expand search?
```

Do not use this file as:

```text
A meaning authority.
A dense-vector-first retrieval spec.
A Phase promotion rule.
```

## `MIGRATION_LEGACY_REGISTER.md`

Role:

```text
Migration and rejection register.
Tracks which legacy concepts were migrated, reinterpreted, rejected, or absorbed.
```

Use this file to answer:

```text
What happened to legacy wave_geometry, psi, spectral matrix, loop guard, web search, remote sync, and adoption audit?
Which legacy terms survive under new canonical meanings?
```

Do not use this file as:

```text
An implementation authority.
A runtime flow definition.
A place to introduce new canonical behavior.
```

## `NOTE_QUATERNION_PHILOSOPHY.md`

Role:

```text
Short philosophy note.
Explains the q = w + xi + yj + zk framing and why order matters.
```

Use this file to answer:

```text
What is the conceptual role of w, xi, yj, zk?
Why is meaning treated as connection form?
How does Phase Attention fit the exploration philosophy?
```

Do not use this file as:

```text
Implementation authority.
A table schema.
A mathematical proof obligation.
A replacement for SPEC_CANONICAL_CORE.md.
```

## `legacy_design/`

Role:

```text
Historical archive.
Contains legacy wave-geometry / psi / spectral-band design context.
```

Use these files to answer:

```text
Where did the current design come from?
Which legacy principles were preserved or reinterpreted?
What historical ideas were rejected as canonical authority?
```

Do not use these files as:

```text
Current implementation authority.
Canonical schema source.
Canonical semantic authority.
```

---

# Main Specification Files

```text
SPEC_CANONICAL_CORE.md                  canonical implementation authority
NOTE_SQL_TOKENIZATION_IMPLEMENTATION.md PostgreSQL implementation sketch
NOTE_INDEX_ARRAY_CANONICAL.md           index-array reference rule
NOTE_PHASE_RELATION_CANDIDATE.md        Phase Attention / relation candidate generation
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
legacy_design/LEGACY_PHILOSOPHY.md
legacy_design/LEGACY_MANIFEST.yaml
legacy_design/README_LEGACY_WAVE_GEOMETRY.md
legacy_design/todo_legacy_manifest_wavecore.md
```

These files are not implementation authority.

If legacy files conflict with `SPEC_CANONICAL_CORE.md`, the canonical core spec wins.
