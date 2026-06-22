# Supporting Note: Phase Attention Sleep Maintenance

Canonical authority: `SPEC_CANONICAL_CORE.md`.

This note belongs to the Phase Attention specification set.

Legacy sleep scheduling belongs to the same timing class as Phase Attention.

It is not synchronous reply-time work.

It runs as scheduled, idle-window, preemptible maintenance alongside Phase Attention.

---

# Core Rule

```text
Phase Attention timing
= scheduled / idle / preemptible maintenance timing
```

Sleep maintenance runs in that same window.

It is the low-power background metabolism around Phase Attention.

---

# Semantic Invariant

```text
sleep maintenance must not block user-facing inference
sleep maintenance must pause on user input
sleep maintenance must not directly adopt semantic structures
sleep maintenance may refresh aggregates, pressure surfaces, candidate scores, dormant markers, and maintenance queues through operation-gated paths
```

---

# Timing Model

```text
user-facing path
→ never waits for sleep maintenance

idle / low-activity window
→ refresh logs.current pressure
→ run Phase Attention candidate generation
→ run sleep maintenance tasks
→ update maintenance evidence
→ queue promotion candidates, not adopted mutations
```

---

# Preemption Rule

```text
on user input
→ pause sleep maintenance immediately
→ keep current chunk boundary
→ resume only after idle threshold
```

Legacy implementation guidance:

```text
pause_on_user_input = true
resume_after_seconds_idle = 30
cpu_nice = low
io_nice = low
max_cpu_percent = 15
max_io_ops_per_sec = 100
usage window = hour_of_day histogram
window_days = 14
smoothing = ema(alpha = 0.25)
select lowest-activity hours by percentile
require_continuous_idle_minutes = 10
hard_preempt = true
```

---

# Chunked Background Work

Band / relation refinement may run as chunked background work.

```text
max_wall_ms_per_chunk = 120
max_chunks_per_idle_window = 500
include_dormant = true
```

---

# Allowed Outputs

```text
logs.current refresh
relation pressure refresh
phase candidate score refresh
decoherence recurrence aggregation
dormant / cold markers for weak unused candidates
alias / merge candidates as draft evidence
promotion queue candidates
scheduler_job_run evidence
```

---

# Rejected Outputs

```text
direct adopted grammar mutation
direct adopted vocabulary mutation
direct adopted relation mutation
blocking inference
hard removal of semantic basis / seed structures
```

---

# Short Form

```text
Sleep maintenance runs with Phase Attention.
It is scheduled, idle-aware, preemptible, and low priority.
It refreshes pressure and maintenance evidence.
It does not block replies.
It queues candidates instead of silently adopting them.
```
