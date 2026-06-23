# Specification: Core State and Scheduler

This file is the canonical authority for core state, scheduler tables, notification queue, mastication jobs, and remote event intake tables not owned by Remote Trust.

Canonical table family:

```text
core_state
core_operation_policy
core_notify_queue
scheduler_job
scheduler_job_run
mastication_job
remote_event_inbox
remote_event_quarantine
```

## core_state

`core_state` stores runtime-wide switches and state used by core policies.

Examples include:

```text
distributed_sync.enabled
freeze
policy configuration references
```

`core_state` is not semantic authority.

It gates runtime behavior.

## core_operation_policy

`core_operation_policy` stores tunable policy values used by `core_can_execute(operation_key)`.

Policy values may tune thresholds but must not erase required evidence components.

Operation authority is defined by `SPEC_OPERATION_GATE.md`.

## core_notify_queue

`core_notify_queue` stores notification requests such as unresolved questions or operator-visible events.

It is not a semantic table.

A notification does not promote, delete, or mutate semantic structure by itself.

## scheduler_job

`scheduler_job` stores scheduled job definitions.

Canonical job kinds are defined by `SPEC_CRON_PIPELINE.md`.

Scheduler job kinds are a separate namespace from `operation_key`.

## scheduler_job_run

`scheduler_job_run` stores execution evidence for scheduler jobs.

A scheduler run may generate candidates, refresh aggregates, consolidate evidence, or archive old rows.

A scheduler run does not replace operation-gated promotion.

## mastication_job

`mastication_job` stores corpus-time processing jobs.

Mastication output must obey input subtraction and mirror-output rules defined by `SPEC_SEARCH_AND_VERIFICATION.md`.

## remote_event_inbox

`remote_event_inbox` stores received remote events before acceptance.

Accepted remote events may become `input_observation(source_kind='remote_event')` only after trust and quarantine checks.

Remote trust authority is defined by `SPEC_REMOTE_TRUST.md`.

## remote_event_quarantine

`remote_event_quarantine` stores blocked or unresolved remote events.

Quarantine is not deletion.

Remote quarantine mutation is represented by operation key:

```text
quarantine.remote_event
```
