# NBA Fantasy Point Forecasting

Predicting NBA players' next-season fantasy scoring averages from their prior-season box-score stats, scored under a custom ESPN league format. My first end-to-end data project outside the classroom.

## Headline finding

**Advanced box-score rate stats (efficiency, usage, defensive rates) did not improve the forecast beyond a player's prior-season fantasy average and age.** Added together they overfit and made the model worse; added one at a time, none beat the simple baseline by more than noise.

That's the real result of the project, not a failure to reach it. The information those "sneaky" stats carry is already baked into how many fantasy points a player scored last season — a high-usage, efficient scorer already shows up as a high fantasy average. The honest answer to "do advanced stats add predictive value here?" turned out to be no, and I chose to report that rather than keep adding features until the number moved.

## Final model

**Prior-season fantasy average + age.** Linear regression, evaluated on a held-out test set.

| Model | Features | Test R² | Test MAE |
|---|---|---|---|
| v1 baseline | prior-season average | 0.744 | 5.73 |
| **v2 (final)** | **average + age** | **0.766** | **5.77** |
| v3 | average + age + age² | 0.765 | 5.77 |
| v4 | average + age + 7 rate stats | 0.743 | 5.98 |

Age was the one feature that helped — its negative coefficient (~-0.5 pts/year) captures the downhill side of the aging curve. Age² didn't help, because a minutes-filtered sample doesn't contain the full bell curve of careers.

## How it works

1. **Custom scoring function** — translates my ESPN league's multi-tier rules (per-stat weights plus stacked double-double / triple-double / quadruple-double bonuses) into Python. Validated against hand-calculated game logs before running on real data.
2. **Data collection** — two full seasons of league-wide game logs pulled via `nba_api`, filtered to the regular season.
3. **Filtering** — minimum games and minutes-per-game thresholds, set from a standard-error analysis rather than picked arbitrarily.
4. **Merge** — joined each player's season *N* stats to their season *N+1* fantasy average, producing 243 paired player-seasons.
5. **Modeling** — a series of linear regressions on a fixed train/test split, with correlation + VIF checks for multicollinearity and a stepwise test of each advanced stat.

## Documented limitations

- **Survivorship bias** — the merge only keeps players who cleared the threshold in *both* seasons, so the model never trains on players whose seasons collapsed from injury or lost role. It skews optimistic on players about to decline.
- **Small sample** — 243 paired player-seasons, 49 in the test set. Differences under ~0.01 R² are within the noise; cross-validation would give more stable estimates.
- **Box-score ceiling** — these are lagging outcome stats. Prior-season average alone already explains ~74% of next season, and much of the rest (injuries, trades, role changes) isn't predictable from any box score.

## What I'd do next

The biggest lever isn't a fancier model, it's better data — player-tracking signals (time in the restricted area, shot quality, defensive matchups, on/off splits) that explain *why* the box score came out the way it did. A tree-based model would also be worth trying, specifically to test whether advanced stats have *conditional* value (mattering for young players but not veterans) that a linear model structurally can't capture.

## Running it

Put the notebook and `player_birthdates.csv` in the same folder before running. The CSV is cached birthdate data — it's there to skip a slow (3M+ row) API call for player ages.

**Requirements:** Python 3, `nba_api`, `pandas`, `numpy`, `scikit-learn`, `matplotlib`, `seaborn`

## Repo contents

- `nba-fantasy-forecasting.ipynb` — the full project: pipeline, models, and inline documentation (stage summaries, a debugging log, and a concepts log)
- `player_birthdates.csv` — cached player birthdates
