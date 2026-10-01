# Micro-Scale Displacement Measurement with Computer Vision

**Python and OpenCV tool that measures micrometer-scale layer displacement in bent sandwich sheets from a single image, and writes overlay images, displacement graphs and CSV results automatically.**

![Overlay result](images/Mark2_overlay.jpg)
*Crack pixels in red, reference line in blue, tracked crack path in yellow.*

**Project:** Forming Systems Project, MSc Intelligent Manufacturing, TU Clausthal | Feb 2026
**Team:** Nikhil Vinayagamurthy, project manager | Raghav Dixit | Kevin Kurisinkal Reji | Mudabbir Ahmed Khan | Jashanjot Singh
**My role:** Project manager of the 5-person team.
<!-- TODO: add one sentence on your own technical work, for example which functions you wrote or tested -->

---

## Problem

When a hybrid sandwich sheet is bent, its layers shift against each other by a few micrometers. Contact-based methods cannot measure this displacement reliably. The goal was a non-contact tool that measures it automatically from specimen images.

## Method

### First approach: blob detection, discarded
Reference marks on the specimen edge were detected as blobs and their shift was measured. Scratches and oxidation on the surface caused **685 detections instead of the 2 real marks**, so the approach was not usable on real specimens.

### Final approach: morphology-based crack tracking
Instead of following single points, the tool follows the continuous crack line at the layer interface and measures how far it deviates from a reference line. Following a line makes the measurement robust against scratches and oxidation.

| Step | Operation | Purpose |
|---|---|---|
| 1 | Grayscale and 5x5 Gaussian blur | Remove color and noise |
| 2 | Black-hat morphology, 21x21 kernel | Enhance dark crack regions |
| 3 | Otsu threshold and 3x3 opening | Binary mask without small noise |
| 4 | Vertical structuring element, 3x35 | Keep only vertical crack structures |
| 5 | Region mask on top and bottom thirds | Limit detection to the interface layers |
| 6 | Row-by-row tracking, interpolation, 9-point moving average | Continuous, smooth crack path |
| 7 | Deviation from the median reference line at 1/4 and 3/4 height | Top, bottom and total displacement |
| 8 | Pixel to micrometer conversion with a calibration factor | Physical units |

## Results

- Tested on **14 samples**.
- For every specimen the tool saves an overlay image, a displacement profile graph and a CSV file with top, bottom and total displacement in px, µm and mm, with no manual post-processing.

Example results for Part 1:

| Sample | Top displacement in µm | Bottom displacement in µm | Total displacement in µm |
|---|---|---|---|
| 1 | 103.69 | 89.20 | 192.88 |
| 2 | 129.30 | 91.88 | 221.18 |
| 3 | 71.53 | 73.18 | 144.70 |
| 4 | 81.80 | 36.86 | 118.66 |
| 5 | 69.94 | 17.06 | 86.99 |

All results: [results/CV3_results.csv](results/CV3_results.csv)

![Displacement graph](images/Displacement_graph.png)
*Displacement profile over specimen height. The red lines mark the measured top and bottom displacement.*

### Next steps
- Compare the results against an independent reference measurement.
- Refine crack edge detection beyond whole-pixel resolution.

## Images to add
- A short GIF showing input image, crack mask, tracked path and final overlay.
- A side-by-side of the blob detection result with its false detections next to the final crack tracking result.

## Repository structure

```
src/measure_displacement.py     full pipeline
results/CV3_results.csv         results for all samples
images/                         overlays, graph, image capture setup
requirements.txt
README.md
```

## How to run

```bash
git clone https://github.com/Nikhilvinayagamurthy/Measurement-of-Micro-Scale-Displacement-Using-Computer-Vision.git
cd Measurement-of-Micro-Scale-Displacement-Using-Computer-Vision
pip install -r requirements.txt
```

Set `IMG_PATH` and `PIXEL_TO_MICRON` at the top of `src/measure_displacement.py`, then run:

```bash
python src/measure_displacement.py
```

The overlay, graph and CSV are saved for the input image.

## Tools
Python, OpenCV, NumPy, Pandas, Matplotlib

---
Technische Universität Clausthal | MSc Intelligent Manufacturing
