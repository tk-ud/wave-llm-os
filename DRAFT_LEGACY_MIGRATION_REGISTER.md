# Draft Legacy Migration Register

## Status

This document tracks legacy elements that must be migrated, reinterpreted, archived, or rejected before the Quaternion Exploration Core becomes canonical.

Existing legacy files remain in the repository.

They are not deleted.

They become historical references after the new draft is promoted to the canonical specification.

---

# 1. Migration Classes

Each legacy element is assigned one of the following classes:

```text
migrate      = move into the new canonical core
reinterpret = preserve wording or intent but change mechanism
archive      = keep as historical reference only
reject       = explicitly exclude from canonical authority
absorbed     = handled by an already-canonical mechanism, with no new table/layer
pending      = not yet decided
```

---

# 2. Core Principles

| Legacy element | Class | New location / treatment |
|---|---:|---|
| Meaning originates from observation | migrate | Core premise |
| Meaning does not originate from fixed neural weights | migrate | Core premise |
| Persistent semantic state | migrate | PostgreSQL persistent tables |
| Inspectable semantic state | migrate | token / vocabulary / grammar / relation / logs |
| Expandable semantic state | migrate | draft/adopted spaces, decoherence bank, Phase candidates |
| Decoder does not originate meaning | migrate | coherence decoder + decoder/collapse invariants |
| Explicit collapse | migrate | runtime pipeline |
| Non-destructive learning | migrate | logs.coherence, logs.current, logs.diff, adoption gates |
| Local semantic sovereignty | migrate | distributed observation section |
| Raw external text must not directly update meaning | migrate | mastication and external observation |

---

# 3. wave_geometry

| Legacy element | Class | Reason |
|---|---:|---|
| `wave_geometry` as primary semantic authority | reject | conflicts with grammar/vocabulary/relation core |
| `psi` as primary semantic state | reject | new core has no single semantic state vector/matrix |
| Euler-integrated wave update | archive | historical mechanism only |
| STFT / spectral observer | archive | optional feature channel only, not canonical core |
| spectral basis as meaning basis | reject | replaced by token/vocabulary/grammar/grammar_relation hierarchy |
| wave_signature retrieval | reinterpret | may become optional evidence, not required retrieval key |
| coherence/decoherence vocabulary | migrate | redefined as DB/log/decoherence-bank behavior |
| collapse language | migrate | explicit output collapse and adoption gate |

Canonical statement:

```text
wave_geometry is preserved as legacy archive.
It does not define canonical runtime authority.
```

---

# 4. Runtime Authority

| Legacy element | Class | New location / treatment |
|---|---:|---|
| constitution order | reinterpret | canonical authority order |
| manifest as executable authority | reinterpret | Draft canonical docs + DB functions |
| runtime constants | migrate | core_state and operation policy |
| learning dominance | reinterpret | operation policy + scheduled aggregation |
| fallback hierarchy | reinterpret | decoherence-triggered notify / offline blend / question notify |
| external observation last resort | reinterpret | web search notify from decoherence pressure |
| semantic freeze | migrate | core_state + operation gate |
| semantic snapshot / restore | reject | unnecessary in canonical core because freeze/function gates prevent coherence/current/draft mutation |
| scheduler / maintenance loop | migrate | backend-triggered scheduler_job / scheduler_job_run with DB-owned lock and operation gate |
| Phase relation candidate generation | migrate | DRAFT_PHASE_RELATION_CANDIDATE.md; outputs grammar_index arrays only |
| structural near-neighbor search | migrate | DRAFT_NEAR_NEIGHBOR_SEARCH.md; index arrays are structural semantic vectors |
| canonical index arrays | migrate | DRAFT_INDEX_ARRAY_CANONICAL.md; semantic reference is index only |
| adoption audit table | reject | adoption is mutation evidence; separate table would duplicate logs.diff |
| adoption audit view | migrate | logs.adoption_audit view over logs.diff |
| constraint_rule table | reject | current core already has operation policy and decoder/collapse invariants |
| constraint layer as independent authority | reject | would create a second rule authority after decoder |
| decoder/collapse invariants | absorbed | handled by current core runtime invariants, not a table |
| dense embedding vector search as semantic authority | reject | dense vectors erase structural meaning when treated as source of truth |
| pgvector derived retrieval index | reinterpret | allowed only as acceleration over derived structural vectors; not semantic authority |

Snapshot note:

```text
The canonical core does not need semantic_snapshot / restore as a separate semantic mechanism.

When freeze.enabled is true and operation gates are enforced, logs.coherence, logs.current, draft spaces, adopted spaces, and relation spaces do not mutate through semantic write functions.

Therefore snapshot/restore is not migrated into the canonical runtime.
```

Index-array note:

```text
UUID is identity.
Index is semantic reference.
Array is index array.
Semantic reference never uses UUID.
```

Adoption audit note:

```text
adoption_audit is not a table.

logs.diff is the source of truth for adoption/promotion mutation evidence.

logs.adoption_audit is a view over logs.diff.
```

Constraint note:

```text
constraint_rule is not a canonical table.

Current core mechanisms are sufficient:
- core_state
- core_operation_policy
- core_can_execute()
- logs.diff
- decoder invariants
- collapse invariants

The decoder/collapse stage must not originate meaning, promote candidates, adopt candidates, mutate grammar_relation, mutate core_state, or erase decoherence evidence.
```

Scheduler note:

```text
SQL does not run the scheduler by itself.

A backend worker, cron process, or external runner wakes up and calls DB functions.

The database owns job registry, lock, operation gate, blocked/skipped decisions, and run evidence.
```

Phase note:

```text
Phase relation candidate generation outputs grammar_index arrays.

Because token_index → vocabulary_index → grammar_index → grammar_index array is hierarchical, a decided grammar_index array determines the vocabulary and token references.

Missing slots are completed by reverse hierarchical near-neighbor search.
```

Near-neighbor note:

```text
The index array is the semantic vector.

The hierarchy is the meaning.

Dense embedding vectors are not canonical authority.

pgvector may accelerate retrieval over derived structural vectors, but structural verification decides.
```

---

# 5. Freeze

Migrated as core state and mutation gate.

Canonical state:

```text
freeze.enabled
```

Freeze allows:

- coherence lookup
- relation lookup
- decoder projection
- decoder/collapse invariant check
- output collapse

Freeze blocks:

- adopt.vocabulary
- adopt.grammar
- adopt.relation
- promote.decoherence_to_draft
- promote.phase_relation
- mastication.learn
- external_observation.ingest
- remote semantic update
- current aggregation
- grammar flag update
- phase candidate generation
- core state mutation unless explicitly authorized

Blocked mutations must be recorded in `logs.diff`.

---

# 6. Mastication

Migrated as controlled external/remote observation ingestion.

Canonical objects:

```text
mastication_job
mastication_observation_result
core_state.mastication.enabled
core_operation_policy
logs.diff
```

Rules:

- external text is never direct semantic authority
- remote payloads are never direct semantic authority
- raw text policy defaults to discard after ingest
- mastication learning is blocked during freeze
- mastication queue may remain allowed during freeze

---

# 7. External Observation / Web Search

Legacy fallback hierarchy is reinterpreted.

Canonical behavior:

```text
coherence miss / low-hit
→ decoherence_bank insert on conflict update
→ core_state branch
```

If online:

```text
online.enabled = true
external_observation.enabled = true
freeze.enabled = false
→ notify.web_search_requested
```

If offline:

```text
online.enabled = false
near_blend.enabled = true
→ near vocabulary / grammar / relation replacement
```

If residual pressure recurs:

```text
question_notify.enabled = true
→ notify.question_requested
```

Web search must not be called directly inside ordinary inference.

It is requested from decoherence pressure.

---

# 8. Distributed Sync

Legacy distributed sync is reinterpreted as remote observation intake.

Migrated principles:

- local semantic authority
- no global consensus meaning
- voluntary remote observation
- quarantine before acceptance
- no remote direct adopted mutation

Canonical objects:

```text
remote_event_inbox
remote_event_quarantine
mastication_job
logs.diff
core_state.distributed_sync.enabled
```

Backend may verify signatures, hashes, and replay facts.

Database decides semantic acceptance.

---

# 9. Logs and Aggregates

Migrated / newly canonical:

```text
logs.coherence
logs.current
logs.diff
logs.adoption_audit view
```

Rules:

- `logs.coherence` is append-only observation evidence
- semantic log targets use `target_index`, not `target_uuid`
- `logs.current` is scheduled aggregate read model
- `logs.diff` is mutation/change/time-series evidence
- `logs.adoption_audit` is a view, not a table

End-of-sentence behavior is derived from coherence aggregation:

```text
end_of_sentence_score = 100 * end_of_sentence_count / coherence_count
```

Provisional flag rule:

```text
coherence_count >= 20
end_of_sentence_score >= 70
```

---

# 10. Tokenization

Migrated with mechanism change.

Canonical rule:

- no language-specific morphological analyzer as semantic authority
- tokenizer creates raw atomic records and boundary signals
- `token_index` is the semantic token reference
- meaning aggregation begins at vocabulary, grammar, relation, decoherence, and adoption layers

Canonical tables:

```text
token
vocabulary
vocabulary_token_link
grammar
grammar_vocabulary_link
grammar_relation
grammar_relation_link
```

All semantic arrays and semantic links use indexes.

---

# 11. SQLite

| Legacy / draft element | Class | Reason |
|---|---:|---|
| SQLite MVP profile | reject | lacks canonical near-neighbor/policy/function/notify requirements |
| text UUID fallback | reject | canonical target is PostgreSQL UUID for row identity and BIGINT index for semantic reference |
| JSON as text | reject | canonical target is JSONB |

Canonical storage is PostgreSQL.

---

# 12. Scheduler Registry

Migrated as backend-triggered DB-owned scheduled work control.

Canonical objects:

```text
scheduler_job
scheduler_job_run
core_operation_policy
logs.diff
```

Rules:

- backend worker wakes up jobs
- DB owns job registry
- DB owns lock state
- DB owns operation gate
- DB records run evidence
- freeze blocks semantic write jobs

Initial jobs:

```text
aggregate.logs_current
update.grammar_eos_flag
detect.promotion_candidate
generate.phase_relation_candidate
process.mastication_queue
process.remote_event_inbox
consume.notify_queue
cleanup.expired_locks
```

---

# 13. Phase Relation Candidate

Migrated as scheduled grammar-relation candidate generation.

Canonical object:

```text
phase_relation_candidate
phase_relation_candidate_link
```

Rules:

- Phase reads `logs.current` and other normalized aggregate evidence
- Phase does not inspect raw input as its primary source
- Phase outputs grammar_index arrays only
- grammar_index arrays determine vocabulary/token levels through hierarchy
- missing slots are filled by reverse hierarchical near-neighbor search
- Phase candidates must not directly mutate `grammar_relation`
- promotion must pass `core_can_execute('promote.phase_relation')`

---

# 14. Structural Near-Neighbor Search

Migrated as index-array-based structural search.

Canonical object:

```text
DRAFT_NEAR_NEIGHBOR_SEARCH.md
DRAFT_INDEX_ARRAY_CANONICAL.md
```

Rules:

- ordered BIGINT index arrays are structural semantic vectors
- semantic reference uses index only
- UUID is internal row identity only
- link tables preserve position using indexes
- `logs.current` supplies pressure
- `logs.diff` supplies mutation/adoption evidence
- dense embedding vectors are rejected as semantic authority
- pgvector may be used only as derived retrieval acceleration
- pg_trgm may only support raw-text fallback and is not semantic authority

---

# 15. Pending Migration Items

The following still need detailed canonical table/function specs:

- web search result mastication schema
- remote node trust registry
- decoder trace / loop guard equivalent

---

# 16. Promotion Path

Before this draft becomes canonical:

```text
1. Rewrite Draft philosophy around canonical authority order.
2. Rewrite SQL draft around PostgreSQL canonical tables.
3. Add this migration register.
4. Fill pending migration items.
5. Add canonical function specs.
6. Mark legacy files as historical references.
7. Promote Draft to canonical specification.
```
