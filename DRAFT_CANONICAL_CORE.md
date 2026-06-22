# Draft: Canonical Core

## Authority

This file is the single draft authority for the new core.

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
remote_event_inbox
remote_event_quarantine
```

No canonical `constraint_rule` table.

No canonical `adoption_audit` table.

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

# Input Sources

All observations enter the same input pipeline with source metadata.

```text
source_kind = user | web | file | remote_event | manual
source_hash
raw_text_policy
learning_weight
metadata_json
```

Web search results are not a separate semantic authority.

They are source-kinded input observations.

---

# Near Neighbor Search

Primary search is structural over index arrays.

`pgvector` may accelerate retrieval over derived structural vectors.

`pgvector` is not semantic authority.

Structural verification decides.

---

# Phase

Phase reads aggregate pressure and evidence.

Phase outputs grammar_index arrays only.

Promotion must pass operation gate.

---

# Decoder / Collapse

Decoder does not originate meaning.

Decoder/collapse invariants are handled by the current core.

No independent constraint table.

---

# Pending

```text
remote node trust registry
decoder trace / loop guard
input source metadata schema details
```
