# Supporting Note: PostgreSQL Core Tables

Canonical authority: routed SPEC files.

This file is an implementation sketch, not the source of truth.

When this file conflicts with a SPEC file, the SPEC file wins.

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

# Lifecycle Rule

`draft_flag` is the canonical lifecycle truth.

```text
draft_flag = true
→ draft / residual / unconfirmed / anti-pattern / fallback candidate

draft_flag = false
→ active / promoted / ordinary search target
```

No enum status table is required.

No semantic table in this sketch uses `status` as canonical truth.

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

# Semantic Tables

## token

`token` is the canonical token-level seed table.

Tokenization is indexing and segmentation, not semantic interpretation.

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
  metadata_json null,
  created_at timestamptz not null default now()
);
```

## vocabulary

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

`vocabulary.token_array` is an ordered token-index array.

`vocabulary_hash` deduplicates stable token sequences or raw text.

The tokenizer does not decide whether vocabulary is promoted.

It only records stable candidates and recurrence pressure.

Optional vocabulary-token link table:

```sql
create table vocabulary_token_link (
  vocabulary_index bigint not null references vocabulary(vocabulary_index),
  token_index bigint not null references token(token_index),
  position integer not null,
  primary key (vocabulary_index, position)
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
  deleted_flag boolean not null default false,
  first_seen_at timestamptz not null default now(),
  last_seen_at timestamptz not null default now()
);
```

## grammar_relation

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

`grammar_relation` is the canonical coherence relation layer.

It stores ordered semantic continuity between grammar candidates.

Relation is not optional metadata.

Optional relation link table:

```sql
create table grammar_relation_link (
  grammar_relation_index bigint not null references grammar_relation(grammar_relation_index),
  grammar_index bigint not null references grammar(grammar_index),
  position integer not null,
  role text null,
  primary key (grammar_relation_index, position)
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
  rejected_flag boolean not null default false,
  deleted_flag boolean not null default false,
  created_at timestamptz not null default now(),
  last_seen_at timestamptz not null default now(),
  updated_at timestamptz not null default now()
);
```

All semantic arrays are `bigint[]` index arrays.

`phase_relation_candidate.grammar_array` is the Phase Attention output path.

`phase_relation_candidate.relation_array` is optional and references relation indexes only.

## decoherence_bank

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

Required `layer_bundle_json` shape:

```json
{
  "vocabulary": {},
  "grammar": {},
  "relation": {}
}
```

`decoherence_bank` is not trash.

It is a searchable fallback store that preserves vocabulary, grammar, and relation context together.

---

# Scope Split

Long input should not be processed as one giant string.

Initial rule:

```text
scope_size = 1000 characters
optional_overlap = 50 to 100 characters
```

Overlap prevents boundary loss.

Duplicate vocabulary candidates are deduplicated by `vocabulary_hash` or token sequence hash.

---

# Split Strategy

Within each scope, split in cheap bounded stages.

Recommended order:

```text
1. token split_flag boundaries
2. registered vocabulary split_flag boundaries
3. lightweight regex rules
4. remaining residual chunks
```

Regex should be used as a controlled boundary or extraction rule, not as a massive full parser.

Regex rules must be prioritized and bounded.

---

# Iterative Consume Instead of Global Replace

When a split succeeds, do not blindly run global replace.

A global replace may delete multiple matching substrings unintentionally.

Safer idea:

```text
match = first match with start/end
consume only [start, end]
continue with the remaining text or unconsumed ranges
```

For more precise implementation, store consumed ranges instead of mutating text.

```text
consumed_ranges = [[start, end], ...]
```

Then emit unconsumed ranges as residual candidates.

---

# Upsert Pattern

Vocabulary candidates should be upserted by hash.

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

The tokenizer does not decide whether a vocabulary candidate is promoted.

Promotion and reinforcement are handled by verified coherence hits, scheduled Phase promotion, Sleep consolidation, and `core_can_execute(operation_key)`.

---

# Connection to xi / yj / zk

```text
scoped input
→ token / vocabulary candidates
→ xi coherence layer
```

If a token or vocabulary candidate hits active grammar, active vocabulary, or existing draft vocabulary:

```text
candidate → coherence hit bundle
```

If a candidate cannot be absorbed:

```text
candidate → no-hit / low-hit residual → decoherence_bank
```

If multiple grammar or vocabulary candidates are found across a scope or adjacent scopes, they should be connected.

```text
candidate A
candidate B
candidate C
→ grammar_relation candidate
```

Without relation, output becomes a mirror of local input fragments.

Relation is what allows the system to connect the beginning and end of a divided input.

The tokenizer does not produce final output.

It only prepares stable candidates for the coherence / decoherence / relation pipeline.

---

# Decoherence Bank Promotion

The decoherence bank is not an external trash bin and is not removed from the core search space.

Sleep and maintenance may send unused, unstable, moth-eaten, mirror-output, or unresolved structures into `decoherence_bank`, but they do not hard-delete them.

Ordinary promotion is not:

```text
decoherence_bank → draft vocabulary
draft vocabulary → promoted vocabulary
decoherence_bank → draft grammar
draft grammar → promoted grammar
```

That older path may exist only as historical or diagnostic wording. It must not be treated as the canonical promotion path.

The reply-time path is:

```text
active structure search
→ no hit
→ decoherence_bank fallback search
→ structural verification
→ input grammar / grammar_relation diff verification
→ xi coherence hit if verified
→ core_can_execute('promote.decoherence_hit')
→ promote / reinforce
→ logs.diff
```

Recurring residuals may become Draft evidence, but Draft remains an unconfirmed candidate filter and anti-pattern surface for Phase Attention.

A token, vocabulary, grammar, or relation path that was decohered yesterday may become a coherence-layer hit tomorrow if fallback search verifies it against the current input grammar.

Deletion from `decoherence_bank` is not a sleep-time or promotion-time action.

Hard deletion is an explicit manual/operator action only.

---

# Relation Growth

The relation layer connects divided grammar and vocabulary candidates.

If those pieces are not reconnected, the system only mirrors nearby fragments.

Example:

```text
scope 1: account access failed
scope 2: recovery link expired
scope 3: deadline is today
```

Without relation:

```text
three local hits
candidate output - input grammar = empty or reorder-only
→ mirror_output evidence
```

With relation:

```text
access failure → expired recovery path → urgent deadline
```

The connected form is not meaningful merely because it looks different from the input.

The relation path must still be verified by input grammar / grammar_relation diff.

Corpus-time output must also subtract input grammar:

```text
meaningful corpus output
= candidate output - input grammar
```

If the subtraction leaves no meaningful delta, the result is mirror_output evidence and must not reinforce `grammar_relation`.

---

# Logs and Operation Evidence

The canonical log schemas and operation-key permission rules live in routed SPEC files, especially `SPEC_LOG_AGGREGATE_ARCHIVE.md` and `SPEC_OPERATION_GATE.md`.

Implementation code must preserve enough evidence to reproduce:

```text
promote.coherence_hit
promote.decoherence_hit
promote.phase_relation
sleep.decohere_structure
delete.semantic_structure
delete.decoherence_entry
```

Near-neighbor retrieval alone is not promotion evidence.

Promotion requires structural verification, input grammar / grammar_relation diff verification, scoring or policy evidence, and operation-gate success.

---

# Phase Attention Sketch

```text
normalized vocabulary and grammar tables
→ grammar_relation aggregates
→ aggregate.current pressure
→ scheduled Phase Attention
→ phase_relation_candidate.grammar_array
→ core_can_execute('promote.phase_relation')
→ grammar_relation
→ logs.diff
```

Phase Attention is not the synchronous reply path.

---

# Minimal First Implementation

A minimal implementation can start with:

```text
1. token
2. vocabulary
3. grammar
4. grammar_relation
5. decoherence_bank with layer_bundle_json
6. split_flag tokens
7. scope_size = 1000
8. simple split rules
9. vocabulary_hash upsert
10. no-hit / low-hit insert into decoherence_bank
11. relation upsert from adjacent candidate bundles
12. scheduled aggregate refresh for aggregate.current pressure and optional logs.current snapshot
13. fallback search / Phase candidate surfacing from aggregate pressure
14. structural verification and input grammar / grammar_relation diff verification
15. operation-gated promotion or reinforcement
```

This minimal path must not be described as:

```text
decoherence_bank → draft → promoted
```

Canonical minimal boundary:

```text
Count creates pressure.
Pressure surfaces candidates.
Verification creates promotion evidence.
Operation gate permits mutation.
```

No MeCab, UniDic, or language-specific tokenizer is required for the first version.

The tokenizer is:

```text
UUID / index assigner
boundary splitter
hash aggregator
```

The relation layer is:

```text
semantic continuity memory
```

---

# Short Form

```text
Do not make the tokenizer smart.
Make the tokenizer stable.
Do not parse meaning at the entrance.
Index, split, hash, and upsert.
Known candidates go to xi.
Unknown candidates go to yj.
Connected candidates go to grammar_relation.
Aggregated relations feed Phase.
Corrected grammar output comes from zk.
Everything mutable goes through core_can_execute().
```
