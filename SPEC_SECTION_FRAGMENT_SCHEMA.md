# Specification Draft: Section Fragment Schema

## Authority

This draft defines the SQL Response Engine section fragment schema.

It wires the SQL-produced fragment shape from `SPEC_API_SECTION_LOOP.md` to SQL runtime persistence, temporary context merge, reinjection, runtime envelope references, and evidence references.

It does not define semantic authority, lifecycle authority, reply-time verification semantics, promotion policy, or output emission behavior.

When this file conflicts with a routed runtime or core `SPEC_*` file, the routed `SPEC_*` file wins.

---

# Role

A section fragment is a committed SQL-produced response fragment for API-side ordering, temporary decoded context storage, merge, reinjection, audit, retry, and envelope reference.

It is not raw source context.

It is not semantic authority.

It is not user-visible output by itself.

---

# Canonical Fragment Fields

Canonical fields:

```text
section_fragment_id
section_run_id
response_engine_run_id
request_id
section_kind
res_key
sequence_tag
split_parent_token_index
split_position
merge_order
res_context
source_refs
evidence_refs
state
created_at
committed_at
expires_at
```

Minimum API-facing shape remains compatible with `SPEC_API_SECTION_LOOP.md`:

```text
section_kind
section_run_id
res_key
sequence_tag
split_parent_token_index
split_position
res_context
source_refs
evidence_refs
state
```

---

# Identity Rule

`section_fragment_id` identifies the internal fragment row.

`section_run_id` identifies one section execution.

`response_engine_run_id` identifies the SQL Response Engine step that produced the fragment.

`request_id` links the fragment to the runtime request.

Recommended identity shape:

```text
section_fragment_id = uuid
section_run_id = time-sortable uuid
response_engine_run_id = time-sortable uuid
request_id = uuid
```

UUIDs identify rows, runs, requests, or events.

UUIDs must not be used as semantic reference arrays.

Semantic references inside `source_refs`, `evidence_refs`, and `res_context` must use canonical index or bigint[] path references when referring to semantic structures.

---

# Section Kind Rule

Canonical section kinds:

```text
split
think
generate
audit
```

The default section order is:

```text
split -> think -> generate -> audit
```

Section order may be represented by a derived ordering value during API merge.

The section order does not create semantic authority.

---

# res_key Rule

`res_key` is the stable lookup key for a fragment within a section run.

Recommended shape:

```text
res_key = deterministic hash of:
  request_id
  section_run_id
  section_kind
  sequence_tag
  split_parent_token_index
  split_position
  canonical source_refs hash
  canonical res_context hash
```

`res_key` may be represented as text.

A hash is a deduplication and lookup key.

A hash is not a semantic reference key.

---

# sequence_tag Rule

`sequence_tag` is a stable text tag for section-local ordering and fragment role.

Recommended tag families:

```text
split.input
split.child
think.purpose_candidate
think.exit_candidate
generate.reduced_candidate
audit.result
reply.partial
reply.final
error.blocked
error.failed
```

`sequence_tag` is enum-like text.

It is not lifecycle authority.

It is not semantic authority.

---

# Split Ordering Rule

Split fragments must preserve original order using explicit ordering metadata.

Canonical split fields:

```text
split_parent_token_index
split_position
```

`split_parent_token_index` references the parent token index when a token or long-span body is split.

`split_position` stores the child position under the parent split.

SQL split output must preserve order by explicit position fields or ordered index arrays.

API reconstruction must not infer order from unordered JSON object keys.

---

# Merge Ordering Rule

Recommended reconstruction order:

```text
section_order
merge_order
sequence_tag
split_position
res_key
```

`merge_order` may be assigned by API-side orchestration when shaping temporary context entries.

SQL-owned fields must be sufficient to avoid unordered reconstruction.

When `sequence_tag` and `split_position` both exist, `sequence_tag` identifies the fragment role and `split_position` identifies the original split order within that role.

---

# res_context Rule

`res_context` is a SQL-produced decoded context projection.

It may contain:

```text
fragment_kind
section_kind
summary
output_delta
purpose_candidates
exit_candidates
reduced_candidates
audit_result
ordered_decode_keys
source_refs
evidence_refs
metadata_json
```

`res_context` must not contain raw canonical reply context as authority.

`res_context` must not override PostgreSQL semantic authority, reply pipeline state, operation gate results, or evidence authority.

Large bodies should be referenced when stable committed references are available.

---

# source_refs Rule

`source_refs` preserve links back to committed source and reply-core structures.

Recommended shape:

```json
{
  "input_observation_indexes": [],
  "token_indexes": [],
  "vocabulary_indexes": [],
  "grammar_indexes": [],
  "grammar_relation_indexes": [],
  "target_paths": []
}
```

Semantic references must use generated indexes or bigint[] paths.

Evidence UUIDs may appear only as internal event identifiers inside evidence JSON when needed for traceability.

---

# evidence_refs Rule

`evidence_refs` preserve links back to committed logs and decision evidence.

Recommended shape:

```json
{
  "coherence_log_indexes": [],
  "diff_log_indexes": [],
  "current_log_indexes": [],
  "aggregate_refs": [],
  "scheduler_job_run_refs": [],
  "archive_indexes": []
}
```

Logs, aggregates, and archives are evidence or pressure surfaces.

They are not semantic authority.

---

# State Rule

`state` is runtime processing state only.

Recommended values:

```text
pending
committed
failed
blocked
expired
```

`state` must not become canonical lifecycle authority.

Semantic lifecycle authority remains in canonical semantic table flags such as `draft_flag` and `deleted_flag`, interpreted through routed SPEC files and operation-gated evidence.

If a display status is needed, it must be derived from runtime state, lifecycle flags, and log evidence.

---

# Persistence Rule

Section fragments should be persisted by the SQL Response Engine before being exposed to API merge.

Recommended table:

```text
response_engine.section_fragment
```

Rationale:

```text
ordered merge
retry safety
audit visibility
reinjection references
evidence_refs preservation
runtime envelope references
```

A function return may include fragment references, but the fragment body should remain recoverable from committed SQL storage or committed temporary decoded projection storage.

---

# Commit Rule

A fragment becomes API-usable only after SQL commit success.

Canonical commit condition:

```text
section fragment row committed
+ referenced evidence rows committed
+ referenced decoded projection committed when present
+ runtime envelope committed or returned after commit boundary
= API-usable fragment candidate
```

A failed transaction invalidates fragments produced by that transaction.

The API Thinking Engine may shape committed fragments into temporary context entries only after commit success.

---

# Temporary Context Mapping

API temporary context entries may copy or reference fragment fields:

```text
tmp_context_key
section_kind
section_run_id
res_key
sequence_tag
merge_order
state
res_context
source_refs
evidence_refs
parent_tmp_context_key
created_at
```

The API may add `merge_order`, `parent_tmp_context_key`, and orchestration state.

The API must not rewrite `res_context` as semantic authority.

---

# Runtime Envelope Mapping

Runtime envelopes may reference fragments through:

```text
response_engine_run_id
output_text_ref
suggested_context_json
evidence_refs
coherence_log_indexes
diff_log_indexes
aggregate_refs
scheduler_job_run_refs
archive_indexes
```

The envelope is a small committed result summary.

It is not semantic authority.

---

# Prohibited Patterns

```text
Do not reconstruct context from unordered JSON object keys.
Do not use uuid[] as semantic reference arrays.
Do not treat res_key hashes as semantic reference keys.
Do not treat fragment state as lifecycle authority.
Do not treat res_context as semantic authority.
Do not emit section fragments directly as user-visible output without envelope and commit success.
Do not create decoder_trace, loop_guard, status enum authority, adoption_audit, or adopt.* operation keys through fragment schema.
```
