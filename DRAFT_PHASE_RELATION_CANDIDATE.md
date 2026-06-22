# Supporting Note: Phase Relation Candidate

Canonical authority: `DRAFT_CANONICAL_CORE.md`.

---

# Rule

Phase is scheduled relation candidate generation.

It is not synchronous raw-input reasoning.

Output:

```text
grammar_array bigint[] of grammar.grammar_index
```

No token/vocabulary arrays in Phase output.

No UUID semantic references.

---

# Table Sketch

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
  created_at timestamptz not null default now(),
  updated_at timestamptz not null default now()
);
```

---

# Promotion

```text
phase_relation_candidate
→ near-neighbor selection
→ core_can_execute('promote.phase_relation')
→ grammar_relation
→ logs.diff
```

Freeze blocks generation and promotion.

---

# Missing Slots

```text
grammar_index array
→ grammar-index near search
→ vocabulary-index near search
→ token-index near search
```
