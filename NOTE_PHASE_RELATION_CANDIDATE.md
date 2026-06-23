# Supporting Note: Phase Attention / Phase Relation Candidate

Canonical authority: `SPEC_CANONICAL_CORE.md`.

This document is a draft supporting specification for Phase Attention in Wave LLMOS.

Phase Attention preserves selected legacy principles from Wave LLMOS, but its implementation authority is the canonical PostgreSQL-backed core.

Legacy principles preserved here:

```text
meaning originates from observation
raw input is chewed through mastication
decoder does not originate meaning
collapse is explicit
semantic state is persistent and inspectable
```

Phase Attention extends those principles by defining how normalized vocabulary, grammar, and relation aggregates generate cross-layer semantic relation candidates.

---

# 1. Core Definition

Phase Attention is aggregate-weighted relation candidate generation.

```text
Phase Attention
= scheduled aggregate-weighted relation candidate generation
```

It is not Transformer-style attention.

It does not directly read raw text as its primary input.

It does not run as the main synchronous reply-time reasoning process.

It works after observation, tokenization, mastication, coherence hits, decoherence banking, relation accumulation, and aggregate refresh.

---

# 2. What Phase Attention Is Not

Phase Attention is not:

```text
Transformer Attention
softmax over token embeddings
raw input reasoning
next-token prediction
decoder logic
one-shot semantic search
reply-time adoption authority
```

It must not be treated as:

```text
input → Phase Attention → answer
```

That would make the system heavy and structurally similar to ordinary attention systems.

Phase Attention is a scheduled relation-growth mechanism.

The normal reply path should read Phase-generated candidates, not generate the full relation search from scratch.

Reply-time coherence promotion belongs to the canonical core reply path, not to Phase Attention.

---

# 3. Canonical Mastication Order

The public or canonical raw seed is token-level.

```text
raw seed = token table
```

Vocabulary and grammar are not copied directly from raw text.

They are generated from token sequences through mastication.

Canonical order:

```text
raw observation
→ token table
→ mastication
→ word candidates
→ vocabulary candidates
→ grammar candidates
→ relation candidates
→ phase relation candidates
→ adopted structures
```

Phase Attention reads the normalized structures produced by this pipeline.

---

# 4. Inputs

Primary inputs:

```text
adopted vocabulary
draft vocabulary
adopted grammar
draft grammar
grammar_relation
logs.coherence
logs.current
logs.diff
decoherence_bank
relation aggregate weights
hit counts
co-occurrence counts
scope-crossing continuity
promotion history
decoherence-bank recurrence
```

Optional inputs:

```text
candidate bundle history
near-hit records
partial-hit residuals
unmatched grammar slots
relation density metrics
semantic-axis candidates
mirror-output indicators
decoder-success relation usage
```

Phase Attention avoids raw observation text unless controlled audit or debugging requires it.

---

# 5. Runtime Boundary

Synchronous reply path:

```text
input
→ token / word candidates
→ xi coherence hit
→ yj residual bank
→ coherence relation lookup
→ zk coherence decoder
→ output collapse
→ core-owned coherence promotion when structural verification passes
```

Scheduled Phase path:

```text
normalized vocabulary
+ normalized grammar
+ grammar_relation aggregates
+ logs.current pressure
+ hit / recurrence / promotion weights
→ Phase Attention
→ phase_relation_candidate
→ scheduled verification / promotion or decoder support
```

This keeps ordinary response generation light.

Phase Attention is cron-like scheduled work.

It must not own the synchronous reply-time adoption path.

---

# 6. Relation Is Required Before Phase Becomes Useful

If relation is thin, output becomes input mirroring.

Without relation:

```text
input fragments in
similar fragments out
```

The system may recognize local vocabulary and grammar, but it cannot connect divided input scopes.

Example:

```text
scope 1: account access failed
scope 2: recovery link expired
scope 3: deadline is today
```

Without relation:

```text
account
recovery link
today
```

With relation:

```text
access failure → expired recovery path → urgent deadline
```

The second form is not merely a mirror of the input.

It is a connected grammar path.

Therefore, Phase Attention depends on the growth of the coherence relation layer.

---

# 7. Coherence Relation Layer

The coherence relation layer stores semantic continuity.

It connects divided vocabulary and grammar candidates.

It may store:

```text
grammar-to-grammar relation
vocabulary-to-grammar relation
vocabulary-to-vocabulary relation
scope-crossing relation
order-sensitive grammar chain
use-case relation
```

Conceptually:

```text
coherence relation layer
= semantic continuity memory
```

Without this layer, Phase Attention has no stable structure to slide across.

---

# 8. Sliding Across Layers

Phase Attention slides across normalized layers.

The target is not one nearest candidate.

The target is a candidate relation path.

Example layer slide:

```text
vocabulary bundle
→ grammar candidate
→ relation chain
→ corrected grammar candidate
```

Another example:

```text
adopted grammar A
→ draft grammar B
→ recurring relation C
→ phase relation candidate D
```

This allows the system to generate relation candidates that are not directly present as a single raw input span.

---

# 9. Pressure-Guided Grammar Expansion

Phase Attention reads `logs.current` as the scheduled aggregate pressure surface.

`logs.current` does not create grammar arrays and is not the parent of new `grammar_array` rows.

It exposes pressure used to select attended grammar paths and exploration budgets.

`logs.current` may expose:

```text
unresolved grammar paths
repeated grammar returns
high-pressure decoherence clusters
scope-crossing partial relations
near-hit grammar candidates
missing grammar slots
mirror-output indicators
decoder-success relation usage
```

Expansion process:

```text
attended semantic field / selected grammar path
+ logs.current pressure
→ identify terminal grammar slot or missing slot
→ slide vocabulary / grammar / grammar_relation arrays
→ search near grammar arrays
→ test scope-crossing continuity
→ generate expanded grammar_array
→ store as phase_relation_candidate
→ wait for recurrence / decoder evidence / scheduled verification
→ promote through core_can_execute('promote.phase_relation')
```

Phase Attention therefore does not merely search similar grammar.

It grows candidate grammar paths through structural search guided by accumulated unresolved pressure.

---

# 10. Aggregate Weights

Phase Attention may use inspectable aggregate weights such as:

```text
observed_count
hit_count
near_hit_count
partial_hit_count
co_occurrence_count
scope_transition_count
relation_frequency
coherence_weight
decoherence_recurrence
promotion_count
relation_density
mirror_output_count
decoder_success_count
contradiction_count
```

These are not neural parameters.

They are database aggregates.

They should be queryable and reproducible.

---

# 11. Candidate Generation

Phase Attention generates draft relation candidates such as:

```text
grammar A → grammar B
grammar B → grammar C
vocabulary bundle X → grammar A
vocabulary bundle Y → grammar B
grammar A + grammar C recur under the same use case
relation R appears across multiple scopes
relation R predicts grammar candidate G
```

These are draft candidates.

They do not automatically become adopted structures.

---

# 12. Canonical Output

Phase output is:

```text
grammar_array bigint[] of grammar.grammar_index
```

No token arrays in Phase output.

No vocabulary arrays in Phase output.

No UUID semantic references.

UUIDs are row/event identity only.

Semantic references use indexes.

Arrays use `bigint[]` index arrays only.

---

# 13. Phase Relation Candidate Table

Draft table sketch:

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
  status text not null default 'draft',

  created_at timestamptz not null default now(),
  last_seen_at timestamptz not null default now(),
  updated_at timestamptz not null default now()
);
```

A Phase candidate becomes useful only after recurrence, scheduled structural verification, or decoder usage evidence.

`relation_array` is optional and must reference relation indexes, not UUIDs.

`grammar_array` is the canonical semantic path output.

---

# 14. Promotion

Phase promotion is scheduled and evidence-driven.

It is not reply-time promotion.

Human review is not part of ordinary promotion.

Reply-time coherence promotion belongs to `SPEC_CANONICAL_CORE.md` and the core reply path.

State examples:

```text
draft
observed
reinforced
candidate_for_adoption
adopted
rejected
archived
```

Phase promotion evidence may include:

```text
recurrence
coherence weight
successful decoder usage
low contradiction rate
stable scope-crossing appearance
mirror-output reduction
near-neighbor stability
scheduled structural verification
```

Scheduled Phase promotion path:

```text
phase_relation_candidate
→ near-neighbor selection
→ structural verification passes
→ core_can_execute('promote.phase_relation')
→ grammar_relation
→ logs.diff
```

Freeze blocks candidate generation and promotion.

Contradiction or policy failure blocks promotion and keeps evidence as draft, rejected, or quarantined state.

This preserves the principle:

```text
Phase grows candidates on schedule;
core owns reply-time coherence adoption;
adoption must be explicit, gated, logged, and automatic when evidence passes.
```

---

# 15. Relationship to xi / yj / zk

Phase Attention is not a replacement for `xi`, `yj`, or `zk`.

It supports them.

```text
xi
= checks whether token / vocabulary / grammar candidates cohere with known or draft space

yj
= stores no-hit, low-hit, and unresolved residuals

coherence relation layer
= connects coherent candidates into semantic continuity

Phase Attention
= uses normalized aggregates to generate new relation candidates

zk
= projects hit bundles and relation evidence into corrected grammar output
```

Short form:

```text
xi finds hits.
yj stores unresolved difference.
relation connects hits.
Phase grows relation candidates.
zk decodes through corrected grammar.
```

---

# 16. Why This Is Phase Attention

It is called Phase Attention because it does not merely attend to tokens.

It attends to the phase-like arrangement of already-normalized structures.

The system observes:

```text
which candidates recur
which candidates appear near each other
which candidates bridge scopes
which grammar candidates slide into each other
which relation chains accumulate pressure
```

Attention is not local token weighting here.

It is relation-field exploration.

---

# 17. Difference from Transformer Attention

Transformer Attention:

```text
computes token-to-token weights inside a learned representation
usually operates during inference
compresses meaning into model parameters
```

Phase Attention:

```text
uses inspectable aggregate weights over normalized vocabulary, grammar, and relation tables
primarily operates outside ordinary reply-time inference
grows inspectable relation candidates in persistent tables
```

---

18. Failure Mode: Mirror Output

Mirror output is not merely weak relation.

Mirror output is evidence that the system failed to extract meaningful relation delta from input.

Failure pattern:

input contains A, B, C
candidate output repeats A, B, C in cleaned or reordered form
candidate output - input grammar = empty or reorder-only delta
→ mirror_output evidence
→ do not reinforce as new relation

Mirror output can occur when:

active relation search misses
input grammar / grammar_relation diff is skipped
input grammar / grammar_relation diff fails
candidate output - input grammar is empty
candidate output - input grammar is reorder-only
decoherence_bank fallback fails to re-cohere

Relation weakness is one cause, not the full definition.

Vocabulary and grammar may be present, and relation may even be nearby, but the output is still mirror output if input subtraction leaves no meaningful relation delta.

Phase Attention should read mirror-output indicators as pressure, but it does not fix mirror output during reply-time.

Core reply-time verification owns:

input grammar / grammar_relation diff
candidate output - input grammar
active search miss handling
decoherence_bank fallback search

Phase uses the aggregated evidence later to avoid generating candidates that reproduce the same draft anti-patterns.

This failure should be logged, not hidden.
---

# 19. Growth Metrics

Phase Attention development should be tracked with relation-growth metrics.

Useful metrics:

```text
relation_count
relation_density
average_relation_length
scope_crossing_relation_count
phase_relation_candidate_count
phase_candidate_promotion_rate
decoder_success_with_relation
mirror_output_rate
novel_relation_usage_count
coherence_weight_distribution
```

These metrics matter more than raw database size alone.

A large database with weak relation may still mirror input.

A smaller database with strong relation may produce better connected output.

---

# 20. Preemptible Sleep Scheduler / Phase Maintenance

Legacy sleep scheduling belongs to the same timing class as Phase Attention.

It is not synchronous reply-time work.

It runs as scheduled, idle-window, preemptible maintenance alongside Phase Attention.

Core rule:

```text
Phase Attention timing
= scheduled / idle / preemptible maintenance timing
```

Sleep maintenance runs in that same window.

It is the low-power background metabolism around Phase Attention.

Semantic invariant:

```text
sleep / maintenance tasks must not block user-facing inference
sleep / maintenance tasks must pause on user input
sleep / maintenance tasks must not directly adopt semantic structures
sleep / maintenance tasks may refresh aggregates, pressure surfaces, candidate scores, dormant markers, and maintenance queues through operation-gated paths
```

Recommended timing model:

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

Preemption rule:

```text
on user input
→ pause sleep maintenance immediately
→ keep current chunk boundary
→ resume only after idle threshold
```

Legacy parameters preserved as implementation guidance:

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

Band / relation refinement may run as chunked background work:

```text
max_wall_ms_per_chunk = 120
max_chunks_per_idle_window = 500
include_dormant = true
```

In the current core vocabulary, legacy `basis` refinement maps to nearby `vocabulary`, `grammar`, and `grammar_array` path refinement.

This maintenance is not merely storage of alias or merge evidence.

It periodically shapes the search field so future Phase Attention passes need fewer repeated near-neighbor searches.

Attractor basin refinement:

```text
similar vocabulary candidates
+ similar grammar candidates
+ similar grammar_array paths
→ generate alias / merge candidates as draft evidence
→ refine the attractor basin around stable semantic paths
→ reduce future near-neighbor search count
→ reduce future Phase Attention exploration count
→ keep synchronous reply-time lookup cheap
```

This does not collapse candidates into adopted structures by itself.

It prepares draft evidence for later operation-gated adoption or rejection.

Vector / SQL role split:

```text
vector geometry
= basin shape / scale / density error inspection for sleep-time integration

SQL join diff
= shared structure / residual / missing / excess inspection for decode-time and verification
```

Sleep integration uses both:

```text
vector geometry finds candidate basins
SQL join diff verifies structural compatibility
merge / alias / dormant decisions remain draft evidence until operation-gated adoption
```

Recommended decision pattern:

```text
vector shape close + scale error small + SQL residual small
→ merge candidate

vector shape close + scale error large + SQL overlap high
→ alias candidate or parent-child candidate

vector shape close + SQL residual large
→ false attractor / polysemy risk

vector shape far + SQL overlap high
→ boundary split issue or vocabulary segmentation issue
```

Canonical reference stays unchanged:

```text
grammar.vocabulary_array = semantic reference
```

Derived retrieval projections may include:

```text
grammar_structural_vector
grammar_expanded_token_array
grammar_shape_metrics
grammar_scale_metrics
join_diff_cache
```

Decode should prefer SQL join diff because it exposes where structures match, diverge, disappear, or remain unresolved.

Sleep integration should prefer vector geometry first because it exposes basin shape and scale error before structural verification.

Allowed sleep maintenance outputs:

```text
logs.current refresh
relation pressure refresh
phase candidate score refresh
decoherence recurrence aggregation
dormant / cold markers for weak unused candidates
alias / merge candidates as draft evidence
vocabulary alias / merge candidates as draft evidence
grammar alias / merge candidates as draft evidence
grammar_array path alias / merge candidates as draft evidence
attractor basin refinement evidence
future search-count reduction evidence
vector geometry basin candidate evidence
SQL join diff verification evidence
promotion queue candidates
scheduler_job_run evidence
```

Rejected sleep maintenance outputs:

```text
direct adopted grammar mutation
direct adopted vocabulary mutation
direct adopted relation mutation
blocking inference
permanent removal of semantic basis / seed structures
```

Sleep maintenance is therefore the low-power background metabolism of the Phase system.

---

# 21. Short Form

```text
Phase Attention does not read raw text first.
It reads normalized vocabulary, grammar, and relation aggregates.
It reads logs.current as pressure.
It slides across layers.
It generates grammar_array relation candidates.
It runs on a schedule.
Phase is cron-like maintenance, not reply generation.
Core owns reply-time coherence adoption.
Human review is not required for ordinary promotion.
Sleep maintenance runs in the same scheduled / idle / preemptible window.
Sleep maintenance refines nearby vocabulary, grammar, and grammar_array paths as draft evidence.
Sleep maintenance shapes attractor basins to reduce future search count.
Vector geometry is useful for sleep-time basin integration.
SQL join diff is useful for decode-time structural verification.
It supports zk.
It prevents the system from becoming a mirror of the input.
```

Legacy mastication produces the material.

Relation connects the material.

Phase grows new relation paths.

Sleep maintenance keeps Phase metabolism cheap, idle-aware, and non-blocking.
