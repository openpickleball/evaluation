# Side-out unit: declaring the V1 scale from the ridge

**Status: part of the initial evaluation, not pre-registered.** It applies a rule fixed before running, on the scale and K ridge that `ridge-check-2026-11` confirmed, and feeds openpickleball/engine#149. It makes no scored comparison and sets no spread target.

## Provenance

- Engine: `openpickleball-engine` 0.3.0.dev0 at commit `e62468c508126ed46b5a26a82f175ba17dd78371`.
- Data version: `28bcab9de3af`.
- Matches: 1601 from 2025-08-10 to 2026-11-20, 1601 doubles, 0 rally. Only the side-out scale is declared.

## The rule

- Declared side-out scale 1.0.
- `k_min` and `k_max` multiplied by 1.0 / s_fit, where s_fit is the strict tune's side-out scale at default K on every match (fit window from 2026-04-12, burn-in 5).
- Bounds kept at 2.0 and 8.0, and every other setting, `a` included, equal to the fitted engine's.

## The declared engine

- s_fit = 1.112 (shown as 1.112).
- Declared k_min = 0.1079136690647482 (default 0.12 / s_fit), declared k_max = 0.539568345323741 (default 0.6 / s_fit).
- Fitted side-out win calibration a = 0.476000, used unchanged by the declared engine.
- For information only, not judged: `a` refitted under the declared config on the same fit window is 0.476000 (difference +0.00e+00).

## Check 1: the declared engine predicts what the fitted engine predicts

Prequential predictions of every doubles match the engine predicts (side-out scoring), no burn-in, from the first match date. Each cell is the largest absolute per-match difference between the two engines over the same match ids. Two NaN values count as equal; NaN against a number is an infinite difference.

| Mode | Matches predicted | No win model | Calibrated win prob | Raw win prob | Expected point share |
| --- | --- | --- | --- | --- | --- |
| scheduled | 1601 | 0 | 2.4e-15 | 4.6e-15 | 1.3e-15 |
| live | 1601 | 0 | 2.4e-15 | 4.6e-15 | 1.3e-15 |

Rule: check 1 passes if each of these six largest differences is at most 1e-09.

Check 1 verdict: **pass**

## Check 2: no match touches a bound

A match touches a bound if a player's doubles rating is within 1e-9 of 2.0 or 8.0 before or after it, on a live replay of every match.

| Engine | Matches at the floor | Matches at the ceiling | Matches at either |
| --- | --- | --- | --- |
| Declared (judged) | 0 | 0 | 0 |
| Fitted (reference) | 0 | 0 | 0 |

Rule: check 2 passes if the declared engine's floor, ceiling and either counts are all 0.

Check 2 verdict: **pass**

## Distribution

Doubles ratings on 2026-11-20 for players with at least 10 matches (inclusive quartiles).

| Engine | Players | Lowest | Q1 | Median | Q3 | Highest | IQR |
| --- | --- | --- | --- | --- | --- | --- | --- |
| Declared | 122 | 2.619 | 3.354 | 3.521 | 3.686 | 4.078 | 0.332 |
| Fitted (reference) | 122 | 2.521 | 3.337 | 3.523 | 3.707 | 4.142 | 0.369 |

Declared IQR times s_fit is 0.369459, against the fitted IQR 0.369459 (equal under the ridge).

## What a rating gap means

Expected point share of the stronger team at a given team-rating gap.

| Engine | Scale | 0.5 gap | 1.0 gap |
| --- | --- | --- | --- |
| Declared | 1.000 | 76.0% | 90.9% |
| Fitted | 1.112 | 73.8% | 88.8% |

