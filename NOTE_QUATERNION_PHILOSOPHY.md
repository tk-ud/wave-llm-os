# Supporting Note: Quaternion Exploration Philosophy

Canonical authority: `SPEC_CANONICAL_CORE.md`.

---

# Premise

Meaning is not inside a word, vector, or wave state.

Meaning is the connection form that survives observation, coherence, decoherence, relation, aggregation, promotion, decoding, and collapse.

A word, token, character, table value, flag, phrase, event, or grammar fragment is not meaningful by itself.

Meaning appears when those elements are connected through grammar, vocabulary, usage, recurrence, pressure, residuals, relation, and operation-gated promotion.

---

# Rejected Legacy Authority

```text
wave_geometry as primary semantic authority
psi / spectral matrix as meaning authority
Euler wave state as semantic substrate
STFT / spectral basis as canonical meaning basis
wave_signature as required retrieval key
```

These remain historical references only.

The current core is not a continuous spectral matrix update.

It is a layered quaternion-style coherence / decoherence / relation / decoder system backed by PostgreSQL structures.

---

# Core Equation

```text
q = w + xi + yj + zk
```

```text
q  = output candidate state
w  = input anchor
xi = coherence layer
yj = decoherence bank
zk = coherence decoder
```

Expanded:

```text
x = token bundle / hash bundle / segmented input fragments
i = regular grammar candidate, regular vocabulary candidate, or draft vocabulary candidate

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

The important algebraic property is procedural non-commutativity.

```text
ij != ji
```

Order matters.

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

# Layered Semantic Spaces

The core separates semantic space into layered spaces.

```text
input grammar space
hash space
active grammar space
active vocabulary space
draft grammar space
draft vocabulary space
decoherence_bank
grammar_relation
phase_relation_candidate
residual space
```

These spaces are not fixed ontologies.

They are observed, updated, and separated by usage, pressure, recurrence, relation, `draft_flag`, and operation-gated promotion.

---

# Draft Flag and Promotion

`draft_flag` is the canonical lifecycle truth.

```text
draft_flag = true
→ draft / residual / unconfirmed / anti-pattern / fallback candidate

draft_flag = false
→ active / promoted / ordinary search target
```

No `status` enum table is required.

No `adopt.*` operation is canonical.

Promotion uses:

```text
promote.coherence_hit
promote.decoherence_hit
promote.phase_relation
```

Deletion uses explicit delete operations:

```text
delete.semantic_structure
delete.decoherence_entry
```

Manual/operator actions are processing classifications, not public UI design.

---

# Input Grammar Space

Input grammar space represents the observed structural form of the current input.

It observes sequence, boundaries, repeated forms, separators, punctuation, line breaks, and local connection patterns.

---

# Hash Space

Hash space is a language-independent indexing layer.

It is not a semantic embedding model.

It provides stable keys for observed character sequences, chunks, residuals, and candidate bundles.

Character-level identity is only an atomic reference layer.

Meaning aggregation occurs at chunk, candidate, residual, grammar, vocabulary bundle, decoherence-bank, and relation levels.

---

# Active Grammar / Vocabulary Space

Active grammar and vocabulary spaces contain structures where `draft_flag = false` and `deleted_flag = false`.

They represent ordinary search targets.

They are not immutable truth.

They can be reinforced, contradicted, routed to decoherence evidence, or explicitly deleted through operation-gated paths.

---

# Draft Grammar / Vocabulary Space

Draft grammar and vocabulary contain structures where `draft_flag = true`.

Draft is not a human-review queue.

Draft is not the primary promotion target.

Draft is an unconfirmed candidate filter and anti-pattern evidence surface.

Draft evidence may reduce Phase candidate score when fresh coherence evidence does not overcome the penalty.

---

# Decoherence Bank

The decoherence bank stores no-hit, low-hit, unabsorbed, unused, unstable, moth-eaten, mirror-output, or unresolved residual structures.

It is not an error table.

It is not an external trash bin.

It remains part of the core semantic search space.

It stores the three semantic layers together in JSONB:

```json
{
  "vocabulary": {},
  "grammar": {},
  "relation": {}
}
```

Active structures are searched first.

If active search misses, `decoherence_bank` may be searched as a fallback layer.

```text
active structure search
→ no hit
→ decoherence_bank fallback search
→ structural verification
→ input grammar / grammar_relation diff verification
→ xi coherence hit if verified
```

If fallback search verifies the candidate against the current input grammar, the result may be promoted or reinforced through the operation gate.

```text
decoherence_bank candidate
→ structural verification passes
→ input grammar / grammar_relation diff verification passes
→ xi coherence hit
→ core_can_execute('promote.decoherence_hit')
→ promote / reinforce
→ logs.diff
```

Repeated entries may become Draft evidence, but Draft is not the canonical promotion queue.

Sleep may send structures to `decoherence_bank`.

Sleep must not hard-delete them.

Hard deletion is explicit manual/operator action only.

---

# Grammar Relation

`grammar_relation` stores how grammar and vocabulary candidates connect.

It is semantic continuity memory.

Without relation, the system can identify local fragments but cannot connect the beginning and end of a divided input.

Relation is necessary, but relation alone is not sufficient to prevent mirror output.

Relation prevents local mirroring only when the current input grammar is diff-checked against candidate `grammar_relation`, and corpus / candidate output is treated as:

```text
meaningful output
= candidate output - input grammar
```

If the remaining delta is empty or reorder-only, the result is mirror_output evidence rather than relation evidence.

---

# Phase Relation Candidate Space

`phase_relation_candidate` stores relation candidates generated from normalized aggregate values.

Phase does not primarily inspect raw input.

It works from active vocabulary, active grammar, draft vocabulary, draft grammar, existing relation, hit counts, relation weights, and co-occurrence aggregates.

Phase-generated candidates are draft candidates.

They must not silently mutate active grammar or active relation.

---

# xi: Coherence Layer

The `xi` term is the coherence layer.

The coherence layer searches whether input fragments can be absorbed by existing spaces.

It checks:

```text
active grammar candidates
active vocabulary candidates
existing draft vocabulary candidates
near-neighbor candidates
partial matches
incomplete grammar candidates
```

If `x` hits or nearly hits `i`, the system records coherence evidence.

The result is an exploration hit bundle plus any unabsorbed differences.

---

# yj: Decoherence Bank

The `yj` term is the decoherence bank.

```text
y = unsearchable difference / no-hit residual / low-hit residual
j = draft grammar candidate and draft accumulation context
```

The decoherence bank stores what the coherence layer could not absorb.

This includes:

```text
new words
unknown expressions
no-hit token hashes
low-hit token hashes
unresolved grammar fragments
unmatched grammar slots
repeated but not-yet-promoted patterns
```

Unabsorbed differences must not be discarded.

They are stored as residual, draft, or decoherence-bank evidence.

Yesterday's `yj` candidate may become tomorrow's `xi` hit when fallback search verifies it against current input grammar.

No count-only promotion example is canonical.

Count creates pressure.

Pressure surfaces candidates.

Verification creates promotion evidence.

Operation gate permits mutation.

---

# ij = k: Procedural Composition

`ij = k` means that the composition of the coherence candidate axis and the decoherence-bank candidate axis produces the coherence decoder axis.

```text
i = coherence candidate axis
j = decoherence-bank candidate axis
k = coherence decoder axis
```

Operationally:

```text
known hits
+ stored unknown residuals
→ output-capable corrected grammar candidate
```

The order matters.

```text
ij = k
ji != k
```

Searching known candidates first and then banking residuals does not produce the same result as starting from residuals first.

---

# zk: Coherence Decoder

The `zk` term is the coherence decoder.

```text
z = exploration hit bundle, including relation evidence
k = corrected grammar candidate
```

The coherence decoder projects surviving hit bundles into output-capable corrected grammar.

It uses:

```text
hit bundles from the coherence layer
relation evidence from grammar_relation
unresolved slots from incomplete grammar candidates
promoted candidates from decoherence_bank
active grammar candidates
active vocabulary candidates
draft candidates that remain relevant
Phase-generated relation candidates when available
```

The decoder does not originate meaning.

It projects candidates into usable output grammar.

This decoder is not a next-token predictor.

It is a grammar projection layer over surviving coherence, decoherence, and relation evidence.

---

# Runtime Path

```text
input with source_kind
→ token candidates
→ vocabulary candidates
→ grammar candidates
→ coherence lookup
→ logs.coherence
→ decoherence_bank if unresolved
→ grammar_relation lookup/upsert
→ coherence decoder
→ decoder/collapse invariant check
→ collapse/output
→ mutation only through core_can_execute()
```

Scheduled Phase path:

```text
logs.current pressure
+ grammar_relation aggregates
+ decoherence_bank recurrence
→ Phase Attention
→ phase_relation_candidate.grammar_array
→ later promotion or decoder support
```

---

# Candidate Bundles Must Not Collapse Early

Near-neighbor search must preserve multiple candidates.

The system must not select only one nearest grammar or vocabulary item too early.

Instead, it should operate on bundles:

```text
grammar_near_candidates = {g1, g2, g3, ...}
vocabulary_near_candidates = {v1, v2, v3, ...}
grammar_residual_candidates = {Δg1, Δg2, Δg3, ...}
relation_candidates = {r1, r2, r3, ...}
residual_candidates = {d1, d2, d3, ...}
```

If only one candidate is selected, meaning correction becomes impossible.

---

# Long Context Handling

The system does not need to hold long context as one continuous internal state.

It can absorb long input by distributing it into:

```text
grammar candidates
vocabulary candidates
relation candidates
decoherence-bank entries
residuals
aggregate differences
```

This is not summarization.

It is semantic distribution.

Known parts reinforce existing grammar and vocabulary spaces through the coherence layer.

Unknown parts are stored in the decoherence bank.

Separated coherent parts are connected through grammar_relation.

Repeated relation patterns increase relation pressure.

Stable residuals may become draft vocabulary or draft grammar.

Stable relation candidates may become promoted relation or promoted grammar paths.

The system stores structured semantic traces instead of relying on one opaque context window.

---

# Summarization as Grammar-Space Projection

Summarization is not merely deletion.

Summarization is projection into a selected grammar space.

Compression level is not only a token budget.

It is a choice of grammar projection and relation depth.

---

# Tokenization Philosophy

The system should not depend on language-specific morphological analyzers.

Tokenization should not decide meaning at the entrance.

A tokenizer should act as:

```text
UUID / index assigner
boundary splitter
hash aggregator
```

not as semantic authority.

One-character UUID assignment is an identity layer, not a meaning aggregation layer.

The system must not aggregate meaning one character at a time.

---

# Event-Driven Runtime

The core runtime should be mostly database-driven.

Primary operations:

```text
key assignment
neighbor lookup
aggregate diff
candidate bundle update
residual upsert
decoherence bank insert/update
relation insert/update
pressure calculation
scheduled aggregation
Phase relation candidate generation
notify on hit / no-hit / threshold crossing
```

The system may notify when:

```text
a known vocabulary space is hit
no matching vocabulary candidate exists
no matching grammar candidate exists
residual pressure exceeds a threshold
draft grammar repeatedly appears
draft vocabulary repeatedly appears
active grammar coherence increases
relation pressure increases
Phase relation candidates exceed thresholds
a summary projection candidate is available
decoherence-bank entries exceed surfacing thresholds
```

The system should not silently rewrite active meaning.

It should preserve evidence, apply promotion through operation gate, and keep delete operations explicit.

---

# Relationship to Transformer Attention

This architecture is not Transformer Attention.

Transformer Attention computes weighted relationships inside a learned representation system and typically collapses candidates through softmax-like selection pressure.

The current core preserves candidate bundles and avoids early collapse.

It does not scale meaning through larger parameter matrices.

It reduces computation by using:

```text
coherence-layer hits
decoherence-bank accumulation
grammar_relation continuity
Phase relation candidate generation
coherence-decoder projection
grammar-space near candidates
vocabulary-space near candidates
incomplete grammar slot search
aggregate differences
residual-only updates
event-driven notifications
explicit promotion
```

Phase uses normalized aggregate values to grow relation candidates outside ordinary reply-time processing.

The goal is not larger context windows.

The goal is structured, inspectable, non-destructive semantic accumulation.

---

# Authority Order

```text
1. Canonical Core
2. Core State / Operation Gate
3. Persistent Semantic Tables
4. Coherence / Decoherence / Relation Runtime
5. Scheduled Aggregation / Phase Attention / Relation Candidate Generation
6. Coherence Decoder
7. Decoder / Collapse Invariants
8. External / Remote Observation Intake
9. Legacy Archive
```

No independent constraint table.

---

# Short Form

```text
Meaning is connection form.
The DB is semantic memory.
The decoder does not originate meaning.
Candidate bundles must not collapse early.
Long context is distributed into inspectable semantic traces.
Summarization is grammar-space projection.
Phase Attention grows relation paths.
Collapse is explicit.
Mutation is operation-gated.
draft_flag is lifecycle truth.
```
