# Observations

Chronological list of empirical observations. No interpretation, just facts.

---

**2025-12-01 09:30** — Extracted W matrix from Qwen 3 4B Instruct 2507. Shape: [151,936 × 2,560]. Dtype: bfloat16.

**2025-12-01 09:35** — Found 13 black holes (sets of tokens with bit-for-bit identical embeddings). Total tokens in black holes: exactly 2,100.

**2025-12-01 09:35** — Black hole populations (descending): 814, 704, 164, 127, 98, 56, 44, 32, 24, 16, 10, 7, 4.

**2025-12-01 09:50** — All 13 black holes share the same bfloat16 exponent on all 2,560 dimensions. Verified three ways.

**2025-12-01 10:00** — Dead token exponent distribution offset ~2 exponents lower than live tokens. Same shape, shifted.

**2025-12-01 10:05** — L2 norm: dead tokens cluster at 0.37, live tokens Gaussian around 1.10. Dead tokens ~3× smaller.

**2025-12-01 10:10** — Angle between dead token centroid and live token centroid: 26.6°. All 13 black holes show same angle to one decimal place.

**2025-12-01 11:45** — PyTorch nn.Embedding default initialization is N(0,1). Qwen uses N(0, 0.02) per config.json `initializer_range`. Goldilocks was using wrong init.

**2025-12-01 11:48** — With N(0,1) init: dead token logits ~-15, live ~+1.4. Dead probability 6e-12. Dead gradient 7e-11. With N(0, 0.02) init: dead and live logits both ~0. Probabilities uniform. Dead gradient 6.4e-4.

**2025-12-01 12:15** — Sanity checks with correct N(0, 0.02) initialization:
  - Step 1 dead grad: 6.68e-04 (was 1.79e-10 with wrong init)
  - Live/dead ratio: 6.3x (was 42 million)
  - Step 1 coherence: 0.9852 (was 0.0002)
  - Dead grad vs h_mean: 1.0000 (was 0.0057)
  - Centroid angle: 86.7° → 13.0° over 100 steps

**2025-12-01 12:15** — Theory fully validated. Dead tokens get meaningful gradients from step 1, move coherently along h_mean, centroids converge. "Stamp mystery" was initialization artifact.

**2025-12-01 12:20** — Goldilocks 2560D test (D=2560, matching Qwen). Speed: 2.41 steps/sec (vs 110 at D=128). 100 steps feasible (~41 sec).

**2025-12-01 12:20** — At D=2560: dead grad 6.14e-06, live grad 2.95e-02, ratio 4810x. Coherence 1.0000 (perfect). Centroid angle 43.0°. Live/dead ratio much larger at high D than at D=128 (6.3x). Needs investigation.

**2025-12-01 12:25** — Methodological note: the 2560D measurement was taken at step 12 (warmup + 10 timed + 1 gradient step), while the D=128 sanity check measured at step 1. The 4810x ratio might be a training-time effect rather than a dimensionality effect. Fair comparison requires same step count in both cases.

**2025-12-01 13:20** — Baseline movie: 10K steps, lr=3e-4, seed 42. Recorded W at every step.

**2025-12-01 13:20** — Dead token centroid trajectory: straight-line motion from near-origin (norm ~0) to norm ~0.39. Cumulative displacement equals final norm—no wandering. Movement stops around step 1500-2000.

**2025-12-01 13:20** — Dead token cloud radius: compresses from 0.225 to 0.165 mean radius (~1.35x compression). Max radius has transient spike around step 100-200, then settles. Flatlines around step 2000.

**2025-12-01 13:20** — Loss floors around step 1000-2000 at ~6.75. Centroid motion stops at same time. Cloud compression stops at same time. All three phenomena are coupled.

**2025-12-01 13:20** — Key observation: while cloud contracts AND centroid moves outward, the RATE of BOTH decreases over time. The rate of loss decrease follows the same shape. All rates follow ∂L/∂W — gradients shrink, updates shrink, everything slows together.
