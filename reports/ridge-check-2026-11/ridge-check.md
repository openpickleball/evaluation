# Ridge check: did the 2.0 floor break the K and scale ridge?

**Status: a diagnostic in the initial evaluation, not pre-registered.** It checks whether the declared-scale proposal (`declared-scale-2026-11`) lost because of the 2.0 rating floor. It chooses no scale and scores nothing in the test part; it sees only matches on or before the tuning end. Any new candidate for openpickleball/engine#149 needs its own test.

## Provenance

- Engine: `openpickleball-engine` 0.3.0.dev0 at commit `e62468c508126ed46b5a26a82f175ba17dd78371`.
- Data version: `28bcab9de3af`.
- Tuning part: 1087 matches on or before 2026-08-23, 1087 doubles, 0 rally. Only the side-out scale is multiplied by `c`.
- Fit window: scheduled predictions from 2026-02-15 to the tuning end, burn-in 5, the window and `a` search `pipeline.tune` uses.

## Step 1: baseline (the current tuned engine)

- Fitted side-out scale s0 = 0.868 (shown as 0.868), K (k_min 0.12, k_max 0.6), win calibration a 0.402.
- Tuning Brier 0.234582, tuning log loss 0.661881, on 494 fit-window matches.
- IQR at the tuning end 0.341 over 81 players with at least 10 matches (inclusive quartiles).
- Matches touching a bound: floor 0, ceiling 0, either 0. Players at or below 2.2 at the tuning end: 0 (0 with 10+ matches).

Each ridge config is the baseline with side-out scale s, c = s / s0, k_min and k_max times c, and `a` refitted. Differences are the largest absolute difference against the baseline, per fit-window match, over the same set of matches.

## Step 2: ridge without bounds (min_rating -100, max_rating 100)

| Scale | c | k_min | k_max | a | Tuning Brier | Tuning log loss | IQR | IQR / c | Largest win prob difference | Largest raw win prob difference |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| 1.30 | 1.4977 | 0.1797 | 0.8986 | 0.402 | 0.234582 | 0.661881 | 0.511 | 0.341 | 2.2e-15 | 4.7e-15 |
| 1.74 | 2.0046 | 0.2406 | 1.2028 | 0.402 | 0.234582 | 0.661881 | 0.684 | 0.341 | 2.1e-15 | 5.2e-15 |
| 2.20 | 2.5346 | 0.3041 | 1.5207 | 0.402 | 0.234582 | 0.661881 | 0.865 | 0.341 | 2.1e-15 | 5.2e-15 |
| 2.90 | 3.3410 | 0.4009 | 2.0046 | 0.402 | 0.234582 | 0.661881 | 1.140 | 0.341 | 2.0e-15 | 4.3e-15 |

Prediction, written before running: step 2 matches the baseline at every scale, the predictions differ by no more than float rounding, and so do Brier and log loss.

Rule: step 2 holds if, at every scale, the largest win probability difference, the absolute Brier difference and the absolute log loss difference are all at most 1e-09.

Step 2 verdict: **holds**

## Step 3: ridge with the default bounds (2.0 and 8.0)

| Scale | c | k_min | k_max | a | Tuning Brier | Tuning log loss | IQR | IQR / c | Largest win prob difference | Largest raw win prob difference |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| 1.30 | 1.4977 | 0.1797 | 0.8986 | 0.402 | 0.234582 | 0.661881 | 0.511 | 0.341 | 2.2e-15 | 4.7e-15 |
| 1.74 | 2.0046 | 0.2406 | 1.2028 | 0.410 | 0.234489 | 0.661867 | 0.682 | 0.340 | 7.6e-02 | 1.8e-01 |
| 2.20 | 2.5346 | 0.3041 | 1.5207 | 0.418 | 0.235713 | 0.664769 | 0.825 | 0.325 | 1.3e-01 | 2.6e-01 |
| 2.90 | 3.3410 | 0.4009 | 2.0046 | 0.402 | 0.238381 | 0.670338 | 1.010 | 0.302 | 2.3e-01 | 3.3e-01 |

Matches touching a bound (a player's rating within 1e-9 of the floor or ceiling, before or after the match, on a live replay of the tuning part), and players at or below 2.2 at the tuning end:

| Scale | Matches at the floor | Matches at the ceiling | Matches at either | Players at or below 2.2 | Of them with 10+ matches |
| --- | --- | --- | --- | --- | --- |
| 1.30 | 0 | 0 | 0 | 0 | 0 |
| 1.74 | 10 | 0 | 10 | 2 | 2 |
| 2.20 | 57 | 0 | 57 | 3 | 3 |
| 2.90 | 94 | 0 | 94 | 7 | 6 |

Prediction, written before running: step 3 loses Brier as the scale grows, and the loss follows the count of matches at the floor.

Rule: step 3 holds if tuning Brier strictly increases with the scale across the scales, and the floor-touch count is non-decreasing with the scale and greater than zero at 2.9.

Step 3 verdict: **does not hold**

## What this does not decide

This diagnostic chooses no scale and scores nothing in the test part, which the declared-scale test has already scored. A smaller declared scale, a fix for the floor (openpickleball/engine#27) or a different starting rating each needs its own test, with its rule fixed before anything is scored.

