# Demo subset

This folder is a **minimal** example you can keep in git for supervisors and reviewers:

- `clean/demo_source/page_0001.png` — synthetic clean page
- `degraded/<preset>/demo_source/page_0001.png` — synthetic water damage for each preset
- `manifest.csv` — four rows (one per preset)

Regenerate after changing `water_damage.py`:

```bash
python scripts/build_dataset.py --dataset-root dataset/demo --dpi 200
```

For ~1000 samples, render many PDF pages into `dataset/clean/` and run the same script with `--dataset-root dataset` (or your chosen root). See the repository [`README.md`](../../README.md) and [`DATASET.md`](../../DATASET.md).
