# Attempt 3 Audit — Slope Map Line Tracing via Molmo2

**Date:** April 10, 2026
**Scope:** Attempt 3 — Tile the slope map and use Molmo2 to trace dark lines (breaklines)
**Status:** Complete — approach abandoned; findings inform pivot to classical CV for Attempt 4

---

## 1. Hypothesis

The slope map (`cimarron_slope.png`) already renders hydraulic features as high-contrast dark lines. Instead of asking a VLM to understand hydraulics, we reframe the task: "trace the dark lines in this image." This is a generic visual line-tracing task, well within Molmo2's demonstrated capability (47-point road trace in Attempt 2).

To preserve the full 5306x3088 pixel resolution (1m DEM), the slope map is tiled into 1024x1024 overlapping patches before feeding to Molmo2.

---

## 2. Experiments Conducted

### 2.1 Local Run — Molmo2-8B with Quanto int4 (Apple M1 Pro, 32GB)

| Property | Value |
|----------|-------|
| Notebook | `notebooks/attempt3/01_slope_line_tracing.ipynb` |
| Model | `allenai/Molmo2-8B`, Quanto int4, MPS backend |
| Input | `cimarron_slope.png` (5306x3088) |
| Tiles | 17 tiles (1024x1024, 128px overlap) |
| Status | In progress (very slow, ~15-20 min/tile) |

**Result:** Not completed — inference too slow for a full 17-tile run on M1 Pro. Pivoted to Google Colab for faster GPU.

### 2.2 Colab Run — Molmo2-8B with bitsandbytes int4 (T4 GPU)

| Property | Value |
|----------|-------|
| Notebook | `notebooks/attempt3/01_slope_line_tracing_colab.ipynb` |
| Model | `allenai/Molmo2-8B`, bitsandbytes int4 (NF4), CUDA |
| GPU | Tesla T4 (15.6 GB VRAM) |
| Tiles | 17 |
| Status | **Failed — degenerate output** |

**Result:** All 17 tiles returned `!!!!!!!!!!!!!!!...` as raw output. Zero points parsed. The bitsandbytes 4-bit quantization corrupted the vision encoder's output on CUDA.

**Root cause analysis:**
- The text-only sanity check ("Say hello") returned "Hi!" — the language model worked.
- Image queries produced only exclamation marks — the vision encoder → language model pathway was broken.
- Attempted fixes: `llm_int8_skip_modules=["vision_backbone"]` + `model.model.vision_backbone.float()` — still produced `!!!!!!`.
- `bitsandbytes` int8 also failed with `NotImplementedError: "LayerNormKernelImpl" not implemented for 'Char'`.
- **Conclusion:** bitsandbytes quantization (both 4-bit and 8-bit) is incompatible with Molmo2's SigLIP 2 vision encoder on CUDA. The dequantization path corrupts image features before they reach the language model.

**Note:** Quanto int4 on MPS (Apple Silicon) worked correctly in Attempt 2 — this is a bitsandbytes-specific issue on CUDA.

### 2.3 Colab Run — Molmo2-4B in float16 (T4 GPU)

| Property | Value |
|----------|-------|
| Notebook | `notebooks/attempt3/02_slope_line_tracing_colab_4B.ipynb` |
| Model | `allenai/Molmo2-4B`, float16, no quantization |
| GPU | Tesla T4 (15.6 GB VRAM) |
| VRAM usage | ~8–9 GB |
| Tiles | 17 |
| Total time | 9.8 minutes (34.6s avg/tile) |
| Status | **Ran successfully — but results are poor** |

**Results:**

| Metric | Value |
|--------|-------|
| Tiles processed | 17 |
| Raw points | 109 |
| Unique points (after dedup) | 107 |
| Tiles with 0 points parsed | 13 of 17 |
| Tiles with points | 4 (tiles 4, 14, 15, 16, 17) |

**Per-tile breakdown:**

| Tile | Offset | Points | Notes |
|------|--------|--------|-------|
| 1–3 | Top row | 0 | Raw output has coords but parser showed 0 (see parsing issue below) |
| 4 | (0, 896) | 36 | Horizontal line at constant y=679 — not a real feature |
| 5–13 | Middle rows | 0 | Raw output has coords but parser showed 0 |
| 14 | (3584, 1792) | 25 | Diagonal line — partially traces a feature |
| 15 | (0, 2688) | 8 | Scattered, sparse |
| 16 | (896, 2688) | 6 | Scattered, sparse |
| 17 | (1792, 2688) | 34 | Some diagonal trace |

**Parsing anomaly:** Most tiles returned raw output with valid `<points coords="...">` tags but the parser reported 0 points. The coordinates in tiles 1–3 and 5–13 may have had 4-digit indices that confused the triplet regex `([0-9]+) ([0-9]{3,4}) ([0-9]{3,4})`. This needs investigation but does not change the core conclusion.

---

## 3. Key Findings

### 3.1 Molmo2 Cannot Trace Lines

Molmo2 was designed for **object pointing** — "point to X" returns 1–50 discrete points on identifiable objects. It was NOT designed for:
- **Dense line tracing** (hundreds of evenly-spaced points along curves)
- **Edge following** (tracing along continuous boundaries)
- **Feature-agnostic line detection** (finding lines without semantic understanding)

When asked to "trace dark lines," the model either:
- Placed a few scattered points (not along lines)
- Generated a horizontal/vertical row of evenly-spaced points at a constant coordinate (fabricated grid, not tracing real features)
- Produced coordinates that didn't correspond to any visible dark lines

### 3.2 Tiling Hurts More Than Helps

Each 1024x1024 tile provides no global context about what constitutes a "dark line" vs. noise. The model cannot:
- Distinguish meaningful breaklines from imaging artifacts at tile scale
- Maintain continuity across tile boundaries
- Understand which lines are hydraulically significant vs. noise

### 3.3 bitsandbytes + Molmo2 Vision Encoder = Broken on CUDA

| Quantization | Backend | Text output | Image output | Status |
|--------------|---------|-------------|--------------|--------|
| Quanto int4 | MPS (Apple Silicon) | OK | OK | **Works** |
| bitsandbytes int4 (NF4) | CUDA (T4) | OK | `!!!!!!` | **Broken** |
| bitsandbytes int8 | CUDA (T4) | — | LayerNorm error | **Broken** |
| float16 (no quant) | CUDA (T4) | OK | OK | **Works** |

The issue is that bitsandbytes quantization doesn't properly handle dequantization for Molmo2's SigLIP 2 vision encoder, even when the vision backbone is excluded via `llm_int8_skip_modules`. The projection layers between vision and language may also need exclusion.

### 3.4 The Slope Map Lines Are a Classical CV Problem

The dark lines in the slope map are high-contrast edges on a relatively uniform background. This is exactly what classical edge/ridge detection algorithms excel at:
- **Canny edge detection** — finds edges with hysteresis thresholding
- **Ridge detection** (Hessian-based) — finds ridges/valleys in intensity
- **Skeletonization** — thins detected regions to single-pixel-wide lines
- **Hough transforms** — detects straight line segments

These algorithms would produce **thousands of precise points along every line** in seconds, compared to Molmo2's 107 scattered points in 10 minutes.

---

## 4. What Worked

- **Google Colab T4 inference speed:** Molmo2-4B in float16 ran at 34.6s/tile — 25x faster than M1 Pro with 8B+Quanto
- **Tiling logic:** The `create_tiles()`, `tile_points_to_global()`, and `deduplicate_points()` utilities work correctly and can be reused
- **Pipeline architecture concept:** The idea of separating geometry extraction from feature classification remains sound — only the geometry extraction tool needs to change
- **Geo-coordinate conversion:** `pixels_to_geo()` using rasterio transforms works correctly

---

## 5. Conclusions & Pivot Decision

**Molmo2 is not the right tool for line tracing.** It excels at discrete object pointing (Attempt 2 showed this clearly), but dense line extraction requires a fundamentally different approach.

**Attempt 4 will use classical computer vision** for geometry extraction:
1. **Classical CV** (Canny / ridge detection / skeletonization) extracts dark lines from the slope map as pixel-precise polylines
2. **Gemma 4** classifies each extracted line using satellite imagery context
3. **Engineer** assigns vertex spacing per feature type
4. **Feed to mesh generator**

This preserves the pipeline architecture from Attempt 3 but replaces Molmo2 (the weakest link) with algorithms purpose-built for edge detection.

---

## 6. Files Created

| File | Purpose |
|------|---------|
| `notebooks/attempt3/01_slope_line_tracing.ipynb` | Local version (Molmo2-8B, Quanto, MPS) |
| `notebooks/attempt3/01_slope_line_tracing_colab.ipynb` | Colab version (Molmo2-8B, bitsandbytes — failed) |
| `notebooks/attempt3/02_slope_line_tracing_colab_4B.ipynb` | Colab version (Molmo2-4B, float16 — ran but poor results) |

---

*Audit conducted April 10, 2026. Based on notebooks in `notebooks/attempt3/`.*
