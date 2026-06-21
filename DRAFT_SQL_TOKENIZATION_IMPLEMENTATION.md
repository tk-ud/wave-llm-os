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

Meaning is decided later by coherence search, decoherence banking, promotion, and decoding.

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

## 7.3 zk: coherence decoder

When enough hit bundle evidence exists:

```text
z → corrected grammar candidate k → output projection
```

The tokenizer does not produce final output.

It only prepares stable candidates for the coherence/decoherence pipeline.

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

# 9. Why SQL Is Suitable Here

This layer is mostly:

```text
scope split
index assignment
hash generation
upsert
range tracking
count aggregation
scheduled promotion
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

# 10. Minimal First Implementation

A minimal implementation can start with:

```text
1. token_registry
2. word_registry
3. decoherence_bank
4. split_flag tokens
5. scope_size = 1000
6. simple split rules
7. word_hash upsert
8. no-hit insert into decoherence_bank
9. scheduled count-based promotion
```

No MeCab, UniDic, or language-specific tokenizer is required for the first version.

The tokenizer is:

```text
UUID indexer
boundary splitter
hash aggregator
```

The language model behavior emerges later from coherence hits, decoherence banking, promotion, corrected grammar, and output projection.

---

# 11. Short Form

```text
Do not make the tokenizer smart.
Make the tokenizer stable.

Do not parse meaning at the entrance.
Index, split, hash, and upsert.

Known candidates go to xi.
Unknown candidates go to yj.
Corrected grammar output comes from zk.
```
