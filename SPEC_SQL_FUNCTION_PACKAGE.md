# Specification: SQL Function Package Wiring

## Authority

このファイルは SQL Response Engine function package の配線表です。

新規 runtime behavior、semantic authority、promotion policy、DDL、具体 signature、transaction / idempotency の最終設計を定義しません。

API orchestration、API section loop、SQL Response Engine boundary、tmp_context、runtime envelope、operation gate、scheduler、reference model など、既存SPECへの routing を固定します。

このファイルの行は次の形式で読みます。

```text
- file名, セクション名, 検索key
```

When this file conflicts with a routed `SPEC_*` file, the routed `SPEC_*` file wins.

---

## Intent

README の bundle は repository-level entry point です。

このファイルは README の重複ではなく、SQL Response Engine の function family ごとに「どの既存SPECを読めばよいか」を固定する local package wiring です。

実装Agentが `response_engine.*` の関数単位で作業するとき、README の広い bundle だけでは、reply section、ingest、split、decode、envelope、operation、scheduler の参照先が同じ粒度に見えます。

このファイルの意図は、その迷いを function package 単位で潰すことです。

---

## Use when

- SQL Response Engine の public function family を実装・監査する。
- API orchestration から呼ばれる SQL function family の責務境界を確認する。
- 実装プロンプトで、関数ごとの参照SPECを短く渡す。

---

## Do not use for

- repository 全体の入口を探す。README を読む。
- runtime envelope の field 定義そのものを確認する。`SPEC_RUNTIME_RESULT_ENVELOPE.md` を読む。
- tmp_context の storage / merge rule そのものを確認する。`SPEC_TMP_CONTEXT_JSON_BOUNDARY.md` を読む。
- operation policy を決める。`SPEC_OPERATION_GATE.md` を読む。
- scheduler job semantics を決める。`SPEC_DB_SCHEDULED_JOB_BOUNDARY.md` と `SPEC_CRON_PIPELINE.md` を読む。

---

## Non-duplication rule

新しい package wiring file は、README の bundle より細かい作業単位を固定できる場合だけ作る。

既存SPEC 1本で十分に閉じている topic は、新しい package wiring file に分離しない。

---

# response_engine.run パッケージ

- `SPEC_SQL_RESPONSE_ENGINE_BOUNDARY.md`, `# Function Package Boundary`, `sql_function_package_boundary`
- `SPEC_API_THINKING_ENGINE_BOUNDARY.md`, `# Ownership`, `api_thinking_ownership`
- `SPEC_RUNTIME_RESULT_ENVELOPE.md`, `# Branching Rule`, `envelope_branching_rule`
- `SPEC_PROHIBITED_CANONICAL_PATTERNS.md`, `# Short Form`, `prohibited_short_form`

---

# response_engine.reply_step パッケージ

- `SPEC_API_SECTION_LOOP.md`, `# Generic Section Loop`, `api_generic_section_loop`
- `SPEC_API_SECTION_LOOP.md`, `# Section Kinds`, `api_section_kinds`
- `SPEC_REPLY_PIPELINE.md`, `Canonical flow`, `reply_canonical_flow`
- `SPEC_SEARCH_AND_VERIFICATION.md`, `## Verification Boundary`, `verification_boundary`
- `SPEC_SQL_RESPONSE_ENGINE_BOUNDARY.md`, `# Ownership`, `sql_response_engine_ownership`

---

# response_engine.ingest_input パッケージ

- `SPEC_SEMANTIC_TABLES.md`, `## input_observation`, `input_observation`
- `SPEC_SQL_RESPONSE_ENGINE_BOUNDARY.md`, `# Ownership`, `sql_response_engine_ownership`
- `SPEC_REMOTE_TRUST.md`, `Acceptance rule`, `remote_event_acceptance_rule`
- `NOTE_SQL_TOKENIZATION_IMPLEMENTATION.md`, `# input_observation`, `input_observation_ddl_projection`

---

# response_engine.tokenize_and_split パッケージ

- `SPEC_SQL_RESPONSE_ENGINE_BOUNDARY.md`, `# Tokenization and Split Ownership`, `sql_tokenization_split_ownership`
- `SPEC_API_SECTION_LOOP.md`, `# SQL Response Fragment Shape`, `sql_response_fragment_shape`
- `SPEC_REFERENCE_MODEL.md`, `# Canonical Reference Chain`, `canonical_reference_chain`
- `NOTE_SQL_TOKENIZATION_IMPLEMENTATION.md`, `# token`, `token_ddl_projection`

---

# response_engine.decode_context パッケージ

- `SPEC_TMP_CONTEXT_JSON_BOUNDARY.md`, `# Decode Projection Rule`, `tmp_context_decode_projection`
- `SPEC_API_SECTION_LOOP.md`, `# API Temporary Context Entry Shape`, `api_tmp_context_entry_shape`
- `SPEC_RUNTIME_RESULT_ENVELOPE.md`, `# Output Text Rule`, `envelope_output_text_ref`
- `SPEC_REPLY_PIPELINE.md`, `## Decoder / Collapse`, `decoder_collapse`

---

# response_engine.build_envelope パッケージ

- `SPEC_RUNTIME_RESULT_ENVELOPE.md`, `# Commit Rule`, `envelope_commit_rule`
- `SPEC_RUNTIME_RESULT_ENVELOPE.md`, `# Canonical Fields`, `envelope_canonical_fields`
- `SPEC_API_THINKING_ENGINE_BOUNDARY.md`, `# Output Branching`, `api_output_branching`

---

# response_engine.operation_step パッケージ

- `SPEC_OPERATION_GATE.md`, `core_can_execute`, `core_can_execute`
- `SPEC_OPERATION_GATE.md`, `Canonical operation keys`, `operation_keys`
- `SPEC_LOG_AGGREGATE_ARCHIVE.md`, `# logs.diff`, `logs_diff`
- `SPEC_PROHIBITED_CANONICAL_PATTERNS.md`, `# Adoption Rule`, `prohibited_adopt_operations`

---

# response_engine.scheduler_step パッケージ

- `SPEC_DB_SCHEDULED_JOB_BOUNDARY.md`, `# Canonical Job Kinds`, `db_job_kinds`
- `SPEC_DB_SCHEDULED_JOB_BOUNDARY.md`, `# Execution Bounds`, `db_job_execution_bounds`
- `SPEC_CRON_PIPELINE.md`, `Canonical scheduler job kinds`, `scheduler_job_kinds`
- `SPEC_CORE_STATE_AND_SCHEDULER.md`, `## scheduler_job_run`, `scheduler_job_run`

---

# prohibited patterns パッケージ

- `SPEC_SQL_RESPONSE_ENGINE_BOUNDARY.md`, `# Prohibited Behavior`, `sql_prohibited_behavior`
- `SPEC_API_SECTION_LOOP.md`, `# Prohibited Patterns`, `api_section_prohibited_patterns`
- `SPEC_REFERENCE_MODEL.md`, `# Prohibited Patterns`, `reference_prohibited_patterns`
- `SPEC_PROHIBITED_CANONICAL_PATTERNS.md`, `# Short Form`, `prohibited_short_form`
