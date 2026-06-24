# Specification Draft: Temporary Context JSON Boundary

## Authority

This draft defines temporary orchestration state storage for runtime control.

Temporary orchestration state may be represented as `tmp_context.json`.

It is transient orchestration state, not reply context authority and not semantic authority.

Reply-time core processing remains governed by `SPEC_REPLY_PIPELINE.md`.

---

# Role

`tmp_context.json` stores external working state for API orchestration.

It may contain routing hints, ordered search keys, references, budgets, step state, and committed envelope references.

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

Reply context bodies enter the core through the reply pipeline and are persisted as canonical structures before API memory receives ordered keys.

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

The stored body is orchestration state, not canonical reply context.

---

# API Memory Rule

The API Thinking Engine may retain only:

```text
tmp_context_key
tmp_context_version
ordered_search_keys
small envelope metadata
```

Expanded reply context must not be retained across thinking steps.

Expanded reply context must also not be treated as temporary context authority.

---

# SQL Access Rule

The SQL Response Engine receives temporary orchestration keys and resolves only the sections needed for the current call.

`tmp_context.json` must not override PostgreSQL semantic authority, reply pipeline state, or evidence authority.

---

# Expiry Rule

Temporary orchestration state must expire or be marked complete after final output, timeout, abort, or archival handoff.
