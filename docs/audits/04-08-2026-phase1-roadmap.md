# Phase 1 Roadmap — Proof of Concept

**Created:** April 8, 2026
**Goal:** Determine whether vision models can identify hydraulic features in DEM-derived images.

---

## Task Legend

- `[ ]` — Not started
- `[x]` — Completed (date and time logged)

---

## Stage 1: Environment & Configuration

- [ ] Create `.env.example` with placeholder keys (ANTHROPIC_API_KEY, OPENAI_API_KEY, HF_TOKEN)
- [ ] Create and activate Python virtual environment
- [ ] Install all dependencies from `requirements.txt` and resolve any version conflicts
- [ ] Verify core imports work: `rasterio`, `numpy`, `scipy`, `matplotlib`, `torch`, `transformers`, `Pillow`
- [ ] Verify GPU availability for local model inference (`torch.cuda.is_available()` or MPS for Apple Silicon)
- [ ] Document hardware specs and available compute in this file

## Stage 2: Data Acquisition

- [ ] Identify a suitable test DEM — small area, clear hydraulic features (channel, floodplain, ridges), well-understood terrain
- [ ] Download DEM as GeoTIFF and place in `data/input/`
- [ ] Document DEM metadata: source, CRS, resolution, extent, file size
- [ ] Identify known hydraulic features in the DEM (manual visual inspection) to serve as ground truth reference
- [ ] Document ground truth features with approximate locations and types

## Stage 3: DEM Loading & Rendering

- [ ] Load DEM with `rasterio` — extract elevation array, CRS, transform, bounds, nodata handling
- [ ] Compute and render hillshade (configurable azimuth/altitude) using `numpy`/`scipy`
- [ ] Compute and render slope map (degrees) from elevation grid
- [ ] Compute and render curvature map (profile curvature) from elevation grid
- [ ] Create visualization utility: plot DEM-derived maps with proper extent, colorbar, and title
- [ ] Export hillshade, slope, and curvature as PNG images for vision model input
- [ ] Save all renders to `data/output/` with descriptive filenames

## Stage 4: Vision Model Integration — Molmo

- [ ] Load Molmo model via `transformers` (determine model size based on available hardware)
- [ ] Feed hillshade image to Molmo with prompt: "Point to the main channel in this terrain image"
- [ ] Parse Molmo output — extract normalized (x,y) coordinates and convert to pixel positions
- [ ] Convert pixel coordinates to georeferenced CRS coordinates using rasterio transform
- [ ] Test additional prompts: "Point to the ridges", "Point to the floodplain", "Point to any road embankments"
- [ ] Test with slope map and curvature map as alternative inputs
- [ ] Document which rendering + prompt combinations produce the best results
- [ ] Log all model outputs (coordinates, confidence, response text) in the notebook

## Stage 5: Vision Model Integration — Florence-2

- [ ] Load Florence-2 model via `transformers`
- [ ] Feed hillshade image using object detection task (`<OD>`) with terrain feature labels
- [ ] Feed hillshade image using grounded segmentation task with natural language prompts
- [ ] Parse outputs — extract bounding boxes and/or polygon coordinates
- [ ] Convert output coordinates to georeferenced CRS coordinates
- [ ] Test with slope map and curvature map as alternative inputs
- [ ] Compare Florence-2 results against Molmo results for the same DEM
- [ ] Document which model performs better for terrain feature identification

## Stage 6: SAM/SamGeo Segmentation

- [ ] Install and verify `segment-geospatial` (SamGeo) works with the test DEM
- [ ] Feed Molmo-identified point coordinates as point prompts to SAM
- [ ] Feed Florence-2 bounding boxes as box prompts to SAM
- [ ] Generate pixel-level segmentation masks for each identified feature
- [ ] Convert segmentation masks to vector polygons (shapely geometries)
- [ ] Verify masks carry georeferenced CRS from original DEM
- [ ] Export segmentation results (masks and/or vectors) to `data/output/`

## Stage 7: Evaluation & Results

- [ ] Overlay all detected features on the original DEM/hillshade — create composite visualization
- [ ] Compare detected features against ground truth — visual assessment at minimum
- [ ] Create a summary table: feature type, detected (yes/no), quality (good/rough/missed), which model found it
- [ ] Document which DEM rendering produced the best recognition results
- [ ] Document failure cases — what features were missed or misidentified and why
- [ ] Write Phase 1 conclusion: does this approach have a viable path forward?
- [ ] Identify specific improvements needed for Phase 2

## Stage 8: Cleanup & Deliverables

- [ ] Clean up notebook — clear unnecessary outputs, add markdown explanations between code cells
- [ ] Ensure all results are reproducible (seeds, model versions, exact prompts documented)
- [ ] Save final visualizations to `data/output/`
- [ ] Update README.md Phase 1 checklist with results
- [ ] Update this roadmap with completion notes

---

## Completion Log

*Record each task completion below as it happens.*

| Date | Time | Task | Notes |
|------|------|------|-------|
| — | — | — | — |

---

## Decision Log

*Record key decisions made during Phase 1.*

| Date | Decision | Rationale |
|------|----------|-----------|
| 2026-04-08 | Project scaffolding created | Initial repo setup with README, requirements, notebook skeleton |
| 2026-04-08 | Phase 1 audit and roadmap created | Baseline assessment to guide development |

---

*This roadmap is a living document. Tasks will be checked off and timestamped as they are completed. New tasks may be added if discoveries during development reveal additional needs.*
