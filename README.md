# SIADS 699 — Dota 2 Movement Skill Prediction

Predicting individual player skill (rank tier) from spatio-temporal movement data in Dota 2.

## Team
- Gabriel Mejia
- Changwoo Kim
- Sung-jin Bae

## Project Overview
We use player movement trajectories from ranked Dota 2 replays to predict individual skill level, while carefully controlling for lobby composition leakage.

- **Training set**: 433 Europe matches  
- **Holdout set**: 100 matches (mostly Singapore) — tests whether the signal travels across regions  
- **Label**: `rank_tier`

## Current Status
- [x] Data collection & validation
- [x] Team-relative coordinate transform
- [x] Basic L1 → L2 feature pipeline
- [ ] Full feature set from Feature Spec
- [ ] Teammate-only leakage control
- [ ] Modeling (XGBoost + SHAP)
- [ ] Final report & presentation

## Repository Structure
- `notebooks/` — Feature engineering and analysis notebooks
- `features/` — Processed feature tables (sample only)
- `docs/` — Feature specification and notes
- `data/` — Raw data (not committed)
- `reports/` — Final deliverables

## Key Rules
1. Always transform coordinates to team-relative frame before feature work.
2. ~76% of rank variation comes from `match_id` → teammate-only control is required.
3. Deaths are a mask, never a feature.
4. No gold / XP / KDA / win-loss in the movement feature set.
