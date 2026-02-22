Wave LLMOS Implementation TODO (Manifest-Compliant, Codex Prompt – Full)
This document defines the implementation roadmap for Wave LLMOS strictly compliant with MANIFEST.yaml.
MANIFEST is law. Do not implement any behavior that contradicts the manifest.

CORE PHILOSOPHY & CONSTITUTIONAL CONSTRAINTS (Non-Negotiable)
- Non-parametric: No neural network weights. Meaning is not stored in weights; it accumulates as basis structures.
- Wave Geometry is primary: Wave state is the primary semantic state; sampling is the only collapse.
- Inspectable & Explicit: Meaning space must be explicitly stored and queryable.
- Non-destructive: Never hard-delete basis vectors; dormancy/attenuation allowed; history preserved.
- Manifest-only constraints: No rule-sets embedded in source code; constraints must be loaded from manifest-defined files.
- External observation is Tier-3 last resort: Must be mastication-gated; raw text discarded.
- “Wave” is architectural, not physical equivalence.

Phase 0 — Repository & Module Boundaries (No Circulars)
Priority: CRITICAL

TODO-0.1 Project structure (aligned to manifest components)
Create base structure:
wave_llmos/
├ manifests/              # constraints.yaml, axis.yaml (optional)
├ core/                   # config loader, manifest parser, invariants enforcement
├ wave_core/              # psi state, euler integrator, phase-hash token embedding
├ spectral_observer/      # STFT, banding, band_features (Edb, dEdb, ddEdb)
├ spectral_alignment/     # token->band binding using gradient in band space
├ meaning_space/          # basis lifecycle, projection, vector search, dormancy/decay
├ db_decoder/             # pattern_key, pattern_next, bigram_backoff, blending -> logits
├ learning/               # update rules for basis, token_basis, pattern_next, bigram
├ fallback/               # tier-1 blend, tier-2 ask, tier-3 web (gated)
├ mastication/            # internal_mastication pipeline (silent reading)
├ maintenance/            # sleep scheduler, band_refinement tasks (preemptible)
├ storage/                # sqlite schema + DAOs with delete prohibition
├ api/                    # endpoints: generate, freeze, meaning_space inspection
└ tests/                  # invariants + cold start + regression

Definition of Done:
- Modules import without circular dependencies.
- A single Manifest loader exists and is the source of truth for runtime constants and invariants.

Phase 1 — Manifest Loader & Invariant Enforcement
Priority: CRITICAL

TODO-1.1 Parse MANIFEST.yaml into typed config
- Load constitution_order, runtime, generation_pipeline, storage.schema, semantic_freeze, fallback_hierarchy, internal_mastication, external_observation.
- Provide accessors (e.g., cfg.runtime.wave.psi_bins_B, cfg.runtime.meaning_space.projection_dim, etc.).

Definition of Done:
- Running program prints key runtime config values and exits successfully.
- Any missing required manifest keys fails fast with a clear error.

TODO-1.2 Prohibit “hardcoded rules”
- Implement a constraint engine that loads rules from manifests/constraints.yaml path defined by manifest. No inline rules in code.

Definition of Done:
- Constraints engine refuses to run if no manifest-defined constraints file exists (unless constraints layer disabled by config).

Phase 2 — Storage (SQLite Minimum Schema, No DELETE)
Priority: CRITICAL

TODO-2.1 Implement SQLite schema exactly as manifest storage section
Create tables: basis, token_basis, pattern_next, bigram, basis_label_dictionary and indexes.

Definition of Done:
- Schema migration runs idempotently.
- DAO layer does not expose DELETE methods for basis (or any “hard delete” path).
- basis.vec_blob stores float32 packed vectors (projection_dim=256).

TODO-2.2 Anti-delete enforcement
- Implement DB triggers OR DAO-level guard rails (preferred: both):
  - Reject DELETE FROM basis...
  - Only allow setting dormant=1 and adjusting strength/stability via updates.

Definition of Done:
- Attempting to delete a basis raises an error; dormant path works.

Phase 3 — Tokenization (Manifest runtime.tokenizer)
Priority: CRITICAL

TODO-3.1 Tokenizer integration (MeCab + UniDic)
- Tokenizer engine = mecab, dict = unidic, normalize NFKC true.

Definition of Done:
- tokenize("こんにちは世界") returns 2+ tokens deterministically (exact split may vary by dictionary; ensure determinism).

Phase 4 — Wave Core (Primary Semantic State)
Priority: CRITICAL

TODO-4.1 Implement WaveCore state
- psi: complex vector length = runtime.wave.psi_bins_B (1024).
- psi_ring: ring buffer length = runtime.wave.state_ring (64), store magnitude snapshot(s).

Definition of Done:
- Initialize empty wave state deterministically.
- Pushing tokens updates psi and ring.

TODO-4.2 Token embedding via phase-hash (complex)
Implement:
- phase = 2π * hash(token_id, bin) / M
- value = exp(i*phase)
- Integrator: euler
- Update equation: psi = psi + input_gain*embed(token) - damping*psi with runtime params.

Definition of Done:
- Given same token stream, wave state evolution is deterministic.

Phase 5 — Spectral Observation (Band Space Only)
Priority: CRITICAL

TODO-5.1 Implement SpectralObserver.observe(psi_ring) -> band_features
- Transform method = STFT, window=32, hop=1.
- Bands=8, spacing=log. Features = [Edb, dEdb, ddEdb], eps=1e-8.

Definition of Done:
- Output is fixed dimension, float32 vector compatible with projection_dim=256 after projection shaping.
- No use of raw bin space outside spectral_observer; observation manifold is band space.

Phase 6 — Spectral Alignment (For gradient-based actions / web fallback query builder)
Priority: HIGH

TODO-6.1 token_band_binding via gradient in band space
- gradient_source: abs(dEdb) primary, abs(ddEdb) secondary; weights dEdb=1.0, ddEdb=0.35.
- allow_multi_band true; threshold 0.7.

Definition of Done:
- Returns sparse_map token_index -> band_index list (multi-band allowed).

Phase 7 — Meaning Space (Projection, Basis Lifecycle, No Hard Delete)
Priority: CRITICAL

TODO-7.1 Projection: band_features -> sparse_mixture(topK)
- Similarity metric cosine
- topK = runtime.meaning_space.basis_topK (32)
- residual = 1 - max_cos_similarity, output_name = projection_residual

Definition of Done:
- project(band_features) returns:
  - topK basis weights
  - max_cos
  - residual

TODO-7.2 Basis lifecycle: decoherence creation (similarity-only)
- Trigger: max_similarity below create_threshold_cos (0.35)
- Create basis:
  - vec_source = spectral_observer.output.band_features (NOT raw psi)
  - strength_init_low=0.05, stability_init_low=0.05, dormant=false

Definition of Done:
- Empty DB: first observation creates basis (cold start).

TODO-7.3 Coherence check & label generation (metadata only)
- When stability ≥ stability_threshold (0.75): generate label via token_basis_association_topK and store in basis.label_cache + basis_label_dictionary.
- Label must never be used as retrieval key.

Definition of Done:
- Label appears in inspection APIs, but similarity search uses vec only.

TODO-7.4 Attenuation + dormancy
- Apply strength_decay=0.995 each cycle (policy detail up to you, but must exist).
- If strength < dormant_strength_below (0.01): set dormant true (no deletion).

Definition of Done:
- Weak bases become dormant; remain queryable; never deleted.

Phase 8 — DB Decoder (Pseudo-MLP Replacement)
Priority: CRITICAL

TODO-8.1 pattern_key from quantized projection
- pattern_key = hash_quantized_projection(projection_sparse_mixture)
- quantize:
  - basis_weight_round=0.02
  - basis_count_topN=12

Definition of Done:
- Same projection -> same pattern_key deterministically.

TODO-8.2 Retrieve next-token stats and blend
- retrieval sources: pattern_next and bigram_backoff
- blend weights:
  - geometry=0.75
  - transition=0.25

Definition of Done:
- logits_map produced even with sparse stats (fallback to smoothing / uniform acceptable).

Phase 9 — Constraints Layer (Manifest-Only)
Priority: HIGH

TODO-9.1 Apply constraints post-logit pre-sampling with influence_cap
- apply stage = post_logit_pre_sampling
- influence_cap=0.25
- rules must be loaded from manifest-defined file path; none in source code.

Definition of Done:
- Constraints can modify logits but cannot exceed cap; no semantic origination.

Phase 10 — Sampling (Only Collapse)
Priority: CRITICAL

TODO-10.1 Sampling implementation
- temperature=0.9, top_k=50, top_p=0.92

Definition of Done:
- Sampling is the only collapse point.

Phase 11 — Learning Updates (Observation-Conditional, Additive)
Priority: CRITICAL

TODO-11.1 Update rules (only if semantic_freeze disabled)
Implement:
- basis_strength += reward * projection_weight
- stability = ema(stability, observed_success, stability_ema=0.99)
- vec = ema(vec, geom_proj, vector_ema=0.98) (geom_proj = band_features)
- token_basis_association updates
- pattern_next count update
- bigram count update (reward * 0.25)

Definition of Done:
- No vector corruption; updates are smooth and deterministic given same inputs.

Phase 12 — Semantic Freeze (Not a boolean)
Priority: CRITICAL

TODO-12.1 Semantic freeze state model
- Must include:
  - enabled
  - freeze_timestamp required
  - freeze_state_hash (sha256) with verification on load
  - audit log required
- Freeze prohibits:
  - basis creation
  - basis vector/strength/stability updates
  - transition stats updates
  - external observation integration
  - mastication learning
- Freeze allows inference/projection/decoding/search read-only

Definition of Done:
- Freeze ON: system still generates output but learning paths are no-ops.
- Restart: hash verification prevents tampered semantic state.

Phase 13 — Fallback Hierarchy (Tier-1 blend → Tier-2 ask → Tier-3 web)
Priority: HIGH

TODO-13.1 Implement tier checks
- Tier-1 Concept Blending:
  - Trigger: projection_residual > residual_for_blend(0.45)
  - Constraints: max bases 4, min weight 0.12
- Tier-2 Active Querying:
  - Trigger: logits_entropy > 3.2 AND user_present
  - Rate limit: 1/min
- Tier-3 External Observation (Web):
  - Trigger includes spectral_gradient_band + residual + entropy thresholds AND user answer unavailable
  - Action: web_search then internal_mastication.pipeline

Definition of Done:
- System attempts blend first, then asks user, then (optionally) web.

Phase 14 — Internal Mastication (Web → Token Stream → Wave → Observe → Update)
Priority: HIGH

TODO-14.1 Implement silent reading pipeline
- Non-generative: sampling disabled; no user-visible output
- Steps: normalize -> chunk -> tokenize_mecab -> replay_as_virtual_experience(silent_reading) -> wave update -> spectral observe -> meaning project -> learning_update(allowed targets) -> discard_raw_text
- Budget caps per day: docs/tokens as runtime specifies.

Definition of Done:
- Raw text is discarded post-mastication; only aggregates persist.

Phase 15 — Maintenance (Semantic Sleep, Preemptible)
Priority: MEDIUM → HIGH

TODO-15.1 Sleep scheduler (usage-aware + preemptible)
- Must not modify wave_core.state.psi.
- May update meaning_space via quarantined adoption pipelines; never block inference.

TODO-15.2 Band refinement tasks
- Target weak/unstable/unused bases
- Actions:
  - alias link, EMA merge vec, reinjection boost, demotion to cold tier, optional dormant
- No hard delete; must remain inspectable.

Definition of Done:
- Maintenance runs with CPU/IO caps; pauses on user input; resumes after idle.

Phase 16 — Distributed Sync (Optional, Event-sourced)
Priority: MEDIUM

TODO-16.1 basis_event serialization + signature + replay protection
- canonical semantic state: none
- semantic authority: local_node_only
- signature required ed25519; verification mandatory
- adoption pipeline includes quarantine_buffer -> mastication -> dominance_constraint -> basis_integration

Definition of Done:
- Export/import of derived semantic events works locally.
- distributed_sync remains disabled by default.

Phase 17 — API & Inspectability
Priority: HIGH

TODO-17.1 Endpoints
- POST /generate : run generation pipeline
- POST /freeze : enable/disable semantic_freeze (admin-only)
- GET /meaning_space : list basis metadata (strength, stability, dormant, counts)
- GET /health : show manifest version, freeze hash status

Definition of Done:
- JSON responses valid; inspectability demonstrable.

Phase 18 — Tests (Cold Start & Invariants)
Priority: CRITICAL

TODO-18.1 Cold start (empty DB)
- Empty DB should still tokenize → wave update → observe → create basis → decode → sample output.

TODO-18.2 Invariant tests
- Delete basis prohibited (DB + DAO)
- Labels never used as retrieval keys
- Sampling is only collapse point
- Freeze prohibits learning/mastication integration
- External observation cannot update aggregates without mastication

Success Criteria (Manifest-Compliant)
Wave LLMOS can tokenize input (MeCab), update wave state (phase-hash complex embedding), observe band_features (STFT), project into meaning space (cosine topK), create/update bases additively (no hard delete), decode logits via DB decoder (pattern_next + bigram backoff), apply manifest-only constraints, sample tokens (only collapse), optionally use tiered fallback and mastication, maintain semantic sleep tasks, and expose inspectable APIs—strictly respecting the constitution order and prohibitions.