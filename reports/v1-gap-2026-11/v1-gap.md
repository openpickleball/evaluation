# What a rating gap means under v1

**Status: model figures for the writeup's method section, not measured win rates.** The command reads no match data. It is not part of any pre-registered test.

## Provenance

- Engine: `openpickleball-engine` 1.0.0 at commit `be2a0ff59065f7273b5dc13e395e5ec25ee01d3c` (tag `v1.0.0`).
- Parameter set `v1`, loaded with `Config.published("v1")`: parameter fingerprint `f9149f3c38a01efc`, side-out scale 1.0, side-out win calibration `a` = 0.476.
- Two singles players are seeded at 3.5 + gap and 3.5. Singles only sets an exact gap: the point share and the win model depend on the gap and the scoring type, not the rating format.

## What a gap means: point share

The stronger team's expected share of all points, from `RatingEngine.predict_point_share` under `v1`. This is what a rating gap means. It is not a win probability.

| Team rating gap | Expected point share |
| --- | --- |
| 0.5 | 76.0% |
| 1.0 | 90.9% |

## As a win probability

The stronger team's chance to win, for side-out scoring to 11, win by 2, no cap (`GameFormat()`). Raw is the engine's `match_win_probability` of the point share, with points and games independent. Calibrated is `RatingEngine.predict_win_probability`: the raw value clipped to [1e-06, 1 - 1e-06], then calibrated with `a` = 0.476. This is the probability the evaluation scores.

| Team rating gap | Best of | Raw win probability | Raw clipped | Calibrated win probability |
| --- | --- | --- | --- | --- |
| 0.5 | 1 | 99.65% | no | 93.64% |
| 0.5 | 3 | 99.996% | no | 99.23% |
| 1.0 | 1 | 99.99997% | yes | 99.86% |
| 1.0 | 3 | 99.99999999997% | yes | 99.86% |

A raw value that is clipped gives the largest calibrated value `v1` can give for side-out scoring, so every gap and best-of at the clip has the same calibrated value. The calibrated figures depend on the game format, the number of games and `a`, as well as the gap.
