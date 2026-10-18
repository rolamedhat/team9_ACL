# Report tables (from notebook sections 6–8)

## Data split

| Split | Training | Validation | Test | Purpose |
|---|---|---|---|---|
| **Main — chronological** (`split`) | 1950–2019 | 2020–2021 | 2022–2024 | Predict upcoming races from the past only; used for tuning, threshold, model comparison, ablation and XAI |
| **Robustness check — by era** (`era_split`) | 55 seasons | 10 seasons | 10 seasons | Check the models hold up across all eras |

Era split (five 15-season eras, seed 42, 11/2/2 seasons per era):

| Era | Validation | Test |
|---|---|---|
| 1950–1964 | 1951, 1962 | 1958, 1963 |
| 1965–1979 | 1971, 1976 | 1974, 1979 |
| 1980–1994 | 1992, 1993 | 1980, 1983 |
| 1995–2009 | 1999, 2009 | 1998, 2006 |
| 2010–2024 | 2017, 2019 | 2015, 2022 |

## Why each feature group was chosen

| Group | Hypothesis | Evidence |
|---|---|---|
| Grid (`grid_filled`, `grid_zero_flag`, `field_size`) | Starting position is the strongest pre-race signal; a driver with no recorded slot (grid 0) is assumed to start at the back; with more starters each driver's podium chance is lower | Podium rate falls steeply with grid position (4.9.12); 50–89% of front-row starters reach the podium at the circuits in Q1; the podium rate per season follows field size (4.9.12); decile plots (6.8) |
| Qualifying (`quali_position_filled`, `has_qualifying`, `quali_gap_pct`) | Qualifying measures one-lap pace before any grid penalty, and the Q1 time gap shows *how much* faster a driver is, not only the order | Grid and qualifying position differ for some drivers (Q1 table); decile plot of `quali_gap_pct` (6.8); `has_qualifying` lets the model tell real values from values filled from the grid |
| Driver form (last 5 races, career, circuit) | Strong drivers in strong cars keep finishing near the front, so recent and long-term podium rates carry over to the next race | Correlation table and decile plots (6.8, training seasons only) |
| Constructor form (last 5 races) | Car performance is shared by a team's drivers and persists from race to race | Correlation table and decile plots (6.8); Q3 shows clear, lasting differences between teams |
| Standings before the race | The championship table summarises a whole season of form up to the previous race | Correlation table and decile plots (6.8); built only from earlier races (6.7 check); freshness check (6.5.1) |
| Context (`driver_age`, `home_race`, `round`, `hybrid_era`) | Experience and era may change how results come about; `hybrid_era` because front-row conversion rose after 2014 (Q1) | Yes/no plot (6.8); `home_race` showed no effect once grid is controlled for (Q2), so it is kept only for the ablation to confirm and is a candidate to drop. `is_sprint_weekend` was dropped: sprints start in 2021, so it is 0 for every training row |

## Why each feature is valid at the pre-race cutoff

| Group | Features | Known before the race because… |
|---|---|---|
| Grid | `grid_filled`, `grid_zero_flag`, `field_size` | The starting grid is fixed before the start (penalties already applied) |
| Qualifying | `quali_position_filled`, `has_qualifying`, `quali_gap_pct` | Qualifying finishes before the race |
| Driver form | `drv_prev_races`, `drv_podium_rate_last5`, `drv_avg_finish_last5`, `drv_career_podium_rate`, `drv_circuit_podium_rate` | Built from earlier races only (`shift(1)`, sorted by date; checked in 6.7) |
| Constructor form | `cons_podiums_last5`, `cons_avg_finish_last5` | Same, per team |
| Standings | `drv_pre_points_share`, `drv_pre_wins`, `drv_pre_position`, `cons_pre_points_share`, `cons_pre_position` | Standings after the previous race of the season, not this one (checked in 6.7) |
| Context | `driver_age`, `home_race`, `round`, `hybrid_era` | Calendar, nationality and date of birth are known in advance |
