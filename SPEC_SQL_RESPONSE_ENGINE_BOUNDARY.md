# Specification Draft: SQL Response Engine Boundary

## Authority

This draft defines the SQL Response Engine boundary.

The SQL Response Engine is a PostgreSQL function package.

It owns committed database-backed response computation.

It does not own API-side thinking orchestration or user-visible streaming control.

---

# Ownership

The SQL Response Engine may perform:

```text
input ingestion
tokenization
length-based splitting
semantic search
structural verification
input / relation diff checks
operation gate checks
logs.coherence writes
logs.diff writes
aggregate.current refreshes
phase candidate generation
sleep consolidation
archive rolloff
corpus / decode work
decoded context projection writes
runtime result envelope generation
```

Decoded context projection writes are temporary response projections.

They must preserve references to committed reply-core structures and evidence.

They do not create semantic authority.

---

# Tokenization and Split Ownership

The SQL Response Engine owns tokenization and text splitting.

The API Thinking Engine may request split execution, but it must not decide token boundaries, vocabulary boundaries, grammar boundaries, or length-based split policy.

Length-based split rules must be implemented inside SQL functions or SQL-owned policy tables.

SQL split output must preserve order by explicit position fields or ordered index arrays.

---

# Function Package Boundary

The SQL Response Engine must expose explicit entry functions.

Recommended entry families:

```text
response_engine.run
response_engine.reply_step
response_engine.ingest_input
response_engine.tokenize_and_split
response_engine.scheduler_step
response_engine.operation_step
response_engine.decode_context
response_engine.build_envelope
```

Internal function dispatch may branch by route kind, job kind, operation key, and split policy.

Dispatch must remain inside the SQL Response Engine boundary.

---

# Transaction Boundary

The SQL Response Engine returns a runtime result envelope.

The envelope is provisional until the surrounding SQL transaction commits.

A failed transaction invalidates the envelope.

Decoded context projections are usable for API merge only after commit success.

---

# Bounded Execution

SQL loops must be bounded.

Recommended bounds:

```text
max_rows_per_run
max_iterations_per_run
max_runtime_hint_ms
watermark
idempotency_key
```

---

# Prohibited Behavior

```text
SQL Response Engine must not emit user-visible output directly.
SQL Response Engine must not perform external HTTP/API side effects.
SQL Response Engine must not replace API Thinking Engine continuation decisions.
SQL Response Engine must not promote without operation gate authority.
Decoded context projections must not replace semantic authority.
```
