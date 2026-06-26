# Specification: Cron Pipeline

This file is the canonical authority for scheduled processing.

Canonical scheduler job kinds:

```text
current_refresh
phase_candidate_generation
sleep_consolidation
archive_rolloff
```

The scheduler job kind list is a namespace list.

Canonical execution order is defined by the canonical flow below.

Operation keys and scheduler job kinds are separate namespaces.

Promotion is not a scheduler job.

The scheduler may generate candidates or consolidate evidence, but it does not replace coherence as the promotion decision axis.

Do not add phase_candidate_verification, phase_candidate_promotion, or promotion-eligibility scheduler jobs.

## Canonical flow

```text
logs.coherence + logs.diff + semantic tables
-> aggregate.current
-> optional logs.current snapshot
-> phase_candidate_generation
-> sleep_consolidation
-> archive registry update
```

## Phase

Phase Attention is cron-like scheduled aggregate-weighted relation candidate generation.

Phase reads aggregate.current pressure, Draft evidence, decoherence fallback pressure, and relation evidence.

Phase may read `decoherence_bank` as a source of Draft candidates.

When Phase generates a Draft grammar_relation from `decoherence_bank` evidence, its child vocabulary, grammar, and grammar_relation elements must remain `draft_flag = true` for that candidate path.

Phase writes draft relation paths to `phase_relation_candidate`.

Phase outputs grammar_index arrays only.

Promotion must pass the operation gate.

Phase is not the synchronous raw-input reply path.

Phase must not define reply-time promotion semantics; reply-time coherence promotion belongs to the core reply path.

Phase candidate generation must reduce or reject candidates that reproduce unresolved Draft anti-patterns without new coherence evidence.

Phase relation candidates become promotion candidates only through phase_score and operation gate rules.

## Sleep

Core expands semantic space during reply generation.

Sleep contracts and cleans semantic space during scheduled maintenance.

```text
reply-time core = expansion
sleep / cron    = consolidation, pruning, decoherence
```

Sleep reads aggregate usage windows from aggregate.current, logs.coherence, logs.diff, and logs.current history.

Sleep sends unused or unstable vocabulary, grammar, grammar slots, and relation paths to `decoherence_bank`.

Sleep does not make reply generation stricter.

Sleep makes future search cheaper by sending unstable structures to the fallback search layer and letting Phase avoid Draft anti-patterns.

Sleep never hard-deletes semantic structures.

Deletion remains explicit manual/operator operation.

## Archive

Archive rolloff may move old logs.current and logs.diff history out of hot storage after registry evidence is written.

Archive rolloff must not remove current semantic authority.

Cron jobs do not replace operation-gated promotion.
