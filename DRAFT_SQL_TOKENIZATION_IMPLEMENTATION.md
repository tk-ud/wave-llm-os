# Draft Implementation Note: SQL-Based Tokenization and Scope Split

## Status

This document is an implementation draft for the tokenizer / splitter layer of the Quaternion Exploration Core.

It is intentionally practical.

The goal is not to build a language-specific morphological analyzer.

The goal is to create a lightweight database-driven tokenizer that can feed:

```text
q = w + xi + yj + zk

xi = coherence layer
yj = decoherence bank
zk = coherence decoder
```

The tokenizer should produce stable token and word candidates without deciding meaning at the input boundary.

---

# 1. Basic Idea

Tokenization is treated as indexing and segmentation, not semantic interpretation.

The entrance layer should only do the minimum necessary work:

```text
raw input
→ scoped text window
→ character/token indexing
→ split boundary detection
→ word candidate generation
→ hash/upsert
→ coherence layer
```

Meaning is not assigned by the tokenizer.

Meaning is decided later by coherence search, decoherence banking, relation formation, promotion, and decoding.

---

# 2. Core Tables

## 2.1 token table

The token table stores atomic raw tokens.

At the simplest level, one character may be one token.

```sql
create table token_registry (
  token_uuid uuid primary key default gen_random_uuid(),
  token_index bigint not null unique,
  raw_token text not null,
  split_flag boolean not null default false,
  split_kind text null,
  created_at timestamptz not null default now()
);
```

Suggested meaning:

```text
token_uuid  = stable identity of the token
token_index = deterministic or insertion-order index
raw_token   = raw character or atomic unit
split_flag  = true when this token acts as a split boundary
split_kind  = space, newline, comma, period, slash, bracket, etc.
```

Example split tokens:

```text
" "  → space
"\n" → line_break
"。" → japanese_period
"、" → japanese_comma
","  → comma
"."  → period
":"  → colon
"/"  → slash
"("  → open_paren
")"  → close_paren
```

## 2.2 word table

The word table stores candidate chunks made from token indexes.

```sql
create table word_registry (
  word_uuid uuid primary key default gen_random_uuid(),
  token_index_array bigint[] not null,
  raw_word text not null,
  word_hash text not null unique,
  split_flag boolean not null default false,
  split_kind text null,
  observed_count bigint not null default 1,
  first_seen_at timestamptz not null default now(),
  last_seen_at timestamptz not null default now()
);
```

Suggested meaning:

```text
word_uuid         = stable identity of a word/chunk candidate
token_index_array = ordered token indexes included in the chunk
raw_word          = raw chunk text
word_hash         = stable hash of raw_word or token sequence
split_flag        = true when this word/chunk acts as a split boundary
split_kind        = registered split type, if any
observed_count    = recurrence pressure
```

A join table can be added later if array search becomes insufficient.

```sql
create table word_token_link (
  word_uuid uuid not null references word_registry(word_uuid),
  token_uuid uuid not null references token_registry(token_uuid),
  position integer not null,
  primary key (word_uuid, position)
);
```

For the first implementation, `token_index_array[]` may be enough.

---

# 3. Scope Split

Long input should not be processed as one giant string.

If input length exceeds a threshold, process it in local scopes.

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

Duplicate word candidates are deduplicated by `word_hash` or token sequence hash.

---

# 4. Split Strategy

Within each scope, split in cheap stages.

Recommended order:

```text
1. token split_flag boundaries
2. registered word split_flag boundaries
3. lightweight regex rules
4. remaining residual chunks
```

This avoids heavy language-specific parsing.

## 4.1 token split boundaries

Use atomic split tokens first.

Examples:

```text
space
newline
punctuation
brackets
slashes
commas
periods
```

## 4.2 word split boundaries

Some registered chunks can become split markers.

Examples:

```text
"。\n"
"---"
"```"
"###"
"http://"
"https://"
```

## 4.3 regex split rules

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

Regex rules should be prioritized and bounded.

---

# 5. Iterative Consume Instead of Global Replace

When a split succeeds, do not blindly run global replace.

A global replace may delete multiple matching substrings unintentionally.

Instead, use match ranges.

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

# 6. Upsert Pattern

Word candidates should be upserted by hash.

```sql
insert into word_registry (
  token_index_array,
  raw_word,
  word_hash,
  split_flag,
  split_kind
)
values (
  :token_index_array,
  :raw_word,
  :word_hash,
  :split_flag,
  :split_kind
)
on conflict (word_hash)
do update set
  observed_count = word_registry.observed_count + 1,
  last_seen_at = now();
```

The tokenizer does not decide whether a word is adopted vocabulary.

It only records recurrence.

Adoption is handled by scheduled promotion rules and explicit adoption logic.

---

# 7. Connection to xi / yj / zk

The tokenizer feeds the quaternion-style processing layers.

```text
scoped input
→ token/word candidates
→ xi coherence layer
```

## 7.1 xi: coherence layer

If a token/word candidate hits adopted grammar, adopted vocabulary, or existing draft vocabulary:

```text
candidate → coherence hit bundle
```

This becomes part of `z`, the exploration hit bundle.

## 7.2 yj: decoherence bank

If a token/word candidate cannot be absorbed:

```text
candidate → no-hit / low-hit residual → decoherence bank
```

This becomes part of `y`, stored through `j`.

## 7.3 relation layer

If multiple grammar or vocabulary candidates are found across a scope or adjacent scopes, they should be connected.

```text
candidate A
candidate B
candidate C
→ coherence relation candidate
```

Without relation, output becomes a mirror of local input fragments.

Relation is what allows the system to connect the beginning and end of a divided input.

## 7.4 zk: coherence decoder

When enough hit bundle and relation evidence exists:

```text
z + relation evidence → corrected grammar candidate k → output projection
```

The tokenizer does not produce final output.

It only prepares stable candidates for the coherence/decoherence/relation pipeline.

---

# 8. Decoherence Bank Promotion

Unknown or repeated residuals should be promoted only after recurrence or coherence thresholds.

Example:

```sql
select word_hash, raw_word, count(*) as observed_count
from decoherence_bank
group by word_hash, raw_word
having count(*) > 10;
```

Promotion examples:

```text
decoherence bank → draft vocabulary
draft vocabulary → adopted vocabulary
decoherence bank → draft grammar
draft grammar → adopted grammar
```

This is the practical learning path.

A token that was unknown yesterday may become a coherence-layer hit tomorrow.

---

# 9. Coherence Relation Layer

The coherence relation layer connects divided grammar and vocabulary candidates.

This is required because tokenization and scope splitting break input into local pieces.

If those pieces are not reconnected, the system only mirrors nearby fragments.

The relation layer preserves ordered semantic continuity.

Example:

```text
scope 1: refrigeration does not cool
scope 2: cargo may be damaged
scope 3: customer wants repair by tomorrow
```

Without relation:

```text
three local hits
```

With relation:

```text
cooling failure → customer damage risk → deadline requirement
```

## 9.1 relation table

Minimal form:

```sql
create table coherence_relation_layer (
  relation_uuid uuid primary key default gen_random_uuid(),
  relation_index bigint not null unique,

  relation_array uuid[] not null,

  relation_kind text not null default 'grammar_to_grammar',
  source_scope_uuid uuid null,
  source_observation_uuid uuid null,

  observed_count bigint not null default 1,
  coherence_weight numeric not null default 0,
  first_seen_at timestamptz not null default now(),
  last_seen_at timestamptz not null default now()
);
```

Suggested meaning:

```text
relation_uuid      = stable identity of the relation
relation_index     = deterministic or insertion-order relation index
relation_array     = ordered candidate UUIDs participating in the relation
relation_kind      = grammar_to_grammar, vocab_to_grammar, vocab_to_vocab, etc.
source_scope_uuid  = scope that produced the relation, if available
observed_count     = recurrence pressure
coherence_weight   = normalized relation strength
```

## 9.2 relation link table

For search and aggregation, a link table is recommended.

```sql
create table coherence_relation_link (
  relation_uuid uuid not null references coherence_relation_layer(relation_uuid),
  candidate_uuid uuid not null,
  candidate_kind text not null,
  position integer not null,
  role text null,

  primary key (relation_uuid, position)
);
```

Suggested candidate kinds:

```text
token
word
draft_vocabulary
adopted_vocabulary
draft_grammar
adopted_grammar
corrected_grammar
```

## 9.3 relation upsert

Relation can be upserted by a stable relation hash.

A production table may include `relation_hash`:

```sql
alter table coherence_relation_layer
add column relation_hash text unique;
```

Then:

```sql
insert into coherence_relation_layer (
  relation_index,
  relation_array,
  relation_hash,
  relation_kind,
  source_scope_uuid,
  source_observation_uuid
)
values (
  :relation_index,
  :relation_array,
  :relation_hash,
  :relation_kind,
  :source_scope_uuid,
  :source_observation_uuid
)
on conflict (relation_hash)
do update set
  observed_count = coherence_relation_layer.observed_count + 1,
  last_seen_at = now();
```

Relation is not optional metadata.

It is the memory that allows segmented grammar to remain connected.

---

# 10. Phase Relation Candidate Generation

Phase Attention should not run as a heavy synchronous reply-time process.

It should operate on normalized and aggregated tables.

Phase does not primarily inspect raw input.

It reads:

```text
adopted vocabulary table
adopted grammar table
draft vocabulary table
draft grammar table
coherence relation layer
relation aggregate weights
hit counts
coherence weights
co-occurrence frequencies
scope-crossing continuity
```

Phase uses these normalized aggregate values to generate relation candidates.

## 10.1 What Phase does

Phase slides across layers and searches for relation candidates.

```text
vocabulary layer
→ grammar layer
→ relation layer
→ corrected grammar layer
```

It may generate candidates such as:

```text
grammar A → grammar B
grammar B → grammar C
vocabulary bundle X → grammar A
vocabulary bundle Y → grammar B
grammar A + grammar C recur under the same use case
```

The goal is to discover relation candidates that are not directly present as one raw input span.

## 10.2 Phase should use aggregates

Phase should use pre-aggregated values such as:

```text
observed_count
coherence_weight
co_occurrence_count
relation_frequency
scope_transition_count
promotion_count
semantic_axis_candidate_count
```

This keeps reply-time processing light.

## 10.3 Phase output

Phase should output candidates, not mutate adopted structures directly.

Possible output tables:

```sql
create table phase_relation_candidate (
  phase_candidate_uuid uuid primary key default gen_random_uuid(),
  candidate_relation_array uuid[] not null,
  candidate_kind text not null default 'relation_candidate',
  source_aggregate_window text null,
  phase_score numeric not null default 0,
  evidence_count bigint not null default 0,
  status text not null default 'draft',
  created_at timestamptz not null default now(),
  last_seen_at timestamptz not null default now()
);
```

A candidate may later be reviewed, promoted, or used by the coherence decoder as draft relation evidence.

## 10.4 Runtime boundary

Synchronous reply path:

```text
input
→ token/word candidates
→ xi coherence hit
→ yj residual bank
→ relation lookup
→ zk coherence decoder
```

Asynchronous / scheduled path:

```text
normalized vocabulary and grammar tables
→ relation aggregates
→ Phase Attention
→ phase_relation_candidate
→ later promotion or decoder support
```

This prevents Phase from becoming a heavy bottleneck in ordinary response generation.

---

# 11. Why Relation Matters

If relation is thin, output will mirror input.

The system may still identify local words and grammar fragments, but it cannot connect them into a semantic flow.

With weak relation:

```text
input fragments in
similar fragments out
```

With strong relation:

```text
input fragments
→ connected grammar sequence
→ corrected grammar candidate
→ useful output
```

Therefore, relation growth is one of the main differences between:

```text
surface-level tokenizer
```

and:

```text
language-model-like behavior
```

A mature system should track not only vocabulary growth and grammar growth, but relation growth.

Useful growth metrics:

```text
relation_count
relation_density
average relation length
scope-crossing relation count
phase_relation_candidate count
relation promotion count
coherence decoder success rate
```

---

# 12. Why SQL Is Suitable Here

This layer is mostly:

```text
scope split
index assignment
hash generation
upsert
range tracking
count aggregation
relation insert/update
scheduled promotion
phase candidate generation
notify
```

These are database-friendly operations.

A C# or other worker may still be useful for:

- efficient streaming input
- complex regex libraries
- external file handling
- crawler integration
- background scheduling

But the semantic authority should remain in the database structures.

The worker should assist execution, not decide meaning.

---

# 13. Minimal First Implementation

A minimal implementation can start with:

```text
1. token_registry
2. word_registry
3. decoherence_bank
4. coherence_relation_layer
5. coherence_relation_link
6. split_flag tokens
7. scope_size = 1000
8. simple split rules
9. word_hash upsert
10. no-hit insert into decoherence_bank
11. relation upsert from adjacent candidate bundles
12. scheduled count-based promotion
13. scheduled Phase candidate generation from aggregates
```

No MeCab, UniDic, or language-specific tokenizer is required for the first version.

The tokenizer is:

```text
UUID indexer
boundary splitter
hash aggregator
```

The relation layer is:

```text
semantic continuity memory
```

The language model behavior emerges later from coherence hits, decoherence banking, relation formation, promotion, corrected grammar, and output projection.

---

# 14. Short Form

```text
Do not make the tokenizer smart.
Make the tokenizer stable.

Do not parse meaning at the entrance.
Index, split, hash, and upsert.

Known candidates go to xi.
Unknown candidates go to yj.
Connected candidates go to the coherence relation layer.
Aggregated relations feed Phase.
Corrected grammar output comes from zk.
```
