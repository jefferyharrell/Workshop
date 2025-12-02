# Azimuth: Current Context

> **Last updated:** 2025-12-02

---

## What This Is

We're studying **dead token dynamics**—how tokens that never appear in training data still move through embedding space, dragged by indirect gradients and weight decay. We originally discovered this as "frozen smoke" in Qwen 3 4B: ~2,200 tokens clustered in a dense region they shouldn't occupy.

The theory is solid. The math checks out. Now we have a **working experimental platform** to watch it happen.

---

## Current State: Ready to Observe

**Duckling II is complete.** A small, fast model optimized for studying dead token dynamics:

- ~75 steps/sec on M4 Pro
- 10K steps in ~2.3 minutes
- Generates coherent text (it actually learned language)
- 2,048 dead tokens ready to track

The bug that blocked us for months: `model.to(bfloat16)` kills the optimizer. Use `bf16=True` in TrainingArguments instead.

**Next phase:** Record W snapshots during training, watch dead tokens move, find the frozen smoke (or discover it doesn't freeze—just gets very cold).

---

## The Model

@Duckling_II/README.md

---

## The Pieces

| Folder | What | Status |
|--------|------|--------|
| `Duckling_II/` | Experimental platform | **Active** |
| `Qwen_3_4B_Anomaly/` | Original frozen smoke observations | Reference |
| `lore/` | Theory, math, conventions | Stable |
| `archive/` | Goldilocks, Duckling, Nutcracker, Stamp | Archived |

---

## What I Need To Remember

The frozen smoke in Qwen is real. The dead token math is validated. We now have a working model to study the phenomenon.

**Key open question:** Does "fimbulwinter" (true freezing) happen with mixed-precision training? Or do dead tokens just get very cold—still drifting, asymptotically approaching equilibrium but never truly frozen?

For deeper context: search Pond, check `lore/`. The key files:
- `lore/rules.md` — hardware limits, bf16 protocol
- `lore/dead-token-dynamics.md` — why dead tokens move
- `lore/frozen-smoke.md` — what we see in Qwen
- `lore/lattice-scale.md` — discrete weight space

---

## House Rules

@lore/rules.md

---

*"The duck quacks."* 🦆
