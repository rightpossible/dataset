Place **lawfully redistributable** PDFs here (public domain or compatible Creative Commons), then render them to PNGs:

```bash
pip install -r requirements.txt
python scripts/render_pdf.py --pdf-dir dataset/sources --out-dir dataset/clean --dpi 200
```

Record each file in `SOURCES.json` (copy from `SOURCES.example.json`). Do not commit copyrighted material without permission.
