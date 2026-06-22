# Legacy Design: Manifest / WaveCore TODO

> Legacy archive only.
> This document is not current implementation authority.
> Current authority: `../SPEC_CANONICAL_CORE.md`.

---

This file preserves the previous implementation roadmap as legacy context.

That legacy roadmap was based on:

```text
MANIFEST.yaml as law
WaveCore.psi as primary semantic state
SpectralObserver.band_features
MeaningSpace basis vectors
SQLite basis / pattern tables
constraint engine
sampling as only collapse point
```

These are no longer current implementation authority.

Current design has moved to PostgreSQL-backed connection structure:

```text
meaning = connection form
```

See:

```text
../SPEC_CANONICAL_CORE.md
../MIGRATION_LEGACY_REGISTER.md
```
