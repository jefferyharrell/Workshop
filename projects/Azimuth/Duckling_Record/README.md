# The Duckling Record

Observations of dead token dynamics in Duckling II.

*"This is not for you."* — House of Leaves

---

## Vision

We have 2,048 dead tokens—ghosts in the embedding matrix, tokens that never appear in training data but still move through weight space. We want to watch them.

**The big question:** What happens to dead tokens over training?

- Do they drift together or apart?
- Do they form structure, or dissolve into noise?
- Do they freeze solid (fimbulwinter), or just get very cold?

---

## What We're Measuring

### Cloud Bulk Properties

| Metric | What it tells us |
|--------|------------------|
| **Centroid position** | Where is the dead token cloud? Does it drift? |
| **Cloud radius** | Is the cloud expanding, contracting, or stable? |
| **Centroid velocity** | Is drift accelerating, decelerating, or constant? |
| **Cloud shape** | Spherical? Elongated? Shearing? Twisting? |

### Per-Token Dynamics

| Metric | What it tells us |
|--------|------------------|
| **Displacement ΔW** | How far did each token move? (continuous, float32) |
| **Lattice displacement ΔW′** | How many bf16 cells did it cross? (discrete, ULP units) |
| **Velocity over time** | Cooling curve—are tokens slowing down? |
| **Coherence** | Are tokens moving in the same direction? |

### Lattice-Scale Analysis

Using the framework from `lore/lattice-scale.md`:

| Regime | Condition | Meaning |
|--------|-----------|---------|
| **Teleporting** | L2 > √128 ≈ 11 | Multi-cell jumps, continuous physics dominates |
| **Lattice neighbor** | L∞ = 1 | Moved to adjacent cell (including diagonal) |
| **Face-adjacent** | L1 = 1 | Single dimension, single cell hop |
| **Frozen** | L1 = 0 | No movement at all |

**The fimbulwinter test:** Plot the fraction of tokens in each regime over time.
- Classic fimbulwinter: teleporting → neighbor → frozen (everyone freezes)
- Cold-not-frozen: teleporting → neighbor → *stays neighbor* (eternal jitter)

---

## Immediate Experiments

### 01: First Recording

Record W every N steps during a 10K training run. Compute:
- Dead token centroid at each timestep
- Dead token cloud radius at each timestep
- Per-token displacement magnitudes

**Output:** Basic trajectory visualization. Does the cloud move? Does it shrink?

### 02: Lattice Regime Census

For each timestep, classify every dead token's displacement into lattice regimes. Plot regime fractions over time.

**Output:** The cooling curve. When does teleporting stop? Does freezing happen?

### 03: Coherence Analysis

Are dead tokens moving in the same direction? Compute pairwise cosine similarity of displacement vectors.

**Output:** Coherence over time. Do they move as a swarm or independently?

---

## Links

- **Platform:** `../Duckling_II/` (model, dataset, template)
- **Theory:** `../lore/dead-token-dynamics.md`, `../lore/lattice-scale.md`
- **Original observation:** `../Qwen_3_4B_Anomaly/` (frozen smoke discovery)

---

*The house is not the house. The record is not the record. The duck is definitely a duck.* 🦆
