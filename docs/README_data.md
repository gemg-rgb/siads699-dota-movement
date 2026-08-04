# Dota 2 movement dataset — v1

Team DotA Science, SIADS 699. Built by `build_dataset_v1.py` from parse runs:
all_matches.

Position data only. **Feature engineering has not been done** — see
`Feature_Spec_v0.1.html` in the project folder for the proposed design.

## Files

| File | Rows | What it is |
|---|---|---|
| `positions.parquet` | 7,168,360 | One row per player per 2 seconds |
| `players.csv` | 5,330 | One row per player per match — labels and keys |
| `matches.csv` | 533 | One row per match — region, patch, length |

533 matches, 533 of them with player records.

## positions.parquet

| Column | Meaning |
|---|---|
| `match_id` | Join key |
| `tick` | Game tick, 30 per second |
| `sec` | Seconds since the first sample of the match |
| `player` | Clarity player index, 0-9 |
| `player_slot` | OpenDota slot: 0-4 Radiant, 128-132 Dire. **Use this to join, not `player`** |
| `x`, `y` | Raw world coordinates |

The index-to-slot mapping was verified on 200 players (both by Steam ID and by
hero) — see `Standup3_Details.md` section 2.3.

## players.csv — this is where the label lives

| Column | Meaning |
|---|---|
| `match_id`, `player_slot` | Join keys into positions.parquet |
| `account_id` | Player identity. **Group train/test splits on this** |
| `hero_id` | Key only — not a model feature (hero bias) |
| `rank_tier` | **The label.** medal x 10 + star; 1 Herald ... 8 Immortal. Legend starts at 50 |
| `leaver_status` | 0/1 fine, >=2 abandoned |
| `is_anonymous` | account_id missing. Was 0% in the pilot |

## matches.csv

`match_id`, `avg_rank_tier` (match average — only available here, not in the
match JSON), `skill_label`, `cluster`, `region`, `duration_s`, `start_time`,
`patch`, `status`, `run_id`.

## Do this before any feature work

Raw `x, y` cannot be used directly. Radiant spawns at low coordinates and Dire
at high ones, so the same number means the opposite thing for the two teams —
a model will learn which side you were on before it learns anything about skill.

```python
import numpy as np, pandas as pd

R, D = np.array([9515.0, 9975.0]), np.array([23451.0, 22746.0])   # fountains, measured
AXIS = D - R; L = float(np.linalg.norm(AXIS))
U = AXIS / L; N = np.array([-U[1], U[0]])
DIRE_SLOTS = {128, 129, 130, 131, 132}

def to_team_frame(df):
    """Raw x,y -> own_depth (0 = own fountain, 1 = enemy) and own_lateral
    (signed distance from the mid diagonal). Mirrored for Dire so the two
    teams are directly comparable. Do this before ANY feature work: raw x,y
    means the opposite thing for the two teams, and a model will learn side
    before it learns skill."""
    v = df[["x", "y"]].to_numpy() - R
    depth, lateral = (v @ U) / L, v @ N
    dire = df["player_slot"].isin(DIRE_SLOTS).to_numpy()
    df = df.copy()
    df["own_depth"]   = np.where(dire, 1.0 - depth, depth)
    df["own_lateral"] = np.where(dire, -lateral, lateral)
    return df
```

## Rules that travel with this data

1. **Group splits by `match_id` as well as `account_id`.** Ten players from one
   match share the whole game context; splitting them across folds lets a model
   memorise match-specific traits.

2. **76% of the variation in a player's rank is explained by which match they
   were in.** Matchmaking groups similar players, so any feature built from the
   other nine (team spread, distance to nearest enemy, ...) partly tells the
   model the lobby's skill level rather than the player's. Before trusting such
   features, run the teammate-only control: predict the target's rank using only
   the other nine players' features. Whatever it scores is the size of the
   shortcut.

3. **Deaths are a mask, not a feature.** A death is visible in the position data
   (the player teleports to their fountain), but death count is part of KDA and
   the leakage guide excludes outcome signals. Use it to drop dead time from
   movement statistics, never as an input.

4. **Speed is partly bought with gold.** Boots raise movement speed, so speed
   features smuggle economy into a movement-only model. Normalise, caveat, or
   ablate — do not use them unexamined.

5. **No gold, XP, KDA or win/loss in the movement feature set.** They belong to
   the separate economy baseline.

## Sampling frame

Ranked All Pick, 15+ minutes, no abandons, Perfect World (China) clusters
excluded, collected within patch 7.41d.

A replay can only be downloaded if OpenDota holds its replay key. Across the 541
Europe candidates, 427 did - an 83% yield. We tested whether the 88 unavailable
matches differ from the 427 we collected:

| Compared | With key | Without key | p |
|---|---|---|---|
| Average rank tier (median) | 43 | 41 | 0.151 |
| Match length (median) | 2,405 s | 2,540 s | 0.035 |
| High vs low label | - | - | 0.152 |

**Rank is not biased**, which is the one that matters for a skill label. Match
length shows a small difference - matches we could download run about two minutes
shorter at the median - but that is one significant result out of four tests, so
it does not survive a multiple-comparison correction and should be read as weak
evidence rather than an established fact. If it is real, the sample slightly
under-represents long games.

Power: with 427 against 88, only differences above 0.33 SD are detectable, which
is roughly 0.6 of a medal in rank terms. Smaller biases would not show up here.

Note the holdout is 85% Singapore, so it tests whether the model travels to
Singapore rather than to non-Europe generally.

## Reproducing or extending

Raw replays and match JSON are archived under `replays/` and `match_json/` in
the same bucket. Re-parsing costs about 14 seconds per replay, so extra fields
(facing angle, action state, combat events) can be added later without
re-collecting.
