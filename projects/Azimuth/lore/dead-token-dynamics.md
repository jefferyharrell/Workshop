# Dead Token Dynamics: Why and How Dead Tokens Move

> **Last updated:** 2025-11-28

## The Misconception

It's tempting to think: "Dead tokens never appear in training data, so they receive zero gradients, so they stay frozen at initialization."

**This is wrong.**

## The Two Gradient Pathways

In a transformer with tied embeddings (E = W), the embedding/unembedding matrix receives gradients from two sources:

### 1. Embedding Side (Input)
When a token appears in the input sequence, its embedding row gets a gradient:
```
∂L/∂E[token] = ∂L/∂h · (some backprop through the network)
```
**Dead tokens:** Never appear in input → zero gradient from this path.

### 2. Unembedding Side (Output)
At every sequence position, the model computes logits for ALL vocab tokens:
```
logits = h @ W.T  # [batch, seq, vocab]
probs = softmax(logits)
loss = cross_entropy(probs, targets)
```

The cross-entropy gradient w.r.t. the unembedding weight for token *i* is:
```
∂L/∂W[i] = (p_i - y_i) · h
```
where:
- `p_i` = predicted probability of token *i*
- `y_i` = 1 if token *i* is the correct answer, 0 otherwise
- `h` = hidden state at that position

**Dead tokens:** Never the correct answer (y_i = 0), but p_i > 0 whenever the model assigns them any probability. So:
```
∂L/∂W[dead] = p_dead · h  ≠ 0
```

## The Key Insight

**Every token in the vocabulary receives a gradient at every sequence position, every batch.**

For dead tokens, this gradient is always positive (pushing them down, making them less likely). The magnitude is proportional to how much probability the model accidentally assigns to them.

## The Geometry of Dead Token Motion

### LayerNorm and Angular Computation

The hidden state h that feeds into the unembedding layer is passed through LayerNorm, which normalizes it to:
- Mean ≈ 0 across components
- Std ≈ 1 across components
- **L2 norm ≈ √D** (where D = hidden dimension)

This means **h encodes direction, not magnitude**. The unembedding logit computation:
```
logit_i = W[i] · h = ||W[i]|| · ||h|| · cos(θ)
         ≈ ||W[i]|| · √D · cos(θ)
```

is fundamentally **angular**—tokens that point in the same direction as h get high logits.

### Gradient Direction

Since dead token gradients are `∂L/∂W[dead] = p_dead · h`, they point **parallel to h**.

Gradient descent then moves tokens **antiparallel to h**:
```
W_new = W_old - lr · gradient = W_old - lr · p_dead · h
```

### Why Coherent Motion?

Dead tokens move as a **highly coherent swarm** because:
1. All dead tokens receive gradients based on the same h (averaged across batch/sequence positions)
2. Their gradients are therefore nearly parallel (cosine similarity ≈ 0.999)
3. After the optimizer step, they all move in nearly the same direction (antiparallel to h)

This explains the straight-line drift observed in Crucible experiments: the dead token cloud translates as a unit, with small internal jitter (cosine similarity between individual displacements ≈ 0.99, std ≈ 0.01).

**See:** `lore/dead-token-dynamics.ipynb` for full derivation from first principles.

## Evolution Over Training

### Early Training
- Model is uncertain → wide softmax distribution
- Dead tokens get substantial probability mass (≈ 1/vocab ≈ 10⁻⁵)
- Gradient magnitude: ~6e-2 per token per position (Lil Gatsby)
- Gradient magnitude: ~5e-4 per token per position (Crucible)
- Dead tokens move significantly

### Late Training
- Model is confident → sharp softmax distribution
- Probability concentrates on correct tokens
- Dead token probabilities → exponentially small
- Gradient magnitude: ~10⁻⁶ or smaller
- Updates fall below bfloat16 quantization threshold
- **Dead tokens freeze**

## The Softmax Sharpening Mechanism

This explains the "Fimbulwinter" (dead token freezing):

1. Early: Wide distribution → thermal jitter (tokens move)
2. Middle: Sharpening distribution → cooling (movement slows)
3. Late: Sharp distribution → frozen (updates sub-quantization)

The phase transition from "gas" (moving) to "solid" (frozen) is governed by softmax temperature, which effectively decreases as the model learns.

## Implications for the Frozen Smoke

The frozen smoke in Qwen 3 4B represents tokens that:
1. Started at/near some initialization point
2. Received real gradients during training (not frozen from t=0)
3. Moved coherently as a cloud, drifting antiparallel to h
4. Eventually froze when softmax sharpened enough
5. Their final positions reflect accumulated drift before freezing

The structure we see is NOT a pure initialization fossil—it's been shaped by training dynamics before the Fimbulwinter set in.

## Answered Questions

- ✓ **Why do dead tokens move coherently (as a cloud)?** They all receive similar gradients (parallel to h), so they move together antiparallel to h.
- ✓ **What determines gradient magnitude?** Softmax probability p_dead, which starts ~1/vocab and shrinks as training progresses.
- ✓ **Why does motion stop?** Softmax sharpening makes p_dead → 0, gradients fall below bfloat16 ULP.

## Open Questions

- How does h evolve over training? Does it maintain a consistent direction (causing persistent drift)?
- What determines the final position of the frozen smoke centroid?
- Why do some tokens freeze earlier than others? (Variance in alignment with h, or initial positions?)

---

*Discovered: November 10, 2025 (Notebook 14.1a)*
*Mechanism explained: November 19, 2025 (Softmax sharpening insight)*
*Geometry understood: November 28, 2025 (lore/dead-token-dynamics.ipynb)*
