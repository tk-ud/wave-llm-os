# Specification Draft: Temporary Context JSON Boundary

## Authority

This draft defines temporary context storage for runtime orchestration.

Temporary context is represented as `tmp_context.json`.

It is transient working context, not semantic authority.

---

# Role

`tmp_context.json` stores external working memory for the API Thinking Engine and SQL Response Engine.

It may contain routing hints, references, budgets, and step state.

It should not contain large canonical payloads when references are sufficient.

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

---

# API Memory Rule

The API Thinking Engine may retain only:

```text
tmp_context_key
tmp_context_version
small envelope metadata
```

Expanded temporary context must not be retained across thinking steps.

---

# SQL Access Rule

The SQL Response Engine receives the temporary context key and resolves only the sections needed for the current call.

`tmp_context.json` must not override PostgreSQL semantic authority or evidence authority.

---

# Expiry Rule

Temporary contexts must expire or be marked complete after final output, timeout, abort, or archival handoff.
