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

Tokenization is indexing and segmentation, not semantic interpretation.

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

`split_flag` means the token row is part of a controlled split process or acts as a split boundary.

It is not semantic authority by itself.

Semantic references still use `token_index`.

Example split kinds:

```text
space
line_break
japanese_period
japanese_comma
comma
period
colon
slash
open_paren
close_paren
length_gt_1000
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
  status text not null default 'draft',
  first_seen_at timestamptz not null default now(),
  last_seen_at timestamptz not null default now()
);
```

`vocabulary.token_array` is an ordered token-index array.

`vocabulary_hash` deduplicates stable token sequences or raw text.

The tokenizer does not decide whether vocabulary is adopted.

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
  relation_kind text not null default 'grammar_to_grammar',
  relation_weight numeric not null default 0,
  observed_count bigint not null default 1,
  draft_flag boolean not null default true,
  status text not null default 'draft',
  first_seen_at timestamptz not null default now(),
  last_seen_at timestamptz not null default now()
);
```

`grammar_relation` is the canonical coherence relation layer.

It stores ordered semantic continuity between grammar candidates.

Relation is not optional metadata.

It is the memory that allows segmented grammar to remain connected.

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

# Scope Split

Long input should not be processed as one giant string.

Initial rule:

```text
scope_size = 1000 characters
optional_overlap = 50 to 100 characters
```

Example:

```text
scope 1: 0..1000
scope 2: 900..1900
scope 3: 1800..2800
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

Examples of token split boundaries:

```text
space
newline
punctuation
brackets
slashes
commas
periods
```

Examples of vocabulary split boundaries:

```text
"。\n"
"---"
"```"
"###"
"http://"
"https://"
```

Regex should be used as a controlled boundary or extraction rule, not as a massive full parser.

Examples:

```text
dates
URLs
email addresses
numbers
model numbers
file paths
SQL identifiers
currency values
error codes
```

Regex rules must be prioritized and bounded.

---

# Iterative Consume Instead of Global Replace

When a split succeeds, do not blindly run global replace.

A global replace may delete multiple matching substrings unintentionally.

Unsafe idea:

```text
replace(txt, match_text, "")
```

Safer idea:

```text
match = first match with start/end
consume only [start, end]
continue with the remaining text or unconsumed ranges
```

Pseudo flow:

```text
remaining = scope_text

while remaining has split match:
  match = find_first_split_match(remaining)
  before = remaining[0:match.start]
  hit    = remaining[match.start:match.end]
  after  = remaining[match.end:]

  save_candidate(before) if not empty
  save_split(hit)

  remaining = after

save_candidate(remaining) if not empty
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

The tokenizer does not decide whether a vocabulary candidate is adopted.

Adoption is handled by scheduled promotion rules and explicit adoption logic.

---

# Connection to xi / yj / zk

```text
scoped input
→ token / vocabulary candidates
→ xi coherence layer
```

If a token or vocabulary candidate hits adopted grammar, adopted vocabulary, or existing draft vocabulary:

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

When enough hit bundle and relation evidence exists:

```text
hit bundle + relation evidence
→ corrected grammar candidate
→ output projection
```

The tokenizer does not produce final output.

It only prepares stable candidates for the coherence / decoherence / relation pipeline.

---

# Decoherence Bank Promotion

The decoherence bank is not an external trash bin and is not removed from the core search space.

Sleep and maintenance may send unused, unstable, moth-eaten, or unresolved structures into `decoherence_bank`, but they do not hard-delete them.

Ordinary promotion is not:

```text
decoherence_bank → draft vocabulary
draft vocabulary → adopted vocabulary
decoherence_bank → draft grammar
draft grammar → adopted grammar
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

Hard deletion is an explicit UI action only.

---

## Relation Growth

The relation layer connects divided grammar and vocabulary candidates.

If those pieces are not reconnected, the system only mirrors nearby fragments.

Example:

scope 1: account access failed
scope 2: recovery link expired
scope 3: deadline is today

Without relation:

three local hits
candidate output - input grammar = empty or reorder-only
→ mirror_output evidence

With relation:

access failure → expired recovery path → urgent deadline

The relation path must still be verified by input grammar / grammar_relation diff.

Corpus-time output must also subtract input grammar:

meaningful corpus output
= candidate output - input grammar

If the subtraction leaves no meaningful delta, the result is mirror_output evidence and must not reinforce "grammar_relation".

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

Decision pattern:

```text
output_delta contains new relation evidence
→ relation candidate / grammar_relation reinforcement

output_delta contains only copied input
→ mirror_output evidence
→ do not reinforce as new relation

output_delta contains missing or unstable slots
→ residual / decoherence evidence

output_delta bridges input scopes
→ candidate grammar_relation evidence
```

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
normalized vocabulary and grammar tables
→ grammar_relation aggregates
→ logs.current pressure
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
3. decoherence_bank
4. grammar_relation
5. grammar_relation_link
6. split_flag tokens
7. scope_size = 1000
8. simple split rules
9. vocabulary_hash upsert
10. no-hit insert into decoherence_bank
11. relation upsert from adjacent candidate bundles
12. scheduled count-based promotion
13. scheduled Phase candidate generation from aggregates
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
