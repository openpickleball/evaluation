# OPEN engine: initial evaluation, technical report

**Status: initial, not pre-registered.** On 2026-11-21 an earlier evaluation plan was rescoped to an initial evaluation on all existing data. The split date, metrics and comparisons were chosen before any result below was computed, but they were not locked, and the data include an earlier plan's holdout, which was seen before. Treat every result as evidence to plan the next, pre-registered test, not as a validation.

## Provenance

- Engine: `openpickleball-engine` 0.3.0.dev0 at commit `e62468c508126ed46b5a26a82f175ba17dd78371` (tag `eval-2026-11` in the engine repository).
- Data version: `28bcab9de3af` (sha256 prefix of the private processed match file).
- Bootstrap: 2000 resamples, seed 20261114, 95% percentile intervals.

## Data

- 1601 doubles matches, 189 players: league nights and one-day tournaments from one club, side-out scoring to 11. League games have no cap; tournament games are capped at 13.
- Tuning part (on or before 2026-08-23): 1087 matches (1033 league, 54 tournament) on 41 dates.
- Test part (from 2026-08-24): 514 matches (250 league, 264 tournament) on 11 dates. 79 players in the test part never played in the tuning part.

## Tuned configuration

- Side-out scale 0.868 (at bound: no), win calibration a 0.402 (at bound: no, n = 494).
- Fit window from 2026-02-15 to 2026-08-23; every other `Config` field at the engine default. Config fingerprint `e0430f2abfbe59de`.
- One-point team gap: calibrated win probability 0.996 for one side-out game to 11.
- Tuned Elo K = 16 (tuning part only).

## 1. Match outcome prediction (test part)

Scored: 285 test matches where all four players have at least 5 earlier matches, from 11 dates. Not scored: below burn-in 229. Intervals resample match dates.

| Metric | Tuned engine [95%] |
| --- | --- |
| Brier score (lower is better) | 0.2224 [0.2127, 0.2312] |
| Log loss | 0.6340 [0.6123, 0.6546] |
| Accuracy | 0.628 [0.579, 0.687] |
| Brier before calibration | 0.2351 [0.2166, 0.2527] |
| Point-share MSE | 0.03906 [0.03415, 0.04389] |
| Favourite-side point-share bias | -0.0143 [-0.0423, +0.0201] |

Baselines: every row is scored on the same matches, the tuned engine's scored test matches that every row also scores (285 matches, 11 league nights). Only the tuned engine row is calibrated. Intervals are 95% cluster-bootstrap intervals, resampling league nights (2000 resamples, seed 20261114).

| Model | Settings | Brier | Log loss | Accuracy |
|---|---|---|---|---|
| Engine, tuned | side-out scale 0.868, a = 0.402, scheduled replays | 0.2224 [0.2127, 0.2312] | 0.6340 [0.6123, 0.6546] | 0.628 [0.579, 0.687] |
| Engine, default config | Config(): side-out scale 2.9, a = 1, scheduled replays | 0.2253 [0.2101, 0.2425] | 0.6407 [0.6042, 0.6808] | 0.618 [0.572, 0.650] |
| Elo, tuned | K = 16, tuned on the tuning period | 0.2399 [0.2336, 0.2453] | 0.6726 [0.6599, 0.6836] | 0.582 [0.528, 0.633] |
| Elo, default | K = 32, start 1500, untuned | 0.2386 [0.2290, 0.2466] | 0.6698 [0.6497, 0.6864] | 0.579 [0.526, 0.631] |
| Glicko-2, tuned | tau 0.3, initial RD 350, rating period 14 days, volatility 0.06, tuned on the tuning period | 0.2294 [0.2204, 0.2355] | 0.6505 [0.6317, 0.6630] | 0.619 [0.573, 0.668] |
| Glicko-2, default | tau 0.5, initial RD 350, rating period 7 days, volatility 0.06, untuned | 0.2306 [0.2203, 0.2378] | 0.6530 [0.6316, 0.6679] | 0.605 [0.564, 0.654] |
| TrueSkill, tuned | beta 0.75 x 25/6, tau 2 x 25/300, draw probability 0, tuned on the tuning period | 0.2328 [0.2139, 0.2454] | 0.6617 [0.6211, 0.6912] | 0.621 [0.580, 0.678] |
| TrueSkill, default | beta 1 x 25/6, tau 1 x 25/300, draw probability 0, untuned | 0.2311 [0.2146, 0.2427] | 0.6552 [0.6205, 0.6814] | 0.614 [0.569, 0.678] |
| Higher-rated team wins | hard call from the tuned engine's expected share | 0.3719 [0.3127, 0.4214] | not defined | 0.628 [0.579, 0.687] |
| Coin flip | 0.5 for every match | 0.2500 (constant, no interval) | 0.6931 (constant, no interval) | 0.500 (constant, no interval) |

Log loss is not defined for Higher-rated team wins: hard 0 or 1 calls make it infinite on any miss.

Coin flip: constant, no interval. The metrics are the same on any set of matches (Brier 0.25, log loss ln 2, accuracy 0.5), so there is nothing to resample. The paired difference below still has an interval.

Paired Brier difference against the tuned engine, on the common set: the row's Brier minus the engine's, paired by match (positive favours the engine), for every row except the tuned engine itself. 95% intervals resample league nights (2000 resamples, seed 20261114). For information only; no success criterion depends on them (criterion 1 is decided on the paired set).

| Model | Brier difference, row minus engine | 95% interval |
|---|---|---|
| Engine, default config | +0.0029 | [-0.0062, +0.0138] |
| Elo, tuned | +0.0175 | [+0.0095, +0.0259] |
| Elo, default | +0.0162 | [+0.0071, +0.0258] |
| Glicko-2, tuned | +0.0070 | [-0.0002, +0.0130] |
| Glicko-2, default | +0.0082 | [+0.0010, +0.0131] |
| TrueSkill, tuned | +0.0104 | [-0.0075, +0.0203] |
| TrueSkill, default | +0.0087 | [-0.0068, +0.0173] |
| Higher-rated team wins | +0.1496 | [+0.0955, +0.1948] |
| Coin flip | +0.0276 | [+0.0188, +0.0373] |

By match type (tuned engine, same scoring rules; Elo on the matches it also scores):

| Type | Matches | Engine Brier [95%] | Engine accuracy | Tuned Elo Brier |
| --- | --- | --- | --- | --- |
| league | 222 | 0.2233 [0.2145, 0.2320] | 0.626 | 0.2412 |
| tournament | 63 | 0.2190 [0.1960, 0.2453] | 0.635 | 0.2353 |

Earlier plans' bars, for reference only (not decisions here):

| Bar | Threshold | Value | Met |
| --- | --- | --- | --- |
| Engine not worse than tuned Elo on holdout Brier | 95% CI of (Elo minus engine) Brier lower bound above -0.005 | 0.0175 [0.0095, 0.0259] (n_paired = 285) | yes |
| Holdout point-share bias, favourite's side | \|bias\| <= 0.015 | -0.0143 [-0.0423, +0.0201] | yes |

## 2. Calibration

Reliability of the tuned engine's win probabilities on the scored test matches (`reliability.png`, with tuned Elo beside it):

| Predicted bin | Mean predicted | Observed win rate [95%] | Matches |
| --- | --- | --- | --- |
| 0.1 to 0.2 | 0.146 | 0.200 [0.000, 0.600] | 5 |
| 0.2 to 0.3 | 0.265 | 0.231 [0.064, 0.407] | 26 |
| 0.3 to 0.4 | 0.365 | 0.311 [0.163, 0.450] | 45 |
| 0.4 to 0.5 | 0.450 | 0.450 [0.313, 0.575] | 80 |
| 0.5 to 0.6 | 0.555 | 0.531 [0.435, 0.656] | 64 |
| 0.6 to 0.7 | 0.638 | 0.644 [0.529, 0.783] | 45 |
| 0.7 to 0.8 | 0.739 | 0.800 [0.600, 0.947] | 15 |
| 0.8 to 0.9 | 0.857 | 1.000 [1.000, 1.000] | 3 |
| 0.9 to 1.0 | 0.935 | 1.000 [1.000, 1.000] | 2 |

## 3. Frozen at event start against live within the event

Every test-part tournament match on a date after the first test-part tournament date that has a live prediction: 219 matches from 4 events. The engine frozen at event start predicts from ratings as of the day before the event; the engine live within the event also learns from the event's earlier matches. Brand-new players are included (no burn-in).

Intervals here resample matches, not dates: with only a few events a date bootstrap is not meaningful. Matches share players, so these intervals are too narrow.

| System | Matches | Accuracy [95%] | AUC | Brier [95%] | Log loss |
| --- | --- | --- | --- | --- | --- |
| Engine, frozen at event start | 219 | 0.568 [0.505, 0.632] | 0.638 | 0.2338 [0.2166, 0.2528] | 0.6585 |
| Engine, live within the event | 219 | 0.564 [0.500, 0.630] | 0.626 | 0.2394 [0.2195, 0.2592] | 0.6711 |

Paired Brier difference, engine live minus engine frozen: +0.0056 [-0.0070, +0.0190] (positive means updating within the event predicted worse).

## 4. Convergence for new players

Prediction quality by the fewest earlier matches among the four players (test part, no burn-in; tuned Elo on the same matches):

| Fewest earlier matches | Matches | Dates | Engine Brier [95%] | Engine accuracy | Tuned Elo Brier |
| --- | --- | --- | --- | --- | --- |
| 0 | 46 | 7 | 0.2602 [0.2238, 0.2944] | 0.424 | 0.2454 |
| 1 to 4 | 183 | 9 | 0.2345 [0.2154, 0.2497] | 0.612 | 0.2390 |
| 5 to 9 | 88 | 10 | 0.2202 [0.1854, 0.2444] | 0.670 | 0.2369 |
| 10 to 19 | 97 | 9 | 0.2221 [0.2069, 0.2342] | 0.619 | 0.2469 |
| 20+ | 100 | 10 | 0.2246 [0.2008, 0.2448] | 0.600 | 0.2356 |

Rating settling, for the 64 players with at least 30 matches. A player's later level is the mean of their ratings over their last 10 matches. Every player starts at 3.5 (`convergence.png`):

| After match | Median distance from later level | Share within 0.25 | Players |
| --- | --- | --- | --- |
| 1 | 0.164 | 0.69 | 64 |
| 3 | 0.187 | 0.70 | 64 |
| 5 | 0.116 | 0.69 | 64 |
| 10 | 0.132 | 0.86 | 64 |
| 15 | 0.109 | 0.88 | 64 |
| 20 | 0.093 | 0.91 | 64 |
| 25 | 0.068 | 0.92 | 64 |
| 30 | 0.059 | 0.95 | 64 |

Matches until a rating stays within 0.25 of its later level: median 6, quartiles 1 to 18; 0 players never stayed within it before their last ten matches.

## 5. Stability over time

Weekly replays (68 weeks). Players with at least 10 matches before the week:

| Measure | Value |
| --- | --- |
| Median weekly rating change, players who played | 0.033 |
| 90th percentile weekly change, players who played | 0.083 |
| Largest weekly change, players who did not play | 0.0000 |
| Rank correlation with the previous week: median, minimum | 0.992, 0.956 |

Accuracy by month, every match past burn-in (5). Months in the tuning part are in-sample for the tuned scale and a (and Elo's K), so only test months are out of sample (`accuracy-over-time.png`):

| Month | Part | Matches | Engine Brier | Engine accuracy | Elo Brier | Elo accuracy |
| --- | --- | --- | --- | --- | --- | --- |
| 2025-08 | tuning | 50 | 0.2040 | 0.640 | 0.2429 | 0.600 |
| 2025-09 | tuning | 82 | 0.2292 | 0.646 | 0.2451 | 0.549 |
| 2025-10 | tuning | 9 | 0.1756 | 0.889 | 0.2441 | 0.778 |
| 2025-11 | tuning | 97 | 0.2279 | 0.588 | 0.2490 | 0.546 |
| 2025-12 | tuning | 24 | 0.2695 | 0.500 | 0.2628 | 0.542 |
| 2026-01 | tuning | 21 | 0.2489 | 0.524 | 0.2546 | 0.524 |
| 2026-02 | tuning | 69 | 0.2296 | 0.609 | 0.2492 | 0.522 |
| 2026-03 | tuning | 64 | 0.2314 | 0.672 | 0.2403 | 0.578 |
| 2026-04 | tuning | 39 | 0.2284 | 0.744 | 0.2332 | 0.615 |
| 2026-05 | tuning | 98 | 0.2185 | 0.663 | 0.2406 | 0.612 |
| 2026-06 | tuning | 45 | 0.2334 | 0.600 | 0.2555 | 0.467 |
| 2026-07 | tuning | 70 | 0.2514 | 0.629 | 0.2552 | 0.543 |
| 2026-08 | tuning | 130 | 0.2429 | 0.600 | 0.2530 | 0.492 |
| 2026-09 | test | 21 | 0.2024 | 0.762 | 0.2306 | 0.619 |
| 2026-10 | test | 155 | 0.2282 | 0.613 | 0.2431 | 0.555 |
| 2026-11 | test | 109 | 0.2179 | 0.624 | 0.2371 | 0.615 |

Rolling-origin splits over the whole dataset are in `rolling-splits.md`.

## 6. Rating distributions

| Distribution | Players | Min | Q1 | Median | Q3 | Max |
| --- | --- | --- | --- | --- | --- | --- |
| engine at tuning end | 81 | 2.73 | 3.32 | 3.53 | 3.69 | 4.04 |
| engine at end | 122 | 2.61 | 3.32 | 3.52 | 3.70 | 4.09 |

See `rating-distribution.png`. Engine ratings are for players with at least 10 matches.

## 7. Limitations

- One club, one region, under 2,000 matches and about 190 players: a small, mostly closed pool. Results may not carry over.
- Not pre-registered. The data include an earlier plan's holdout, seen before.
- The test part is one block of about three months. Rolling splits show how much results move between blocks.
- Order of play within a night is the export's row order, which approximates it.
- Tournament first names were mapped to full names by hand; a wrong mapping merges two players' histories.
- Bootstrap intervals treat dates (or matches) as independent, but players recur, so intervals are likely too narrow.

