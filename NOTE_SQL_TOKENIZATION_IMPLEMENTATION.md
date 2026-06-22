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

```text
token(token_uuid, token_index, raw)
vocabulary(vocabulary_uuid, vocabulary_index, token_array)
grammar(grammar_uuid, grammar_index, vocabulary_array)
grammar_relation(grammar_relation_uuid, grammar_relation_index, grammar_array)
```

All semantic arrays are `bigint[]` index arrays.

---

# Logs

```text
logs.coherence = observation evidence
logs.current   = scheduled aggregate read model
logs.diff      = mutation / decision evidence
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

# Short Form

```text
PostgreSQL is canonical storage.
All input enters through input_observation.
Remote trust is gated by core_state.
Index arrays are semantic structure.
logs.diff is mutation evidence.
Everything mutable goes through core_can_execute().
```
