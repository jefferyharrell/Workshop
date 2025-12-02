# Duckling II

A small, fast language model for studying dead token dynamics.

---

## Quick Start

**To run experiments:** Copy `07_experiment_template.ipynb` and add your code.

**What you get:**
- ~75 steps/sec on M4 Pro
- 10K steps in ~2.3 minutes
- Loss ~3.7 at 10K steps, still learning
- Generates coherent TinyStories-style sentences

---

## The Model

| Property | Value |
|----------|-------|
| Architecture | GPT2LMHeadModel (HuggingFace) |
| Layers | 2 |
| Hidden dim | 128 |
| Attention heads | 2 |
| FFN dim | 512 |
| Context length | 512 |
| Parameters | ~1.5M |
| Precision | bf16 (mixed) |

## The Vocabulary

| Property | Value |
|----------|-------|
| Total vocab | 8,192 |
| Live tokens | 6,144 (IDs 0-6143) |
| Dead tokens | 2,048 (IDs 6144-8191) |
| Coverage | 99.5% of TinyStories |

Dead tokens never appear in training data. They exist only as rows in the embedding matrix W, dragged through weight space by indirect gradients. These are our experimental subjects.

**W matrix:** 8,192 × 128 = 1,048,576 params = 2.0 MB at bf16

## Training Defaults

| Parameter | Value |
|-----------|-------|
| Batch size | 2 |
| Learning rate | 1e-4 |
| Weight decay | 0.01 |
| Optimizer | AdamW |
| Scheduler | Linear decay |
| Seed | 42 |

---

## Files

| File | Purpose |
|------|---------|
| `07_experiment_template.ipynb` | **Start here.** Copy for new experiments. |
| `tokenized_dataset/` | Pre-chunked TinyStories (921K sequences × 512 tokens) |
| `token_mapping.json` | Vocabulary mapping (GPT-2 ↔ compact IDs) |

### Development Notebooks (for reference)

| Notebook | What it does |
|----------|--------------|
| 01 | Token frequency analysis → minimum vocab size |
| 02 | Build tokenizer and dataset |
| 03 | Verify training works with bf16 |
| 04 | Layer sweep (2, 4, 8) → 2 layers wins |
| 05 | Batch size sweep (1-64) → 2 wins on speed |
| 06 | Long run (10K steps) + generation test |

---

## Key Lessons

Things we learned the hard way:

1. **bf16 requires proper mixed precision.** Use `bf16=True` in TrainingArguments, NOT `model.to(bfloat16)`. The latter puts optimizer states in bf16, causing Adam's variance to underflow. Training floors at loss ~5.8.

2. **Batch size 2 is fastest** (~75 steps/sec) while still learning. Larger batches learn better per step but slower overall.

3. **2 layers is enough.** Sweep showed 2-8 layers all reach similar loss; 2 is 2x faster than 8.

4. **Use HuggingFace Trainer.** Hand-rolled training loops have hidden failure modes.

---

## Experimental Platform

Duckling II is optimized for *observing dynamics*, not for *quality*. The model generates repetitive text under greedy decoding—that's fine. What matters is:

- Live tokens receive meaningful gradient updates
- Dead tokens get dragged along by weight decay and indirect gradients
- Training is fast enough to iterate quickly
- The embedding matrix W is small enough to record densely

**Recording budget (24 GB):**
- W snapshots per GB: 512
- Max steps at every-step recording: 12,288
- At 75 steps/sec: ~2.7 minutes of training

---

*"The duck quacks."* 🦆
