# Specification: Canonical Core

## Authority

This file is the single implementation authority for the current core.

Supporting files may explain details, but this file wins on conflict.

---
# Canonical Interpretation Rules

Repository-defined meanings take priority over common implementation patterns.

Terms including `current`, `promotion`, `verification`, `scheduler`, `draft`, `coherence`, and `decoherence` must be interpreted from this specification first.

Do not split canonical concepts because a common database, workflow, approval, queue, or review-process pattern suggests doing so.

## `logs.current`

`logs.current` is the canonical scheduled aggregate pressure surface.

It may be implemented as a table or materialized view.

Its canonical identity remains `logs.current`.

Sleep, Phase Attention, operator actions, and scheduled maintenance may read `logs.current` as the aggregate comparison surface.

Do not introduce a separate canonical `current` table plus `logs.current` history view unless this specification explicitly defines that split.

## Promotion

Promotion is not a scheduler job.

Promotion is a mutation operation after coherence / structural verification evidence passes and `core_can_execute(operation_key)` allows the operation.

The scheduler may generate candidates or consolidate evidence, but it does not replace coherence as the promotion decision axis.

Do not add `phase_candidate_verification`, `phase_candidate_promotion`, or any promotion-eligibility scheduler job.

## Namespace split

`operation_key` and `scheduler_job.job_kind` are separate namespaces.

Canonical scheduler job kinds are:

```text
current_refresh
sleep_consolidation
phase_candidate_generation
```

## Phase

Phase Attention is scheduled relation-candidate generation.

Phase reads aggregate pressure, Draft evidence, decoherence fallback pressure, and relation evidence.

Phase writes draft relation paths to `phase_relation_candidate`.

Phase does not own reply-time promotion semantics.

Phase does not become an approval queue.

## Draft

Draft is not a human-review queue.

Draft is not an approval waiting room.

Draft is an unconfirmed candidate shape, weak or unstable relation evidence, candidate-generation anti-pattern, and Phase Attention negative pressure.

Do not reinterpret Draft through ordinary review workflow language.

## Coherence

Coherence is the promotion axis.

When a candidate coheres through the required structural verification path, it may promote or reinforce through the operation gate.

A verified coherence hit is not merely a recommendation for later human or scheduler review.

A decoherence-bank fallback hit can cohere again if verified against the current input grammar.

## Decoherence

Decoherence is fallback pressure, not trash.

`decoherence_bank` is part of the core semantic search space.

Sleep may send unused, unstable, moth-eaten, or unresolved structures to `decoherence_bank`, but this keeps them searchable as fallback.

Hard deletion remains explicit manual/operator deletion only.

---

---

# Core Rules

```text
UUID  = internal row / event identity only
index = semantic reference key
array = bigint[] index array only
```

Semantic references never use UUID.

Canonical schema does not define `uuid[]` semantic arrays.

Evidence UUIDs may live in JSONB evidence only.

---

# Draft Flag Rule

`draft_flag` is the canonical lifecycle truth.

```text
draft_flag = true
→ draft / residual / unconfirmed / anti-pattern / fallback candidate

draft_flag = false
→ active / promoted / ordinary search target
```

Canonical semantic tables must not depend on a separate enum status table.

`status` is not canonical truth.

If a display label is needed, derive it from flags and log evidence.

---

# Semantic Hierarchy

```text
token_index
→ vocabulary.token_array bigint[]
→ grammar.vocabulary_array bigint[]
→ grammar_relation.grammar_array bigint[]
→ phase_relation_candidate.grammar_array bigint[]
```

The index array is the structural semantic vector.

---

# Canonical Tables

```text
input_observation
token
vocabulary
grammar
grammar_relation
decoherence_bank
logs.coherence
logs.current
logs.diff
core_state
core_operation_policy
core_notify_queue
scheduler_job
scheduler_job_run
phase_relation_candidate
structural_vector_index
mastication_job
remote_node_trust
remote_event_inbox
remote_event_quarantine
```

No canonical `constraint_rule` table.

No canonical `adoption_audit` table.

No canonical web-result-only semantic table.

No canonical `decoder_trace` or `loop_guard` table.

No canonical `status` enum table.

---

# Input Observation

All observations enter the same input pipeline.

Differences are metadata, not separate semantic authority.

```sql
create table input_observation (
  observation_uuid uuid primary key default gen_random_uuid(),
  observation_index bigint generated always as identity unique,
  source_kind text not null,
  source_hash text not null,
  body text not null,
  raw_text_policy text not null default 'discard_after_ingest',
  learning_weight numeric not null default 1.0,
  metadata_json jsonb null,
  created_at timestamptz not null default now()
);
```

Allowed source kinds:

```text
user
web
file
remote_event
manual
system
```

Web search results are normal input observations with `source_kind = 'web'`.

Remote events are normal input observations after inbox/quarantine acceptance.

---

# Remote Trust

Remote trust is linked to core state.

Global switch:

```text
core_state.distributed_sync.enabled
```

Node registry:

```sql
create table remote_node_trust (
  remote_node_uuid uuid primary key default gen_random_uuid(),
  remote_node_id text not null unique,
  trust_level text not null default 'unknown',
  enabled boolean not null default false,
  public_key text null,
  metadata_json jsonb null,
  created_at timestamptz not null default now(),
  updated_at timestamptz not null default now()
);
```

Trust levels:

```text
unknown
trusted
limited
blocked
```

Acceptance rule:

```text
core_state.distributed_sync.enabled = true
and remote_node_trust.enabled = true
and trust_level in ('trusted', 'limited')
→ remote_event may become input_observation(source_kind='remote_event')
```

Even trusted nodes cannot mutate semantic tables directly.

Remote trust changes are mutations and must pass operation gate.

---

# Semantic Tables

## token

```sql
create table token (
  token_uuid uuid primary key default gen_random_uuid(),
  token_index bigint generated always as identity unique,
  raw text null,
  raw_length integer generated always as (char_length(raw)) stored,
  split_flag boolean not null default false,
  split_kind text null,
  split_parent_token_index bigint null references token(token_index),
  split_position integer null,
  metadata_json jsonb null,
  created_at timestamptz not null default now()
);
```

`token` is the canonical token-level seed table.

Tokenization is indexing and segmentation, not semantic interpretation.

## vocabulary

```sql
create table vocabulary (
  vocabulary_uuid uuid primary key default gen_random_uuid(),
  vocabulary_index bigint generated always as identity unique,
  token_array bigint[] not null,
  raw_text text null,
  vocabulary_hash text not null unique,
  split_flag boolean not null default false,
  split_kind text null,
  observed_count bigint not null default 1,
  draft_flag boolean not null default true,
  deleted_flag boolean not null default false,
  first_seen_at timestamptz not null default now(),
  last_seen_at timestamptz not null default now()
);
```

`vocabulary.token_array` is an ordered token-index array.

The tokenizer does not decide whether vocabulary is promoted.

## grammar

```sql
create table grammar (
  grammar_uuid uuid primary key default gen_random_uuid(),
  grammar_index bigint generated always as identity unique,
  vocabulary_array bigint[] not null,
  grammar_hash text not null unique,
  end_of_sentence_flag boolean not null default false,
  draft_flag boolean not null default true,
  deleted_flag boolean not null default false,
  first_seen_at timestamptz not null default now(),
  last_seen_at timestamptz not null default now()
);
```

## grammar_relation

```sql
create table grammar_relation (
  grammar_relation_uuid uuid primary key default gen_random_uuid(),
  grammar_relation_index bigint generated always as identity unique,
  grammar_array bigint[] not null,
  relation_hash text not null unique,
  relation_kind text not null default 'grammar_to_grammar',
  relation_weight numeric not null default 0,
  observed_count bigint not null default 1,
  draft_flag boolean not null default true,
  deleted_flag boolean not null default false,
  first_seen_at timestamptz not null default now(),
  last_seen_at timestamptz not null default now()
);
```

`grammar_relation` is the canonical coherence relation layer.

It stores ordered semantic continuity between grammar candidates.

Relation is not optional metadata.

Optional relation link table:

```sql
create table grammar_relation_link (
  grammar_relation_index bigint not null references grammar_relation(grammar_relation_index),
  grammar_index bigint not null references grammar(grammar_index),
  position integer not null,
  role text null,
  primary key (grammar_relation_index, position)
);
```

## phase_relation_candidate

```sql
create table phase_relation_candidate (
  phase_relation_candidate_uuid uuid primary key default gen_random_uuid(),
  phase_relation_candidate_index bigint generated always as identity unique,
  grammar_array bigint[] not null,
  relation_array bigint[] null,
  relation_hash text not null unique,
  evidence_json jsonb not null,
  phase_score numeric not null default 0,
  pressure numeric not null default 0,
  evidence_count bigint not null default 0,
  draft_flag boolean not null default true,
  rejected_flag boolean not null default false,
  deleted_flag boolean not null default false,
  created_at timestamptz not null default now(),
  last_seen_at timestamptz not null default now(),
  updated_at timestamptz not null default now()
);
```

Phase candidates are draft candidates until operation-gated promotion changes their downstream relation target.

## decoherence_bank

`decoherence_bank` stores unresolved, no-hit, low-hit, unused, unstable, moth-eaten, and mirror-output evidence.

It is searchable fallback structure, not trash.

The bank stores all three semantic layers in JSONB so a fallback candidate can preserve vocabulary, grammar, and relation context together.

```sql
create table decoherence_bank (
  decoherence_uuid uuid primary key default gen_random_uuid(),
  decoherence_index bigint generated always as identity unique,

  residual_hash text not null,
  residual_kind text not null,

  target_table text null,
  target_index bigint null,
  target_path bigint[] null,

  layer_bundle_json jsonb not null,
  evidence_json jsonb not null default '{}'::jsonb,

  pressure numeric not null default 0,
  draft_flag boolean not null default true,
  deleted_flag boolean not null default false,

  first_seen_at timestamptz not null default now(),
  last_seen_at timestamptz not null default now()
);
```

Required `layer_bundle_json` shape:

```json
{
  "vocabulary": {},
  "grammar": {},
  "relation": {}
}
```

Recommended `residual_kind` values:

```text
no_hit
low_hit
partial_hit
unabsorbed
unused
unstable
moth_eaten
mirror_output
manual_delete_candidate
```

`decoherence_bank` entries may be promoted or reinforced only through verified fallback search and operation gate.

They must not follow a `decoherence_bank → draft → adopted` path.

---

# Logs

Canonical log roles:

```text
logs.coherence = append-only observation evidence
logs.current   = scheduled aggregate read model
logs.diff      = mutation / decision evidence
```

Semantic log targets use:

```text
target_table
target_index
target_path
```

## `logs.coherence`

```sql
create table logs.coherence (
  coherence_uuid uuid primary key default gen_random_uuid(),
  coherence_index bigint generated always as identity unique,
  observation_index bigint null references input_observation(observation_index),
  source_path text not null,
  event_kind text not null,
  target_table text null,
  target_index bigint null,
  target_path bigint[] null,
  input_grammar_path bigint[] null,
  candidate_path bigint[] null,
  near_neighbor_score numeric null,
  structural_verification_result text null,
  input_relation_diff_result text null,
  output_delta_kind text null,
  residual_count bigint not null default 0,
  mirror_output_flag boolean not null default false,
  contradiction_flag boolean not null default false,
  decoder_usage_flag boolean not null default false,
  evidence_json jsonb not null default '{}'::jsonb,
  created_at timestamptz not null default now()
);
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

## `logs.diff`

```sql
create table logs.diff (
  diff_uuid uuid primary key default gen_random_uuid(),
  diff_index bigint generated always as identity unique,
  operation_key text not null,
  operation_status text not null,
  target_table text not null,
  target_index bigint null,
  target_path bigint[] null,
  previous_draft_flag boolean null,
  next_draft_flag boolean null,
  previous_deleted_flag boolean null,
  next_deleted_flag boolean null,
  policy_result text not null,
  policy_reason text null,
  evidence_json jsonb not null,
  created_at timestamptz not null default now()
);
```

Allowed `operation_status` values:

```text
allowed
blocked
applied
rejected
quarantined
```

## `logs.current`

`logs.current` is a scheduled aggregate read model.

It may be a table or materialized view, but it must be reproducible from `logs.coherence`, `logs.diff`, and canonical semantic tables.

```sql
create table logs.current (
  current_uuid uuid primary key default gen_random_uuid(),
  target_table text not null,
  target_index bigint null,
  target_path bigint[] null,
  aggregate_window text not null,
  observed_count bigint not null default 0,
  hit_count bigint not null default 0,
  near_hit_count bigint not null default 0,
  partial_hit_count bigint not null default 0,
  residual_count bigint not null default 0,
  contradiction_count bigint not null default 0,
  mirror_output_count bigint not null default 0,
  decoder_success_count bigint not null default 0,
  scope_transition_count bigint not null default 0,
  relation_frequency bigint not null default 0,
  parent_structure_usage_count bigint not null default 0,
  missing_slot_count bigint not null default 0,
  replaced_slot_count bigint not null default 0,
  slot_missing_or_replaced_rate numeric not null default 0,
  moth_eaten_score numeric not null default 0,
  draft_antipattern_score numeric not null default 0,
  phase_score numeric not null default 0,
  pressure numeric not null default 0,
  refreshed_at timestamptz not null default now(),
  unique (target_table, target_index, aggregate_window)
);
```

`logs.current` is not semantic authority.

It is an inspectable pressure surface used by Sleep, Phase Attention, operator actions, and scheduled maintenance.

## Promotion audit view

Promotion audit is a view over `logs.diff`.

```sql
create view logs.promotion_audit as
select *
from logs.diff
where operation_key in (
  'promote.coherence_hit',
  'promote.decoherence_hit',
  'promote.phase_relation'
);
```

Deletion audit is a separate view.

```sql
create view logs.deletion_audit as
select *
from logs.diff
where operation_key in (
  'delete.semantic_structure',
  'delete.decoherence_entry'
);
```

---

# Operation Gate

Mutation-capable operations must call:

```text
core_can_execute(operation_key)
```

Freeze blocks semantic mutation.

Freeze allows read-only lookup, decoder projection, decoder/collapse invariant check, and output collapse.

## Canonical operation keys

```text
promote.coherence_hit
promote.decoherence_hit
promote.phase_relation
sleep.decohere_structure
delete.semantic_structure
delete.decoherence_entry
reject.candidate
quarantine.remote_event
```

No `adopt.*` operation keys are canonical.

Promotion is represented by `promote.*` operations that set or reinforce `draft_flag = false`.

Deletion is explicit and represented by `delete.*` operations.

## Required operation evidence

`core_can_execute(operation_key)` must evaluate policy against explicit evidence.

Near-neighbor similarity alone is never sufficient promotion evidence.

### `promote.coherence_hit`

Allowed only if all are true:

```text
freeze = false
structural_verification_result = 'pass'
input_relation_diff_result in ('pass', 'compatible_delta')
contradiction_flag = false
mirror_output_flag = false
policy_result = 'allowed'
```

Effect:

```text
target.draft_flag = false
logs.diff records previous_draft_flag and next_draft_flag
```

### `promote.decoherence_hit`

Allowed only if all are true:

```text
freeze = false
source_path = decoherence_bank_fallback
structural_verification_result = 'pass'
input_relation_diff_result in ('pass', 'compatible_delta')
contradiction_flag = false
policy_result = 'allowed'
```

A decoherence-bank candidate may cohere again only after verification against the current input grammar.

### `promote.phase_relation`

Allowed only if all are true:

```text
freeze = false
phase_score >= policy.phase_relation.min_phase_score
pressure >= policy.phase_relation.min_pressure
evidence_count >= policy.phase_relation.min_evidence_count
scheduled_structural_verification_result = 'pass'
contradiction_count <= policy.phase_relation.max_contradiction_count
mirror_output_reduction = true
policy_result = 'allowed'
```

Default policy values:

```text
policy.phase_relation.min_phase_score = 0.70
policy.phase_relation.min_pressure = 0.50
policy.phase_relation.min_evidence_count = 3
policy.phase_relation.max_contradiction_count = 0
```

### `sleep.decohere_structure`

Allowed only if all are true:

```text
freeze = false
target_table in ('vocabulary', 'grammar', 'grammar_relation', 'phase_relation_candidate')
reason in ('unused', 'unstable', 'moth_eaten', 'unresolved', 'mirror_output')
policy_result = 'allowed'
```

Sleep never hard-deletes semantic structures.

### `delete.semantic_structure`

Allowed only for explicit manual/operator delete.

```text
freeze = false
source_path in ('manual_operation', 'operator_action')
policy_result = 'allowed'
```

Effect:

```text
target.deleted_flag = true
ordinary search must ignore deleted_flag = true unless explicit maintenance analysis requests it
```

### `delete.decoherence_entry`

Allowed only for explicit manual/operator delete.

```text
freeze = false
source_path in ('manual_operation', 'operator_action')
policy_result = 'allowed'
```

Effect:

```text
decoherence_bank.deleted_flag = true
```

---

# Scoring and Thresholds

Scores are inspectable aggregate calculations, not neural parameters.

Implementations may tune policy values through `core_operation_policy`, but must preserve input evidence and output score components.

## Phase score

```text
phase_score = clamp01(
  0.25 * recurrence_score
+ 0.20 * coherence_score
+ 0.20 * relation_density_score
+ 0.15 * scope_crossing_score
+ 0.10 * decoder_success_score
+ 0.10 * output_delta_score
- 0.20 * contradiction_penalty
- 0.20 * mirror_output_penalty
- 0.15 * draft_antipattern_penalty
)
```

## Draft anti-pattern score

```text
draft_antipattern_score = clamp01(
  0.35 * similarity_to_unresolved_draft
+ 0.25 * mirror_output_rate
+ 0.20 * contradiction_rate
+ 0.20 * unresolved_slot_rate
- 0.30 * fresh_coherence_score
)
```

Candidate generation rule:

```text
new candidate
+ draft_antipattern_score >= 0.60
+ fresh_coherence_score < 0.30
→ reduce phase_score or reject candidate
```

## Moth-eaten score

Moth-eaten detection is based on repeated missing or replaced slots inside an otherwise active parent structure.

```text
missing_rate = missing_slot_count / greatest(parent_structure_usage_count, 1)
replaced_rate = replaced_slot_count / greatest(parent_structure_usage_count, 1)
slot_missing_or_replaced_rate = (missing_slot_count + replaced_slot_count) / greatest(parent_structure_usage_count, 1)
```

```text
moth_eaten_score = clamp01(
  0.50 * missing_rate
+ 0.35 * replaced_rate
+ 0.15 * residual_rate
)
```

Default moth-eaten decision:

```text
parent_structure_usage_count >= 5
and (missing_slot_count + replaced_slot_count) >= 3
and slot_missing_or_replaced_rate >= 0.30
→ moth_eaten evidence
→ sleep.decohere_structure may send the unstable slot or attachment to decoherence_bank
```

Moth-eaten evidence is not immediate deletion.

It is scheduled decoherence evidence.

---

# Core Reply Path

Reply generation is core synchronous work.

It is not Phase Attention and must not wait for scheduled Phase jobs.

Core reply flow:

```text
input_observation
→ token / vocabulary / grammar candidates
→ near-neighbor retrieval
→ structural verification
→ input grammar / grammar_relation diff verification
→ xi coherence hit or yj residual bank
→ coherence relation lookup
→ zk coherence decoder
→ output collapse
```

A reply-time coherence hit must immediately promote a new structure or reinforce an existing active structure through the operation gate.

```text
near-neighbor candidate
→ structural verification passes
→ input grammar / grammar_relation diff verification passes
→ xi coherence hit
→ zk decoder usage
→ core_can_execute('promote.coherence_hit')
→ promote or reinforce immediately
→ logs.diff
```

This promotion is automatic and evidence-driven.

Human review is not part of ordinary reply-time promotion.

Freeze, policy failure, contradiction, or operation gate failure blocks promotion and leaves evidence as draft, residual, rejected, or quarantined state.

---

# Input Grammar / Grammar Relation Diff

Input grammar and candidate grammar_relation must always be diff-compared.

This is mandatory before output collapse, grammar_relation promotion, grammar_relation reinforcement, or decoherence hit promotion.

Near-neighbor similarity alone is not sufficient.

```text
input grammar candidate
↔ candidate grammar_relation.grammar_array
→ structural diff
→ coherence / residual / decoherence decision
```

The diff must inspect:

```text
shared grammar indexes
missing grammar slots
extra relation path elements
replaced vocabulary slots
replaced grammar slots
order shifts
scope boundary mismatch
terminal / sentence-end flag mismatch
```

Required decision pattern:

```text
near relation hit
+ input grammar diff passes
→ coherence relation hit
→ promote / reinforce grammar_relation

near relation hit
+ input grammar diff has missing slots
→ residual / decoherence evidence
→ possible question notification

near relation hit
+ input grammar diff shows repeated missing vocabulary or unstable slot
→ moth-eaten evidence
→ sleep may send vocabulary / slot / relation attachment to decoherence_bank

active relation miss
→ decoherence_bank fallback search
→ input grammar / grammar_relation diff verification
→ promote.decoherence_hit if verified
```

Diff output must be logged as evidence.

---

# Corpus Output / Input Subtraction

Corpus-time output must subtract input.

Mastication, corpus processing, and candidate generation must not treat copied input as meaningful output.

```text
input grammar path
+ candidate output grammar path
→ subtract input grammar path
→ output_delta / relation_delta / residual_delta
```

If subtraction leaves no meaningful delta, the result is mirror output risk.

```text
candidate output
- input grammar
= empty or reorder-only delta
→ mirror_output evidence
→ residual / draft / decoherence evidence
```

Corpus output must therefore be stored or scored as difference, relation, or residual, not as a raw echo of the input.

---

# Decoherence Bank Runtime

`decoherence_bank` is part of the core semantic search space.

It is not an external trash bin.

Primary reply-time search checks active coherent structures first.

If active near-neighbor search does not hit, core may search `decoherence_bank` as a fallback layer.

```text
active structure search
→ no hit
→ decoherence_bank fallback search
→ structural verification
→ input grammar / grammar_relation diff verification
→ xi coherence hit if verified
```

If `decoherence_bank` search hits and structural verification passes, the candidate has cohered again.

A decoherence hit must be promoted or reinforced through the operation gate.

```text
decoherence_bank candidate
→ structural verification passes
→ input grammar / grammar_relation diff verification passes
→ xi coherence hit
→ core_can_execute('promote.decoherence_hit')
→ promote / reinforce
→ logs.diff
```

Sleep may send structures to `decoherence_bank`, but sleep must not hard-delete them.

Deletion from `decoherence_bank` is explicit manual/operator delete only.

---

# Core Expansion / Draft / Sleep Consolidation

Core expands semantic space during reply generation.

Sleep contracts and cleans semantic space during scheduled maintenance.

```text
reply-time core = expansion
sleep / cron    = consolidation, pruning, decoherence
```

Context-local names, file names, branch names, issue numbers, commit identifiers, and other proper nouns must be usable immediately during reply generation.

They must not wait for scheduled Phase confirmation before they can participate in output.

Draft is not a human-review queue.

Draft is not the primary promotion target.

Draft is an unconfirmed candidate filter and anti-pattern evidence store.

Candidate generation must read Draft evidence as negative pressure.

Sleep reads aggregate usage windows from `logs.current`, `logs.coherence`, and `logs.diff`.

Sleep sends unused or unstable vocabulary, grammar, grammar slots, and relation paths to `decoherence_bank`.

Sleep does not make reply generation stricter.

Sleep makes future search cheaper by sending unstable structures to the fallback search layer and letting Phase avoid Draft anti-patterns.

```text
Core first expands.
Sleep later sends unstable structures to decoherence_bank.
Draft filters future Phase candidates as anti-pattern evidence.
Decoherence remains searchable by fallback.
```

---

# Near Neighbor Search

Primary search is structural over index arrays.

`pgvector` may accelerate retrieval over derived structural vectors.

`pgvector` is not semantic authority.

Structural verification decides.

Active structures are searched first.

`decoherence_bank` is searched only when active structures do not hit, or when explicit operator / maintenance analysis requests it.

Relation candidates returned by near-neighbor search must be verified by input grammar / grammar_relation diff before reply-time use.

---

# Phase

Phase Attention is cron-like scheduled aggregate-weighted relation candidate generation.

Phase reads aggregate pressure and evidence.

Phase reads `logs.current` as the scheduled pressure surface.

Phase reads Draft evidence as unconfirmed candidate filters and anti-pattern pressure.

Phase may read `decoherence_bank` as a source of Draft candidates.

When Phase generates a Draft grammar_relation from `decoherence_bank` evidence, its child vocabulary, grammar, and grammar_relation elements must remain `draft_flag = true` for that candidate path.

Phase writes draft relation paths to `phase_relation_candidate`.

Phase outputs grammar_index arrays only.

Promotion must pass operation gate.

Phase is not the synchronous raw-input reply path.

Phase must not define reply-time promotion semantics; reply-time coherence promotion belongs to the core reply path.

Phase candidate generation must reduce or reject candidates that reproduce unresolved Draft anti-patterns without new coherence evidence.

Phase relation candidates become promotion candidates only through the `phase_score` and operation gate rules defined in this file.

---

# Decoder / Collapse

Decoder does not originate meaning.

Decoder/collapse invariants are handled by the current core.

No independent constraint table.

No decoder trace table.

No loop guard table.

Looping is unresolved exploration pressure, not a separate error state.

Repeated return to the same grammar path is handled by:

```text
grammar_array structure
Phase Attention candidate search
near-neighbor expansion
decoherence_bank pressure
logs.current pressure
input grammar / grammar_relation diff evidence
```

If repeated search still cannot resolve, the result remains draft/unresolved and may trigger question notification.
