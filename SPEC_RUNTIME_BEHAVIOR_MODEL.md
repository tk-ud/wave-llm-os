# Specification: Runtime Behavior Model

This file is the canonical authority for abstract runtime behavior names.

PostgreSQL tables, rows, indexes, arrays, logs, and archives are concrete inspectable projections.

They are not the behavior itself.

Read direction:

```text
abstract runtime behavior
-> concrete inspectable PostgreSQL projection
```

---

# Behavior Names

The following terms are runtime behavior names, not poetic labels:

```text
wave
coherence
decoherence
pressure
Phase
Sleep
collapse
```

---

# wave

`wave` is the moving semantic field formed by observation, vocabulary, grammar, relation, residual, and pressure.

It is not a table.

Tables project inspectable parts of the wave.

---

# coherence

`coherence` is verified semantic continuity.

It may promote or reinforce structure only through operation-gated evidence.

Projection examples:

```text
grammar_relation
logs.coherence
logs.diff promotion evidence
```

---

# decoherence

`decoherence` is unresolved, unstable, unused, moth-eaten, mirror-output, or residual evidence preserved for fallback search.

It is not trash.

Projection examples:

```text
decoherence_bank
logs.coherence residual evidence
sleep.decohere_structure
```

---

# pressure

`pressure` is aggregate evidence that changes future search, Phase generation, Sleep consolidation, or operator attention.

Projection examples:

```text
aggregate.current
logs.current history
phase_score
draft_antipattern_score
moth_eaten_score
```

---

# Phase

`Phase` is scheduled aggregate-weighted relation candidate generation.

It is not the synchronous reply path.

Projection examples:

```text
phase_relation_candidate
aggregate.current pressure
SPEC_CRON_PIPELINE.md
```

---

# Sleep

`Sleep` is scheduled consolidation, decoherence routing, and cleanup of unstable semantic structures.

Sleep never hard-deletes semantic structures.

Projection examples:

```text
sleep_consolidation
sleep.decohere_structure
decoherence_bank
```

---

# collapse

`collapse` is reply output emission after verification and decoder projection.

Collapse is not promotion.

Promotion is operation-gated mutation.

Projection examples:

```text
SPEC_REPLY_PIPELINE.md
logs.coherence output evidence
logs.diff only when mutation occurs
```

---

# Projection Rule

Do not read database tables as semantic authority by themselves.

Do not reify a behavior name as a table unless a routed SPEC explicitly defines that projection.

When behavior and table naming conflict, the routed SPEC wins.
