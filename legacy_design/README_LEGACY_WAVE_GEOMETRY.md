# Legacy Design: Wave Geometry README

> Legacy archive only.
> This document is not current implementation authority.
> Current authority: `../SPEC_CANONICAL_CORE.md`.

---

This file preserves the previous README direction as legacy context.

That legacy design treated meaning as spectral band geometry and wave-state evolution.

It is no longer the active implementation authority.

Current design has moved to PostgreSQL-backed connection structure:

```text
meaning = connection form
```

See:

```text
../SPEC_CANONICAL_CORE.md
../MIGRATION_LEGACY_REGISTER.md
```

Legacy terms retained here for history:

```text
WaveCore.psi
spectral band geometry
MeaningSpace basis vectors
STFT / band_features
SQLite basis tables
sampling collapse
```
