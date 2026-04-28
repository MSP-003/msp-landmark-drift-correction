# MSP Landmark Drift Correction Tool

Interactive rigid-body 3D drift correction for longitudinal confocal imaging data. Designed for in-vivo imaging studies where the same animal is re-imaged across multiple sessions (days to weeks), this tool corrects both translational and rotational drift by letting users click on recognizable structures (e.g., amyloid plaques) across timepoints, then computes and applies the optimal rigid body transformation. It reads and writes Imaris `.ims` files directly — no intermediate format conversions required.

---

## Quick Start

```bash
pip install -r requirements.txt
python landmark_drift_correction.py
```

This launches the interactive mode. Select **[1] Single file** or **[2] Batch folder**, pick your files via the dialog, and the landmark picker window opens automatically.

You can also use command-line mode:

```bash
python landmark_drift_correction.py -i <input_folder> -o <output_folder>
python landmark_drift_correction.py -i <input_folder> -o <output_folder> --file m05_pos1
```

---

## Workflow

### 1. Landmark Placement
The tool opens a fullscreen window showing **maximum intensity projections (MIPs)** of every timepoint side by side. Click the same recognizable structure in each timepoint to define a landmark correspondence. Repeat for **3–5 landmarks** spread across the field of view.

Alternatively, click **Auto-place** to automatically detect bright structures and match them across all timepoints. Use **Shift+click** to correct any misplaced markers.

### 2. Sub-Voxel Refinement
After clicking "Done File," each user click is refined to sub-voxel precision using **local phase correlation** — a small cube around each landmark is extracted and cross-correlated between T0 and Tn. A **confidence overlay** briefly displays showing match quality (green = strong, red = weak).

### 3. Rigid Body Fitting
The tool computes the optimal rotation matrix **R** and translation vector **t** using **SVD-based Procrustes alignment**. Outlier landmarks (those that don't fit the rigid model) are automatically rejected. Rotation is decomposed into in-plane spin (rx) and out-of-plane tilt (ry, rz), with tilt capped to protect thin Z-stacks.

### 4. Two-Pass Residual Correction
After the coarse landmark-based correction, a second pass extracts large cubes around each landmark in the roughly-aligned volumes and runs phase correlation to measure and correct any remaining drift — capturing what rotation capping couldn't handle.

### 5. Output
The corrected volume is written as an `.ims` (Imaris HDF5) file with:
- Expanded canvas (no data loss — all timepoints fit with padding)
- Updated metadata at all three IMS dimension locations
- Rebuilt resolution pyramid
- Correct physical coordinates (ExtMin/ExtMax) preserving voxel size

---

## Controls

| Action | Control |
|---|---|
| Place landmark | Left click |
| Undo last click | Right click |
| Correct a marker | Shift + click on marker |
| Zoom (synced) | Scroll wheel |
| Reset zoom | R |
| Skip timepoint | S |
| Toggle magnifier | M |
| Adjust contrast | `[` / `]` |
| Finish landmarks | D or "Done File" button |

### UI Controls
- **Max Rot ° slider** — adjustable rotation lock (default 15°, range 0.5–45°)
- **N slider** — number of landmarks for Suggest/Auto-place (3–12)
- **Suggest** — shows candidate positions as circles on T0
- **Auto-place** — detects and places landmarks across all timepoints automatically
- **Before/After Preview** checkbox — shows alignment comparison before writing

---

## Requirements

- Python 3.8+
- h5py
- numpy
- scipy
- matplotlib
- scikit-image

All packages are open-source and available via pip.

---

## Output Files

| File | Description |
|---|---|
| `*_aligned.ims` | Drift-corrected Imaris hyperstack |
| `landmark_correction_log_*.txt` | Full processing log with transforms and RMSE |
| `alignment_report_*.html` | Visual HTML summary (for batch runs) |

---

## Author

**Mark St. Pierre**
Neurology Discovery — Translational Models

---

## License

MIT
