# Phase 1 Roadmap — Proof of Concept

**Created:** April 8, 2026
**Goal:** Determine whether vision models can identify hydraulic features in DEM-derived images.

---

## Task Legend

- `[ ]` — Not started
- `[x]` — Completed (date and time logged)

---

## Stage 1: Environment & Configuration

- [x] Create `.env.example` with placeholder keys (ANTHROPIC_API_KEY, OPENAI_API_KEY, HF_TOKEN)
- [x] Create and activate Python virtual environment
- [x] Install all dependencies from `requirements.txt` and resolve any version conflicts
- [x] Verify core imports work: `rasterio`, `numpy`, `scipy`, `matplotlib`, `torch`, `transformers`, `Pillow`
- [x] Verify GPU availability for local model inference (`torch.cuda.is_available()` or MPS for Apple Silicon)
- [x] Document hardware specs and available compute in this file

### Hardware & Compute Profile

| Spec | Value |
|------|-------|
| Machine | Apple M1 Pro |
| RAM | 32 GB unified memory |
| GPU Cores | 16 (Metal 4) |
| CUDA | Not available |
| MPS (Apple Silicon) | Available and verified |
| Python | 3.10.12 |
| PyTorch | 2.11.0 (MPS backend) |
| Transformers | 5.5.0 |

**Note:** M1 Pro with 32GB unified memory should handle Molmo 8B and Florence-2 locally via MPS. If memory pressure is too high, fall back to HF Inference API.

## Stage 2: Data Acquisition

- [x] Identify a suitable test DEM — small area, clear hydraulic features (channel, floodplain, ridges), well-understood terrain
- [x] Download DEM as GeoTIFF and place in `data/input/`
- [x] Document DEM metadata: source, CRS, resolution, extent, file size
### Test DEM Profile

| Property | Value |
|----------|-------|
| File | `data/input/1m elevation.tif` |
| Source | Cimarron River, Oklahoma |
| CRS | EPSG:26914 (UTM Zone 14N, NAD83) |
| Resolution | 1m x 1m |
| Dimensions | 5,306 x 3,088 pixels (~5.3km x 3.1km) |
| File size | 63 MB |
| Data type | float32 |
| Elevation range | 265.50m – 300.49m (35m relief) |
| Mean elevation | 271.32m |
| NoData | -999999.0 (57.4% of pixels — irregular river corridor shape) |

## Stage 3: DEM Loading & Rendering

- [x] Load DEM with `rasterio` — extract elevation array, CRS, transform, bounds, nodata handling
- [x] Compute and render hillshade (configurable azimuth/altitude) using `numpy`/`scipy`
- [x] Compute and render slope map (degrees) from elevation grid
- [x] Compute and render curvature map (profile curvature) from elevation grid
- [x] Create visualization utility: plot DEM-derived maps with proper extent, colorbar, and title
- [x] Export hillshade, slope, and curvature as PNG images for vision model input
- [x] Save all renders to `data/output/` with descriptive filenames

## Stage 4: Vision Model Feature Detection — Gemma 4

- [x] Connect to Gemma 4 (google/gemma-4-26b-a4b-it) via OpenRouter API
- [x] Feed hillshade image with general survey prompt — identify all hydraulic/geomorphic features
- [x] Test targeted prompts: channel description, floodplain identification, ridges/embankments
- [x] Test with slope map, curvature map, and elevation map as alternative inputs
- [x] Document which rendering + prompt combinations produce the best results
- [x] Log all model outputs (full response text) in the notebook

### Stage 4 Results

| Rendering | Features Identified | Quality |
|-----------|-------------------|---------|
| Hillshade | Channel, point bars, cut banks, floodplain, terrace edges, side channels, road embankments, ridges | Excellent — richest descriptions |
| Slope | Channel, point bars, cut banks, floodplain, terrace edges, ridges, roads, drainage features | Excellent |
| Curvature | Channel, point bars, cut banks, floodplain, terrace edges, ridges, roads, field boundaries | Excellent — most subtle features |
| Elevation | Channel, secondary channels, floodplain, levees, terraces, point bars, road embankments | Very good |

**Conclusion:** Vision models CAN identify hydraulic features in DEM-derived images. All four renderings produced accurate, detailed feature identification. The approach is viable.

## Stage 5: Vision Model Coordinate Grounding — Molmo 2, Qwen2.5-VL, Florence-2

### 5a: Molmo 2 Pointing (Blocked)
- [x] Create notebook `02_molmo2_pointing_test.ipynb` with parsing utilities
- [ ] ~~Test Molmo 2 pointing on hillshade~~ — **BLOCKED:** Model unavailable on OpenRouter (no providers serving it)

### 5b: Qwen2.5-VL Bounding Box Grounding (Failed)
- [x] Create notebook `03_qwen25vl_grounding_test.ipynb`
- [x] Connect to `qwen/qwen2.5-vl-72b-instruct` via OpenRouter
- [x] Test bbox grounding on hillshade (4 prompts: channel, ridges, floodplain, manmade)
- [x] Test bbox grounding on slope, curvature, elevation
- [x] Run qualitative comparison (same survey prompt as Gemma 4)
- [x] Document results in `docs/model-outputs/qwen25vl/`

**Result: FAILED.** 5/7 tests returned whole-image bounding boxes (`[0, 0, 1036, 588]`). Only slope and curvature channel tests produced plausibly localized but still imprecise bboxes. Model cannot spatially localize terrain features in DEM imagery.

### 5c: Florence-2 Local Detection (Failed)
- [x] Create notebook `04_florence2_local_test.ipynb`
- [x] Fix `forced_bos_token_id` error — switched to `florence-community/Florence-2-large` with native `Florence2ForConditionalGeneration` (transformers 5.5.0 compatibility)
- [x] Test object detection (`<OD>`) on hillshade
- [x] Test phrase grounding (`<CAPTION_TO_PHRASE_GROUNDING>`) on hillshade
- [x] Test dense region captioning (`<DENSE_REGION_CAPTION>`) on hillshade
- [x] Test phrase grounding on slope, curvature, elevation
- [x] Document results in `docs/model-outputs/florence2/`

**Result: FAILED.** All tasks returned entire-domain bounding boxes. Florence-2 cannot localize terrain features in DEM-derived imagery — too far from its natural image training distribution.

### Stage 5 Conclusion

| Model | Qualitative (describe) | Spatial (localize) | Verdict |
|-------|----------------------|-------------------|---------|
| Gemma 4 | Excellent | N/A (text only) | Best for descriptions |
| Qwen2.5-VL | Good | Failed (whole-image bboxes) | Cannot ground on DEM imagery |
| Florence-2 | N/A | Failed (whole-domain bboxes) | Cannot detect in DEM imagery |
| Molmo 2 | Unknown | Unknown (blocked) | Most promising — needs local setup |

**Root cause:** VLMs are trained on natural photographs. DEM renderings (hillshade, slope, curvature) are abstract terrain derivatives that look nothing like training data. Models can *reason* about features from descriptions but cannot *localize* them visually.

**Pivot decision:** Add satellite imagery (NAIP, 0.6m) as primary input for feature localization. Use DEM derivatives for precision refinement only. See `docs/audits/04-08-2026-phase1-roadmap-update.md` for the revised plan.

## Stages 6–8: Deferred

Stages 6 (SAM), 7 (Evaluation), and 8 (Cleanup) are deferred pending the pipeline redesign. The revised roadmap is in `docs/audits/04-08-2026-phase1-roadmap-update.md`.

---

## Completion Log

*Record each task completion below as it happens.*

| Date | Time | Task | Notes |
|------|------|------|-------|
| 2026-04-08 | 11:14 | Stage 1: Create `.env.example` | Template with ANTHROPIC_API_KEY, OPENAI_API_KEY, HF_TOKEN |
| 2026-04-08 | 11:14 | Stage 1: Create and activate venv | Python 3.10.12, venv at `./venv/` |
| 2026-04-08 | 11:14 | Stage 1: Install all dependencies | All packages installed — zero conflicts |
| 2026-04-08 | 11:14 | Stage 1: Verify core imports | All 13 core packages import successfully |
| 2026-04-08 | 11:14 | Stage 1: Verify GPU availability | MPS available and verified with tensor test |
| 2026-04-08 | 11:14 | Stage 1: Document hardware specs | M1 Pro, 32GB, 16 GPU cores, MPS backend |
| 2026-04-08 | 11:20 | Stage 2: Identify test DEM | Cimarron River, 1m resolution, clear river corridor |
| 2026-04-08 | 11:20 | Stage 2: Place DEM in data/input/ | `1m elevation.tif`, 63MB |
| 2026-04-08 | 11:20 | Stage 2: Document DEM metadata | EPSG:26914, 5306x3088px, 265–300m elev, 57.4% nodata |
| 2026-04-08 | 11:24 | Stage 3: Load DEM with rasterio | Elevation array, CRS, transform, bounds, nodata mask |
| 2026-04-08 | 11:24 | Stage 3: Compute hillshade | az=315, alt=45, range 0.000–1.000 |
| 2026-04-08 | 11:24 | Stage 3: Compute slope map | Range 0–89.73 degrees |
| 2026-04-08 | 11:24 | Stage 3: Compute curvature | Laplacian, range -1.74 to 211.01 |
| 2026-04-08 | 11:24 | Stage 3: Create plot_terrain utility | Reusable viz with extent, colorbar, title |
| 2026-04-08 | 11:24 | Stage 3: Export PNGs | hillshade (5MB), slope (10MB), curvature (16MB), elevation (2MB) |
| 2026-04-08 | 11:24 | Stage 3: Save to data/output/ | 4 files: cimarron_{hillshade,slope,curvature,elevation}.png |
| 2026-04-08 | 12:20 | Stage 4: Connect to Gemma 4 via OpenRouter | google/gemma-4-26b-a4b-it, paid tier, images resized to 1024px |
| 2026-04-08 | 12:20 | Stage 4: Hillshade feature detection (4 prompts) | General survey, channel, floodplain, ridges/embankments — all excellent |
| 2026-04-08 | 12:20 | Stage 4: Alternative rendering tests | Slope, curvature, elevation — all correctly identified features |
| 2026-04-08 | 12:20 | Stage 4: Results summary | 7/7 queries successful, all renderings viable, approach confirmed |
| 2026-04-08 | 15:00 | Stage 5a: Molmo 2 notebook created | Notebook ready but model unavailable on OpenRouter |
| 2026-04-08 | 15:30 | Stage 5b: Qwen2.5-VL grounding tests | 7 tests run — all bboxes useless (whole-image or imprecise) |
| 2026-04-08 | 16:00 | Stage 5c: Florence-2 load failure | `forced_bos_token_id` error with transformers 5.5.0 |
| 2026-04-08 | 16:15 | Stage 5c: Florence-2 fix applied | Switched to `florence-community/Florence-2-large` (native class) |
| 2026-04-08 | 16:30 | Stage 5c: Florence-2 detection tests | All tasks returned whole-domain bboxes — DEM imagery out of distribution |
| 2026-04-08 | 17:00 | Pipeline pivot decided | Add satellite imagery (NAIP 0.6m) for feature localization, DEM for precision only |

---

## Decision Log

*Record key decisions made during Phase 1.*

| Date | Decision | Rationale |
|------|----------|-----------|
| 2026-04-08 | Project scaffolding created | Initial repo setup with README, requirements, notebook skeleton |
| 2026-04-08 | Phase 1 audit and roadmap created | Baseline assessment to guide development |
| 2026-04-08 | Switched from Molmo (local) to Gemma 4 (OpenRouter API) | Molmo unavailable on OpenRouter; Gemma 4 produces excellent qualitative results for free/$0.10/M tokens |
| 2026-04-08 | VLM grounding on DEM imagery does not work | Qwen2.5-VL and Florence-2 both return whole-image bboxes — DEM renderings are out of distribution for these models |
| 2026-04-08 | Pivot to dual-input pipeline (satellite + DEM) | VLMs understand satellite imagery (in-distribution). Use satellite for feature identification, DEM for precision breaklines |
| 2026-04-08 | NAIP (0.6m) selected as satellite source | Free, no API key, covers all US, higher resolution than our 1m DEM, programmatic download via Planetary Computer |
| 2026-04-08 | Florence-2 checkpoint changed | `microsoft/Florence-2-large` → `florence-community/Florence-2-large` for transformers >= 5.0 compatibility |

---

*This roadmap is a living document. Tasks will be checked off and timestamped as they are completed. New tasks may be added if discoveries during development reveal additional needs.*
