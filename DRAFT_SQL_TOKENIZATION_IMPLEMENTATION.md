# Draft Implementation Note: PostgreSQL Canonical Core Tables

## Status

This document is the implementation draft for the canonical database layer of the Quaternion Exploration Core.

It replaces the earlier tokenizer-only draft with a PostgreSQL-first schema for:

- token identity
- vocabulary candidates
- grammar candidates
- grammar relation candidates
- decoherence bank
- coherence logs
- scheduled aggregate current values
- diff/change evidence
- core state
- operation policy gates
- notify queue
- mastication jobs
- remote event intake

SQLite is intentionally not a target for the canonical implementation.

The next core requires PostgreSQL features such as UUID, JSONB, arrays, upsert, functions, triggers, notify/listen, scheduled workers, and near-neighbor search extensions.

---

# 1. Storage Decision

Canonical storage target:

```text
PostgreSQL
```

Rationale:

- UUID is first-class
- JSONB is first-class
- arrays can store ordered candidate bundles
- link tables can support search and aggregation
- `insert ... on conflict` is required
- functions can centralize operation gates
- triggers can enqueue notify events
- LISTEN/NOTIFY can drive backend workers
- extensions can support near-neighbor search
- scheduled aggregation can be DB-centered

SQLite is not used for the canonical core because the design requires database-side policy, aggregation, and near-neighbor behavior.

---

# 2. Schema Responsibilities

```text
logs.coherence = append-only coherence observation log
logs.current   = scheduled aggregate read model from logs.coherence
logs.diff      = mutation / normalization / flag / core_state evidence

token            = raw atomic identity
vocabulary       = ordered token bundle
grammar          = ordered vocabulary bundle
grammar_relation = ordered grammar bundle

decoherence_bank = no-hit / low-hit / unabsorbed residuals
core_state       = runtime state authority
core_operation_policy = operation gate rules
core_notify_queue = DB-created notifications for backend workers
```

The backend transports work.

The database decides state-aware semantic acceptance.

---

# 3. Core Logs

## 3.1 logs.coherence

`logs.coherence` is append-only.

It records observed coherence facts. It does not decide adoption.

```sql
create schema if not exists logs;

create table logs.coherence (
  coherence_uuid uuid primary key default gen_random_uuid(),

  target_table text not null,
  target_uuid uuid not null,

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
  on logs.coherence (target_table, target_uuid);

create index idx_logs_coherence_observed_at
  on logs.coherence (observed_at);
```

## 3.2 logs.current

`logs.current` is the scheduled aggregate result table.

It is the read model used by runtime, schedulers, promotion logic, flag decisions, and Phase candidate generation.

```sql
create table logs.current (
  current_uuid uuid primary key default gen_random_uuid(),

  target_table text not null,
  target_uuid uuid not null,

  current_value jsonb not null,

  aggregation_version bigint not null default 1,

  aggregated_from timestamptz null,
  aggregated_to timestamptz not null default now(),

  updated_at timestamptz not null default now(),

  unique (target_table, target_uuid)
);

create index idx_logs_current_target
  on logs.current (target_table, target_uuid);

create index idx_logs_current_value_gin
  on logs.current using gin (current_value);
```

Example `current_value`:

```json
{
  "coherence_count": 124,
  "hit_count": 103,
  "near_hit_count": 12,
  "partial_hit_count": 9,
  "end_of_sentence_count": 91,
  "end_of_sentence_score": 73.38,
  "last_coherence_score": 0.82,
  "draft_pressure": 41,
  "relation_pressure": 18
}
```

## 3.3 logs.diff

`logs.diff` records all meaningful changes.

It is used for:

- decoherence-to-normalized changes
- draft-to-adopted changes
- flag changes
- core state changes
- freeze-blocked mutation evidence
- remote event decisions
- mastication decisions

```sql
create table logs.diff (
  diff_uuid uuid primary key default gen_random_uuid(),

  target_table text not null,
  target_uuid uuid not null,

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
  on logs.diff (target_table, target_uuid);

create index idx_logs_diff_operation
  on logs.diff (operation_key, status);
```

---

# 4. Token Table

`token` stores atomic raw identities.

The token layer is not semantic authority.

```sql
create table token (
  token_uuid uuid primary key default gen_random_uuid(),

  token_index bigint not null unique,

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

Example split kinds:

```text
space
newline
japanese_period
japanese_comma
comma
period
colon
slash
open_paren
close_paren
code_fence
url_boundary
```

---

# 5. Vocabulary Table

`vocabulary` is an ordered token bundle.

```sql
create table vocabulary (
  vocabulary_uuid uuid primary key default gen_random_uuid(),

  vocabulary_index bigint not null unique,

  token_array uuid[] not null,

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
```

Search/aggregation link table:

```sql
create table vocabulary_token_link (
  vocabulary_uuid uuid not null references vocabulary(vocabulary_uuid),
  token_uuid uuid not null references token(token_uuid),
  position integer not null,

  primary key (vocabulary_uuid, position)
);

create index idx_vocabulary_token_link_token
  on vocabulary_token_link (token_uuid, position);
```

---

# 6. Grammar Table

`grammar` is an ordered vocabulary bundle.

```sql
create table grammar (
  grammar_uuid uuid primary key default gen_random_uuid(),

  grammar_index bigint not null unique,

  vocabulary_array uuid[] not null,

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
```

Search/aggregation link table:

```sql
create table grammar_vocabulary_link (
  grammar_uuid uuid not null references grammar(grammar_uuid),
  vocabulary_uuid uuid not null references vocabulary(vocabulary_uuid),
  position integer not null,

  primary key (grammar_uuid, position)
);

create index idx_grammar_vocabulary_link_vocabulary
  on grammar_vocabulary_link (vocabulary_uuid, position);
```

---

# 7. Grammar Relation Table

`grammar_relation` is an ordered grammar bundle.

It is the canonical successor of the older coherence relation layer.

```sql
create table grammar_relation (
  grammar_relation_uuid uuid primary key default gen_random_uuid(),

  grammar_relation_index bigint not null unique,

  grammar_array uuid[] not null,

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
```

Search/aggregation link table:

```sql
create table grammar_relation_link (
  grammar_relation_uuid uuid not null references grammar_relation(grammar_relation_uuid),
  grammar_uuid uuid not null references grammar(grammar_uuid),
  position integer not null,

  primary key (grammar_relation_uuid, position)
);

create index idx_grammar_relation_link_grammar
  on grammar_relation_link (grammar_uuid, position);
```

---

# 8. Decoherence Bank

`decoherence_bank` stores no-hit, low-hit, and unabsorbed residuals.

It is also the trigger point for web-search notifications, offline near-neighbor replacement, and question notifications.

```sql
create table decoherence_bank (
  decoherence_uuid uuid primary key default gen_random_uuid(),

  residual_hash text not null unique,
  residual_kind text not null,
  -- token | vocabulary | grammar | grammar_relation | external | remote

  raw_text text null,
  token_array uuid[] null,
  vocabulary_array uuid[] null,
  grammar_array uuid[] null,

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
```

Upsert pattern:

```sql
insert into decoherence_bank (
  residual_hash,
  residual_kind,
  raw_text,
  token_array,
  vocabulary_array,
  grammar_array,
  source_observation_uuid,
  source_scope_uuid
)
values (
  :residual_hash,
  :residual_kind,
  :raw_text,
  :token_array,
  :vocabulary_array,
  :grammar_array,
  :source_observation_uuid,
  :source_scope_uuid
)
on conflict (residual_hash)
do update set
  observed_count = decoherence_bank.observed_count + 1,
  pressure = decoherence_bank.pressure + 1,
  updated_at = now()
returning decoherence_uuid;
```

After a successful insert or conflict update, the runtime should evaluate:

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

# 9. Core State

`core_state` is the runtime authority for mutable execution state.

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

---

# 10. Operation Policy Gate

All mutation-capable operations must pass through an operation gate.

```sql
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
```

Example policies:

```sql
insert into core_operation_policy (
  state_key,
  expected_bool,
  operation_key,
  allowed,
  reason
)
values
  ('freeze.enabled', true, 'adopt.vocabulary', false, 'freeze prohibits adopted vocabulary mutation'),
  ('freeze.enabled', true, 'adopt.grammar', false, 'freeze prohibits adopted grammar mutation'),
  ('freeze.enabled', true, 'adopt.relation', false, 'freeze prohibits adopted relation mutation'),
  ('freeze.enabled', true, 'promote.decoherence_to_draft', false, 'freeze prohibits promotion'),
  ('freeze.enabled', true, 'promote.phase_relation', false, 'freeze prohibits relation promotion'),
  ('freeze.enabled', true, 'mastication.learn', false, 'freeze prohibits mastication learning'),
  ('freeze.enabled', true, 'external_observation.ingest', false, 'freeze prohibits external adoption'),
  ('freeze.enabled', true, 'decode.output', true, 'freeze allows read-only decoding'),
  ('freeze.enabled', true, 'coherence.lookup', true, 'freeze allows read-only coherence lookup'),
  ('freeze.enabled', true, 'relation.lookup', true, 'freeze allows read-only relation lookup');
```

Gate function:

```sql
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

---

# 11. Notify Queue

`core_notify_queue` is the DB-to-backend work queue.

The database enqueues semantic notifications; the backend consumes and executes transport or worker tasks.

```sql
create table core_notify_queue (
  notify_uuid uuid primary key default gen_random_uuid(),

  notify_kind text not null,
  -- web_search_requested | question_requested | promotion_candidate_detected
  -- phase_relation_candidate_ready | freeze_blocked_mutation

  source_operation text not null,
  source_table text null,
  source_uuid uuid null,

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

# 12. Mastication Jobs

Mastication is controlled external/remote observation ingestion.

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

# 14. Scheduled Aggregation

## 14.1 Coherence to Current

Scheduled aggregation reads `logs.coherence` and upserts `logs.current`.

Example for grammar:

```sql
insert into logs.current (
  target_table,
  target_uuid,
  current_value,
  aggregated_to,
  updated_at
)
select
  'grammar' as target_table,
  target_uuid,
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
group by target_uuid
on conflict (target_table, target_uuid)
do update set
  current_value = excluded.current_value,
  aggregated_to = excluded.aggregated_to,
  updated_at = now();
```

## 14.2 End-of-Sentence Flag Update

Provisional rule:

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
  and c.target_uuid = g.grammar_uuid
  and (c.current_value->>'coherence_count')::int >= 20
  and (c.current_value->>'end_of_sentence_score')::numeric >= 70
  and g.end_of_sentence_flag = false;
```

The update must be recorded in `logs.diff`.

---

# 15. Canonical Runtime Path

```text
input
→ scope split
→ token upsert
→ vocabulary candidate upsert
→ grammar candidate upsert
→ xi coherence lookup
→ logs.coherence append
→ no-hit / low-hit residual
→ decoherence_bank upsert
→ online/offline branch
   online  → web_search_requested notify
   offline → near vocabulary / grammar / relation blend
   frequent unresolved residual → question_requested notify
→ grammar_relation lookup/upsert
→ zk coherence decoder
→ constraint apply
→ collapse/output
→ mutation-capable updates only through core_can_execute()
```

---

# 16. Minimal Function List

```text
core_get_state(state_key)
core_set_state(state_key, value, reason)
core_can_execute(operation_key)
core_record_diff(...)
core_append_coherence(...)
core_upsert_decoherence(...)
core_enqueue_notify(...)
core_ingest_remote_event(...)
core_queue_mastication(...)
core_promote_candidate(...)
core_adopt_candidate(...)
core_freeze(reason)
core_unfreeze(reason)
```

The application should call database functions rather than reimplement policy decisions in backend code.
