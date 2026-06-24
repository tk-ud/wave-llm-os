# Specification Draft: API Section Loop

## Authority

This draft defines the API-side section loop for split, think, generate, and audit orchestration.

It is routed by `SPEC_RUNTIME_ENGINE_BOUNDARY.md` and depends on:

```text
SPEC_API_THINKING_ENGINE_BOUNDARY.md
SPEC_SQL_RESPONSE_ENGINE_BOUNDARY.md
SPEC_TMP_CONTEXT_JSON_BOUNDARY.md
SPEC_REPLY_PIPELINE.md
```

---

# Core Principle

The API does not tokenize, split, verify, promote, or decide semantic authority.

The SQL Response Engine performs:

```text
input ingestion
tokenization
length-based splitting
corpus / decode
res_context generation
committed evidence reference generation
```

The API performs:

```text
section orchestration
tmp entry shaping
key retention
sequence tag management
sort / merge
context reconstruction
reinjection
delivery
```

---

# Section Kinds

Canonical API section kinds:

```text
split
think
generate
audit
```

The default section order is:

```text
split -> think -> generate -> audit
```

Each section follows the same loop shape.

---

# Generic Section Loop

```text
API receives source or reconstructed context
-> API calls SQL Response Engine with section_kind and section_instruction
-> SQL tokenizes / splits / corpus-decodes
-> SQL returns res fragments with order and evidence refs
-> API shapes key / sequence_tag / state / res_context into tmp_context
-> API keeps tmp_context_key and ordered_decode_keys
-> API sorts keys by sequence tags
-> API reconstructs decoded context
-> API reinjects reconstructed context into the next section
```

SQL owns the split.

API owns the merge.

---

# Section Instructions

## split

```text
section_kind = split
section_instruction = ingest and split the received context into ordered decoded context fragments
```

Purpose:

```text
source context -> ordered decoded context
```

## think

```text
section_kind = think
section_instruction = 以下のコンテキストから目的と出口候補を探索せよ
```

Purpose:

```text
ordered decoded context -> purpose candidates + exit candidates
```

## generate

```text
section_kind = generate
section_instruction = 以下のコンテキストから重複や冗長を排除せよ
```

Purpose:

```text
purpose / exit candidates -> reduced output candidate context
```

## audit

```text
section_kind = audit
section_instruction = 以下のコンテキストからinputコンテキストとoutputコンテキストを比較検証し、目的に対して正しく出力整形出来ているか監査せよ
```

Purpose:

```text
input decoded context + output decoded context -> audit result context
```

Audit is not semantic promotion.

Audit result is orchestration evidence for output branching.

---

# SQL Response Fragment Shape

SQL Response Engine section calls should return fragments shaped like:

```text
section_kind
section_run_id
res_key
sequence_tag
split_parent_token_index
split_position
res_context
source_refs
evidence_refs
state
```

`res_context` is SQL-produced decoded context.

It is not raw source context and not semantic authority.

`source_refs` and `evidence_refs` must preserve links back to committed reply-core structures and logs.

---

# API Temporary Context Entry Shape

API stores SQL-produced fragments into temporary context using entries shaped like:

```text
tmp_context_key
section_kind
section_run_id
res_key
sequence_tag
merge_order
state
res_context
source_refs
evidence_refs
parent_tmp_context_key
created_at
```

The API may add `merge_order`, `parent_tmp_context_key`, and orchestration state.

The API must not rewrite `res_context` as semantic authority.

---

# Ordering Rule

API context reconstruction must sort by stable ordering metadata.

Recommended order:

```text
section_order
merge_order
sequence_tag
split_position
res_key
```

SQL split output must preserve order using explicit position fields or ordered index arrays.

API reconstruction must not infer order from unordered JSON object keys.

---

# Reinjection Rule

A reconstructed decoded context may be reinjected into the next section.

Reinjected context must carry:

```text
parent_tmp_context_key
ordered_decode_keys
section lineage
source_refs
evidence_refs
```

Reinjection is orchestration input.

It is not semantic verification, promotion, or mutation authority.

---

# Output Rule

Only committed envelopes and committed decoded projections may be used for API output branching.

The API may emit partial or final output only after SQL-side commit success.

---

# Prohibited Patterns

```text
API must not tokenize.
API must not split text.
API must not decide length-based split policy.
API must not promote semantic structures.
API must not treat tmp_context as semantic authority.
API must not reconstruct context from unordered fragments.
```
