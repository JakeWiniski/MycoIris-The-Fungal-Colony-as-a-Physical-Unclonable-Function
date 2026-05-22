# MycoIris

**Fungal colony morphology as a polymorphic analogue to the human iris for biometric-grade physical unclonable functions.**

MycoIris is a proof-of-concept system that demonstrates how the macroscopic growth patterns of pigmented fungal colonies can be segmented, normalized, and encoded using the same computational pipeline employed in human iris biometrics. The resulting binary encodings — called *mycocodes* — are highly reproducible for repeated measurements of the same colony and strongly discriminative across different colonies, establishing fungal morphology as a viable substrate for iris-style pattern analysis and physical verification.

A full technical report is included in this repository: [The Fungal Colony as a Polymorphic Analog to the Human Iris in Biometric Workflows](The%20Fungal%20Colony%20as%20a%20Polymorphic%20Analog%20to%20the%20Human%20Iris%20in%20Biometric%20Workflows.pdf).

---

## Motivation

In an environment where proof of physicality, provenance, and personhood are increasingly valued, biological systems offer unique advantages as physical unclonable functions (PUFs). Like the human iris, a fungal colony exhibits complex, radially organized, high-entropy patterning that emerges through self-directed growth rather than explicit design. Unlike the iris, however, fungal tokens are non-personal, externalized, and revocable — a compromised token can simply be destroyed and replaced, making the system function more like a cryptographic key than a biometric trait.

Fungal colonies also provide access to multiple orthogonal entropy channels within a single physical object: fine-scale texture from hyphal morphology, chromatic signatures from rare pigments like xylindein, and geometric features from colony boundary shape. These independent feature spaces can be combined, weighted, or selectively sampled depending on application requirements.

## Pipeline Overview

The processing pipeline directly parallels the stages of conventional iris recognition:

### 1. Colony Segmentation

The resin-embedded colony is isolated from its background using Hough circle detection, CIE Lab\* color analysis, and a hybrid darkness/greenness scoring function with adaptive Otsu thresholding. The bright central core ("pupil") is then detected and removed to isolate the pigment-rich annular region analogous to the iris stroma.

### 2. Polar Unwrapping

The segmented annulus is converted from Cartesian to polar coordinates via boundary-normalized radial sampling, producing a rectangular "rubber sheet" representation where columns correspond to angular position and rows to normalized radial distance. This step preserves circumferential texture while equalizing radial scale across eccentric boundaries.

### 3. Angular Canonicalization

A canonical angular orientation is determined by fusing multi-channel intensity profiles (G, a\*, L\*) across a mid-radial band. The unwrapped image is circularly shifted so that this reference direction defines column zero, enabling rotationally aligned comparison across specimens.

### 4. Binary Mycocode Encoding

A multi-channel Gabor filter bank (3 scales x 4 orientations across G, a\*, and L\* channels) is applied to the aligned image. Complex filter responses are decomposed into real and imaginary components and quantized into 2-bit phase codes with mask-aware validity, producing a compact binary tensor stored as a compressed `.npz` file.

### 5. Hamming Distance Analysis

Pairwise similarity is computed using mask-aware Hamming distance with a circular-shift sweep for rotation robustness. The analysis supports both single-pair comparison with per-channel diagnostics and batch mode for full pairwise evaluation across a specimen gallery.

### 6. Pupil-Boundary Fusion (Optional)

A rotation-invariant descriptor is extracted from the inner colony boundary using Fourier magnitude spectra. This geometric channel is fused with texture-based Hamming distances to provide an orthogonal fallback that stabilizes discrimination when the primary texture signal is degraded.

## Results

Eight resin-stabilized *Chlorociboria aeruginascens* colonies were each imaged twice under identical conditions, producing 16 images processed through the full pipeline.

| Metric | Value |
|---|---|
| Mean intra-token Hamming distance | 0.32 +/- 0.11 |
| Mean inter-token Hamming distance | 0.68 +/- 0.01 |
| Decision threshold | ~0.50 |
| Observed misclassifications | 0 / 120 |
| Nearest-neighbor accuracy | 8 / 8 (all replicates correctly matched) |
| Entropy per bit | 0.90 bits |
| Total encoded bits per specimen | ~1.57 x 10^6 |

The clear bimodal separation between intra-token and inter-token distances, with no observed overlap at the decision threshold, confirms that each colony produces a distinct, reproducible digital signature. Every replicate scan was correctly identified as its token's nearest non-self neighbor.

### Notebooks

**MycoIris MycoCode Processing.ipynb** — End-to-end pipeline for a single specimen image:

- Step 1: Colony segmentation and background removal
- Step 2: Bright-core ("pupil") detection and removal
- Step 3: Polar unwrapping with boundary-normalized sampling
- Step 4: Angular canonicalization and mycocode v0.1 feature extraction
- Step 5: Iris-style binary encoding via multi-channel Gabor filter bank
- Step 6: Mycocode visualization (bit mosaic and per-filter views)

**MycoIris Hamming Distance.ipynb** — Batch comparison and analysis:

- Pairwise and batch mask-aware Hamming distance computation
- Rotation-robust similarity scoring
- Distance matrix heatmap and nearest-neighbor analysis
- Pupil-boundary descriptor extraction and score fusion
- Discrimination metrics (AUC, d-prime) and ROC curve generation

## Getting Started

### Requirements

Python 3.8+ with the following packages:

- `numpy`
- `pandas`
- `matplotlib`
- `opencv-python` (`cv2`)
- `scikit-image`
- `scipy`
- `imageio`
- `scikit-learn` (optional, for ROC/AUC)

Install all dependencies:

```bash
pip install numpy pandas matplotlib opencv-python scikit-image scipy imageio scikit-learn
```

### Usage

1. **Update file paths.** Both notebooks use relative paths (e.g., `./data/sample.png` and `./data`) in their configuration sections. Update these to point to your image directory before running.

2. **Process specimens.** Open `MycoIris MycoCode Processing.ipynb` and run all cells for each specimen image. The pipeline will produce segmentation masks, unwrapped images, aligned outputs, and the final `*_mycocode_bits.npz` encoding.

3. **Compare specimens.** Once all specimens are encoded, open `MycoIris Hamming Distance.ipynb`, set `MYCOCODE_FOLDER` to the directory containing the `.npz` files, and run all cells. The notebook will compute pairwise distances, generate heatmaps, and report discrimination metrics.

## Species and Cultivation

The proof-of-concept uses *Chlorociboria aeruginascens*, a wood-decay fungus that produces the rare naphthoquinone pigment xylindein. Colonies were cultivated on malt-extract agar with cellulose-nitrate membrane overlays, dried, and permanently stabilized in UV-protective epoxy resin to create optically clear, archival-quality tokens. Imaging was performed using transmitted-light scanning at 1600 dpi. See the full report for detailed cultivation, stabilization, and imaging protocols.

## Citation

If you use this work, please cite:

> Winiski, J. (2026). *MycoIris: The Fungal Colony as a Polymorphic Analog to the Human Iris in Biometric Workflows.* Proof-of-concept report.

## License

This project is provided for research and educational purposes. See the repository for license details.
