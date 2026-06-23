# Specification: Prohibited Canonical Patterns

This file is the canonical authority for explicitly non-canonical tables, concepts, and implementation patterns.

The goal is to preserve negative rules from the former monolithic core.

Do not reintroduce these patterns through common database, workflow, review, queue, or decoder implementation assumptions.

---

# Prohibited Canonical Tables

No canonical `constraint_rule` table.

No canonical `adoption_audit` table.

No canonical web-result-only semantic table.

No canonical `decoder_trace` table.

No canonical `loop_guard` table.

No canonical `status` enum table.

---

# Status Rule

`draft_flag` is the canonical lifecycle truth.

`status` is not canonical truth.

If a display label is needed, derive it from flags and log evidence.

---

# Decoder / Constraint Rule

Decoder does not originate meaning.

Decoder/collapse invariants are handled by the current core and routed SPEC files.

Do not add an independent constraint table as semantic authority.

Do not add decoder trace or loop guard as canonical semantic state.

Looping is unresolved exploration pressure, not a separate error state.

---

# Web Result Rule

Web search results are normal input observations with `source_kind = 'web'`.

Do not create a web-result-only semantic table.

---

# Adoption Rule

No `adopt.*` operation keys are canonical.

Promotion is represented by `promote.*` operation keys.

Promotion evidence and mutation authority are defined by `SPEC_OPERATION_GATE.md`.

---

# Audit Rule

Promotion and deletion audit are views or projections over `logs.diff`.

Do not create `adoption_audit` as canonical authority.

---

# Short Form

```text
No constraint_rule.
No adoption_audit.
No web-result-only semantic table.
No decoder_trace.
No loop_guard.
No status enum authority.
No adopt.* operation keys.
```
