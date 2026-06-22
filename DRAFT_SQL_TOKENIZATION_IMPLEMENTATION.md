# Supporting Note: PostgreSQL Core Tables

Canonical authority: `DRAFT_CANONICAL_CORE.md`.

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
  public_key text null,
  metadata_json jsonb null,
  created_at timestamptz not null default now(),
  updated_at timestamptz not null default now()
);
```

Acceptance:

```text
distributed_sync.enabled = true
enabled = true
trust_level in ('trusted', 'limited')
→ input_observation(source_kind='remote_event')
```

Trusted nodes still cannot mutate semantic tables directly.

---

# Semantic Tables

```sql
create table token (
  token_uuid uuid primary key default gen_random_uuid(),
  token_index bigint generated always as identity unique,
  raw text not null,
  split_flag boolean not null default false,
  split_kind text null,
  created_at timestamptz not null default now()
);

create table vocabulary (
  vocabulary_uuid uuid primary key default gen_random_uuid(),
  vocabulary_index bigint generated always as identity unique,
  token_array bigint[] not null,
  raw_text text null,
  vocabulary_hash text not null unique,
  draft_flag boolean not null default true,
  status text not null default 'draft'
);

create table grammar (
  grammar_uuid uuid primary key default gen_random_uuid(),
  grammar_index bigint generated always as identity unique,
  vocabulary_array bigint[] not null,
  grammar_hash text not null unique,
  end_of_sentence_flag boolean not null default false,
  draft_flag boolean not null default true,
  status text not null default 'draft'
);

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

---

# Logs

```sql
create schema if not exists logs;

create table logs.coherence (
  coherence_uuid uuid primary key default gen_random_uuid(),
  target_table text not null,
  target_index bigint not null,
  coherence_kind text not null default 'hit',
  coherence_score numeric null,
  end_of_sentence_observed boolean not null default false,
  context jsonb null,
  observed_at timestamptz not null default now()
);

create table logs.current (
  current_uuid uuid primary key default gen_random_uuid(),
  target_table text not null,
  target_index bigint not null,
  current_value jsonb not null,
  unique (target_table, target_index)
);

create table logs.diff (
  diff_uuid uuid primary key default gen_random_uuid(),
  target_table text not null,
  target_index bigint null,
  operation_key text not null,
  status text not null default 'applied',
  before_value jsonb null,
  after_value jsonb null,
  diff_value jsonb null,
  reason text null,
  actor text null,
  created_at timestamptz not null default now()
);
```

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

# Decoder / Collapse

No `constraint_rule` table.

Use current core gates and decoder/collapse invariants.

---

# Short Form

```text
PostgreSQL is canonical storage.
All input enters through input_observation.
Remote trust is gated by core_state.
Index arrays are semantic structure.
logs.diff is mutation evidence.
Everything mutable goes through core_can_execute().
```
