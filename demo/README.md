# Demo subset

Small example: flat `page_*.png` files in `original/` and matching files in each `*_degraded/` folder, plus `manifest.csv`.

**Regenerate** (from project root):

```bash
pip install -r requirements.txt
python scripts/generate_synthetic_pdfs.py
python scripts/render_pdf.py --pdf dataset/sources/synthetic_letter_demo.pdf --out-dir dataset/demo/original --dpi 200 --grayscale
python scripts/build_dataset.py --dataset-root dataset/demo --dpi 200 --presets light,moderate,severe,extreme
```

See [`README.md`](../../README.md) and [`DATASET.md`](../../DATASET.md).
