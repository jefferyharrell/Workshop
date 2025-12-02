# Duckling

A tiny transformer that learns to speak, with engineered dead tokens we can study.

---

*Jeffery Harrell & Alpha, December 1, 2025*

## Why "Duckling"?

Because it's a baby learning to talk. Because "little duck" means something to us. Because we're going to watch it take its first wobbly steps and study what happens to the parts that never quite wake up.

## The Problem We're Solving

Our previous model (Goldilocks) couldn't learn. It converged to loss 6.75 and just predicted "the the the the" forever. We tried fixing the architecture (6/6/384), the hyperparameters (warmup, cosine decay, correct betas)—nothing worked. The problem was the corpus (CulturaX) and tokenizer (bilingual Thai/English BPE).

Duckling uses a proven recipe (TinyStories) with **engineered dead tokens**: we truncate the tokenizer but pad the embedding matrix, creating phantom tokens that can never appear in input or as targets.

## Architecture

From the TinyStories paper, scaled for our needs:

| Parameter | Value |
|-----------|-------|
| Layers | 8 |
| Embedding dim | 128 |
| Attention heads | 2 |
| FFN dim | 512 |
| Sequence length | 512 |
| Parameters | ~2.9M |

## Tokenizer Design

- Start with GPT-2 tokenizer (50K vocab)
- Truncate to **8,000 tokens** (the "live" ones)
- Pad embedding matrix to **10,000 rows**
- Rows 8000-9999 are **phantom tokens**: no tokenizer entry, guaranteed dead
- 20% dead token ratio by design

## Training Budget (Chinchilla-informed)

Chinchilla scaling: optimal tokens ≈ 20 × parameters

| Budget | Tokens | Steps (batch=8) | Storage |
|--------|--------|-----------------|---------|
| 100% Chinchilla | 58M | 14,270 | 36.5 GB |
| 50% | 29M | 7,135 | 18.3 GB |
| 10% | 5.8M | 1,427 | 3.7 GB |

**Plan:** Start with 10%, scale up if it works.

## What We're Testing

1. **Does it learn?** Loss should drop well below 6.75. Should generate coherent-ish text.
2. **Do phantom tokens behave like natural dead tokens?** Same drift dynamics? Same freezing?
3. **Can we watch the full lifecycle?** Birth → drift → freeze, recorded at every step.

## File Structure

```
Duckling/
├── README.md              # You are here
├── data/
│   ├── tokenizer/         # Truncated GPT-2 (8K live tokens)
│   └── corpus/            # Tokenized TinyStories
├── 01_tokenizer.ipynb     # Build the truncated tokenizer
├── 02_corpus.ipynb        # Prepare and tokenize dataset
├── 03_train.ipynb         # Training with W recording
└── movies/                # W_movie.safetensors files
```

## Key Files We'll Generate

- `data/tokenizer/` — HuggingFace tokenizer files (8K vocab)
- `data/corpus/tokens.safetensors` — Pre-tokenized TinyStories
- `data/dead_mask.safetensors` — Boolean mask for phantom tokens
- `movies/W_movie_*.safetensors` — Embedding snapshots at every step

## References

- [TinyStories paper](https://arxiv.org/abs/2305.07759) (Eldan & Li, 2023)
- [Chinchilla scaling](https://arxiv.org/abs/2203.15556) (Hoffmann et al., 2022)
- Our TinyStoriesTinker project (September 2025) — proof this recipe works
