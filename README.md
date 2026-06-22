# Wave LLMOS

Wave LLMOS is defined by the PostgreSQL-backed canonical core specification.

```text
Implementation authority:
SPEC_CANONICAL_CORE.md
```

Legacy wave-geometry / psi / spectral-band design notes are archived under `legacy_design/` as historical context.

---

# Design Philosophy: Wave and Quaternion

Wave LLMOS treats meaning as something that moves, interferes, resonates, and sometimes collapses.

A conversation is not only a sequence of tokens.

It is a wave field formed by vocabulary, grammar, relation, residuals, and accumulated pressure.

When a new input arrives, Wave does not immediately jump to an answer.

It fixes an anchor, then sweeps the surrounding semantic field.

Vocabulary candidates appear.

Grammar candidates bend around them.

Relation paths connect distant fragments.

Residuals remain as unresolved difference.

Nearby waves may be absorbed into the same basin.

Distant waves remain separate.

Similar but structurally different waves become alias, branch, or parent-child candidates.

This is why the system is described as quaternion-like.

The expression is not a claim that the runtime literally computes physical quaternions.

It is a way to describe one output scope as a composition of multiple semantic axes:

```text
q = w + xi + yj + zk
```

```text
w  = input anchor
xi = vocabulary-side coherence search
yj = grammar-side residual and draft exploration
zk = relation-guided correction and decode
q  = output candidate state
```

In implementation, these steps are computed in order.

In one conversation scope, they form a single composed output state.

Wave therefore uses two complementary views:

```text
implementation view:
anchor-fixed phase sweep

conversation-scope view:
quaternion-like output composition
```

Sleep maintenance then reshapes the field.

It does not merely clean data.

It refines attractor basins.

It lets nearby waves be absorbed.

It leaves distant waves alone.

It keeps ambiguous waves as draft evidence instead of collapsing them too early.

The goal is not to make meaning mysterious.

The goal is to keep meaning inspectable while still allowing it to move.

---

# Document Agenda

Read documents by responsibility, not by file order.

The canonical runtime flow must be understood as one connected machine:

```text
observation
→ token / vocabulary / grammar mastication
→ coherence / decoherence
→ grammar_relation
→ logs.current
→ Phase Attention
→ phase_relation_candidate
→ promotion gate
→ decoder / collapse
```

Supporting notes may explain parts of this machine, but they do not replace the canonical flow.

## `SPEC_CANONICAL_CORE.md`

```text
Single implementation authority.
Defines canonical tables, reference rules, logs, operation gate, Phase, decoder/collapse boundary.
```

Use this file to answer:

```text
What is canonical?
Which tables exist?
Which references are allowed?
Which mutations require operation gate?
What wins on conflict?
```

Do not use this file as a complete algorithm implementation manual.

## `NOTE_SQL_TOKENIZATION_IMPLEMENTATION.md`

```text
PostgreSQL implementation sketch.
Shows table-shape examples and storage conventions.
```

Use this file to answer:

```text
How might the canonical tables be represented in SQL?
Which columns are expected in sketches?
How do input_observation, semantic arrays, logs, and Phase candidates map to storage?
```

Do not use this file as the source of truth when it conflicts with `SPEC_CANONICAL_CORE.md`.

## `NOTE_INDEX_ARRAY_CANONICAL.md`

```text
Reference rule note.
Explains why semantic references use bigint index arrays and not UUID arrays.
```

Use this file to answer:

```text
What is identity?
What is semantic reference?
Which arrays are canonical?
Why is uuid[] rejected for semantic paths?
```

## `NOTE_PHASE_RELATION_CANDIDATE.md`

```text
Phase Attention and phase_relation_candidate supporting specification.
Explains scheduled aggregate-weighted relation candidate generation.
```

Use this file to answer:

```text
What is Phase Attention?
What does Phase read?
What does Phase write?
How does logs.current pressure drive grammar_array expansion?
What is a phase_relation_candidate?
How are draft candidates promoted?
What is mirror output?
How does sleep maintenance refine attractor basins?
```

## `NOTE_NEAR_NEIGHBOR_SEARCH.md`

```text
Structural near-neighbor search note.
Explains search over index arrays, SQL join diff verification, and derived-vector acceleration boundaries.
```

Use this file to answer:

```text
How are similar grammar_array paths searched?
What does structural verification mean?
How can pgvector be used without becoming semantic authority?
How does SQL join diff expose shared structure and residuals?
How does repeated grammar return expand search?
```

## `NOTE_QUATERNION_PHILOSOPHY.md`

```text
Short philosophy note.
Explains the q = w + xi + yj + zk framing and why order matters.
```

Use this file to answer:

```text
What is the conceptual role of w, xi, yj, zk?
Why is meaning treated as connection form?
How does Phase Attention fit the exploration philosophy?
```

## `MIGRATION_LEGACY_REGISTER.md`

```text
Migration and rejection register.
Tracks which legacy concepts were migrated, reinterpreted, rejected, or absorbed.
```

Use this file to answer:

```text
What happened to legacy wave_geometry, psi, spectral matrix, loop guard, web search, remote sync, and adoption audit?
Which legacy terms survive under new canonical meanings?
```

## `legacy_design/`

```text
Historical archive.
Useful for tracing old design intent, not for overriding the canonical core.
```
