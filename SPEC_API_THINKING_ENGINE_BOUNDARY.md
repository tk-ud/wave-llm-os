# Specification: API Thinking Engine Boundary

## Authority

This file defines the API Thinking Engine boundary.

The API Thinking Engine is the API-side orchestration layer.

It controls continuation, stop, ask, defer, delivery, decoded-context merge, reinjection, and output branching decisions from committed envelopes.

Detailed split / think / generate / audit loop rules live in `SPEC_API_SECTION_LOOP.md`.

It does not own semantic mutation authority.

The API is a reception, merge, reinjection, and delivery layer.

Reply-time core processing remains governed by `SPEC_REPLY_PIPELINE.md`.

---

# Ownership

The API Thinking Engine may decide:

```text
continue_internal
continue_output
merge_decoded_context
reinject_context
stop
ask_user
defer
abort
```

It may call Wave LLM for thinking orchestration.

It may call the SQL Response Engine.

It may branch only from committed runtime result envelopes and SQL-produced decoded context projections.

Retry judgment belongs to SQL-side idempotency and response-engine results, not API-side semantic judgment.

---

# DB Instruction Boundary

The API Thinking Engine may instruct the database only through SQL Response Engine functions.

The API Thinking Engine must not use database tables as API-side working memory, orchestration state, semantic authority, or decision authority.

```text
API -> SQL Response Engine function call -> committed envelope / fragments / refs -> API orchestration
```

The API may keep local temporary runtime files such as `tmp_context.json` for merge and reinjection control.

Those temporary files are API-side orchestration artifacts, not database authority, not canonical logs, and not long-term storage.

Direct database table reads or writes from API orchestration would collapse the API / DB responsibility boundary.

---

# Default Serial Step Pipeline

The default serial thinking pipeline has four API orchestration steps:

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

The active limit must be recorded in temporary decoded context, orchestration state, or envelope metadata.

These API orchestration steps do not replace the synchronous reply-time core flow.

The `split` step is an orchestration request, not API-owned tokenization or text splitting.

---

# Reply Core Boundary

The API Thinking Engine must not move reply context bodies into temporary context storage as the source of truth.

Reply-time context enters the core through the canonical reply pipeline:

```text
input_observation
-> token / vocabulary / grammar candidates
-> near-neighbor retrieval
-> structural verification
-> input grammar / grammar_relation diff verification
-> coherence / residual handling
-> decoder
-> output collapse
-> evidence logs
```

The API Thinking Engine may orchestrate calls, but tokenization, text splitting, vocabulary, grammar, verification, promotion, residual handling, collapse, and evidence logging are core / SQL Response Engine responsibilities.

---

# Parallel Split and Decode Boundary

When reply work is split, the API requests split execution from the SQL Response Engine.

The SQL Response Engine decides tokenization and split boundaries, including length-based split behavior.

Split input must be persisted through the SQL Response Engine into canonical reply-time structures before decoded context projections are exposed to API merge.

The SQL Response Engine may then corpus / decode each split and return decoded projection fragments, keys, and refs to the API.

The API Thinking Engine may hold only ordered decode keys for split decoded context projections.

```text
API requests split
-> SQL Response Engine tokenizes / splits
-> input_observation / token / vocabulary / grammar stored
-> SQL-side corpus / decode
-> SQL returns decoded projection fragments / refs
-> API stores decoded projections in local tmp_context.json
-> ordered_decode_keys returned
-> API merges decoded projections in original sequence order
-> merged decoded context may be reinjected
```

The API Thinking Engine must not hold expanded split input bodies in memory.

---

# In-Memory Boundary

The API Thinking Engine may hold only small routing metadata in process memory:

```text
request_id
session_id
turn_id
step_id
tmp_context_key
orchestration_state_key
tmp_context_version
ordered_decode_keys
output_state
next_thinking_action
small envelope metadata
```

Expanded reply input belongs to canonical reply-time storage, not API memory and not temporary context as authority.

Decoded context projections may be merged through temporary context, but API memory should keep ordered keys and references whenever possible.

---

# Prohibited Payloads

The API Thinking Engine must not retain:

```text
expanded source conversation context
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

`no_output` is allowed because reply-time input, vocabulary, and grammar may already be stored by the SQL Response Engine.

---

# Context Reinjection Boundary

The API Thinking Engine may merge split decoded context projections and reinject the merged decoded context into a later Wave LLM or SQL Response Engine call.

Reinjected context must carry references to the temporary decoded context, ordered decode keys, and committed reply-core evidence that produced it.

Reinjection is orchestration input.

It is not semantic promotion, verification, or authority.

---

# Wave LLM Call Boundary

The API Thinking Engine may call Wave LLM for thinking orchestration.

Wave LLM may receive merged decoded context, keys, summaries, budgets, and committed envelope metadata.

Wave LLM must not replace SQL Response Engine verification, operation gate checks, or committed database evidence.

---

# Boundary Rule

The API Thinking Engine expands reasoning by orchestrating committed SQL Response Engine calls, merging SQL-produced decoded context projections, and reinjecting merged decoded context as orchestration input.

Semantic judgment, retry authority, tokenization, splitting, verification, and mutation authority remain database-backed through the SQL Response Engine and operation gate.

API orchestration may instruct database-backed computation, but it must not directly use database tables as API-side working memory or authority.
