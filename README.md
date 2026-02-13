# we

A small model weight viewer for Hugging Face checkpoints that avoids downloading too much:

<p align="center">
  <img src="docs/assets/deepseek_example.png" alt="Screenshot of we listing DeepSeek-R1 tensors" width="542" />
</p>

Today it focuses on model introspection. Next steps are compression experiments (TTNN `bfp8`/`bfp4`/`bfp2`) and quality metrics (`pcc`, `rtol`, `atol`) against original weights.

## Quick Start

```bash
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
```

Run the tool:

```bash
./we Qwen/Qwen3-0.6B
```

## Usage

```bash
we [repo_or_url] [filter_query] [--revision REVISION]
```

Examples:

```bash
# Full model tree
./we Qwen/Qwen3-0.6B

# Substring filter (includes matching module + all descendants)
./we Qwen/Qwen3-0.6B mlp

# Torch-style exact path filter
./we Qwen/Qwen3-0.6B model.layers.1

# URL input also works
./we https://huggingface.co/Qwen/Qwen3-0.6B/tree/main

# Specific branch/tag/commit
./we Qwen/Qwen3-0.6B --revision main
```

## Output Notes

- Repeated numeric modules collapse into ranges when possible, e.g. `layers (0-27)`.
- Mixed structures (for example early dense layers then MoE layers) collapse by contiguous matching runs.
- F8 pair pattern (`weight` + `weight_scale_inv`) is rendered as a single logical row and hides `weight_scale_inv`.
- Percentages account for collapsed multiplicity (so `%` reflects true total contribution).

## Caching

`we` uses a local metadata cache so repeated runs are fast without downloading full weights.

- Cache directory (default): `~/.cache/we/metadata`
- Override cache location: `WE_CACHE_DIR`
- Automatic refresh: based on remote revision identity (`sha`), with a 1-hour TTL on revision checks

This means:

- repeated calls within 1 hour are fully local
- after 1 hour, `we` checks remote revision and refreshes only if changed

## Color

`we` uses 256-color pastel output in terminals that support it.

- Disable colors: `NO_COLOR=1`
- Force colors: `FORCE_COLOR=1`

## Project Layout

- `we` - executable CLI script
- `requirements.txt` - Python dependencies
- `docs/assets/deepseek_example.png` - README screenshot
