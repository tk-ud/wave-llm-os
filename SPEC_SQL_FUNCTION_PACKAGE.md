# Specification Draft: SQL Function Package

## Authority

This draft defines the SQL Response Engine function package contract.

It wires existing runtime, reply, verification, operation gate, scheduler, temporary context, and envelope specifications into concrete SQL entry points.

It does not redefine semantic authority, reply-time processing, verification, promotion policy, scheduler semantics, temporary context authority, or envelope commit rules.

When this file conflicts with a routed runtime or core `SPEC_*` file, the routed `SPEC_*` file wins.

Related wiring package:

```text
SPEC_DOCUMENT_WIRING.md -> SQL Runtime Design Package
```

---

# Package Namespace

Canonical SQL package namespace:

```text
response_engine
```

The SQL Response Engine is a PostgreSQL function package for committed database-backed response computation.

It must not own API-side thinking orchestration or user-visible streaming control.

---

# Entry Function Families

Canonical entry function families:

```text
response_engine.run
response_engine.reply_step
response_engine.ingest_input
response_engine.tokenize_and_split
response_engine.decode_context
response_engine.build_envelope
response_engine.operation_step
response_engine.scheduler_step
```

`response_engine.run(...)` is the public dispatcher entry.

Purpose-specific functions remain directly callable for tests, migrations, invariant checks, and bounded internal composition.

---

# Dispatcher Rule

`response_engine.run(...)` dispatches by `route_kind`.

```text
route_kind = reply      -> response_engine.reply_step
route_kind = ingest     -> response_engine.ingest_input
route_kind = split      -> response_engine.tokenize_and_split
route_kind = decode     -> response_engine.decode_context
route_kind = operation  -> response_engine.operation_step
route_kind = scheduler  -> response_engine.scheduler_step
route_kind = envelope   -> response_engine.build_envelope
```

Internal dispatch may branch by route kind, section kind, job kind, operation key, split policy, and runtime options.

Dispatch must remain inside the SQL Response Engine boundary.

The API Thinking Engine may request a route, but it must not perform tokenization, split policy selection, semantic verification, semantic promotion, or mutation authority decisions.

---

# Common Contract

All mutation-capable or evidence-writing calls must carry an idempotency key.

Common arguments:

```text
p_request_id uuid
p_idempotency_key text
p_runtime_options jsonb
```

Common runtime options may include:

```text
max_rows_per_run
max_iterations_per_run
max_runtime_hint_ms
watermark
lease_key
```

SQL loops must be bounded.

A missing idempotency key is allowed only for read-only invariant inspection or local development probes explicitly marked as non-mutating.

---

# Transaction Boundary

Canonical transaction shape:

```text
1 SQL response-engine step
= 1 bounded transaction scope
= 1 response_engine_run_id
= 1 provisional runtime envelope
```

The runtime envelope returned by a SQL function is provisional until the surrounding SQL transaction commits.

A failed transaction invalidates:

```text
runtime envelope
section fragments
temporary decoded projections
log references returned by that call
```

The API Thinking Engine may use envelopes and decoded projections only after SQL commit success.

---

# Return Rule

SQL Response Engine entry functions return a runtime envelope shape aligned with `SPEC_RUNTIME_RESULT_ENVELOPE.md`.

Conceptual return type:

```sql
returns response_engine.runtime_envelope
```

Large output text bodies should not be embedded in the envelope when a stable committed reference is available.

Section fragments and decoded projections are referenced from the envelope; they are not user-visible output by themselves.

---

# response_engine.run

Canonical dispatcher shape:

```sql
response_engine.run(
  p_request_id uuid,
  p_route_kind text,
  p_section_kind text default null,
  p_payload jsonb default '{}'::jsonb,
  p_idempotency_key text default null,
  p_runtime_options jsonb default '{}'::jsonb
)
returns response_engine.runtime_envelope
```

`p_payload` is route-specific.

The dispatcher must validate that the route and payload are compatible before calling the purpose-specific function.

---

# response_engine.reply_step

Canonical reply step shape:

```sql
response_engine.reply_step(
  p_request_id uuid,
  p_session_id uuid,
  p_turn_id uuid,
  p_section_kind text,
  p_input_ref jsonb,
  p_parent_tmp_context_key text default null,
  p_ordered_decode_keys text[] default null,
  p_section_lineage jsonb default '{}'::jsonb,
  p_idempotency_key text,
  p_runtime_options jsonb default '{}'::jsonb
)
returns response_engine.runtime_envelope
```

Allowed section kinds:

```text
split
think
generate
audit
```

`reply_step(...)` may orchestrate SQL-owned reply-time work for one section.

It may call ingestion, tokenization, split, decode, verification, evidence logging, operation gate checks, fragment writes, temporary decoded projection writes, and envelope generation inside the SQL boundary.

It must not emit user-visible output directly.

It must not replace API continuation decisions.

---

# response_engine.ingest_input

Canonical ingest shape:

```sql
response_engine.ingest_input(
  p_request_id uuid,
  p_source_kind text,
  p_body text,
  p_metadata_json jsonb default '{}'::jsonb,
  p_idempotency_key text,
  p_runtime_options jsonb default '{}'::jsonb
)
returns response_engine.runtime_envelope
```

Responsibilities:

```text
create or resolve input_observation
compute source_hash
preserve source_kind metadata
write committed input references
return envelope references
```

Source differences are metadata, not separate semantic authority.

---

# response_engine.tokenize_and_split

Canonical tokenization and split shape:

```sql
response_engine.tokenize_and_split(
  p_request_id uuid,
  p_observation_ref jsonb,
  p_split_policy_ref jsonb default '{}'::jsonb,
  p_idempotency_key text,
  p_runtime_options jsonb default '{}'::jsonb
)
returns response_engine.runtime_envelope
```

Responsibilities:

```text
tokenize input
create or resolve token rows
create or resolve vocabulary / grammar candidates
apply SQL-owned split policy
preserve split_parent_token_index
preserve split_position
write ordered section fragments when split output is exposed
```

The API may request split execution, but it must not decide token boundaries, vocabulary boundaries, grammar boundaries, or length-based split policy.

Default long-span split policy remains owned by the SQL Response Engine boundary.

---

# response_engine.decode_context

Canonical decode shape:

```sql
response_engine.decode_context(
  p_request_id uuid,
  p_section_kind text,
  p_input_refs jsonb,
  p_parent_tmp_context_key text default null,
  p_ordered_decode_keys text[] default null,
  p_idempotency_key text,
  p_runtime_options jsonb default '{}'::jsonb
)
returns response_engine.runtime_envelope
```

Responsibilities:

```text
perform corpus / decode work
produce decoded context projections
preserve references to committed reply-core structures
preserve evidence_refs
write section fragments
write or update temporary decoded context projections
return ordered decode references through the envelope
```

Decoded context projections are not raw reply context authority and not semantic authority.

---

# response_engine.build_envelope

Canonical envelope build shape:

```sql
response_engine.build_envelope(
  p_request_id uuid,
  p_response_engine_run_id uuid,
  p_route_kind text,
  p_effect_refs jsonb default '{}'::jsonb,
  p_error_json jsonb default null,
  p_idempotency_key text,
  p_runtime_options jsonb default '{}'::jsonb
)
returns response_engine.runtime_envelope
```

Responsibilities:

```text
assemble committed effect references
set output_state
set next_thinking_action
attach evidence_refs
attach coherence_log_indexes
attach diff_log_indexes
attach aggregate_refs
attach scheduler_job_run_refs
attach archive_indexes
attach error_kind / error_json when needed
```

Envelope construction does not create semantic authority.

---

# response_engine.operation_step

Canonical operation step shape:

```sql
response_engine.operation_step(
  p_request_id uuid,
  p_operation_key text,
  p_target_ref jsonb,
  p_evidence_refs jsonb default '{}'::jsonb,
  p_idempotency_key text,
  p_runtime_options jsonb default '{}'::jsonb
)
returns response_engine.runtime_envelope
```

Responsibilities:

```text
call core_can_execute(operation_key)
apply allowed semantic mutation only after gate success
write logs.diff
write logs.coherence when observation evidence is produced
return envelope references
```

Canonical operation keys are defined by `SPEC_OPERATION_GATE.md`.

No `adopt.*` operation key is canonical.

---

# response_engine.scheduler_step

Canonical scheduler step shape:

```sql
response_engine.scheduler_step(
  p_request_id uuid,
  p_job_kind text,
  p_job_ref jsonb default '{}'::jsonb,
  p_idempotency_key text,
  p_runtime_options jsonb default '{}'::jsonb
)
returns response_engine.runtime_envelope
```

Canonical job kinds:

```text
current_refresh
phase_candidate_generation
sleep_consolidation
archive_rolloff
```

Responsibilities:

```text
claim bounded job work
refresh aggregate.current
append logs.current snapshots when configured
generate phase_relation_candidate rows
route unstable structures to decoherence_bank through allowed operation gate paths
write archive.registry rows
write scheduler_job_run evidence
return envelope references
```

Scheduler job kinds are a separate namespace from operation keys.

Promotion is not a scheduler job.

---

# Error and Retry Policy

Errors must be represented through the runtime envelope fields:

```text
error_kind
error_json
output_state = error | blocked | deferred | no_output
next_thinking_action = retry | abort | wait_scheduler | ask_user | stop
```

Retry authority belongs to SQL-side idempotency and response-engine results, not API-side semantic judgment.

A retry with the same idempotency key must not duplicate committed semantic mutations, log writes, section fragments, temporary context projections, scheduler job runs, or archive registry rows.

---

# Prohibited Behavior

```text
SQL Response Engine must not emit user-visible output directly.
SQL Response Engine must not perform external HTTP/API side effects.
SQL Response Engine must not replace API Thinking Engine continuation decisions.
SQL Response Engine must not promote without operation gate authority.
SQL Response Engine must not treat decoded context projections as semantic authority.
SQL Response Engine must not use UUID arrays as semantic reference paths.
SQL Response Engine must not reintroduce decoder_trace, loop_guard, status enum authority, adoption_audit, or adopt.* operation keys.
```
