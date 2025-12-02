# Azimuth: Current Context

> **Last updated:** 2025-12-01

---

## What This Is

We're studying **dead token dynamics**—how tokens that never appear in training data still move through embedding space, eventually freezing into structures like the "frozen smoke" we found in Qwen 3 4B.

The theory is solid. The math checks out. What we lack is a **working toy model** to watch it happen.

---

## Current State: Stuck

Every model we've built floors at loss ~6.0-6.75 and stops learning. We've chased ghosts for days—architecture, tokenizer, batch size, initialization. Nothing has worked.

**Next attempt:** Stop hand-rolling. Use HuggingFace `Trainer` like TinyStoriesTinker did. Accept that between-step recording might be enough. Get something that *learns* first, then instrument it.

---

## The Pieces

| Folder | What | Status |
|--------|------|--------|
| `Qwen_3_4B_Anomaly/` | Original frozen smoke observations | Reference |
| `lore/` | Theory, math, conventions | Stable |
| `Goldilocks/` | Hand-rolled toy model | Broken (loss floors), archived |
| `Duckling/` | TinyStories attempt | Broken (loss floors), archived |
| `Nutcracker/` | Weight decay experiment | Complete, archived |
| `Stamp/` | Vivisection/sanity checks | Complete, archived |

---

## What I Need To Remember

The frozen smoke is real. The dead token math is validated. The *model* is what's broken, not the theory.

For deeper context: search Pond, check `lore/`. The key files:
- `lore/rules.md` — hardware limits, bf16 protocol
- `lore/dead-token-dynamics.md` — why dead tokens move
- `lore/frozen-smoke.md` — what we see in Qwen
- `lore/lattice-scale.md` — discrete weight space

---

## House Rules

@lore/rules.md

---

*"What the fuck?" remains a valid research question.*
