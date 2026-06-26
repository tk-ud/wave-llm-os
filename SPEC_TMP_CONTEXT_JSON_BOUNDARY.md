# Specification: Temporary Context JSON Boundary

## Authority

This file defines temporary decoded context storage for runtime control.

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
tmp_context_state
```

Recommended `tmp_context_state` values:

```text
active
merged
reinjected
completed
expired
failed
```

`tmp_context_state` is an API runtime label for temporary orchestration only.

It is not canonical semantic lifecycle truth.

---

# Storage Forms

Default implementation form:

```text
API server local temporary file: tmp_context.json
```

Other physical storage forms may exist only as implementation projections:

```text
tmp_context table jsonb
object storage
hybrid table + external body
```

The logical boundary is the same regardless of storage form.

The stored body is decoded context projection and orchestration state, not raw canonical reply context.

---

# Logging Boundary

`tmp_context.json` body must not be written to canonical logs.

Canonical logs preserve reply-core evidence through `logs.coherence` and mutation / decision evidence through `logs.diff`.

Temporary context may preserve only references needed for runtime control while the temporary file exists:

```text
tmp_context_key
tmp_context_version
tmp_context_hash
ordered_decode_keys
source_refs
evidence_refs
committed envelope references
```

If decoded context, residual, unstable structure, or unresolved evidence needs long-term preservation, it must be represented through canonical reply-core evidence, logs, or `decoherence_bank`, not by retaining `tmp_context.json`.

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

# Cleanup Rule

`tmp_context.json` is deleted after each API orchestration run completes, aborts, fails, times out, or reaches final output.

Temporary context is not a long-term trace, archive, retry store, or semantic evidence store.

Thinking-mode input is expected to enter the canonical reply pipeline and cohere into logged reply-core evidence.

Long-lived or reusable unresolved material belongs to canonical evidence structures or `decoherence_bank`, not to temporary context retention.
