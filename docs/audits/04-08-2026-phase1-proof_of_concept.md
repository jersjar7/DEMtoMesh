# Phase 1 Audit — Proof of Concept

**Date:** April 8, 2026
**Scope:** Phase 1 — Can vision models see hydraulic features in a DEM?
**Status:** Just started — project scaffolding only

---

## 1. What Exists

| Item | Status | Notes |
|------|--------|-------|
| `README.md` | Complete | Full project vision, pipeline diagram, 4-phase roadmap, tech stack, references |
| `requirements.txt` | Complete | All dependencies listed with role comments |
| `.gitignore` | Complete | Python, venv, secrets, Jupyter, docs exclusions |
| `data/input/` | Empty | No DEM files acquired yet |
| `data/output/` | Empty | No outputs generated yet |
| `notebooks/01_feature_detection_test.ipynb` | Skeleton only | Contains markdown description + empty TODO cell |
| `docs/research/04-08-2026.md` | Complete | Full brainstorming log with model research, pipeline design, challenges |

## 2. What's Missing for Phase 1

### 2.1 Environment & Configuration

| Missing Item | Priority | Description |
|--------------|----------|-------------|
| `.env.example` | High | Template for API keys (ANTHROPIC_API_KEY, OPENAI_API_KEY, HF_TOKEN). Mentioned in research doc scaffold but never created |
| Virtual environment setup verification | Medium | No evidence that `pip install -r requirements.txt` has been tested — dependency conflicts are likely (torch + transformers + segment-geospatial) |
| GPU/compute assessment | Medium | No documentation of available hardware. Local Molmo/Florence-2 inference needs 8GB+ VRAM. Fallback: HF Inference API |

### 2.2 Data Acquisition

| Missing Item | Priority | Description |
|--------------|----------|-------------|
| Test DEM file | Critical | No DEM has been acquired. Need a small, well-understood DEM with clear hydraulic features (channel, floodplain, ridges). Sources: Dr. Zundel's class materials or USGS 3DEP |
| Ground truth reference | High | Need a known-good feature map or expert-built mesh for the test DEM to evaluate model outputs against |
| DEM metadata documentation | Medium | Once acquired: document CRS, resolution, extent, known features, and why this DEM was chosen |

### 2.3 DEM Processing Code

| Missing Item | Priority | Description |
|--------------|----------|-------------|
| DEM loading with rasterio | Critical | Read GeoTIFF, extract elevation array, CRS, transform, bounds |
| Hillshade rendering | Critical | Generate hillshade from elevation using scipy/numpy (azimuth, altitude parameters) |
| Slope map rendering | High | Compute slope in degrees or percent from elevation grid |
| Curvature map rendering | High | Profile and/or plan curvature from elevation grid |
| Image export (PNG) | High | Save rendered maps as images for vision model input |
| Visualization functions | Medium | matplotlib plotting with proper georeferencing, colorbars, titles |

### 2.4 Vision Model Integration

| Missing Item | Priority | Description |
|--------------|----------|-------------|
| Molmo inference pipeline | Critical | Load Molmo model via transformers, feed terrain image, parse (x,y) coordinate output (normalized 0–100 values) |
| Florence-2 inference pipeline | Critical | Load Florence-2 via transformers, use grounding/detection tasks, parse bounding box and polygon outputs |
| Prompt engineering for terrain | High | Develop and test prompts: "Point to the main channel", "Detect all ridges", "Identify the floodplain boundary", etc. |
| Coordinate conversion | High | Convert model output coordinates (normalized/pixel) back to georeferenced CRS coordinates |
| HF Inference API fallback | Medium | Alternative path if local GPU is insufficient — use Hugging Face serverless inference |
| Multi-rendering strategy | Medium | Test which DEM rendering (hillshade, slope, curvature, composite) produces best model recognition |

### 2.5 Segmentation Pipeline

| Missing Item | Priority | Description |
|--------------|----------|-------------|
| SAM/SamGeo integration | High | Take point prompts or bounding boxes from vision models and produce pixel-level masks |
| Mask-to-vector conversion | Medium | Convert segmentation masks to vector polygons (shapely geometries) |
| Georeferenced mask output | Medium | Ensure segmentation results carry CRS and transform from original DEM |

### 2.6 Evaluation & Visualization

| Missing Item | Priority | Description |
|--------------|----------|-------------|
| Overlay visualization | High | Plot model-identified features overlaid on DEM/hillshade |
| Accuracy assessment | Medium | Compare detected features against ground truth — qualitative at minimum, quantitative if reference exists |
| Results documentation | Medium | Notebook cells that clearly show what worked, what failed, and conclusions |

### 2.7 Project Infrastructure

| Missing Item | Priority | Description |
|--------------|----------|-------------|
| Utility module (`src/` or similar) | Low | Not needed yet — keep everything in notebooks for Phase 1. Extract to modules in Phase 2 |
| Testing framework | Low | Not needed for Phase 1 proof of concept |
| CI/CD | Low | Not needed yet |

## 3. Risk Assessment

### High Risk
- **Vision models may not recognize terrain features in hillshade images.** These models were trained on natural photographs, not DEM renderings. This is the fundamental question Phase 1 must answer.
- **Dependency conflicts.** PyTorch + transformers + segment-geospatial + rasterio can have complex version interactions. Must verify early.

### Medium Risk
- **GPU availability.** Molmo 8B and Florence-2 need meaningful VRAM for local inference. If hardware is insufficient, HF Inference API adds latency and rate limits.
- **Coordinate precision.** Molmo outputs normalized 0–100 coordinates. Rounding/quantization error may reduce spatial precision. Need to measure this.

### Low Risk
- **DEM acquisition.** Multiple free sources available (USGS 3DEP, OpenTopography). Not blocked, just needs to be done.
- **SAM/SamGeo stability.** Well-established library with active maintenance.

## 4. Summary

Phase 1 is **almost entirely unbuilt**. The project has solid documentation and planning but no functional code, no data, and no environment verification. The critical path is:

1. Acquire a test DEM
2. Set up the environment and verify dependencies work together
3. Build DEM processing (load, render hillshade/slope/curvature)
4. Integrate Molmo and/or Florence-2 for feature detection
5. Connect to SAM for segmentation
6. Evaluate and document results

The entire phase hinges on one question: **can vision models identify hydraulic features in DEM-derived images?** Everything should be oriented toward answering this as quickly as possible.

---

*Audit conducted on April 8, 2026 against README.md, docs/research/04-08-2026.md, and current repository state.*
