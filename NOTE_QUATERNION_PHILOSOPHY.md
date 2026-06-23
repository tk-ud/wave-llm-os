# Supporting Note: Quaternion Analogy

Canonical authority: routed SPEC files.

This file is explanatory background only.

It must not define lifecycle, promotion, fallback, scoring, scheduler, decoder, search, verification, or storage authority.

If this file conflicts with a SPEC file, the SPEC file wins.

---

# Purpose

This NOTE preserves the quaternion-style analogy formerly used to explain Wave LLMOS behavior.

The analogy is not strict Hamilton quaternion algebra.

It is a naming aid for thinking about ordered runtime stages.

---

# Analogy

```text
q = w + xi + yj + zk
```

```text
w  = input anchor
xi = coherence-side candidate handling
yj = decoherence-side residual handling
zk = decoder-side output projection
```

The useful intuition is procedural non-commutativity:

```text
ij != ji
```

Order matters in runtime processing.

---

# Boundary

This NOTE does not define the actual runtime path.

Read routed SPEC files for:

```text
reference model
semantic tables
reply pipeline
cron pipeline
search and verification
operation gate
scoring and thresholds
runtime behavior model
```

---

# Short Form

Quaternion language is a metaphor for ordered candidate handling.

SPEC files define the system.
