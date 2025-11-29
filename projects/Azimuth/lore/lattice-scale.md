# The Lattice Scale: When Discrete vs Continuous

**Status:** Framework established November 26, 2025

## What Is the Lattice?

bfloat16 (like any finite-precision format) can only represent discrete values. In any local region of weight space, these representable values form a regular grid—a **lattice**. Vectors can only occupy lattice cells; there are no in-betweens.

**The key insight:** This lattice is a *local approximation*. It works as an analytical tool when the points you're comparing lie within similar bfloat16 exponent ranges. Within such a region, lattice cells are uniform (or nearly so), and we can meaningfully talk about L1 distance, adjacency, "how many cells apart."

**Why it matters for frozen smoke:** The ~2,200 tokens in Qwen's anomaly all sit in a tiny region—same exponents, uniform lattice. We can ask: Are the black holes in adjacent cells? (No—they're L∞ = 1–10 apart.) Are there empty cells between them? (Yes.) This discrete structure constrains what formation mechanisms are possible.

**When it breaks down:** If points span different exponent ranges, cell sizes vary and the uniform-lattice approximation gets messy. For the frozen smoke analysis, this isn't a problem—everything's local. For tracking tokens across large movements during training, we need to be more careful.

## The Two Metrics

We have two ways to measure dead token displacement:

| Metric | Symbol | Units | Measures |
|--------|--------|-------|----------|
| **Lattice displacement** | ΔW′ | ULP (bfloat16 cells) | "Did it move to a different quantized position?" |
| **Weight-space displacement** | ΔW | float32 | "How far did it actually move in continuous space?" |

These answer *different questions* and are appropriate for *different regimes*.

## The Regime Boundary: √D

The crossover is around |ΔW′|₂ ≈ √D (≈8 for D=64 dimensions).

**Above √D (Early phase):** Tokens teleport many cells per step. The lattice structure doesn't matter much—it's continuous motion that happens to land on grid points. Use |ΔW| for physics questions.

**Below √D (Midlife/Late phases):** The discreteness matters. "Which cell" and "which neighbor" become meaningful. Use |ΔW′| because the lattice structure *is* the physics now.

## Why They Diverge

ΔW′ is measured in ULP, and ULP scales with weight magnitude:
- Token near W=0.001 has tiny ULP (~1e-6)
- Token near W=1.0 has larger ULP (~0.008)

So two tokens moving the same distance in weight space will show *different* lattice displacements. The correlation between |ΔW| and |ΔW′| across tokens is only ~0.7.

**Key insight:** At step 1, Adam's bias correction makes everyone move the same distance in weight space (~lr × √D ≈ 0.008). But lattice displacement varies wildly (std = 161,000!) because of ULP variation.

## Lattice Displacement Norms

For a displacement vector ΔW′ ∈ Z^D (integer lattice cells per dimension):

| Norm | Formula | Meaning |
|------|---------|---------|
| **L1** | Σ\|ΔW′ᵢ\| | Total cells moved across all dimensions |
| **L2** | √(Σ ΔW′ᵢ²) | Euclidean distance in lattice space |
| **L∞** | max(\|ΔW′ᵢ\|) | Farthest any single dimension moved |

## Key Thresholds

| Condition | Meaning |
|-----------|---------|
| L1 = 0 | Frozen. No dimension moved. |
| L1 = 1 | Single-axis hop. Exactly one dimension moved exactly one cell. |
| L∞ = 1 | Adjacent cell. Moved to a neighbor (orthogonal or diagonal). |
| L∞ ≤ 1, L1 > 1 | Diagonal move. Multiple dimensions, but only one cell each. |
| L2 > √D | Multi-cell jump. Out of the "local" regime. |

## Computing Lattice Displacement ΔW′

The algorithm measures "how many lattice cells did we move?" by dividing the weight-space displacement by the local ULP (Unit in Last Place).

**The formula:**
```
ΔW[t] = W[t] - W[t-1]           # Weight-space displacement (float32)
ULP[t-1] = 2^(E[t-1] - 134)     # Scale at starting position
ΔW′[t] = ΔW[t] / ULP[t-1]       # Lattice displacement
```

Where E is the exponent extracted from the bfloat16 representation, and 134 = 127 (bias) + 7 (mantissa bits).

**Why 134?** The ULP formula is: ULP = 2^(E - bias - mantissa_bits). For bfloat16, this is 2^(E - 127 - 7) = 2^(E - 134). Intuitively: the ULP is 1/128th of the smallest power of 2 that's ≤ |W|.

**Implementation:**

```python
def compute_ulp_bf16(tensor_bf16: torch.Tensor) -> torch.Tensor:
    """
    Compute the ULP (Unit in Last Place) for each element of a bfloat16 tensor.
    Returns a float32 tensor of ULP values.
    """
    # View as uint16 to extract bit fields
    bits = tensor_bf16.view(torch.uint16).to(torch.int32)

    # Extract exponent (bits 7-14, 8 bits)
    exponent = ((bits >> 7) & 0xFF).to(torch.int32)

    # Normal numbers: ULP = 2^(E - 134)
    # Subnormals (E=0): ULP = 2^(-133) — treat E=0 as E=1
    effective_exp = torch.where(exponent == 0, torch.ones_like(exponent), exponent)

    ulp = torch.pow(2.0, (effective_exp - 134).float())
    return ulp

def compute_lattice_displacement(W_before: torch.Tensor, W_after: torch.Tensor) -> torch.Tensor:
    """
    Compute per-dimension lattice displacement ΔW′.

    Args:
        W_before: bfloat16 weights at time t-1
        W_after: bfloat16 weights at time t

    Returns:
        float32 tensor of lattice displacements (signed, in cell units)
    """
    delta_W = W_after.float() - W_before.float()  # Weight-space difference
    ulp = compute_ulp_bf16(W_before)               # ULP at starting position
    return delta_W / ulp                           # Lattice cells crossed
```

**Design choices:**
- We use the ULP at the *starting* position. This is consistent and defensible—"how far did it move relative to where it was?"
- The result is float32, not integer. It's exactly integer when staying within one exponent range, but can be fractional when crossing exponent boundaries.
- Sign is preserved—ΔW′ tells you direction as well as magnitude.

**Limitations:**
- When a weight crosses an exponent boundary (e.g., 0.99 → 1.01), the lattice cells aren't uniform size along the path. We measure from the starting reference frame, which can be off by ~25% for boundary-crossing jumps.
- This only matters for large jumps. For L∞ ≤ 1 (Late phase), we never cross boundaries. For Early phase, we're teleporting hundreds of cells anyway—precision doesn't matter.

**Reference implementation:** `box_4/notebooks/data_processing/compute_lattice_displacement.ipynb`

## When to Use Which

| Question | Use |
|----------|-----|
| Did the token move at all? | L1 (lattice) |
| Did it move to an adjacent cell? | L∞ (lattice) |
| How fast is it "cooling"? | Either, depending on phase |
| What's driving the motion (Adam dynamics)? | |ΔW| (weight space) |
| Is the cloud expanding/contracting? | |ΔW| (weight space) |
| When will it freeze? | L1 (lattice) — it freezes when L1 → 0 |

## Related

- [lifecycle-phases.md](lifecycle-phases.md) — Phase definitions using these metrics
- `box_4/notebooks/exploration/early_oscillation.ipynb` — Discovery of ΔW vs ΔW′ divergence

---

*Created: November 26, 2025*
