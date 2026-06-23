# Supporting Note: Agent Abstract Runtime Mapping

Canonical authority: `SPEC_RUNTIME_BEHAVIOR_MODEL.md`.

This document is explanatory background only.

Runtime behavior authority lives in routed SPEC files.

In this document, "metaphor" means an abstract runtime behavior label, not poetic decoration.

---

# Purpose

Wave LLMOS uses terms such as wave, coherence, decoherence, pressure, Phase Attention, Sleep, and collapse.

Agents must not treat these terms as mystical, poetic, or physical-wave claims.

They are abstract runtime behavior names.

Concrete PostgreSQL structures are projections of those behaviors into inspectable state.

They are not semantic authority by themselves.

Agents must preserve this mapping direction:

```text
abstract runtime behavior
→ concrete inspectable PostgreSQL projection
```

Agents must not reverse it into:

```text
database table / row
→ semantic authority
```

This document maps those abstract names to inspectable database behavior, neural-network-like behavior, and agent interpretation rules.

---

# Core Interpretation

Wave LLMOS is not a rejection of neural-network-like structure.

It can be read as an open hierarchical neural-network-like system whose nodes, relations, parameters, failures, and corrections are projected into inspectable PostgreSQL data.

```text
NN-like physical structure
+ open node set
+ open parameter set
+ SQL-verifiable structural correction
= Wave LLMOS runtime
```

The database state makes the runtime behavior inspectable, reproducible, and operation-gated.

It does not replace the abstract behavior.

The current design rejects the legacy single-waveform / single-psi authority model.

It does not reject wave-like behavior.

Wave-like behavior is preserved as candidate movement through:

```text
coherence
decoherence
pressure
relation growth
fallback search
Sleep maintenance
Phase candidate generation
output collapse
operation-gated promotion
```

---

# Mapping Table

| Abstract Label / Metaphor | Runtime Meaning | Concrete Inspectable Projection | NN / Neural Analogy | Agent Rule |
|---|---|---|---|---|
| wave | Candidate movement through coherence, decoherence, pressure, and relation | candidate paths, residuals, logs, pressure | activation pattern over a network | Do not interpret as a physical wave equation; do not reduce to a table name |
| single waveform | Legacy single-state semantic authority | legacy psi / spectral basis | one global latent state | Treat as rejected legacy model |
| open wave field | Expandable candidate space | vocabulary, grammar, relation, decoherence_bank | growing neural graph | Nodes and parameters may increase |
| node | Addressable semantic unit in an open node set | token, vocabulary, grammar, grammar_relation | neuron / feature node | Node identity is an index, not a semantic UUID; node set is not closed |
| parameter | Inspectable state data in an open parameter set | observed_count, relation_weight, pressure, phase_score, logs | learned weight / activation history | Parameters are data projections, not hidden matrix weights |
| edge | Semantic continuity | grammar_relation, relation_array, phase_relation_candidate | connection weight / pathway | Relation is not optional metadata |
| activation | Candidate hit during Reply | near-neighbor hit, coherence hit | forward activation | Hit does not equal promotion |
| forward pass | Reply candidate generation | Core Reply Path | inference pass | Reply is synchronous core work |
| backprop-like correction | Structural diff feedback | missing slot, replaced slot, residual, pressure | error correction / backpropagation | No gradient; use SQL diff and logs |
| loss | Structural mismatch | mirror_output, contradiction, missing slots | prediction error | Store as evidence, not just failure |
| gradient | Direction of correction pressure | logs.current pressure, draft_antipattern_score | accumulated correction signal | Use pressure to guide Phase and Sleep |
| coherence | Successful absorption into known structure | structural verification pass, relation diff pass | stable activation / fit | Can promote only through operation gate |
| decoherence | Unabsorbed but preserved difference | decoherence_bank | residual error memory | Not trash; searchable fallback |
| pressure | Accumulated unresolved evidence | logs.current pressure | repeated error / activation pressure | Used by Sleep, Phase, and maintenance |
| draft | Unconfirmed candidate / anti-pattern evidence | draft_flag = true | weak activation / unstable candidate | Not a human review queue |
| promotion | Evidence-gated long-term structural update | promote.* operation, draft_flag=false | memory consolidation / weight update | Must pass `core_can_execute()` |
| sleep | Scheduled consolidation | sleep_consolidation | sleep replay / pruning | Must not block Reply |
| Phase Attention | Scheduled relation candidate generation | phase_relation_candidate | offline association growth | Not Transformer attention |
| zk decoder | Relation-guided output projection | decoder/collapse path | readout layer | Decoder does not originate meaning |
| collapse | Delayed fixation of surviving output candidate | output collapse | final output selection | Collapse is not semantic authority |
| mirror output | Input copied without meaningful delta | output_delta empty / reorder-only | overfit echo / no useful prediction | Must be logged as failure evidence |
| moth-eaten | Repeated missing/replaced slots | moth_eaten_score | degraded pathway | Sleep may decohere unstable structure |
| fallback | Retry from unresolved memory | decoherence_bank fallback search | recall from weak memory | Fallback can cohere again if verified |

---

# Runtime Reading

Agents should read Wave LLMOS runtime as a structural candidate system.

```text
retrieval = candidate discovery
verification = candidate survival test
diff = structural error detection
residual = preserved unresolved difference
pressure = accumulated correction signal
Phase = scheduled relation growth
Sleep = scheduled consolidation
promotion = operation-gated memory update
collapse = final reply fixation
```

Reply behavior should be interpreted as:

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

This means Reply is not simple similarity selection.

It is candidate retrieval followed by structural survival.

The concrete tables are the inspectable projection of this behavior.

They are not a replacement for the behavior.

---

# Backprop-Like Interpretation

Wave LLMOS correction can be compared to backpropagation only at the behavioral level.

Ordinary backpropagation:

```text
output error
→ propagate error across fixed layers
→ update hidden weights through gradient descent
```

Wave LLMOS structural correction:

```text
candidate / output structural mismatch
→ inspect grammar and relation diff
→ detect missing slots, replaced slots, order drift, mirror output, contradiction
→ store residual evidence
→ accumulate pressure
→ generate or suppress relation candidates
→ promote only after verification and operation gate
```

The correction target is not a hidden weight matrix.

The correction target is the inspectable projection of runtime state:

```text
node candidates
relation paths
residual bundles
pressure aggregates
draft flags
promotion logs
decoherence fallback entries
```

These projections are concrete storage surfaces for abstract correction behavior.

---

# Agent Reading Rules

1. Do not explain Wave LLMOS as merely “a wave model” without mapping the term to runtime behavior.

2. Do not say the design rejects neural networks. It externalizes neural-network-like hierarchy, parameters, activation history, and correction pressure into database structures.

3. Do not treat Phase Attention as synchronous reasoning. Phase is scheduled relation candidate generation.

4. Do not treat Draft as a review queue. Draft is unconfirmed candidate state and negative pressure evidence.

5. Do not treat decoherence as deletion. Decoherence is preserved fallback pressure.

6. Do not treat similarity as semantic truth. Similarity retrieves candidates; structural verification decides.

7. Do not treat output text as semantic authority. Meaning is the surviving relation, diff, evidence, and promotion path.

8. Do not treat the node set or parameter set as closed. They are open to observation-driven growth.

9. Do not treat the legacy single-waveform model as current authority. Current behavior is multi-layer candidate transition.

10. Do not invent a human approval workflow. Promotion is automatic and evidence-gated when policy allows it.

11. Do not reduce abstract behavior labels to implementation artifacts. Database rows make behavior inspectable; they do not replace the behavior.

12. Do not reverse the mapping direction. Read from abstract runtime behavior to concrete inspectable projection, not from table names to semantic authority.

13. Do not treat metaphor mapping as poetic translation. In this document, metaphor mapping is abstraction-to-concretion mapping.

14. Do not collapse Wave LLMOS into “just a database model.” PostgreSQL is the inspection and mutation surface for an open candidate runtime.

---

# Short Form

```text
Wave = candidate dynamics
Node = open semantic index row
Parameter = inspectable DB state
Forward = Reply candidate path
Backprop = structural diff correction
Loss = residual / mirror / contradiction evidence
Gradient = pressure
Learning = promotion / reinforcement / decoherence
Sleep = scheduled consolidation
Phase = scheduled relation growth
```

```text
Abstract behavior = runtime semantics
Concrete DB state = inspectable projection
Mapping direction = abstract → concrete
Agent failure = concrete → authority
```

Wave LLMOS should be read as:

```text
an open hierarchical neural-network-like system
whose nodes, parameters, failures, and corrections
are projected into inspectable PostgreSQL structures.
```
