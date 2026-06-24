# Specification Draft: Runtime Engine Boundary

## Authority

This draft defines the runtime boundary between the API Thinking Engine, the SQL Response Engine, temporary context storage, and database scheduled jobs.

This file is a draft. It is not canonical until routed from the core resume.

---

# Runtime Roles

```text
API Thinking Engine = API-side orchestration and continuation control
SQL Response Engine = PostgreSQL function package for committed response computation
Temporary Context   = tmp_context.json working-memory projection
PostgreSQL Tables   = semantic authority, evidence authority, and pressure surfaces
DB Scheduled Jobs   = bounded database jobs callable through SQL functions
```

The API Thinking Engine and SQL Response Engine are separate runtime roles.

They must not be collapsed into one scheduler.

---

# API Thinking Engine Boundary

The API Thinking Engine owns external orchestration.

It may decide whether to continue, stop, ask the user, defer, retry, or finish.

It may hold only small routing metadata in process memory:

```text
request_id
session_id
turn_id
step_id
tmp_context_key
tmp_context_version
output_state
next_thinking_action
small response envelope metadata
```

Expanded context, candidate bodies, evidence bodies, log payloads, and search result bodies belong outside API memory.

---

# SQL Response Engine Boundary

The SQL Response Engine is a PostgreSQL function package.

It owns committed database-backed response computation:

```text
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
runtime result envelope generation
```

The SQL Response Engine returns a runtime result envelope.

The envelope is usable by the API Thinking Engine only after the surrounding SQL transaction commits successfully.

---

# Temporary Context JSON Boundary

Temporary working context is represented as `tmp_context.json`.

`tmp_context.json` is a transient working-context projection, not semantic authority.

The API Thinking Engine may retain only the temporary context key, version, and small envelope metadata.

The SQL Response Engine receives the key and resolves only the sections needed for the current function call.

Required identity fields:

```text
tmp_context_key
tmp_context_version
tmp_context_hash
expires_at
status
```

---

# Runtime Result Envelope

The SQL Response Engine returns a small envelope to the API Thinking Engine.

Canonical envelope fields:

```text
request_id
response_engine_run_id
route_kind
committed_effect_kind
output_state
output_text_ref
output_delta_kind
next_thinking_action
suggested_request_kind
suggested_context_json
evidence_refs
coherence_log_indexes
diff_log_indexes
aggregate_refs
scheduler_job_run_refs
archive_indexes
error_kind
error_json
created_at
```

Recommended `output_state` values:

```text
no_output
partial_output
final_output
ask_user
deferred
blocked
error
```

Recommended `next_thinking_action` values:

```text
stop
call_response_engine
wait_scheduler
ask_user
retry
```

---

# DB Scheduled Jobs

DB Scheduled Jobs are not the API Thinking Engine.

They are bounded database jobs callable through SQL functions.

Canonical job kinds remain defined by `SPEC_CRON_PIPELINE.md`:

```text
current_refresh
phase_candidate_generation
sleep_consolidation
archive_rolloff
```

Scheduled jobs must be batch-bounded.

Recommended execution controls:

```text
max_rows_per_run
max_iterations_per_run
max_runtime_hint_ms
watermark
lease_key
idempotency_key
```

---

# Commit Boundary

No user-visible output may be emitted before SQL commit success.

```text
SQL function result != visible output
SQL commit success + envelope = visible output candidate
```

The API Thinking Engine may branch only from committed envelopes.

---

# Prohibited Cross-Boundary Behavior

```text
API Thinking Engine must not directly write semantic tables.
API Thinking Engine must not directly write logs or aggregate.current.
API Thinking Engine must not hold expanded temporary context across steps.
SQL Response Engine must not emit user-visible output directly.
SQL loops must be bounded.
DB Scheduled Jobs must not directly promote semantic structures.
Vector distance must not promote semantic structures.
tmp_context.json must not become semantic authority.
```
