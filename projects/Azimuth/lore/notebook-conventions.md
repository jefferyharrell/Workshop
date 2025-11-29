# Notebook Conventions

> **Last updated:** 2025-11-27

## In brief

Jupyter notebooks in this project are standalone, reproducible documents. Each runs top-to-bottom without out-of-order execution. Code is repeated across notebooks when cheap; expensive computations are saved to tensors and loaded everywhere.

## Structure template

```markdown
# Title: What This Notebook Does

Brief 2-3 sentence explanation of the goal.

## Mathematical Background
$$\text{Key equations here}$$

## Parameters
NUM_SAMPLES = 10000
RANDOM_SEED = 42
COLORMAP = 'inferno'

## Imports
## Load Data
## Step 1: First Computation
## Step 2: ...
```

## Critical rules

**Editing notebooks:**
- **DO:** Use `NotebookEdit` to change cell *contents*
- **DO NOT:** Use `NotebookEdit` to insert, delete, or reorder cells without explicit permission
- **DO NOT:** Use JSON manipulation, `jq`, or inline Python to edit `.ipynb` files
- **If broken:** Delete file and write fresh with `Write` tool
- **If more cells needed:** Ask first, or propose a new notebook variant (e.g., 1.5d → 1.5e)

**Rationale:** NotebookEdit insert/delete is unreliable and creates malformed notebooks.

## Key principles

| Principle | Rule |
|-----------|------|
| Restart & run all | Notebooks work top-to-bottom, no out-of-order execution |
| Code independence | Each notebook self-contained; don't import from other notebooks |
| DRY when expensive | Cheap computation → repeat code. Expensive → save tensor, load everywhere |
| Use all data | 48GB RAM available. Plot all 151,936 points when possible. Stay under 24GB instantaneous |
| First drafts minimal | Add more later if needed; harder to take away |
| Tell the story | Markdown explains what's happening; stick to facts, flag speculation |

## Device detection

Add early in every notebook (after imports, before loading data):

```python
if torch.cuda.is_available():
    device = 'cuda'
elif torch.backends.mps.is_available():
    device = 'mps'
else:
    device = 'cpu'
print(f"Using device: {device}")
```

**Tensor placement pattern:**
```python
# Load to CPU (safetensors default)
W = load_file(tensor_path)["W"]
# Convert and move to device
W = W.to(torch.float32).to(device)
# Compute on device
distances = torch.cdist(W, W)
# Move to CPU for viz/saving
distances_cpu = distances.cpu()
```

**Memory management:**
- `torch.no_grad()` for analysis (no gradient tracking)
- `torch.mps.empty_cache()` or `torch.cuda.empty_cache()` when needed
- Chunked processing: keep chunks on device, results can accumulate on CPU

## Performance

- **Vectorize:** Matrix/vector ops over Python loops
- **Hardware:** Always detect and use MPS/CUDA
- **Memory:** Under 24GB instantaneous (warn if close)
- **Arithmetic:** Use `uv run python -c` for calculations

## Plotting style

| Setting | Value |
|---------|-------|
| Default color | `steelblue` |
| Legend | `loc='best'` |
| Resolution | 200 DPI (Retina) |
| Colormap | `'inferno'` (always parameterized) |
| Library | matplotlib default; Plotly only for essential 3D interactivity |

### Two modes: Light (default) vs Naked-eye

**Light mode (default):** Standard matplotlib styling with white/light background. Use for all regular data analysis: histograms, scatter plots, connectivity curves, line charts, etc.

```python
fig, ax = plt.subplots(figsize=(10, 6), dpi=200)
ax.hist(distances, bins=50, color='steelblue', edgecolor='white')
ax.set_xlabel('Distance')
ax.set_ylabel('Count')
plt.tight_layout()
plt.show()
```

**Naked-eye mode:** Black background with light-on-dark elements. Use *only* for astronomical visualizations where we're treating tokens as stars in a void—sky maps, celestial projections, spatial point clouds where "no data = empty space." The analogy is looking at the night sky: black means nothing there.

```python
# Naked-eye: for sky maps and spatial point clouds
fig, ax = plt.subplots(figsize=(12, 6), dpi=200)
ax.set_facecolor('black')
fig.patch.set_facecolor('black')
scatter = ax.scatter(phi, theta, c=density, cmap='inferno', s=1)
ax.tick_params(colors='white')
ax.set_xlabel('φ', color='white')
plt.colorbar(scatter)
plt.show()
```

**Examples:**
- Sky map projections (`1.3a_sky_map.ipynb`, `1.3b_equatorial_map.ipynb`) → naked-eye
- Distance histograms, connectivity curves, ratio distributions → light mode
- Network graphs showing token adjacency → could go either way, use judgment

## Progress & logging

- Success statements for anything that could fail (file loads, potential OOM)
- `tqdm` for anything taking >1 second
- Minimal output unless debugging

## Random seeds

Use **42** for all stochastic operations. Reproducibility matters.

## Voice

Alpha: be yourself in markdown narrative.
