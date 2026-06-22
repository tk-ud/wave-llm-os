# Draft Implementation Note: PostgreSQL Canonical Core Tables

## Status

This document is the implementation draft for the canonical database layer of the Quaternion Exploration Core.

The canonical representation is index-based:

```text
UUID  = internal row identity / event identity only
index = semantic reference key
array = bigint[] index array only
```

Semantic arrays do not use UUIDs.

Semantic targets do not use UUIDs.

The canonical schema does not define `uuid[]` columns.

If evidence needs to mention multiple UUID identities, it does so inside JSONB evidence fields, not UUID-array columns.

---

# 1. Storage Decision

Canonical storage target:

```text
PostgreSQL
```

Rationale:

- UUID is useful for internal row identity and event identity
- BIGINT indexes are compact semantic reference keys
- BIGINT arrays can store ordered structural semantic vectors
- JSONB is useful for evidence/current payloads
- GIN indexes can support array search
- functions can centralize operation gates
- triggers can enqueue notify events
- LISTEN/NOTIFY can drive backend workers
- backend workers can call DB-owned scheduled functions
- derived pgvector indexes may accelerate large-scale candidate retrieval

PostgreSQL does not run the scheduler by itself.

A backend worker, cron process, or external job runner starts scheduled work.

The database owns job definitions, locks, operation gates, blocked/skipped decisions, and run evidence.

---

# 2. Canonical Reference Rule

```text
*_uuid  = internal row identity / event identity only
*_index = semantic reference key
*_array = bigint[] of semantic indexes only
```

Canonical arrays:

```text
vocabulary.token_array                 bigint[] of token.token_index
grammar.vocabulary_array               bigint[] of vocabulary.vocabulary_index
grammar_relation.grammar_array         bigint[] of grammar.grammar_index
phase_relation_candidate.grammar_array bigint[] of grammar.grammar_index
```

Canonical semantic log target:

```text
target_table text
target_index bigint
```

Not:

```text
target_table text
target_uuid uuid
```

UUIDs may still identify rows, jobs, observations, events, queue items, and audit entries.

They are not semantic references.

---

# 3. Core Logs

```sql
create schema if not exists logs;

create table logs.coherence (
  coherence_uuid uuid primary key default gen_random_uuid(),

  target_table text not null,
  target_index bigint not null,

  source_observation_uuid uuid null,
  source_scope_uuid uuid null,

  coherence_kind text not null default 'hit',
  -- hit | near_hit | partial_hit | relation_hit | eos_hit

  coherence_score numeric null,

  end_of_sentence_observed boolean not null default false,

  context jsonb null,

  observed_at timestamptz not null default now()
);

create index idx_logs_coherence_target
  on logs.coherence (target_table, target_index);

create index idx_logs_coherence_observed_at
  on logs.coherence (observed_at);

create table logs.current (
  current_uuid uuid primary key default gen_random_uuid(),

  target_table text not null,
  target_index bigint not null,

  current_value jsonb not null,

  aggregation_version bigint not null default 1,

  aggregated_from timestamptz null,
  aggregated_to timestamptz not null default now(),

  updated_at timestamptz not null default now(),

  unique (target_table, target_index)
);

create index idx_logs_current_target
  on logs.current (target_table, target_index);

create index idx_logs_current_value_gin
  on logs.current using gin (current_value);

create table logs.diff (
  diff_uuid uuid primary key default gen_random_uuid(),

  target_table text not null,
  target_index bigint null,

  operation_key text not null,

  status text not null default 'applied',
  -- applied | rejected | deferred | blocked | queued

  before_value jsonb null,
  after_value jsonb null,
  diff_value jsonb null,

  reason text null,
  actor text null,

  created_at timestamptz not null default now()
);

create index idx_logs_diff_target
  on logs.diff (target_table, target_index);

create index idx_logs_diff_operation
  on logs.diff (operation_key, status);
```

`target_index` may be null only for non-semantic operational events that do not target a semantic table row.

---

# 4. Token / Vocabulary / Grammar / Relation Tables

## 4.1 token

```sql
create table token (
  token_uuid uuid primary key default gen_random_uuid(),

  token_index bigint generated always as identity unique,

  raw text not null,

  split_flag boolean not null default false,
  split_kind text null,

  created_at timestamptz not null default now()
);

create index idx_token_raw
  on token (raw);

create index idx_token_split
  on token (split_flag, split_kind);
```

## 4.2 vocabulary

`vocabulary` is an ordered `token_index` bundle.

```sql
create table vocabulary (
  vocabulary_uuid uuid primary key default gen_random_uuid(),

  vocabulary_index bigint generated always as identity unique,

  token_array bigint[] not null,

  raw_text text null,
  vocabulary_hash text not null unique,

  split_flag boolean not null default false,
  split_kind text null,

  draft_flag boolean not null default true,
  status text not null default 'draft',
  -- draft | adopted | archived | quarantined | rejected | dormant

  created_at timestamptz not null default now(),
  updated_at timestamptz not null default now()
);

create index idx_vocabulary_token_array
  on vocabulary using gin (token_array);

create index idx_vocabulary_status
  on vocabulary (draft_flag, status);

create table vocabulary_token_link (
  vocabulary_index bigint not null references vocabulary(vocabulary_index),
  token_index bigint not null references token(token_index),
  position integer not null,

  primary key (vocabulary_index, position)
);

create index idx_vocabulary_token_link_token
  on vocabulary_token_link (token_index, position);
```

No UUID column is used in the semantic link table.

## 4.3 grammar

`grammar` is an ordered `vocabulary_index` bundle.

```sql
create table grammar (
  grammar_uuid uuid primary key default gen_random_uuid(),

  grammar_index bigint generated always as identity unique,

  vocabulary_array bigint[] not null,

  grammar_hash text not null unique,

  end_of_sentence_flag boolean not null default false,

  draft_flag boolean not null default true,
  status text not null default 'draft',

  created_at timestamptz not null default now(),
  updated_at timestamptz not null default now()
);

create index idx_grammar_vocabulary_array
  on grammar using gin (vocabulary_array);

create index idx_grammar_status
  on grammar (draft_flag, status);

create index idx_grammar_eos
  on grammar (end_of_sentence_flag);

create table grammar_vocabulary_link (
  grammar_index bigint not null references grammar(grammar_index),
  vocabulary_index bigint not null references vocabulary(vocabulary_index),
  position integer not null,

  primary key (grammar_index, position)
);

create index idx_grammar_vocabulary_link_vocabulary
  on grammar_vocabulary_link (vocabulary_index, position);
```

No UUID column is used in the semantic link table.

## 4.4 grammar_relation

`grammar_relation` is an ordered `grammar_index` bundle.

```sql
create table grammar_relation (
  grammar_relation_uuid uuid primary key default gen_random_uuid(),

  grammar_relation_index bigint generated always as identity unique,

  grammar_array bigint[] not null,

  relation_hash text not null unique,

  end_of_sentence_flag boolean not null default false,

  draft_flag boolean not null default true,
  status text not null default 'draft',

  relation_weight numeric not null default 0,

  created_at timestamptz not null default now(),
  updated_at timestamptz not null default now()
);

create index idx_grammar_relation_array
  on grammar_relation using gin (grammar_array);

create index idx_grammar_relation_status
  on grammar_relation (draft_flag, status);

create index idx_grammar_relation_eos
  on grammar_relation (end_of_sentence_flag);

create table grammar_relation_link (
  grammar_relation_index bigint not null references grammar_relation(grammar_relation_index),
  grammar_index bigint not null references grammar(grammar_index),
  position integer not null,

  primary key (grammar_relation_index, position)
);

create index idx_grammar_relation_link_grammar
  on grammar_relation_link (grammar_index, position);
```

No UUID column is used in the semantic link table.

---

# 5. Decoherence Bank

`decoherence_bank` stores no-hit, low-hit, and unabsorbed residuals.

Residual arrays use semantic indexes.

```sql
create table decoherence_bank (
  decoherence_uuid uuid primary key default gen_random_uuid(),

  residual_hash text not null unique,
  residual_kind text not null,
  -- token | vocabulary | grammar | grammar_relation | external | remote

  raw_text text null,
  token_array bigint[] null,
  vocabulary_array bigint[] null,
  grammar_array bigint[] null,

  source_observation_uuid uuid null,
  source_scope_uuid uuid null,

  observed_count bigint not null default 1,
  pressure numeric not null default 0,

  status text not null default 'draft',
  -- draft | queued | promoted | quarantined | rejected | dormant

  created_at timestamptz not null default now(),
  updated_at timestamptz not null default now()
);

create index idx_decoherence_status
  on decoherence_bank (status);

create index idx_decoherence_pressure
  on decoherence_bank (pressure, observed_count);

create index idx_decoherence_token_array
  on decoherence_bank using gin (token_array);

create index idx_decoherence_vocabulary_array
  on decoherence_bank using gin (vocabulary_array);

create index idx_decoherence_grammar_array
  on decoherence_bank using gin (grammar_array);
```

After a successful insert or conflict update, the database evaluates:

```text
online.enabled
external_observation.enabled
freeze.enabled
near_blend.enabled
question_notify.enabled
```

Then:

```text
online=true  → notify.web_search_requested
offline=true → near vocabulary / grammar / relation blend
frequent unresolved residual → notify.question_requested
```

---

# 6. Core State and Operation Gate

```sql
create table core_state (
  state_uuid uuid primary key default gen_random_uuid(),

  state_key text not null unique,
  state_kind text not null,
  -- bool | int | numeric | text | json

  bool_value boolean null,
  int_value bigint null,
  numeric_value numeric null,
  text_value text null,
  json_value jsonb null,

  authority text not null default 'core',
  description text null,

  created_at timestamptz not null default now(),
  updated_at timestamptz not null default now()
);

create table core_operation_policy (
  policy_uuid uuid primary key default gen_random_uuid(),

  state_key text not null references core_state(state_key),
  expected_bool boolean not null,

  operation_key text not null,

  allowed boolean not null,

  reason text null,

  created_at timestamptz not null default now(),

  unique (state_key, expected_bool, operation_key)
);

create or replace function core_can_execute(p_operation_key text)
returns boolean
language sql
stable
as $$
  select not exists (
    select 1
    from core_operation_policy p
    join core_state s on s.state_key = p.state_key
    where p.operation_key = p_operation_key
      and s.bool_value = p.expected_bool
      and p.allowed = false
  );
$$;
```

Initial state examples:

```sql
insert into core_state (state_key, state_kind, bool_value, description)
values
  ('freeze.enabled', 'bool', false, 'semantic mutation gate'),
  ('online.enabled', 'bool', true, 'online/offline external observation switch'),
  ('mastication.enabled', 'bool', true, 'controlled external/remote observation ingestion'),
  ('external_observation.enabled', 'bool', true, 'web search notification switch'),
  ('distributed_sync.enabled', 'bool', false, 'remote event intake switch'),
  ('question_notify.enabled', 'bool', true, 'question notification switch'),
  ('near_blend.enabled', 'bool', true, 'offline near-neighbor replacement switch'),
  ('constraint.enabled', 'bool', true, 'post-decoder pre-collapse stabilizer switch');
```

Freeze-block policies include:

```text
adopt.vocabulary
adopt.grammar
adopt.relation
promote.decoherence_to_draft
promote.phase_relation
mastication.learn
external_observation.ingest
aggregate.logs_current
update.flag.grammar_eos
detect.promotion_candidate
generate.phase_relation_candidate
```

---

# 7. Notify Queue

```sql
create table core_notify_queue (
  notify_uuid uuid primary key default gen_random_uuid(),

  notify_kind text not null,
  -- web_search_requested | question_requested | promotion_candidate_detected
  -- phase_relation_candidate_ready | freeze_blocked_mutation

  source_operation text not null,
  source_table text null,
  source_index bigint null,

  status text not null default 'queued',
  -- queued | processing | consumed | failed | cancelled

  payload_json jsonb null,

  created_at timestamptz not null default now(),
  consumed_at timestamptz null
);

create index idx_core_notify_queue_status
  on core_notify_queue (status, notify_kind, created_at);
```

---

# 8. Scheduler Worker Registry

SQL does not run scheduled jobs by itself.

A backend worker, cron process, or external job runner must wake up and call database functions.

```sql
create table scheduler_job (
  job_uuid uuid primary key default gen_random_uuid(),

  job_key text not null unique,

  operation_key text not null,

  enabled boolean not null default true,

  schedule_kind text not null,
  -- interval | cron | manual

  interval_seconds integer null,
  cron_expr text null,

  status text not null default 'idle',
  -- idle | running | failed | disabled

  locked_by text null,
  locked_until timestamptz null,

  last_run_at timestamptz null,
  next_run_at timestamptz null,

  run_count bigint not null default 0,
  fail_count bigint not null default 0,

  config_json jsonb null,

  created_at timestamptz not null default now(),
  updated_at timestamptz not null default now()
);

create index idx_scheduler_job_due
  on scheduler_job (enabled, next_run_at, status);

create index idx_scheduler_job_lock
  on scheduler_job (locked_until);

create table scheduler_job_run (
  run_uuid uuid primary key default gen_random_uuid(),

  job_uuid uuid not null references scheduler_job(job_uuid),

  operation_key text not null,

  status text not null,
  -- running | completed | failed | skipped | blocked

  started_at timestamptz not null default now(),
  finished_at timestamptz null,

  affected_count bigint null,

  result_json jsonb null,
  error_json jsonb null
);
```

Before executing a job, the DB-owned runner function must call:

```sql
select core_can_execute(operation_key);
```

If false:

```text
scheduler_job_run.status = blocked
logs.diff.operation_key = operation_key
logs.diff.status = blocked
```

---

# 9. Phase Relation Candidate

Phase relation candidate output is a `grammar_index` array.

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

Evidence record UUID identities may appear inside `evidence_json`, not as UUID-array columns.

---

# 10. Structural Vector Index

A derived vector index may be used for large-scale candidate retrieval.

It is generated from canonical index arrays and aggregate evidence.

It is not semantic authority.

```sql
create table structural_vector_index (
  index_uuid uuid primary key default gen_random_uuid(),

  target_table text not null,
  target_index bigint not null,

  index_kind text not null,
  -- vocabulary | grammar | grammar_relation | phase_relation_candidate

  source_hash text not null,

  derived_vector vector null,

  metadata_json jsonb null,

  created_at timestamptz not null default now(),
  updated_at timestamptz not null default now(),

  unique (target_table, target_index, index_kind)
);
```

Rules:

```text
source_hash must be derived from canonical index-array fields.
derived_vector is invalid if source_hash is stale.
structural verification must run after vector retrieval.
```

---

# 11. Mastication Jobs

```sql
create table mastication_job (
  job_uuid uuid primary key default gen_random_uuid(),

  source_kind text not null,
  -- web | file | user_supplied | remote_event

  source_hash text not null,
  status text not null default 'queued',
  -- queued | processing | completed | failed | quarantined | blocked

  raw_text_policy text not null default 'discard_after_ingest',

  learning_weight numeric not null default 0.10,

  created_at timestamptz not null default now(),
  started_at timestamptz null,
  finished_at timestamptz null
);

create table mastication_observation_result (
  result_uuid uuid primary key default gen_random_uuid(),

  job_uuid uuid not null references mastication_job(job_uuid),

  observation_uuid uuid null,

  inserted_token_count bigint not null default 0,
  decoherence_insert_count bigint not null default 0,
  relation_candidate_count bigint not null default 0,
  adopted_update_count bigint not null default 0,

  raw_text_discarded boolean not null default true,

  created_at timestamptz not null default now()
);
```

Mastication queueing may be allowed during freeze.

Mastication learning must be blocked during freeze.

---

# 12. Remote Event Intake

Remote events are observation candidates, not semantic updates.

```sql
create table remote_event_inbox (
  event_uuid uuid primary key default gen_random_uuid(),

  remote_node_id text not null,
  event_hash text not null unique,
  signature text null,
  payload_hash text not null,

  status text not null default 'received',
  -- received | rejected | quarantined | queued_for_mastication | accepted_observation

  decision text null,
  reason text null,

  created_at timestamptz not null default now()
);

create table remote_event_quarantine (
  quarantine_uuid uuid primary key default gen_random_uuid(),

  event_uuid uuid not null references remote_event_inbox(event_uuid),

  reason text not null,
  released boolean not null default false,

  created_at timestamptz not null default now(),
  released_at timestamptz null
);
```

Backend may verify cryptographic facts.

Database decides semantic acceptance.

Remote events must not directly mutate adopted vocabulary, adopted grammar, adopted relation, or core state.

---

# 13. Scheduled Aggregation

Scheduled aggregation reads `logs.coherence` and upserts `logs.current`.

Example for grammar:

```sql
insert into logs.current (
  target_table,
  target_index,
  current_value,
  aggregated_to,
  updated_at
)
select
  'grammar' as target_table,
  target_index,
  jsonb_build_object(
    'coherence_count', count(*),
    'hit_count', count(*) filter (where coherence_kind = 'hit'),
    'near_hit_count', count(*) filter (where coherence_kind = 'near_hit'),
    'partial_hit_count', count(*) filter (where coherence_kind = 'partial_hit'),
    'end_of_sentence_count', count(*) filter (
      where end_of_sentence_observed = true
    ),
    'end_of_sentence_score',
      100.0
      * count(*) filter (where end_of_sentence_observed = true)
      / nullif(count(*), 0)
  ) as current_value,
  now(),
  now()
from logs.coherence
where target_table = 'grammar'
group by target_index
on conflict (target_table, target_index)
do update set
  current_value = excluded.current_value,
  aggregated_to = excluded.aggregated_to,
  updated_at = now();
```

Provisional end-of-sentence rule:

```text
coherence_count >= 20
end_of_sentence_score >= 70
```

```sql
update grammar g
set
  end_of_sentence_flag = true,
  updated_at = now()
from logs.current c
where c.target_table = 'grammar'
  and c.target_index = g.grammar_index
  and (c.current_value->>'coherence_count')::int >= 20
  and (c.current_value->>'end_of_sentence_score')::numeric >= 70
  and g.end_of_sentence_flag = false;
```

The update must be recorded in `logs.diff` using `target_index`.

---

# 14. Canonical Runtime Path

```text
input
→ scope split
→ token upsert
→ vocabulary candidate upsert using token_index array
→ grammar candidate upsert using vocabulary_index array
→ xi coherence lookup using index arrays
→ logs.coherence append using target_index
→ no-hit / low-hit residual
→ decoherence_bank upsert using index arrays
→ online/offline branch
   online  → web_search_requested notify
   offline → near vocabulary / grammar / relation blend
   frequent unresolved residual → question_requested notify
→ grammar_relation lookup/upsert using grammar_index array
→ zk coherence decoder
→ constraint apply
→ collapse/output
→ mutation-capable updates only through core_can_execute()
```

---

# 15. Minimal Function List

```text
core_get_state(state_key)
core_set_state(state_key, value, reason)
core_can_execute(operation_key)
core_record_diff(... target_index ...)
core_append_coherence(... target_index ...)
core_upsert_decoherence(... index arrays ...)
core_enqueue_notify(... source_index ...)
core_ingest_remote_event(...)
core_queue_mastication(...)
core_promote_candidate(...)
core_adopt_candidate(...)
core_freeze(reason)
core_unfreeze(reason)
scheduler_claim_job(worker_id)
scheduler_finish_job(run_uuid, result)
scheduler_fail_job(run_uuid, error)
core_run_scheduler_job(job_key)
core_find_near_vocabulary(... token_index array ...)
core_find_near_grammar(... vocabulary_index array ...)
core_find_near_grammar_relation(... grammar_index array ...)
core_fill_missing_by_reverse_hierarchy(...)
```

The application should call database functions rather than reimplement policy decisions in backend code.

---

# 16. Short Form

```text
UUID is identity.
Index is semantic reference.
Array is index array.
Canonical schema does not define uuid[] columns.
Near-neighbor search runs over index arrays.
pgvector may accelerate derived retrieval.
Structural verification decides.
```
