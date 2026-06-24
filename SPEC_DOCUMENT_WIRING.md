# Specification: Document Wiring Map

## Authority

This file defines the repository-level document wiring map.

It is a routing index for finding the authoritative section for a design topic.

It does not define runtime behavior, semantic authority, lifecycle rules, promotion policy, scheduler behavior, table meaning, or SQL implementation detail.

When this file conflicts with a routed `SPEC_*` file, the routed `SPEC_*` file wins.

`NOTE_*` files remain implementation projections or explanatory background unless explicitly promoted by a routed `SPEC_*` file.

---

# Reading Rule

Each package lists the files and sections that should be read together.

Format:

```text
- file, section, search_key
```

`search_key` is a stable lookup hint for humans, agents, and implementation work.

---

# Core Authority Package

- `README.md`, `# Repository Resume`, `repository_resume`
- `SPEC_CANONICAL_CORE_RESUME.md`, `# Canonical Interpretation Rules`, `canonical_interpretation`
- `SPEC_CANONICAL_CORE_RESUME.md`, `# Core Invariants`, `core_invariants`
- `SPEC_CANONICAL_CORE_RESUME.md`, `# Document Routing`, `document_routing`
- `NOTE_SQL_IMPLEMENTATION_MAP.md`, `# NOTE to SPEC Map`, `note_to_spec_map`

---

# Runtime Behavior Package

- `README.md`, `# 波動と四元数`, `wave_quaternion_intro`
- `SPEC_RUNTIME_BEHAVIOR_MODEL.md`, `# Behavior Names`, `runtime_behavior_names`
- `SPEC_RUNTIME_BEHAVIOR_MODEL.md`, `# wave`, `wave_behavior`
- `SPEC_RUNTIME_BEHAVIOR_MODEL.md`, `# coherence`, `coherence_behavior`
- `SPEC_RUNTIME_BEHAVIOR_MODEL.md`, `# decoherence`, `decoherence_behavior`
- `SPEC_RUNTIME_BEHAVIOR_MODEL.md`, `# pressure`, `pressure_behavior`
- `SPEC_RUNTIME_BEHAVIOR_MODEL.md`, `# Phase`, `phase_behavior`
- `SPEC_RUNTIME_BEHAVIOR_MODEL.md`, `# Sleep`, `sleep_behavior`
- `SPEC_RUNTIME_BEHAVIOR_MODEL.md`, `# collapse`, `collapse_behavior`
- `SPEC_RUNTIME_BEHAVIOR_MODEL.md`, `# Projection Rule`, `runtime_projection_rule`
- `NOTE_AGENT_METAPHOR_MAPPING.md`, `# Labels`, `agent_label_mapping`
- `NOTE_QUATERNION_PHILOSOPHY.md`, `# Analogy`, `quaternion_analogy`

---

# Reference / Identity Package

- `SPEC_REFERENCE_MODEL.md`, `# Core Rule`, `reference_core_rule`
- `SPEC_REFERENCE_MODEL.md`, `# Identity Layers`, `identity_layers`
- `SPEC_REFERENCE_MODEL.md`, `# Canonical Reference Chain`, `canonical_reference_chain`
- `SPEC_REFERENCE_MODEL.md`, `# Allowed Semantic Array Columns`, `allowed_semantic_array_columns`
- `SPEC_REFERENCE_MODEL.md`, `# Hashes`, `structural_hashes`
- `SPEC_REFERENCE_MODEL.md`, `# Reference Use by Table Family`, `reference_use_by_table_family`
- `SPEC_REFERENCE_MODEL.md`, `# Decoder and Collapse Boundary`, `decoder_collapse_reference_boundary`
- `SPEC_REFERENCE_MODEL.md`, `# Prohibited Patterns`, `reference_prohibited_patterns`
- `NOTE_INDEX_ARRAY_CANONICAL.md`, `# Implementation Reminder`, `index_array_implementation`

---

# Semantic Table Package

- `SPEC_SEMANTIC_TABLES.md`, `Canonical table family`, `semantic_table_family`
- `SPEC_SEMANTIC_TABLES.md`, `## input_observation`, `input_observation`
- `SPEC_SEMANTIC_TABLES.md`, `## token`, `token_table`
- `SPEC_SEMANTIC_TABLES.md`, `## vocabulary`, `vocabulary_table`
- `SPEC_SEMANTIC_TABLES.md`, `## grammar`, `grammar_table`
- `SPEC_SEMANTIC_TABLES.md`, `## grammar_relation`, `grammar_relation_table`
- `SPEC_SEMANTIC_TABLES.md`, `## grammar_relation_link`, `grammar_relation_link_table`
- `SPEC_SEMANTIC_TABLES.md`, `## phase_relation_candidate`, `phase_relation_candidate_table`
- `SPEC_SEMANTIC_TABLES.md`, `## decoherence_bank`, `decoherence_bank_table`
- `SPEC_SEMANTIC_TABLES.md`, `## structural_vector_index`, `structural_vector_index`
- `NOTE_SQL_TOKENIZATION_IMPLEMENTATION.md`, `# Table List Projection`, `sql_table_projection`
- `NOTE_SQL_TOKENIZATION_IMPLEMENTATION.md`, `# input_observation`, `input_observation_ddl_projection`
- `NOTE_SQL_TOKENIZATION_IMPLEMENTATION.md`, `# token`, `token_ddl_projection`
- `NOTE_SQL_TOKENIZATION_IMPLEMENTATION.md`, `# vocabulary`, `vocabulary_ddl_projection`
- `NOTE_SQL_TOKENIZATION_IMPLEMENTATION.md`, `# grammar`, `grammar_ddl_projection`
- `NOTE_SQL_TOKENIZATION_IMPLEMENTATION.md`, `# grammar_relation`, `grammar_relation_ddl_projection`
- `NOTE_SQL_TOKENIZATION_IMPLEMENTATION.md`, `# phase_relation_candidate`, `phase_relation_candidate_ddl_projection`
- `NOTE_SQL_TOKENIZATION_IMPLEMENTATION.md`, `# decoherence_bank`, `decoherence_bank_ddl_projection`

---

# Reply / Search / Verification Package

- `SPEC_REPLY_PIPELINE.md`, `Canonical flow`, `reply_canonical_flow`
- `SPEC_REPLY_PIPELINE.md`, `## Reply-time promotion`, `reply_time_promotion`
- `SPEC_REPLY_PIPELINE.md`, `## Decoherence fallback`, `decoherence_fallback`
- `SPEC_REPLY_PIPELINE.md`, `## Expansion boundary`, `reply_expansion_boundary`
- `SPEC_REPLY_PIPELINE.md`, `## Draft meaning`, `draft_meaning`
- `SPEC_REPLY_PIPELINE.md`, `## Decoder / Collapse`, `decoder_collapse`
- `SPEC_SEARCH_AND_VERIFICATION.md`, `## Input Grammar / Grammar Relation Diff`, `input_relation_diff`
- `SPEC_SEARCH_AND_VERIFICATION.md`, `## Corpus Output / Input Subtraction`, `input_subtraction`
- `SPEC_SEARCH_AND_VERIFICATION.md`, `## Verification Boundary`, `verification_boundary`
- `NOTE_NEAR_NEIGHBOR_SEARCH.md`, `# Implementation Boundary`, `near_neighbor_implementation_boundary`
- `NOTE_NEAR_NEIGHBOR_SEARCH.md`, `# Implementation Sketch`, `near_neighbor_implementation_sketch`

---

# Operation Gate Package

- `SPEC_OPERATION_GATE.md`, `core_can_execute`, `core_can_execute`
- `SPEC_OPERATION_GATE.md`, `Canonical operation keys`, `operation_keys`
- `SPEC_OPERATION_GATE.md`, `## promote.coherence_hit`, `promote_coherence_hit`
- `SPEC_OPERATION_GATE.md`, `## promote.decoherence_hit`, `promote_decoherence_hit`
- `SPEC_OPERATION_GATE.md`, `## promote.phase_relation`, `promote_phase_relation`
- `SPEC_OPERATION_GATE.md`, `## sleep.decohere_structure`, `sleep_decohere_structure`
- `SPEC_OPERATION_GATE.md`, `## delete.semantic_structure`, `delete_semantic_structure`
- `SPEC_OPERATION_GATE.md`, `## delete.decoherence_entry`, `delete_decoherence_entry`
- `SPEC_PROHIBITED_CANONICAL_PATTERNS.md`, `# Adoption Rule`, `prohibited_adopt_operations`

---

# Cron / Phase / Sleep Package

- `SPEC_CRON_PIPELINE.md`, `Canonical scheduler job kinds`, `scheduler_job_kinds`
- `SPEC_CRON_PIPELINE.md`, `## Canonical flow`, `cron_canonical_flow`
- `SPEC_CRON_PIPELINE.md`, `## Phase`, `phase_candidate_generation`
- `SPEC_CRON_PIPELINE.md`, `## Sleep`, `sleep_consolidation`
- `SPEC_CRON_PIPELINE.md`, `## Archive`, `archive_rolloff`
- `SPEC_SCORING_AND_THRESHOLDS.md`, `## Phase score`, `phase_score`
- `SPEC_SCORING_AND_THRESHOLDS.md`, `## Draft anti-pattern score`, `draft_antipattern_score`
- `SPEC_SCORING_AND_THRESHOLDS.md`, `## Moth-eaten score`, `moth_eaten_score`
- `NOTE_PHASE_RELATION_CANDIDATE.md`, `# Candidate Storage Sketch`, `phase_relation_candidate_storage`
- `NOTE_PHASE_RELATION_CANDIDATE.md`, `# Query Input Sketch`, `phase_relation_candidate_query_inputs`
- `NOTE_PHASE_RELATION_CANDIDATE.md`, `# Implementation Flow Sketch`, `phase_relation_candidate_flow`

---

# Logs / Aggregate / Archive Package

- `SPEC_LOG_AGGREGATE_ARCHIVE.md`, `# Role Summary`, `log_aggregate_archive_roles`
- `SPEC_LOG_AGGREGATE_ARCHIVE.md`, `# Semantic Log Targets`, `semantic_log_targets`
- `SPEC_LOG_AGGREGATE_ARCHIVE.md`, `# logs.coherence`, `logs_coherence`
- `SPEC_LOG_AGGREGATE_ARCHIVE.md`, `# logs.diff`, `logs_diff`
- `SPEC_LOG_AGGREGATE_ARCHIVE.md`, `# aggregate.current`, `aggregate_current`
- `SPEC_LOG_AGGREGATE_ARCHIVE.md`, `# logs.current`, `logs_current`
- `SPEC_LOG_AGGREGATE_ARCHIVE.md`, `# Current Refresh Rule`, `current_refresh_rule`
- `SPEC_LOG_AGGREGATE_ARCHIVE.md`, `# Archive Registry`, `archive_registry`

---

# Runtime Engine Boundary Package

- `SPEC_RUNTIME_ENGINE_BOUNDARY.md`, `# Runtime Role Summary`, `runtime_role_summary`
- `SPEC_RUNTIME_ENGINE_BOUNDARY.md`, `# Routed Specification Map`, `runtime_routed_specification_map`
- `SPEC_RUNTIME_ENGINE_BOUNDARY.md`, `# Boundary Invariants`, `runtime_boundary_invariants`
- `SPEC_API_THINKING_ENGINE_BOUNDARY.md`, `# Ownership`, `api_thinking_ownership`
- `SPEC_API_THINKING_ENGINE_BOUNDARY.md`, `# Default Serial Step Pipeline`, `api_serial_step_pipeline`
- `SPEC_API_THINKING_ENGINE_BOUNDARY.md`, `# Reply Core Boundary`, `api_reply_core_boundary`
- `SPEC_API_THINKING_ENGINE_BOUNDARY.md`, `# Parallel Split and Decode Boundary`, `api_split_decode_boundary`
- `SPEC_API_THINKING_ENGINE_BOUNDARY.md`, `# In-Memory Boundary`, `api_in_memory_boundary`
- `SPEC_API_THINKING_ENGINE_BOUNDARY.md`, `# Output Branching`, `api_output_branching`
- `SPEC_API_THINKING_ENGINE_BOUNDARY.md`, `# Context Reinjection Boundary`, `api_reinjection_boundary`
- `SPEC_API_THINKING_ENGINE_BOUNDARY.md`, `# Wave LLM Call Boundary`, `wave_llm_call_boundary`
- `SPEC_API_SECTION_LOOP.md`, `# Core Principle`, `api_section_core_principle`
- `SPEC_API_SECTION_LOOP.md`, `# Section Kinds`, `api_section_kinds`
- `SPEC_API_SECTION_LOOP.md`, `# Generic Section Loop`, `api_generic_section_loop`
- `SPEC_API_SECTION_LOOP.md`, `# Section Instructions`, `api_section_instructions`
- `SPEC_API_SECTION_LOOP.md`, `# SQL Response Fragment Shape`, `sql_response_fragment_shape`
- `SPEC_API_SECTION_LOOP.md`, `# API Temporary Context Entry Shape`, `api_tmp_context_entry_shape`
- `SPEC_API_SECTION_LOOP.md`, `# Ordering Rule`, `api_ordering_rule`
- `SPEC_API_SECTION_LOOP.md`, `# Reinjection Rule`, `api_reinjection_rule`
- `SPEC_API_SECTION_LOOP.md`, `# Output Rule`, `api_output_rule`
- `SPEC_API_SECTION_LOOP.md`, `# Prohibited Patterns`, `api_section_prohibited_patterns`

---

# SQL Function Package

- `SPEC_SQL_FUNCTION_PACKAGE.md`, `# Package Namespace`, `sql_function_package_namespace`
- `SPEC_SQL_FUNCTION_PACKAGE.md`, `# Entry Function Families`, `sql_entry_function_families`
- `SPEC_SQL_FUNCTION_PACKAGE.md`, `# Dispatcher Rule`, `sql_dispatcher_rule`
- `SPEC_SQL_FUNCTION_PACKAGE.md`, `# Common Contract`, `sql_common_contract`
- `SPEC_SQL_FUNCTION_PACKAGE.md`, `# Transaction Boundary`, `sql_function_transaction_boundary`
- `SPEC_SQL_FUNCTION_PACKAGE.md`, `# Return Rule`, `sql_function_return_rule`
- `SPEC_SQL_FUNCTION_PACKAGE.md`, `# response_engine.run`, `response_engine_run`
- `SPEC_SQL_FUNCTION_PACKAGE.md`, `# response_engine.reply_step`, `response_engine_reply_step`
- `SPEC_SQL_FUNCTION_PACKAGE.md`, `# response_engine.ingest_input`, `response_engine_ingest_input`
- `SPEC_SQL_FUNCTION_PACKAGE.md`, `# response_engine.tokenize_and_split`, `response_engine_tokenize_and_split`
- `SPEC_SQL_FUNCTION_PACKAGE.md`, `# response_engine.decode_context`, `response_engine_decode_context`
- `SPEC_SQL_FUNCTION_PACKAGE.md`, `# response_engine.build_envelope`, `response_engine_build_envelope`
- `SPEC_SQL_FUNCTION_PACKAGE.md`, `# response_engine.operation_step`, `response_engine_operation_step`
- `SPEC_SQL_FUNCTION_PACKAGE.md`, `# response_engine.scheduler_step`, `response_engine_scheduler_step`
- `SPEC_SQL_FUNCTION_PACKAGE.md`, `# Error and Retry Policy`, `sql_error_retry_policy`
- `SPEC_SQL_FUNCTION_PACKAGE.md`, `# Prohibited Behavior`, `sql_function_prohibited_behavior`

---

# Section Fragment Schema Package

- `SPEC_SECTION_FRAGMENT_SCHEMA.md`, `# Role`, `section_fragment_role`
- `SPEC_SECTION_FRAGMENT_SCHEMA.md`, `# Canonical Fragment Fields`, `section_fragment_fields`
- `SPEC_SECTION_FRAGMENT_SCHEMA.md`, `# Identity Rule`, `section_fragment_identity_rule`
- `SPEC_SECTION_FRAGMENT_SCHEMA.md`, `# Section Kind Rule`, `section_kind_rule`
- `SPEC_SECTION_FRAGMENT_SCHEMA.md`, `# res_key Rule`, `res_key_rule`
- `SPEC_SECTION_FRAGMENT_SCHEMA.md`, `# sequence_tag Rule`, `sequence_tag_rule`
- `SPEC_SECTION_FRAGMENT_SCHEMA.md`, `# Split Ordering Rule`, `split_ordering_rule`
- `SPEC_SECTION_FRAGMENT_SCHEMA.md`, `# Merge Ordering Rule`, `merge_ordering_rule`
- `SPEC_SECTION_FRAGMENT_SCHEMA.md`, `# res_context Rule`, `res_context_rule`
- `SPEC_SECTION_FRAGMENT_SCHEMA.md`, `# source_refs Rule`, `source_refs_rule`
- `SPEC_SECTION_FRAGMENT_SCHEMA.md`, `# evidence_refs Rule`, `evidence_refs_rule`
- `SPEC_SECTION_FRAGMENT_SCHEMA.md`, `# State Rule`, `fragment_state_rule`
- `SPEC_SECTION_FRAGMENT_SCHEMA.md`, `# Persistence Rule`, `fragment_persistence_rule`
- `SPEC_SECTION_FRAGMENT_SCHEMA.md`, `# Commit Rule`, `fragment_commit_rule`
- `SPEC_SECTION_FRAGMENT_SCHEMA.md`, `# Temporary Context Mapping`, `fragment_tmp_context_mapping`
- `SPEC_SECTION_FRAGMENT_SCHEMA.md`, `# Runtime Envelope Mapping`, `fragment_envelope_mapping`
- `SPEC_SECTION_FRAGMENT_SCHEMA.md`, `# Prohibited Patterns`, `fragment_prohibited_patterns`

---

# SQL Response Engine Package

- `SPEC_SQL_RESPONSE_ENGINE_BOUNDARY.md`, `# Ownership`, `sql_response_engine_ownership`
- `SPEC_SQL_RESPONSE_ENGINE_BOUNDARY.md`, `# Tokenization and Split Ownership`, `sql_tokenization_split_ownership`
- `SPEC_SQL_RESPONSE_ENGINE_BOUNDARY.md`, `# Function Package Boundary`, `sql_function_package_boundary`
- `SPEC_SQL_RESPONSE_ENGINE_BOUNDARY.md`, `# Transaction Boundary`, `sql_transaction_boundary`
- `SPEC_SQL_RESPONSE_ENGINE_BOUNDARY.md`, `# Bounded Execution`, `sql_bounded_execution`
- `SPEC_SQL_RESPONSE_ENGINE_BOUNDARY.md`, `# Prohibited Behavior`, `sql_prohibited_behavior`
- `SPEC_SQL_FUNCTION_PACKAGE.md`, `# Entry Function Families`, `sql_entry_function_families`
- `SPEC_SECTION_FRAGMENT_SCHEMA.md`, `# Canonical Fragment Fields`, `section_fragment_fields`
- `SPEC_API_SECTION_LOOP.md`, `# SQL Response Fragment Shape`, `sql_response_fragment_shape`
- `SPEC_RUNTIME_RESULT_ENVELOPE.md`, `# Canonical Fields`, `runtime_envelope_fields`

---

# Temporary Context Package

- `SPEC_TMP_CONTEXT_JSON_BOUNDARY.md`, `# Role`, `tmp_context_role`
- `SPEC_TMP_CONTEXT_JSON_BOUNDARY.md`, `# Decode Projection Rule`, `tmp_context_decode_projection`
- `SPEC_TMP_CONTEXT_JSON_BOUNDARY.md`, `# API Merge and Reinjection Rule`, `tmp_context_merge_reinjection`
- `SPEC_TMP_CONTEXT_JSON_BOUNDARY.md`, `# Identity Fields`, `tmp_context_identity_fields`
- `SPEC_TMP_CONTEXT_JSON_BOUNDARY.md`, `# Storage Forms`, `tmp_context_storage_forms`
- `SPEC_TMP_CONTEXT_JSON_BOUNDARY.md`, `# API Memory Rule`, `tmp_context_api_memory_rule`
- `SPEC_TMP_CONTEXT_JSON_BOUNDARY.md`, `# SQL Access Rule`, `tmp_context_sql_access_rule`
- `SPEC_TMP_CONTEXT_JSON_BOUNDARY.md`, `# Expiry Rule`, `tmp_context_expiry_rule`
- `SPEC_SECTION_FRAGMENT_SCHEMA.md`, `# Temporary Context Mapping`, `fragment_tmp_context_mapping`

---

# Runtime Envelope Package

- `SPEC_RUNTIME_RESULT_ENVELOPE.md`, `# Commit Rule`, `envelope_commit_rule`
- `SPEC_RUNTIME_RESULT_ENVELOPE.md`, `# Canonical Fields`, `envelope_canonical_fields`
- `SPEC_RUNTIME_RESULT_ENVELOPE.md`, `# output_state Values`, `envelope_output_state`
- `SPEC_RUNTIME_RESULT_ENVELOPE.md`, `# next_thinking_action Values`, `envelope_next_thinking_action`
- `SPEC_RUNTIME_RESULT_ENVELOPE.md`, `# Output Text Rule`, `envelope_output_text_ref`
- `SPEC_RUNTIME_RESULT_ENVELOPE.md`, `# Branching Rule`, `envelope_branching_rule`
- `SPEC_SQL_FUNCTION_PACKAGE.md`, `# Return Rule`, `sql_function_return_rule`
- `SPEC_SECTION_FRAGMENT_SCHEMA.md`, `# Runtime Envelope Mapping`, `fragment_envelope_mapping`

---

# DB Scheduled Job Package

- `SPEC_DB_SCHEDULED_JOB_BOUNDARY.md`, `# Canonical Job Kinds`, `db_job_kinds`
- `SPEC_DB_SCHEDULED_JOB_BOUNDARY.md`, `# Job Ownership`, `db_job_ownership`
- `SPEC_DB_SCHEDULED_JOB_BOUNDARY.md`, `# Execution Bounds`, `db_job_execution_bounds`
- `SPEC_DB_SCHEDULED_JOB_BOUNDARY.md`, `# Queue Driver Boundary`, `db_queue_driver_boundary`
- `SPEC_DB_SCHEDULED_JOB_BOUNDARY.md`, `# Promotion Boundary`, `db_job_promotion_boundary`
- `SPEC_CORE_STATE_AND_SCHEDULER.md`, `## scheduler_job`, `scheduler_job_table`
- `SPEC_CORE_STATE_AND_SCHEDULER.md`, `## scheduler_job_run`, `scheduler_job_run_table`
- `SPEC_SQL_FUNCTION_PACKAGE.md`, `# response_engine.scheduler_step`, `response_engine_scheduler_step`

---

# Core State / Scheduler / Remote Trust Package

- `SPEC_CORE_STATE_AND_SCHEDULER.md`, `## core_state`, `core_state`
- `SPEC_CORE_STATE_AND_SCHEDULER.md`, `## core_operation_policy`, `core_operation_policy`
- `SPEC_CORE_STATE_AND_SCHEDULER.md`, `## core_notify_queue`, `core_notify_queue`
- `SPEC_CORE_STATE_AND_SCHEDULER.md`, `## scheduler_job`, `scheduler_job`
- `SPEC_CORE_STATE_AND_SCHEDULER.md`, `## scheduler_job_run`, `scheduler_job_run`
- `SPEC_CORE_STATE_AND_SCHEDULER.md`, `## mastication_job`, `mastication_job`
- `SPEC_CORE_STATE_AND_SCHEDULER.md`, `## remote_event_inbox`, `remote_event_inbox`
- `SPEC_CORE_STATE_AND_SCHEDULER.md`, `## remote_event_quarantine`, `remote_event_quarantine`
- `SPEC_REMOTE_TRUST.md`, `Global switch`, `distributed_sync_enabled`
- `SPEC_REMOTE_TRUST.md`, `Canonical remote node registry`, `remote_node_trust`
- `SPEC_REMOTE_TRUST.md`, `Acceptance rule`, `remote_event_acceptance_rule`

---

# Scale / Cost Package

- `SPEC_SCALE_AND_COST_MODEL.md`, `Growth layers`, `growth_layers`
- `SPEC_SCALE_AND_COST_MODEL.md`, `Per-reply search cost`, `per_reply_search_cost`
- `SPEC_SCALE_AND_COST_MODEL.md`, `Hot-path intelligence`, `hot_path_intelligence`
- `README.md`, `探索量の直感`, `expected_search_cost_intro`

---

# Prohibited / Legacy Guard Package

- `SPEC_PROHIBITED_CANONICAL_PATTERNS.md`, `# Prohibited Canonical Tables`, `prohibited_tables`
- `SPEC_PROHIBITED_CANONICAL_PATTERNS.md`, `# Status Rule`, `prohibited_status_authority`
- `SPEC_PROHIBITED_CANONICAL_PATTERNS.md`, `# Decoder / Constraint Rule`, `prohibited_decoder_trace_loop_guard`
- `SPEC_PROHIBITED_CANONICAL_PATTERNS.md`, `# Web Result Rule`, `prohibited_web_result_table`
- `SPEC_PROHIBITED_CANONICAL_PATTERNS.md`, `# Adoption Rule`, `prohibited_adopt_operations`
- `SPEC_PROHIBITED_CANONICAL_PATTERNS.md`, `# Audit Rule`, `prohibited_adoption_audit`
- `MIGRATION_LEGACY_REGISTER.md`, `# Migrated`, `legacy_migrated_patterns`
- `MIGRATION_LEGACY_REGISTER.md`, `# Reinterpreted`, `legacy_reinterpreted_patterns`
- `MIGRATION_LEGACY_REGISTER.md`, `# Rejected`, `legacy_rejected_patterns`
- `MIGRATION_LEGACY_REGISTER.md`, `# Absorbed`, `legacy_absorbed_patterns`
- `MIGRATION_LEGACY_REGISTER.md`, `# Pending`, `legacy_pending`

---

# SQL Runtime Design Package

This package is the preferred lookup bundle before defining concrete SQL runtime functions, section fragments, temporary context tables, or runtime envelopes.

- `SPEC_SQL_FUNCTION_PACKAGE.md`, `# Entry Function Families`, `sql_entry_function_families`
- `SPEC_SQL_FUNCTION_PACKAGE.md`, `# Dispatcher Rule`, `sql_dispatcher_rule`
- `SPEC_SQL_FUNCTION_PACKAGE.md`, `# Transaction Boundary`, `sql_function_transaction_boundary`
- `SPEC_SQL_FUNCTION_PACKAGE.md`, `# Error and Retry Policy`, `sql_error_retry_policy`
- `SPEC_SECTION_FRAGMENT_SCHEMA.md`, `# Canonical Fragment Fields`, `section_fragment_fields`
- `SPEC_SECTION_FRAGMENT_SCHEMA.md`, `# Merge Ordering Rule`, `merge_ordering_rule`
- `SPEC_SECTION_FRAGMENT_SCHEMA.md`, `# Commit Rule`, `fragment_commit_rule`
- `SPEC_SQL_RESPONSE_ENGINE_BOUNDARY.md`, `# Function Package Boundary`, `sql_function_package_boundary`
- `SPEC_API_SECTION_LOOP.md`, `# SQL Response Fragment Shape`, `sql_response_fragment_shape`
- `SPEC_API_SECTION_LOOP.md`, `# Ordering Rule`, `api_ordering_rule`
- `SPEC_TMP_CONTEXT_JSON_BOUNDARY.md`, `# Identity Fields`, `tmp_context_identity_fields`
- `SPEC_TMP_CONTEXT_JSON_BOUNDARY.md`, `# SQL Access Rule`, `tmp_context_sql_access_rule`
- `SPEC_RUNTIME_RESULT_ENVELOPE.md`, `# Commit Rule`, `envelope_commit_rule`
- `SPEC_RUNTIME_RESULT_ENVELOPE.md`, `# Canonical Fields`, `envelope_canonical_fields`
- `SPEC_DB_SCHEDULED_JOB_BOUNDARY.md`, `# Execution Bounds`, `db_job_execution_bounds`
- `SPEC_OPERATION_GATE.md`, `Canonical operation keys`, `operation_keys`
- `SPEC_REFERENCE_MODEL.md`, `# Core Rule`, `reference_core_rule`
- `SPEC_LOG_AGGREGATE_ARCHIVE.md`, `# Semantic Log Targets`, `semantic_log_targets`
- `SPEC_PROHIBITED_CANONICAL_PATTERNS.md`, `# Short Form`, `prohibited_short_form`
