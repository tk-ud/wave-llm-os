# Specification Draft: API Thinking Engine Boundary

## Authority

This draft defines the API Thinking Engine boundary.

The API Thinking Engine is the API-side orchestration layer.

It controls continuation, stop, ask, retry, defer, and output branching decisions.

It does not own semantic mutation authority.

---

# Ownership

The API Thinking Engine may decide:

```text
continue_internal
continue_output
stop
ask_user
defer
retry
abort
```

It may call the SQL Response Engine.

It may branch only from committed runtime result envelopes.

---

# In-Memory Boundary

The API Thinking Engine may hold only small routing metadata in process memory:

```text
request_id
session_id
turn_id
step_id
tmp_context_key
tmp_context_version
output_state
next_thinking_action
small envelope metadata
```

Expanded context belongs to temporary context storage, not API memory.

---

# Prohibited Payloads

The API Thinking Engine must not retain:

```text
expanded conversation context
large candidate sets
evidence bodies
log payloads
grammar / relation search result bodies
uncommitted semantic mutation candidates
```

---

# Output Branching

The API Thinking Engine maps committed envelopes to API behavior:

```text
no_output      -> continue internally or wait
partial_output -> emit partial output and possibly continue
final_output   -> emit final output and stop
ask_user       -> ask the user
blocked        -> return blocked response
deferred       -> return queued / accepted response
error          -> return error response
```

---

# Boundary Rule

The API Thinking Engine expands reasoning by orchestrating committed SQL Response Engine calls through temporary context keys, not by retaining expanded context in process memory.
