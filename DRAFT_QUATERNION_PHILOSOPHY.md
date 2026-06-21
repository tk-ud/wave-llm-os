# Draft Philosophy: Quaternion Exploration Core

## Status

This document is a draft philosophy for the next conceptual core of Wave LLMOS.

The existing spectral-wave architecture is preserved as legacy. It remains historically important because it established the original premise:

- meaning does not originate from fixed neural weights
- meaning is generated from observation
- semantic state should be persistent, inspectable, and expandable
- the decoder must not be treated as the origin of meaning
- collapse must be explicit and bounded

However, the legacy design treated semantic formation too much as a single-space spectral process.

The next core redefines the system around a quaternion-style processing equation, grammar space, vocabulary space, candidate bundles, residuals, a decoherence bank, and explicit adoption.

---

# 1. Core Premise

Meaning does not exist inside a word alone.

Meaning emerges from the connection form between elements.

A word, token, character, table value, status, phrase, or event is not meaningful by itself.

Meaning appears when those elements are connected through grammar, usage, recurrence, pressure, residuals, and adoption.

Therefore, the system must not begin by assuming a fixed semantic vector space.

It must observe input, compare it against existing grammar and vocabulary spaces, preserve multiple near-neighbor candidates, and only then generate draft or adopted meaning structures.

---

# 2. Legacy Principle Preserved

The legacy Wave LLMOS principle remains valid:

> Meaning originates from observation, not from a predefined vector space.

The new core preserves this principle.

However, the mechanism changes.

Legacy core:

```text
token
→ phase hash
→ wave state update
→ spectral observation
→ basis search
→ decoder
```

Next core:

```text
input
→ coherence layer
→ decoherence bank
→ coherence decoder
→ output candidate state
```

The system is no longer primarily a continuously integrated spectral matrix or vector state.

It becomes a layered quaternion-style exploration system over grammar, vocabulary, hash, residual, draft, and adopted spaces.

---

# 3. Why the Legacy Core Becomes Insufficient

The legacy design had a vocabulary-like wave space, but it did not sufficiently define grammar space.

It could represent observed patterns, spectral signatures, and basis formation.

But it did not clearly define:

- how grammar candidates are formed
- how vocabulary candidates and grammar candidates interact
- how incomplete grammar matches should trigger vocabulary exploration
- how residuals become draft vocabulary
- how draft grammar becomes adopted grammar
- how long context can be absorbed without continuous full-sequence computation
- how a decoder can be generated from coherence and decoherence rather than pre-attached as a separate black box

Without explicit grammar space, the system risks becoming another single-space similarity engine.

That would repeat the same structural problem found in large attention systems:

- input is pulled into one representational space
- candidates are collapsed too early
- weak but important alternatives are lost
- long context becomes computationally expensive
- meaning is compressed rather than distributed into inspectable structures

The next core exists to avoid that.

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

x = token bundle / hash bundle / segmented input fragments
i = regular grammar candidate, regular vocabulary candidate, or draft vocabulary candidate

xi = coherence layer
   = search whether x can be absorbed by i


y = unsearchable difference / no-hit residual / low-hit residual
j = draft grammar candidate, including draft grammar and draft vocabulary accumulation context

yj = decoherence bank
   = store y into j-side draft accumulation


z = exploration hit bundle
k = corrected grammar candidate

zk = coherence decoder
   = project z through k into an output-capable grammar candidate
```

This is quaternion-style, not a strict Hamilton quaternion algebra at the draft stage.

The symbols `i`, `j`, and `k` are candidate imaginary axes.

They are not assumed to be fixed orthogonal mathematical basis vectors.

They are candidate-bearing virtual axes that appear through the exploration process.

The important algebraic property is procedural non-commutativity.

Search order matters.

```text
ij ≠ ji
```

In this model, the following processing relation is central:

```text
ij = k
```

This means:

```text
coherence candidate axis
composed with
decoherence bank candidate axis
produces
coherence decoder axis
```

Or, operationally:

```text
xi finds what can be absorbed.
yj stores what cannot be absorbed.
The composition of i and j produces k.
zk uses k to decode the surviving hit bundle into corrected grammar.
```

The system does not start from the question:

```text
Which token is closest?
```

It starts from:

```text
Which coherence/decoherence path can produce a corrected grammar candidate for output?
```

The database stores the spaces.

The quaternion-style equation defines the exploration and decoding process.

The runtime evaluates candidate paths.

Projection turns a corrected grammar candidate into usable output.

Collapse selects or emits an output.

Adoption updates persistent spaces only through explicit rules.

---

# 5. Layered Semantic Spaces

The next core separates semantic space into layered spaces.

Initial candidate stack:

```text
input grammar space
hash space
adopted grammar space
adopted vocabulary space
draft grammar space
draft vocabulary space
decoherence bank
residual space
```

These spaces are not fixed ontologies.

They are observed, updated, and separated by usage, pressure, recurrence, and explicit adoption.

## 5.1 Input Grammar Space

The input grammar space represents the observed structural form of the current input.

It does not assume that the input has already been tokenized into meaningful words.

It observes sequence, boundaries, repeated forms, separators, punctuation, line breaks, and local connection patterns.

## 5.2 Hash Space

Hash space is a language-independent indexing layer.

It is not a semantic embedding model.

It provides stable keys for observed character sequences, chunks, residuals, and candidate bundles.

Characters may receive stable UUID or index identities, but the system must not aggregate meaning one character at a time.

Character-level identity is only an atomic reference layer.

Meaning aggregation occurs at chunk, candidate, residual, grammar, vocabulary bundle, and decoherence-bank levels.

## 5.3 Adopted Grammar Space

Adopted grammar space contains grammar bundles that have been repeatedly observed, reinforced, or explicitly adopted.

These bundles represent stable connection forms.

They are not universal grammar rules.

They are locally adopted semantic-grammar structures.

## 5.4 Adopted Vocabulary Space

Adopted vocabulary space contains vocabulary items that have stabilized through observation, usage, recurrence, and adoption.

A vocabulary item may be a word, phrase, business term, code, label, status, category, or domain-specific expression.

## 5.5 Draft Grammar Space

Draft grammar space contains grammar candidates that have not yet stabilized.

A draft grammar candidate may emerge from residuals, incomplete matches, repeated unknown structures, or near-neighbor mismatches.

Draft grammar must not automatically become adopted grammar.

## 5.6 Draft Vocabulary Space

Draft vocabulary space contains residual token hashes, repeated chunks, candidate words, or unknown expressions that have not yet stabilized.

Draft vocabulary is the system's holding area for unresolved expression.

## 5.7 Decoherence Bank

The decoherence bank stores no-hit, low-hit, and unabsorbed residuals.

It is not an error table.

It is a structured bank of future vocabulary and grammar candidates.

Repeated entries may become draft vocabulary or draft grammar.

Recurring draft entries may later be promoted into adopted vocabulary or adopted grammar.

---

# 6. xi: Coherence Layer

The `xi` term is the coherence layer.

```text
x = token bundle / hash bundle / segmented input fragments
i = regular grammar candidate, regular vocabulary candidate, or draft vocabulary candidate
```

The coherence layer searches whether the input fragments can be absorbed by existing spaces.

It checks:

- adopted grammar candidates
- adopted vocabulary candidates
- existing draft vocabulary candidates
- near-neighbor candidates
- partial matches
- incomplete grammar candidates

If `x` hits or nearly hits `i`, the system records coherence evidence.

The purpose of this layer is not to produce final output.

The purpose is to identify which parts of the input already belong to known or draft semantic space.

Operationally:

```text
input fragments
→ adopted grammar / adopted vocabulary / draft vocabulary candidates
→ hit, near-hit, partial-hit, or no-hit
```

The result of this layer is an exploration hit bundle plus any unabsorbed differences.

---

# 7. yj: Decoherence Bank

The `yj` term is the decoherence bank.

```text
y = unsearchable difference / no-hit residual / low-hit residual
j = draft grammar candidate and draft accumulation context
```

The decoherence bank stores what the coherence layer could not absorb.

This includes:

- new words
- unknown expressions
- no-hit token hashes
- low-hit token hashes
- unresolved grammar fragments
- unmatched grammar slots
- repeated but not-yet-adopted patterns

The bank is intentionally persistent.

Unabsorbed differences must not be discarded.

They are stored as draft vocabulary or draft grammar candidates.

A scheduled aggregation job may later inspect the bank.

Example promotion logic:

```sql
select token_hash, count(*) as observed_count
from decoherence_bank
group by token_hash
having count(*) > 10;
```

When recurrence, count, or coherence exceeds a promotion threshold, a draft may become an adopted vocabulary or adopted grammar candidate.

Operationally:

```text
no-hit / low-hit residual
→ decoherence bank
→ scheduled aggregation
→ promotion candidate
→ adopted vocabulary or adopted grammar
```

This is how unknown input can become known input later.

Yesterday's `yj` candidate may become tomorrow's `xi` hit.

---

# 8. ij = k: Procedural Composition

In this draft, `ij = k` is a processing relation.

It means that the composition of the coherence candidate axis and the decoherence-bank candidate axis produces the coherence decoder axis.

```text
i = coherence candidate axis
j = decoherence-bank candidate axis
k = coherence decoder axis
```

Operationally:

```text
coherence layer
+ decoherence bank
→ corrected grammar axis
```

Or:

```text
known hits
+ stored unknown residuals
→ output-capable corrected grammar candidate
```

This is not merely symbolic decoration.

The system cannot output from `xi` alone.

The coherence layer only tells what was hit or nearly hit.

The system cannot output from `yj` alone.

The decoherence bank only stores what could not be absorbed.

The system needs `zk` because final output requires a corrected grammar candidate.

Thus:

```text
ij = k
zk = coherence decoder
```

The order matters.

```text
ij = k
ji ≠ k
```

Searching known candidates first and then banking residuals does not produce the same result as starting from residuals first.

This is the procedural meaning of non-commutativity in this model.

---

# 9. zk: Coherence Decoder

The `zk` term is the coherence decoder.

```text
z = exploration hit bundle
k = corrected grammar candidate
```

The coherence decoder projects surviving hit bundles into output-capable corrected grammar.

It uses:

- hit bundles from the coherence layer
- unresolved slots from incomplete grammar candidates
- promoted candidates from the decoherence bank
- adopted grammar candidates
- adopted vocabulary candidates
- draft candidates that remain relevant

The decoder does not originate meaning.

Meaning candidates are generated by exploration, hit evidence, residual banking, recurrence, and adoption.

The coherence decoder projects those candidates into a usable output grammar.

Operationally:

```text
exploration hit bundle
→ corrected grammar candidate
→ output projection
→ collapse
→ emitted output
```

This decoder is not a next-token predictor.

It is a grammar projection layer over surviving coherence and decoherence evidence.

---

# 10. Candidate Bundles Must Not Collapse Early

Near-neighbor search must always preserve multiple candidates.

The system must not select only one nearest grammar or vocabulary item too early.

Instead, it should operate on bundles:

```text
grammar_near_candidates = {g1, g2, g3, ...}
vocabulary_near_candidates = {v1, v2, v3, ...}
grammar_residual_candidates = {Δg1, Δg2, Δg3, ...}
residual_candidates = {r1, r2, r3, ...}
```

This is essential.

If only one candidate is selected, meaning correction becomes impossible.

If multiple incomplete grammar candidates are preserved, the unmatched slots become search openings.

For example:

```text
"pork meat that has been grilled"
```

may partially match grammar candidates such as:

```text
{ingredient} + {cooking method} + {dish}
{animal/meat} + {heating method} + {food}
{material} + {process} + {result}
```

The unmatched vocabulary slots can then search vocabulary space and produce candidates such as:

```text
pork steak
grilled meat
char siu
roast pork
```

Meaning correction does not require a separate magical semantic table.

It can emerge from incomplete grammar candidates, unresolved vocabulary slots, and the coherence decoder.

---

# 11. Long Context Handling

The system does not need to hold long context as one continuous internal state.

It can absorb long input by distributing it into grammar candidates, vocabulary candidates, decoherence-bank entries, residuals, and aggregate differences.

This is not summarization.

It is semantic distribution.

Known parts of a long input reinforce existing grammar and vocabulary spaces through the coherence layer.

Unknown parts are stored in the decoherence bank.

Repeated residuals increase pressure.

Stable residuals may become draft vocabulary or draft grammar.

Adopted structures remain inspectable.

Therefore, long context can be ingested without compressing all content into a single hidden vector state.

The system stores structured semantic traces instead of relying on one opaque context window.

---

# 12. Summarization as Grammar-Space Projection

Summarization is not merely deletion.

Summarization is projection into a selected grammar space.

When summarization is requested, the system should:

```text
observe vocabulary pressure
→ identify strong vocabulary bundles
→ search grammar space from those vocabulary pressures
→ produce multiple summary grammar candidates
→ select compression level from candidate grammar forms
→ decode through corrected grammar
→ generate output
```

Compression level is not only a token budget.

It is a choice of grammar projection.

Examples:

```text
headline summary
argument summary
technical summary
business summary
evidence-preserving summary
video-script summary
```

Each summary type is a different grammar-space projection.

---

# 13. Tokenization Philosophy

The system should not depend on language-specific morphological analyzers.

Tokenization should not decide meaning at the entrance.

Instead:

- characters receive stable identities
- boundaries are detected through separators, spaces, punctuation, line breaks, code patterns, and repeated structure
- chunks become hash candidates
- hash candidates are compared against grammar and vocabulary spaces
- only residual candidates are inserted or reinforced

A tokenizer should act as:

```text
UUID indexer
boundary splitter
hash aggregator
```

not as a semantic authority.

One-character UUID assignment is an identity layer, not a meaning aggregation layer.

The system must not aggregate meaning one character at a time.

---

# 14. Event-Driven Runtime

The core runtime should be mostly database-driven.

Special programming should be minimal.

Primary operations:

```text
key assignment
neighbor lookup
aggregate diff
candidate bundle update
residual upsert
decoherence bank insert/update
pressure calculation
scheduled aggregation
notify on hit / no-hit / threshold crossing
```

The system may notify when:

- a known vocabulary space is hit
- no matching vocabulary candidate exists
- no matching grammar candidate exists
- residual pressure exceeds a threshold
- draft grammar repeatedly appears
- draft vocabulary repeatedly appears
- adopted grammar coherence increases
- a summary projection candidate is available
- decoherence-bank entries exceed promotion thresholds

The system should not automatically rewrite adopted meaning.

It should recommend, notify, and preserve evidence.

---

# 15. Relationship to Transformer Attention

This architecture is not Transformer Attention.

Transformer Attention computes weighted relationships inside a learned representation system and typically collapses candidates through softmax-like selection pressure.

The next core preserves candidate bundles and avoids early collapse.

It does not attempt to scale meaning through larger parameter matrices.

It attempts to reduce computation by using:

- coherence-layer hits
- decoherence-bank accumulation
- coherence-decoder projection
- grammar-space near candidates
- vocabulary-space near candidates
- incomplete grammar slot search
- aggregate differences
- residual-only updates
- event-driven notifications
- explicit adoption

In this model, attention and decoder are not fully separate black boxes.

The quaternion-style process attends through `xi`, banks unresolved difference through `yj`, and decodes through `zk`.

The goal is not larger context windows.

The goal is structured, inspectable, non-destructive semantic accumulation.

---

# 16. Relationship to Legacy Wave LLMOS

Legacy Wave LLMOS remains valuable as the origin of the project.

It established the rejection of fixed semantic vector space as primary authority.

It established observation-derived meaning.

It established that the decoder must not originate meaning.

However, the next core replaces the legacy spectral-wave mechanism with a layered quaternion-style grammar-vocabulary exploration model.

Legacy terms such as wave, coherence, decoherence, and collapse may remain as conceptual vocabulary.

But the implementation must not depend on a single Euler-integrated wave matrix.

The new core is not a continuous spectral matrix update.

It is a layered quaternion-style coherence/decoherence/decoder system.

Short form:

```text
q = w + xi + yj + zk

w  = input anchor
xi = coherence layer
yj = decoherence bank
zk = coherence decoder

ij = k
ji ≠ k
```

Meaning is not a word.

Meaning is not a vector.

Meaning is the connection form that survives observation, coherence search, decoherence banking, corrected grammar decoding, and adoption.
