# Draft: Phase Relation Candidate

## Status

This document defines the canonical draft behavior for Phase relation candidate generation in the Quaternion Exploration Core.

Phase is not synchronous raw-input reasoning.

Phase is scheduled relation candidate generation from normalized aggregate values.

Its canonical output is a grammar index array.

---

# 1. Position in the Core

The semantic hierarchy is:

```text
token_index
→ vocabulary_index
→ grammar_index
→ grammar_index array
```

Because grammar is already an ordered vocabulary-index bundle, and vocabulary is already an ordered token-index bundle, a decided grammar-index array determines the vocabulary and token levels by hierarchical index traversal.

Therefore, Phase relation candidate output does not carry token arrays or vocabulary arrays as canonical output.

It outputs grammar-index arrays.

UUIDs may identify rows, but UUIDs are not semantic references.

The canonical schema does not define `uuid[]` columns.

If evidence must mention multiple UUID identities, they live inside `evidence_json`.

---

# 2. Canonical Output

Phase generates candidates for `grammar_relation`.

Canonical candidate form:

```text
grammar_array bigint[]
```

Where:

```text
grammar_array = ordered grammar.grammar_index values
```

A Phase candidate is a draft relation candidate until promoted.

It must not directly mutate `grammar_relation`.

Promotion must pass through:

```text
core_can_execute('promote.phase_relation')
```

---

# 3. Table

```sql
create table phase_relation_candidate (
  phase_relation_candidate_uuid uuid primary key default gen_random_uuid(),

  phase_relation_candidate_index bigint generated always as identity unique,

  grammar_array bigint[] not null,

  relation_hash text not null unique,

  evidence_json jsonb not null,

  score numeric not null default 0,
  pressure numeric not null default 0,

  draft_flag boolean not null default true,
  status text not null default 'draft',
  -- draft | promoted | adopted | rejected | dormant

  created_at timestamptz not null default now(),
  updated_at timestamptz not null default now()
);

create index idx_phase_relation_candidate_grammar_array
  on phase_relation_candidate using gin (grammar_array);

create index idx_phase_relation_candidate_status
  on phase_relation_candidate (draft_flag, status);

create index idx_phase_relation_candidate_score
  on phase_relation_candidate (score, pressure);

create index idx_phase_relation_candidate_evidence
  on phase_relation_candidate using gin (evidence_json);
```

`evidence_json` may include current record UUID identities.

Those UUIDs are audit evidence, not semantic references.

---

# 4. Link Table

The array is the ordered canonical bundle.

The link table exists for search, aggregation, and positional lookup.

It uses indexes as semantic references.

```sql
create table phase_relation_candidate_link (
  phase_relation_candidate_index bigint not null references phase_relation_candidate(phase_relation_candidate_index),
  grammar_index bigint not null references grammar(grammar_index),
  position integer not null,

  primary key (phase_relation_candidate_index, position)
);

create index idx_phase_relation_candidate_link_grammar
  on phase_relation_candidate_link (grammar_index, position);
```

No UUID column is used in the semantic link table.

---

# 5. Inputs

Phase reads normalized aggregate and candidate evidence.

Primary inputs:

```text
logs.current
grammar
grammar_relation
decoherence_bank
logs.diff
```

Important aggregate values include:

```text
coherence_count
hit_count
near_hit_count
partial_hit_count
end_of_sentence_score
draft_pressure
relation_pressure
co-occurrence pressure
promotion history
```

---

# 6. Generation Rule

Phase generates a candidate when aggregate evidence suggests that grammar candidates repeatedly form a relation path.

Examples:

```text
grammar_index A → grammar_index B
grammar_index B → grammar_index C
grammar_index A + grammar_index C recur under nearby scopes
grammar_index A and grammar_index B repeatedly appear around the same residual pressure
a decoherence residual repeatedly appears between known grammar candidates
```

The result is always represented as:

```text
{grammar_index A, grammar_index B, grammar_index C, ...}
```

---

# 7. Missing Slot Completion

If a relation candidate has a missing slot, the system fills it by reverse hierarchical near-neighbor search.

Reverse completion order:

```text
grammar index array
→ grammar-index near-neighbor search
→ vocabulary-index near-neighbor search
→ token-index near-neighbor search
```

The system starts from the highest available structure.

If the grammar array candidate is partially known, missing grammar indexes are searched first.

If grammar cannot be resolved, vocabulary indexes inside nearby grammar candidates are searched.

If vocabulary cannot be resolved, token indexes are searched last.

This keeps relation completion aligned with the hierarchy.

The system must not start missing-slot completion from raw tokens when a higher-level grammar context exists.

---

# 8. Promotion

Phase candidates are not adopted directly.

Promotion path:

```text
phase_relation_candidate
→ near-neighbor candidate selection
→ promotion candidate notify
→ core_can_execute('promote.phase_relation')
→ grammar_relation
→ logs.diff
```

A candidate may be promoted when near-neighbor search repeatedly selects it as a viable relation candidate.

Promotion is based on candidate selection pressure, score, recurrence, and relation usefulness.

Freeze blocks both generation and promotion:

```text
freeze.enabled = true
→ generate.phase_relation_candidate blocked
→ promote.phase_relation blocked
```

---

# 9. Derived Vector Acceleration

Phase candidate search may use a derived vector index for large-scale retrieval.

The derived vector must be generated from:

```text
grammar_array bigint[]
+ positional features
+ logs.current pressure
+ evidence_json features
```

The derived vector is only a retrieval accelerator.

It is not semantic authority.

Final candidate selection must verify the grammar-index array structurally.

---

# 10. Evidence

`evidence_json` should preserve why the candidate exists.

Example:

```json
{
  "source": "logs.current",
  "source_current_record_ids": ["current_uuid values as strings"],
  "coherence_count": 84,
  "relation_pressure": 32,
  "near_selection_count": 17,
  "decoherence_bridge_count": 5,
  "evidence_kind": [
    "co_occurrence",
    "scope_proximity",
    "decoherence_bridge"
  ]
}
```

The evidence is not semantic authority by itself.

It is promotion material.

---

# 11. Non-Goals

Phase relation candidate generation is not:

- synchronous reply-time reasoning
- raw token attention
- UUID-array relation building
- UUID-based semantic reference
- a replacement for grammar relation
- direct mutation of adopted relation
- global semantic consensus
- a decoder

It is scheduled candidate generation from aggregate evidence.

---

# 12. Short Form

```text
Phase reads current aggregate pressure.
Phase outputs grammar_index arrays.
Grammar_index arrays determine vocabulary/token references through hierarchy.
Missing slots are completed in reverse hierarchy.
Promotion happens only after near-neighbor selection and operation-gated promotion.
Evidence UUID identities live in JSONB evidence, not uuid[] columns.
```
