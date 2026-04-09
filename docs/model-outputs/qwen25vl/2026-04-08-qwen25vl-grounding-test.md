# Qwen2.5-VL — DEM Grounding Test Results

**Date:** April 8, 2026

**Model:** `qwen/qwen2.5-vl-72b-instruct` (via OpenRouter)

**DEM:** Cimarron River, 1m resolution, EPSG:26914

**Images:** Resized to 1024px wide before sending to API

**Notebook:** `notebooks/03_qwen25vl_grounding_test.ipynb`

---

## Grounding: Hillshade Channel

**Prompt:** Detect all dark sinuous lines and curves in this grayscale terrain image. These are river channels. Output a JSON array where each object has {"bbox_2d": [x1, y1, x2, y2], "label": "name"}. Coordinates should be pixel positions. Return multiple tight bounding boxes for different segments.

**Detections:** 5 (5 bboxes, 0 points)

**Bounding boxes (original pixel coords):**

- (54, 1544, 1884, 2537) — name
- (1787, 1423, 3303, 2426) — name
- (3508, 1323, 4007, 1544) — name
- (3952, 551, 5089, 772) — name
- (4115, 193, 4981, 441) — name

**Geo-coordinates (EPSG:26914):**

- (646084.5, 3983749.5) — name
- (647659.5, 3983865.5) — name
- (648872.5, 3984356.5) — name
- (649635.5, 3985128.5) — name
- (649663.5, 3985472.5) — name

**Raw output:**

```
```json
[
	{"bbox_2d": [10, 280, 348, 460], "label": "name"},
	{"bbox_2d": [330, 258, 610, 440], "label": "name"},
	{"bbox_2d": [648, 240, 740, 280], "label": "name"},
	{"bbox_2d": [730, 100, 940, 140], "label": "name"},
	{"bbox_2d": [760, 35, 920, 80], "label": "name"}
]
```
```

---

## Grounding: Hillshade Ridges

**Prompt:** Detect all bright elevated linear features and sharp edges in this grayscale terrain image. These include ridges and terrace edges. Output a JSON array where each object has {"bbox_2d": [x1, y1, x2, y2], "label": "name"}. Coordinates should be pixel positions.

**Detections:** 1 (1 bboxes, 0 points)

**Bounding boxes (original pixel coords):**

- (1776, 1379, 2956, 1561) — ridge

**Geo-coordinates (EPSG:26914):**

- (647481.5, 3984320.5) — ridge

**Raw output:**

```
```json
[
    {"bbox_2d": [328, 250, 546, 283], "label": "ridge"},
    {"bbox_2d": [140, 314, 171, 332], "label": "ridge"},
    {"bbox_2d": [28, 398, 122, 421], "label": "ridge"},
    {"bbox_2d": [592, 273, 654, 383], "label": "ridge"},
    {"bbox_2d": [642, 151, 758, 263], "label": "ridge"},
    {"bbox_2d": [300, 312, 340, 403], "label": "ridge"},
    {"bbox_2d": [657, 155, 756, 223], "label": "ridge"},
    {"bbox_2d": [733, 6, 941, 67], "label": "ridge"},
    {"bbox_2d": [636, 173, 974, 265], "label": "ridge"},
    {"bbox_2d": [760, 86, 858, 155], "label": "ridge"},
    {"bbox_2d": [716, 280, 762, 326], "label": "ridge"},
    {"bbox_2d": [134, 299, 199, 392], "label": "ridge"},
    {"bbox_2d": [66, 276, 174, 293], "label": "ridge"},
    {"bbox_2d": [423, 257, 534, 290], "label": "ridge"},
    {"bbox_2d": [607, 208, 622, 244], "label": "ridge"},
    {"bbox_2d": [796, 205, 
```

---

## Grounding: Hillshade Floodplain

**Prompt:** Detect the distinct flat smooth-textured regions in this grayscale terrain image. These are floodplain areas — they appear as uniform gray zones between the textured higher ground and the dark channel. Output a JSON array where each object has {"bbox_2d": [x1, y1, x2, y2], "label": "name"}. Coordinates should be pixel positions. Return separate boxes for each distinct area.

**Detections:** 3 (3 bboxes, 0 points)

**Bounding boxes (original pixel coords):**

- (1798, 1566, 3303, 2509) — name
- (325, 1654, 1841, 2509) — name
- (3292, 1594, 3481, 2095) — name

**Geo-coordinates (EPSG:26914):**

- (647665.5, 3983752.5) — name
- (646197.5, 3983708.5) — name
- (648501.5, 3983945.5) — name

**Raw output:**

```
```json
[
	{"bbox_2d": [332, 284, 610, 455], "label": "name"},
	{"bbox_2d": [60, 300, 340, 455], "label": "name"},
	{"bbox_2d": [608, 289, 643, 380], "label": "name"}
]
```
```

---

## Grounding: Hillshade Manmade

**Prompt:** Detect all perfectly straight lines and linear elevated features in this grayscale terrain image. These are road embankments and man-made structures — they contrast with the natural curved features. Output a JSON array where each object has {"bbox_2d": [x1, y1, x2, y2], "label": "name"}. Coordinates should be pixel positions.

**Detections:** 8 (8 bboxes, 0 points)

**Bounding boxes (original pixel coords):**

- (3314, 1588, 3454, 2095) — road embankment
- (3508, 1373, 3655, 1527) — road embankment
- (3508, 1527, 3655, 1654) — road embankment
- (3508, 1654, 3655, 1776) — road embankment
- (3384, 1070, 3519, 1213) — road embankment
- (3980, 176, 5187, 634) — road embankment
- (3980, 579, 5187, 1037) — road embankment
- (3980, 993, 5187, 1434) — road embankment

**Geo-coordinates (EPSG:26914):**

- (648498.5, 3983948.5) — road embankment
- (648696.5, 3984339.5) — road embankment
- (648696.5, 3984199.5) — road embankment
- (648696.5, 3984075.5) — road embankment
- (648566.5, 3984648.5) — road embankment
- (649698.5, 3985384.5) — road embankment
- (649698.5, 3984982.5) — road embankment
- (649698.5, 3984576.5) — road embankment

**Raw output:**

```
```json
[
    {"bbox_2d": [612, 288, 638, 380], "label": "road embankment"},
    {"bbox_2d": [648, 249, 675, 277], "label": "road embankment"},
    {"bbox_2d": [648, 277, 675, 300], "label": "road embankment"},
    {"bbox_2d": [648, 300, 675, 322], "label": "road embankment"},
    {"bbox_2d": [625, 194, 650, 220], "label": "road embankment"},
    {"bbox_2d": [735, 32, 958, 115], "label": "road embankment"},
    {"bbox_2d": [735, 105, 958, 188], "label": "road embankment"},
    {"bbox_2d": [735, 180, 958, 260], "label": "road embankment"}
]
```
```

---

## Grounding: Slope Channel

**Prompt:** Detect all dark sinuous lines and curves in this terrain image. These are river channels. Output a JSON array where each object has {"bbox_2d": [x1, y1, x2, y2], "label": "name"}. Coordinates should be pixel positions. Return multiple tight bounding boxes for different segments.

**Detections:** 1 (1 bboxes, 0 points)

**Bounding boxes (original pixel coords):**

- (536, 1500, 3541, 2476) — name

**Geo-coordinates (EPSG:26914):**

- (647153.5, 3983802.5) — name

**Raw output:**

```
```json
[
  {"bbox_2d": [99, 272, 654, 449], "label": "name"}
]
```
```

---

## Grounding: Curvature Channel

**Prompt:** Detect all dark sinuous lines and curves in this terrain image. These are river channels. Output a JSON array where each object has {"bbox_2d": [x1, y1, x2, y2], "label": "name"}. Coordinates should be pixel positions. Return multiple tight bounding boxes for different segments.

**Detections:** 1 (1 bboxes, 0 points)

**Bounding boxes (original pixel coords):**

- (2913, 2145, 3232, 2366) — name

**Geo-coordinates (EPSG:26914):**

- (648187.5, 3983534.5) — name

**Raw output:**

```
```json
[
    {"bbox_2d": [538, 389, 597, 429], "label": "name"}
]
```
```

---

## Grounding: Elevation Channel

**Prompt:** Detect all dark sinuous lines and curves in this terrain image. These are river channels. Output a JSON array where each object has {"bbox_2d": [x1, y1, x2, y2], "label": "name"}. Coordinates should be pixel positions. Return multiple tight bounding boxes for different segments.

**Detections:** 8 (8 bboxes, 0 points)

**Bounding boxes (original pixel coords):**

- (49, 2233, 1132, 2901) — name
- (260, 2123, 1446, 2724) — name
- (444, 1869, 2805, 2768) — name
- (920, 1384, 3124, 2333) — name
- (1760, 1020, 3909, 2040) — name
- (3249, 1566, 3931, 2189) — name
- (3519, 722, 4700, 1682) — name
- (3947, 66, 4824, 871) — name

**Geo-coordinates (EPSG:26914):**

- (645705.5, 3983223.5) — name
- (645967.5, 3983366.5) — name
- (646739.5, 3983471.5) — name
- (647137.5, 3983931.5) — name
- (647949.5, 3984259.5) — name
- (648704.5, 3983912.5) — name
- (649224.5, 3984587.5) — name
- (649500.5, 3985321.5) — name

**Raw output:**

```
```json
[
  {"bbox_2d": [9, 405, 209, 526], "label": "name"},
  {"bbox_2d": [48, 385, 267, 494], "label": "name"},
  {"bbox_2d": [82, 339, 518, 502], "label": "name"},
  {"bbox_2d": [170, 251, 577, 423], "label": "name"},
  {"bbox_2d": [325, 185, 722, 370], "label": "name"},
  {"bbox_2d": [600, 284, 726, 397], "label": "name"},
  {"bbox_2d": [650, 131, 868, 305], "label": "name"},
  {"bbox_2d": [729, 12, 891, 158], "label": "name"}
]
```
```

---

## Qualitative Survey

**Prompt:** This is a hillshade rendering of a 1-meter resolution DEM (digital elevation model) of a river corridor. Identify all hydraulic and geomorphic features you can see. For each feature, describe its type, approximate location in the image, and shape. Look for: river channels, floodplains, terrace edges, ridges, point bars, oxbow lakes, levees, road embankments, or any other man-made structures.

**Response:**

Certainly! Here's an analysis of the hydraulic and geomorphic features visible in the provided hillshade rendering of the 1-meter resolution DEM of a river corridor:

### 1. **River Channels**
   - **Type**: The primary river channel is clearly visible.
   - **Approximate Location**: It runs diagonally from the bottom left corner towards the top right corner of the image.
   - **Shape**: The channel is sinuous, with meanders evident along its course. The channel appears relatively broad and flat in some sections, which is indicative of a floodplain.

### 2. **Floodplains**
   - **Type**: Floodplains are areas adjacent to the river channel that are periodically inundated.
   - **Approximate Location**: These are situated alongside the river, particularly on the left side of the channel and at the bottom of the image.
   - **Shape**: They appear as relatively flat, elongated areas immediately adjacent to the river channel, with a gentle slope away from the channel.

### 3. **Terrorre edges**
   - **Type**: Terrace edges are remnants of older floodplains or river channels.
   - **Approximate Location**: They can be seen as faint, linear features on the right side of the river channel.
   - **Shape**: They appear as slightly raised, linear edges with a subtle change in slope.

### 4. **Ridges**
   - **Type**: Ridges are narrow, elongated elevated features.
   - **Approximate Location**: A prominent ridge runs along the right side of the river, parallel to the channel.
   - **Shape**: The ridge has a linear, elongated shape that runs roughly parallel to the river channel.

### 5. **Point Bars**
   - **Type**: Point bars are small landforms that form on the inside of a river bend.
   - **Approximate Location**: Visible in various bends of the river channel.
   - **Shape**: They appear as smaller, crescent-shaped features on the inner bend of river meanders.

### 6. **Other Man-Made Structures**
   - **Type**: Road embankments or levees.
   - **Approximate Location**: There's a set of parallel linear features near the middle-right section of the image.
   - **Shape**: These features are straight and narrow, running roughly perpendicular to the river channel.

### Overall Interpretation:
- The hillshade rendering clearly shows a complex river corridor with natural features like meandering channels, floodplains, terraces, and points bars. Human influence is also evident with the presence of potential levees or road embankments.


---

## Summary

| Test | BBoxes | Points | Notes |
|------|--------|--------|-------|
| hillshade_channel | 5 | 0 | detections parsed |
| hillshade_ridges | 1 | 0 | detections parsed |
| hillshade_floodplain | 3 | 0 | detections parsed |
| hillshade_manmade | 8 | 0 | detections parsed |
| slope_channel | 1 | 0 | detections parsed |
| curvature_channel | 1 | 0 | detections parsed |
| elevation_channel | 8 | 0 | detections parsed |
