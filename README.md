# Dataset directory

This folder is the **dataset root** for full builds: `clean/` (rendered pages), `degraded/` (synthetic water damage), and `manifest.csv` (paired paths and metadata). A small checked-in example lives under [`demo/`](demo/README.md).

| Path | Role |
|------|------|
| [`sources/`](sources/README.md) | Redistributable PDF/EPUB inputs before rendering |
| [`SOURCES.example.json`](SOURCES.example.json) | Copy to `SOURCES.json` and record licenses and URLs |
| `clean/<source_id>/page_XXXX.png` | Rendered clean pages (`scripts/render_pdf.py`) |
| `degraded/<preset>/<source_id>/page_XXXX.png` | Degraded copies (`scripts/build_dataset.py`) |
| `manifest.csv` | One row per clean/degraded pair (when you build at this root) |

Preset names and reproducibility are defined in [`DATASET.md`](../DATASET.md).

---

## Reset (clean rebuild)

A **reset** means deleting outputs from a previous run so the next run does not mix old pages, DPIs, or code versions with new ones. It does **not** remove your source PDFs/EPUBs in `sources/` unless you delete them yourself.

**What to remove for a full reset of the generated corpus** (paths are relative to this `dataset/` folder):

- `clean/`
- `degraded/`
- `manifest.csv` (if present)

**Windows (PowerShell), from the repository root:**

```powershell
Remove-Item -Recurse -Force dataset\clean, dataset\degraded -ErrorAction SilentlyContinue
Remove-Item -Force dataset\manifest.csv -ErrorAction SilentlyContinue
```

Then rerun `scripts/render_pdf.py` and `scripts/build_dataset.py` with the same `--dpi` you intend to document in the manifest.

**When to reset:** before changing DPI or presets for a published snapshot; after fixing `water_damage.py` if you need a consistent corpus; or before a full re-render from a new set of sources.

---

## Degradation (four presets per page)

**Degradation** here means synthetic water/flood-style damage applied to each clean page image by `water_damage.apply_water_fade`, not camera noise or scan artifacts. The batch step is `scripts/build_dataset.py`, which:

1. Walks every `page_*.png` under `clean/`.
2. For each page, writes **four** outputs under `degraded/<preset>/…` — default presets: `light`, `moderate`, `severe`, `extreme`.
3. Appends one row per pair to `manifest.csv`, including a **deterministic seed** per `(source_id, page_stem, preset)` and the **git commit** of the repo when the manifest was built (see [`DATASET.md`](../DATASET.md)).

So each clean page typically yields **four** degraded images (one per preset), unless you limit with `--presets` or `--max-samples`.

---

## Publishing to GitHub while generation is still running

You can push **code and docs** anytime. For **binary outputs** (`clean/`, `degraded/`, large `manifest.csv`), choose one of these patterns:

1. **Wait for a consistent snapshot** — Easiest for reproducibility: finish render + `build_dataset.py`, then commit once. Avoid committing half-finished `degraded/` trees unless you are okay documenting that commit as partial.
2. **Incremental pushes** — If you must push before completion, prefer committing **completed source folders** (e.g. one `source_id` at a time) and regenerate `manifest.csv` after the full build, or accept that early commits lack the final manifest rows.
3. **Large files** — Plain git is painful for many large PNGs. Use **[Git LFS](https://git-lfs.com/)** for tracked images, or ship a **zip** via **GitHub Releases** / **Zenodo** and keep only `demo/` plus `manifest.csv` samples in the repo. Details: [`DATASET.md`](../DATASET.md) (Hosting section).

The template [`.gitignore`](../.gitignore) includes commented lines for `dataset/clean/`, `dataset/degraded/`, and `dataset/manifest.csv` — uncomment them if you want the repo to track **code only** and host binaries elsewhere; leave them commented if you commit the full dataset in git.

---

## Quick commands (reference)

Render everything under `sources/`:

```bash
python scripts/render_pdf.py --pdf-dir dataset/sources --out-dir dataset/clean --dpi 200
```

Render only specific files:

```bash
python scripts/render_pdf.py --pdf-dir dataset/sources --out-dir dataset/clean --dpi 200 --only book1.pdf book2.epub
```

Build degraded images and `manifest.csv`:

```bash
python scripts/build_dataset.py --dataset-root dataset --dpi 200
```
