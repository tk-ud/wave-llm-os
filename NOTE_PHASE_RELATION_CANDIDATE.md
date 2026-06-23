# Supporting Note: Phase Attention / Phase Relation Candidate

Canonical authority: `SPEC_CANONICAL_CORE.md`.

This document explains Phase Attention as scheduled aggregate-weighted relation candidate generation.

It is not implementation authority over the canonical core.

---

# 1. Core Definition

```text
Phase Attention
= scheduled aggregate-weighted relation candidate generation
```

Phase Attention is not Transformer-style attention.

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
reply-time promotion authority
```

It must not be treated as:

```text
input → Phase Attention → answer
```

Phase Attention is scheduled relation growth.

The normal reply path may read Phase-generated candidates, but reply-time coherence promotion belongs to the canonical core reply path.

---

# 3. Canonical Mastication Order

```text
raw observation
→ token table
→ mastication
→ word candidates
→ vocabulary candidates
→ grammar candidates
→ relation candidates
→ phase relation candidates
→ operation-gated promotion / reinforcement
```

Vocabulary and grammar are not copied directly from raw text.

They are generated from token sequences through mastication.

Phase Attention reads normalized structures produced by this pipeline.

---

# 4. Inputs

Primary inputs:

```text
active vocabulary
active grammar
draft vocabulary
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

Phase Attention must not own synchronous reply-time promotion.

---

# 6. Relation Is Required Before Phase Becomes Useful

If relation is thin, output becomes input mirroring.

Without relation:

```text
input fragments in
similar fragments out
```

With relation:

```text
access failure → expired recovery path → urgent deadline
```

The connected form is not meaningful merely because it looks different from the input.

It must survive input grammar / grammar_relation diff and input subtraction.

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

---

# 8. Pressure-Guided Grammar Expansion

Phase Attention reads `logs.current` as the scheduled aggregate pressure surface.

`logs.current` does not create grammar arrays and is not the parent of new `grammar_array` rows.

It exposes pressure used to select attended grammar paths and exploration budgets.

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

---

# 9. Aggregate Weights

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

# 10. Candidate Generation

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

They do not automatically become active structures.

---

# 11. Canonical Output

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

# 12. Phase Relation Candidate Table

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
  rejected_flag boolean not null default false,
  deleted_flag boolean not null default false,

  created_at timestamptz not null default now(),
  last_seen_at timestamptz not null default now(),
  updated_at timestamptz not null default now()
);
```

`draft_flag` is the canonical lifecycle truth.

No enum status table is required.

A Phase candidate becomes useful only after recurrence, scheduled structural verification, or decoder usage evidence.

`relation_array` is optional and must reference relation indexes, not UUIDs.

`grammar_array` is the canonical semantic path output.

---

# 13. Promotion

Phase promotion is scheduled and evidence-driven.

It is not reply-time promotion.

Human review is not part of ordinary promotion.

Reply-time coherence promotion belongs to `SPEC_CANONICAL_CORE.md` and the core reply path.

Canonical Phase promotion path:

```text
phase_relation_candidate
→ near-neighbor selection
→ structural verification passes
→ core_can_execute('promote.phase_relation')
→ grammar_relation
→ logs.diff
```

Freeze blocks candidate generation and promotion.

Contradiction or policy failure blocks promotion and keeps evidence as draft, rejected, or quarantined evidence.

This preserves the principle:

```text
Phase grows candidates on schedule.
Core owns reply-time coherence promotion.
Promotion must be explicit, gated, logged, and automatic when evidence passes.
```

No `adopt.*` operation is canonical.

Promotion uses `promote.*` and flips or reinforces `draft_flag = false` on the promoted target.

---

# 14. Relationship to xi / yj / zk

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

# 15. Difference from Transformer Attention

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

# 16. Failure Mode: Mirror Output

Mirror output is not merely weak relation.

Mirror output is evidence that the system failed to extract meaningful relation delta from input.

Failure pattern:

```text
input contains A, B, C
candidate output repeats A, B, C in cleaned or reordered form
candidate output - input grammar = empty or reorder-only delta
→ mirror_output evidence
→ do not reinforce as new relation
```

Phase Attention should read mirror-output indicators as pressure, but it does not fix mirror output during reply-time.

Core reply-time verification owns:

```text
input grammar / grammar_relation diff
candidate output - input grammar
active search miss handling
decoherence_bank fallback search
```

Phase uses the aggregated evidence later to avoid generating candidates that reproduce the same draft anti-patterns.

This failure should be logged, not hidden.

---

# 17. Growth Metrics

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

---

# 18. Preemptible Sleep Scheduler / Phase Maintenance

Legacy sleep scheduling belongs to the same timing class as Phase Attention.

It is not synchronous reply-time work.

It runs as scheduled, idle-window, preemptible maintenance alongside Phase Attention.

Semantic invariant:

```text
sleep / maintenance tasks must not block user-facing inference
sleep / maintenance tasks must pause on user input
sleep / maintenance tasks must not directly promote semantic structures without operation gate
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
→ queue promotion candidates, not direct ungated mutations
```

Preemption rule:

```text
on user input
→ pause sleep maintenance immediately
→ keep current chunk boundary
→ resume only after idle threshold
```

Legacy `basis` refinement maps to nearby `vocabulary`, `grammar`, and `grammar_array` path refinement.

This maintenance is not merely storage of alias or merge evidence.
