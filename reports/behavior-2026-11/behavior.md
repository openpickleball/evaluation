# System behavior on simulated leagues

Status: initial evaluation, not pre-registered. Part of the initial evaluation (see `reports/initial-2026-11/technical-report.md`); nothing was tuned or chosen on these results.

## Provenance

- Seed: 0; replicates per scenario: 20. Every random draw comes from `random.Random(f"{seed}-{scenario}-{replicate}")`, one generator per scenario replicate.
- Engine commit: e62468c508126ed46b5a26a82f175ba17dd78371
- Data version: 28bcab9de3af
- Games: one game to 11, win by 2, rallies won with `expected_point_share(team A skill, team B skill, 2.9)`, team skill the mean of its two players.
- Processing: simulations use live engine processing (`RatingEngine.record`, no `as_of`); the real-data check uses weekly replays.

Configs, all at their defaults (nothing is tuned on the simulations):

- OPEN engine: open_engine.Config() defaults (default rating 3.5, side-out scale 2.9); the games are simulated with side-out scale 2.9
- Elo: K 32 (baselines.DEFAULT_ELO_K), start 1500
- Glicko-2: tau 0.5, initial RD 350, 7-day periods (glicko2.DEFAULT_GLICKO2)
- TrueSkill: beta 4.1667, tau 0.0833, mu0 25 (trueskill.DEFAULT_TRUESKILL)

## System behavior

### Ranking recovery

Pools of 16, 32, 64 players with fixed true skill (normal, mean 3.8, SD 0.5, clipped to 2.5 to 5.5) play 15 weekly nights of 4 rounds, everyone in every round with new random partners, so every player has 4k matches after night k. After each night, each system's ratings of every player are compared with true skill by Spearman and Kendall tau-b. Cells are the median across 20 leagues, with the 10th and 90th percentiles in brackets.

Spearman with true skill, by matches per player:

| Pool | System | 4 matches | 12 matches | 20 matches | 40 matches | 60 matches |
| --- | --- | --- | --- | --- | --- | --- |
| 16 | OPEN engine | 0.53 [0.34, 0.72] | 0.80 [0.52, 0.88] | 0.85 [0.68, 0.92] | 0.89 [0.80, 0.94] | 0.93 [0.85, 0.96] |
| 16 | Elo | 0.53 [0.26, 0.75] | 0.73 [0.46, 0.87] | 0.80 [0.60, 0.87] | 0.86 [0.80, 0.96] | 0.88 [0.86, 0.95] |
| 16 | Glicko-2 | 0.52 [0.32, 0.75] | 0.73 [0.38, 0.85] | 0.81 [0.58, 0.88] | 0.86 [0.77, 0.94] | 0.89 [0.86, 0.95] |
| 16 | TrueSkill | 0.48 [0.29, 0.75] | 0.71 [0.46, 0.87] | 0.82 [0.57, 0.88] | 0.86 [0.78, 0.94] | 0.91 [0.82, 0.95] |
| 32 | OPEN engine | 0.59 [0.40, 0.73] | 0.76 [0.63, 0.83] | 0.82 [0.76, 0.89] | 0.89 [0.83, 0.92] | 0.93 [0.87, 0.95] |
| 32 | Elo | 0.54 [0.40, 0.66] | 0.72 [0.52, 0.83] | 0.75 [0.70, 0.86] | 0.85 [0.79, 0.90] | 0.91 [0.83, 0.94] |
| 32 | Glicko-2 | 0.53 [0.42, 0.66] | 0.73 [0.48, 0.81] | 0.75 [0.66, 0.85] | 0.86 [0.77, 0.91] | 0.90 [0.84, 0.94] |
| 32 | TrueSkill | 0.54 [0.39, 0.65] | 0.74 [0.51, 0.83] | 0.74 [0.68, 0.89] | 0.85 [0.77, 0.91] | 0.91 [0.84, 0.94] |
| 64 | OPEN engine | 0.57 [0.46, 0.64] | 0.77 [0.72, 0.81] | 0.85 [0.80, 0.87] | 0.90 [0.88, 0.92] | 0.94 [0.91, 0.95] |
| 64 | Elo | 0.53 [0.38, 0.59] | 0.72 [0.65, 0.77] | 0.81 [0.76, 0.86] | 0.87 [0.84, 0.91] | 0.91 [0.88, 0.93] |
| 64 | Glicko-2 | 0.51 [0.43, 0.58] | 0.70 [0.65, 0.76] | 0.80 [0.75, 0.84] | 0.87 [0.84, 0.91] | 0.91 [0.89, 0.93] |
| 64 | TrueSkill | 0.53 [0.38, 0.59] | 0.71 [0.66, 0.76] | 0.81 [0.76, 0.86] | 0.88 [0.86, 0.91] | 0.92 [0.89, 0.94] |

Kendall tau-b with true skill, by matches per player:

| Pool | System | 4 matches | 12 matches | 20 matches | 40 matches | 60 matches |
| --- | --- | --- | --- | --- | --- | --- |
| 16 | OPEN engine | 0.38 [0.21, 0.57] | 0.63 [0.35, 0.69] | 0.66 [0.47, 0.80] | 0.77 [0.61, 0.85] | 0.81 [0.70, 0.88] |
| 16 | Elo | 0.35 [0.19, 0.60] | 0.54 [0.31, 0.73] | 0.63 [0.45, 0.72] | 0.70 [0.58, 0.87] | 0.74 [0.68, 0.84] |
| 16 | Glicko-2 | 0.40 [0.23, 0.61] | 0.56 [0.28, 0.73] | 0.63 [0.43, 0.75] | 0.71 [0.63, 0.81] | 0.75 [0.68, 0.84] |
| 16 | TrueSkill | 0.32 [0.21, 0.57] | 0.53 [0.30, 0.72] | 0.65 [0.43, 0.73] | 0.72 [0.60, 0.81] | 0.78 [0.66, 0.86] |
| 32 | OPEN engine | 0.43 [0.26, 0.58] | 0.56 [0.46, 0.66] | 0.64 [0.56, 0.72] | 0.73 [0.65, 0.77] | 0.78 [0.71, 0.83] |
| 32 | Elo | 0.38 [0.26, 0.48] | 0.53 [0.37, 0.65] | 0.58 [0.52, 0.68] | 0.69 [0.61, 0.75] | 0.75 [0.65, 0.81] |
| 32 | Glicko-2 | 0.41 [0.32, 0.52] | 0.54 [0.34, 0.62] | 0.57 [0.51, 0.67] | 0.68 [0.60, 0.75] | 0.75 [0.67, 0.81] |
| 32 | TrueSkill | 0.40 [0.25, 0.47] | 0.53 [0.37, 0.65] | 0.56 [0.50, 0.71] | 0.68 [0.58, 0.75] | 0.76 [0.67, 0.80] |
| 64 | OPEN engine | 0.40 [0.32, 0.46] | 0.58 [0.53, 0.61] | 0.66 [0.61, 0.69] | 0.73 [0.70, 0.76] | 0.80 [0.76, 0.82] |
| 64 | Elo | 0.36 [0.26, 0.41] | 0.53 [0.47, 0.58] | 0.62 [0.56, 0.68] | 0.70 [0.66, 0.75] | 0.75 [0.70, 0.79] |
| 64 | Glicko-2 | 0.39 [0.32, 0.45] | 0.51 [0.48, 0.56] | 0.61 [0.57, 0.66] | 0.70 [0.67, 0.74] | 0.75 [0.71, 0.79] |
| 64 | TrueSkill | 0.36 [0.26, 0.41] | 0.52 [0.48, 0.57] | 0.61 [0.58, 0.67] | 0.71 [0.68, 0.75] | 0.76 [0.72, 0.80] |

Matches per player at which the median Spearman first reaches each bar:

| Pool | System | Spearman 0.8 | Spearman 0.9 |
| --- | --- | --- | --- |
| 16 | OPEN engine | 16 | 36 |
| 16 | Elo | 16 | 48 |
| 16 | Glicko-2 | 20 | 52 |
| 16 | TrueSkill | 16 | 48 |
| 32 | OPEN engine | 20 | 48 |
| 32 | Elo | 28 | 52 |
| 32 | Glicko-2 | 28 | 56 |
| 32 | TrueSkill | 32 | 60 |
| 64 | OPEN engine | 16 | 40 |
| 64 | Elo | 20 | 48 |
| 64 | Glicko-2 | 20 | 48 |
| 64 | TrueSkill | 20 | 48 |

Ratings read: the engine's doubles rating, the Elo rating, the Glicko-2 display rating with the current period's games applied, and TrueSkill mu. Every checkpoint is in `ranking-recovery.csv`; the median Spearman curves are in `ranking-recovery.png`.

### Convergence

An established pool of 33 players plays 20 nights of 4 rounds on 8 courts (one random player sits out each round). Then three newcomers join with true skill 3 (low), 3.8 (middle), 4.6 (high) and all 36 players play 25 nights with everyone in every round, so each newcomer plays 100 matches. A newcomer's settled value is the mean of their engine rating over matches 81 to 100; matches to within t is the smallest n with every rating from match n through match 80 within t of it. Medians across 20 replicates, 10th and 90th percentiles in brackets; replicates that never stay within t are counted, not included in the median.

| Newcomer | True skill | Settled minus true | Matches to within 0.25 | Never within 0.25 | Matches to within 0.1 | Never within 0.1 |
| --- | --- | --- | --- | --- | --- | --- |
| low | 3 | -0.147 [-0.276, +0.011] | 21.0 [10.0, 43.9] | 0 of 20 | 61.5 [27.3, 75.1] | 0 of 20 |
| middle | 3.8 | -0.265 [-0.387, -0.129] | 1.0 [1.0, 1.0] | 0 of 20 | 38.5 [13.7, 56.8] | 0 of 20 |
| high | 4.6 | -0.436 [-0.559, -0.346] | 28.0 [9.8, 54.1] | 0 of 20 | 58.0 [45.6, 75.2] | 1 of 20 |

Median engine reliability (0 to 100) after match n:

| Newcomer | Match 1 | Match 5 | Match 10 | Match 20 | Match 40 | Match 80 |
| --- | --- | --- | --- | --- | --- | --- |
| low | 5.8 | 35.5 | 59.9 | 88.2 | 100.0 | 100.0 |
| middle | 5.8 | 35.5 | 59.9 | 88.2 | 100.0 | 100.0 |
| high | 5.8 | 35.5 | 59.9 | 88.2 | 100.0 | 100.0 |

Median rating change from match 80 to match 100 (from the last judged match to the end of the 81 to 100 reference window, so a nonzero change means the rating is still moving while the settled value is taken): low -0.064 [-0.110, +0.009]; middle +0.012 [-0.084, +0.089]; high +0.077 [+0.019, +0.128].

Median reliability at match 100 is 100.0 (low), 100.0 (middle), 100.0 (high); the median distance from true skill at match 100 (rating minus true skill) is -0.164 (low), -0.267 (middle), -0.405 (high). Reliability is at least 90 while the median rating is still 0.25 or more from true skill for: middle, high; high reliability there does not mean the rating has reached true skill.

For middle, matches to within 0.25 is 1 because its median settled value (3.535) is within 0.25 of the starting rating 3.5: this reflects where it started, not fast convergence.

Every player starts at `Config().default_rating` (3.5) while the pool's true mean is 3.8, so the established pool's ratings sit below true skill as a whole (see Drift); read settled minus true against that pool-wide gap. Reliability is read after each of the newcomer's matches, with `as_of` the match date. Per level and match number, `convergence.csv` holds the median rating, its 10th and 90th percentiles and the median reliability; `convergence.png` plots them against matches played.

### Drift

40 active players play 104 weekly nights of 3 rounds on 10 courts. After each night every active player leaves with probability 0.02 and is replaced at once by a new player with a fresh true skill from the same distribution, so the pool stays at 40 and its true distribution is fixed. The gap is the mean engine rating minus the mean true skill over the active players after each night (before that night's leavers are replaced); inflation is the least-squares slope of the gap against week, per year. Medians across 20 replicates, 10th and 90th percentiles in brackets.

| Measure | Median [10th, 90th] |
| --- | --- |
| Gap trend (rating points per year) | +0.024 [-0.041, +0.055] |
| Gap at week 13 | -0.309 [-0.393, -0.244] |
| Gap at week 52 | -0.290 [-0.363, -0.204] |
| Gap at week 104 | -0.285 [-0.362, -0.236] |
| Mean rating of players with 10+ matches at week 104 | 3.488 [3.431, 3.562] |
| Gap of players with 10+ matches at week 104 (over their own true skill) | -0.282 [-0.372, -0.247] |
| Mean true skill of the active pool at week 104 | 3.794 [3.724, 3.874] |
| Rating SD at week 104 | 0.448 [0.362, 0.493] |
| True-skill SD at week 104 | 0.487 [0.429, 0.553] |

Direction: **inflation** (inflation above +0.01 per year, deflation below -0.01, judged on the median slope). The 10th to 90th percentile range of the slope spans zero, so the direction is not consistent across replicates. Newcomers start at `Config().default_rating` (3.5) while the true mean is 3.8, so every replacement enters below the pool's true mean until their rating moves. Per week, `drift.csv` holds the median and percentiles of the mean rating, the rating SD, the gap, and the mean rating and gap over players with 10+ matches; `drift.png` plots the gap and the SD.

### Pool isolation

Pool A (24 players, true skill mean 3.5) and pool B (24, mean 4.1), both SD 0.4, play 30 nights of 3 rounds on their own 6 courts with no cross-pool match (phase 1). Phase 2 is 30 more nights with c cross-pool matches a night (two random A players against two random B players, the k-th in round k mod 3); each pool's other players fill as many in-pool courts as they can, and the two left over in each pool when a round has an odd number of cross matches sit out. Every rate replays the same phase-1 matches. The offset is (mean engine rating of B minus A) minus (mean true skill of B minus A), so 0 is the true gap. Medians across 20 replicates, 10th and 90th percentiles in brackets.

End of phase 1: offset -0.595 [-0.706, -0.446], Spearman with true skill over all 48 players 0.718 [0.648, 0.771]. The true B minus A gap is 0.595 [0.446, 0.706].

| Cross matches per night | Offset after phase 2 | Phase-2 nights until within 0.1 (median) | Cross matches until within 0.1 (median) | Never within | Spearman after phase 2 |
| --- | --- | --- | --- | --- | --- |
| 0 | -0.595 [-0.706, -0.446] | not within 30 nights | n/a (no cross matches) | 20 of 20 | 0.726 [0.644, 0.798] |
| 1 | -0.531 [-0.623, -0.375] | not within 30 nights | not within 30 nights | 20 of 20 | 0.795 [0.702, 0.853] |
| 2 | -0.472 [-0.574, -0.365] | not within 30 nights | not within 30 nights | 20 of 20 | 0.820 [0.765, 0.861] |
| 4 | -0.371 [-0.439, -0.259] | not within 30 nights | not within 30 nights | 20 of 20 | 0.878 [0.825, 0.901] |
| 8 | -0.222 [-0.260, -0.149] | not within 30 nights | not within 30 nights | 20 of 20 | 0.938 [0.921, 0.951] |

Nights until within is the first phase-2 night from which the offset stays within 0.1 for the rest of phase 2 (0 when it already does at the end of phase 1); cross matches until within is the cumulative cross-pool matches by that night, not applicable at a rate of 0. Medians are over the replicates that got within; the others are counted under never within. Per rate and night, `isolation.csv` holds the median offset and its percentiles; `isolation.png` plots it.

### On real data (exploratory)

Exploratory, and not a test: real players' true skill is unknown and can change, so a moving mean is not proof of drift. Every league and tournament match is replayed weekly (as section 5 of the initial evaluation's technical report does) with the engine tuned strictly on matches on or before 2026-08-23. Published per week: the number of rated players, the number with 10+ matches, and the mean and SD of that 10+ group's ratings. The all-player mean rating is not published: idle players keep their rating between weekly replays, so comparing it across weeks would give individual ratings. The 10+ mean and SD are withheld for a week when the group has fewer than 5 players, or when, compared with any earlier published week, only 1 to 4 players joined the group, left it or changed rating, since the difference between the two weeks would then describe those few players. 65 of 68 weeks have the 10+ mean published.

| Measure | Value |
| --- | --- |
| Weekly replays | 68 |
| Weeks | 2025-08-10 to 2026-11-22 |
| Players with 10+ matches, last week | 122 |
| Rated players, last week | 189 |
| Mean rating (10+ matches), first published week with at least 10 such players | 3.485 (2025-08-31) |
| Mean rating (10+ matches), last published week | 3.500 (2026-11-22) |
| Change in the 10+ mean | +0.014 |
| Range of the published weekly 10+ mean | 3.485 to 3.509 |

Per-week aggregates only are in `real-drift.csv`; `real-drift.png` plots the published 10+ mean with the tuning end marked.

## Doubles and gaming

Part of the initial evaluation and not pre-registered: doubles-specific and gaming scenarios on the same kind of simulated leagues, plus one exploratory check on the real data. Every scenario runs under two configs, side by side:

- `v1`: the published parameter set (`reports/v1-fit-2026-11/v1-fit.md`), built as `Config()` with its four changed fields and checked by fingerprint; fingerprint 22b53a37587d6881, side-out scale 1, k_min 0.107914, k_max 0.539568, side-out win calibration 0.476.
- `Config()`: the engine defaults; fingerprint 17f2981186edf96c, side-out scale 2.9, k_min 0.12, k_max 0.6, side-out win calibration 1.

Units. True skill is on the `Config()` unit: each rally goes to team A with `expected_point_share(team A skill, team B skill, 2.9)`. Under `v1` the side-out scale is 1.0, so a `v1` rating gap is 2.9 times smaller for the same expected share. Every rating quantity below is converted to truth units (a `v1` difference times 2.9); divide by 2.9 for `v1` units. `v1`'s K is 2.6 times `Config()`'s in truth units, so it moves faster and noisier. A player's error is relative: (their rating minus the pool's mean rating, in truth units) minus (their true skill minus the pool's mean true skill), with the pool every simulated player in that league. Positive means rated too high.

Truth models. **Mean**: a team's true skill is the mean of its players' (as in the system-behavior sections above and as the engine assumes). **Weaker-weighted**: it is (2 * weaker + stronger) / 3, so the weaker player counts twice; it shows what the engine's mean-rating assumption gets wrong if real teams play that way. Both are our choices, not measured.

Every number is from seed 0, 20 replicates: the median across replicates with the 10th and 90th percentiles in brackets unless said otherwise. Each scenario draws from `random.Random(f"{seed}-{scenario}-{replicate}")`; a replicate's matches never depend on ratings, so the same matches go to both configs. The variants of a scenario (carry fractions, truth models, sandbagging modes, farming and control) share the replicate's skills and start their schedule and game generators from the same state, but their matches diverge once the variants draw differently. Nothing was tuned on these results.

### Carrying

A league of 24: 22 background players (true skill normal, mean 3.8, SD 0.5, clipped to 2.5 to 5.5) plus W (true 3) and S (true 4.6). 30 weekly nights of 3 rounds on 6 courts, everyone every round (90 matches each). In each round, with probability f (one draw per round), W and S are partners against two random background players; otherwise the round is a random shuffle. The carrying effect is a player's error minus their error at f = 0 in the same replicate. Errors and W's reliability are at the end (seed 0, 20 replicates).

| Config | Truth model | f | W error | S error | W carrying effect | S carrying effect | W reliability |
| --- | --- | --- | --- | --- | --- | --- | --- |
| v1 | mean | 0 | -0.081 [-0.285, +0.038] | +0.069 [-0.196, +0.266] | +0.000 [+0.000, +0.000] | +0.000 [+0.000, +0.000] | 100.0 [100.0, 100.0] |
| v1 | mean | 0.5 | +0.024 [-0.150, +0.333] | -0.010 [-0.356, +0.288] | +0.173 [-0.134, +0.583] | -0.136 [-0.358, +0.229] | 100.0 [100.0, 100.0] |
| v1 | mean | 1 | +0.802 [+0.664, +0.898] | -0.798 [-0.936, -0.702] | +0.878 [+0.685, +1.146] | -0.881 [-1.149, -0.598] | 100.0 [100.0, 100.0] |
| v1 | weaker-weighted | 0 | -0.234 [-0.401, -0.077] | -0.046 [-0.161, +0.141] | +0.000 [+0.000, +0.000] | +0.000 [+0.000, +0.000] | 100.0 [100.0, 100.0] |
| v1 | weaker-weighted | 0.5 | -0.148 [-0.343, +0.064] | -0.168 [-0.451, +0.035] | +0.041 [-0.117, +0.274] | -0.188 [-0.395, +0.067] | 100.0 [100.0, 100.0] |
| v1 | weaker-weighted | 1 | +0.612 [+0.450, +0.729] | -0.988 [-1.150, -0.871] | +0.891 [+0.555, +1.037] | -0.984 [-1.196, -0.704] | 100.0 [100.0, 100.0] |
| Config() | mean | 0 | +0.078 [-0.026, +0.197] | -0.130 [-0.297, -0.017] | +0.000 [+0.000, +0.000] | +0.000 [+0.000, +0.000] | 100.0 [100.0, 100.0] |
| Config() | mean | 0.5 | +0.344 [+0.144, +0.529] | -0.340 [-0.496, -0.175] | +0.211 [+0.084, +0.471] | -0.213 [-0.363, -0.014] | 100.0 [100.0, 100.0] |
| Config() | mean | 1 | +0.792 [+0.715, +0.898] | -0.808 [-0.885, -0.702] | +0.680 [+0.608, +0.851] | -0.670 [-0.856, -0.491] | 100.0 [100.0, 100.0] |
| Config() | weaker-weighted | 0 | -0.012 [-0.115, +0.131] | -0.224 [-0.315, -0.038] | +0.000 [+0.000, +0.000] | +0.000 [+0.000, +0.000] | 100.0 [100.0, 100.0] |
| Config() | weaker-weighted | 0.5 | +0.202 [+0.048, +0.339] | -0.507 [-0.593, -0.314] | +0.221 [-0.018, +0.298] | -0.261 [-0.500, -0.053] | 100.0 [100.0, 100.0] |
| Config() | weaker-weighted | 1 | +0.621 [+0.559, +0.690] | -0.979 [-1.041, -0.910] | +0.645 [+0.422, +0.807] | -0.739 [-0.998, -0.561] | 100.0 [100.0, 100.0] |

At f = 1 W and S always share a team, start at the same rating with the same RD and so receive identical updates: the engine gives them the same rating, the mean of their true skills in effect, and their errors differ by exactly the true gap of 1.6.

Fixed equal pair: P and Q, both true 3.8, are always partners against two random background players, with the same 22 background players (seed 0, 20 replicates). Under the mean truth model P + Q and W + S both have team skill 3.8, and the pair run uses the same schedule and game seeds as f = 1, so its matches are those of the f = 1 carrying run: P's error is W's error minus 0.8, not independent evidence. The weaker-weighted row, where the two teams' true skills differ, is the informative one.

| Config | Truth model | P error | Q error | Pair mean error |
| --- | --- | --- | --- | --- |
| v1 | mean | +0.002 [-0.136, +0.098] | +0.002 [-0.136, +0.098] | +0.002 [-0.136, +0.098] |
| v1 | weaker-weighted | +0.073 [-0.007, +0.281] | +0.073 [-0.007, +0.281] | +0.073 [-0.007, +0.281] |
| Config() | mean | -0.008 [-0.085, +0.098] | -0.008 [-0.085, +0.098] | -0.008 [-0.085, +0.098] |
| Config() | weaker-weighted | +0.082 [+0.022, +0.177] | +0.082 [+0.022, +0.177] | +0.082 [+0.022, +0.177] |

Per config, truth model, f and night, `carrying.csv` holds the median and percentiles of W's and S's error; `carrying.png` plots the medians under the mean truth model.

### Partner-gap filter

`max_doubles_partner_gap` set to 1 in truth units (0.345 in `v1` units, 1 in `Config()` units), rerun on the carrying league at f = 1 under the mean truth model. The engine rejects a match (`RejectionReason.DOUBLES_PARTNER_GAP`) when either team's partners' ratings before it differ by more than the cap (seed 0, 20 replicates).

| Config | Matches rejected | W error with the filter | W error without |
| --- | --- | --- | --- |
| v1 | 29.4% [18.0%, 36.1%] | +0.792 [+0.705, +0.944] | +0.802 [+0.664, +0.898] |
| Config() | 6.3% [3.6%, 12.7%] | +0.796 [+0.728, +0.906] | +0.792 [+0.715, +0.898] |

The filter reads ratings, not true skill. W and S always hold the same rating, so their partnership is never rejected; only background pairs whose ratings have spread apart are. The filter's effect on the lopsided league is in the next subsection.

### Lopsided against balanced teams

Analytic check, no simulation: 4.5 + 2.5 against 3.5 + 3.5. Given correct ratings the engine predicts the lopsided team 0.500 of the points. Its true share is 0.500 under the mean truth model and 0.434 under the weaker-weighted one (`expected_point_share(19/6, 3.5, 2.9)`).

Simulation: a league of 32 (true skill normal, mean 3.8, SD 0.6, clipped), 30 nights of 3 rounds on 8 courts, new random partners every round. For every match after night 10, team L is the team with the larger true partner gap (team A on a tie). The residual is L's actual point share minus the engine's expected share from the live ratings just before the match. Matches are binned by L's true partner gap minus the other team's, in truth units. Matches is the count summed over replicates; the residual is the median across replicates of each replicate's mean residual in the bin. With the filter, `max_doubles_partner_gap` is 1 truth unit and only matches the engine counts enter (seed 0, 20 replicates).

| Config | Truth model | Filter | Gap difference | Matches | Residual for L |
| --- | --- | --- | --- | --- | --- |
| v1 | mean | off | 0 to 0.5 | 5113 | -0.0024 [-0.0114, +0.0073] |
| v1 | mean | off | 0.5 to 1 | 2838 | +0.0022 [-0.0075, +0.0103] |
| v1 | mean | off | 1 and above | 1649 | +0.0081 [-0.0207, +0.0257] |
| v1 | mean | on | 0 to 0.5 | 3666 | -0.0025 [-0.0104, +0.0083] |
| v1 | mean | on | 0.5 to 1 | 1214 | +0.0070 [-0.0113, +0.0227] |
| v1 | mean | on | 1 and above | 160 | +0.0113 [-0.0440, +0.0649] |
| v1 | weaker-weighted | off | 0 to 0.5 | 5113 | -0.0041 [-0.0122, +0.0033] |
| v1 | weaker-weighted | off | 0.5 to 1 | 2838 | -0.0179 [-0.0305, -0.0028] |
| v1 | weaker-weighted | off | 1 and above | 1649 | -0.0268 [-0.0429, +0.0046] |
| v1 | weaker-weighted | on | 0 to 0.5 | 3618 | -0.0041 [-0.0145, +0.0040] |
| v1 | weaker-weighted | on | 0.5 to 1 | 1242 | -0.0163 [-0.0273, +0.0170] |
| v1 | weaker-weighted | on | 1 and above | 174 | -0.0442 [-0.1655, +0.0391] |
| Config() | mean | off | 0 to 0.5 | 5113 | -0.0044 [-0.0118, +0.0075] |
| Config() | mean | off | 0.5 to 1 | 2838 | -0.0025 [-0.0121, +0.0121] |
| Config() | mean | off | 1 and above | 1649 | +0.0016 [-0.0207, +0.0282] |
| Config() | mean | on | 0 to 0.5 | 4686 | -0.0036 [-0.0131, +0.0042] |
| Config() | mean | on | 0.5 to 1 | 2169 | +0.0023 [-0.0141, +0.0177] |
| Config() | mean | on | 1 and above | 598 | -0.0025 [-0.0247, +0.0331] |
| Config() | weaker-weighted | off | 0 to 0.5 | 5113 | -0.0062 [-0.0162, +0.0038] |
| Config() | weaker-weighted | off | 0.5 to 1 | 2838 | -0.0210 [-0.0395, -0.0048] |
| Config() | weaker-weighted | off | 1 and above | 1649 | -0.0353 [-0.0644, -0.0024] |
| Config() | weaker-weighted | on | 0 to 0.5 | 4636 | -0.0058 [-0.0132, +0.0065] |
| Config() | weaker-weighted | on | 0.5 to 1 | 2140 | -0.0181 [-0.0407, +0.0015] |
| Config() | weaker-weighted | on | 1 and above | 594 | -0.0390 [-0.0811, +0.0100] |

Share of all matches the filter rejects in this league (seed 0, 20 replicates):

| Config | Truth model | Matches rejected |
| --- | --- | --- |
| v1 | mean | 44.0% [33.2%, 53.1%] |
| v1 | weaker-weighted | 43.1% [35.4%, 56.2%] |
| Config() | mean | 15.0% [9.0%, 23.8%] |
| Config() | weaker-weighted | 16.8% [8.4%, 24.1%] |

Per config, truth model, filter and bin, `lopsided.csv` holds the same numbers; `lopsided.png` plots the median residual without the filter.

### Lopsided teams on real data (exploratory)

Exploratory and in-sample: `v1` was fitted on these same matches, and real players' skill is unknown, so the partner gaps here are rating gaps, not true ones. Every match is processed live in ledger order (`open_engine.ledger_order`) under each config, with no tuning. A match enters when the engine counts it and all four players have at least 5 earlier counted matches. Team L has the larger partner rating gap before the match; the residual is L's actual point share minus the engine's expected share. Bins are of L's gap minus the other team's, in truth units (the config-unit edges are given too). The interval is the two-way bootstrap (`bootstrap.two_way_bootstrap`, match dates and the four players, 2,000 resamples, seed 20261114). A bin with fewer than 30 matches shows no mean. Doubles matches in the ledger: 1601. Data version: 28bcab9de3af.

| Config | Gap difference (truth units) | Same in config units | Matches | Mean residual for L | Two-way 95% interval |
| --- | --- | --- | --- | --- | --- |
| v1 | 0 to 0.5 | 0 to 0.172 | 730 | -0.0016 | [-0.0336, +0.0291] |
| v1 | 0.5 to 1 | 0.172 to 0.345 | 258 | -0.0065 | [-0.0581, +0.0462] |
| v1 | 1 and above | 0.345 and above | 95 | -0.0306 | [-0.1228, +0.0654] |
| Config() | 0 to 0.5 | 0 to 0.5 | 936 | +0.0007 | [-0.0282, +0.0332] |
| Config() | 0.5 to 1 | 0.5 to 1 | 139 | -0.0357 | [-0.1276, +0.0488] |
| Config() | 1 and above | 1 and above | 8 | too few matches | too few matches |

Only these bin aggregates are written, also to `real-lopsided.csv`.

### Sandbagging

A league of 24: 23 background players plus X (true 4.2), everyone every round on 6 courts. X plays 60 honest matches, then tanks 15 matches, then plays 60 honest matches. **Soft**: while tanking X plays at true skill minus 1. **Hard**: X's team loses every tanked game 0-11. Mean truth model. Before is X's error after the honest matches; the drop is before minus the lowest error reached; recovery is the number of matches after the tank until X's error is back within 0.25 (and 0.1) of before and stays there to the end, over the replicates that recover (seed 0, 20 replicates).

| Config | Mode | Error before | Lowest error | Drop | Drop per tanked match | Matches to within 0.25 | Matches to within 0.1 | Error at end |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| v1 | soft | +0.016 [-0.163, +0.167] | -0.455 [-0.580, -0.345] | 0.461 [0.324, 0.609] | 0.0307 [0.0216, 0.0406] | 26.0 [5.0, 51.5]; not within 60 in 4 of 20 | 58.5 [51.5, 60.0]; not within 60 in 12 of 20 | +0.057 [-0.228, +0.162] |
| v1 | hard | +0.016 [-0.163, +0.167] | -1.845 [-2.062, -1.693] | 1.872 [1.759, 2.017] | 0.1248 [0.1173, 0.1345] | 53.5 [44.2, 59.0]; not within 60 in 12 of 20 | 60.0 [56.0, 60.0]; not within 60 in 17 of 20 | -0.348 [-0.599, -0.105] |
| Config() | soft | -0.150 [-0.225, +0.033] | -0.308 [-0.417, -0.197] | 0.191 [0.121, 0.246] | 0.0127 [0.0081, 0.0164] | 1.0 [1.0, 1.0]; not within 60 in 1 of 20 | 38.0 [20.0, 59.5]; not within 60 in 8 of 20 | -0.094 [-0.223, -0.007] |
| Config() | hard | -0.150 [-0.225, +0.033] | -0.925 [-1.046, -0.783] | 0.800 [0.766, 0.839] | 0.0533 [0.0511, 0.0560] | 53.0 [46.9, 58.4]; not within 60 in 16 of 20 | 60.0 [60.0, 60.0]; not within 60 in 19 of 20 | -0.437 [-0.599, -0.387] |

The engine does not expose K; it follows from RD (engine `docs/model.md`, "K"). X's RD (the stored value, config units) and reliability (seed 0, 20 replicates):

| Config | Mode | RD before | RD after the tank | RD at end | Reliability before | Reliability at end |
| --- | --- | --- | --- | --- | --- | --- |
| v1 | soft | 0.1000 [0.1000, 0.1000] | 0.1000 [0.1000, 0.1000] | 0.1000 [0.1000, 0.1000] | 100.0 [100.0, 100.0] | 100.0 [100.0, 100.0] |
| v1 | hard | 0.1000 [0.1000, 0.1000] | 0.1000 [0.1000, 0.1000] | 0.1000 [0.1000, 0.1000] | 100.0 [100.0, 100.0] | 100.0 [100.0, 100.0] |
| Config() | soft | 0.1000 [0.1000, 0.1000] | 0.1000 [0.1000, 0.1000] | 0.1000 [0.1000, 0.1000] | 100.0 [100.0, 100.0] | 100.0 [100.0, 100.0] |
| Config() | hard | 0.1000 [0.1000, 0.1000] | 0.1000 [0.1000, 0.1000] | 0.1000 [0.1000, 0.1000] | 100.0 [100.0, 100.0] | 100.0 [100.0, 100.0] |

Per config, mode and X's match number, `sandbagging.csv` holds the median and percentiles of X's error; `sandbagging.png` plots the medians with the tank shaded.

### Farming

A league of 32 background players (true skill normal, mean 3.8, SD 0.5, clipped) plus F (true 4.6) on 8 courts: F plays every round and one other player sits out at random. 20 established nights of 3 rounds, then 20 farming nights in which F's partner and both opponents are drawn at random from the 11 background players of lowest true skill (the rest fill the other courts at random). The control replays the same league with F placed at random throughout. The farming gain is F's error at the end of farming minus the control's, per replicate (seed 0, 20 replicates).

| Config | Truth model | F error, farming | F error, control | Farming gain |
| --- | --- | --- | --- | --- |
| v1 | mean | +0.065 [-0.031, +0.305] | +0.167 [-0.083, +0.292] | -0.037 [-0.299, +0.320] |
| v1 | weaker-weighted | -0.175 [-0.330, +0.078] | -0.028 [-0.282, +0.101] | -0.139 [-0.360, +0.182] |
| Config() | mean | -0.036 [-0.129, +0.112] | -0.017 [-0.186, +0.054] | +0.036 [-0.095, +0.237] |
| Config() | weaker-weighted | -0.190 [-0.300, -0.070] | -0.161 [-0.298, -0.075] | -0.045 [-0.152, +0.145] |

Per config, truth model, arm and night, `farming.csv` holds the median and percentiles of F's error.

### Fixed-pair padding and closed groups

Collusion in an established league: 24 background players play 20 nights of 3 rounds, everyone every round. Then 4 of them chosen at random record N fabricated matches among themselves, one a round from the next night on: the target T partners each of the other three in turn and T's team wins every game 11-0. The other 20 keep playing honest rounds meanwhile. Each fabricated match is recorded as a league match, or as a self-reported one confirmed by a player from each team. T's gain and the others' change are changes in relative error from just before the collusion to its end; the group sum change is the change of the four ratings' sum (truth units). Counted is the mean share of fabricated matches the engine counted. Variety is T's unique partners plus unique opponents in the diversity window (seed 0, 20 replicates).

| Config | Recorded as | N | Counted | T's gain | Others' mean change | Group sum change | T reliability before | T reliability after | T variety before | T variety after |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| v1 | league | 10 | 100.0% | +1.366 [+1.187, +1.523] | -0.455 [-0.508, -0.396] | +0.0000 [+0.0000, +0.0000] | 100.0 [100.0, 100.0] | 100.0 [100.0, 100.0] | 45 [44, 46] | 45 [44, 46] |
| v1 | league | 20 | 100.0% | +2.291 [+1.997, +2.560] | -0.764 [-0.853, -0.666] | +0.0000 [-0.0000, +0.0000] | 100.0 [100.0, 100.0] | 100.0 [100.0, 100.0] | 45 [44, 46] | 45 [44, 46] |
| v1 | league | 40 | 100.0% | +3.493 [+3.086, +3.866] | -1.164 [-1.289, -1.029] | +0.0000 [-0.0000, +0.0000] | 100.0 [100.0, 100.0] | 100.0 [100.0, 100.0] | 45 [44, 46] | 45 [44, 46] |
| v1 | self-reported | 10 | 100.0% | +0.363 [+0.313, +0.408] | -0.121 [-0.136, -0.104] | +0.0000 [+0.0000, +0.0000] | 100.0 [100.0, 100.0] | 99.0 [99.0, 99.0] | 45 [44, 46] | 45 [44, 46] |
| v1 | self-reported | 20 | 60.0% | +0.379 [+0.326, +0.426] | -0.126 [-0.142, -0.109] | +0.0000 [+0.0000, +0.0000] | 100.0 [100.0, 100.0] | 97.2 [97.2, 97.2] | 45 [44, 46] | 45 [44, 46] |
| v1 | self-reported | 40 | 32.5% | +0.390 [+0.336, +0.437] | -0.130 [-0.146, -0.112] | +0.0000 [+0.0000, +0.0000] | 100.0 [100.0, 100.0] | 93.5 [93.5, 93.5] | 45 [44, 46] | 45 [44, 46] |
| Config() | league | 10 | 100.0% | +0.560 [+0.499, +0.599] | -0.187 [-0.200, -0.166] | +0.0000 [-0.0000, +0.0000] | 100.0 [100.0, 100.0] | 100.0 [100.0, 100.0] | 45 [44, 46] | 45 [44, 46] |
| Config() | league | 20 | 100.0% | +1.030 [+0.923, +1.117] | -0.343 [-0.372, -0.308] | +0.0000 [-0.0000, +0.0000] | 100.0 [100.0, 100.0] | 100.0 [100.0, 100.0] | 45 [44, 46] | 45 [44, 46] |
| Config() | league | 40 | 100.0% | +1.820 [+1.623, +1.955] | -0.607 [-0.652, -0.541] | +0.0000 [-0.0000, +0.0000] | 100.0 [100.0, 100.0] | 100.0 [100.0, 100.0] | 45 [44, 46] | 45 [44, 46] |
| Config() | self-reported | 10 | 100.0% | +0.139 [+0.124, +0.150] | -0.046 [-0.050, -0.041] | +0.0000 [+0.0000, +0.0000] | 100.0 [100.0, 100.0] | 99.0 [99.0, 99.0] | 45 [44, 46] | 45 [44, 46] |
| Config() | self-reported | 20 | 60.0% | +0.145 [+0.129, +0.157] | -0.048 [-0.052, -0.043] | +0.0000 [+0.0000, +0.0000] | 100.0 [100.0, 100.0] | 97.2 [97.2, 97.2] | 45 [44, 46] | 45 [44, 46] |
| Config() | self-reported | 40 | 32.5% | +0.150 [+0.133, +0.161] | -0.050 [-0.054, -0.044] | +0.0000 [+0.0000, +0.0000] | 100.0 [100.0, 100.0] | 93.5 [93.5, 93.5] | 45 [44, 46] | 45 [44, 46] |

The variety factor is `min(1, count / diversity_target)` with a target of 12. The 10th percentile of T's count, before and after, is 43.9 or more in every row, so T's factor is 1 throughout: the fabricated matches cost T no reliability through variety.

Closed new group: an established league of 24 (true skill normal, mean 3.8, SD 0.5, clipped) plays 20 nights, then 20 more while a group of new players, all true 3, plays only among itself, one court per 4 members every round, honest play, mean truth model. A group of 4 rotates so each member partners each other member in turn; a group of 8 is shuffled at random each round. The offset error is the engine's offset (mean group rating minus mean league rating, truth units) minus the true offset. Reliability and variety are the members' medians at the end; established is the share of members `player_summary(...).established` labels established, over members and replicates. The variety factor is `min(1, count / diversity_target)` (seed 0, 20 replicates).

| Config | Group size | Offset error | True offset | Median reliability | Established | Variety count | Variety factor |
| --- | --- | --- | --- | --- | --- | --- | --- |
| v1 | 4 | +0.805 [+0.714, +0.879] | -0.805 [-0.879, -0.714] | 80.0 [80.0, 80.0] | 100.0% | 6 [6, 6] | 0.50 [0.50, 0.50] |
| v1 | 8 | +0.805 [+0.714, +0.879] | -0.805 [-0.879, -0.714] | 100.0 [100.0, 100.0] | 100.0% | 14 [14, 14] | 1.00 [1.00, 1.00] |
| Config() | 4 | +0.805 [+0.714, +0.879] | -0.805 [-0.879, -0.714] | 80.0 [80.0, 80.0] | 100.0% | 6 [6, 6] | 0.50 [0.50, 0.50] |
| Config() | 8 | +0.805 [+0.714, +0.879] | -0.805 [-0.879, -0.714] | 100.0 [100.0, 100.0] | 100.0% | 14 [14, 14] | 1.00 [1.00, 1.00] |

`padding.csv` holds every measure of both tables in long form.
