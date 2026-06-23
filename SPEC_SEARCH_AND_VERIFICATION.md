# Specification: Search and Verification

This file is the canonical authority for structural search and verification.

Search starts from semantic index arrays.

Near-neighbor search finds structurally close candidates.

Vector search may be used as an accelerator, but vector distance is not semantic authority.

Structural verification decides.

Active structures are searched first.

`decoherence_bank` is searched only when active structures do not hit, or when explicit operator / maintenance analysis requests it.

## Input Grammar / Grammar Relation Diff

Input grammar and candidate grammar_relation must always be diff-compared.

This is mandatory before output collapse, grammar_relation promotion, grammar_relation reinforcement, or decoherence hit promotion.

Near-neighbor similarity alone is not sufficient.

```text
input grammar candidate
<-> candidate grammar_relation.grammar_array
-> structural diff
-> coherence / residual / decoherence decision
```

The diff must inspect:

```text
shared grammar indexes
missing grammar slots
extra relation path elements
replaced vocabulary slots
replaced grammar slots
order shifts
scope boundary mismatch
terminal / sentence-end flag mismatch
```

Required decision pattern:

```text
near relation hit + input grammar diff passes
-> coherence relation hit
-> promote / reinforce grammar_relation

near relation hit + input grammar diff has missing slots
-> residual / decoherence evidence
-> possible question notification

near relation hit + repeated missing vocabulary or unstable slot
-> moth-eaten evidence
-> sleep may route unstable slot or attachment to decoherence_bank

active relation miss
-> decoherence_bank fallback search
-> input grammar / grammar_relation diff verification
-> promote.decoherence_hit if verified
```

Diff output must be logged as evidence.

## Corpus Output / Input Subtraction

Corpus-time output must subtract input.

Mastication, corpus processing, and candidate generation must not treat copied input as meaningful output.

```text
input grammar path
+ candidate output grammar path
-> subtract input grammar path
-> output_delta / relation_delta / residual_delta
```

If subtraction leaves no meaningful delta, the result is mirror output risk.

```text
candidate output - input grammar = empty or reorder-only delta
-> mirror_output evidence
-> residual / draft / decoherence evidence
```

Corpus output must be stored or scored as difference, relation, or residual, not as a raw echo of the input.

## Verification Boundary

Verification returns to canonical structures:

```text
index arrays
vocabulary paths
grammar paths
relation paths
input subtraction
output delta checks
```

A candidate that only mirrors or reorders input should be recorded as mirror-output evidence.

A candidate that passes structural verification may reinforce or promote only through the operation gate.

Fallback hits from `decoherence_bank` must be verified against current input grammar before reuse.
