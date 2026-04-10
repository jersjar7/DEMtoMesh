# DEMtoMesh

**AI-assisted mesh generation for 2D hydraulic models.**

DEMtoMesh uses computer vision and LLM reasoning to automatically identify hydraulic features from Digital Elevation Models (DEMs) and generate optimized computational meshes for software like SMS and HEC-RAS 2D — reducing what typically takes engineers hours of manual work to minutes of AI-assisted workflow.

---

## The Problem

Building a quality 2D hydraulic mesh is the most time-consuming part of flood modeling. Engineers manually inspect terrain, identify features (channels, floodplains, ridges, road embankments), and painstakingly assign mesh element types, sizes, and alignment. This process can consume 60–80% of a project's modeling time.

## The Idea

What if we skip training custom models entirely and instead:

1. **See** the terrain using existing vision models (Molmo, Florence-2)
2. **Segment** features precisely using SAM/SamGeo
3. **Reason** about mesh specifications using LLM engineering knowledge
4. **Generate** the mesh using existing meshing engines

The AI does 80% of the work in minutes. The engineer reviews, adjusts, and approves.

---

## Pipeline Overview

```
┌─────────────┐     ┌──────────────────┐     ┌─────────────────┐
│  Load DEM   │────▶│  Render as image  │────▶│  Vision models  │
│  (GeoTIFF)  │     │  (hillshade/slope)│     │  identify       │
└─────────────┘     └──────────────────┘     │  features       │
                                              └────────┬────────┘
                                                       │
                                                       ▼
┌─────────────┐     ┌──────────────────┐     ┌─────────────────┐
│  Output     │◀────│  LLM reasons     │◀────│  SAM segments   │
│  mesh file  │     │  about mesh      │     │  features at    │
│  (.2dm)     │     │  specifications  │     │  pixel level    │
└─────────────┘     └──────────────────┘     └─────────────────┘
```

## Key Models & Tools

| Role | Tool | Why |
|------|------|-----|
| Feature identification | **Molmo 2** (Ai2) | Returns precise (x,y) pixel coordinates from natural language prompts |
| Feature identification | **Florence-2** (Microsoft) | Returns bounding boxes, polygons, and segmentation from prompts |
| Precise segmentation | **SAM / SamGeo** (Meta) | Pixel-level segmentation masks on georeferenced rasters |
| Terrain-aware segmentation | **TASAM** | SAM variant with DEM/terrain priors built in |
| Mesh reasoning | **Claude / GPT-4o** | Domain reasoning about element types, sizes, transitions |
| Mesh generation | **Gmsh** (Python API) | Open-source meshing engine with fine-grained control |

## Hydraulic Features to Detect

- **Channels** → Quad elements aligned with flow direction, fine resolution
- **Floodplains** → Coarser triangles, size based on terrain variability
- **Ridges / Levees** → Breaklines to preserve sharp elevation changes
- **Road embankments** → Breaklines + local refinement
- **Bridge/culvert crossings** → Special mesh treatment (likely manual)
- **Flat storage areas** → Coarse elements
- **Steep hillslopes** → Medium elements, may exclude from active mesh

---

## Development Roadmap

### Phase 1: Proof of Concept (Current)
> **Goal:** Can vision models see hydraulic features in a DEM?

- [ ] Load a small, well-understood DEM
- [ ] Render as hillshade, slope, curvature
- [ ] Feed images to Molmo and/or Florence-2
- [ ] Ask models to identify: channel, floodplain, ridges, roads
- [ ] Evaluate: do the identified locations match reality?
- [ ] Feed identified points to SAM for precise segmentation
- [ ] Visualize everything

### Phase 2: Mesh Specification
> **Goal:** Can an LLM reason about mesh parameters from feature data?

- [ ] Extract DEM statistics per segmented region (width, slope, area)
- [ ] Prompt LLM with feature data + meshing rules
- [ ] Get back element type, size, alignment recommendations
- [ ] Compare to expert-built mesh for the same DEM

### Phase 3: Mesh Generation
> **Goal:** Can we produce an actual usable mesh file?

- [ ] Convert mesh specifications to Gmsh input
- [ ] Generate mesh
- [ ] Export to SMS (.2dm) or HEC-RAS format
- [ ] Run a simple hydraulic simulation to validate

### Phase 4: Engineer Review Interface
> **Goal:** Semi-automatic workflow with human approval

- [ ] Visualize AI feature map overlaid on DEM
- [ ] Allow engineer to adjust classifications and mesh parameters
- [ ] Re-generate mesh with adjustments
- [ ] Simple GUI (Streamlit or Panel)

---

## Known Challenges & Edge Cases

| Challenge | Status | Mitigation |
|-----------|--------|------------|
| Do vision models recognize terrain in hillshades? | **Unknown — test first** | Try multiple renderings (hillshade, slope, multi-band) |
| Large DEM tiling and edge consistency | Partially solved | SamGeo handles tiling; test boundary features |
| LLM stochasticity | Solved by design | Human review step; AI proposes, engineer approves |
| Subtle features (shallow swales, low levees) | Unknown | Vertical exaggeration, curvature maps may help |
| Hydraulic structures (bridges, culverts) | Likely manual | Exclude from auto-mesh; flag for engineer |
| SMS/HEC-RAS mesh format specifics | Need Dr. Zundel's input | Codify constraints in config file |

---

## Tech Stack

- **Language:** Python
- **IDE:** VS Code + Jupyter extension
- **DEM handling:** rasterio, numpy, scipy
- **Visualization:** matplotlib
- **Vision models:** Molmo2-8B (HuggingFace, local), Florence-2 (transformers), Gemma 4, Qwen2.5-VL
- **Segmentation:** segment-geospatial (SamGeo)
- **LLM reasoning:** anthropic / openai SDKs
- **Mesh generation:** gmsh (Python API)
- **Mesh I/O:** meshio

---

## Getting Started

```bash
# Clone
git clone https://github.com/jersjar7/DEMtoMesh.git
cd DEMtoMesh

# Create virtual environment
python -m venv venv
source venv/bin/activate  # or venv\Scripts\activate on Windows

# Install dependencies
pip install -r requirements.txt
```

Then open in VS Code and start with the notebooks in `notebooks/`.

---

## References

- [Molmo 2 — Ai2](https://allenai.org/blog/molmo2)
- [Florence-2 — Microsoft](https://huggingface.co/docs/transformers/en/model_doc/florence2)
- [SamGeo — Segment Geospatial](https://samgeo.gishub.org/)
- [TASAM — Terrain-Aware SAM](https://arxiv.org/html/2509.15795)
- [Gmsh](https://gmsh.info/)
- [SMS — Aquaveo](https://www.aquaveo.com/software/sms-surface-water-modeling-system-introduction)
- [HEC-RAS 2D — USACE](https://www.hec.usace.army.mil/software/hec-ras/)

---

## License

MIT

---

*This project explores a novel application of foundation models to hydraulic engineering — using AI vision and reasoning to automate what has traditionally been the most labor-intensive part of 2D flood modeling.*
