# Supporting Note: Structural Near-Neighbor Search

Canonical authority: `SPEC_CANONICAL_CORE.md`.

---

# Rule

Near-neighbor search is structural over index arrays.

```text
index array = semantic vector
hierarchy   = meaning
```

Primary structures:

```text
vocabulary.token_array
grammar.vocabulary_array
grammar_relation.grammar_array
phase_relation_candidate.grammar_array
```

All are `bigint[]` index arrays.

---

# Scoring Inputs

Use:

```text
array overlap
ordered overlap
position / gap distance
prefix / suffix continuity
logs.current pressure
logs.diff history
decoherence_bank pressure
status / draft_flag / relation_weight
```

Loop-like repetition is treated as pressure, not a separate guard state.

---

# SQL Join Diff Examples

SQL join diff is the preferred structural verification method.

```text
inner join      = shared structure
left/right diff = missing / excess structure
full outer join = residual / symmetric difference
```

This is useful for decode-time inspection because it exposes exactly where structures match, diverge, disappear, or remain unresolved.

## Grammar vocabulary-array diff

Compare grammar structures by `vocabulary_index` membership and position.

```sql
with
g1 as (
  select vocabulary_index, ord
  from unnest(:grammar_a_vocabulary_array) with ordinality
    as t(vocabulary_index, ord)
),
g2 as (
  select vocabulary_index, ord
  from unnest(:grammar_b_vocabulary_array) with ordinality
    as t(vocabulary_index, ord)
)
select
  coalesce(g1.vocabulary_index, g2.vocabulary_index) as vocabulary_index,
  g1.ord as a_pos,
  g2.ord as b_pos,
  case
    when g1.vocabulary_index is not null and g2.vocabulary_index is not null then 'shared'
    when g1.vocabulary_index is not null then 'only_a'
    when g2.vocabulary_index is not null then 'only_b'
  end as diff_kind
from g1
full outer join g2
  on g1.vocabulary_index = g2.vocabulary_index;
```

This exposes:

```text
shared vocabulary
missing vocabulary
excess vocabulary
position differences
order drift
```

## Expanded vocabulary token diff

When grammar-level vocabulary differs, expand each vocabulary row into its `token_array` and compare token structure.

```sql
with
g1_tokens as (
  select
    gv.ord as vocab_pos,
    v.vocabulary_index,
    tok.token_index,
    tok.ord as token_pos
  from unnest(:grammar_a_vocabulary_array) with ordinality
    as gv(vocabulary_index, ord)
  join vocabulary v
    on v.vocabulary_index = gv.vocabulary_index
  cross join unnest(v.token_array) with ordinality
    as tok(token_index, ord)
),
g2_tokens as (
  select
    gv.ord as vocab_pos,
    v.vocabulary_index,
    tok.token_index,
    tok.ord as token_pos
  from unnest(:grammar_b_vocabulary_array) with ordinality
    as gv(vocabulary_index, ord)
  join vocabulary v
    on v.vocabulary_index = gv.vocabulary_index
  cross join unnest(v.token_array) with ordinality
    as tok(token_index, ord)
)
select
  coalesce(g1_tokens.token_index, g2_tokens.token_index) as token_index,
  g1_tokens.vocab_pos as a_vocab_pos,
  g2_tokens.vocab_pos as b_vocab_pos,
  g1_tokens.token_pos as a_token_pos,
  g2_tokens.token_pos as b_token_pos,
  case
    when g1_tokens.token_index is not null and g2_tokens.token_index is not null then 'shared_token'
    when g1_tokens.token_index is not null then 'only_a_token'
    when g2_tokens.token_index is not null then 'only_b_token'
  end as diff_kind
from g1_tokens
full outer join g2_tokens
  on g1_tokens.token_index = g2_tokens.token_index;
```

This exposes:

```text
shared token structure
missing token structure
excess token structure
vocabulary-boundary drift
token-position drift
```

Mapping to runtime concepts:

```text
inner join hit
→ xi coherence hit

outer join residual
→ yj decoherence / unresolved difference
```

---

# Vector / SQL Role Split

Vector search and SQL join diff have different roles.

```text
vector geometry
= basin shape / scale / density error inspection

SQL join diff
= shared structure / residual / missing / excess verification
```

Recommended use:

```text
sleep integration
→ vector geometry first
→ SQL join diff verification second
→ merge / alias / dormant decisions remain draft evidence

decode / reply-time verification
→ SQL join diff first
→ vector distance is supporting evidence only
```

---

# pgvector

Allowed:

```text
pgvector as derived retrieval accelerator
```

Rejected:

```text
pgvector as semantic authority
```

Flow:

```text
pgvector retrieves candidates
structural verification decides
```

Dense vector similarity may help candidate retrieval, but index-array verification decides semantic fit.

---

# Missing Slot Completion

```text
grammar_array
→ grammar_index search
→ vocabulary_index search
→ token_index search
```

Do not start from raw tokens when higher-level context exists.

Token-level descent is last resort.

---

# Loop Escape

No dedicated `loop_guard` table.

Repeated grammar return expands search through:

```text
grammar_array alternatives
phase_relation_candidate.grammar_array
near-neighbor candidates
decoherence pressure
logs.current pressure
```
