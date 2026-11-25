# The OPEN rating engine on one club's league data: an initial evaluation

**Status: initial evaluation, not pre-registered.** This writeup assembles the evaluation of the OPEN pickleball rating engine into one document. It has not been externally reviewed (see "Review" at the end). Every citation of the OPEN engine is to release 1.0.0 (`openpickleball-engine` 1.0.0, tag `v1.0.0`, commit `be2a0ff`), and each link points at the cited file at that tag. Paths without a repository name are in this repository.

Every number in this writeup comes from a report in `reports/`, named next to it. The few numbers that come from elsewhere (the engine's retune milestones, the count of league exports, 50% as even odds, and the date of the internal check) are named in "Traceability" in section 8.

## Limitations first

- **Small and closed.** One club, six leagues, 1601 doubles matches among 189 players on 52 dates. The pool is mostly closed, so ratings are relative to these players only, and nothing here shows how the engine does elsewhere.
- **Not pre-registered.** Every result on real data below is from the initial evaluation. Each method was fixed before it ran, but nothing was locked, and the data include an earlier plan's holdout, which was seen before. No result here is confirmatory.

- **No measurable gain for `v1` over `Config()`.** The evaluated parameter set `v1` predicted better than the engine defaults on every point estimate, but the two-way 95% interval of the Brier difference, +0.0047 [-0.0044, +0.0139], includes zero. So `v1` ships as a named set, and the defaults were kept. The non-inferiority bar is on Brier only, and it passed by 0.0006 (lower bound -0.0044 against -0.005). The log-loss lower bound is -0.0106, below -0.005, so `v1` is not shown to be no worse on log loss ([PARAMETERS.md, "Default-change rule and result"](https://github.com/openpickleball/engine/blob/v1.0.0/PARAMETERS.md#default-change-rule-and-result)).
- **Gaming.** The simulations found carrying, sandbagging, fabricated league matches and closed groups each move ratings with no warning from reliability (section 4.8).
- **No external review.** No external reviewer was available. An independent internal check ran on this draft instead (see "Review").

Section 5 gives every limitation in full.

## 1. Method

This section says what was evaluated and how. The engine's full equations are in its model description, [`docs/model.md`](https://github.com/openpickleball/engine/blob/v1.0.0/docs/model.md). This section summarises them and adds what is specific to the evaluation: the configuration, how win probabilities were scored, the assumptions, and the procedures behind each number. Every number has the committed file it comes from next to it. Paths without a repository name are in this repository.

### The engine in brief

OPEN is a performance-based Elo rating. The steps below follow one match through the engine ([model.md](https://github.com/openpickleball/engine/blob/v1.0.0/docs/model.md), sections "[Team rating](https://github.com/openpickleball/engine/blob/v1.0.0/docs/model.md#team-rating)" to "[Reliability](https://github.com/openpickleball/engine/blob/v1.0.0/docs/model.md#reliability)").

- **Team rating.** A team's rating is the mean of its players' ratings before the match.
- **Expected point share.** Team A's expected share of all points is `E = 1 / (1 + 10**((R_B - R_A) / scale))`. There is one scale for side-out scoring and one for rally scoring.
- **Result.** Team A's performance `S` is its share of all points in the match, summed over games. A weight `win_weight` can blend in the win. In the evaluated configuration it is 0, so `S` is the point share. The surprise is `S - E`.
- **Evidence weight.** A match counts by a weight from its match type (tournament 1.0, league 0.9), times a multiplier for game length and number of games, capped at 1.3 ([model.md, "Evidence weight"](https://github.com/openpickleball/engine/blob/v1.0.0/docs/model.md#evidence-weight); the values are the defaults in "[Parameters](https://github.com/openpickleball/engine/blob/v1.0.0/docs/model.md#parameters)", which `v1` keeps).
- **K schedule.** Each player has a rating deviation (RD). Their `K` grows linearly with their own RD, from `k_min` at `rd_min` to `k_max` at `rd_max`. A partner's or opponent's RD never changes a player's K.
- **Update.** Each player moves by `K * weight * surprise`, team A up and team B down, clamped to the rating range. Because K is per player, a match need not be zero-sum.
- **RD.** RD grows in quadrature with days of inactivity, up to `rd_max`. With each counted match it shrinks by the fraction `rd_shrink_per_match` times the match weight, down to `rd_min`. Inactivity never moves a rating.
- **Reliability.** A 0 to 100 label: `100 * certainty * (0.60 + 0.40 * variety)`. Certainty comes from RD. Variety counts recent partners and opponents against a target. Reliability does not enter the update or the win probability, so this evaluation does not score it.

### The evaluated configuration

The configuration evaluated is the engine's published parameter set `v1`. In engine 1.0.0 it ships in the package as [`src/open_engine/params/v1.json`](https://github.com/openpickleball/engine/blob/v1.0.0/src/open_engine/params/v1.json) and is listed in [PARAMETERS.md, "v1"](https://github.com/openpickleball/engine/blob/v1.0.0/PARAMETERS.md#v1). A caller loads it with `Config.published("v1")`. That reads the packaged file and passes it to `Config.from_parameter_set`, which accepts it only if its label is a set this release publishes and its values give the parameter fingerprint published for that label ([docs/parameters.md, "The loader"](https://github.com/openpickleball/engine/blob/v1.0.0/docs/parameters.md#the-loader); `Config.published` in [`src/open_engine/_config.py`](https://github.com/openpickleball/engine/blob/v1.0.0/src/open_engine/_config.py)). So a config loaded this way always has the published values. The evaluation loads `v1` this way. `v1` was fitted in `reports/v1-fit-2026-11/v1-fit.md` on every match through the freeze date 2026-11-20: 1,601 doubles matches, all side-out scoring, 1,283 league and 318 tournament, among 189 players on 52 dates from 2025-08-10 (v1-fit.md, "Data"). The fit ran on engine 0.3.0.dev0 at commit `e62468c`. The engine reports that the rating math has not changed between that commit and the engine that added `v1` ([PARAMETERS.md, "Fingerprints"](https://github.com/openpickleball/engine/blob/v1.0.0/PARAMETERS.md#fingerprints)). Rerunning the command behind each report that ran on `e62468c`, on the same data under engine 1.0.0, changed no fit, fold, score, difference or verdict. Only provenance lines differ (engine version and commit, config fingerprints, evaluation commit), plus a new field-table row for `params_version` in v1-fit.md. The committed reports were not regenerated, so they still name `e62468c`.

**Fingerprints.** The evaluation recorded `Config.fingerprint()` on engine 0.3.0.dev0: `22b53a37587d6881` for `v1` and `17f2981186edf96c` for `Config()` (v1-fit.md, "The V1 config"). Engine 1.0.0 publishes `v1` with parameter fingerprint `f9149f3c38a01efc` ([PARAMETERS.md, "v1"](https://github.com/openpickleball/engine/blob/v1.0.0/PARAMETERS.md#v1); [`v1.json`](https://github.com/openpickleball/engine/blob/v1.0.0/src/open_engine/params/v1.json)). `Config.published("v1").parameter_fingerprint()` gives the same value under the installed 1.0.0 (`reports/v1-gap-2026-11/v1-gap.md`, "Reproduction"). The values are the same; the hashes differ because the older scheme hashed the package version with the values, while `parameter_fingerprint()` leaves out both version labels and hashes the parameter values alone ([PARAMETERS.md, "Fingerprints"](https://github.com/openpickleball/engine/blob/v1.0.0/PARAMETERS.md#fingerprints); [docs/parameters.md, "Identity"](https://github.com/openpickleball/engine/blob/v1.0.0/docs/parameters.md#identity)).

**What was fitted.** Only two quantities were fitted, both by the engine's strict tune on the fit window (see "Procedures" below).

- The side-out scale, by point-share mean squared error over [0.5, 8.0] at default K: s_fit = 1.112, not at a bound, on 667 matches scored in the window (v1-fit.md, "Side-out scale").
- The side-out win calibration, by Brier score: `a` = 0.476, not at a bound, on the same 667 predictions. Fit-window Brier is 0.2302 at the fitted `a` and 0.2408 at `a` = 1 (v1-fit.md, "Side-out win calibration").

**What was declared.** The side-out scale is then declared 1.0, a unit, and `k_min` and `k_max` are multiplied by 1.0 / s_fit. Scale and K together only set the units of a rating while no rating reaches a bound ([model.md, "What a rating means"](https://github.com/openpickleball/engine/blob/v1.0.0/docs/model.md#what-a-rating-means), "Scale and K together"). So this rescaling leaves every prediction unchanged. `reports/side-out-unit-2026-11/side-out-unit.md` checks this on every match: the largest difference in any predicted quantity is 4.6e-15, and no match touches 2.0 or 8.0. The rally scale is declared equal to the side-out scale, and rally `a` is 1.0. There are no rally matches, so neither is validated.

**Every value.** The table lists every `Config` field in `v1` except the two version labels. Values and statuses are from v1-fit.md, "The V1 config", and match [`v1.json`](https://github.com/openpickleball/engine/blob/v1.0.0/src/open_engine/params/v1.json) at `v1.0.0`. "Fitted" here includes `k_min` and `k_max`, which are the default K divided by s_fit. "Default" means left at the `Config()` value, mostly because the data cannot identify it. "Label" means it changes no rating.

| Field | `v1` value | `Config()` value | Status |
| --- | --- | --- | --- |
| `scale_by_scoring` | side-out 1.0, rally 1.0 | 2.9, 2.9 | declared |
| `win_calibration_by_scoring` | side-out 0.476, rally 1.0 | 1.0, 1.0 | side-out fitted, rally declared |
| `k_min`, `k_max` | 0.1079136690647482, 0.539568345323741 | 0.12, 0.6 | fitted (default / s_fit) |
| `min_rating`, `max_rating`, `default_rating` | 2.0, 8.0, 3.5 | same | declared |
| `rd_min`, `rd_max`, `new_player_rd` | 0.1, 0.8, 0.8 | same | default |
| `rd_shrink_per_match`, `rd_growth_per_day` | 0.08, 0.012 | same | default |
| `external_seed_rd`, `category_seed_rd` | 0.42, 0.5 | same | default |
| `win_weight` | 0.0 | same | declared |
| `match_weights` | tournament 1.0, league 0.9, rec_verified 0.75, self_reported 0.4 | same | declared |
| `game_length_weights` | 11: 1.0, 15: 1.1, 21: 1.2 | same | default |
| `game_count_bonus`, `max_information_multiplier` | 0.12, 1.3 | same | default |
| `allowed_game_formats` | side-out and rally, to 11, 15 and 21 | same | declared |
| `replay_interval_days` | 7 | same | declared |
| `self_reported_requires_both_teams`, `self_reported_repeat_window_days`, `self_reported_repeat_factor`, `self_reported_max_repeats` | true, 90, 0.5, 4 | same | declared |
| `reject_content_duplicates` | false | same | declared |
| `max_singles_rating_gap`, `max_doubles_partner_gap` | null, null | same | declared |
| `category_feeds_parent` | true | same | default |
| `diversity_window_days`, `diversity_target` | 365, 12 | same | label |
| `import_reliability_carry_matches`, `import_reliability_max_age_days` | 10, 365 | same | label |
| `established_reliability_threshold`, `established_min_verified_matches` | 60.0, 3 | same | label |
| `connectivity_reliability_cap`, `connectivity_min_shared_contacts`, `connectivity_min_contact_days`, `connectivity_min_outside_players` | null, 4, 3, 8 | same | label |

Only four fields differ from `Config()`: `scale_by_scoring`, `win_calibration_by_scoring`, `k_min` and `k_max`. The RD schedule fields (`rd_min`, `rd_max`, `new_player_rd`, `rd_shrink_per_match`, `rd_growth_per_day`) were left at default because fitting them on about 1,600 matches of mostly regular players would fit noise. `external_seed_rd` matters only for seeding, and `v1` seeds no one. The category fields have no category matches in the data. The game-length fields were left at default because every game is to 11 and best of 3 occurs only in tournaments (v1-fit.md, "The V1 config"). The version labels (`version`, and on the engine side `params_version`) change no rating.

### Win probability and calibration

Scoring uses the engine's own win probability ([model.md, "Win probability"](https://github.com/openpickleball/engine/blob/v1.0.0/docs/model.md#win-probability)).

1. The expected point share `p` is the probability that team A wins any single point. Points are treated as independent and identically distributed.
2. The game win probability follows exactly from `p`, the points needed, the win-by rule and any cap. A best-of series is combined the same way, with games independent.
3. The raw probability is clipped to [0.000001, 0.999999] and calibrated: `P = 1 / (1 + exp(-a * ln(p_c / (1 - p_c))))`. An `a` below 1 pulls probabilities toward 0.5. The independence assumption only approximates side-out scoring, where only the serving side can score. The evaluation treats the calibration as the correction for it.

Every win metric of the engine (Brier score, log loss, accuracy) uses the calibrated probability. Point-share metrics use `E` directly. Predictions are prequential and scheduled, as the engine publishes ratings: each match is predicted from a replay as of the latest weekly replay date strictly before it, plus the matches recorded live since. No match is predicted from ratings that have seen it.

### The scale on its own terms

A rating places a player relative to the players they have met, directly or through others. It is not tied to an outside measure of skill ([model.md, "What a rating means"](https://github.com/openpickleball/engine/blob/v1.0.0/docs/model.md#what-a-rating-means), "The level"). Ratings run from 2.0 to 8.0. Every player starts at 3.5. Under `v1` no player is seeded.

**What a gap means.** A rating gap is stated as a share of points, not as a chance of winning. Under `v1` the side-out scale is 1.0. A team rating gap of 0.5 means the stronger team is expected to win about 76.0% of the points. A gap of 1.0 means about 90.9% of the points ([model.md, "What a rating means"](https://github.com/openpickleball/engine/blob/v1.0.0/docs/model.md#what-a-rating-means); [README, "What a rating means"](https://github.com/openpickleball/engine/blob/v1.0.0/README.md#what-a-rating-means); `reports/side-out-unit-2026-11/side-out-unit.md`, "What a rating gap means"; `reports/v1-gap-2026-11/v1-gap.md`, "What a gap means: point share"). This meaning is declared, so it does not move when the data or the fit changes. Changing it takes a new published parameter set.

**As a win probability.** The chance of winning depends on more than the gap: on the game format, the number of games and the win calibration `a` ([model.md, "What a rating means"](https://github.com/openpickleball/engine/blob/v1.0.0/docs/model.md#what-a-rating-means)). Under `v1`, for side-out scoring to 11, win by 2, no cap (`GameFormat()`), the stronger team's chance to win is:

| Team rating gap | Best of | Raw win probability | Calibrated win probability (`a` = 0.476) |
| --- | --- | --- | --- |
| 0.5 | 1 | 99.65% | 93.64% |
| 0.5 | 3 | 99.996% | 99.23% |
| 1.0 | 1 | 99.99997% | 99.86% |
| 1.0 | 3 | 99.99999999997% | 99.86% |

Source: `reports/v1-gap-2026-11/v1-gap.md`, "As a win probability". The calibrated value is the one the evaluation scores. A small edge on each point adds up over a game, so the raw win probability is far from 50%, much further than the point share. The fitted `a` of 0.476 pulls it back toward 50%. At a gap of 1.0 the raw probability is above 1 - 1e-6, so the engine clips it before calibrating, and 99.86% is the largest calibrated value `v1` can give for side-out scoring.

*Method.* The calculation reads no match data. Under engine 1.0.0 it loads `v1` with `Config.published("v1")` (parameter fingerprint `f9149f3c38a01efc`) and seeds two singles players at 3.5 + gap and 3.5. It takes the point share from `RatingEngine.predict_point_share`, the raw probability from the engine's `match_win_probability` of that share, and the calibrated probability from `RatingEngine.predict_win_probability`. This is the same method as model.md's example for the `Config()` defaults ([model.md, "Win probability"](https://github.com/openpickleball/engine/blob/v1.0.0/docs/model.md#win-probability)). Singles is used only to set an exact gap; the point share and the win model depend on the gap and the scoring type, not the rating format. These are model figures, not measured win rates.

### Assumptions

These are assumptions of the method, not claims that they hold for real play ([model.md, "Assumptions"](https://github.com/openpickleball/engine/blob/v1.0.0/docs/model.md#assumptions)). Whether ratings built on them predict well is what the evaluation tests.

- **Matches are independent given the ratings before them.** Each update reads only the match and the players' states before it. No later match revises an earlier update.
- **Points within a game are independent**, each won by team A with the same probability `p`. Games are independent too. This is used only for win probability. It suits rally scoring better than side-out scoring. The evaluation fits the calibration `a` to correct for it.
- **One model for doubles and singles.** Both use the same expected share, performance, weight, K and update, with a separate rating in each format. This evaluation has only doubles, so the singles side is untested here.
- **No partner effect.** A team rating is the mean of its players' ratings. The model has no term for how well two partners play together.
- **A closed pool, every player starting at 3.5.** Ratings are relative to this pool. Pools that share no players are not comparable.

### Procedures behind the numbers

All the procedures below are part of the initial evaluation and were not pre-registered. Each rule was written down before it ran on real data. The test-part matches had been seen in aggregate before, so no result here is confirmatory.

**Strict tune.** The scale fit and the calibration fit use scheduled predictions with a burn-in of 5 matches per player. Only predictions on or after the fit-window start enter the loss. The window starts at the first match date on or after the midpoint of the first and last dates, 2026-04-12 for the final fit (v1-fit.md, "The procedure"). The tune refuses, rather than returning a value, when a fit lands at its search bound.

**Rolling-origin layout.** An expanding window. Fold 1's training part ends at the fit-window midpoint date, 2026-04-12. Then come disjoint test blocks of 5 match dates each, to the last match. That gives 6 folds, the last with 2 dates. Each fold runs the full procedure on its training part only and predicts its block once. Paired differences are pooled across folds. The blocks are disjoint, and the two-way bootstrap resamples players to allow for the training data and players the folds share. The fold dates were written down before any run.

**Two-way bootstrap.** Players recur across dates, so resampling dates alone gives intervals that are too narrow. The decision interval resamples match dates and players independently. It is conservative: it counts some shared dependence twice, so it can be wider than needed (v1-fit.md, "The default-change test"). Intervals are 95%, from 2,000 resamples with seed 20261114. The date bootstrap is reported next to it for information only.

**The default-change rule.** The rule, fixed before any run: `v1` replaces the `Config()` defaults only if the pooled paired difference, `Config()` minus `v1`, has a two-way 95% lower bound above 0 on Brier or on log loss, and its Brier lower bound is above -0.005. On the 6 folds, 1,018 paired matches:

| Metric | `Config()` minus `v1` | Two-way 95% interval |
| --- | --- | --- |
| Brier | +0.0047 | [-0.0044, +0.0139] |
| Log loss | +0.0093 | [-0.0106, +0.0295] |

Source: v1-fit.md, "The default-change test" and "Decision". The point estimates favour `v1`. The non-inferiority bar, which is on Brier only, passed by 0.0006 (lower bound -0.0044 against -0.005). The log-loss lower bound, -0.0106, is below -0.005, so `v1` is not shown to be no worse on log loss ([PARAMETERS.md, "Default-change rule and result"](https://github.com/openpickleball/engine/blob/v1.0.0/PARAMETERS.md#default-change-rule-and-result)). Neither lower bound is above 0, so the gain bar did not pass. The defaults were not replaced. `v1` ships as a named parameter set, loaded explicitly, and is the configuration this paper evaluates.

## 2. Data

**Source.** The data are one club's play: the exports of six leagues and the results of one-day tournaments at the same club, from 2025-08-10 to 2026-11-20. Every match is doubles with side-out scoring, and every game is to 11. There are 1601 matches among 189 players on 52 match dates: 1283 league and 318 tournament matches, and 0 rally matches (`reports/v1-fit-2026-11/v1-fit.md`, "Data").

**Size.** This is a small sample: under 2,000 matches and about 190 players, a small, mostly closed pool (`reports/initial-2026-11/technical-report.md`, "Limitations"). For the initial evaluation's time-ordered split, the tuning part (on or before 2026-08-23) has 1087 matches (1033 league, 54 tournament) on 41 dates, and the test part (from 2026-08-24) has 514 matches (250 league, 264 tournament) on 11 dates. 79 players in the test part never played in the tuning part (technical report, "Data").

**Cleaning and exclusions.** League games have no cap. Tournament games are capped at 13 (technical report, "Data"). Best of 3 occurs only in tournaments (v1-fit.md, "The V1 config"). The tournament results give mostly first names, which were mapped to full names by hand; a wrong mapping would merge two players' histories (technical report, "Limitations"). No match falls after the freeze date 2026-11-20, so the final fit excludes 0 matches (v1-fit.md, "Data"). Scoring excludes matches whose players are below the burn-in: in the initial test part, 285 matches have all four players with at least 5 earlier matches, and 229 do not (technical report, section 1). The cleaning log, which records every dropped, corrected or merged record, holds player names, so it stays private. This writeup does not give its counts.

**Names and anonymization.** The processed data hold real player names. They are never committed or published, even with names removed. Only aggregates are published, and only those that pass a player-name check. The check refuses to write any report file that contains a player's full name or any part of a name longer than one character, matched case-insensitively on whole words. It does not check a lone name part that is also a word the reports print as a table label; the full name and its other parts are still checked. No per-player file is written. The exploratory real-data drift check withholds a week's mean rating when the group is small or when only a few players changed, so that no week's figure describes individual players (`reports/behavior-2026-11/behavior.md`, "On real data (exploratory)").

## 3. Validation design

All of the design below is part of the initial evaluation and was not pre-registered. Each rule was written down before it ran on real data. No match is ever predicted from ratings that have seen it.

**Time-ordered split.** The initial evaluation tunes on the tuning part only (on or before 2026-08-23) and scores the test part (from 2026-08-24) once. The strict tune fits the side-out scale and the win calibration `a` on a fit window from 2026-02-15 to 2026-08-23, and tuned Elo's K on the tuning part only (technical report, "Tuned configuration"). Each test match is predicted from a scheduled replay as of the latest weekly replay date strictly before it, plus the matches recorded live since (section 1, "Win probability and calibration"). A match is scored when all four players have at least 5 earlier matches.

**Rolling splits inside the initial evaluation.** For information only, 4 expanding-window folds each re-tune the engine and every baseline on every match before a test block of 6 nights, then score the block. The folds share training data and players, so they are summarised across folds with no pooled interval, and they are not an independent test (`reports/initial-2026-11/rolling-splits.md`).

**Rolling-origin layout.** The default-change test for `v1` was run on a second layout over the whole dataset. Fold 1's training part ends at the fit-window midpoint date, 2026-04-12. Then come disjoint test blocks of 5 match dates each, to the last match. That gives 6 folds, the last with 2 dates (2026-11-08 to 2026-11-20). Each fold runs the full procedure on its training part only and predicts its block once, so each fold refits s_fit and `a` (0.671 to 1.079 and 0.393 to 0.469 across folds). Paired differences are pooled across folds: 1018 paired matches. The blocks are disjoint, and the two-way bootstrap resamples players to allow for the training data and players the folds share. The fold dates were written down before any run (`reports/v1-fit-2026-11/v1-fit.md`, "The default-change test").

**Intervals.** Metrics on real data have 95% percentile intervals from 2000 bootstrap resamples with seed 20261114, except the coin flip, which is constant. The initial evaluation resamples match dates (league nights and tournament days), except its frozen-against-live check, which resamples matches. Players recur across dates, so these date intervals are likely too narrow (technical report, "Limitations"). The two-way bootstrap resamples match dates and players independently. It is conservative: it counts some shared dependence twice, so it can be wider than needed (v1-fit.md, "The default-change test"). The two-way interval decides the default-change test; the date interval is shown beside it for information. The simulations report the 10th and 90th percentiles across replicates, not intervals.

**Baselines.** Every baseline is scored on the same matches as the tuned engine (285 test matches on 11 match dates in the initial evaluation, 63 of them tournament matches): the engine with its default config, Elo tuned (K = 16) and default (K = 32, start 1500), Glicko-2 tuned and default, TrueSkill tuned and default, "higher-rated team wins" (a hard call from the engine's expected share) and a coin flip (0.5 for every match). Tuned baselines are tuned on the tuning part only. Only the tuned engine row is calibrated. All baselines use win or loss only (technical report, section 1).

**Metrics.** Brier score, log loss and accuracy of the calibrated win probability; the Brier score before calibration; the point-share mean squared error; the favourite-side point-share bias (actual minus expected share, from the predicted favourite's side); the paired Brier difference of each baseline against the engine; reliability bins of predicted against observed win rate; and, for tournaments, accuracy, AUC, Brier and log loss of the engine frozen at event start against the engine live within the event (technical report, sections 1 to 3).

**Decision rules.** Two questions had a rule fixed before running. The declared-scale proposal passes if the paired Brier difference's lower end is above -0.005. `v1` replaces the defaults only if the two-way lower bound of the pooled difference is above 0 on Brier or log loss, and its Brier lower bound is above -0.005.

**Bars from the earlier plans.** Two earlier, unpublished evaluation plans set two bars: the engine not worse than tuned Elo (the lower end of the Elo-minus-engine Brier interval above -0.005), and a favourite-side point-share bias within 0.015 in absolute value. The initial evaluation reports them for reference only; they decide nothing here (technical report, section 1).

## 4. Results with intervals

These results are from the initial evaluation, which ran on engine 0.3.0.dev0 at commit `e62468c`. Rerunning the commands under engine 1.0.0 on the same data changed no fit, fold, score, difference or verdict; only provenance lines differ (section 1, "The evaluated configuration"). Section 4.1, section 4.2 (but for its last paragraph) and section 4.6 score the configuration tuned on the tuning part (side-out scale 0.868, `a` = 0.402), not the published set `v1`, and section 4.3 retunes the same procedure in each fold. Section 4.4, the last paragraph of section 4.2, and the side-out unit check of section 4.5 use the `v1` procedure (the strict tune with the side-out unit declared), and the doubles and gaming simulations run `v1` and `Config()` side by side.

### 4.1 Predictive accuracy and baselines

On the 285 scored test matches on 11 match dates (technical report, section 1), with 95% intervals resampling match dates (the report calls them league nights, but 63 of the matches are tournament matches):

| Model | Brier | Log loss | Accuracy |
|---|---|---|---|
| Engine, tuned | 0.2224 [0.2127, 0.2312] | 0.6340 [0.6123, 0.6546] | 0.628 [0.579, 0.687] |
| Engine, default config | 0.2253 [0.2101, 0.2425] | 0.6407 [0.6042, 0.6808] | 0.618 [0.572, 0.650] |
| Elo, tuned | 0.2399 [0.2336, 0.2453] | 0.6726 [0.6599, 0.6836] | 0.582 [0.528, 0.633] |
| Elo, default | 0.2386 [0.2290, 0.2466] | 0.6698 [0.6497, 0.6864] | 0.579 [0.526, 0.631] |
| Glicko-2, tuned | 0.2294 [0.2204, 0.2355] | 0.6505 [0.6317, 0.6630] | 0.619 [0.573, 0.668] |
| Glicko-2, default | 0.2306 [0.2203, 0.2378] | 0.6530 [0.6316, 0.6679] | 0.605 [0.564, 0.654] |
| TrueSkill, tuned | 0.2328 [0.2139, 0.2454] | 0.6617 [0.6211, 0.6912] | 0.621 [0.580, 0.678] |
| TrueSkill, default | 0.2311 [0.2146, 0.2427] | 0.6552 [0.6205, 0.6814] | 0.614 [0.569, 0.678] |
| Higher-rated team wins | 0.3719 [0.3127, 0.4214] | not defined | 0.628 [0.579, 0.687] |
| Coin flip | 0.2500 (constant) | 0.6931 (constant) | 0.500 (constant) |

Paired Brier difference, the row's Brier minus the tuned engine's (positive favours the engine): default-config engine +0.0029 [-0.0062, +0.0138]; tuned Elo +0.0175 [+0.0095, +0.0259]; default Elo +0.0162 [+0.0071, +0.0258]; tuned Glicko-2 +0.0070 [-0.0002, +0.0130]; default Glicko-2 +0.0082 [+0.0010, +0.0131]; tuned TrueSkill +0.0104 [-0.0075, +0.0203]; default TrueSkill +0.0087 [-0.0068, +0.0173]; coin flip +0.0276 [+0.0188, +0.0373].

The tuned configuration (not `v1`) was ahead of tuned Elo by +0.0175 [+0.0095, +0.0259], on a date interval that is likely too narrow, and by only +0.0011 [-0.0130, +0.0168] in rolling fold 2 (section 4.3). It is also ahead of tuned Glicko-2 and tuned TrueSkill on point estimates, but those intervals include zero, and so does the gap to the engine's own default config. The point-share MSE is 0.03906 [0.03415, 0.04389], and the favourite-side point-share bias is -0.0143 [-0.0423, +0.0201]. By match type, the engine's Brier is 0.2233 [0.2145, 0.2320] on 222 league matches and 0.2190 [0.1960, 0.2453] on 63 tournament matches. Against the earlier plans' bars, for reference only: the engine is not worse than tuned Elo, and the bias point estimate is within 0.015.

### 4.2 Calibration

The tuned win calibration `a` is 0.402, below 1, so it pulls the raw probabilities toward 0.5. Before calibration the engine's Brier score is 0.2351 [0.2166, 0.2527], behind tuned Glicko-2 and tuned TrueSkill though ahead of tuned Elo; after it, 0.2224. The reliability table bins the calibrated probabilities (technical report, section 2):

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

The top two bins hold only 3 and 2 matches, so their observed rates say little. The report has no merged row for them, so the bins are shown as reported.

In the final `v1` fit on every match, the fitted `a` is 0.476, with a fit-window Brier of 0.23023298364793812 at the fitted `a` against 0.2408311593261697 at `a` = 1 (v1-fit.md, "Side-out win calibration"). Every `a` fitted on the real data, in every fold of both layouts, is between 0.393 and 0.503 (v1-fit.md, "The default-change test"; `rolling-splits.md`). That is consistent with the independent-points model being too sharp for side-out play.

### 4.3 Rolling splits

Across the 4 folds of the initial evaluation's rolling splits (`rolling-splits.md`, "Spread across folds"), the tuned engine's Brier ranges from 0.2199 to 0.2584 (mean 0.2310). Tuned Elo minus engine ranges from +0.0011 to +0.0305 (mean +0.0166), so the engine is ahead in every fold, though by very little in fold 2: +0.0011 [-0.0130, +0.0168]. Tuned Glicko-2 minus engine ranges from -0.0147 to +0.0089 (mean +0.0016), and tuned TrueSkill minus engine from -0.0054 to +0.0214 (mean +0.0077): in fold 2 both were ahead of the engine. In fold 2 the tuned engine's point estimate was worse than a coin flip (Brier 0.2584 against 0.2500; coin flip minus engine -0.0084 [-0.0190, +0.0041], so the interval includes zero), and tuned Glicko-2 beat it by 0.0147 (Glicko-2 minus engine -0.0147 [-0.0270, -0.0026]). The favourite-side bias ranges from -0.0383 to +0.0373, and is outside 0.015 in 3 of the 4 folds. Results move a lot between blocks of about 6 nights.

### 4.4 The `v1` fit and the default-change test

The final fit on all 1601 matches gives s_fit = 1.112 and `a` = 0.476, neither at a bound, on 667 fit-window matches (v1-fit.md, "The final fit"). On the 6 folds of the rolling-origin layout, `Config()` scored Brier 0.2379 and `v1` 0.2332 on 1018 paired matches (log loss 0.6689 and 0.6596; accuracy 0.6007 and 0.6125).

| Metric | `Config()` minus `v1` | Two-way 95% interval (decision) | Date bootstrap (too narrow) |
| --- | --- | --- | --- |
| Brier | +0.0047 | [-0.0044, +0.0139] | [+0.0005, +0.0090] |
| Log loss | +0.0093 | [-0.0106, +0.0295] | [-0.0001, +0.0186] |

The non-inferiority bar, which is on Brier only, passed by 0.0006 (Brier lower bound -0.0044 against -0.005). The log-loss lower bound is -0.0106, below -0.005, so `v1` is not shown to be no worse on log loss (engine PARAMETERS.md at `v1.0.0`). The gain bar did not pass (neither lower bound is above 0). So `v1` shows no measurable gain over `Config()` on these data, and the defaults were not replaced (v1-fit.md, "Decision"). The test had little power for a gain this size: the two-way interval reaches about twice as far below the Brier estimate of +0.0047 as the estimate is above 0 (lower bound -0.0044).

### 4.5 Sensitivity and alternatives

**Declared scale with K tuned.** Declaring the side-out scale 2.9 and tuning K instead scored Brier 0.2257 [0.2153, 0.2364] on the 285 test matches, against 0.2224 for the tuned engine. The paired difference, current minus declared, is -0.0033 [-0.0069, -0.0003], whose lower end is not above -0.005: **FAIL**. The declared engine also put 60 matches at the 2.0 rating floor (`reports/declared-scale-2026-11/declared-scale.md`).

**The K and scale ridge.** Without rating bounds, multiplying the scale and K by the same factor changes no prediction: the largest win probability difference is 2.2e-15 or less at every scale tried, and Brier stays 0.234582 (step 2 holds). With the 2.0 and 8.0 bounds, larger scales push ratings to the floor (94 matches at 2.9) and the predictions differ, though Brier does not rise strictly with the scale (step 3 does not hold) (`reports/ridge-check-2026-11/ridge-check.md`).

**The side-out unit.** Declaring the scale 1.0 and dividing K by s_fit predicts what the fitted engine predicts: the largest per-match difference in the calibrated win probability is 2.4e-15 over 1601 matches, and no match touches a bound (`reports/side-out-unit-2026-11/side-out-unit.md`).

**Updating within a tournament day.** Updating ratings after each match within a tournament day, rather than freezing them at the start of the day, made no clear difference. In the initial test part, on 219 tournament matches from 4 events, live minus frozen Brier is +0.0056 [-0.0070, +0.0190], with intervals that resample matches and so are too narrow (technical report, section 3).

**Not run.** An earlier plan's sensitivity grid (multiples of the scale and `a`, K, burn-in and `win_weight`) was never run: that plan was rescoped to the initial evaluation before its lock. No report holds it.

### 4.6 New players and stability

Prediction is worst when a player is new. With no burn-in, the engine's Brier is 0.2602 [0.2238, 0.2944] on 46 test matches where some player had no earlier match, against 0.2454 for tuned Elo and 0.2500 for a coin flip (a worse point estimate than the coin flip, though the interval includes it), and 0.2345 [0.2154, 0.2497] on 183 matches where the least experienced had 1 to 4. From 5 earlier matches on it is between 0.2202 and 0.2246, ahead of tuned Elo in every group (technical report, section 4). `v1` seeds no one, so every new player starts at 3.5. For the 64 players with at least 30 matches, the median number of matches until a rating stays within 0.25 of its later level is 6 (quartiles 1 to 18).

Ratings are stable week to week. Over 68 weekly replays, the median weekly change for players with at least 10 earlier matches who played is 0.033 (90th percentile 0.083), players who did not play never move (0.0000), and the median rank correlation with the previous week is 0.992 (minimum 0.956) (technical report, section 5).

### 4.7 System behavior on simulated leagues

These simulations use `Config()`, Elo, Glicko-2 and TrueSkill at their defaults, on leagues whose games are generated from the engine's own point-share model, so they favour the engine by construction (`reports/behavior-2026-11/behavior.md`, "System behavior"; 20 replicates, seed 0; brackets are the 10th and 90th percentiles across replicates).

- **Ranking recovery.** In pools of 16, 32 and 64, the engine's median Spearman correlation with true skill reaches 0.9 at 36, 48 and 40 matches per player, earliest of the four systems in each pool. After 60 matches it is 0.93 [0.85, 0.96], 0.93 [0.87, 0.95] and 0.94 [0.91, 0.95].
- **Newcomer convergence.** Newcomers who join an established pool settle below their true skill: by -0.147 [-0.276, +0.011] (true 3), -0.265 [-0.387, -0.129] (true 3.8) and -0.436 [-0.559, -0.346] (true 4.6). The pool's own ratings sit below true skill because every player starts at 3.5 while the true mean is 3.8. Median reliability reaches 100.0 by match 40, yet for the middle and high newcomers it was at least 90 while the median rating was still 0.25 or more from true skill.
- **Inflation and deflation.** In a pool of 40 with turnover over 104 weeks, the gap between mean rating and mean true skill trends by +0.024 [-0.041, +0.055] rating points per year. The median slope counts as inflation, but the range spans zero, so the direction is not consistent. The gap itself stays near -0.285 [-0.362, -0.236] at week 104.
- **Pool isolation.** Two pools whose true means differ by 0.595 [0.446, 0.706], playing only within themselves, end with an offset of -0.595 [-0.706, -0.446]: the engine puts them level. With 8 cross-pool matches a night for 30 nights the offset is still -0.222 [-0.260, -0.149], and it is never within 0.1 in any replicate at any rate tried.

**On the real data (exploratory).** Over 68 weekly replays, the mean rating of players with 10 or more matches moved from 3.485 to 3.500 (+0.014), within a range of 3.485 to 3.509. Real players' true skill is unknown, so this is not a test of drift (behavior.md, "On real data (exploratory)").

### 4.8 Doubles and gaming

This section asks how the engine handles doubles-specific situations and deliberate gaming. Every number comes from `reports/behavior-2026-11/behavior.md`, section "Doubles and gaming", run with seed 0 and 20 replicates. The subsection of that section is named next to each number. Ranges in brackets are the 10th and 90th percentiles across the 20 replicates.

#### Setup and units

Each scenario simulates a league with known true skill. Each rally goes to team A with `expected_point_share(team A skill, team B skill, 2.9)`, one game to 11, win by 2. Every scenario runs under parameter set `v1` (the evaluated configuration; section 1, "The evaluated configuration") and under the engine defaults `Config()`. Both configs see the same simulated matches. The variants of a scenario (carry fractions, truth models, sandbagging modes, farming and control) share the replicate's skills and the starting state of its random generators, but their matches diverge once the variants draw differently.

True skill is on the `Config()` unit. Under `v1` the side-out scale is 1.0, so a `v1` rating gap is 2.9 times smaller for the same expected share. Every rating quantity here is in truth units: divide by 2.9 for `v1` units. For example, an error of 0.8 truth units is 0.28 in `v1` units. In truth units `v1`'s K is 2.6 times `Config()`'s, so `v1` moves faster and is noisier.

A player's **error** is relative: (their rating minus the pool's mean rating) minus (their true skill minus the pool's mean true skill). Positive means rated too high.

Two truth models set how strong a team really is. Under the **mean** model a team's skill is the mean of its players', which is what the engine assumes ([model.md, "Team rating"](https://github.com/openpickleball/engine/blob/v1.0.0/docs/model.md#team-rating)). Under the **weaker-weighted** model the weaker player counts twice: (2 * weaker + stronger) / 3. We chose both; neither was measured on real play.

#### Carrying

A weak player W (true 3.0) and a strong player S (true 4.6) join 22 background players. Everyone plays 90 matches. In each round, with probability f, W and S are partners. The carrying effect is a player's error minus their error at f = 0 in the same replicate.

At f = 1, under the mean model, W ends rated too high by +0.802 [+0.664, +0.898] under `v1` and +0.792 [+0.715, +0.898] under `Config()`. S ends rated too low by -0.798 [-0.936, -0.702] and -0.808 [-0.885, -0.702]. The carrying effect on W is +0.878 [+0.685, +1.146] under `v1` and +0.680 [+0.608, +0.851] under `Config()`. At f = 0.5 it is still +0.173 [-0.134, +0.583] and +0.211 [+0.084, +0.471]. W's reliability is 100.0 in every case. Under the weaker-weighted model at f = 1, S is underrated by -0.988 [-1.150, -0.871] and -0.979 [-1.041, -0.910].

Source: behavior.md, "Carrying" (seed 0, 20 replicates).

A fixed pair of equal players (P and Q, both 3.8) always partners against random background players. P and Q are equal, so the weaker-weighted model changes only the other teams' skill: their weaker players count twice, which makes them weaker than their mean rating says. Under that model the pair is rated slightly high: +0.073 [-0.007, +0.281] under `v1` and +0.082 [+0.022, +0.177] under `Config()`, whose band excludes zero. Under the mean model the row, +0.002 [-0.136, +0.098] and -0.008 [-0.085, +0.098], is not independent evidence: P + Q and W + S both have team skill 3.8 and use the same schedule and game seeds, so the matches are those of the f = 1 carrying run and P's error is W's error minus 0.8.

Source: behavior.md, "Carrying", fixed equal pair table (seed 0, 20 replicates).

**Vulnerable.** Two players who always play together start at the same rating and receive identical updates, so they always hold the same rating. The engine cannot tell who carries whom. At full carrying the weaker partner is overrated, and the stronger underrated, by about half their true gap of 1.6. Reliability gives no warning.

#### The partner-gap filter

`max_doubles_partner_gap` was set to 1.0 truth unit (0.345 in `v1` units) and the carrying league rerun at f = 1. The filter rejected 29.4% [18.0%, 36.1%] of matches under `v1` and 6.3% [3.6%, 12.7%] under `Config()`. W's error barely moved: +0.792 [+0.705, +0.944] with the filter against +0.802 [+0.664, +0.898] without under `v1`, and +0.796 [+0.728, +0.906] against +0.792 [+0.715, +0.898] under `Config()`.

Source: behavior.md, "Partner-gap filter" (seed 0, 20 replicates).

In the lopsided league below, the same filter rejected 44.0% [33.2%, 53.1%] of matches under `v1` and 15.0% [9.0%, 23.8%] under `Config()` (mean model).

Source: behavior.md, "Lopsided against balanced teams", rejection table (seed 0, 20 replicates).

**Vulnerable.** The filter reads ratings, not skill. W and S hold the same rating, so their partnership is never rejected. What it does reject is a large share of honest matches, more under `v1` than under `Config()`. A likely reason, not measured here, is that `v1`'s K is 2.6 times larger in truth units, so its ratings move further and spread more within the same nights.

#### Lopsided against balanced teams

**Analytic check.** For 4.5 + 2.5 against 3.5 + 3.5, the engine with correct ratings predicts the lopsided team 0.500 of the points. Its true share is 0.500 under the mean model and 0.434 under the weaker-weighted model. Source: behavior.md, "Lopsided against balanced teams" (no simulation).

**Simulation.** In a league of 32 with random partners, team L is the team with the larger true partner gap. The residual is L's actual point share minus the engine's expected share. Under the mean model the residual stays near zero in every bin: +0.0081 [-0.0207, +0.0257] under `v1` and +0.0016 [-0.0207, +0.0282] under `Config()` when L's gap exceeds the other team's by 1.0 or more. Under the weaker-weighted model the engine overpredicts lopsided teams more as the gap difference grows. The residual is -0.0179 [-0.0305, -0.0028] (`v1`) and -0.0210 [-0.0395, -0.0048] (`Config()`) for a difference of 0.5 to 1.0, and -0.0268 [-0.0429, +0.0046] and -0.0353 [-0.0644, -0.0024] for 1.0 and above.

Source: behavior.md, "Lopsided against balanced teams" (seed 0, 20 replicates).

**Real data (exploratory, in-sample).** On the real league and tournament matches, under each config, with all four players past 5 counted matches, gaps are rating gaps, not true ones. `v1` was fitted on these same matches. The mean residual is -0.0016 (730 matches, two-way 95% interval [-0.0336, +0.0291]) under `v1` and +0.0007 (936 matches, [-0.0282, +0.0332]) under `Config()` for gap differences under 0.5. For the largest bins it is -0.0306 (95 matches, [-0.1228, +0.0654]) under `v1` for 1.0 and above (0.345 and above in `v1` units), and -0.0357 (139 matches, [-0.1276, +0.0488]) under `Config()` for 0.5 to 1.0. `Config()` had only 8 matches at 1.0 and above, too few to report.

Source: behavior.md, "Lopsided teams on real data (exploratory)".

**Possibly vulnerable.** If real teams play closer to their weaker player, the engine has no partner term to correct for it (section 1, "Assumptions"). The real-data residuals point the same way in the largest bins, but every interval includes zero, so the data neither show nor rule out the effect.

#### Sandbagging

X (true 4.2) plays 60 honest matches, tanks 15, then plays 60 honest ones. In the soft tank X plays 1.0 below true skill. In the hard tank X's team loses every game 0-11.

The hard tank dropped X's error by 1.872 [1.759, 2.017] under `v1` (0.1248 per tanked match) and 0.800 [0.766, 0.839] under `Config()` (0.0533 per match). The soft tank dropped it by 0.461 [0.324, 0.609] and 0.191 [0.121, 0.246]. After a hard tank, X's error was not back within 0.25 of its pre-tank value after 60 honest matches in 12 of 20 replicates under `v1` and 16 of 20 under `Config()`. X ended at -0.348 [-0.599, -0.105] and -0.437 [-0.599, -0.387]. X's RD stayed at the minimum, 0.1000, and reliability at 100.0 throughout.

Source: behavior.md, "Sandbagging" (seed 0, 20 replicates).

**Vulnerable.** Fifteen thrown games put an established player far below their skill, and the rating stays low for most of the next 60 honest matches. RD sits at its floor, so K is at its smallest and recovery is slow. The engine raises no signal: RD and reliability do not change.

#### Farming

F (true 4.6) plays every round. After 20 established nights, F plays 20 nights with partner and opponents drawn from the 11 weakest players. A control places F at random. The farming gain is F's error after farming minus the control's. Under the mean model it is -0.037 [-0.299, +0.320] under `v1` and +0.036 [-0.095, +0.237] under `Config()`. Under the weaker-weighted model it is -0.139 [-0.360, +0.182] and -0.045 [-0.152, +0.145].

Source: behavior.md, "Farming" (seed 0, 20 replicates).

**Not shown to be vulnerable.** The engine scores point share against the expected share, so beating weak players by the expected margin earns nothing. Every interval spans zero.

#### Fixed-pair padding and closed groups

**Collusion.** In an established league of 24, four players record N fabricated matches among themselves. The target T partners each of the others in turn and wins every game 11-0. Recorded as league matches, every one counted. T gained +1.366 [+1.187, +1.523] at N = 10 and +3.493 [+3.086, +3.866] at N = 40 under `v1`, and +0.560 [+0.499, +0.599] and +1.820 [+1.623, +1.955] under `Config()`. The other three lost a third of that each (-1.164 [-1.289, -1.029] and -0.607 [-0.652, -0.541] at N = 40). The group's rating sum did not change (+0.0000), as expected from equal K. T's reliability stayed at 100.0. T's variety count was 45 [44, 46] before and after, well above the target of 12, so the variety term did not react.

Recorded as self-reported matches confirmed by a player from each team, the engine counts at most 4 per matchup in its 90-day window, each with half the weight of the one before. It counted 100.0%, 60.0% and 32.5% of N = 10, 20 and 40. T's gain levelled off at +0.390 [+0.336, +0.437] under `v1` and +0.150 [+0.133, +0.161] under `Config()` at N = 40, and T's reliability fell to 93.5.

Source: behavior.md, "Fixed-pair padding and closed groups", collusion table (seed 0, 20 replicates).

**Closed new group.** New players of true skill 3.0 join beside an established league of 24 and play only among themselves for 20 nights. The group's offset from the league was wrong by +0.805 [+0.714, +0.879] in both configs and for groups of 4 and 8: the engine placed them level with the league, 0.805 above their true place. Every member was labelled established (100.0%). Median reliability was 80.0 for a group of 4 (variety count 6, factor 0.50) and 100.0 for a group of 8 (count 14, factor 1.00).

Source: behavior.md, "Fixed-pair padding and closed groups", closed-group table (seed 0, 20 replicates).

**Vulnerable.** Fabricated league matches are unlimited: four accounts can move one of them up by several truth units at no reliability cost, and zero-sum updates mean the cost falls on the colluders' other accounts. Self-reported matches are capped by the repeat rule, but a capped gain is still a gain. A closed group stays at the starting rating whatever its skill, and the established label and reliability do not flag it. The connectivity cap meant for this case (`connectivity_reliability_cap`) is off in both configs ([docs/behavior.md, "Network connectivity"](https://github.com/openpickleball/engine/blob/v1.0.0/docs/behavior.md#network-connectivity); [README, "Known limitations"](https://github.com/openpickleball/engine/blob/v1.0.0/README.md#known-limitations)).

#### Weaknesses

- A fixed partnership shares one rating. At full carrying the weaker partner is overrated by about +0.80 and the stronger underrated by about -0.80 truth units, in both configs (behavior.md, "Carrying").
- `max_doubles_partner_gap` cannot catch carrying, because carried partners hold equal ratings. At 1.0 truth unit it rejected 29.4% (`v1`) and 6.3% (`Config()`) of carrying-league matches and 44.0% and 15.0% in the lopsided league (behavior.md, "Partner-gap filter" and "Lopsided against balanced teams").
- The mean-rating team model overpredicts lopsided teams by up to 0.027 (`v1`) and 0.035 (`Config()`) of the points if teams play nearer their weaker player (behavior.md, "Lopsided against balanced teams").
- Sandbagging is cheap and lasting: a 15-match hard tank lowered an established player by 1.872 (`v1`) and 0.800 (`Config()`), and 60 honest matches later the player was still more than 0.25 low in most replicates. RD and reliability give no signal (behavior.md, "Sandbagging").
- Fabricated league matches count without limit: 40 of them raised the target by +3.493 (`v1`) and +1.820 (`Config()`) with no reliability cost (behavior.md, "Fixed-pair padding and closed groups").
- A closed group is rated level with the main pool whatever its true strength (offset error +0.805), and every member is labelled established (behavior.md, "Fixed-pair padding and closed groups").
- `v1`'s larger K in truth units makes the sandbagging and collusion effects about 1.9 to 2.6 times larger than under `Config()` (the carrying effect on W is 1.3 times larger).

#### Limitations

- The real-data lopsided check is exploratory and in-sample: `v1` was fitted on the same matches, real gaps are rating gaps, and `Config()`'s largest bin had too few matches.
- The truth models are our choice. The weaker-weighted model is an assumption, not a measurement, and the mean model is the engine's own assumption.
- Only one game to 11, side-out style, with rallies independent, was simulated. There are no rally-scoring games, no best of 3 and no tournaments.
- Nothing was tuned on the simulations, and each scenario has one fixed design: one league size, one schedule in which the players under study play every round (everyone does, except in farming, where one non-F player sits out each round), so their RD sits at its floor, and one filter value (1.0 truth unit).
- The connectivity cap was not run (it is off in both configs), and no `win_weight` grid was run on sandbagging.
- Self-reported collusion was confirmed by the colluders themselves, which the engine cannot tell apart from honest confirmation. Collusion was tested only within an established league.
- The closed group of 8 was shuffled at random each round rather than rotated. Its variety count reaches 14 either way.
- Twenty replicates per scenario. The brackets are percentiles across replicates, not confidence intervals.
- Every scenario in the plan was run; none is missing.

## 5. Limitations

These limits apply to every result above.

- **Sample size.** 1601 matches, 189 players and 52 match dates. The initial test part has 285 scored matches on 11 dates, and the default-change test 1018 paired matches. Intervals are wide: most differences between the engine and the stronger baselines include zero.
- **One closed pool.** One club and six leagues in one region, with every player starting at 3.5 and no outside anchor. Ratings mean something only relative to this pool, and the results may not carry over to other clubs.
- **New players.** On matches with a player who had no earlier match, the engine's point estimate is worse than a coin flip (Brier 0.2500): 0.2602 for the tuned configuration, whose interval [0.2238, 0.2944] includes 0.2500 (section 4.6). `v1` seeds no one, so every new player starts at 3.5 with nothing to place them.

- **Rating inflation and deflation.** Every player starts at 3.5, so a pool whose true mean differs from 3.5 sits off its true level as a whole. In the simulations the mean drifts by +0.024 [-0.041, +0.055] points per year, with no consistent direction, and the gap between mean rating and mean true skill at week 104 is -0.285 [-0.362, -0.236]. Real players' skill is unknown, so drift on the real data cannot be measured.
- **Pool isolation.** Pools that share no players are not comparable. Two simulated pools with a true gap of 0.595 were rated level, and 8 cross-pool matches a night for 30 nights closed only part of the gap.

- **Independence.** Matches are assumed independent given the ratings before them, and points and games independent within a match. Players recur across dates, so the date bootstrap is too narrow; the two-way bootstrap allows for it but counts some dependence twice and errs wide. Neither interval is exact.
- **The side-out approximation.** The win probability treats every point as independent with the same probability, which suits rally scoring better than side-out scoring, where only the serving side can score. One fitted `a` (0.476 in `v1`) corrects the overall sharpness only. There are no rally matches, so rally scoring is not validated at all.
- **Order within a night.** The order of play within a league night is the export's row order, which only approximates it. Updates within a night, and the within-day check, depend on it.

- **`v1` shows no measurable gain over `Config()`.** Its point estimates are better (Brier +0.0047, log loss +0.0093), but neither two-way lower bound is above 0. The evaluated configuration is not shown to be better than the defaults it did not replace. `v1` was also fitted on this same pool, and its procedure was chosen and scored on the same folds, which is slightly optimistic.

- **Gaming weaknesses.** A fixed partnership shares one rating, so a carried player is overrated and the carrier underrated, and the partner-gap filter cannot catch it. Fifteen thrown matches lower an established player for most of the next 60. Fabricated league matches count without limit. A closed group is rated level with the main pool whatever its strength. Reliability flags none of these (section 4.8).

- **Not pre-registered.** Each rule was fixed before it ran, but none was locked, and the data include a holdout seen before. Every result here is evidence for a later pre-registered test, not a validation.
- **Pre-specification not public.** Each rule was written down before it ran, in a private record, so a reader cannot check what was fixed when. Nor can a reader check the rerun under engine 1.0.0, whose outputs are not published.
- **Tuned configuration, not `v1`, in most tables.** Sections 4.1 to 4.3 and 4.6 score the configuration tuned on the tuning part (retuned in each fold of section 4.3), and the system-behavior simulations use `Config()`. Only section 4.4, the `v1` fit in section 4.2, the side-out unit check in section 4.5, and the doubles and gaming simulations use the `v1` procedure or set. On real data `v1` was compared only with `Config()`: every Elo, Glicko-2 and TrueSkill result is for the tuned configuration.
- **Names in the private data.** Tournament first names were mapped to full names by hand; a wrong mapping merges two players' histories, and the published reports cannot show it.
- **No external review.** No external reviewer was available. This writeup was not externally reviewed. An independent internal check ran on this draft instead; see "Review".

## 6. Engine maintenance

This section summarises the engine's maintenance rules, from [`docs/parameters.md`](https://github.com/openpickleball/engine/blob/v1.0.0/docs/parameters.md) at `v1.0.0`. The engine's output is not an OPEN rating, and no OPEN rating is live. Ratings are provisional: they come from one club's closed pool, and pools that share no players are not comparable.

- **Parameter sets.** A parameter set is a named group of `Config` values, such as `v1`. Each ships in the package as one file, `src/open_engine/params/v<N>.json`, and is listed in [PARAMETERS.md](https://github.com/openpickleball/engine/blob/v1.0.0/PARAMETERS.md) with its values, data, evidence and fingerprints. A file never holds match data.
- **Identity.** A config has two labels, the package version and the parameter set label, and two fingerprints. `parameter_fingerprint()` hashes every field but the two labels and names the parameter values; `fingerprint()` hashes every field. A label is trusted only when the parameter fingerprint equals the one published for that set ([docs/parameters.md, "Identity"](https://github.com/openpickleball/engine/blob/v1.0.0/docs/parameters.md#identity)). Every shown rating carries the set label, the parameter fingerprint and the release.
- **Loading.** `Config.published("v1")` reads the packaged file and checks its label and recorded fingerprint against the sets the release publishes, so a config loaded this way always has the published values ([docs/parameters.md, "The loader"](https://github.com/openpickleball/engine/blob/v1.0.0/docs/parameters.md#the-loader)).
- **A published set never changes.** Retuning publishes a new label, such as `v2`, and `v1` keeps its values. A change to the `Config()` defaults comes in a minor release and is recorded in the changelog ([docs/parameters.md, "A published set never changes"](https://github.com/openpickleball/engine/blob/v1.0.0/docs/parameters.md#a-published-set-never-changes)).

- **Why `Config()` is not `v1`.** A new set replaces the defaults only when it passes the default-change rule, fixed before the evaluation runs: a lower bound above 0 on the Brier or log-loss improvement, and a Brier lower bound above -0.005. `v1` passed the non-inferiority bar, which is on Brier only, by 0.0006 (lower bound -0.0044 against -0.005), but not the gain bar. Its log-loss lower bound, -0.0106, is below -0.005, so it is not shown to be no worse on log loss. So `Config()` keeps its defaults and `v1` ships as a named set that a caller loads on purpose ([docs/parameters.md, "Why `Config()` is not v1"](https://github.com/openpickleball/engine/blob/v1.0.0/docs/parameters.md#why-config-is-not-v1); section 4.4).

- **Existing ratings when a set changes.** When a new set goes live, authoritative ratings are rebuilt by a full replay of the whole ledger under it; no mode applies a new set to future matches only. A snapshot from another set cannot be resumed. Before a re-rate goes live, the shift by percentile is published. For `v1` none is needed, because no OPEN rating has been published ([docs/parameters.md, "Existing ratings when a set changes"](https://github.com/openpickleball/engine/blob/v1.0.0/docs/parameters.md#existing-ratings-when-a-set-changes)).
- **Evidence each new set needs.** A gate on held-out data, fixed before the run (tuning and scoring on the same matches does not count), and a shift report. Both come from an evaluation outside the engine repository, since it reads real match data ([docs/parameters.md, "Evidence each new set needs"](https://github.com/openpickleball/engine/blob/v1.0.0/docs/parameters.md#evidence-each-new-set-needs)). This writeup reports that evaluation for `v1`.
- **Change caps.** A planned rule gives each parameter a limit on how far it may move in one retune. Neither the limits nor the helper that checks them is written yet ([docs/parameters.md, "Change caps"](https://github.com/openpickleball/engine/blob/v1.0.0/docs/parameters.md#change-caps)).

- **Schedule.** Early on, parameters are retuned at data milestones, such as 5,000, 10,000 and 25,000 matches. Once they settle, they are retuned every six months. A retune ships only when its gate passes ([docs/parameters.md, "Schedule"](https://github.com/openpickleball/engine/blob/v1.0.0/docs/parameters.md#schedule)).

- **Payload changes.** Adding, removing or renaming a `Config` field changes every parameter fingerprint, so a published file would fail to load. How a published set carries across such a release is not decided yet ([docs/parameters.md, "A release that changes the `Config` payload"](https://github.com/openpickleball/engine/blob/v1.0.0/docs/parameters.md#a-release-that-changes-the-config-payload)).

## 7. Related work

OPEN rates pickleball players from match scores. It belongs to the family of online paired-comparison rating systems that started with Elo. This section places it among them. It covers Elo, the Bradley-Terry model underneath Elo, Glicko-2, TrueSkill, and briefly other team and score-margin work. For each it says how OPEN differs and why. The empirical side of the comparison is in the evaluation reports, summarised at the end.

### The OPEN model in brief

The full equations are in the engine's model description, [`docs/model.md`](https://github.com/openpickleball/engine/blob/v1.0.0/docs/model.md). In short:

- A team's rating is the mean of its players' ratings.
- The expected point share of team A is `E = 1 / (1 + 10**((R_B - R_A) / scale))`, with one scale for side-out and one for rally scoring.
- A team's performance `S` is its share of all points in the match. A weight `win_weight` can blend in the win, and its default is 0.
- Each player moves by `K * weight * (S - E)`. `weight` depends on match type, game length, number of games and, for self-reported matches, a repeat factor.
- Each player's `K` grows linearly with their own rating deviation (RD), between `k_min` and `k_max`.
- RD grows in quadrature with days of inactivity, up to `rd_max`. It shrinks by a fixed fraction per unit of match weight, down to `rd_min`. Inactivity never moves a rating.
- Reliability, a 0 to 100 label, combines RD with the number of distinct recent partners and opponents. It changes no rating and no prediction.
- A match's win probability comes from the expected point share, assuming independent points and games, then a one-parameter calibration.
- An imported rating from another system is a starting seed only. It is never converted into an OPEN rating, RD or reliability.

Ratings run from 2.0 to 8.0, and a new player starts at 3.5 ([model.md, "Parameters"](https://github.com/openpickleball/engine/blob/v1.0.0/docs/model.md#parameters)).

### Elo

Elo's 1978 book describes the rating system that chess federations had already adopted. Each player has one number. The expected score of player A against B is a function of the rating difference alone. After a game, each rating moves by `K` times the difference between the actual score (1, 0.5 or 0) and the expected score. Elo's book develops the expectation from a normal curve. The logistic form, `1 / (1 + 10**((R_B - R_A) / 400))`, is the one most implementations use, including the Elo baseline in this evaluation. `K` is a fixed number or a short schedule by player category. There is no explicit measure of how sure the system is about a rating.

**How OPEN differs.** OPEN keeps Elo's structure: one number per player, a logistic expectation in base 10, and an update proportional to actual minus expected. It changes three things.

- *The outcome.* OPEN's actual score is the share of points won, not win or loss. A game won 11-7 gives 18 points of evidence, not one result. A team can win and still lose rating if it won by less than expected ([model.md, "Match performance"](https://github.com/openpickleball/engine/blob/v1.0.0/docs/model.md#match-performance)). This is the main reason OPEN is not plain Elo: pickleball scores carry information that a win/loss system discards.
- *The scale.* OPEN's ratings run from 2.0 to 8.0, and the scale divides the rating gap in the expectation. The scale sets what a gap means as a point share. In parameter set `v1` a gap of 1.0 means the stronger team is expected to win about 90.9% of the points ([model.md, "What a rating means"](https://github.com/openpickleball/engine/blob/v1.0.0/docs/model.md#what-a-rating-means)). Elo's 400 points plays the same role.
- *The K schedule.* Each player's `K` comes from their own RD. A new player moves fast, an established one slowly, and an inactive one faster than before. Because K is per player, a match need not be zero-sum. The engine reports the imbalance rather than hiding it ([model.md, "K"](https://github.com/openpickleball/engine/blob/v1.0.0/docs/model.md#k), "Inactivity" and "The update").

### Bradley-Terry

Bradley and Terry (1952) proposed a model for paired comparisons. Each item `i` has a positive strength `pi_i`, and `i` is preferred to `j` with probability `pi_i / (pi_i + pi_j)`. Writing `pi_i = 10**(R_i / 400)` gives exactly Elo's logistic expectation. Elo's update can then be read as an online, one-match-at-a-time adjustment of Bradley-Terry strengths. Huang, Weng and Lin (2006) generalised the model to comparisons between teams, with a team's strength the sum of its members' strengths.

**How OPEN differs.** OPEN applies the Bradley-Terry form to points, not to matches. With strengths `10**(R / scale)`, OPEN's expected point share is the Bradley-Terry probability that team A wins a single point. The match win probability then follows from the game and match structure, assuming independent points ([model.md, "Win probability"](https://github.com/openpickleball/engine/blob/v1.0.0/docs/model.md#win-probability)). This keeps one model for the rating update and the prediction. The assumption suits rally scoring better than side-out scoring, where only the serving side can score. OPEN has a calibration exponent that can be fitted on data. Its default, 1.0, leaves the probability unchanged.

For teams, OPEN averages ratings rather than summing strengths. On the strength scale, a mean of ratings is a geometric mean of strengths. This is a modelling choice, not an estimate. OPEN has no term for how well two partners play together ([model.md, "Assumptions"](https://github.com/openpickleball/engine/blob/v1.0.0/docs/model.md#assumptions)).

### Glicko and Glicko-2

Glickman (1999) introduced the Glicko system. Each player has a rating and a rating deviation (RD), the standard deviation of a normal belief about their strength. RD grows with time between games and shrinks with each game. A player's update is scaled by their own RD, and the opponent's RD reduces the weight of the result. Glicko-2 (Glickman 2001) adds a volatility per player. Volatility is the expected size of genuine changes in strength, and it is itself updated from the results. Glickman's worked description of Glicko-2 (Glickman 2022) treats games within a rating period as simultaneous. It says the system works best when players average at least 10 to 15 games in a rating period. Glicko-2 is defined for two players. A doubles version needs a team construction. The baseline here uses a composite opponent built from the two opponents' values.

**What each measures.** Glicko-2's RD is a posterior standard deviation in a stated statistical model. In Glickman's predictive formula, which the baseline here uses, both sides' RDs enter through `g(sqrt(phi_A^2 + phi_B^2))`. A more uncertain matchup gets a probability closer to 0.5. OPEN's RD borrows Glicko's shape: it grows in quadrature with inactivity, it sets each player's step size, and a match shrinks it. But OPEN's RD is not derived from a probability model. Its growth rate, shrink rate and range are set parameters, and the defaults are placeholders, not fitted values ([model.md, "Parameters"](https://github.com/openpickleball/engine/blob/v1.0.0/docs/model.md#parameters)). OPEN has no volatility. OPEN's reliability is a further label on top of RD: `100 * certainty * (0.60 + 0.40 * variety)`, where certainty is 1 minus the position of RD (grown to the reference date) in its range, and variety is the count of recent partners plus recent opponents (within 365 days by default, each counted separately) divided by a target (12 by default), capped at 1 ([model.md, "Reliability"](https://github.com/openpickleball/engine/blob/v1.0.0/docs/model.md#reliability)). Reliability does not enter the win probability or any update.

**Why OPEN uses reliability instead.** These are design choices, stated with their costs.

- *Transparency.* Every OPEN update is a few closed-form lines that a reader can recompute by hand. The model description gives a worked example ([model.md, "A worked example"](https://github.com/openpickleball/engine/blob/v1.0.0/docs/model.md#a-worked-example)). Glicko-2's volatility update needs an iterative root search.
- *Explainability to players.* A 0 to 100 score with two named parts (how much recent play, and how varied the partners and opponents) is easier to explain than a deviation on the rating scale. The variety part reflects something RD does not: a player who only ever plays the same three people is less well placed in the pool, however many matches they play. That is the intent. In the evaluation's simulations it reacted only partly. A closed group of four new players, each meeting only the other three, had reliability 80.0, and a group of eight 100.0. Every member was still labelled established, though the group was rated about 0.8 above its true place on the `Config()` scale (`reports/behavior-2026-11/behavior.md`, "Fixed-pair padding and closed groups"). Variety is not aimed at carrying, and a weak player who always partnered a strong one, against varied opponents, kept reliability 100.0 (behavior.md, "Carrying"). The engine's own README lists both as known limitations ([README, "Known limitations"](https://github.com/openpickleball/engine/blob/v1.0.0/README.md#known-limitations); section 4.8, "Carrying" and "Fixed-pair padding and closed groups").
- *Imported ratings.* An imported rating sets a starting state with a set RD, which grows from the capture date, and freezes the player for matches on or before that date. The imported reliability is shown separately and never converted ([model.md, "Starting states"](https://github.com/openpickleball/engine/blob/v1.0.0/docs/model.md#starting-states)). A Glicko-style posterior would have to turn an outside rating and its uncertainty into an RD on OPEN's scale. OPEN avoids that conversion on purpose. OPEN's first release goes further and seeds no player from an external rating: every player starts as a new player ([model.md, "What a rating means"](https://github.com/openpickleball/engine/blob/v1.0.0/docs/model.md#what-a-rating-means)).
- *Per-match updates.* OPEN updates after every match in ledger order, so a player's rating reflects each match as it is recorded. Glicko-2 batches games into rating periods, and the period length is one more setting to choose.

**What OPEN gives up.** OPEN's RD and reliability are not calibrated probabilities. A reliability of 80 does not mean a stated interval holds 80% of the time. OPEN's predictions do not move toward 0.5 for an uncertain player, as Glicko-2's do. Whether these choices cost accuracy is an empirical question. The baseline results below are the first evidence.

### TrueSkill

Herbrich, Minka and Graepel (2006) introduced TrueSkill. Each player's skill is a Gaussian belief with mean `mu` and standard deviation `sigma`. A player's performance in a game is their skill plus Gaussian noise with standard deviation `beta`. A team's performance is the sum of its players' performances. The observed ranking of teams is the evidence, and the beliefs are updated by approximate message passing on a factor graph. A dynamics term `tau` adds variance between games. TrueSkill models teams directly, so it handles doubles without an extra construction. Each partner's update depends on their own `sigma`. Minka, Cleven and Zaykov (2018) extended it as TrueSkill 2, which adds game-specific evidence such as individual statistics and experience.

**How OPEN differs.** TrueSkill and OPEN both rate individuals from team results, and both move uncertain players more. TrueSkill does this through a full Bayesian update. OPEN does it through `K` from RD. TrueSkill's evidence is the ranking of teams: in doubles, who won. OPEN's evidence is the point share. OPEN's team rating is a mean of ratings rather than a sum of performances, and it has no per-player performance noise. In OPEN all players on a team share one surprise `S - E`, scaled by each player's own `K`. In TrueSkill the split between partners comes from their relative uncertainty. Neither model has a term for a pair. In OPEN, two new players who always partner start at the same rating and RD, get the same updates, and so always hold the same rating, so the engine cannot tell who carries whom. In the evaluation's simulations, a weaker player who always partnered a stronger one (true gap 1.6) ended about 0.8 too high and the stronger one about 0.8 too low on the `Config()` scale, under both `v1` and `Config()`, and reliability gave no warning (`reports/behavior-2026-11/behavior.md`, "Carrying"; [README, "Known limitations"](https://github.com/openpickleball/engine/blob/v1.0.0/README.md#known-limitations); section 4.8, "Carrying"). TrueSkill was not run on that scenario. We expect any model whose only evidence is team results to meet the same limit for a pair that never plays apart, since the results say how strong the pair is, not how it splits. This is an argument, not a tested result. OPEN chose the simpler update for the same transparency reasons as above. Explaining why a partner moved more is one sentence in OPEN ("their RD is higher, so their K is higher"). In TrueSkill it needs the factor graph.

### Other team and score-margin work

**Team ratings.** Beyond the generalised Bradley-Terry model above, Weng and Lin (2011) derived simple analytic Bayesian updates for games with several teams and players. They report results competitive with TrueSkill with much shorter running time. OPEN shares the aim of closed-form updates, but uses point share rather than ranks.

**Score margin.** Several Elo extensions use the margin of victory. Hvattum and Arntzen (2010) let `K` grow with the goal difference in association football. Kovalchik (2020) proposed four ways to extend Elo to margin of victory and compared them with standard Elo on men's tennis. Some extensions, like Hvattum and Arntzen's, keep win/loss as the outcome and use the margin to scale the step. OPEN instead makes the point share the outcome itself. The expected and actual score are then on the same scale, and the same expectation gives the win probability.

### Empirical comparison in this evaluation

The evaluation ran Elo, Glicko-2 and TrueSkill as baselines on the same matches as the engine. Each had a tuned row, tuned on the tuning part only, and a default row. All baselines used win or loss only. Only the tuned engine row was calibrated. The results are in `reports/initial-2026-11/technical-report.md`, section 1, and `reports/behavior-2026-11/behavior.md`.

Two cautions apply to all of it. First, it is an initial evaluation, not pre-registered. Second, these runs used engine 0.3.0.dev0 at commit `e62468c`, before 1.0.0. The prediction results used the tuned configuration in the report. The simulations used every system's defaults, including the engine's `Config()`. Rerunning the commands behind the initial and behavior reports on the same data under engine 1.0.0 changed no number in either; only provenance lines (engine version and commit, config fingerprints, evaluation commit) differ. The committed reports were not regenerated, so they still name `e62468c`. Neither the tuned configuration nor `Config()` is the published parameter set `v1`, so the prediction and ranking-recovery results below are not results for `v1`.

- **Prediction on held-out matches** (technical report, section 1; 285 test matches on 11 match dates where every player had at least 5 earlier matches, 95% intervals resampling match dates). Brier scores: tuned engine 0.2224 [0.2127, 0.2312], tuned Glicko-2 0.2294 [0.2204, 0.2355], tuned TrueSkill 0.2328 [0.2139, 0.2454]. The paired Brier difference, baseline minus engine, is +0.0070 [-0.0002, +0.0130] for tuned Glicko-2 and +0.0104 [-0.0075, +0.0203] for tuned TrueSkill. Both point estimates favour the engine, and both intervals include zero. The gaps over tuned Glicko-2 and tuned TrueSkill could be chance. Only the tuned engine row had a fitted win calibration. Before calibration its Brier score is 0.2351 [0.2166, 0.2527], behind both. The default-config engine, which is not calibrated, scores 0.2253 [0.2101, 0.2425]. The tuned configuration (not `v1`) was ahead of tuned Elo by +0.0175 [+0.0095, +0.0259], on a date interval that is likely too narrow.
- **Ranking recovery on simulated leagues** (behavior report). In pools of 16, 32 and 64 players, all four systems reach a median Spearman correlation of 0.8 with true skill within 12 matches per player of one another (each weekly night is 4 matches). The engine reaches 0.9 earliest in each pool, by one night in the pool of 32. These first-reach figures have no intervals. The simulated games are generated from OPEN's own point-share model, and the baselines see only who won. So the simulation favours the engine by construction. It shows the engine recovers rankings under its own assumptions, not that it would on real play.

A fair reading: on this one club's data, the tuned configuration (not `v1`) was ahead of tuned Elo by +0.0175 [+0.0095, +0.0259], on a date interval that is likely too narrow, and by only +0.0011 [-0.0130, +0.0168] in rolling fold 2. This comparison does not separate point share from the engine's other differences (per-player K, a fitted scale, and calibration). Against tuned Glicko-2 and tuned TrueSkill, the engine's point estimates are better on the main test, and so are the uncalibrated default-config engine's. The paired intervals against the tuned engine include zero, and in rolling fold 2 tuned Glicko-2 beat the engine by 0.0147 [-0.0270, -0.0026] (section 4.3), so these data do not separate them. The case for OPEN's reliability design rests on transparency and the handling of imported ratings, not on a measured accuracy advantage over Glicko-2 or TrueSkill.

## 8. Reproducibility appendix

**Code and data.** The reports were produced by the evaluation's own code, which reads the private match data. Neither the code nor the data is published here. The processed data stay private, even with names removed.

**Seeds and fixed inputs.** The bootstrap uses 2000 resamples with seed 20261114. The behavior simulations use seed 0 and 20 replicates. The fold dates of the rolling-origin layout were fixed before running. The `v1-gap` figures read no data.

**Engine pin.** The evaluation installs the engine from PyPI:

- Package: `openpickleball-engine==1.0.0`, tag `v1.0.0`, commit `be2a0ff` (`be2a0ff59065f7273b5dc13e395e5ec25ee01d3c`).
- Wheel: `openpickleball_engine-1.0.0-py3-none-any.whl`, sha256 `403201b8d2e543726e322adef2d96dadeb58d067fd703bb7f02f71766133c9c3`.
- Parameter set `v1`, parameter fingerprint `f9149f3c38a01efc`.

The reports, except `reports/v1-gap-2026-11` (engine 1.0.0), were generated on engine 0.3.0.dev0 at commit `e62468c`. Rerunning their commands on the same data under engine 1.0.0 changed no fit, fold, score, difference or verdict; only provenance lines differ (section 1, "The evaluated configuration"). The rerun outputs are not published, so a reader cannot check this.

**Traceability.** Every number names the report in `reports/` it comes from. One statement has no published report: the rerun under engine 1.0.0 (below). Numbers inside code (hashes, identifiers) and version numbers are not results. The numbers that come from elsewhere: "six" leagues (the count of league exports; no report counts them), the retune milestones 5,000, 10,000 and 25,000 matches (engine `docs/parameters.md`, "Schedule"), 50% as even odds (section 1, "As a win probability"), and 2026-11-24, the date of the internal check ("Review"). Publication years and other bibliographic numbers in the related work are citations, not results. A few derived figures (a rounding, a ratio or a unit conversion) are arithmetic on numbers in the named report.

The report folders are `reports/<name>-2026-11/`: `initial`, `declared-scale`, `ridge-check`, `side-out-unit`, `v1-fit`, `v1-gap` and `behavior`.

## Review

This writeup was not externally reviewed: no external reviewer was available. On 2026-11-24 two internal checks ran, each by a fresh reviewer with no part in the draft, reporting errors only: one on the private draft, and one on this public version, against the published reports. They are internal checks, not external reviews. Their findings on numbers, overclaims, fairness, references, privacy and candour were fixed before this version, except where the limitations above say a statement cannot be checked.

## References

### Literature

- Bradley, R. A., and Terry, M. E. (1952). Rank analysis of incomplete block designs: I. The method of paired comparisons. *Biometrika*, 39(3/4), 324-345. doi:10.2307/2334029.
- Elo, A. E. (1978). *The Rating of Chessplayers, Past and Present*. Arco Publishing, New York.
- Glickman, M. E. (1999). Parameter estimation in large dynamic paired comparison experiments. *Journal of the Royal Statistical Society, Series C (Applied Statistics)*, 48(3), 377-394. doi:10.1111/1467-9876.00159.
- Glickman, M. E. (2001). Dynamic paired comparison models with stochastic variances. *Journal of Applied Statistics*, 28(6), 673-689. doi:10.1080/02664760120059219.
- Glickman, M. E. (2022). Example of the Glicko-2 system. Boston University, dated March 22, 2022. http://www.glicko.net/glicko/glicko2.pdf.
- Herbrich, R., Minka, T., and Graepel, T. (2006). TrueSkill: A Bayesian skill rating system. *Advances in Neural Information Processing Systems 19* (NIPS 2006), 569-576.
- Huang, T.-K., Weng, R. C., and Lin, C.-J. (2006). Generalized Bradley-Terry models and multi-class probability estimates. *Journal of Machine Learning Research*, 7, 85-115.
- Hvattum, L. M., and Arntzen, H. (2010). Using ELO ratings for match result prediction in association football. *International Journal of Forecasting*, 26(3), 460-470. doi:10.1016/j.ijforecast.2009.10.002.
- Kovalchik, S. (2020). Extension of the Elo rating system to margin of victory. *International Journal of Forecasting*, 36(4), 1329-1341. doi:10.1016/j.ijforecast.2020.01.006.
- Minka, T., Cleven, R., and Zaykov, Y. (2018). TrueSkill 2: An improved Bayesian skill rating system. Technical report MSR-TR-2018-8, Microsoft Research.
- Weng, R. C., and Lin, C.-J. (2011). A Bayesian approximation method for online ranking. *Journal of Machine Learning Research*, 12, 267-300.

### Reports

- `reports/initial-2026-11/`: the initial evaluation (`technical-report.md`, `summary.md`, `rolling-splits.md` and figures).
- `reports/v1-fit-2026-11/v1-fit.md`: the `v1` fit and the default-change test.
- `reports/v1-gap-2026-11/v1-gap.md`: what a rating gap means under `v1`.
- `reports/declared-scale-2026-11/declared-scale.md`, `reports/ridge-check-2026-11/ridge-check.md` and `reports/side-out-unit-2026-11/side-out-unit.md`: the scale and K checks.
- `reports/behavior-2026-11/behavior.md`, with its CSV and figure files: system behavior, doubles and gaming on simulated leagues.

### Engine

- OPEN rating engine (2026). `openpickleball-engine` version 1.0.0, tag `v1.0.0`, commit `be2a0ff`. GitHub release: https://github.com/openpickleball/engine/releases/tag/v1.0.0. PyPI: https://pypi.org/project/openpickleball-engine/1.0.0/. Cited files: [`docs/model.md`](https://github.com/openpickleball/engine/blob/v1.0.0/docs/model.md), [`docs/parameters.md`](https://github.com/openpickleball/engine/blob/v1.0.0/docs/parameters.md), [`docs/behavior.md`](https://github.com/openpickleball/engine/blob/v1.0.0/docs/behavior.md), [`PARAMETERS.md`](https://github.com/openpickleball/engine/blob/v1.0.0/PARAMETERS.md), [`README.md`](https://github.com/openpickleball/engine/blob/v1.0.0/README.md), [`src/open_engine/params/v1.json`](https://github.com/openpickleball/engine/blob/v1.0.0/src/open_engine/params/v1.json) and [`src/open_engine/_config.py`](https://github.com/openpickleball/engine/blob/v1.0.0/src/open_engine/_config.py).
