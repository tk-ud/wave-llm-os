# Amendment 001: Aggregate Current / Current History Split

Canonical target: `SPEC_CANONICAL_CORE.md`.

This amendment captures the intended split between the hot aggregate surface and aggregate history.

It supersedes conflicting `current` wording in `SPEC_CANONICAL_CORE.md` until the canonical core file is rewritten directly.

---

# Corrected Interpretation

`aggregate.current` is the hot aggregate pressure surface.

Sleep, Phase Attention, operator actions, reply support, and scheduled maintenance read `aggregate.current` when they need current aggregate values.

`logs.current` is not the hot aggregate surface.

`logs.current` is the time-series evidence of aggregate snapshots.

It records how aggregate values changed over refresh cycles and may be archived or rolled up.

```text
aggregate.current = current aggregate value surface
logs.current      = aggregate snapshot history / evidence
```

Neither table is semantic authority.

Canonical semantic authority remains in the semantic tables, structural verification evidence, relation paths, and operation-gated promotion results.

---

# Canonical Table Additions

```text
aggregate.current
archive.registry
```

The corrected log/aggregate roles are:

```text
logs.coherence     = append-only observation evidence
logs.diff          = mutation / decision evidence
logs.current       = aggregate snapshot history / time-series evidence
aggregate.current  = hot aggregate read model / pressure surface
archive.registry   = archive batch registry and restore manifest
```

---

# `aggregate.current`

`aggregate.current` stores the latest aggregate values for each target and aggregate window.

It may be implemented as a table or materialized view.

It may be updated in place.

```sql
create schema if not exists aggregate;

create table aggregate.current (
  current_uuid uuid primary key default gen_random_uuid(),
  target_table text not null,
  target_index bigint null,
  target_path bigint[] null,
  aggregate_window text not null,
  observed_count bigint not null default 0,
  hit_count bigint not null default 0,
  near_hit_count bigint not null default 0,
  partial_hit_count bigint not null default 0,
  residual_count bigint not null default 0,
  contradiction_count bigint not null default 0,
  mirror_output_count bigint not null default 0,
  decoder_success_count bigint not null default 0,
  scope_transition_count bigint not null default 0,
  relation_frequency bigint not null default 0,
  parent_structure_usage_count bigint not null default 0,
  missing_slot_count bigint not null default 0,
  replaced_slot_count bigint not null default 0,
  slot_missing_or_replaced_rate numeric not null default 0,
  moth_eaten_score numeric not null default 0,
  draft_antipattern_score numeric not null default 0,
  phase_score numeric not null default 0,
  pressure numeric not null default 0,
  refreshed_at timestamptz not null default now(),
  unique (target_table, target_index, aggregate_window)
);
```

The unique current-surface constraint belongs to `aggregate.current`, not `logs.current`.

---

# `logs.current`

`logs.current` stores aggregate snapshot evidence over time.

It must not be treated as the single current row for a target/window.

It has no canonical uniqueness constraint on `(target_table, target_index, aggregate_window)` because multiple snapshots may exist for the same target/window.

```sql
create table logs.current (
  current_log_uuid uuid primary key default gen_random_uuid(),
  current_log_index bigint generated always as identity unique,
  target_table text not null,
  target_index bigint null,
  target_path bigint[] null,
  aggregate_window text not null,
  observed_count bigint not null default 0,
  hit_count bigint not null default 0,
  near_hit_count bigint not null default 0,
  partial_hit_count bigint not null default 0,
  residual_count bigint not null default 0,
  contradiction_count bigint not null default 0,
  mirror_output_count bigint not null default 0,
  decoder_success_count bigint not null default 0,
  scope_transition_count bigint not null default 0,
  relation_frequency bigint not null default 0,
  parent_structure_usage_count bigint not null default 0,
  missing_slot_count bigint not null default 0,
  replaced_slot_count bigint not null default 0,
  slot_missing_or_replaced_rate numeric not null default 0,
  moth_eaten_score numeric not null default 0,
  draft_antipattern_score numeric not null default 0,
  phase_score numeric not null default 0,
  pressure numeric not null default 0,
  aggregate_refreshed_at timestamptz not null,
  logged_at timestamptz not null default now()
);
```

`logs.current` is evidence history.

It may be used for trend inspection, pressure drift, regression analysis, archive verification, and scheduled maintenance diagnostics.

Hot-path current reads should use `aggregate.current`.

---

# Refresh Rule

A `current_refresh` job should:

```text
logs.coherence
+ logs.diff
+ canonical semantic tables
→ compute aggregate values
→ upsert aggregate.current
→ optionally append snapshot to logs.current
```

The snapshot cadence may be lower than the aggregate refresh cadence.

For example, implementations may refresh `aggregate.current` frequently but append `logs.current` snapshots only when values materially change, a threshold is crossed, or a scheduled snapshot interval arrives.

---

# Archive Registry

`archive.registry` records archive batches for logs and aggregate history.

It is a registry and manifest, not semantic authority.

```sql
create schema if not exists archive;

create table archive.registry (
  archive_uuid uuid primary key default gen_random_uuid(),
  archive_index bigint generated always as identity unique,
  source_table text not null,
  archive_kind text not null,
  min_row_index bigint null,
  max_row_index bigint null,
  time_from timestamptz null,
  time_to timestamptz null,
  row_count bigint not null default 0,
  aggregate_window text null,
  storage_kind text not null default 'table',
  storage_uri text null,
  archive_table text null,
  content_hash text null,
  manifest_json jsonb not null default '{}'::jsonb,
  restore_policy text not null default 'manual',
  deleted_after_archive boolean not null default false,
  created_at timestamptz not null default now(),
  created_by text null
);
```

Recommended `archive_kind` values:

```text
hot_window_rolloff
monthly_rollup
snapshot_archive
audit_preservation
cold_storage_export
```

---

# Retention Rule

`logs.current` and `logs.diff` may use bounded hot retention windows.

Recommended initial hot windows:

```text
logs.diff    = 100,000 to 500,000 recent rows
logs.current = 100,000 to 500,000 recent aggregate snapshots
```

Older rows may be archived or rolled up when:

```text
1. the relevant current state is represented in aggregate.current or canonical semantic tables
2. required audit/status summaries have been snapshotted
3. unresolved, quarantined, blocked, or manual investigation rows are excluded from deletion
4. archive.registry records the archived range, row count, storage target, and verification hash
```

Hot-path intelligence should not depend on unbounded log growth.

The hot path depends on current semantic structures, current aggregate pressure, searchable fallback structures, and recent decision evidence.

---

# Migration From Previous Wording

If an implementation already has a unique `logs.current` table with one row per `(target_table, target_index, aggregate_window)`, that table represents `aggregate.current` under this amendment.

If an implementation has append-only or periodically retained aggregate snapshots, those rows represent `logs.current`.

The previous statement that `logs.current` is the canonical scheduled aggregate pressure surface should be read as applying to `aggregate.current`.

The previous statement that `logs.current` is a scheduled aggregate read model should be read as:

```text
aggregate.current = scheduled aggregate read model
logs.current      = scheduled aggregate snapshot evidence
```

---

# Short Form

```text
Do not make logs.current do two jobs.
aggregate.current is the current pressure surface.
logs.current is the pressure history.
archive.registry proves what moved out of hot storage.
Old aggregate snapshots may be archived.
Hot intelligence reads current state, not infinite history.
```
