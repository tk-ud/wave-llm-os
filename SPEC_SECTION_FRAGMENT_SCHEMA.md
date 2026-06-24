# Specification: Section Fragment Runtime Wiring

## Authority

このファイルは section fragment を runtime 時系列に沿って既存資料へ接続する配線表です。

コアロジック、DDL、具体 field 型、id 生成方式、transaction / idempotency の最終判断は書きません。

ここに書くのは runtime 上の接続点と参照先だけです。

```text
- file名, セクション名, 検索key
```

---

# SQL section output -> fragment shape パッケージ

- `SPEC_API_SECTION_LOOP.md`, `# SQL Response Fragment Shape`, `sql_response_fragment_shape`
- `SPEC_SQL_RESPONSE_ENGINE_BOUNDARY.md`, `# Ownership`, `sql_response_engine_ownership`
- `SPEC_SQL_FUNCTION_PACKAGE.md`, `API section loop -> response_engine.reply_step パッケージ`, `response_engine_reply_step_package`

---

# fragment ordering -> API merge パッケージ

- `SPEC_API_SECTION_LOOP.md`, `# Ordering Rule`, `api_ordering_rule`
- `SPEC_SQL_RESPONSE_ENGINE_BOUNDARY.md`, `# Tokenization and Split Ownership`, `sql_tokenization_split_ownership`
- `NOTE_SQL_TOKENIZATION_IMPLEMENTATION.md`, `# token`, `token_ddl_projection`

---

# fragment res_context -> tmp_context パッケージ

- `SPEC_API_SECTION_LOOP.md`, `# API Temporary Context Entry Shape`, `api_tmp_context_entry_shape`
- `SPEC_TMP_CONTEXT_JSON_BOUNDARY.md`, `# Decode Projection Rule`, `tmp_context_decode_projection`
- `SPEC_TMP_CONTEXT_JSON_BOUNDARY.md`, `# SQL Access Rule`, `tmp_context_sql_access_rule`
- `SPEC_SQL_FUNCTION_PACKAGE.md`, `decode projection -> response_engine.decode_context パッケージ`, `response_engine_decode_context_package`

---

# fragment refs -> semantic reference / evidence パッケージ

- `SPEC_REFERENCE_MODEL.md`, `# Core Rule`, `reference_core_rule`
- `SPEC_REFERENCE_MODEL.md`, `# Reference Use by Table Family`, `reference_use_by_table_family`
- `SPEC_LOG_AGGREGATE_ARCHIVE.md`, `# Semantic Log Targets`, `semantic_log_targets`
- `SPEC_LOG_AGGREGATE_ARCHIVE.md`, `# logs.coherence`, `logs_coherence`
- `SPEC_LOG_AGGREGATE_ARCHIVE.md`, `# logs.diff`, `logs_diff`

---

# tmp_context -> reinjection パッケージ

- `SPEC_API_SECTION_LOOP.md`, `# Reinjection Rule`, `api_reinjection_rule`
- `SPEC_TMP_CONTEXT_JSON_BOUNDARY.md`, `# API Merge and Reinjection Rule`, `tmp_context_merge_reinjection`
- `SPEC_TMP_CONTEXT_JSON_BOUNDARY.md`, `# API Memory Rule`, `tmp_context_api_memory_rule`
- `SPEC_API_THINKING_ENGINE_BOUNDARY.md`, `# Context Reinjection Boundary`, `api_context_reinjection_boundary`

---

# committed fragment/envelope -> API output branching パッケージ

- `SPEC_API_SECTION_LOOP.md`, `# Output Rule`, `api_output_rule`
- `SPEC_RUNTIME_RESULT_ENVELOPE.md`, `# Commit Rule`, `envelope_commit_rule`
- `SPEC_RUNTIME_RESULT_ENVELOPE.md`, `# Canonical Fields`, `envelope_canonical_fields`
- `SPEC_API_THINKING_ENGINE_BOUNDARY.md`, `# Output Branching`, `api_output_branching`

---

# fragment state/status -> prohibited authority パッケージ

- `SPEC_API_SECTION_LOOP.md`, `# SQL Response Fragment Shape`, `sql_response_fragment_shape`
- `SPEC_TMP_CONTEXT_JSON_BOUNDARY.md`, `# Identity Fields`, `tmp_context_identity_fields`
- `SPEC_PROHIBITED_CANONICAL_PATTERNS.md`, `# Status Rule`, `prohibited_status_authority`
- `SPEC_REFERENCE_MODEL.md`, `# Prohibited Patterns`, `reference_prohibited_patterns`
