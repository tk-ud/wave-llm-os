# Specification Draft: API Thinking Engine Boundary

## Authority

This draft defines the API Thinking Engine boundary.

The API Thinking Engine is the API-side orchestration layer.

It controls continuation, stop, ask, defer, delivery, and output branching decisions from committed envelopes.

It does not own semantic mutation authority.

The API is a reception and delivery layer.

---

# Ownership

The API Thinking Engine may decide:

```text
continue_internal
continue_output
stop
ask_user
defer
abort
```

It may call Wave LLM for thinking orchestration.

It may call the SQL Response Engine.

It may branch only from committed runtime result envelopes.

Retry judgment belongs to SQL-side idempotency and response-engine results, not API-side semantic judgment.

---

# Default Serial Step Pipeline

The default serial thinking pipeline has four steps:

```text
split
think
generate
audit
```

Default `max_serial_steps`:

```text
max_serial_steps = 4
```

`max_serial_steps` is a tunable user / runtime performance setting.

The API Thinking Engine may reduce or extend the step limit according to configured performance policy, but the active limit must be recorded in temporary context or envelope metadata.

---

# Parallel Split Boundary

When a temporary context exists, context may be split and processed in parallel.

Parallel split work must store vocabulary and grammar through the SQL Response Engine before result ordering is exposed to API memory.

The API Thinking Engine may hold only ordered search keys for the split results.

```text
context split
-> parallel SQL Response Engine calls
-> vocabulary / grammar stored
-> ordered search keys returned
-> API memory keeps keys in original sequence order
```

The API Thinking Engine must not hold expanded split context bodies in memory.

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
ordered_search_keys
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

Partial output is allowed when the SQL Response Engine returns a committed corpus-derived candidate or output reference marked as `partial_output`.

This supports user experience without moving semantic authority into API memory.

`no_output` is allowed because context vocabulary and grammar may already be stored by the SQL Response Engine.

---

# Wave LLM Call Boundary

The API Thinking Engine may call Wave LLM for thinking orchestration.

Wave LLM should receive keys, summaries, budgets, and committed envelope metadata rather than expanded context bodies when possible.

Wave LLM must not replace SQL Response Engine verification, operation gate checks, or committed database evidence.

---

# Boundary Rule

The API Thinking Engine expands reasoning by orchestrating committed SQL Response Engine calls through temporary context keys, not by retaining expanded context in process memory.

The API layer is a reception and delivery layer.

Semantic judgment, retry authority, verification, and mutation authority remain database-backed through the SQL Response Engine and operation gate.
