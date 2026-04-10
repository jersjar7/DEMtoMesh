# Attempt 2 Audit — VLM Satellite Imagery Findings

**Date:** April 10, 2026
**Scope:** Attempt 2 — Evaluate 4 vision-language models on satellite imagery of the Cimarron River
**Status:** Complete — findings inform pipeline pivot for Attempt 3

---

## 1. Overview

Four VLMs were tested on satellite imagery of the Cimarron River to determine which models can identify and spatially locate hydraulic features (channel, meanders, point bars, vegetation boundaries, roads, bridges). Each model was evaluated in a dedicated notebook under `notebooks/attempt2/`.

---

## 2. Model Results

### 2.1 Gemma 4 — Notebook 01

| Property | Value |
|----------|-------|
| Model | `google/gemma-4-26b-a4b-it` via OpenRouter API |
| Capability | Rich qualitative text descriptions |

**Results:** Excellent. Described channel geometry, vegetation patterns, and infrastructure accurately. Best at understanding what features exist and their spatial relationships.

**Limitation:** Zero spatial output. No coordinates, no bounding boxes, no masks. Purely textual.

**Best use:** Scene understanding, feature classification, enriching prompts for other models.

### 2.2 Molmo 2 — Notebook 02

| Property | Value |
|----------|-------|
| Model | `allenai/Molmo2-8B` via HuggingFace Transformers (Quanto int4, MPS) |
| Capability | Point coordinates from natural language prompts |
| Coordinate format | `<points coords="idx x y idx x y...">` on a 0--1000 scale |

**Results by feature:**

| Feature | Points returned | Quality |
|---------|----------------|---------|
| Channel | 1 | Single point on the river — minimal for a broad feature |
| Channel bends | 13 | Traced the meander path — excellent |
| Roads | 47 | Traced the east-west road — best result across all models |
| Vegetation boundaries | 7 | Forest/grassland transitions identified |
| Point bars | 9 | Sandy deposits along inner bends |
| Bridges | 2 | Exactly where the road crosses the river |

**Limitation:** Inference is slow (~15 min/query on M1 Pro). Image squeezed to 1024px loses resolution.

**Best use:** Spatial pointing. The only model tested that returns usable coordinates.

### 2.3 Qwen2.5-VL — Notebook 03

| Property | Value |
|----------|-------|
| Model | `qwen/qwen2.5-vl-72b-instruct` via OpenRouter API |
| Capability | Bounding box grounding |

**Results by feature:**

| Feature | Quality | Notes |
|---------|---------|-------|
| Sandbars | Good | 2 compact, well-placed boxes |
| Channel | Moderate | 3 boxes trace the path but very loose — includes floodplain |
| Vegetation | Mixed | Grassland/agriculture boxes OK; "forests" box covers 73% of image |
| Roads | Poor | Single full-height vertical strip for a diagonal road |
| Bridges | Poor | Nearly identical to the roads bbox |

**Limitation:** Bounding boxes are too loose for most features. Cannot handle diagonal linear features.

**Best use:** Coarse region identification for features with clear, compact boundaries (e.g., sandbars).

### 2.4 Florence-2 — Notebook 04

| Property | Value |
|----------|-------|
| Model | `florence-community/Florence-2-large` (local, 777M params) |
| Capability | Object detection, phrase grounding, dense captioning |

**Results:** Complete failure. Every bounding box covered ~99% of the image area. Object detection labeled the entire scene as "animal." Dense captioning returned nothing.

**Root cause:** Florence-2 was trained on COCO/natural photos. Overhead satellite imagery is completely out-of-distribution.

**Best use:** None for this project. Dropped from pipeline.

---

## 3. Comparison Table

| Criterion | Gemma 4 | Molmo 2 | Qwen2.5-VL | Florence-2 |
|-----------|---------|---------|-------------|------------|
| Output type | Text | Point coords | Bounding boxes | Bboxes / captions |
| Spatial precision | None | High (point-level) | Low (loose boxes) | None (full-image) |
| Scene understanding | Excellent | N/A | Moderate | Failed |
| Linear features (roads) | Describes well | 47-point trace | Single strip | Failed |
| Compact features (sandbars) | Describes well | 9-point trace | Good boxes | Failed |
| Inference speed | Fast (API) | ~15 min/query (local) | Fast (API) | Fast (local) |
| Satellite imagery support | Yes | Yes | Partial | No |
| Keep in pipeline? | Yes | Yes | Maybe (sandbars only) | No |

---

## 4. Key Insight — Pipeline Pivot for Attempt 3

The slope map (`cimarron_slope.png`) already renders all hydraulic features as dark lines — channel banks, road embankments, terrace edges. Instead of asking VLMs to "understand hydraulics" from satellite imagery, we can ask Molmo 2 to simply "trace the dark lines" in the slope map. This reframes the problem from domain-specific scene understanding to generic visual line-tracing, which is well within Molmo 2's demonstrated capability.

### Proposed Attempt 3 Pipeline

1. **Tile the slope map** at full DEM resolution (1 m/pixel)
2. **Feed each tile to Molmo 2:** "Trace all dark lines"
3. **Stitch tile results** back into full-image coordinates
4. **Classify each traced line** using Gemma 4 + satellite imagery
5. **Engineer assigns vertex spacing** per classification (e.g., channel banks = dense, terrace edges = sparse)
6. **Feed vertices to mesh generator**

This separates **geometry extraction** (slope map + Molmo 2) from **feature classification** (satellite + Gemma 4), playing to each model's strength.

---

## 5. Conclusions

- **Molmo 2** is the only model that produces usable spatial coordinates. It is the geometry backbone of the pipeline.
- **Gemma 4** is the best scene interpreter. It provides the classification layer.
- **Qwen2.5-VL** has narrow utility (compact features with clear boundaries). It may serve as a sanity check but is not central to the pipeline.
- **Florence-2** is out-of-distribution on satellite imagery and is dropped entirely.
- The slope map is a better input than satellite imagery for geometry extraction because it already encodes hydraulic features as high-contrast lines.

---

*Audit based on notebooks `01_gemma4_satellite.ipynb`, `02_molmo2_satellite.ipynb`, `03_qwen25vl_satellite.ipynb`, and `04_florence2_satellite.ipynb` in `notebooks/attempt2/`. Conducted April 10, 2026.*
