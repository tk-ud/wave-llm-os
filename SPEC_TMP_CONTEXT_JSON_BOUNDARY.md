# Specification Draft: Temporary Context JSON Boundary

## Authority

This draft defines temporary decoded context storage for runtime control.

Temporary decoded context may be represented as `tmp_context.json`.

It stores SQL-produced corpus / decoded context projections for API-side merge and reinjection.

It is not raw reply context authority and not semantic authority.

Reply-time core processing remains governed by `SPEC_REPLY_PIPELINE.md`.

---

# Role

`tmp_context.json` stores decoded context projections produced by the SQL Response Engine after corpus / decode work.

It may contain:

```text
decoded_context_fragments
ordered_decode_keys
merge_plan
routing hints
budgets
step state
committed envelope references
```

It must not replace canonical reply-time structures such as:

```text
input_observation
token
vocabulary
grammar
grammar_relation
logs.coherence
logs.diff
```

Raw reply context bodies enter the core through the reply pipeline and are persisted as canonical structures before decoded context projections are written to temporary context.

---

# Decode Projection Rule

The SQL Response Engine owns corpus and decode work.

Temporary context may store only decoded projections or references produced after SQL-side corpus / decode processing.

API-side merge may read or reference these decoded projections, but API-side merge does not make them semantic authority.

---

# API Merge and Reinjection Rule

The API Thinking Engine may merge split decoded context projections in original sequence order.

The merged decoded context may be reinjected as orchestration input for a later Wave LLM or SQL Response Engine call.

Reinjection must preserve references back to the committed reply-core structures and logs that produced the decoded projection.

---

# Identity Fields

Required identity fields:

```text
tmp_context_key
tmp_context_version
tmp_context_hash
expires_at
status
```

Recommended `status` values:

```text
active
merged
reinjected
completed
expired
failed
archived
```

---

# Storage Forms

Physical storage may be:

```text
tmp_context table jsonb
local temporary file
object storage
hybrid table + external body
```

The logical boundary is the same regardless of storage form.

The stored body is decoded context projection and orchestration state, not raw canonical reply context.

---

# API Memory Rule

The API Thinking Engine may retain:

```text
tmp_context_key
tmp_context_version
ordered_decode_keys
small envelope metadata
```

Expanded raw reply context must not be retained across thinking steps.

Decoded context bodies may be merged through temporary context, but API memory should retain references and ordered keys whenever possible.

---

# SQL Access Rule

The SQL Response Engine receives temporary context keys and resolves only the sections needed for the current call.

`tmp_context.json` must not override PostgreSQL semantic authority, reply pipeline state, or evidence authority.

---

# Expiry Rule

Temporary decoded context must expire or be marked complete after final output, timeout, abort, merge completion, reinjection completion, or archival handoff.
