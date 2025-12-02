# Operation Stamp

> *The supernova, step by step.*

**Status:** In Progress (November 30, 2025)

## The Question

What happens in the first 10-100 steps of training? We call this the "supernova phase"—rapid early motion before tokens settle into their long drift toward freezing.

Specifically:
- Do dead tokens **tighten** into a coherent swarm, or **spread** apart?
- How does the centroid of the dead token cloud move?
- When does the population transition from "flying through space" (L2 > √D) to "settling into the lattice" (L2 ≤ √D)?

## The Hypothesis

Dead tokens start randomly distributed around the origin (N(0, 0.02) initialization). At each step, they all receive gradients proportional to h (the mean hidden state). If h is consistent step-to-step, they get pushed in the same direction—and the cloud **tightens** as it moves.

Prediction: The centroid starts near 0, then moves significantly (maybe ~0.005) after just one step. Tokens on the "front" of the cloud (already pointing in the direction of motion) get pushed further ahead; tokens on the "back" get pushed toward the center. Angular spread decreases. Cosine similarity to the centroid increases.

## The Math

At step 1, Adam's bias correction makes the update magnitude uniform:
```
m̂₁ = g,  v̂₁ = g²
update = η · g/|g| = η · sign(g)
```

Everyone moves exactly η in the direction of their gradient's sign. For dead tokens, that gradient is proportional to h. So they all move ~parallel, but from different starting positions.

## Notebooks

| # | Notebook | Question |
|---|----------|----------|
| 01 | `01_first_ten_steps.ipynb` | What literally happens in steps 1-10? |

## Key Quantities

| Symbol | Meaning |
|--------|---------|
| √D | Regime boundary (~11.3 for D=128). Above = flying, below = settling. |
| centroid(t) | Mean position of dead token cloud at step t |
| cos(Wᵢ, centroid) | How aligned is token i with the cloud's center? |

---

*Conceived: November 30, 2025 (scotch was involved)*
