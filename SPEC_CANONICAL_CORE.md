# Specification: Canonical Core

## Authority

This file is the single implementation authority for the new core.

Supporting files may explain details, but this file wins on conflict.

---

# Core Rules

```text
UUID  = internal row / event identity only
index = semantic reference key
array = bigint[] index array only
```

Semantic references never use UUID.

Canonical schema does not define `uuid[]` columns.

Evidence UUIDs live in JSONB evidence only.

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

# Logs

```text
logs.coherence = append-only observation evidence
logs.current   = scheduled aggregate read model
logs.diff      = mutation / decision evidence
```

Semantic log targets use:

```text
target_table
target_index
```

Adoption audit is a view:

```sql
create view logs.adoption_audit as
select *
from logs.diff
where operation_key in (
  'adopt.vocabulary',
  'adopt.grammar',
  'adopt.relation',
  'promote.coherence_hit',
  'promote.decoherence_hit',
  'promote.decoherence_to_draft',
  'promote.phase_relation'
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

If near-neighbor retrieval hits and structural verification passes, the candidate has cohered.

A reply-time coherence hit must immediately adopt a new structure or reinforce an existing adopted structure through the operation gate.

```text
near-neighbor candidate
→ structural verification passes
→ input grammar / grammar_relation diff verification passes
→ xi coherence hit
→ zk decoder usage
→ core_can_execute('promote.coherence_hit')
→ adopt or reinforce immediately
→ logs.diff
```

This promotion is automatic and evidence-driven.

Human review is not part of ordinary reply-time promotion.

Freeze, policy failure, contradiction, or operation gate failure blocks adoption and leaves evidence as draft, residual, rejected, or quarantined state.

---

# Input Grammar / Grammar Relation Diff

Input grammar and candidate grammar_relation must always be diff-compared.

This is mandatory before output collapse, grammar_relation adoption, grammar_relation reinforcement, or decoherence hit promotion.

Near-neighbor similarity alone is not sufficient.

```text
input grammar candidate
↔ candidate grammar_relation.grammar_array
→ structural diff
→ coherence / residual / decoherence decision
```

Input grammar means the grammar candidate or grammar path generated from the current input scope.

Candidate grammar_relation means an active `grammar_relation.grammar_array`, a draft `phase_relation_candidate.grammar_array`, or a relation path recovered from `decoherence_bank` fallback search.

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

The purpose of input is to create and reinforce grammar_relation.

Without this diff, the system may select a nearby relation path while failing to notice that the current input grammar is moth-eaten, shifted, incomplete, or merely mirrored.

Required decision pattern:

```text
near relation hit
+ input grammar diff passes
→ coherence relation hit
→ adopt / reinforce grammar_relation

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

```text
input_grammar_index or input_grammar_path
candidate_relation_index or candidate_relation_path
diff_kind
missing_slots
extra_slots
replaced_slots
order_shift
terminal_flag_match
scope_boundary_match
decision
```

This diff is part of core reply-time verification.

Phase Attention may generate candidate grammar_relation paths, but core decides reply-time coherence by comparing current input grammar against the candidate grammar_relation.

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

Required corpus decision pattern:

```text
output_delta contains new relation evidence
→ relation candidate / grammar_relation reinforcement

output_delta contains only copied input
→ mirror_output evidence
→ do not reinforce as new relation

output_delta contains missing or unstable slots
→ residual / decoherence evidence

output_delta bridges input scopes
→ candidate grammar_relation evidence
```

This rule applies before Phase candidate generation, before grammar_relation reinforcement, and before decoder/collapse evidence is counted as success.

The purpose is to prevent corpus processing from rewarding parroting.

```text
meaningful corpus output
= candidate output - input grammar
```

---

# Decoherence Bank

`decoherence_bank` is part of the core semantic search space.

It is not an external trash bin.

Core does not remove decohered structures from possible future use.

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

Deletion from `decoherence_bank` is explicit UI action only.

```text
sleep / cron
→ send unused or unstable structures to decoherence_bank
→ keep searchable as fallback
→ explicit UI deletion only
```

`decoherence_bank` and Draft collections should be visible to UI tooling for token assignment and human-readable analysis.

This UI analysis is not ordinary promotion review.

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

```text
context-local observation
→ temporary vocabulary / grammar candidates
→ reply-time usage
→ logs.coherence evidence
→ sleep decides whether to retain, merge, or send to decoherence_bank
```

Draft is not a human-review queue.

Draft is not the primary promotion target.

Draft is an unconfirmed candidate filter and anti-pattern evidence store.

```text
draft
= unconfirmed candidate shape
= weak or unstable relation evidence
= candidate-generation anti-pattern
= Phase Attention negative pressure
```

Candidate generation must read Draft evidence as negative pressure.

If a new Phase candidate is structurally close to an unresolved Draft anti-pattern, candidate score must be reduced unless fresh coherence evidence overcomes that penalty.

```text
new candidate
+ similar draft anti-pattern
+ no new coherence evidence
→ lower phase_score
→ keep draft / residual / rejected
```

Sleep runs through `scheduler_job` / cron-like maintenance.

Sleep reads aggregate usage windows from `logs.current`, `logs.coherence`, and `logs.diff`.

Sleep sends unused or unstable vocabulary, grammar, grammar slots, and relation paths to `decoherence_bank`.

Unused vocabulary:

```text
vocabulary unused for configured window
→ send to decoherence_bank
→ keep searchable as fallback
→ explicit UI deletion only
```

Moth-eaten vocabulary inside active upper structures:

```text
parent_structure_usage_count is high
and vocabulary_missing_slot_count / parent_structure_usage_count is high
→ vocabulary decoheres from that parent grammar or relation path
→ send decoherence evidence to decoherence_bank
→ keep searchable as fallback
```

Moth-eaten grammar slots:

```text
parent_structure_usage_count is high
and slot_missing_or_replaced_count / parent_structure_usage_count is high
→ grammar slot decoheres from that parent grammar or relation path
→ send decoherence evidence to decoherence_bank
→ keep searchable as fallback
```

Here, `parent_structure_usage_count` may refer to an active grammar, grammar_relation, or grammar_array path.

`vocabulary_missing_slot_count` counts repeated cases where the parent structure is used but the vocabulary slot is missing, replaced, unresolved, or pushed into residual.

`slot_missing_or_replaced_count` counts repeated cases where a grammar slot itself remains unstable even when the parent structure is active.

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

`decoherence_bank` is searched only when active structures do not hit, or when explicit UI / maintenance analysis requests it.

Relation candidates returned by near-neighbor search must be verified by input grammar / grammar_relation diff before reply-time use.

---

# Phase

Phase Attention is cron-like scheduled aggregate-weighted relation candidate generation.

Phase reads aggregate pressure and evidence.

Phase reads `logs.current` as the scheduled pressure surface.

Phase reads Draft evidence as unconfirmed candidate filters and anti-pattern pressure.

Phase may read `decoherence_bank` as a source of Draft candidates.

When Phase generates a Draft grammar_relation from `decoherence_bank` evidence, its child vocabulary, grammar, and grammar_relation elements must also be assigned Draft status for that candidate path.

Phase writes draft relation paths to `phase_relation_candidate`.

Phase outputs grammar_index arrays only.

Promotion must pass operation gate.

Phase is not the synchronous raw-input reply path.

Phase must not define reply-time adoption semantics; reply-time coherence promotion belongs to the core reply path.

Phase candidate generation must reduce or reject candidates that reproduce unresolved Draft anti-patterns without new coherence evidence.

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
