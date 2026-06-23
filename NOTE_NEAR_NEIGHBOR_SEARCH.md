# Supporting Note: Structural Near-Neighbor Implementation

Canonical authority: `SPEC_SEARCH_AND_VERIFICATION.md`.

This file is not a source of truth.

It only preserves implementation guidance for retrieval support.

If this file conflicts with a SPEC file, the SPEC file wins.

---

# Implementation Boundary

Near-neighbor retrieval may help find candidate structures.

Structural verification remains defined by `SPEC_SEARCH_AND_VERIFICATION.md`.

Vector distance is retrieval support only.

SQL join-diff is a useful implementation method for inspecting shared, missing, excess, or residual structure.

---

# Implementation Sketch

```text
candidate retrieval
-> SQL structural diff
-> evidence row
-> routed SPEC decision
```

Missing-slot completion should descend from higher-level semantic context when available:

```text
grammar_array
-> grammar_index search
-> vocabulary_index search
-> token_index search
```

Token-level descent is an implementation fallback.
