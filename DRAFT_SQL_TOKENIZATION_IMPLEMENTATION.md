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

There is no canonical `constraint_rule` table.

Constraint behavior is handled by the existing core operation gate and decoder/collapse invariants.

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

Adoption audit is not a separate table.

Adoption is mutation evidence, so `logs.diff` is the source of truth.

`logs.adoption_audit` is a read-only view over `logs.diff`.

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

```sql
create table decoherence_bank (
  decoherence_uuid uuid primary key default gen_random_uuid(),
  residual_hash text not null unique,
  residual_kind text not null,
  raw_text text null,
  token_array bigint[] null,
  vocabulary_array bigint[] null,
  grammar_array bigint[] null,
  source_observation_uuid uuid null,
  source_scope_uuid uuid null,
  observed_count bigint not null default 1,
  pressure numeric not null default 0,
  status text not null default 'draft',
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
  ('near_blend.enabled', 'bool', true, 'offline near-neighbor replacement switch');
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

# 7. Decoder / Collapse Invariants

There is no `constraint_rule` table.

The current core already provides the required guardrails through:

```text
core_state
core_operation_policy
core_can_execute()
logs.diff
decoder invariants
collapse invariants
```

Decoder/collapse invariants are fixed runtime invariants, not mutable semantic records.

They include:

- decoder must not originate meaning
- decoder must not adopt candidates
- decoder must not promote candidates
- decoder must not mutate grammar_relation
- decoder must not mutate core_state
- decoder must not erase decoherence evidence
- collapse must not silently convert draft candidates into adopted meaning
- freeze must still block semantic mutation

A separate constraint table is rejected because it would duplicate the operation policy layer and create a second rule authority.

---

# 8. Notify Queue

```sql
create table core_notify_queue (
  notify_uuid uuid primary key default gen_random_uuid(),
  notify_kind text not null,
  source_operation text not null,
  source_table text null,
  source_index bigint null,
  status text not null default 'queued',
  payload_json jsonb null,
  created_at timestamptz not null default now(),
  consumed_at timestamptz null
);

create index idx_core_notify_queue_status
  on core_notify_queue (status, notify_kind, created_at);
```

---

# 9. Scheduler Worker Registry

SQL does not run scheduled jobs by itself.

A backend worker, cron process, or external job runner must wake up and call database functions.

```sql
create table scheduler_job (
  job_uuid uuid primary key default gen_random_uuid(),
  job_key text not null unique,
  operation_key text not null,
  enabled boolean not null default true,
  schedule_kind text not null,
  interval_seconds integer null,
  cron_expr text null,
  status text not null default 'idle',
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

create table scheduler_job_run (
  run_uuid uuid primary key default gen_random_uuid(),
  job_uuid uuid not null references scheduler_job(job_uuid),
  operation_key text not null,
  status text not null,
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

---

# 10. Phase Relation Candidate

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
```

No UUID column is used in the semantic link table.

Evidence record UUID identities may appear inside `evidence_json`, not as UUID-array columns.

---

# 11. Structural Vector Index

A derived vector index may be used for large-scale candidate retrieval.

It is generated from canonical index arrays and aggregate evidence.

It is not semantic authority.

```sql
create table structural_vector_index (
  index_uuid uuid primary key default gen_random_uuid(),
  target_table text not null,
  target_index bigint not null,
  index_kind text not null,
  source_hash text not null,
  derived_vector vector null,
  metadata_json jsonb null,
  created_at timestamptz not null default now(),
  updated_at timestamptz not null default now(),
  unique (target_table, target_index, index_kind)
);
```

---

# 12. Mastication Jobs

```sql
create table mastication_job (
  job_uuid uuid primary key default gen_random_uuid(),
  source_kind text not null,
  source_hash text not null,
  status text not null default 'queued',
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

---

# 13. Remote Event Intake

Remote events are observation candidates, not semantic updates.

```sql
create table remote_event_inbox (
  event_uuid uuid primary key default gen_random_uuid(),
  remote_node_id text not null,
  event_hash text not null unique,
  signature text null,
  payload_hash text not null,
  status text not null default 'received',
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

# 14. Scheduled Aggregation

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
    'end_of_sentence_count', count(*) filter (where end_of_sentence_observed = true),
    'end_of_sentence_score',
      100.0 * count(*) filter (where end_of_sentence_observed = true) / nullif(count(*), 0)
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

The flag update must be recorded in `logs.diff` using `target_index`.

---

# 15. Canonical Runtime Path

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
→ decoder/collapse invariant check
→ collapse/output
→ mutation-capable updates only through core_can_execute()
```

---

# 16. Minimal Function List

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

# 17. Short Form

```text
UUID is identity.
Index is semantic reference.
Array is index array.
Canonical schema does not define uuid[] columns.
Adoption audit is a view over logs.diff, not a table.
Constraint tables are not canonical.
Decoder/collapse invariants are absorbed into the current core.
Near-neighbor search runs over index arrays.
pgvector may accelerate derived retrieval.
Structural verification decides.
```
