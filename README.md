# SIADS 699 — Dota 2 Movement Skill Prediction

**Team DotA Science**  
Gabriel Mejia · Changwoo Kim · Sung-jin Bae  
University of Michigan School of Information

Can you tell high-rank Dota 2 players from lower-rank ones using **movement alone** (no gold, XP, KDA, or win/loss)?

Short answer from this project: a little. Higher-rank lobbies move slightly faster and idle slightly less. Lane choice and average map depth look about the same. Movement features beat chance in an XGBoost model (AUC ≈ 0.79 on a simple split), but the gaps are small and need careful handling of lobby leakage and hero/role effects.

## Repository layout

```text
notebooks/
  00_replay_parsing_prototype.ipynb   # optional: one-match Clarity parse demo (needs Java 17)
  01_feature_engineering.ipynb        # main path: features, plots, model ablations
data/
  positions_sample.parquet            # 5-match sample (~1 MB) for offline runs
  players_sample.csv
  matches_sample.csv
  README.md                           # what the sample contains
  /samples
docs/
  Feature_Spec_v0.1.html              # coordinates, zones, leakage notes
  Feature_Spec_Extensions.html
  README_data.md                      # data dictionary
features/                             # notebook outputs (gitignored or small samples)
reports/
  SIADS699_Team_DotA_Science_Report.pdf
  SIADS699_Team_DotA_Science_Report.docx
  figures/                            # report PNGs
requirements.txt
```

The full position table (`positions.parquet`, ~7.2M rows) is **not** in git. Use GCS (below) or the sample files in `data/`.

## Quick start

```bash
python -m venv .venv
source .venv/bin/activate          # Windows: .venv\Scripts\activate
pip install -r requirements.txt
```

### Option A — Sample data (no cloud auth)

```bash
mkdir -p data
# if samples are already under data/, skip this
# otherwise copy positions_sample.parquet, players_sample.csv, matches_sample.csv into data/
```

Open `notebooks/01_feature_engineering.ipynb` and run top to bottom.:

```text
data/
features/
reports/figures/
```

If `positions.parquet` is missing, it automatically loads `positions_sample.parquet` (and the matching sample CSVs). That is enough to test the pipeline and plots. Report numbers used the full 533-match extract.

### Option B — Full dataset (GCS)

Needs the [Google Cloud SDK](https://cloud.google.com/sdk/docs/install) and an account with access to the project bucket:

```bash
gcloud auth login
mkdir -p data
gsutil -m cp -r gs://siads699-dota/dataset/v1/* data/
```

Or run the download cell at the top of `01_feature_engineering.ipynb` (Colab auth works there; elsewhere use `gcloud auth login` first).

Expected files:

```text
data/positions.parquet
data/players.csv
data/matches.csv
```

## Notebooks

| Notebook | Role |
|----------|------|
| `01_feature_engineering.ipynb` | **Main.** Load data → team-relative coordinates → L2 features → report figures → XGBoost ablations + SHAP |
| `00_replay_parsing_prototype.ipynb` | **Optional.** Download one `.dem`, parse positions with Clarity. Requires **Java 17+**. Not needed to regenerate report figures |

## Data access and ownership

- Match metadata and replay URLs were obtained through the public [OpenDota API](https://docs.opendota.com/).
- Positions were parsed from ranked All-Draft replays with [Clarity](https://github.com/skadistats/clarity-examples).
- Underlying game content is owned by Valve. This repo only ships **derived research samples** (feature tables / a few matches of positions), not a redistribution of Valve client assets.
- Full research extract used for the report:

```text
gs://siads699-dota/dataset/v1/
```

Access to that bucket is limited to accounts granted permission (course/team GCP). If `gsutil` returns `401 Anonymous caller`, use the sample files or ask a teammate who has access.

### Labels

- `skill_label` (match): high vs low from lobby average `rank_tier`
- `rank_tier` (player): individual medal score
- Europe-heavy training pool; holdout slice mostly Singapore

A large share of rank variance sits at the match/lobby level, so features built from the other nine players can leak the label. The feature sets used for modeling stay focused on each player’s own movement. See `docs/Feature_Spec_v0.1.html`.

## Methods (very short)

1. Convert raw `(x, y)` to team-relative `own_depth` / `own_lateral` so Radiant and Dire are comparable.
2. Mask short windows after fountain teleports (death proxy).
3. Aggregate to player-match features: depth, speed, idle share, path stats, lane × phase shares.
4. Compare high vs low lobbies; train XGBoost on movement-only features; ablate feature groups; inspect SHAP.

We do **not** use gold, XP, KDA, or win/loss in the movement feature set.

## Main result

Higher-rank lobbies move a bit more and idle a bit less. Average depth and lane occupancy stay similar. On a held-out player split, the best movement-only models reached about **AUC 0.79** and **accuracy 0.73**; a tiny depth/path/idle-only set was much weaker (~0.62 AUC). That is above chance, not product-ready skill detection. Lobby labels mix individual ranks, and hero/role can mimic skill in spatial data.

## Report

- PDF/DOCX: `reports/`
- Figures: `reports/figures/`
- Regenerated by running the report-figure cells in `01_feature_engineering.ipynb`

## Requirements

See `requirements.txt` (pandas, pyarrow, scikit-learn, xgboost, shap, matplotlib, jupyter, …).

## Statement of work (summary)

| Person | Focus |
|--------|--------|
| Gabriel Mejia | Framing, feature-spec design, phase/lane analysis, report, mentor-feedback integration |
| Changwoo Kim | Collection at scale, GCS layout, missing-replay checks, parsing pipeline |
| Sung-jin Bae | Modeling support, ablations/SHAP, visualization support, project coordination |

## Course

SIADS 699 Capstone · Master of Applied Data Science · University of Michigan
