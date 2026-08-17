# SIADS 699 — Dota 2 Movement Skill Prediction

**Team DotA Science**  
Gabriel Mejia · Changwoo Kim · Sung-jin Bae  
University of Michigan School of Information

Predicting player skill from **spatio-temporal movement** in Dota 2 ranked replays, without using gold, XP, KDA, or win/loss.

## Project status

- [x] Data collection and validation (533 matches, ~5k labeled players)
- [x] Team-relative coordinate transform (`own_depth`, `own_lateral`)
- [x] L1 (per-timestep) → L2 (player-match) feature pipeline
- [x] Phase × lane exploratory analysis
- [x] XGBoost models + feature ablations + SHAP
- [x] Final report

## Repository layout

```
notebooks/
  00_replay_parsing_prototype.ipynb   # Clarity-based position extraction (prototype)
  01_feature_engineering.ipynb        # Main feature + analysis notebook
docs/
  Feature_Spec_v0.1.html              # Coordinate transform, zones, leakage notes
  Feature_Spec_Extensions.html
  README_data.md                      # Data dictionary
data/
  positions_sample.parquet
  players_sample.csv
  matches_sample.csv
  README.md                           # optional short note
  sample/
    matches.csv                       # Match metadata + skill_label
    players.csv                       # Player keys, rank_tier, hero_id
features/
  l2_sample.csv                       # Small sample of player-match features
reports/
  SIADS699_Team_DotA_Science_Report.docx
  SIADS699_Team_DotA_Science_Report.pdf
  figures/                            # Report figures
```

Full position data (`positions.parquet`, ~7.2M rows) is **not** stored in git. See data access below.

## Quick start

```bash
python -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
```

1. Place sample CSVs under `data/sample/`.
2. For full analysis, download the position table into `data/` or point the notebook paths at your copy.
3. Open `notebooks/01_feature_engineering.ipynb` and run top to bottom.

Notebook paths are **relative** (`../data/sample`, `../features`). Adjust if you use Google Colab.

## Data access

Public ranked match metadata is included under `data/sample/`.

Full movement table used in the report:

```text
gs://siads699-dota/dataset/v1/
  positions.parquet
  players.csv
  matches.csv
  README.md
```

Example (University of Michigan / GCP auth as configured for the course):

```bash
gcloud auth login
gsutil -m cp -r gs://siads699-dota/dataset/v1/ ./data/
```

### Label definition

- Match-level `skill_label`: high vs low from lobby `avg_rank_tier`
- Player-level `rank_tier`: individual medal score (preferred for sensitivity checks)
- Training-heavy region: Europe; holdout slice: mostly Singapore

See `docs/README_data.md` and `docs/Feature_Spec_v0.1.html` for details, including the note that a large share of rank variance is explained by `match_id` (lobby composition). Features built from the other nine players can leak the answer.

## Methods

Raw `(x, y)` positions are converted to a team-relative frame so Radiant and Dire are comparable. We aggregate movement into player-match features (depth, speed, idle share, path efficiency, lane × phase time shares), then train XGBoost classifiers for high vs low skill. Evaluation emphasizes descriptive gaps, ablations, and SHAP rather than a single accuracy claim. Hero identity is treated as a control because role strongly affects pathing.

## Main result

Higher-rank lobbies move a bit faster and idle a bit less. Average depth and lane occupancy look similar across ranks. Movement alone carries signal above chance, but effects are **small**. Claims about skill-from-motion need hero controls and leakage checks.

## Report

See `reports/SIADS699_Dota2_Movement_Skill_Report.docx` (and PDF). Figures used in the report live under `reports/figures/`.

## Key rules we followed

1. Transform coordinates to the team-relative frame before any feature work.
2. Treat lobby leakage seriously; prefer self-focused movement features.
3. Deaths are a mask, not a skill feature.
4. No gold / XP / KDA / win-loss in the movement feature set.
5. No API keys or service-account JSON in this repository.

## Statement of work (summary)

| Person | Focus |
|--------|--------|
| Gabriel Mejia | Framing, feature-spec design, phase/lane analysis, report, mentor-feedback integration |
| Changwoo Kim | Collection at scale, storage layout, missing-replay bias checks |
| Sung-jin Bae | Modeling support, ablations/SHAP, visualization support |

## Course context

SIADS 699 Capstone, University of Michigan Master of Applied Data Science.
