# Supporting Note: PostgreSQL Table Projection

Related SPEC files own meaning and processing rules.

This NOTE only keeps SQL / DDL projection sketches.

If behavior, lifecycle, promotion, scoring, scheduler, or verification semantics are needed, read the routed SPEC files first.

---

# Related SPEC Files

```text
SPEC_REFERENCE_MODEL.md
SPEC_SEMANTIC_TABLES.md
SPEC_LOG_AGGREGATE_ARCHIVE.md
SPEC_OPERATION_GATE.md
SPEC_CORE_STATE_AND_SCHEDULER.md
SPEC_REMOTE_TRUST.md
```

---

# Table List Projection

```text
input_observation
token
vocabulary
grammar
grammar_relation
grammar_relation_link
decoherence_bank
phase_relation_candidate
```

Runtime, log, aggregate, scheduler, and remote-event table details belong to their routed SPEC files before being expanded here.

`structural_vector_index` is intentionally omitted from the DDL sketches below.

It is an optional retrieval accelerator defined by `SPEC_SEMANTIC_TABLES.md`, not semantic authority and not required for the core SQL projection in this note.

---

# input_observation

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

---

# token

```sql
create table token (
  token_uuid uuid primary key default gen_random_uuid(),
  token_index bigint generated always as identity unique,
  raw text null,
  raw_length integer generated always as (char_length(raw)) stored,
  split_flag boolean not null default false,
  split_kind text null,
  split_parent_token_index bigint null references token(token_index),
  split_position integer null,
  metadata_json jsonb null,
  created_at timestamptz not null default now()
);
```

---

# vocabulary

```sql
create table vocabulary (
  vocabulary_uuid uuid primary key default gen_random_uuid(),
  vocabulary_index bigint generated always as identity unique,
  token_array bigint[] not null,
  raw_text text null,
  vocabulary_hash text not null unique,
  split_flag boolean not null default false,
  split_kind text null,
  observed_count bigint not null default 1,
  draft_flag boolean not null default true,
  deleted_flag boolean not null default false,
  first_seen_at timestamptz not null default now(),
  last_seen_at timestamptz not null default now()
);
```

Optional link-table projection:

```sql
create table vocabulary_token_link (
  vocabulary_index bigint not null references vocabulary(vocabulary_index),
  token_index bigint not null references token(token_index),
  position integer not null,
  primary key (vocabulary_index, position)
);
```

---

# grammar

```sql
create table grammar (
  grammar_uuid uuid primary key default gen_random_uuid(),
  grammar_index bigint generated always as identity unique,
  vocabulary_array bigint[] not null,
  grammar_hash text not null unique,
  end_of_sentence_flag boolean not null default false,
  draft_flag boolean not null default true,
  deleted_flag boolean not null default false,
  first_seen_at timestamptz not null default now(),
  last_seen_at timestamptz not null default now()
);
```

---

# grammar_relation

```sql
create table grammar_relation (
  grammar_relation_uuid uuid primary key default gen_random_uuid(),
  grammar_relation_index bigint generated always as identity unique,
  grammar_array bigint[] not null,
  relation_hash text not null unique,
  relation_kind text not null default 'grammar_to_grammar',
  relation_weight numeric not null default 0,
  observed_count bigint not null default 1,
  draft_flag boolean not null default true,
  deleted_flag boolean not null default false,
  first_seen_at timestamptz not null default now(),
  last_seen_at timestamptz not null default now()
);
```

Optional link-table projection:

```sql
create table grammar_relation_link (
  grammar_relation_index bigint not null references grammar_relation(grammar_relation_index),
  grammar_index bigint not null references grammar(grammar_index),
  position integer not null,
  role text null,
  primary key (grammar_relation_index, position)
);
```

---

# phase_relation_candidate

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
  rejected_flag boolean not null default false,
  deleted_flag boolean not null default false,
  created_at timestamptz not null default now(),
  last_seen_at timestamptz not null default now(),
  updated_at timestamptz not null default now()
);
```

---

# decoherence_bank

```sql
create table decoherence_bank (
  decoherence_uuid uuid primary key default gen_random_uuid(),
  decoherence_index bigint generated always as identity unique,
  residual_hash text not null,
  residual_kind text not null,
  target_table text null,
  target_index bigint null,
  target_path bigint[] null,
  layer_bundle_json jsonb not null,
  evidence_json jsonb not null default '{}'::jsonb,
  pressure numeric not null default 0,
  draft_flag boolean not null default true,
  deleted_flag boolean not null default false,
  first_seen_at timestamptz not null default now(),
  last_seen_at timestamptz not null default now()
);
```

Required implementation shape:

```json
{
  "vocabulary": {},
  "grammar": {},
  "relation": {}
}
```

---

# Upsert Sketch

```sql
insert into vocabulary (
  token_array,
  raw_text,
  vocabulary_hash,
  split_flag,
  split_kind
)
values (
  :token_array,
  :raw_text,
  :vocabulary_hash,
  :split_flag,
  :split_kind
)
on conflict (vocabulary_hash)
do update set
  observed_count = vocabulary.observed_count + 1,
  last_seen_at = now();
```

---

# Implementation Reminder

This file must not define canonical lifecycle, promotion, fallback, scoring, verification, or scheduler behavior.

Those decisions belong in routed SPEC files.
