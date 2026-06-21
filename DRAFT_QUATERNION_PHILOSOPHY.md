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

The next core redefines the system around quaternion exploration, grammar space, vocabulary space, candidate bundles, residuals, and explicit adoption.

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
→ quaternion exploration
→ grammar-neighbor candidate bundle
→ unmatched grammar slots
→ vocabulary-neighbor candidate bundle
→ residual token hash
→ coherence / decoherence separation
→ draft or adopted grammar-vocabulary spaces
```

The system is no longer primarily a continuously integrated spectral matrix or vector state.

It becomes a layered quaternion exploration system over grammar, vocabulary, hash, residual, draft, and adopted spaces.

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

Without explicit grammar space, the system risks becoming another single-space similarity engine.

That would repeat the same structural problem found in large attention systems:

- input is pulled into one representational space
- candidates are collapsed too early
- weak but important alternatives are lost
- long context becomes computationally expensive
- meaning is compressed rather than distributed into inspectable structures

The next core exists to avoid that.

---

# 4. Quaternion Is the Model

Quaternion is not an optional representation layer.

Quaternion is the model.

More precisely:

> The exploration method itself is quaternion.

The grammar space, vocabulary space, draft space, adopted space, hash space, and residual space are not the model by themselves.

They are structured semantic spaces referenced and traversed by the model.

The model is the quaternion exploration relation that binds these spaces into candidate search paths.

Conceptually:

```text
q = S + (I1 · R1) + (I2 · R2) + (I3 · R3)
```

Where:

```text
S  = scalar input anchor

I1 = hash / draft-vocabulary exploration direction
R1 = hash space and draft vocabulary space

I2 = adopted-grammar / adopted-vocabulary exploration direction
R2 = adopted grammar space and adopted vocabulary space

I3 = residual / decoherence exploration direction
R3 = draft grammar space and unresolved residual space
```

The real-side spaces store observable candidates, evidence, pressure, adoption state, and residuals.

The imaginary-side axes define directional search relations.

The quaternion model does not merely store a state.

It defines how input anchors rotate through grammar, vocabulary, hash, residual, draft, and adopted spaces without collapsing candidate bundles too early.

The system does not ask:

```text
Which token is closest?
```

It asks:

```text
Which grammar-vocabulary-residual exploration path remains coherent under the current input anchor?
```

This is the core distinction.

The database stores the spaces.

The quaternion defines the exploration model.

The runtime evaluates candidate quaternion paths.

Projection turns a quaternion exploration state into a usable output candidate.

Collapse selects or emits an output.

Adoption updates persistent spaces only through explicit rules.

---

# 5. Layered Semantic Space

The next core separates semantic space into layered spaces.

Initial candidate stack:

```text
input grammar space
hash space
adopted grammar space
adopted vocabulary space
draft grammar space
draft vocabulary space
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

Meaning aggregation occurs at chunk, candidate, residual, grammar, and vocabulary bundle levels.

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

## 5.7 Residual Space

Residual space contains what existing grammar and vocabulary spaces could not explain.

Residuals are not errors.

Residuals are future meaning candidates.

---

# 6. Candidate Bundles Must Not Collapse Early

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

It can emerge from incomplete grammar candidates and unresolved vocabulary slots.

---

# 7. Meaning Correction Through Incomplete Grammar

Meaning correction is not a separate post-processing trick.

It is a natural result of quaternion exploration when incomplete grammar candidates remain alive.

The flow is:

```text
input
→ grammar-neighbor candidate bundle
→ matched grammar slots / unmatched grammar slots
→ vocabulary search from unmatched slots
→ candidate vocabulary completion bundle
→ coherence comparison
→ output candidate or residual preservation
```

The important part is that incomplete grammar candidates are not discarded.

A fully matched grammar candidate may produce a direct interpretation.

A partially matched grammar candidate may produce a better semantic search path.

This allows the system to connect descriptive phrases to candidate vocabulary items even when surface strings do not match.

Example:

```text
Input:
"pig meat grilled into a dish"

Incomplete grammar candidates:
{ingredient} + {cooking method} + {dish name}
{meat} + {heat process} + {food result}

Unmatched slot:
{dish name}

Vocabulary candidates:
pork steak
pork saute
grilled pork
char siu
roast pork
```

The system does not need to know the final answer at the input boundary.

The grammar candidate opens the route.

The vocabulary space fills the unresolved slot.

The quaternion exploration path evaluates coherence.

---

# 8. Residual-Driven Learning

The system should not learn everything.

It should first subtract what existing spaces can already explain.

The basic flow is:

```text
input
- grammar-space near candidates, including draft grammar
= grammar residual candidate bundle

grammar residual candidate bundle
- vocabulary-space near candidates, including draft vocabulary
= residual candidate bundle

residual candidate bundle
→ token_hash insert on conflict
```

Only residuals should create pressure for new draft vocabulary or draft grammar.

This prevents the system from exploding computationally.

Known structures are absorbed by adopted or draft spaces.

Only unexplained differences are accumulated.

---

# 9. Long Context Handling

The system does not need to hold long context as one continuous internal state.

It can absorb long input by distributing it into grammar candidates, vocabulary candidates, residuals, and aggregate differences.

This is not summarization.

It is semantic distribution.

Known parts of a long input reinforce existing grammar and vocabulary spaces.

Unknown parts become residual candidates.

Repeated residuals increase pressure.

Stable residuals may become draft vocabulary or draft grammar.

Adopted structures remain inspectable.

Therefore, long context can be ingested without compressing all content into a single hidden vector state.

The system stores structured semantic traces instead of relying on one opaque context window.

---

# 10. Summarization as Grammar-Space Projection

Summarization is not merely deletion.

Summarization is projection into a selected grammar space.

When summarization is requested, the system should:

```text
observe vocabulary pressure
→ identify strong vocabulary bundles
→ search grammar space from those vocabulary pressures
→ produce multiple summary grammar candidates
→ select compression level from candidate grammar forms
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

# 11. Tokenization Philosophy

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

# 12. Event-Driven Runtime

The core runtime should be mostly database-driven.

Special programming should be minimal.

Primary operations:

```text
key assignment
neighbor lookup
aggregate diff
candidate bundle update
residual upsert
pressure calculation
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

The system should not automatically rewrite adopted meaning.

It should recommend, notify, and preserve evidence.

---

# 13. Coherence and Decoherence

Coherence means that an input or residual candidate repeatedly aligns with adopted or stable draft spaces.

Decoherence means that an input or residual candidate cannot be sufficiently absorbed by existing grammar or vocabulary spaces.

Coherence reinforces.

Decoherence drafts.

Neither should automatically overwrite adopted meaning.

The system should preserve:

- adopted grammar
- adopted vocabulary
- draft grammar
- draft vocabulary
- residual token hashes
- candidate evidence
- lineage

Meaning should grow through observable pressure and explicit adoption, not silent mutation.

---

# 14. Human Tool Principle

The system is a tool for humans.

It must not become the decision authority.

It may observe.

It may recommend.

It may preserve evidence.

It may show candidate grammar and vocabulary structures.

But adoption remains explicit.

The goal is not to replace human judgment.

The goal is to reduce invisible cognitive work by surfacing useful grammar, vocabulary, summary, search, and evidence structures from existing input.

---

# 15. Relationship to Transformer Attention

This architecture is not Transformer Attention.

Transformer Attention computes weighted relationships inside a learned representation system and typically collapses candidates through softmax-like selection pressure.

The next core preserves candidate bundles and avoids early collapse.

It does not attempt to scale meaning through larger parameter matrices.

It attempts to reduce computation by using:

- quaternion exploration paths
- grammar-space near candidates
- vocabulary-space near candidates
- incomplete grammar slot search
- aggregate differences
- residual-only updates
- event-driven notifications
- explicit adoption

The goal is not larger context windows.

The goal is structured, inspectable, non-destructive semantic accumulation.

---

# 16. Relationship to Legacy Wave LLMOS

Legacy Wave LLMOS remains valuable as the origin of the project.

It established the rejection of fixed semantic vector space as primary authority.

It established observation-derived meaning.

It established that the decoder must not originate meaning.

However, the next core replaces the legacy spectral-wave mechanism with a layered quaternion grammar-vocabulary exploration model.

Legacy terms such as wave, coherence, decoherence, and collapse may remain as conceptual vocabulary.

But the implementation must not depend on a single Euler-integrated wave matrix.

The new core is not a continuous spectral matrix update.

It is a layered quaternion exploration and residual-difference semantic system.

Short form:

```text
Do not compress meaning into weights.
Do not compress long context into one hidden state.
Do not collapse candidates too early.
Do not let the tokenizer decide meaning.

Observe input.
Explore through quaternion paths.
Search grammar candidates.
Preserve incomplete grammar slots.
Search vocabulary candidates.
Preserve residuals.
Separate coherence and decoherence.
Draft before adoption.
Notify before mutation.
Let humans adopt meaning.
```

Meaning is not a word.

Meaning is not a vector.

Meaning is the connection form that survives observation, residual pressure, candidate comparison, quaternion exploration, and adoption.
