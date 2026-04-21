# Dataset directory

This folder is the **dataset root** for full builds: **`original/`** (grayscale page PNGs from PDFs), **one folder per severity** (e.g. `moderate_degraded/`, `severe_degraded/`, `extreme_degraded/`), and **`manifest.csv`**. A small example lives under [`demo/`](demo/README.md).

| Path | Role |
|------|------|
| [`sources/`](sources/README.md) | Redistributable PDF/EPUB inputs before rendering |
| [`SOURCES.example.json`](SOURCES.example.json) | Copy to `SOURCES.json` and record licenses and URLs |
| `original/page_XXXX.png` | Grayscale renders (default: flat; use `--source-subdirs` for `original/<source_id>/…`) |
| `<preset>_degraded/page_XXXX.png` | Synthetic water damage (same flat layout by default) |
| `manifest.csv` | One row per original/degraded pair (when you build at this root) |

Preset names and reproducibility are defined in [`DATASET.md`](../DATASET.md).

---

## Reset (clean rebuild)

A **reset** means deleting outputs from a previous run so the next run does not mix old pages, DPIs, or code versions with new ones. It does **not** remove your source PDFs/EPUBs in `sources/` unless you delete them yourself.

**What to remove for a full reset** (paths relative to this `dataset/` folder):

- `original/`
- `light_degraded/`, `moderate_degraded/`, `severe_degraded/`, `extreme_degraded/` (any `*_degraded/` folders you created)
- `manifest.csv` (if present)

**Windows (PowerShell), from the parent project root (`finalproject/`):**

```powershell
Remove-Item -Recurse -Force dataset\original -ErrorAction SilentlyContinue
Get-ChildItem dataset -Directory -Filter "*_degraded" | Remove-Item -Recurse -Force
Remove-Item -Force dataset\manifest.csv -ErrorAction SilentlyContinue
```

Then rerun `scripts/render_pdf.py` and `scripts/build_dataset.py` with the same `--dpi` you intend to document in the manifest.

---

## Degradation (presets → separate folders)

**Degradation** is synthetic water/flood-style damage via `water_damage.apply_water_fade`. `scripts/build_dataset.py`:

1. Walks every `page_*.png` in `original/` (files directly in that folder; if none, falls back to nested `original/<source_id>/…`).
2. For each page, writes one PNG per selected preset into **`moderate_degraded/`**, **`severe_degraded/`**, **`extreme_degraded/`** (defaults), not under a shared `degraded/<preset>/` tree.
3. Appends one row per pair to `manifest.csv` (deterministic seed per `(source_id, page_stem, preset)`; see [`DATASET.md`](../DATASET.md)).

Default **`--presets`** is `moderate,severe,extreme`. Override **`--preset-dirs`** if you need custom folder names.

---

## Publishing to GitHub

**Images and `manifest.csv` are intended to be committed** in this layout so others can browse originals and each severity side by side. Very large corpora may still need **[Git LFS](https://git-lfs.com/)** or a **Release zip** / **Zenodo** — see [`DATASET.md`](../DATASET.md). [`.gitignore`](.gitignore) only excludes bulky **PDF/EPUB** sources, not the PNG folders.

---

## Quick commands (reference)

Single PDF at `dataset/historyofworldwa01simouoft.pdf` → grayscale originals:

```bash
python scripts/render_pdf.py --pdf dataset/historyofworldwa01simouoft.pdf --out-dir dataset/original --dpi 200 --grayscale
```

One file from `sources/` (multiple PDFs need `--source-subdirs` or `--only one.pdf`):

```bash
python scripts/render_pdf.py --pdf-dir dataset/sources --out-dir dataset/original --dpi 200 --grayscale --only mybook.pdf
```

Build degraded folders and `manifest.csv`:

```bash
python scripts/build_dataset.py --dataset-root dataset --dpi 200
```
