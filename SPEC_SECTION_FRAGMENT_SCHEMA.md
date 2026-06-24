# Specification: Section Fragment Schema Wiring

## Authority

このファイルは section fragment schema の配線表です。

具体DDL、最終field型、id生成方式、transaction / idempotency の最終判断を定義しません。

`SPEC_API_SECTION_LOOP.md` の SQL Response Fragment Shape を起点に、各 field / concern がどのSPECに由来するかを固定します。

このファイルの行は次の形式で読みます。

```text
- file名, セクション名, 検索key
```

When this file conflicts with a routed `SPEC_*` file, the routed `SPEC_*` file wins.

---

# section_kind パッケージ

- `SPEC_API_SECTION_LOOP.md`, `# Section Kinds`, `api_section_kinds`
- `SPEC_API_SECTION_LOOP.md`, `# Section Instructions`, `api_section_instructions`

---

# section_run_id パッケージ

- `SPEC_API_SECTION_LOOP.md`, `# SQL Response Fragment Shape`, `sql_response_fragment_shape`
- `SPEC_SQL_RESPONSE_ENGINE_BOUNDARY.md`, `# Transaction Boundary`, `sql_transaction_boundary`
- `SPEC_RUNTIME_RESULT_ENVELOPE.md`, `# Canonical Fields`, `envelope_canonical_fields`

---

# res_key パッケージ

- `SPEC_API_SECTION_LOOP.md`, `# SQL Response Fragment Shape`, `sql_response_fragment_shape`
- `SPEC_TMP_CONTEXT_JSON_BOUNDARY.md`, `# Identity Fields`, `tmp_context_identity_fields`
- `SPEC_REFERENCE_MODEL.md`, `# Hashes`, `structural_hashes`

---

# sequence_tag パッケージ

- `SPEC_API_SECTION_LOOP.md`, `# Ordering Rule`, `api_ordering_rule`
- `SPEC_API_SECTION_LOOP.md`, `# SQL Response Fragment Shape`, `sql_response_fragment_shape`

---

# split_parent_token_index / split_position パッケージ

- `SPEC_SQL_RESPONSE_ENGINE_BOUNDARY.md`, `# Tokenization and Split Ownership`, `sql_tokenization_split_ownership`
- `NOTE_SQL_TOKENIZATION_IMPLEMENTATION.md`, `# token`, `token_ddl_projection`
- `SPEC_API_SECTION_LOOP.md`, `# Ordering Rule`, `api_ordering_rule`

---

# res_context パッケージ

- `SPEC_API_SECTION_LOOP.md`, `# SQL Response Fragment Shape`, `sql_response_fragment_shape`
- `SPEC_TMP_CONTEXT_JSON_BOUNDARY.md`, `# Decode Projection Rule`, `tmp_context_decode_projection`
- `SPEC_TMP_CONTEXT_JSON_BOUNDARY.md`, `# SQL Access Rule`, `tmp_context_sql_access_rule`
- `SPEC_PROHIBITED_CANONICAL_PATTERNS.md`, `# Decoder / Constraint Rule`, `prohibited_decoder_trace_loop_guard`

---

# source_refs パッケージ

- `SPEC_REFERENCE_MODEL.md`, `# Core Rule`, `reference_core_rule`
- `SPEC_REFERENCE_MODEL.md`, `# Reference Use by Table Family`, `reference_use_by_table_family`
- `SPEC_LOG_AGGREGATE_ARCHIVE.md`, `# Semantic Log Targets`, `semantic_log_targets`
- `SPEC_API_SECTION_LOOP.md`, `# Reinjection Rule`, `api_reinjection_rule`

---

# evidence_refs パッケージ

- `SPEC_LOG_AGGREGATE_ARCHIVE.md`, `# Semantic Log Targets`, `semantic_log_targets`
- `SPEC_LOG_AGGREGATE_ARCHIVE.md`, `# logs.coherence`, `logs_coherence`
- `SPEC_LOG_AGGREGATE_ARCHIVE.md`, `# logs.diff`, `logs_diff`
- `SPEC_RUNTIME_RESULT_ENVELOPE.md`, `# Canonical Fields`, `envelope_canonical_fields`

---

# state パッケージ

- `SPEC_API_SECTION_LOOP.md`, `# SQL Response Fragment Shape`, `sql_response_fragment_shape`
- `SPEC_PROHIBITED_CANONICAL_PATTERNS.md`, `# Status Rule`, `prohibited_status_authority`
- `SPEC_TMP_CONTEXT_JSON_BOUNDARY.md`, `# Identity Fields`, `tmp_context_identity_fields`

---

# ordering パッケージ

- `SPEC_API_SECTION_LOOP.md`, `# Ordering Rule`, `api_ordering_rule`
- `SPEC_SQL_RESPONSE_ENGINE_BOUNDARY.md`, `# Tokenization and Split Ownership`, `sql_tokenization_split_ownership`

---

# tmp_context mapping パッケージ

- `SPEC_API_SECTION_LOOP.md`, `# API Temporary Context Entry Shape`, `api_tmp_context_entry_shape`
- `SPEC_TMP_CONTEXT_JSON_BOUNDARY.md`, `# API Merge and Reinjection Rule`, `tmp_context_merge_reinjection`
- `SPEC_TMP_CONTEXT_JSON_BOUNDARY.md`, `# API Memory Rule`, `tmp_context_api_memory_rule`

---

# envelope mapping パッケージ

- `SPEC_RUNTIME_RESULT_ENVELOPE.md`, `# Commit Rule`, `envelope_commit_rule`
- `SPEC_RUNTIME_RESULT_ENVELOPE.md`, `# Canonical Fields`, `envelope_canonical_fields`
- `SPEC_API_THINKING_ENGINE_BOUNDARY.md`, `# Output Branching`, `api_output_branching`

---

# prohibited patterns パッケージ

- `SPEC_PROHIBITED_CANONICAL_PATTERNS.md`, `# Short Form`, `prohibited_short_form`
- `SPEC_REFERENCE_MODEL.md`, `# Prohibited Patterns`, `reference_prohibited_patterns`
