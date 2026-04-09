# Florence-2 — DEM Detection Test Results

**Date:** April 8, 2026

**Model:** `florence-community/Florence-2-large` (local, MPS)

**DEM:** Cimarron River, 1m resolution, EPSG:26914

**Notebook:** `notebooks/04_florence2_local_test.ipynb`

---

## Od Hillshade

**Task:** `<OD>`

**Inference time:** 14.4s

**Detections:** 1

**Bounding boxes (pixel coords):**

- (2, 1, 5303, 3086) — footwear

**Geo-coordinates (EPSG:26914, bbox centers):**

- (647767.5, 3984246.5) — footwear

**Raw result:**

```json
{
  "<OD>": {
    "bboxes": [
      [
        2,
        1,
        5303,
        3086
      ]
    ],
    "labels": [
      "footwear"
    ]
  }
}
```

---

## Phrase Hillshade

**Task:** `<CAPTION_TO_PHRASE_GROUNDING>`

**Phrase:** river channel. floodplain. terrace edge. road embankment. ridge.

**Inference time:** 1.9s

**Detections:** 2

**Bounding boxes (pixel coords):**

- (34, 29, 5260, 3043) — river channel
- (34, 29, 5266, 3043) — floodplain

**Geo-coordinates (EPSG:26914, bbox centers):**

- (647762.5, 3984253.5) — river channel
- (647765.5, 3984253.5) — floodplain

**Raw result:**

```json
{
  "<CAPTION_TO_PHRASE_GROUNDING>": {
    "bboxes": [
      [
        34,
        29,
        5260,
        3043
      ],
      [
        34,
        29,
        5266,
        3043
      ]
    ],
    "labels": [
      "river channel",
      "floodplain"
    ]
  }
}
```

---

## Dense Hillshade

**Task:** `<DENSE_REGION_CAPTION>`

**Inference time:** 1.4s

**Detections:** 0

**Raw result:**

```json
{
  "<DENSE_REGION_CAPTION>": {
    "bboxes": [],
    "labels": []
  }
}
```

---

## Phrase Slope Channel

**Task:** `<CAPTION_TO_PHRASE_GROUNDING>`

**Phrase:** river channel

**Rendering:** slope

**Inference time:** 1.5s

**Detections:** 1

**Bounding boxes (pixel coords):**

- (13, 7, 5287, 3074) — river channel

**Geo-coordinates (EPSG:26914, bbox centers):**

- (647765.5, 3984249.5) — river channel

**Raw result:**

```json
{
  "<CAPTION_TO_PHRASE_GROUNDING>": {
    "bboxes": [
      [
        13,
        7,
        5287,
        3074
      ]
    ],
    "labels": [
      "river channel"
    ]
  }
}
```

---

## Phrase Curvature Channel

**Task:** `<CAPTION_TO_PHRASE_GROUNDING>`

**Phrase:** river channel

**Rendering:** curvature

**Inference time:** 1.2s

**Detections:** 1

**Bounding boxes (pixel coords):**

- (13, 7, 5287, 3074) — river channel

**Geo-coordinates (EPSG:26914, bbox centers):**

- (647765.5, 3984249.5) — river channel

**Raw result:**

```json
{
  "<CAPTION_TO_PHRASE_GROUNDING>": {
    "bboxes": [
      [
        13,
        7,
        5287,
        3074
      ]
    ],
    "labels": [
      "river channel"
    ]
  }
}
```

---

## Phrase Elevation Channel

**Task:** `<CAPTION_TO_PHRASE_GROUNDING>`

**Phrase:** river channel

**Rendering:** elevation

**Inference time:** 1.5s

**Detections:** 1

**Bounding boxes (pixel coords):**

- (13, 7, 5287, 3071) — river channel

**Geo-coordinates (EPSG:26914, bbox centers):**

- (647765.5, 3984250.5) — river channel

**Raw result:**

```json
{
  "<CAPTION_TO_PHRASE_GROUNDING>": {
    "bboxes": [
      [
        13,
        7,
        5287,
        3071
      ]
    ],
    "labels": [
      "river channel"
    ]
  }
}
```

---

## Summary

| Test | Task | Detections | Time |
|------|------|------------|------|
| od_hillshade | `<OD>` | 1 | 14.4s |
| phrase_hillshade | `<CAPTION_TO_PHRASE_GROUNDING>` | 2 | 1.9s |
| dense_hillshade | `<DENSE_REGION_CAPTION>` | 0 | 1.4s |
| phrase_slope_channel | `<CAPTION_TO_PHRASE_GROUNDING>` | 1 | 1.5s |
| phrase_curvature_channel | `<CAPTION_TO_PHRASE_GROUNDING>` | 1 | 1.2s |
| phrase_elevation_channel | `<CAPTION_TO_PHRASE_GROUNDING>` | 1 | 1.5s |
