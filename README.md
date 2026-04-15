# Micro-Scale Displacement Measurement with Computer Vision

**Forming Systems Project | TU Clausthal | WiSe 2024/2025**
**Master Program: Intelligent Manufacturing**
**Team (Group):** Nikhil Vinayagamurthy, Raghav Dixit, Kevin Kurisinkal Reji, Mudabbir Ahmed Khan, Jashanjot Singh

---

## Project Overview

Developed a non-contact, computer vision–based optical measurement tool to automatically detect and quantify **micro-scale deformation** in lightweight hybrid sandwich sheet specimens subjected to bending. The project addresses a key challenge in forming systems research: conventional contact-based methods cannot reliably measure the relative in-plane layer displacement that occurs at the micrometer scale during bending of layered composite materials used in aerospace and automotive applications.

The final algorithm - a **morphology-based deformation tracking and displacement quantification pipeline** - was validated across 14 samples spanning 3 different specimen parts, achieving reliable crack path tracking and sub-pixel displacement measurement.

---

## Objectives

- Detect and track vertical crack paths at the layer interface of sandwich sheet specimens
- Quantify top, bottom, and total midpoint displacement in pixels, micrometers, and millimeters
- Produce visual overlays, displacement profile graphs, and structured CSV outputs automatically
- Build a robust tool resistant to surface imperfections like oxidation and scratches

---

## Algorithm Development

### Algorithm 1 — Blob Detection (Failed Approach)

The first approach attempted to detect physical reference markers (blobs) applied to specimen edges and measure pixel displacement between them. However, it detected **685 marks instead of the actual 2**, with surface scratches and oxidation residue causing false positives. This method was deemed unreliable for industrial specimens and was discarded.

### Algorithm 2 — Morphology-Based Deformation Tracking ✅

The final algorithm measures the perpendicular distance between a static reference line and the tracked deformation path of the specimen. By tracking the crack line rather than discrete points, it becomes robust against surface imperfections and delivers precise sub-pixel displacement data across the full specimen height.

---

## System Pipeline

### Step 1 — User Settings

```python
IMG_PATH = "/path/to/specimen.tif"
PIXEL_TO_MICRON = 1200 / 2181   # calibration scale factor
```

The calibration factor converts pixel measurements to physical units (micrometers, millimeters).

### Step 2 — Crack Detection (`detect_vertical_cracks`)

| Operation | Purpose |
|---|---|
| Grayscale conversion | Removes colour information |
| Gaussian blur (5×5) | Reduces noise |
| Black-hat morphology (21×21 kernel) | Enhances dark crack regions |
| Otsu thresholding | Converts to binary mask |
| Morphological opening (3×3) | Removes noise |
| Vertical structuring element (3×35) | Isolates vertical crack structures |
| Region mask (top & bottom thirds) | Limits detection to silver interface layers |

### Step 3 — Crack Path Tracking (`detect_seam_from_mask`)

- Scans each pixel row for crack locations
- Tracks the nearest crack to the previous row's position — prevents jumping between multiple cracks
- Interpolates missing segments for continuity
- Applies a 9-point moving average for smooth path representation

### Step 4 — Displacement Calculation

```python
# Reference line = median X position of crack path
x_ref = int(np.median(seam[:, 0]))

# Measure deviation at quarter-height (top) and three-quarter-height (bottom)
top_px  = abs(top_mid[0]  - x_ref)
bottom_px = abs(bottom_mid[0] - x_ref)

# Convert to physical units
top_um = top_px * PIXEL_TO_MICRON
total_mm = (top_um + bottom_um) / 1000
```

### Step 5 — Outputs

Three outputs are automatically saved per specimen:

| Output | Description |
|---|---|
| `*_overlay.jpg` | Original image with crack pixels (red), reference line (blue), and tracked path (yellow) |
| `*_graph.jpg` | Displacement profile: X = displacement (µm), Y = height (pixels) |
| `*_results.csv` | Tabulated top, bottom, and total displacement in px / µm / mm |

---

## Code

**View Python Script:** [`CV3_WORKING CODE.txt`](CV3_WORKING%20CODE.txt)

**Core beta parameter formula for thermistor-style calibration:**

```python
# Pixel to micron conversion at acquisition calibration
PIXEL_TO_MICRON = 1200 / 2181

# Displacement in physical units
displacement_um = displacement_px * PIXEL_TO_MICRON
displacement_mm = displacement_um / 1000
```

---

## Results

Measurement results across all 14 samples (3 parts):

| Part | Sample ID | Top (µm) | Bottom (µm) | Total (µm) | Total (mm) |
|------|-----------|----------|-------------|------------|------------|
| Part 1 | Sample 1 | 103.69 | 89.20 | 192.88 | 0.1929 |
| Part 1 | Sample 2 | 129.30 | 91.88 | 221.18 | 0.2212 |
| Part 1 | Sample 3 | 71.53 | 73.18 | 144.70 | 0.1447 |
| Part 1 | Sample 4 | 81.80 | 36.86 | 118.66 | 0.1187 |
| Part 1 | Sample 5 | 69.94 | 17.06 | 86.99 | 0.0870 |
| Part 2 | Mark 1 | 105.09 | 2.20 | 107.29 | 0.1073 |
| Part 2 | Mark 2 | 54.47 | 210.73 | 265.20 | 0.2652 |
| Part 2 | Mark 3 | 13.21 | 12.66 | 25.86 | 0.0259 |
| Part 2 | Mark 4 | 38.51 | 67.13 | 105.64 | 0.1056 |
| Part 3 | Mark 1 | 33.01 | 40.72 | 73.73 | 0.0737 |
| Part 3 | Mark 4 | 143.60 | 121.05 | 264.65 | 0.2646 |
| Part 3 | Mark 5 | 26.96 | 30.26 | 57.22 | 0.0572 |
| Part 3 | Mark 6 | 79.78 | 23.11 | 102.89 | 0.1029 |
| Part 3 | Mark 8 | 115.54 | 0.55 | 116.09 | 0.1161 |

**Full results file:** [`CV3_results.csv`](CV3_results.csv)

---

## Sample Visual Outputs

**Mark 2 Overlay — Part 1**
| ![Mark 2](rover-1.jpg) | ![Mark_overlay](rover-2.jpg) |
Crack pixels highlighted in red, reference line in blue, tracked crack path in yellow.

**Mark 2 Displacement Graph — Part 1**
Blue curve shows the full displacement profile; red horizontal lines indicate measured midpoint displacements at top and bottom.

---

## Key Functions Implemented

| Function | Description |
|---|---|
| `detect_vertical_cracks(img)` | Morphological pipeline to isolate vertical crack masks |
| `detect_seam_from_mask(mask)` | Row-by-row crack path tracker with interpolation and smoothing |
| `main()` | Full pipeline: load → detect → measure → save overlay + graph + CSV |

---

## Tools & Libraries

| Library | Usage |
|---|---|
| OpenCV (cv2) | Image I/O, morphological operations, thresholding, drawing |
| NumPy | Array operations, interpolation, convolution smoothing |
| Matplotlib | Displacement profile graph generation |
| Pandas | Structured CSV output |
| OS | File path management, output folder creation |

---

## Conclusion

A morphology-based deformation tracking algorithm was successfully developed to automatically extract vertical crack paths and measure midpoint deformation in hybrid sandwich sheet specimens. The method combines morphological enhancement, statistical segmentation, and continuity-constrained tracking to produce reliable visual and numerical displacement results. The approach is robust to surface imperfections, efficient to run, and directly applicable to engineering deformation analysis in forming systems research.

---

## Affiliation

**Technische Universität Clausthal**
Master Program — Intelligent Manufacturing
Integrated Forming Systems Project | WiSe 2024/2025
