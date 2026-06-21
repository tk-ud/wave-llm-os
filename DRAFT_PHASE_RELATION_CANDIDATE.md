# Draft: Phase Relation Candidate

## Status

This document defines the canonical draft behavior for Phase relation candidate generation in the Quaternion Exploration Core.

Phase is not synchronous raw-input reasoning.

Phase is scheduled relation candidate generation from normalized aggregate values.

Its canonical output is a grammar array.

---

# 1. Position in the Core

The semantic hierarchy is:

```text
token
→ vocabulary
→ grammar
→ grammar array
```

Because grammar is already an ordered vocabulary bundle, and vocabulary is already an ordered token bundle, a decided grammar array determines the vocabulary and token levels by hierarchical reference.

Therefore, Phase relation candidate output does not need to carry token arrays or vocabulary arrays as canonical output.

It outputs grammar arrays.

---

# 2. Canonical Output

Phase generates candidates for `grammar_relation`.

Canonical candidate form:

```text
grammar_array[]
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

  candidate_index bigint not null unique,

  grammar_array uuid[] not null,

  relation_hash text not null unique,

  source_current_array uuid[] not null,

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
```

---

# 4. Link Table

The array is the ordered canonical bundle.

The link table exists for search, aggregation, and positional lookup.

```sql
create table phase_relation_candidate_link (
  phase_relation_candidate_uuid uuid not null references phase_relation_candidate(phase_relation_candidate_uuid),
  grammar_uuid uuid not null references grammar(grammar_uuid),
  position integer not null,

  primary key (phase_relation_candidate_uuid, position)
);

create index idx_phase_relation_candidate_link_grammar
  on phase_relation_candidate_link (grammar_uuid, position);
```

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
grammar A → grammar B
grammar B → grammar C
grammar A + grammar C recur under nearby scopes
grammar A and grammar B repeatedly appear around the same residual pressure
a decoherence residual repeatedly appears between known grammar candidates
```

The result is always represented as:

```text
{grammar A, grammar B, grammar C, ...}
```

---

# 7. Missing Slot Completion

If a relation candidate has a missing slot, the system fills it by reverse hierarchical near-neighbor search.

Reverse completion order:

```text
grammar array
→ grammar near-neighbor search
→ vocabulary near-neighbor search
→ token near-neighbor search
```

The system starts from the highest available structure.

If the grammar array candidate is partially known, missing grammar candidates are searched first.

If grammar cannot be resolved, vocabulary candidates inside nearby grammar candidates are searched.

If vocabulary cannot be resolved, token-level identity is searched last.

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

# 9. Evidence

`evidence_json` should preserve why the candidate exists.

Example:

```json
{
  "source": "logs.current",
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

# 10. Non-Goals

Phase relation candidate generation is not:

- synchronous reply-time reasoning
- raw token attention
- a replacement for grammar relation
- direct mutation of adopted relation
- global semantic consensus
- a decoder

It is scheduled candidate generation from aggregate evidence.

---

# 11. Short Form

```text
Phase reads current aggregate pressure.
Phase outputs grammar arrays.
Grammar arrays determine vocabulary/token references through hierarchy.
Missing slots are completed in reverse hierarchy.
Promotion happens only after near-neighbor selection and operation-gated promotion.
```
