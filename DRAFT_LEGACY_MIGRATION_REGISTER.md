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
| Decoder does not originate meaning | migrate | coherence decoder section |
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
| spectral basis as meaning basis | reject | replaced by vocabulary / grammar / grammar_relation |
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
| constraint layer | migrate | post-decoder pre-collapse stabilizer |

Snapshot note:

```text
The canonical core does not need semantic_snapshot / restore as a separate semantic mechanism.

When freeze.enabled is true and operation gates are enforced, logs.coherence, logs.current, draft spaces, adopted spaces, and relation spaces do not mutate through semantic write functions.

Therefore snapshot/restore is not migrated into the canonical runtime.
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
- constraint apply, if read-only
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
```

Rules:

- `logs.coherence` is append-only observation evidence
- `logs.current` is scheduled aggregate read model
- `logs.diff` is mutation/change/time-series evidence

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
- tokenizer creates raw atomic references and boundary signals
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

---

# 11. SQLite

| Legacy / draft element | Class | Reason |
|---|---:|---|
| SQLite MVP profile | reject | lacks canonical near-neighbor/policy/function/notify requirements |
| text UUID fallback | reject | canonical target is PostgreSQL UUID |
| JSON as text | reject | canonical target is JSONB |

Canonical storage is PostgreSQL.

---

# 12. Pending Migration Items

The following still need detailed canonical table/function specs:

- constraint rule table
- Phase relation candidate table
- adoption audit table, or explicit use of logs.diff for adoption audit
- near-neighbor extension choice
- web search result mastication schema
- remote node trust registry
- scheduler job registry
- decoder trace / loop guard equivalent

---

# 13. Promotion Path

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
