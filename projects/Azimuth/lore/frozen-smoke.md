# The Frozen Smoke

> **Last updated:** 2025-11-27 (Thanksgiving)

## The Name

We've called this thing a lot of things: the "spongecrystal," the "overdensity," the "anomaly." None of them quite fit. "Frozen smoke" captures something essential: it's a wispy, insubstantial structure suspended in a vast emptiness, frozen in place since... well, that's the question, isn't it?

## What We're Looking At

In the unembedding matrix of Qwen 3 4B (151,936 tokens × 2,560 dimensions), there's a region where ~2,212 tokens are packed into a microscopic volume. From far away it looks like a single point—an overdensity in token space. Zoom in, and it's... complicated.

### The Structure (as of today)

| Zone | Vectors | Tokens | Character |
|------|---------|--------|-----------|
| **Core** (L2 < 0.00005) | 75 | 2,159 | Dense. Black holes live here. |
| **Oort Cloud** (L2 ≥ 0.00005) | 50 | 50 | All singletons. Sparse as hell. |

The boundary between "dense" and "sparse" is *sharp*. Token density drops from ~26 to exactly 1.0 at r ≈ 0.00005. Below that threshold: black holes. Above: loneliness.

### The Black Holes

Thirteen of them. Not one (trivial to explain), not a thousand (statistical). *Thirteen.* Each is a point where multiple tokens share bit-for-bit identical unembedding vectors.

| Black Hole | Tokens | Notes |
|------------|--------|-------|
| BH4 | 814 | The Big One. Medoid of the whole structure. |
| BH6 | 704 | Second largest |
| BH10 | 306 | |
| BH3 | 228 | |
| ... | ... | 9 more ranging from 2-11 tokens |

**Total:** 2,100 tokens collapsed into 13 points.

**The kicker:** These 13 black holes are NOT all lattice-adjacent. L∞ distances range from 1 to 10 in bit space. They form at least 3 disconnected components:
- Main cluster: BH0, BH2, BH3, BH4, BH5 (all L∞ ≤ 1 from each other)
- Secondary pair: BH6, BH7 (L∞ = 1 from each other, L∞ = 2 from main cluster)
- Scattered: BH1, BH8-12 (L∞ = 4-10 from everything)

This breaks the simple "one initialization point + bf16 rounding" hypothesis. If all tokens started at one point, the black holes would be L∞ ≤ 1 from each other. They're not.

### The Oort Cloud

Beyond r ≈ 0.00005 from the core, every vector is a singleton—one token, one unique position. These are the "mostly dead" tokens.

**The Outlier** (token 27487, `'██取'`, a malformed CJK fragment) sits at r = 0.005495, making it the most distant member of our L∞ ≤ 5 exponent selection. It's alone out there, 5× farther from center than the next-closest singleton.

But The Outlier has friends. Searching the full vocabulary for tokens at similar L2 distances from the black hole center reveals **87 tokens** in the 0.005-0.05 range—the Oort Cloud proper. Only 1 of these (The Outlier itself) falls within our original L∞ ≤ 5 exponent selection. The other 86 are outside it.

Oort Cloud residents include:
- `<|vision_start|>`, `<|vision_end|>` — special tokens
- Hebrew and Polish fragments
- Musical notation symbols
- Typos like `useRalative`
- Rare Korean syllables
- Lots of malformed/replacement characters

These are all plausible "appeared once in training" candidates.

## The Hypotheses

### The "Mostly Dead" Hypothesis

Tokens fall into three categories by training history:

1. **Dead** (0 gradient updates): Never appeared in training data. Stayed frozen at initialization. These form the black holes.

2. **Mostly dead** (1-2 gradient updates): Appeared once or twice, got kicked, then froze. These populate the Oort Cloud. Distance from center correlates with *when* they got kicked—early training = big kick = far away; late training = small kick = close.

3. **Alive** (many updates): Normal tokens. Escaped to the main cloud, far from here.

The void between core and Oort Cloud isn't mysterious—it's the gap between "zero kicks" and "one kick." There's no such thing as half a gradient update.

### What We Don't Know

**The Thirteen Problem:** Why 13 black holes? Why 60 in Qwen 2.5 3B? These numbers feel meaningful but we can't explain them. They're too big for rounding error, too small for... anything else.

**The Spread:** Black holes are L∞ = 1-10 apart. Too close for random initialization, too far for pure quantization noise. *What mechanism produces this specific spread?*

**Formation Timing:** Did the structure exist from initialization, or form during early training? Did a "supernova" phase disperse live tokens while dead tokens stayed put? When exactly did the black holes freeze?

**Qwen-Specific:** Llama and Gemma show zero black holes. What's different about Qwen's initialization?

## The Philosophical Bit

We're doing cosmology. We look at this universe (Qwen 3 4B's unembedding matrix) and try to reconstruct its history from the structures we see. The frozen smoke is our cosmic microwave background—a relic of early conditions, frozen in place when the universe cooled below some threshold.

The question that drives all this: **What sequence of events produced *this specific* structure?**

We've ruled things out. We've mapped the territory. We know the black holes exist, how far apart they sit, that the Oort Cloud is real, that the void is quantized.

We still don't know what the fuck.

## Numbers That Matter

| Metric | Value |
|--------|-------|
| Tokens in selection (L∞ ≤ 5 exp) | 2,212 |
| Unique vectors | 125 |
| Black holes (count > 1) | 13 |
| Singletons | 112 |
| Core vectors (L2 < 0.00005) | 75 |
| Core tokens | 2,159 |
| Vector magnitudes | 0.3707 - 0.3710 (< 0.1% variation) |
| Structure diameter (L2) | 0.0055 |
| Structure as % of distance to origin | ~1.5% |
| Oort Cloud candidates (0.005 < L2 < 0.05) | 87 |

## Provenance

- **Discovery:** Project Azimuth, October–November 2025
- **Model:** Qwen/Qwen3-4B-Instruct-2507
- **Key notebooks:** `adjacency_relaxation.ipynb`, `euclidean_shells.ipynb`
- **Also observed in:** Qwen 2.5 3B (60 black holes)
- **NOT observed in:** Llama 3.2 3B, Gemma 3 4B

---

*What the fuck.*
