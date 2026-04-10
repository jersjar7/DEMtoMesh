# Phase 1 Roadmap Update — Dual-Input Pipeline

**Created:** April 8, 2026
**Supersedes:** Stages 5–8 of `04-08-2026-phase1-roadmap.md`
**Context:** VLM grounding on DEM-only imagery failed. This roadmap adds satellite imagery and redesigns the pipeline.

---

## Task Legend

- `[ ]` — Not started
- `[x]` — Completed
- `[!]` — Blocked or failed

---

## Stage 5 (Revised): Satellite Data Acquisition

### 5.1 Download NAIP Imagery
- [x] Install satellite dependencies: `pip install pystac-client planetary-computer rioxarray`
- [x] Create notebook `05_satellite_acquisition.ipynb`
- [x] Query Microsoft Planetary Computer STAC API for NAIP tiles covering the DEM bounding box
- [x] Download most recent NAIP imagery (target: 2021 or 2023, 0.6m resolution, RGBN)
- [x] Clip to DEM extent + 500m buffer on all sides
- [x] Reproject to EPSG:26914 (match DEM CRS) if needed
- [x] Save as `data/input/cimarron_naip_rgb.tif` (3-band) and `cimarron_naip_rgbn.tif` (4-band)
- [x] Export satellite PNG for vision model input: `data/output/cimarron_satellite.png`

### 5.2 Verify Alignment
- [x] Overlay satellite PNG on hillshade PNG — confirm spatial alignment
- [x] Plot side-by-side: satellite vs. hillshade vs. slope vs. curvature
- [x] Document satellite metadata: source, date, resolution, CRS, bounds

### Bounding Box Reference

| CRS | West | South | East | North |
|-----|------|-------|------|-------|
| EPSG:26914 | 645,115 | 3,982,702 | 650,421 | 3,985,790 |
| WGS84 | -97.3904 | 35.9780 | -97.3310 | 36.0050 |

### NAIP Details

| Property | Value |
|----------|-------|
| Source | USDA NAIP via Microsoft Planetary Computer |
| Resolution | 0.6m (60cm) per pixel |
| Bands | Red, Green, Blue, Near-Infrared |
| Cost | Free (anonymous access, no API key) |
| Python packages | `pystac-client`, `planetary-computer`, `rioxarray` |

---

## Stage 6 (Revised): VLM Feature Detection on Satellite Imagery

### 6.1 Gemma 4 on Satellite
- [x] Create notebook `06_satellite_feature_detection.ipynb`
- [x] Run general survey prompt on satellite RGB image
- [x] Run targeted prompts: channel, floodplain, ridges, roads, vegetation boundaries
- [x] Compare quality to Gemma 4 on DEM hillshade (notebook 01)

### 6.2 Qwen2.5-VL Grounding on Satellite
- [x] Run bbox grounding prompts on satellite RGB
- [x] Test: channel detection, road detection, vegetation boundaries
- [x] Compare bbox quality to DEM results (expect significant improvement)
- [x] If bboxes are localized, proceed to SAM handoff

### 6.3 Florence-2 Detection on Satellite
- [x] Run `<OD>` (object detection) on satellite RGB
- [x] Run `<CAPTION_TO_PHRASE_GROUNDING>` with hydraulic feature phrases
- [x] Run `<DENSE_REGION_CAPTION>` for open-ended detection
- [x] Compare detection quality to DEM results

### 6.4 Molmo 2 Pointing on Satellite (Local)
- [x] Get Molmo 2 running locally (HuggingFace Transformers, Molmo2-8B)
- [x] Test pointing prompts on satellite RGB: "Point to the river channel", "Point to road embankments"
- [x] Parse point coordinates and overlay on satellite image
- [x] Convert pixel coordinates to geo-coordinates
- [x] Evaluate: are points accurate enough to serve as SAM prompts? — yes, channel bends, roads, bridges all usable

### 6.5 Dual-Input Prompts
- [ ] Test multi-image prompts (models that support it: Qwen2.5-VL, Gemma 4)
- [ ] Send satellite + hillshade together: "Image 1 is a hillshade, Image 2 is a satellite photo of the same area. Identify features visible in both."
- [ ] Test: does dual input improve localization accuracy?

---

## Stage 6 Results & Pipeline Pivot

- **Gemma 4:** Excellent qualitative descriptions of satellite imagery, but produces no spatial output (no coordinates, no bounding boxes).
- **Molmo 2:** Usable point coordinates — 47 road points, 13 channel bends, 2 bridges identified and georeferenced.
- **Qwen2.5-VL:** Loose bounding boxes; only sandbars were well-localized. Other features had bboxes too coarse for downstream use.
- **Florence-2:** Failed completely on satellite imagery — no meaningful detections.

**Key insight:** The slope map already shows hydraulic and infrastructure features as dark lines (high slope = sharp terrain breaks). Instead of forcing VLMs to segment features from scratch, **Attempt 3** will exploit this:

> Tile slope map --> Molmo 2 traces dark lines --> Gemma 4 classifies each line (channel bank, road, terrace, etc.) --> Engineer sets vertex spacing for mesh generation.

---

## Stage 7 (Revised): SAM Segmentation

### 7.1 Tiled Slope Tracing Pipeline (Attempt 3)
- [ ] Create `attempt3` folder and tiled slope tracing notebook
- [ ] Tile slope map into ~1024x1024 tiles with overlap
- [ ] Feed each tile to Molmo 2: "Trace all dark lines"
- [ ] Stitch tile results into full-image georeferenced polylines
- [ ] Use Gemma 4 + satellite to classify each line (channel bank, road, terrace, etc.)
- [ ] Feed classified lines + vertex spacing to SAM / mesh generator

### 7.2 DEM-Guided Mask Refinement
- [ ] Map SAM satellite masks to DEM pixel grid (same CRS, resample if needed)
- [ ] Use DEM curvature to refine mask edges — snap to curvature peaks (breaklines)
- [ ] Use DEM slope to refine channel bank edges — snap to high-slope boundaries
- [ ] Compare: raw SAM mask vs. DEM-refined mask — visualize improvement

### 7.3 SAM Directly on DEM Derivatives (Experimental)
- [ ] Run SAM on hillshade image (SAM doesn't care if it's a photo or terrain rendering)
- [ ] Run SAM on slope map — channel banks have high visual contrast
- [ ] Compare: SAM on satellite vs. SAM on hillshade — which produces tighter masks?

---

## Stage 8 (Revised): Feature Map & Evaluation

### 8.1 Build Georeferenced Feature Map
- [ ] Combine all segmentation masks into a single feature classification raster
- [ ] Assign feature types: channel, floodplain, ridge/levee, road embankment, upland
- [ ] Export as GeoTIFF with feature type per pixel
- [ ] Convert to vector polygons (shapely/geopandas)

### 8.2 Evaluation
- [ ] Manual ground truth: identify known features via visual inspection of satellite + DEM
- [ ] Overlay detected features on DEM hillshade — composite visualization
- [ ] Create summary table: feature type, detected (yes/no), quality, which model + input found it
- [ ] Document: satellite vs. DEM-only — quantify the improvement
- [ ] Document failure cases and edge cases
- [ ] Write Phase 1 conclusion

### 8.3 Cleanup & Deliverables
- [ ] Clean up all notebooks
- [ ] Ensure reproducibility (model versions, prompts, seeds documented)
- [ ] Save final visualizations to `data/output/`
- [ ] Update README.md with Phase 1 results
- [ ] Update roadmap with completion notes

---

## Model Availability & Priority

| Model | Input | Method | Priority | Status |
|-------|-------|--------|----------|--------|
| Gemma 4 | Satellite | Qualitative descriptions (API) | 1 — baseline | Ready |
| Qwen2.5-VL | Satellite | Bbox grounding (API) | 2 — test localization | Ready |
| Florence-2 | Satellite | Detection + grounding (local) | 3 — test local detection | Ready (fixed) |
| Molmo 2 | Satellite | Point coordinates (local) | 4 — ideal for SAM prompts | Ready (HF Transformers) |
| SAM/SamGeo | Satellite + DEM | Segmentation from prompts | 5 — after VLM results | Not started |

---

## New Dependencies

```
# Add to requirements.txt
pystac-client        # STAC API client for Planetary Computer catalog search
planetary-computer   # Anonymous SAS token signing for Azure Blob Storage
rioxarray            # xarray extension for raster data with CRS support
```

---

## Risk Assessment

| Risk | Likelihood | Impact | Mitigation |
|------|-----------|--------|------------|
| VLMs still can't localize on satellite imagery | Low | High | Satellite is in-distribution — these models have seen millions of aerial photos |
| NAIP coverage gap for this site | Very Low | Medium | NAIP covers entire continental US; Oklahoma has confirmed coverage |
| Molmo 2 can't run on M1 Pro 32GB | Low | Medium | 7B model should fit; fallback to Florence-2 or wait for OpenRouter |
| SAM masks too coarse for breaklines | Medium | Medium | DEM curvature refinement should snap edges to sub-meter precision |
| Satellite/DEM misalignment | Low | High | Both in EPSG:26914; verify with overlay plot before proceeding |

---

## Success Criteria for This Phase

1. **At least one VLM produces usable point or bbox coordinates** on satellite imagery (not whole-image)
2. **SAM generates a meaningful segmentation mask** for the river channel from VLM prompts
3. **DEM refinement improves mask edges** — visible snapping to curvature breaklines
4. **Georeferenced feature map** covers the major hydraulic features: channel, floodplain, terrace edges, road embankments

If all four criteria are met, we proceed to Phase 2 (mesh specification via LLM reasoning).

---

*This roadmap update builds on the original Phase 1 roadmap. Stages 1–4 remain as-is. Stages 5–8 are replaced by this document.*
