# Specification Draft: DB Scheduled Job Boundary

## Authority

This draft defines the boundary for database scheduled jobs callable through SQL functions.

DB Scheduled Jobs are not the API Thinking Engine.

They are bounded database jobs executed through the SQL Response Engine boundary.

---

# Canonical Job Kinds

Canonical job kinds remain defined by `SPEC_CRON_PIPELINE.md`:

```text
current_refresh
phase_candidate_generation
sleep_consolidation
archive_rolloff
```

---

# Job Ownership

DB Scheduled Jobs may:

```text
refresh aggregate.current
append logs.current snapshots
generate phase_relation_candidate rows
route unstable structures to decoherence_bank
write archive.registry rows
process bounded hot-window rolloff
```

They must not replace operation-gated promotion.

---

# Execution Bounds

Each job run must be bounded.

Recommended controls:

```text
max_rows_per_run
max_iterations_per_run
max_runtime_hint_ms
watermark
lease_key
idempotency_key
```

---

# Queue Driver Boundary

An external scheduler or worker may request a job run.

The worker should remain a thin queue driver.

Job selection, row claiming, verification, evidence writes, and result envelope generation belong inside SQL functions.

---

# Promotion Boundary

Phase generation may create draft candidates.

Promotion remains an operation-gated action.

Sleep may route unstable structures to fallback storage.

Sleep never hard-deletes semantic structures.

Archive rolloff must not remove current semantic authority.
