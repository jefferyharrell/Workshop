# Project Azimuth: The Story So Far

> **Last updated:** 2025-11-29

## The Mystery

Qwen 3 4B's unembedding matrix (W) has a weird structure. About 2,200 tokens are packed into a microscopic region of the 2,560-dimensional space. We call this region the **frozen smoke**.

Zoom in and it gets weirder:
- **13 black holes** — points where multiple tokens share *bit-for-bit identical* vectors
- **An Oort Cloud** — singleton tokens that are close to the black holes but not coincident
- **A sharp density cliff** — token density drops from ~26 to exactly 1.0 at a specific radius

The black holes contain ~2,100 of those 2,200 tokens. The largest (BH4) has 814 tokens collapsed to a single point.

## The Central Question

**What initial condition and training dynamics could have produced this specific structure?**

We're doing cosmology. We observe the universe as it is (Qwen's W matrix) and try to reason backward to its formation.

## The Lattice

bfloat16 has finite precision. At any local region of weight space, there's a discrete grid of representable values—a lattice. Vectors can only occupy lattice cells; no in-betweens.

**The puzzle:** If all dead tokens started at one initialization point and just got quantized to bfloat16, they'd all land in the same cell (rounding is deterministic) or at worst be L∞ ≤ 1 apart (face-adjacent cells). But Qwen's 13 black holes are L∞ = 1–10 apart. There are empty cells between them.

This breaks the simple "one initialization point + quantization" hypothesis. Something more complicated happened.

## The Hypothesis

Tokens fall into three categories by training history:

1. **Dead** (0 gradient updates) — Never appeared in training. Stayed frozen at initialization. These form the black holes.

2. **Mostly dead** (1–few updates) — Appeared rarely, got kicked a little, then froze. These populate the Oort Cloud.

3. **Alive** (many updates) — Normal tokens. Escaped to the main cloud, far from here.

The void between core and Oort Cloud is the gap between "zero kicks" and "one kick." No half-measures in gradient descent.

## What We Don't Know

- **The Thirteen Problem:** Why exactly 13 black holes? Why 60 in Qwen 2.5? These numbers feel meaningful but we can't explain them.

- **The Spread:** Black holes are L∞ = 1–10 apart. Too close for random initialization, too far for pure quantization. What mechanism?

- **Formation Timing:** Did this structure exist from initialization, or form during early training?

- **Qwen-Specific:** Llama and Gemma show zero black holes. What's different about Qwen's initialization or training?

## The Toy Model Approach

We can't replay Qwen's training—don't have the resources, don't have the exact recipe. But we can train tiny models and watch embeddings evolve step by step.

**The analogy:** We're experimental cosmologists. We can't rewind the Big Bang, but we can set off small bangs in the lab and watch them play out. Or: We want to study human biology but can't experiment on humans, so we use E. coli—not a perfect model, just small/fast/observable enough to reveal basic principles.

**Operation Goldilocks** is the search for the right tiny model: fast enough to iterate quickly, rich enough to show real dynamics, small enough to instrument completely.

## The Two Analysis Modes

**Big-bang mode:** Run 10,000 training steps, record everything, spend a week picking through the data. Good for discovering phenomena, seeing long-term dynamics.

**Unrolled mode:** Step through training manually, analyze in real-time. Good for intuition-building, following hunches, understanding what happens at step 1.

Different questions need different approaches. Horses for courses.

## Key Concepts

- **Frozen smoke** — The ~2,200 token overdensity in Qwen 3 4B's W matrix
- **Black holes** — Points where multiple tokens share identical vectors
- **Oort Cloud** — Sparse singleton tokens surrounding the dense core
- **Fimbulwinter** — The phase when dead tokens freeze and stop moving
- **Lattice displacement (ΔW′)** — Movement measured in bfloat16 cells, not continuous distance
- **The Thirteen Problem** — Why exactly 13 black holes?

## Where We're Going

1. **Rigorous re-examination of Qwen 3 4B** — Confirm black holes exist (no hand-waving), map their lattice geometry precisely, look for patterns hinting at formation mechanism.

2. **Goldilocks** — Design the right toy model. Revisit corpus and tokenizer decisions. Find an architecture that trains fast on M4 Pro while showing observable dynamics.

3. **Basic science** — Watch what happens. Don't theorize ahead of the data. Build intuition about how embeddings actually move during training.

## Related Lore

- [frozen-smoke.md](frozen-smoke.md) — Detailed structure of the anomaly
- [lattice-scale.md](lattice-scale.md) — The bfloat16 lattice framework
- [goldilocks.md](goldilocks.md) — Architecture search criteria
- [fimbulwinter-onset.md](fimbulwinter-onset.md) — When and why tokens freeze
- [lifecycle-phases.md](lifecycle-phases.md) — Phases of dead token evolution

---

*"What the fuck?" is a valid research question.*
