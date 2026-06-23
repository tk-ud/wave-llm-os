# Supporting Note: Index Array Implementation Notes

Canonical authority: `SPEC_REFERENCE_MODEL.md`.

This file is not a source of truth.

It only exists as a short implementation reminder for SQL authors.

If this file conflicts with a SPEC file, the SPEC file wins.

---

# Implementation Reminder

Use generated bigint indexes for semantic references.

Use `bigint[]` only when storing ordered semantic structure.

Keep UUIDs as internal row or event identity.

Do not introduce semantic `uuid[]` columns in SQL projections.
