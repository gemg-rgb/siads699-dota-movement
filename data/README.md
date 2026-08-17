# Sample movement data

Five ranked matches (mix of high/low lobbies) for running the notebook when the full GCS download is unavailable.

Files:
- `positions_sample.parquet` — player positions (~90k rows)
- `players_sample.csv` — player keys / rank_tier / hero_id
- `matches_sample.csv` — match metadata / skill_label

Copy these into the notebook working directory under `data/`:

```bash
mkdir -p data
cp positions_sample.parquet players_sample.csv matches_sample.csv data/
```

The feature-engineering notebook will detect the sample automatically if `positions.parquet` is missing.

This is enough to test the pipeline and plots. Numbers in the report used the full 533-match extract.
