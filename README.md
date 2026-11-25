# OPEN engine evaluation

This repository holds the methodology and results of the first evaluation of the [OPEN reference rating engine](https://github.com/openpickleball/engine) (release 1.0.0 and its published parameter set `v1`) on real pickleball play. Most results ran on a pre-release build of the engine, with a configuration tuned on the earlier matches only; the writeup says which results are for `v1`.

> **Status:** Initial evaluation, not pre-registered. The method was fixed before each result was computed, but it was not locked in advance, so no result here is confirmatory. No OPEN rating is live.

## The short version

- On one club's play (1,601 doubles matches, 189 players, August 2025 to November 2026), the tuned engine's point estimates were ahead of tuned Elo, Glicko-2 and TrueSkill on the main test. Against tuned Glicko-2 and TrueSkill the intervals include zero. Across four rolling test blocks the results swung: in one block tuned Glicko-2 beat the engine, with an interval that excludes zero, and the lead over tuned Elo was close to zero. The comparison favours the engine: only the engine had a fitted win calibration (before it, the engine was behind Glicko-2 and TrueSkill), and only the engine uses point scores, while the baselines see wins and losses.
- The engine predicts worst for brand-new players, who all start at 3.5.
- `v1` scored better than the engine defaults on every point estimate, but not by a measurable margin, so the defaults were kept and `v1` ships as a named parameter set.
- Simulations show that carrying, sandbagging, fabricated league matches and closed groups can each move ratings, and reliability does not flag any of them.

Start with [`writeup.md`](writeup.md). Its "Limitations first" section comes before anything else.

## Layout

| Path | What it holds |
| --- | --- |
| `writeup.md` | The full evaluation: method, data, validation design, results with intervals, limitations, related work |
| `reports/initial-2026-11/` | The initial evaluation: plain-language summary, technical report, rolling splits, figures |
| `reports/v1-fit-2026-11/` | The `v1` fit and the test of whether `v1` should replace the defaults |
| `reports/v1-gap-2026-11/` | What a rating gap means under `v1`, as point share and win probability |
| `reports/declared-scale-2026-11/`, `reports/ridge-check-2026-11/`, `reports/side-out-unit-2026-11/` | Checks on the rating scale and K |
| `reports/behavior-2026-11/` | System behavior, doubles and gaming on simulated leagues, with CSV and figure files |

## What is not here

- **Match data.** The data are private and are not published, even with names removed. Every file here holds aggregates only. Every report was written through a player-name check.
- **Evaluation code.** The reports were produced by code that reads the private data. It is not published here.

## License

[CC BY 4.0](LICENSE).
