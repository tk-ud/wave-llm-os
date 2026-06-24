# Specification: Background Driver API Wiring

## Authority

このファイルは Background Driver API を既存の scheduled / operation / support 正本へ接続する専門配線表です。

Reply API / API Thinking Engine の orchestration、Search / Verification、decode / collapse、user-visible output は扱いません。

Background Driver API は kind を受け、対応する正本SPECへ routing し、SQL function / scheduled job boundary に渡すだけです。

Background Driver API は SQL function call の起動係であり、API側の作業記憶や判断根拠を DB table に置きません。

コアロジック、DDL、具体 signature、field 型、score 式、promotion policy、transaction / idempotency の最終判断は書きません。

```text
- file名, セクション名, 検索key
```

---

# background kind dispatch -> SQL scheduled job boundary パッケージ

- `SPEC_DB_SCHEDULED_JOB_BOUNDARY.md`, `# Canonical Job Kinds`, `db_job_kinds`
- `SPEC_DB_SCHEDULED_JOB_BOUNDARY.md`, `# Queue Driver Boundary`, `db_queue_driver_boundary`
- `SPEC_SQL_RESPONSE_ENGINE_BOUNDARY.md`, `# Function Package Boundary`, `sql_function_package_boundary`
- `SPEC_SQL_FUNCTION_PACKAGE.md`, `scheduled path -> response_engine.scheduler_step パッケージ`, `response_engine_scheduler_step_package`

---

# current_refresh パッケージ

- `SPEC_CRON_PIPELINE.md`, `## Canonical flow`, `cron_canonical_flow`
- `SPEC_DB_SCHEDULED_JOB_BOUNDARY.md`, `# Canonical Job Kinds`, `db_job_kinds`
- `SPEC_LOG_AGGREGATE_ARCHIVE.md`, `# aggregate.current`, `aggregate_current`
- `SPEC_LOG_AGGREGATE_ARCHIVE.md`, `# Current Refresh Rule`, `current_refresh_rule`

---

# phase_candidate_generation パッケージ

- `SPEC_CRON_PIPELINE.md`, `## Phase`, `phase_candidate_generation`
- `SPEC_SCORING_AND_THRESHOLDS.md`, `## Phase score`, `phase_score`
- `SPEC_SEMANTIC_TABLES.md`, `## phase_relation_candidate`, `phase_relation_candidate_table`
- `SPEC_OPERATION_GATE.md`, `## promote.phase_relation`, `promote_phase_relation`
- `NOTE_PHASE_RELATION_CANDIDATE.md`, `# Implementation Boundary`, `phase_relation_candidate_implementation_boundary`

---

# sleep_consolidation パッケージ

- `SPEC_CRON_PIPELINE.md`, `## Sleep`, `sleep_consolidation`
- `SPEC_SCORING_AND_THRESHOLDS.md`, `## Moth-eaten score`, `moth_eaten_score`
- `SPEC_SEMANTIC_TABLES.md`, `## decoherence_bank`, `decoherence_bank_table`
- `SPEC_OPERATION_GATE.md`, `## sleep.decohere_structure`, `sleep_decohere_structure`

---

# archive_rolloff パッケージ

- `SPEC_CRON_PIPELINE.md`, `## Archive`, `archive_rolloff`
- `SPEC_DB_SCHEDULED_JOB_BOUNDARY.md`, `# Promotion Boundary`, `db_job_promotion_boundary`
- `SPEC_LOG_AGGREGATE_ARCHIVE.md`, `# Archive Registry`, `archive_registry`

---

# distributed_sync_intake パッケージ

- `SPEC_REMOTE_TRUST.md`, `Acceptance rule`, `remote_event_acceptance_rule`
- `SPEC_CORE_STATE_AND_SCHEDULER.md`, `## remote_event_inbox`, `remote_event_inbox`
- `SPEC_CORE_STATE_AND_SCHEDULER.md`, `## remote_event_quarantine`, `remote_event_quarantine`
- `SPEC_SEMANTIC_TABLES.md`, `## input_observation`, `input_observation`
- `SPEC_OPERATION_GATE.md`, `Canonical operation keys`, `operation_keys`

---

# no reply output / no API thinking パッケージ

- `SPEC_RUNTIME_ENGINE_BOUNDARY.md`, `# Boundary Invariants`, `runtime_boundary_invariants`
- `SPEC_DB_SCHEDULED_JOB_BOUNDARY.md`, `# Queue Driver Boundary`, `db_queue_driver_boundary`
- `SPEC_RUNTIME_RESULT_ENVELOPE.md`, `# output_state Values`, `envelope_output_state`
- `SPEC_API_THINKING_ENGINE_BOUNDARY.md`, `# Output Branching`, `api_output_branching`
