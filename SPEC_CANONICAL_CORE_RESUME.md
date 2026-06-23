# Specification: Canonical Core Resume

## Authority

This file is the routing-level canonical core.

Detailed authority lives in routed `SPEC_*` files.

`NOTE_*` files are implementation projections or explanatory background only.

---

# Canonical Interpretation Rules

Repository-defined meanings take priority over common implementation patterns.

Terms including `current`, `promotion`, `verification`, `scheduler`, `draft`, `coherence`, and `decoherence` must be interpreted from the routed SPEC files first.

Runtime behavior terms including `wave`, `coherence`, `decoherence`, `pressure`, `Phase`, `Sleep`, and `collapse` are defined by `SPEC_RUNTIME_BEHAVIOR_MODEL.md`.

Do not split canonical concepts because a common database, workflow, approval, queue, or review-process pattern suggests doing so.

---

# Core Invariants

```text
UUID  = internal row / event identity only
index = semantic reference key
array = bigint[] index array only
```

`draft_flag` remains the canonical lifecycle truth.

Operation-capable semantic mutation must pass the operation gate.

Logs, aggregate tables, archive registries, and implementation notes are not semantic authority.

Explicitly prohibited canonical patterns are defined by `SPEC_PROHIBITED_CANONICAL_PATTERNS.md`.

---

# Document Routing

```text
Runtime behavior model        -> SPEC_RUNTIME_BEHAVIOR_MODEL.md
Reference model               -> SPEC_REFERENCE_MODEL.md
Semantic tables               -> SPEC_SEMANTIC_TABLES.md
Log / aggregate/archive       -> SPEC_LOG_AGGREGATE_ARCHIVE.md
Operation gate                -> SPEC_OPERATION_GATE.md
Scoring / thresholds          -> SPEC_SCORING_AND_THRESHOLDS.md
Remote trust                  -> SPEC_REMOTE_TRUST.md
Core state / scheduler        -> SPEC_CORE_STATE_AND_SCHEDULER.md
Prohibited canonical patterns -> SPEC_PROHIBITED_CANONICAL_PATTERNS.md
Reply pipeline                -> SPEC_REPLY_PIPELINE.md
Cron pipeline                 -> SPEC_CRON_PIPELINE.md
Search / verification         -> SPEC_SEARCH_AND_VERIFICATION.md
Scale / cost model            -> SPEC_SCALE_AND_COST_MODEL.md
NOTE role map                 -> NOTE_SQL_IMPLEMENTATION_MAP.md
```

`NOTE_SQL_IMPLEMENTATION_MAP.md` records which NOTE files remain as implementation or explanatory support after canonicalization.

---

# Runtime Behavior Resume

Runtime behavior names are abstract behavior names projected into inspectable database structures.

Detailed authority: `SPEC_RUNTIME_BEHAVIOR_MODEL.md`.

---

# Reply Path Resume

```text
input_observation
-> token
-> vocabulary
-> grammar
-> coherence search
-> structural verification
-> relation or residual handling
-> decoder projection
-> collapse
-> evidence logs
```

Detailed authority: `SPEC_REPLY_PIPELINE.md`.

---

# Cron Path Resume

```text
current_refresh
-> aggregate.current upsert
-> optional logs.current snapshot
-> phase_candidate_generation
-> sleep_consolidation
-> archive_rolloff
```

Detailed authority: `SPEC_CRON_PIPELINE.md`.

---

# Semantic Table Resume

```text
input_observation
token
vocabulary
grammar
grammar_relation
grammar_relation_link
decoherence_bank
phase_relation_candidate
structural_vector_index
```

Detailed authority: `SPEC_SEMANTIC_TABLES.md`.

---

# Log / Aggregate / Archive Resume

```text
logs.coherence     = append-only observation evidence
logs.diff          = mutation / decision evidence
logs.current       = aggregate snapshot history / time-series evidence
aggregate.current  = hot aggregate read model / pressure surface
archive.registry   = archive batch registry and restore manifest
```

Detailed authority: `SPEC_LOG_AGGREGATE_ARCHIVE.md`.

---

# Operation / Policy Resume

All mutation-capable operations pass `core_can_execute(operation_key)`.

Freeze blocks semantic mutation but allows read-only lookup, decoder projection, invariant check, and output collapse.

Detailed authority: `SPEC_OPERATION_GATE.md`.

---

# Core State / Scheduler Resume

Core state, operation policy tables, notification queue, scheduler job tables, mastication jobs, and remote event inbox/quarantine are runtime support tables.

They are not semantic authority.

Detailed authority: `SPEC_CORE_STATE_AND_SCHEDULER.md`.

---

# Prohibited Pattern Resume

No canonical `constraint_rule`, `adoption_audit`, web-result-only semantic table, `decoder_trace`, `loop_guard`, or status enum authority.

No `adopt.*` operation keys are canonical.

Detailed authority: `SPEC_PROHIBITED_CANONICAL_PATTERNS.md`.

---

# Search / Verification Resume

Canonical search uses semantic index arrays first.

Near-neighbor or vector search may accelerate retrieval, but verification must return to structural references.

Mirror-output, residual, fallback, and promotion evidence must be logged.

Detailed authority: `SPEC_SEARCH_AND_VERIFICATION.md`.

---

# Scoring Resume

Scores are inspectable aggregate calculations, not neural parameters.

Detailed authority: `SPEC_SCORING_AND_THRESHOLDS.md`.

---

# Remote Trust Resume

Remote events may become input observations only after trust and quarantine rules pass.

Trusted nodes cannot mutate semantic tables directly.

Detailed authority: `SPEC_REMOTE_TRUST.md`.

---

# Scale / Cost Resume

Hot-path intelligence depends on:

```text
current semantic structures
aggregate.current pressure
searchable fallback structures
recent decision evidence
operation-gated promotion results
```

Canonical semantic growth is expected to become sublinear under repeated observations because existing vocabulary, grammar, and relation structures are reinforced.

Detailed authority: `SPEC_SCALE_AND_COST_MODEL.md`.

---

# Notes After Canonicalization

`NOTE_*` files are implementation or explanatory notes unless explicitly promoted into a `SPEC_*` file.

If a `NOTE_*` file conflicts with a `SPEC_*` file, the `SPEC_*` file wins.
