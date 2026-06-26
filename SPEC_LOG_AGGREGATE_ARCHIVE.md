# Specification: Log / Aggregate / Archive

## Authority

This file is the canonical authority for logs, aggregate current state, aggregate history, archive registry, and hot retention policy.

It canonizes the log and aggregate parts previously embedded in `SPEC_CANONICAL_CORE.md`.

---

# Role Summary

```text
logs.coherence     = append-only observation evidence
logs.diff          = mutation / decision evidence
logs.current       = aggregate snapshot history / time-series evidence
aggregate.current  = hot aggregate read model / pressure surface
archive.registry   = archive batch registry and restore manifest
```

`aggregate.current` is the current pressure surface.

`logs.current` is pressure history.

Do not make `logs.current` do both jobs.

None of these tables is semantic authority.

---

# Semantic Log Targets

```text
target_table
target_index
target_path
```

---

# logs.coherence

`logs.coherence` is append-only observation evidence.

It records search, verification, output-delta, residual, contradiction, mirror-output, decoder-use, and fallback evidence.

Canonical fields:

```text
coherence_uuid
coherence_index
observation_index
source_path
event_kind
target_table
target_index
target_path
input_grammar_path
candidate_path
near_neighbor_score
structural_verification_result
input_relation_diff_result
output_delta_kind
residual_count
mirror_output_flag
contradiction_flag
decoder_usage_flag
evidence_json
created_at
```

Allowed `source_path` values:

```text
core_reply_path
decoherence_bank_fallback
scheduled_phase
sleep_consolidation
manual_operation
operator_action
remote_event_intake
```

No public UI surface is defined here.

`manual_operation` and `operator_action` are processing classifications only.

---

# logs.diff

`logs.diff` is mutation / decision evidence.

It records operation-gated decisions and lifecycle flag transitions.

Canonical fields:

```text
diff_uuid
diff_index
operation_key
operation_status
target_table
target_index
target_path
previous_draft_flag
next_draft_flag
previous_deleted_flag
next_deleted_flag
policy_result
policy_reason
evidence_json
created_at
```

Allowed `operation_status` values:

```text
allowed
blocked
applied
rejected
quarantined
```

Promotion and deletion audit views are derived from `logs.diff`.

Promotion audit operation keys:

```text
promote.coherence_hit
promote.decoherence_hit
promote.phase_relation
```

Deletion audit operation keys:

```text
delete.semantic_structure
delete.decoherence_entry
```

---

# aggregate.current

`aggregate.current` stores the latest aggregate values for each semantic target and aggregate window.

It is the hot aggregate pressure surface used by Sleep, Phase Attention, operator actions, reply support, and scheduled maintenance.

It may be updated in place.

It must be reproducible from `logs.coherence`, `logs.diff`, and canonical semantic tables.

The unique current-surface constraint belongs to `aggregate.current`, not `logs.current`.

Canonical target identity rule:

```text
single semantic target  -> target_identity_kind = 'index', target_index is set
compound semantic path  -> target_identity_kind = 'path',  target_path is set
```

`aggregate.current` must distinguish single-row targets from compound semantic paths.

Implementations may use a stable `target_identity_key` projection, such as an encoded index or stable target-path hash, to enforce uniqueness without treating hashes as semantic authority.

Canonical identity fields:

```text
current_uuid
target_table
target_identity_kind
target_identity_key
target_index
target_path
aggregate_window
refreshed_at
unique(target_table, target_identity_kind, target_identity_key, aggregate_window)
```

Canonical aggregate fields:

```text
observed_count
hit_count
near_hit_count
partial_hit_count
residual_count
contradiction_count
mirror_output_count
decoder_success_count
scope_transition_count
relation_frequency
parent_structure_usage_count
missing_slot_count
replaced_slot_count
slot_missing_or_replaced_rate
moth_eaten_score
draft_antipattern_score
phase_score
pressure
```

---

# logs.current

`logs.current` stores aggregate snapshot evidence over time.

It is the time-series history of aggregate values.

It must not be treated as the single current row for a target/window.

It has no canonical uniqueness constraint on `(target_table, target_identity_kind, target_identity_key, aggregate_window)` because multiple snapshots may exist for the same target/window.

Canonical identity fields:

```text
current_log_uuid
current_log_index
target_table
target_identity_kind
target_identity_key
target_index
target_path
aggregate_window
aggregate_refreshed_at
logged_at
```

`logs.current` carries the same aggregate value fields as `aggregate.current`.

Hot-path current reads should use `aggregate.current`.

---

# Current Refresh Rule

Canonical refresh flow:

```text
logs.coherence
+ logs.diff
+ canonical semantic tables
-> compute aggregate values
-> upsert aggregate.current
-> optionally append snapshot to logs.current
```

The snapshot cadence may be lower than the aggregate refresh cadence.

---

# Archive Registry

`archive.registry` records archive batches for logs and aggregate history.

It is a registry and manifest.

It is not semantic authority.

Canonical fields:

```text
archive_uuid
archive_index
source_table
archive_kind
min_row_index
max_row_index
time_from
time_to
row_count
aggregate_window
storage_kind
storage_uri
archive_table
content_hash
manifest_json
restore_policy
deleted_after_archive
created_at
created_by
```

Recommended `archive_kind` values:

```text
hot_window_rolloff
monthly_rollup
snapshot_archive
audit_preservation
cold_storage_export
```

---

# Log Retention / Rollup Policy

`logs.coherence`, `logs.diff`, and `logs.current` may use bounded hot retention windows.

Recommended initial hot windows:

```text
logs.coherence = 500,000 to 2,000,000 recent evidence rows
logs.diff      = 100,000 to 500,000 recent rows
logs.current   = 100,000 to 500,000 recent aggregate snapshots
```

Archive rolloff must process bounded batches.

Recommended initial archive job limits:

```text
archive_batch_size       = 10,000 to 50,000 rows
max_archive_rows_per_run = 100,000 rows
```

Older rows may be archived or rolled up when the current state is represented elsewhere, required audit/status summaries have been snapshotted, unresolved investigation rows are excluded, and `archive.registry` records the archived range.

Unresolved investigation rows, contradiction evidence, quarantine-related rows, and manual/operator review rows must remain in hot storage until resolved, or be explicitly archived with an `audit_preservation` or equivalent restore policy.

Hot-path intelligence must not depend on unbounded log growth.

---

# Migration Rule

If an implementation already has a unique `logs.current` table with one row per `(target_table, target_index, aggregate_window)`, that table represents `aggregate.current` under the corrected interpretation.

If an implementation has append-only or periodically retained aggregate snapshots, those rows represent `logs.current`.

If an implementation already has aggregate rows keyed only by `(target_table, target_index, aggregate_window)`, single-target rows remain valid but compound target-path rows must be migrated to the canonical target identity rule before being used as `aggregate.current`.
