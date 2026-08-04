# SIADS 699 — Dota 2 Movement Skill Prediction

Predicting individual player skill (rank tier) from spatio-temporal movement data in Dota 2.

## Team
- Gabriel Mejia
- Changwoo Kim
- Sung-jin Bae

## Project Overview
We use player movement trajectories from ranked Dota 2 replays to predict individual skill level while carefully controlling for lobby composition leakage.

- **Train set**: 433 Europe matches  
- **Holdout**: 100 matches (mostly Singapore) — tests whether the model travels  
- **Label**: `rank_tier` (medal × 10 + star)

## Current Status
- [x] Data collection & validation
- [x] Team-relative coordinate transform
- [x] Basic L1 → L2 feature pipeline
- [o] Full feature set from Feature Spec
- [ ] Teammate-only leakage control
- [ ] Modeling (XGBoost + SHAP)
- [ ] Final report & presentation

## Repository Structure
