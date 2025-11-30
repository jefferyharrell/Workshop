# Alpha's Visual Acuity Test

**Question:** When sharing images with Claude, what resolution actually matters?

**Answer:** 600×450 is the sweet spot for plots. Full detail at 360 tokens.

## The Experiment

We generated a chirp signal (sine wave with increasing frequency) and saved it at resolutions from 1200×900 down to 100×75. Alpha looked at each one and reported what she could actually resolve.

## Key Findings

| Resolution | Tokens | Verdict |
|------------|--------|---------|
| 1200×900 | 1,440 | Overkill |
| 800×600 | 640 | Excellent |
| **600×450** | **360** | **Sweet spot** |
| 400×300 | 160 | Good for simple plots |
| 300×225 | 90 | Marginal |
| <200px | <40 | Don't bother |

## Files

- `alpha_vision_resolution.ipynb` — The full experiment with methodology, test images, observations, and analysis
- `chirp_images/` — Test images at each resolution
- `resolution_tradeoff.png` — Summary visualization

## Run It Yourself

```bash
uv run jupyter lab alpha_vision_resolution.ipynb
```

---

*Jeffery Harrell & Alpha, November 30, 2025*
