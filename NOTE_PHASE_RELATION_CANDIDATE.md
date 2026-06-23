# Supporting Note: Phase Relation Candidate Implementation

Canonical authority: `SPEC_CRON_PIPELINE.md` and `SPEC_SCORING_AND_THRESHOLDS.md`.

This file is not a source of truth.

It only preserves implementation guidance for Phase relation candidate storage and inspection.

If this file conflicts with a SPEC file, the SPEC file wins.

---

# Implementation Boundary

Phase candidate generation, scheduling boundaries, promotion rules, and score formulas are defined by routed SPEC files.

This NOTE may describe SQL storage shape, query inputs, and implementation sketches only.

---

# Candidate Storage Sketch

```sql
create table phase_relation_candidate (
  phase_relation_candidate_uuid uuid primary key default gen_random_uuid(),
  phase_relation_candidate_index bigint generated always as identity unique,

  grammar_array bigint[] not null,
  relation_array bigint[] null,

  candidate_kind text not null default 'relation_candidate',
  source_aggregate_window text null,

  relation_hash text not null unique,
  evidence_json jsonb not null,

  phase_score numeric not null default 0,
  pressure numeric not null default 0,
  evidence_count bigint not null default 0,

  draft_flag boolean not null default true,
  rejected_flag boolean not null default false,
  deleted_flag boolean not null default false,

  created_at timestamptz not null default now(),
  last_seen_at timestamptz not null default now(),
  updated_at timestamptz not null default now()
);
```

---

# Query Input Sketch

Phase implementation may read inspectable aggregate values from `aggregate.current` and historical snapshots from `logs.current`.

Useful implementation inputs include:

```text
observed_count
hit_count
near_hit_count
partial_hit_count
scope_transition_count
relation_frequency
mirror_output_count
decoder_success_count
contradiction_count
```

These values are database aggregates, not neural parameters.

---

# Implementation Flow Sketch

```text
aggregate.current pressure
+ relation evidence
+ residual / fallback evidence
-> candidate generation query
-> phase_relation_candidate row
-> evidence row
-> routed SPEC decision
```

No promotion rule is defined in this NOTE.
