# Bunnie's IRIS Scripts

This folder contains scripts developed by **Andrew "bunnie" Huang** as part of the Infra-Red In-Situ (IRIS) Inspection of Silicon toolchain. These scripts are used in our automated IRIS-based functional cell-classification pipeline for tasks such as GDS-to-image conversion, IRIS image stitching, alignment, and cell-level data extraction.

All scripts in this folder are **authored by Andrew "bunnie" Huang** and are included here solely to support the reproducibility of the results reported in our AICAS paper. Full credit and copyright for these scripts belong to their original author.

---

## Origin & Attribution

The scripts contained in this folder originate from the following public repositories. Find all the scripts needed at the GitHub repositories linked below:

| Repository | Author | Purpose | Link |
|---|---|---|---|
| `iris-stitcher` | Andrew "bunnie" Huang | Stitching overlapping IRIS microscopy images into a high-resolution composite image of the whole IC | https://github.com/bunnie/iris-stitcher |
| `iris-layout` | Andrew "bunnie" Huang | Converting GDS layout files to PNG images, IRIS–blueprint image alignment (Fourier–Mellin + OpenCV refinement), and cell-level image extraction | https://github.com/bunnie/iris-layout |

Related resources:

- **IRIS technique (arXiv paper):** A. "bunnie" Huang, *"Infra-Red, In-Situ (IRIS) Inspection of Silicon,"* arXiv:2303.07406, Mar. 2023. https://arxiv.org/abs/2303.07406
- ** Bunnie's IRIS blog category:** https://www.bunniestudios.com/blog/category/iris/

If you use any of these scripts, **please cite Bunnie's original work** in addition to our paper.

---

## Role in Our Pipeline

These scripts implement several stages of the data-curation portion of the pipeline described in Section III.A of our paper:

1. **GDS → PNG conversion** (Section III.A.1) — Isolated poly-layer GDS files of each functional block are converted into rasterized blueprint images using bunnie's script (originally from `iris-layout`), which leverages Python's `gdspy` library. Structured annotation metadata (bounding boxes, class labels, cell names) is exported as JSON alongside the images.
2. **IRIS image stitching** (Section II) — Overlapping IRIS captures (each corresponding to a ~0.8 mm region of the die) are stitched into a single high-resolution composite image using bunnie's `iris-stitcher` algorithm.
3. **IRIS ↔ blueprint alignment and scaling correction** (Section III.A.3) — Fourier–Mellin Transform-based rotation/scale estimation followed by OpenCV-based refinement to align physical IRIS crops with their corresponding GDS blueprint images.
4. **Cell-level data extraction** (Section III.A.4) — Using the aligned IRIS images together with the exported JSON annotations, cell-image regions are cropped and assigned functional-class labels (logic, flip-flop, filler) to construct the final 246,410-image cell-level dataset.

---

## Folder Contents

```
Bunnie_Huang_IRIS_Processing_Scripts/
├── <GDS_to_PNG>.py               # Converts poly-layer GDS files of each functional block into rasterized blueprint images.
├── <extract_dataset_cell>.py     # Applies Fourier–Mellin Transform to align physical IRIS crops with their corresponding GDS blueprint images, followed by cell extraction.
├── <techbase>.py                 # Base class TechBase providing a common interface for any PDK. 
├── <sky130>.py                   # SKY130A PDK-specific subclass of TechBase.
├── <palette>.py                  # HSV/RGB color-mapping utilities for cell/category visualization.
├── <prims>.py                    # Geometry primitives (Point, Rect) and helper operations used by layout/schema utilities.
└── <schema>.py                   # Defines the Schema class, which parses LEF (Library Exchange Format) files from the PDK to build a JSON database (db.json) of standard-cell macros.
```

---

## Requirements

The scripts rely on the following Python libraries (see the original repositories for full requirements):

- `gdspy` — GDS file parsing
- `numpy`
- `opencv-python` (`cv2`) — alignment refinement
- `Pillow` (`PIL`) — image I/O
- `matplotlib` — visualization (optional)

Install with:

```bash
pip install gdspy numpy opencv-python Pillow matplotlib
```

For IRIS image stitching, please refer to `iris-stitcher` for the complete environment and stitching-specific dependencies.

---

## Usage

Each script's usage generally follows the conventions established in Bunnie's original repositories with minimal modification. For detailed CLI options, please refer to the docstrings within each script and the corresponding upstream README:

- https://github.com/bunnie/iris-stitcher
- https://github.com/bunnie/iris-layout

---

## License

The scripts in this folder retain the license terms of their original upstream repositories (`iris-stitcher` and `iris-layout`). Please consult each upstream repository for the applicable license.

Any modifications made in the context of our AICAS pipeline are noted at the top of the respective script (where applicable) and are provided under the same license as the upstream source.

---

## Citation

If you use these scripts, please cite both Bunnie's original IRIS work and our AICAS paper:

**IRIS (original technique):**
```bibtex
@misc{huang2023iris,
  title  = {Infra-Red, In-Situ (IRIS) Inspection of Silicon},
  author = {Huang, Andrew},
  year   = {2023},
  eprint = {2303.07406},
  archivePrefix = {arXiv},
  primaryClass  = {cs.CR},
  doi    = {10.48550/arXiv.2303.07406}
}
```

**Our AICAS paper:**
> *(Citation will be provided once the proceedings are published at the conference!)*

---

## Acknowledgments

We gratefully acknowledge **Andrew "bunnie" Huang** for developing and open-sourcing the IRIS inspection technique and the accompanying scripts, without which this work would not have been possible.
