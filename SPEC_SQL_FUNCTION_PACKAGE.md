# Specification: SQL Function Package

## Authority

This file defines the public SQL Response Engine function package contract.

It decides SQL function entry shape, dispatcher policy, transaction boundary, idempotency rule, retry classification, and internal dispatch limits.

It does not define physical table DDL, storage layout, index implementation, or `tmp_context` table schema.

When this file conflicts with lower-level runtime boundary rules, the routed boundary specification wins.

Related authorities:

```text
SPEC_SQL_RESPONSE_ENGINE_BOUNDARY.md
SPEC_API_SECTION_LOOP.md
SPEC_RUNTIME_RESULT_ENVELOPE.md
SPEC_TMP_CONTEXT_JSON_BOUNDARY.md
SPEC_DB_SCHEDULED_JOB_BOUNDARY.md
SPEC_OPERATION_GATE.md
SPEC_REFERENCE_MODEL.md
SPEC_PROHIBITED_CANONICAL_PATTERNS.md
```

---

# Core Rule

`response_engine` is a PostgreSQL function package for committed response computation.

The API Thinking Engine may call the package, but the SQL Response Engine owns database-backed work:

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

The package must not emit user-visible output directly.

Runtime envelopes and decoded projections are API-usable only after SQL commit success.

---

# Public Entry Strategy

The SQL Response Engine exposes both:

```text
response_engine.run(...)
response_engine.reply_step(...)
response_engine.ingest_input(...)
response_engine.tokenize_and_split(...)
response_engine.decode_context(...)
response_engine.build_envelope(...)
response_engine.operation_step(...)
response_engine.scheduler_step(...)
```

`response_engine.run(...)` is a public dispatcher.

It is allowed only as a thin routing entry over the explicit public functions.

It must not become a hidden orchestration engine, semantic authority, scheduler authority, or operation-gate bypass.

Direct function entries remain canonical and callable.

This keeps generic API / worker calls ergonomic while keeping responsibility boundaries inspectable.

---

# Route Kinds

Canonical `route_kind` values for `response_engine.run(...)`:

```text
reply_step
ingest_input
tokenize_and_split
decode_context
build_envelope
operation_step
scheduler_step
```

`route_kind` selects the public function family.

It must not be inferred from a status enum, JSON object key order, user-visible text, or scheduler queue state alone.

---

# Common Input Contract

Every public entry must carry an explicit idempotency scope.

Common logical inputs:

```text
request_id uuid
idempotency_key text
budget_json jsonb
caller_context_json jsonb
```

`request_id` is row / event / run identity.

It is not a semantic reference.

`idempotency_key` is required for write-capable calls.

`budget_json` may include bounded execution controls:

```text
max_rows_per_run
max_iterations_per_run
max_runtime_hint_ms
watermark
lease_key
```

`caller_context_json` may contain only small routing metadata, keys, ordered decode keys, envelope references, budgets, or orchestration state.

It must not carry expanded reply source context as authority.

---

# Common Result Contract

Public entries return a logical result containing:

```text
runtime_result_envelope
section_fragments[]
ordered_decode_keys
```

`runtime_result_envelope` follows `SPEC_RUNTIME_RESULT_ENVELOPE.md`.

`section_fragments[]` follows `SPEC_SECTION_FRAGMENT_SCHEMA.md`.

A function may return zero fragments when the route only mutates evidence, refreshes aggregates, blocks, errors, or queues work.

The returned result is provisional until the surrounding SQL transaction commits.

A failed transaction invalidates the result.

---

# Function Signatures

The signatures below are canonical logical signatures.

Concrete SQL implementations may choose named composite types, `jsonb`, `SETOF`, or wrapper records as long as the logical contract is preserved.

## response_engine.run

```sql
response_engine.run(
  request_id uuid,
  route_kind text,
  route_json jsonb,
  idempotency_key text,
  budget_json jsonb default '{}'::jsonb,
  caller_context_json jsonb default '{}'::jsonb
) returns response_engine_result
```

Responsibilities:

```text
validate route_kind
validate idempotency scope
dispatch to the matching public function family
return the matching common result contract
```

It must not perform semantic work outside the dispatched SQL Response Engine route.

## response_engine.reply_step

```sql
response_engine.reply_step(
  request_id uuid,
  section_kind text,
  input_ref_json jsonb,
  idempotency_key text,
  budget_json jsonb default '{}'::jsonb,
  caller_context_json jsonb default '{}'::jsonb
) returns response_engine_result
```

Allowed `section_kind` values:

```text
split
think
generate
audit
```

Responsibilities:

```text
execute one SQL-backed API section request
preserve section lineage
produce committed evidence refs
produce decoded context fragments when applicable
build a runtime result envelope
```

`reply_step` is not API-side thinking.

It executes the database-backed work requested by the API section loop.

## response_engine.ingest_input

```sql
response_engine.ingest_input(
  request_id uuid,
  source_kind text,
  source_ref_json jsonb,
  idempotency_key text,
  budget_json jsonb default '{}'::jsonb,
  caller_context_json jsonb default '{}'::jsonb
) returns response_engine_result
```

Responsibilities:

```text
persist input through canonical reply-time entry structures
create or resolve committed input observation references
prepare evidence refs for later tokenization / split / decode work
build a runtime result envelope
```

`ingest_input` does not make temporary decoded context semantic authority.

## response_engine.tokenize_and_split

```sql
response_engine.tokenize_and_split(
  request_id uuid,
  observation_ref_json jsonb,
  idempotency_key text,
  budget_json jsonb default '{}'::jsonb,
  split_policy_ref_json jsonb default '{}'::jsonb,
  caller_context_json jsonb default '{}'::jsonb
) returns response_engine_result
```

Responsibilities:

```text
tokenize input
apply SQL-owned length-based split policy
write or resolve token / vocabulary / grammar-side references as required
preserve split_parent_token_index and split_position for child rows
return ordered section fragments or references when applicable
build a runtime result envelope
```

The API may request this function, but it must not decide token boundaries, vocabulary boundaries, grammar boundaries, or length-based split policy.

## response_engine.decode_context

```sql
response_engine.decode_context(
  request_id uuid,
  section_kind text,
  context_ref_json jsonb,
  idempotency_key text,
  budget_json jsonb default '{}'::jsonb,
  caller_context_json jsonb default '{}'::jsonb
) returns response_engine_result
```

Responsibilities:

```text
resolve only the required committed context references
perform corpus / decode work
write decoded context projection when applicable
return section fragments with source_refs and evidence_refs
build a runtime result envelope
```

Decoded context projection is not raw reply context authority and not semantic authority.

## response_engine.build_envelope

```sql
response_engine.build_envelope(
  request_id uuid,
  route_kind text,
  committed_effect_json jsonb,
  idempotency_key text,
  budget_json jsonb default '{}'::jsonb,
  error_json jsonb default '{}'::jsonb
) returns runtime_result_envelope
```

Responsibilities:

```text
assemble canonical envelope fields
attach committed evidence refs
attach output_state and next_thinking_action
attach error_kind / error_json when needed
```

`build_envelope` does not make commit happen.

The API may branch from the envelope only after transaction commit success.

## response_engine.operation_step

```sql
response_engine.operation_step(
  request_id uuid,
  operation_key text,
  target_ref_json jsonb,
  evidence_json jsonb,
  idempotency_key text,
  budget_json jsonb default '{}'::jsonb,
  caller_context_json jsonb default '{}'::jsonb
) returns response_engine_result
```

Responsibilities:

```text
call core_can_execute(operation_key)
apply allowed operation-gated mutation when permitted
write logs.diff decision evidence
write logs.coherence evidence when applicable
build a runtime result envelope
```

All semantic mutation must pass operation gate authority.

No `adopt.*` operation key is canonical.

## response_engine.scheduler_step

```sql
response_engine.scheduler_step(
  request_id uuid,
  job_kind text,
  scheduler_ref_json jsonb,
  idempotency_key text,
  budget_json jsonb default '{}'::jsonb,
  caller_context_json jsonb default '{}'::jsonb
) returns response_engine_result
```

Canonical `job_kind` values are routed from the scheduled job boundary:

```text
current_refresh
phase_candidate_generation
sleep_consolidation
archive_rolloff
```

Responsibilities:

```text
claim or validate bounded job execution scope
process only bounded rows / iterations
write scheduler_job_run evidence
write logs / aggregate / archive evidence as appropriate
build a runtime result envelope
```

A scheduler job may generate candidates, refresh aggregates, consolidate evidence, or archive rows.

It must not replace operation-gated promotion.

---

# Transaction Boundary

Each public SQL Response Engine call is one logical atomic unit.

The caller may open the SQL transaction, but the function result is not API-usable until that transaction commits.

Required ordering inside the transaction:

```text
validate route / key / budget
perform bounded SQL-owned work
write canonical rows / logs / decoded projections as needed
build runtime result envelope
return result
commit succeeds
API may branch / merge / reinject / deliver
```

If any required write fails, the transaction must fail or return a committed `error` / `blocked` envelope with no partial semantic mutation outside the declared route.

`operation_step` must include operation-gate check, mutation, and `logs.diff` evidence in the same transaction.

`scheduler_step` must include job-run evidence and bounded work evidence in the same transaction.

---

# Idempotency Rule

Idempotency is required for every write-capable public entry.

Canonical idempotency scope:

```text
route_kind
request_id
section_kind | job_kind | operation_key
input / target reference identity
idempotency_key
payload_hash
```

`payload_hash` is a deduplication key for the function request payload.

It is not a semantic reference key.

Repeated calls with the same idempotency scope and same payload must return the committed prior result when available.

Repeated calls with the same idempotency scope but different payload must return `blocked` or `error`.

If the first transaction rolled back, the same idempotency key may be retried.

If the first transaction committed but the API lost the response, the same idempotency key must resolve the committed envelope and fragment references rather than duplicating semantic writes.

---

# Error / Retry Policy

SQL Response Engine retry authority is SQL-side.

The API may follow `next_thinking_action`, but it must not invent semantic retry judgment.

Retryable error kinds:

```text
serialization_failure
lock_timeout
lease_conflict
transient_resource_limit
bounded_budget_exhausted
```

Non-retryable or policy-terminal kinds:

```text
invalid_route_kind
invalid_section_kind
invalid_operation_key
invalid_job_kind
idempotency_conflict
operation_blocked
policy_denied
reference_not_found
invariant_violation
prohibited_pattern
```

`bounded_budget_exhausted` may return either:

```text
output_state = no_output
next_thinking_action = retry
```

or:

```text
output_state = deferred
next_thinking_action = wait_scheduler
```

depending on the route and committed evidence.

Operation-gate denial should return a committed `blocked` result, not a hidden mutation failure.

A failed SQL transaction returns no usable envelope.

---

# Internal Dispatch Rule

Internal dispatch may branch by:

```text
route_kind
section_kind
job_kind
operation_key
split policy
bounded execution budget
watermark
```

Internal dispatch must remain inside the SQL Response Engine boundary.

It must not branch by unordered JSON object keys, status enum authority, user-visible text labels, or legacy adoption patterns.

---

# Prohibited Behavior

```text
No SQL-side user-visible output emission.
No external HTTP/API side effects from SQL Response Engine functions.
No API-side tokenization or split policy decision.
No semantic mutation without operation gate authority.
No scheduler job promotion authority.
No tmp_context or decoded projection as semantic authority.
No UUID semantic reference arrays.
No status enum lifecycle authority.
No adopt.* operation keys.
No decoder_trace / loop_guard / constraint_rule as canonical state.
```
