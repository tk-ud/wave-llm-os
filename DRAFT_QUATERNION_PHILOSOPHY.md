# Draft Philosophy: Quaternion Exploration Core

## Status

This document is the working draft for the next conceptual core of Wave LLMOS.

The existing spectral-wave documents remain in the repository as legacy references. They are not deleted. They preserve the historical origin of the project, but they do not define the canonical runtime authority of the next core.

The next core replaces the old `wave_geometry` authority with a database-centered quaternion exploration model over vocabulary, grammar, grammar relation, decoherence, coherence logs, scheduled aggregation, core state, and explicit adoption.

---

# 1. Preserved Legacy Principles

The following legacy principles remain valid and are migrated into the new core:

- meaning does not originate from fixed neural weights
- meaning is generated from observation
- semantic state should be persistent, inspectable, and expandable
- the decoder must not be treated as the origin of meaning
- collapse must be explicit and bounded
- learning must be non-destructive
- raw external observation must not directly define meaning
- distributed nodes must not create global semantic authority

The following legacy mechanism is not migrated as canonical authority:

- `wave_geometry` as primary semantic state
- `psi` / spectral matrix as meaning authority
- Euler-integrated wave state as the core semantic substrate
- STFT / spectral basis as the canonical meaning basis
- wave-signature search as the required semantic retrieval key

Those elements may remain as historical references or optional experimental feature channels, but they must not override the canonical grammar/vocabulary/relation core.

---

# 2. Core Premise

Meaning does not exist inside a word alone.

Meaning emerges from the connection form between elements.

A word, token, character, table value, status, phrase, event, grammar fragment, or relation fragment is not meaningful by itself.

Meaning appears when those elements are connected through vocabulary, grammar, usage, recurrence, residual pressure, coherence, decoherence, relation, scheduled aggregation, and explicit adoption.

Therefore, the system must not begin by assuming a fixed semantic vector space.

It must observe input, compare it against existing vocabulary/grammar/relation spaces, preserve multiple near-neighbor candidates, store unabsorbed differences, connect divided candidates through relation, and only then generate draft or adopted meaning structures.

---

# 3. Canonical Authority Order

The canonical runtime authority is ordered as follows:

```text
1. Quaternion Exploration Core
2. Core State / Operation Gate
3. Persistent Semantic Tables
   - vocabulary
   - grammar
   - grammar_relation
   - decoherence_bank
   - logs.coherence
   - logs.current
   - logs.diff
4. Coherence / Decoherence / Relation Runtime
5. Scheduled Aggregation and Phase Candidate Generation
6. Coherence Decoder
7. Decoder / Collapse Invariants
8. External / Remote Observation Intake
9. Legacy Archive
```

There is no independent canonical constraint table.

Constraint behavior is absorbed into the existing core operation policy and decoder/collapse invariants.

The backend must not become semantic authority.

The backend may transport input, verify cryptographic facts, execute workers, and call database functions.

The database decides whether an operation is accepted, rejected, deferred, quarantined, queued, or blocked.

The runtime asks:

```text
core_can_execute(operation_key)
```

It does not hardcode semantic mutation rules in application code.

---

# 4. Quaternion-Style Processing Equation

The draft core is expressed as:

```text
q = w + xi + yj + zk
```

Where:

```text
q = output candidate state
w = input anchor

xi = coherence layer
yj = decoherence bank
zk = coherence decoder
```

Expanded:

```text
w = input anchor

x = token bundle / vocabulary bundle / segmented input fragments
i = regular grammar candidate, regular vocabulary candidate, or draft candidate

xi = coherence layer
   = search whether x can be absorbed by i


y = unsearchable difference / no-hit residual / low-hit residual
j = draft grammar candidate and draft accumulation context

yj = decoherence bank
   = store y into j-side draft accumulation


z = exploration hit bundle, including relation evidence
k = corrected grammar candidate

zk = coherence decoder
   = project z through k into an output-capable grammar candidate
```

This is quaternion-style, not strict Hamilton quaternion algebra.

The symbols `i`, `j`, and `k` are candidate-bearing procedural axes.

The important algebraic property is procedural non-commutativity:

```text
ij != ji
```

Operationally:

```text
xi finds what can be absorbed.
yj stores what cannot be absorbed.
Relation connects divided coherent candidates.
The composition of i and j produces k.
zk uses k to decode the surviving hit bundle into corrected grammar.
```

The system does not start from:

```text
Which token is closest?
```

It starts from:

```text
Which coherence/decoherence/relation path can produce a corrected grammar candidate for output?
```

---

# 5. Canonical Semantic Spaces

The next core separates semantic space into layered persistent spaces:

```text
token
vocabulary
grammar
grammar_relation
decoherence_bank
logs.coherence
logs.current
logs.diff
phase_relation_candidate
core_state
core_operation_policy
core_notify_queue
```

These spaces are not fixed ontologies.

They are observed, updated, and separated by usage, pressure, recurrence, relation, scheduled aggregation, and explicit adoption.

## 5.1 token

`token` is an atomic raw identity layer.

It may represent a character, separator, punctuation mark, code fragment, or other atomic unit.

The token layer is not semantic authority.

## 5.2 vocabulary

`vocabulary` is an ordered token bundle.

A vocabulary candidate may be a word, phrase, code, label, status, domain expression, file path, URL fragment, or business term.

## 5.3 grammar

`grammar` is an ordered vocabulary bundle.

A grammar candidate represents a connection form among vocabulary candidates.

It is not a universal grammar rule.

It is a locally observed semantic-grammar structure.

## 5.4 grammar_relation

`grammar_relation` is an ordered grammar bundle.

It is the semantic continuity memory that connects divided grammar candidates across scopes.

Without grammar relation, the system can identify local fragments but cannot connect the beginning and end of divided input.

## 5.5 decoherence_bank

`decoherence_bank` stores no-hit, low-hit, and unabsorbed residuals.

It is not an error table.

It is the source of future draft vocabulary, draft grammar, web-search notifications, offline near-neighbor replacement, and question notifications.

## 5.6 logs.coherence

`logs.coherence` is an append-only observation log.

It records that a candidate cohered, nearly cohered, partially cohered, related, or appeared as sentence-ending in context.

It does not decide adoption.

## 5.7 logs.current

`logs.current` is the scheduled aggregate read model generated from `logs.coherence`.

Runtime and schedulers read current aggregate values from `logs.current`.

## 5.8 logs.diff

`logs.diff` stores mutation evidence and time-series change records.

It records draft-to-adopted changes, normalization, flag changes, core state changes, blocked mutations, remote-event decisions, and mastication decisions.

---

# 6. Coherence Layer: xi

The `xi` term is the coherence layer.

It searches:

- adopted vocabulary
- adopted grammar
- adopted grammar relation
- draft vocabulary
- draft grammar
- draft grammar relation
- near-neighbor candidates
- partial matches
- incomplete grammar candidates

If the input hits or nearly hits candidates, the system appends coherence facts into `logs.coherence`.

Coherence does not immediately become adoption.

Coherence becomes decision material only after scheduled aggregation into `logs.current` and operation-gated promotion/adoption.

---

# 7. Decoherence Bank: yj

The `yj` term is the decoherence bank.

The decoherence bank stores what the coherence layer could not absorb.

This includes:

- new words
- unknown expressions
- no-hit token/vocabulary hashes
- low-hit token/vocabulary hashes
- unresolved grammar fragments
- unmatched grammar slots
- repeated but not-yet-adopted patterns

The decoherence bank is intentionally persistent.

Unabsorbed differences must not be discarded.

A successful insert or conflict update into the decoherence bank is also an event source.

After `decoherence_bank insert on conflict update`, the database evaluates core state:

```text
if online.enabled = true
and external_observation.enabled = true
and freeze.enabled = false:
  enqueue notify.web_search_requested

if online.enabled = false
and near_blend.enabled = true:
  attempt near vocabulary / grammar / relation replacement

if residual pressure becomes frequent enough:
  enqueue notify.question_requested
```

Web search is not performed directly inside ordinary inference.

Web search is requested by notification from decoherence pressure.

Offline mode does not request web search.

Offline mode attempts near-neighbor replacement using known vocabulary, grammar, and relation candidates. If unresolved residuals recur, the system asks the user through a question notification.

---

# 8. Core State and Operation Gate

Runtime state is stored in `core_state`.

Examples:

```text
freeze.enabled
online.enabled
mastication.enabled
external_observation.enabled
distributed_sync.enabled
question_notify.enabled
near_blend.enabled
```

Mutation-capable operations must pass through `core_can_execute(operation_key)`.

Examples:

```text
adopt.vocabulary
adopt.grammar
adopt.relation
promote.decoherence_to_draft
promote.phase_relation
mastication.queue
mastication.observe
mastication.learn
external_observation.ingest
remote_event.ingest
```

`freeze.enabled = true` prohibits semantic mutation.

Freeze allows read-only coherence lookup, relation lookup, decoder projection, and output collapse.

Freeze blocks adoption, promotion, mastication learning, external adoption, and distributed semantic update.

Blocked operations must be recorded in `logs.diff`.

A separate semantic snapshot/restore mechanism is not canonical.

When freeze and operation gates are enforced, coherence logs, current aggregates, draft spaces, adopted spaces, and relation spaces do not mutate through semantic write functions.

---

# 9. Mastication

Mastication is the controlled ingestion pipeline for external or remote observation.

External text or remote payloads must not directly update adopted vocabulary, adopted grammar, adopted relation, or core state.

They may become:

- rejected input
- quarantined observation
- mastication job
- draft candidate evidence
- decoherence pressure

The mastication path is:

```text
external / remote source
→ DB ingest
→ core_state / operation gate
→ quarantine or mastication_job
→ token / vocabulary candidate generation
→ xi coherence lookup
→ yj decoherence bank
→ relation candidate update
→ scheduled aggregation
→ possible draft/adoption decision through operation gate
→ raw text discard policy
```

Mastication may be queued while freeze is enabled, but mastication learning must be blocked during freeze.

---

# 10. Distributed Observation

Distributed synchronization is treated as remote observation intake, not semantic consensus.

Remote events are not semantic updates.

A remote event must not directly mutate:

- adopted vocabulary
- adopted grammar
- adopted grammar relation
- core_state

The backend may verify cryptographic facts such as signatures, hashes, and replay protection.

The database decides semantic acceptance, quarantine, rejection, deferral, or mastication queueing.

There is no canonical distributed semantic state.

Semantic authority remains local-node only.

---

# 11. Phase Relation Candidate Generation

Phase is not synchronous raw-input reasoning.

Phase reads normalized aggregate values, especially from `logs.current`, vocabulary, grammar, grammar relation, decoherence pressure, and promotion history.

Phase produces draft relation candidates.

Phase must not silently mutate adopted grammar or adopted relation.

Phase output is subject to operation-gated promotion/adoption.

---

# 12. Candidate Bundles Must Not Collapse Early

Near-neighbor search must preserve multiple candidates.

The system must not select only one nearest grammar or vocabulary item too early.

Instead, it should operate on bundles:

```text
vocabulary_near_candidates = {v1, v2, v3, ...}
grammar_near_candidates = {g1, g2, g3, ...}
grammar_relation_candidates = {r1, r2, r3, ...}
residual_candidates = {d1, d2, d3, ...}
phase_relation_candidates = {p1, p2, p3, ...}
```

Multiple incomplete candidates preserve openings for correction.

Early collapse destroys correction paths.

---

# 13. End-of-Sentence as Observed Pressure

End-of-sentence flags are not hardcoded by punctuation alone.

When input grammar coheres and the observation context indicates sentence end, the system records:

```text
logs.coherence.end_of_sentence_observed = true
```

A scheduled job aggregates coherence logs into `logs.current`.

Provisional score:

```text
end_of_sentence_score = 100 * end_of_sentence_count / coherence_count
```

Provisional promotion rule:

```text
if coherence_count >= minimum_observation_count
and end_of_sentence_score >= 70:
  set end_of_sentence_flag = true
```

The flag change must be recorded in `logs.diff`.

This makes sentence-ending behavior observation-derived rather than purely rule-defined.

---

# 14. Summarization as Grammar-Space Projection

Summarization is not merely deletion.

Summarization is projection into a selected grammar space and relation depth.

When summarization is requested, the system should:

```text
observe vocabulary pressure
→ identify strong vocabulary bundles
→ search grammar space from those vocabulary pressures
→ include grammar relation evidence
→ produce multiple summary grammar candidates
→ select compression level from candidate grammar forms
→ decode through corrected grammar
→ collapse output
```

Compression level is not only a token budget.

It is a choice of grammar projection and relation depth.

---

# 15. Decoder / Collapse Invariants

There is no independent canonical constraint table.

The current core already provides the necessary constraints through:

```text
core_state
core_operation_policy
core_can_execute()
logs.diff
decoder invariants
collapse invariants
```

Decoder/collapse invariants are not a separate mutable semantic authority.

They are fixed runtime invariants that prevent the decoder from becoming the origin of meaning.

They include:

- decoder must not originate meaning
- decoder must not adopt candidates
- decoder must not promote candidates
- decoder must not mutate grammar_relation
- decoder must not mutate core_state
- decoder must not erase decoherence evidence
- collapse must not silently convert draft candidates into adopted meaning
- freeze must still block semantic mutation

Mutation guard belongs to `core_operation_policy`.

Mutation evidence belongs to `logs.diff`.

Output projection belongs to the decoder/collapse runtime.

A separate `constraint_rule` table is rejected because it would duplicate the current core and introduce a second rule authority.

---

# 16. Runtime Pipeline

Canonical synchronous path:

```text
input
→ core_state read
→ scope split
→ token candidates
→ vocabulary candidates
→ grammar candidates
→ xi coherence lookup
→ logs.coherence append
→ no-hit / low-hit residual
→ yj decoherence_bank upsert
→ online/offline branch
→ grammar_relation lookup/upsert
→ zk coherence decoder
→ decoder/collapse invariant check
→ collapse/output
→ mutation-capable updates only through core_can_execute()
```

Scheduled path:

```text
logs.coherence
→ scheduled aggregation
→ logs.current upsert
→ end-of-sentence flag check
→ promotion candidate detection
→ Phase relation candidate generation
→ core_notify_queue
```

External / remote path:

```text
external / remote event
→ backend transport / cryptographic verification if needed
→ DB ingest
→ core_state / policy gate
→ reject / quarantine / mastication queue
→ same core observation pipeline
```

---

# 17. Legacy Archive Policy

Legacy files remain in the repository as historical references.

They are not deleted.

They no longer define canonical runtime authority after the new core becomes canonical.

Each legacy element may be:

- migrated
- reinterpreted
- archived
- explicitly rejected

`wave_geometry` is archived as a canonical authority.

Migrated from `wave_geometry`:

- observation-derived meaning
- explicit collapse
- non-destructive accumulation
- decoder does not originate meaning
- local semantic sovereignty

Rejected from canonical authority:

- wave state as primary semantic state
- spectral matrix as meaning authority
- `psi` / STFT / spectral basis as canonical core
- wave-signature search as required semantic retrieval

---

# 18. Short Form

```text
Meaning is not a word.
Meaning is not a vector.
Meaning is not wave_geometry.
Meaning is not a local mirror of input.

Meaning is the connection form that survives:
observation,
coherence logging,
decoherence banking,
current aggregation,
relation formation,
Phase candidate generation,
operation-gated adoption,
corrected grammar decoding,
and explicit collapse.

Constraint tables are not canonical.
Decoder/collapse invariants are absorbed into the current core.
```
