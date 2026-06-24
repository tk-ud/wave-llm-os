# Specification: SQL Function Package Runtime Wiring

## Authority

このファイルは SQL function package を runtime 時系列に沿って既存資料へ接続する配線表です。

コアロジック、DDL、具体 signature、field 型、promotion policy、transaction / idempotency の最終判断は書きません。

ここに書くのは runtime 上の接続点と参照先だけです。

```text
- file名, セクション名, 検索key
```

---

# response engine entry family routing パッケージ

- `SPEC_RUNTIME_ENGINE_BOUNDARY.md`, `# Boundary Rule`, `runtime_boundary_rule`
- `SPEC_API_THINKING_ENGINE_BOUNDARY.md`, `# Ownership`, `api_thinking_ownership`
- `SPEC_SQL_RESPONSE_ENGINE_BOUNDARY.md`, `# Function Package Boundary`, `sql_function_package_boundary`
- `SPEC_RUNTIME_RESULT_ENVELOPE.md`, `# Branching Rule`, `envelope_branching_rule`

---

# API section loop -> response_engine.reply_step パッケージ

- `SPEC_API_SECTION_LOOP.md`, `# Section Kinds`, `api_section_kinds`
- `SPEC_API_SECTION_LOOP.md`, `# Generic Section Loop`, `api_generic_section_loop`
- `SPEC_SQL_RESPONSE_ENGINE_BOUNDARY.md`, `# Ownership`, `sql_response_engine_ownership`
- `SPEC_REPLY_PIPELINE.md`, `Canonical flow`, `reply_canonical_flow`

---

# source intake -> response_engine.ingest_input パッケージ

- `SPEC_SEMANTIC_TABLES.md`, `## input_observation`, `input_observation`
- `SPEC_REMOTE_TRUST.md`, `Acceptance rule`, `remote_event_acceptance_rule`
- `NOTE_SQL_TOKENIZATION_IMPLEMENTATION.md`, `# input_observation`, `input_observation_ddl_projection`
- `SPEC_SQL_RESPONSE_ENGINE_BOUNDARY.md`, `# Ownership`, `sql_response_engine_ownership`

---

# split request -> response_engine.tokenize_and_split パッケージ

- `SPEC_SQL_RESPONSE_ENGINE_BOUNDARY.md`, `# Tokenization and Split Ownership`, `sql_tokenization_split_ownership`
- `SPEC_API_SECTION_LOOP.md`, `# SQL Response Fragment Shape`, `sql_response_fragment_shape`
- `NOTE_SQL_TOKENIZATION_IMPLEMENTATION.md`, `# token`, `token_ddl_projection`
- `SPEC_REFERENCE_MODEL.md`, `# Canonical Reference Chain`, `canonical_reference_chain`

---

# decode projection -> response_engine.decode_context パッケージ

- `SPEC_TMP_CONTEXT_JSON_BOUNDARY.md`, `# Decode Projection Rule`, `tmp_context_decode_projection`
- `SPEC_API_SECTION_LOOP.md`, `# SQL Response Fragment Shape`, `sql_response_fragment_shape`
- `SPEC_API_SECTION_LOOP.md`, `# API Temporary Context Entry Shape`, `api_tmp_context_entry_shape`
- `SPEC_REPLY_PIPELINE.md`, `## Decoder / Collapse`, `decoder_collapse`

---

# committed result -> response_engine.build_envelope パッケージ

- `SPEC_RUNTIME_RESULT_ENVELOPE.md`, `# Commit Rule`, `envelope_commit_rule`
- `SPEC_RUNTIME_RESULT_ENVELOPE.md`, `# Canonical Fields`, `envelope_canonical_fields`
- `SPEC_RUNTIME_RESULT_ENVELOPE.md`, `# Branching Rule`, `envelope_branching_rule`
- `SPEC_API_THINKING_ENGINE_BOUNDARY.md`, `# Output Branching`, `api_output_branching`

---

# mutation path -> response_engine.operation_step パッケージ

- `SPEC_OPERATION_GATE.md`, `core_can_execute`, `core_can_execute`
- `SPEC_OPERATION_GATE.md`, `Canonical operation keys`, `operation_keys`
- `SPEC_LOG_AGGREGATE_ARCHIVE.md`, `# logs.diff`, `logs_diff`
- `SPEC_PROHIBITED_CANONICAL_PATTERNS.md`, `# Adoption Rule`, `prohibited_adopt_operations`

---

# scheduled path -> response_engine.scheduler_step パッケージ

- `SPEC_DB_SCHEDULED_JOB_BOUNDARY.md`, `# Canonical Job Kinds`, `db_job_kinds`
- `SPEC_DB_SCHEDULED_JOB_BOUNDARY.md`, `# Execution Bounds`, `db_job_execution_bounds`
- `SPEC_CRON_PIPELINE.md`, `Canonical scheduler job kinds`, `scheduler_job_kinds`
- `SPEC_CORE_STATE_AND_SCHEDULER.md`, `## scheduler_job_run`, `scheduler_job_run`
