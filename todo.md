Wave LLMOS Implementation TODO (Manifest-Compliant, Codex Prompt – Full)
This document defines the implementation roadmap for Wave LLMOS strictly compliant with MANIFEST.yaml.
MANIFEST is law. Do not implement any behavior that contradicts the manifest.

IMPLEMENTATION INVARIANT — DO NOT VIOLATE:
- WaveCore.psi is the primary semantic state.
- SpectralObserver.band_features is the ONLY valid input to MeaningSpace basis creation.
- MeaningSpace basis vectors must NEVER be derived directly from psi, tokens, logits, or embeddings.
- DBDecoder must NEVER access WaveCore.psi or SpectralObserver internal state directly.
- Sampling is the ONLY collapse point in the system.

CORE PHILOSOPHY & CONSTITUTIONAL CONSTRAINTS (Non-Negotiable)
- Non-parametric: No neural network weights.
- Wave Geometry is primary.
- Inspectable & Explicit meaning space.
- Non-destructive: Never hard-delete basis vectors.
- Manifest-only constraints.
- External observation is Tier-3 last resort.
- “Wave” is architectural, not physical equivalence.

----------------------------------------------------------------
Phase 0 — Repository & Module Boundaries
----------------------------------------------------------------

## TODO-0.1 Project structure [status:todo]

Create base structure:

wave_llmos/
├ manifests/
├ core/
├ wave_core/
├ spectral_observer/
├ spectral_alignment/
├ meaning_space/
├ db_decoder/
├ learning/
├ fallback/
├ mastication/
├ maintenance/
├ storage/
├ api/
└ tests/

Definition of Done:
- No circular imports.
- Single manifest loader.

----------------------------------------------------------------
Phase 1 — Manifest Loader & Invariants
----------------------------------------------------------------

## TODO-1.1 Parse MANIFEST.yaml into typed config [status:todo]

Definition of Done:
- Config loads.
- Missing keys fail fast.

## TODO-1.2 Constraint engine (no hardcoded rules) [status:todo]

Definition of Done:
- Rules loaded from manifest path only.

----------------------------------------------------------------
Phase 2 — Storage Layer
----------------------------------------------------------------

## TODO-2.1 SQLite schema implementation [status:todo]

Tables:
- basis
- token_basis
- pattern_next
- bigram
- basis_label_dictionary

Definition of Done:
- Idempotent migration.
- projection_dim = 256 float32 storage.

## TODO-2.2 Anti-delete enforcement [status:todo]

Definition of Done:
- DELETE prohibited.
- dormant flag works.

----------------------------------------------------------------
Phase 3 — Tokenization
----------------------------------------------------------------

## TODO-3.1 MeCab + UniDic tokenizer integration [status:todo]

Definition of Done:
- Deterministic tokenization.
- NFKC normalization.

----------------------------------------------------------------
Phase 4 — Wave Core
----------------------------------------------------------------

## TODO-4.1 WaveCore state implementation [status:todo]

- psi length = 1024 complex
- ring buffer length = 64

Definition of Done:
- Deterministic evolution.

## TODO-4.2 Phase-hash complex token embedding [status:todo]

Definition of Done:
- Same token stream → same psi evolution.

----------------------------------------------------------------
Phase 5 — Spectral Observation
----------------------------------------------------------------

## TODO-5.1 SpectralObserver STFT + band_features [status:todo]

- STFT window=32 hop=1
- 8 log-spaced bands
- features: Edb, dEdb, ddEdb

Definition of Done:
- Fixed dimension vector.
- Only band space leaves module.

----------------------------------------------------------------
Phase 6 — Spectral Alignment
----------------------------------------------------------------

## TODO-6.1 token_band_binding via gradient [status:todo]

Definition of Done:
- Sparse token→band mapping.

----------------------------------------------------------------
Phase 7 — Meaning Space
----------------------------------------------------------------

## TODO-7.1 Projection: band_features → sparse_mixture [status:todo]

Definition of Done:
- topK = 32
- cosine similarity
- residual computed

## TODO-7.2 Basis creation (decoherence trigger) [status:todo]

Trigger:
- max_cos < 0.35

Definition of Done:
- Basis created from band_features ONLY.

## TODO-7.3 Coherence & label generation (metadata only) [status:todo]

Definition of Done:
- Label never used as retrieval key.

## TODO-7.4 Attenuation & dormancy [status:todo]

Definition of Done:
- strength_decay applied
- dormant threshold works

----------------------------------------------------------------
Phase 8 — DB Decoder
----------------------------------------------------------------

## TODO-8.1 pattern_key from quantized projection [status:todo]

Definition of Done:
- Deterministic hash.

## TODO-8.2 pattern_next + bigram blending [status:todo]

Weights:
- geometry=0.75
- transition=0.25

Definition of Done:
- logits produced.

----------------------------------------------------------------
Phase 9 — Constraints Layer
----------------------------------------------------------------

## TODO-9.1 Post-logit constraint application [status:todo]

Definition of Done:
- influence_cap=0.25 enforced.
- No semantic origination.

----------------------------------------------------------------
Phase 10 — Sampling
----------------------------------------------------------------

## TODO-10.1 Sampling implementation [status:todo]

Params:
- temperature=0.9
- top_k=50
- top_p=0.92

Definition of Done:
- Only collapse point.

----------------------------------------------------------------
Phase 11 — Learning Updates
----------------------------------------------------------------

## TODO-11.1 Additive update rules [status:todo]

Definition of Done:
- EMA applied correctly.
- No vector corruption.

----------------------------------------------------------------
Phase 12 — Semantic Freeze
----------------------------------------------------------------

## TODO-12.1 Full semantic_freeze state model [status:todo]

Must include:
- enabled
- freeze_timestamp
- freeze_hash
- audit log

Definition of Done:
- Freeze blocks learning only.
- Inference still works.

----------------------------------------------------------------
Phase 13 — Fallback Hierarchy
----------------------------------------------------------------

## TODO-13.1 Tier-1 / Tier-2 / Tier-3 implementation [status:todo]

Definition of Done:
- Tier order respected.
- Web only via mastication.

----------------------------------------------------------------
Phase 14 — Internal Mastication
----------------------------------------------------------------

## TODO-14.1 Silent reading pipeline [status:todo]

Definition of Done:
- Raw text discarded.
- Only aggregates persist.

----------------------------------------------------------------
Phase 15 — Maintenance
----------------------------------------------------------------

## TODO-15.1 Sleep scheduler [status:todo]

Definition of Done:
- Preemptible.
- Does not modify psi directly.

## TODO-15.2 Band refinement tasks [status:todo]

Definition of Done:
- No hard delete.
- Inspectable state preserved.

----------------------------------------------------------------
Phase 16 — Distributed Sync (Optional)
----------------------------------------------------------------

## TODO-16.1 basis_event serialization + signature [status:todo]

Definition of Done:
- ed25519 verification.
- Disabled by default.

----------------------------------------------------------------
Phase 17 — API & Inspectability
----------------------------------------------------------------

## TODO-17.1 API endpoints implementation [status:todo]

Endpoints:
- POST /generate
- POST /freeze
- GET /meaning_space
- GET /health

Definition of Done:
- Valid JSON.
- Manifest version visible.

----------------------------------------------------------------
Phase 18 — Tests
----------------------------------------------------------------

## TODO-18.1 Cold start test [status:todo]

Definition of Done:
- Empty DB works.

## TODO-18.2 Invariant tests [status:todo]

Must verify:
- Delete prohibited
- Labels not retrieval keys
- Sampling only collapse
- Freeze blocks learning
- External observation gated

----------------------------------------------------------------

SUCCESS CRITERIA:
Wave LLMOS runs full pipeline strictly Manifest-compliant,
expands semantic space additively,
never deletes meaning,
and remains fully inspectable.