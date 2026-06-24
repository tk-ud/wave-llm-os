# Specification Draft: Runtime Result Envelope

## Authority

This draft defines the runtime result envelope returned by the SQL Response Engine to the API Thinking Engine.

The envelope is a small committed result summary.

It is not semantic authority.

---

# Commit Rule

The SQL Response Engine may return an envelope before SQL commit completes.

The API Thinking Engine may use the envelope only after commit success.

```text
SQL function result != visible output
SQL commit success + envelope = visible output candidate
```

A failed transaction invalidates the envelope.

---

# Canonical Fields

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

---

# output_state Values

```text
no_output
partial_output
final_output
ask_user
deferred
blocked
error
```

---

# next_thinking_action Values

```text
stop
call_response_engine
wait_scheduler
ask_user
retry
abort
```

---

# Output Text Rule

`output_text_ref` should reference committed output material.

Large text bodies should not be embedded in the envelope when a stable reference is available.

---

# Branching Rule

The API Thinking Engine branches from `output_state` and `next_thinking_action` only after commit success.
