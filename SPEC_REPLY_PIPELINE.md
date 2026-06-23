# Specification: Reply Pipeline

This file is the canonical authority for reply-time processing.

Reply generation is synchronous core work.

It is not Phase Attention and must not wait for scheduled Phase jobs.

Canonical flow:

```text
input_observation
-> token / vocabulary / grammar candidates
-> near-neighbor retrieval
-> structural verification
-> input grammar / grammar_relation diff verification
-> xi coherence hit or yj residual bank
-> coherence relation lookup
-> zk coherence decoder
-> output collapse
-> logs.coherence / logs.diff evidence
```

Reply-time processing reads `aggregate.current` for current pressure.

`logs.current` is historical aggregate evidence, not the hot current surface.

## Reply-time promotion

A reply-time coherence hit must immediately promote a new structure or reinforce an existing active structure through the operation gate.

```text
near-neighbor candidate
-> structural verification passes
-> input grammar / grammar_relation diff verification passes
-> xi coherence hit
-> zk decoder usage
-> core_can_execute('promote.coherence_hit')
-> promote or reinforce immediately
-> logs.diff
```

This promotion is automatic and evidence-driven.

Human review is not part of ordinary reply-time promotion.

Freeze, policy failure, contradiction, or operation gate failure blocks promotion and leaves evidence as draft, residual, rejected, or quarantined state.

## Decoherence fallback

Primary reply-time search checks active coherent structures first.

If active near-neighbor search does not hit, core may search `decoherence_bank` as a fallback layer.

```text
active structure search
-> no hit
-> decoherence_bank fallback search
-> structural verification
-> input grammar / grammar_relation diff verification
-> xi coherence hit if verified
```

If `decoherence_bank` search hits and structural verification passes, the candidate has cohered again.

A decoherence hit must be promoted or reinforced through the operation gate.

```text
decoherence_bank candidate
-> structural verification passes
-> input grammar / grammar_relation diff verification passes
-> xi coherence hit
-> core_can_execute('promote.decoherence_hit')
-> promote / reinforce
-> logs.diff
```

Sleep may send structures to `decoherence_bank`, but sleep must not hard-delete them.

Deletion from `decoherence_bank` is explicit manual/operator delete only.

## Expansion boundary

Core expands semantic space during reply generation.

Sleep contracts and cleans semantic space during scheduled maintenance.

```text
reply-time core = expansion
sleep / cron    = consolidation, pruning, decoherence
```

Context-local names, file names, branch names, issue numbers, commit identifiers, and other proper nouns must be usable immediately during reply generation.

They must not wait for scheduled Phase confirmation before they can participate in output.

## Draft meaning

Draft is not a human-review queue.

Draft is not the primary promotion target.

Draft is an unconfirmed candidate filter and anti-pattern evidence store.

Candidate generation must read Draft evidence as negative pressure.

## Decoder / Collapse

Decoder does not originate meaning.

Decoder/collapse invariants are handled by the current core.

No independent constraint table.

No decoder trace table.

No loop guard table.

Looping is unresolved exploration pressure, not a separate error state.

Repeated return to the same grammar path is handled by grammar_array structure, Phase candidate search, near-neighbor expansion, decoherence pressure, aggregate.current pressure, and diff evidence.

If repeated search still cannot resolve, the result remains draft/unresolved and may trigger question notification.
