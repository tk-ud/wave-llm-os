# Specification: Cron Pipeline

This file is the canonical authority for scheduled processing.

Canonical jobs:

current_refresh
sleep_consolidation
phase_candidate_generation
archive_rolloff

Canonical flow:

logs.coherence + logs.diff + semantic tables -> aggregate.current -> optional logs.current snapshot -> phase candidate generation -> sleep consolidation -> archive registry update

Phase reads aggregate.current pressure, Draft evidence, fallback pressure, and relation evidence.

Phase writes draft relation paths to phase_relation_candidate.

Sleep consolidates repeated, unstable, unused, or fallback structures without replacing reply-time coherence.

Cron jobs do not replace operation-gated promotion.
