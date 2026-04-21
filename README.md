# Document page dataset with synthetic water degradation

Paired **clean** page rasterizations and **synthetically degraded** versions under controlled, named severity levels. Suitable for OCR robustness, document restoration benchmarks, and related vision tasks. The degradation is **two-dimensional and procedural**; it does not simulate physical fluid mechanics, paper geometry, or capture hardware.

---

## Contents of this tree

| Path | Description |
|------|-------------|
| `sources/` | Optional provenance metadata for redistributable inputs (see `sources/README.md`). |
| `SOURCES.example.json` | Template for recording per-source license and URI; rename to `SOURCES.json` when publishing. |
| `original/page_XXXX.png` | Clean renders (grayscale, fixed DPI recorded in `manifest.csv`). Flat layout, or `original/<source_id>/…` when multiple documents are merged. |
| `light_degraded/`, `moderate_degraded/`, `severe_degraded/`, `extreme_degraded/` | Degraded counterparts; filenames and relative paths mirror `original/`. |
| `manifest.csv` | One row per (clean, degraded) pair with seeds and build metadata. |
| `demo/` | Small illustrative subset (same conventions). |

---

## Rasterization (clean pages)

Source material is paginated vector or reflow text (PDF or EPUB). Each page is rasterized at a fixed resolution.

**Geometry.** PDF page space uses typographic points; 1 pt = 1/72 inch. A uniform scale \(s = \mathrm{DPI}/72\) applied in both axes maps page coordinates to pixels, so physical sampling density matches the stated DPI.

**Color.** Renders are **grayscale** (single luminance channel per pixel, stored as PNG). Degradation code operates on three-channel images internally; single-channel inputs are replicated to BGR before processing.

**Implementation note.** Rasterization is performed with **PyMuPDF (MuPDF)** via an affine `Matrix(s, s)` and a grayscale colorspace; output is written as lossless PNG.

---

## Degradation model

Let \(I \in [0,255]^{H \times W \times 3}\) be the clean page in BGR order (last index \(c \in \{B,G,R\}\)). All blending is done in floating point; the final image is clipped to \([0,255]\) and quantized to 8 bits.

### Wetness field

Two independent fields \(U_1, U_2 \sim \mathcal{U}(0,1)\) are drawn on the pixel grid (seeded PRNG). Each is convolved with a **Gaussian kernel** whose odd width depends on \(\min(H,W)\) and preset constants. The Gaussian is **separable**; the library applies two 1D convolutions. When the kernel size is fixed and \(\sigma\) is left automatic, the implementation derives \(\sigma\) from the kernel width (OpenCV: \(\sigma = 0.3 \cdot ((k-1)/2 - 1) + 0.8\) for \(k\) the 1D kernel extent).

A combined field is \(W_\mathrm{noise} = \alpha U_1' + \beta U_2'\) (preset weights \(\alpha,\beta\)), min–max normalized to \([0,1]\), then compressed with an affine threshold and a power:

\[
W \leftarrow \mathrm{clip}\left( \frac{W_\mathrm{noise} - b}{s}, 0, 1 \right)^{\gamma}
\]

with preset parameters \(b\) (bar), \(s\) (span), \(\gamma\) (power).

### Pool mask

A second map accumulates **filled ellipses** at random centers, axis lengths (scaled from image size), rotation, and opacity. That map is Gaussian-smoothed and added to the wetness field with preset weights, then clipped to \([0,1]\). Denote the result \(M\).

### Tide (ring) term

Let \(G_k\) denote Gaussian smoothing with kernel scale \(k\). The tide channel is

\[
T \leftarrow \mathrm{clip}\left( \lambda \cdot G_{k'}\big(\,|\,M - G_{k'}(M)\,|\,\big),\, 0,\, 1 \right)
\]

with preset gain \(\lambda\). This emphasizes band-like structure between the field and a heavily smoothed version of itself.

### Appearance

A **base** image mixes document and white:

\[
B = \big( \eta I + (1-\eta) \cdot 255 \cdot \mathbf{1} \big) \odot m_\mathrm{BGR}
\]

(preset \(\eta\) via `base_img` / `base_white`, channel-wise multiplier \(m_\mathrm{BGR}\)).

**Staining** subtracts a constant BGR tint scaled by \(M\):

\[
S = B - t \odot (\sigma_t M)
\]

**Washout** pushes \(S\) toward white where \(M\) is large:

\[
W_\mathrm{wash} = S \odot (1 - \omega M) + 255 \cdot \omega M
\]

(preset \(\omega\)).

**Tide color** subtracts a BGR offset scaled by \(T\).

**Selective blur.** Let \(\tilde{W} = G_\sigma(W_\mathrm{wash})\) with Gaussian standard deviation \(\sigma\) (kernel derived from \(\sigma\) when size is implicit). Output:

\[
O = W_\mathrm{wash} \odot (1 - \mu M) + \tilde{W} \odot (\mu M)
\]

(preset \(\mu\)). Finally \(O \leftarrow \mathrm{clip}(O, 0, 255)\).

### Presets

Each named preset fixes \((\alpha,\beta,b,s,\gamma)\), ellipse counts and scales, \(\lambda\), \(\eta\), \(m_\mathrm{BGR}\), \(t\), \(\sigma_t\), \(\omega\), tide BGR, \(\sigma\), \(\mu\), and kernel-size rules. The full parameter vector for a release is implicit in the built files: same `preset` and `git_commit` in `manifest.csv` denote one fixed tuning. Qualitative levels:

| Preset | Nominal severity |
|--------|------------------|
| `light` | Mild staining; text largely intact. |
| `moderate` | Clear damage; partial readability. |
| `severe` | Strong washout and banding. |
| `extreme` | Maximal effect for this generator. |

---

## Randomness and reproducibility

- **Bit generator:** NumPy `Generator` (PCG64) from an integer seed.
- **Per-pair seed:** For `source_id`, page stem \(p\), and preset name \(k\),

  \[
  \texttt{seed} = \Big( \mathrm{SHA256}\big(\texttt{utf8}( \texttt{source\_id} \,\|\, p \,\|\, k )\big)_{0:8} \Big) \bmod 2^{31}
  \]

  interpreting the first eight digest bytes as big-endian unsigned integer.

- **`manifest.csv`** records `seed`, `preset`, `dpi`, relative paths, and `git_commit` when the corpus was built from a Git checkout (empty otherwise).

---

## Manifest schema

Columns:

`sample_id`, `source_id`, `page_stem`, `preset`, `seed`, `dpi`, `clean_path`, `degraded_path`, `git_commit`

Paths use `/` separators. `sample_id` identifies one degraded sample uniquely (typically encodes document id, page stem, and preset).

---

## Library mapping (summary)

| Component | Role |
|-----------|------|
| **PyMuPDF** | PDF/EPUB open; page rasterization at chosen DPI and grayscale colorspace. |
| **OpenCV** | Gaussian convolution; elliptical masks; I/O and color layout (BGR); optional gray/RGBA to BGR conversion before degradation. |
| **NumPy** | Floating-point arrays, uniform sampling, elementwise algebra, clipping, broadcasting of \(M\) over color channels. |

---

## Limitations

- No modeling of lighting, substrate texture, ink bleed physics, or geometric page warp.
- Appearance is tuned for document-like contrast; extreme layouts or heavy halftones may behave differently.
- Re-running the pipeline under a different NumPy major version could in principle alter PRNG draws even for the same integer seed; frozen releases should archive the built PNGs and manifest.

---

## License and provenance

Synthetic images inherit the **license of the source pages** used to build them. Record every redistributable source in `SOURCES.json` (see `SOURCES.example.json`). If this tree is published together with software, that software carries its own `LICENSE`; the dataset files themselves follow the source-document terms above.
