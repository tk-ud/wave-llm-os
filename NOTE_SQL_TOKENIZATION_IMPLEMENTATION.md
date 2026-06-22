# Supporting Note: PostgreSQL Core Tables

Canonical authority: `SPEC_CANONICAL_CORE.md`.

This file is an implementation sketch, not the source of truth.

---

# Reference Rule

```text
UUID  = row / event identity
index = semantic reference
array = bigint[] index array
```

No semantic UUID references.
No `uuid[]` columns.

---

# Core Tables

```text
input_observation
token
vocabulary
grammar
grammar_relation
decoherence_bank
logs.coherence
logs.current
logs.diff
core_state
core_operation_policy
core_notify_queue
scheduler_job
scheduler_job_run
phase_relation_candidate
structural_vector_index
mastication_job
remote_node_trust
remote_event_inbox
remote_event_quarantine
```

---

# Input Observation

```sql
create table input_observation (
  observation_uuid uuid primary key default gen_random_uuid(),
  observation_index bigint generated always as identity unique,
  source_kind text not null,
  source_hash text not null,
  body text not null,
  raw_text_policy text not null default 'discard_after_ingest',
  learning_weight numeric not null default 1.0,
  metadata_json jsonb null,
  created_at timestamptz not null default now()
);
```

Allowed source kinds:

```text
user
web
file
remote_event
manual
system
```

---

# Remote Trust

Remote trust is gated by `core_state.distributed_sync.enabled`.

```sql
create table remote_node_trust (
  remote_node_uuid uuid primary key default gen_random_uuid(),
  remote_node_id text not null unique,
  trust_level text not null default 'unknown',
  enabled boolean not null default false,
  metadata_json jsonb null,
  created_at timestamptz not null default now(),
  updated_at timestamptz not null default now()
);
```

Trusted nodes still cannot mutate semantic tables directly.

---

# Semantic Tables

## token

`token` is the canonical token-level seed table.

Long raw spans must be split before becoming normal semantic tokens.

```sql
create table token (
  token_uuid uuid primary key default gen_random_uuid(),
  token_index bigint generated always as identity unique,
  raw text not null,
  raw_length integer generated always as (char_length(raw)) stored,
  split_flag boolean not null default false,
  split_kind text null,
  split_parent_token_index bigint null references token(token_index),
  split_position integer null,
  metadata_json jsonb null,
  created_at timestamptz not null default now()
);
```

Split rule:

```text
raw having length > 1000
→ split_flag = true
→ split_kind = 'length_gt_1000'
→ child token rows preserve split_parent_token_index and split_position
```

`split_flag` means the token row is part of a controlled split process.

It is not semantic authority by itself.

Semantic references still use `token_index`.

## vocabulary

```sql
create table vocabulary (
  vocabulary_uuid uuid primary key default gen_random_uuid(),
  vocabulary_index bigint generated always as identity unique,
  token_array bigint[] not null,
  raw_text text null,
  vocabulary_hash text not null unique,
  draft_flag boolean not null default true,
  status text not null default 'draft'
);
```

## grammar

```sql
create table grammar (
  grammar_uuid uuid primary key default gen_random_uuid(),
  grammar_index bigint generated always as identity unique,
  vocabulary_array bigint[] not null,
  grammar_hash text not null unique,
  end_of_sentence_flag boolean not null default false,
  draft_flag boolean not null default true,
  status text not null default 'draft'
);
```

## grammar_relation

```sql
create table grammar_relation (
  grammar_relation_uuid uuid primary key default gen_random_uuid(),
  grammar_relation_index bigint generated always as identity unique,
  grammar_array bigint[] not null,
  relation_hash text not null unique,
  relation_weight numeric not null default 0,
  draft_flag boolean not null default true,
  status text not null default 'draft'
);
```

## phase_relation_candidate

```sql
create table phase_relation_candidate (
  phase_relation_candidate_uuid uuid primary key default gen_random_uuid(),
  phase_relation_candidate_index bigint generated always as identity unique,
  grammar_array bigint[] not null,
  relation_array bigint[] null,
  relation_hash text not null unique,
  evidence_json jsonb not null,
  phase_score numeric not null default 0,
  pressure numeric not null default 0,
  evidence_count bigint not null default 0,
  draft_flag boolean not null default true,
  status text not null default 'draft',
  created_at timestamptz not null default now(),
  last_seen_at timestamptz not null default now(),
  updated_at timestamptz not null default now()
);
```

All semantic arrays are `bigint[]` index arrays.

`phase_relation_candidate.grammar_array` is the Phase Attention output path.

`phase_relation_candidate.relation_array` is optional and references relation indexes only.

---

# Logs

```text
logs.coherence = observation evidence
logs.current   = scheduled aggregate read model
logs.diff      = mutation / decision evidence
```

`logs.current` is the scheduled pressure surface for Phase Attention.

Adoption audit is a view over `logs.diff`, not a table.

---

# Operation Gate

```text
core_state
core_operation_policy
core_can_execute(operation_key)
```

Mutation-capable operations must pass the gate.

Freeze blocks semantic mutation.

---

# Phase Attention Sketch

```text
logs.current pressure
+ grammar_relation aggregates
+ decoherence_bank recurrence
→ scheduled Phase Attention
→ phase_relation_candidate.grammar_array
→ core_can_execute('promote.phase_relation')
→ grammar_relation
→ logs.diff
```

Phase Attention is not the synchronous reply path.

---

# Short Form

```text
PostgreSQL is canonical storage.
All input enters through input_observation.
Remote trust is gated by core_state.
Index arrays are semantic structure.
Token split flags preserve controlled long-span splitting.
logs.current is Phase pressure.
logs.diff is mutation evidence.
Everything mutable goes through core_can_execute().
```
