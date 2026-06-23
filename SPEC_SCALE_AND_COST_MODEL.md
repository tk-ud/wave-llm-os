# Specification: Scale and Cost Model

This file is the canonical authority for storage-scale and per-reply cost assumptions.

Raw observations may grow close to linearly.

Canonical semantic growth is sublinear when repeated vocabulary, grammar, and relation structures reinforce existing rows.

Growth layers:

raw observation logs: mostly linear
canonical vocabulary: saturates earlier
canonical grammar: saturates later than vocabulary
relations: continue to grow through combinations
residual / decoherence evidence: tracks unresolved novelty
hot logs: bounded by retention policy

A large storage system matters because semantic traces, relation evidence, residuals, pressure history, and archive manifests can be preserved and reactivated.

Raw text size is not the main growth driver.

Per-reply search cost is bounded by local candidate search, missing-slot expansion, structural verification, and scope-crossing checks.

Illustrative shape:

expected_search_cost ~= near_candidate_topK + sum(missing_slot_candidate_topK * verification_cost) + scope_crossing_verification_cost

Hot-path intelligence should read current semantic structures, aggregate.current pressure, searchable fallback structures, and recent decision evidence.

It should not require unbounded hot logs.
