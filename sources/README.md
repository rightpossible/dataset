Place **lawfully redistributable** PDFs or EPUBs here (public domain or compatible Creative Commons), then render them to PNGs. **Large files are not committed** — they are listed in `SOURCES.json` for attribution and licensing only.

```bash
pip install -r requirements.txt
python scripts/render_pdf.py --pdf-dir dataset/sources --out-dir dataset/original --dpi 200 --grayscale --only mybook.pdf
```

Record each file in `SOURCES.json` (copy from `SOURCES.example.json`). Do not commit copyrighted material without permission.
