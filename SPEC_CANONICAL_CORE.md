# Specification: Canonical Core

## Authority

This file is the single implementation authority for the new core.

Supporting files may explain details, but this file wins on conflict.

---

# Core Rules

```text
UUID  = internal row / event identity only
index = semantic reference key
array = bigint[] index array only
```

Semantic references never use UUID.

Canonical schema does not define `uuid[]` columns.

Evidence UUIDs live in JSONB evidence only.

---

# Semantic Hierarchy

```text
token_index
→ vocabulary.token_array bigint[]
→ grammar.vocabulary_array bigint[]
→ grammar_relation.grammar_array bigint[]
→ phase_relation_candidate.grammar_array bigint[]
```

The index array is the structural semantic vector.

---

# Canonical Tables

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

No canonical `constraint_rule` table.

No canonical `adoption_audit` table.

No canonical web-result-only semantic table.

No canonical `decoder_trace` or `loop_guard` table.

---

# Input Observation

All observations enter the same input pipeline.

Differences are metadata, not separate semantic authority.

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

Web search results are normal input observations with `source_kind = 'web'`.

Remote events are normal input observations after inbox/quarantine acceptance.

---

# Remote Trust

Remote trust is linked to core state.

Global switch:

```text
core_state.distributed_sync.enabled
```

Node registry:

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

Trust levels:

```text
unknown
trusted
limited
blocked
```

Acceptance rule:

```text
core_state.distributed_sync.enabled = true
and remote_node_trust.enabled = true
and trust_level in ('trusted', 'limited')
→ remote_event may become input_observation(source_kind='remote_event')
```

Even trusted nodes cannot mutate semantic tables directly.

Remote trust changes are mutations and must pass operation gate.

---

# Logs

```text
logs.coherence = append-only observation evidence
logs.current   = scheduled aggregate read model
logs.diff      = mutation / decision evidence
```

Semantic log targets use:

```text
target_table
target_index
```

Adoption audit is a view:

```sql
create view logs.adoption_audit as
select *
from logs.diff
where operation_key in (
  'adopt.vocabulary',
  'adopt.grammar',
  'adopt.relation',
  'promote.decoherence_to_draft',
  'promote.phase_relation'
);
```

---

# Operation Gate

Mutation-capable operations must call:

```text
core_can_execute(operation_key)
```

Freeze blocks semantic mutation.

Freeze allows read-only lookup, decoder projection, decoder/collapse invariant check, and output collapse.

---

# Near Neighbor Search

Primary search is structural over index arrays.

`pgvector` may accelerate retrieval over derived structural vectors.

`pgvector` is not semantic authority.

Structural verification decides.

---

# Phase

Phase Attention is scheduled aggregate-weighted relation candidate generation.

Phase reads aggregate pressure and evidence.

Phase reads `logs.current` as the scheduled pressure surface.

Phase writes draft relation paths to `phase_relation_candidate`.

Phase outputs grammar_index arrays only.

Promotion must pass operation gate.

Phase is not the synchronous raw-input reply path.

---

# Decoder / Collapse

Decoder does not originate meaning.

Decoder/collapse invariants are handled by the current core.

No independent constraint table.

No decoder trace table.

No loop guard table.

Looping is unresolved exploration pressure, not a separate error state.

Repeated return to the same grammar path is handled by:

```text
grammar_array structure
Phase Attention candidate search
near-neighbor expansion
decoherence_bank pressure
logs.current pressure
```

If repeated search still cannot resolve, the result remains draft/unresolved and may trigger question notification.
