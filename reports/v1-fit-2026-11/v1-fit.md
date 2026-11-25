# Final V1 fit and the default-change test

**Status: part of the initial evaluation, not pre-registered.** The procedure, the freeze date and the rule were written down on 2026-11-22, before any part of it ran on real data. It applies the side-out unit rule (`side-out-unit-2026-11`), with no seeding, on rolling-origin folds whose dates were fixed before any run, with a two-way bootstrap that resamples match dates and players. It feeds [openpickleball/engine#48](https://github.com/openpickleball/engine/issues/48).

The folds estimate how well the procedure predicts; they choose no number. The V1 numbers come from the final fit alone. The test-part matches had been seen in aggregate before, and selecting a procedure and quoting its score on the same folds is slightly optimistic.

## Provenance

- Engine: `openpickleball-engine` 0.3.0.dev0 at commit `e62468c508126ed46b5a26a82f175ba17dd78371`.
- Data version: `28bcab9de3af`.

## Data

| Quantity | Value |
| --- | --- |
| Matches used (on or before the freeze date) | 1601 |
| Matches excluded as after the freeze date | 0 |
| Doubles matches | 1601 |
| Side-out matches | 1601 |
| Rally matches | 0 |
| League matches | 1283 |
| Tournament matches | 318 |
| Distinct match dates | 52 |
| Distinct players | 189 |
| First match date | 2025-08-10 |
| Last match date | 2026-11-20 |
| Freeze date | 2026-11-20 |
| Fit-window start (final fit) | 2026-04-12 |
| Data version | 28bcab9de3af |

## The procedure

- The strict tune once on every match on or before 2026-11-20 (`tuning_end` = the freeze date, burn-in 5, fit window from the midpoint start 2026-04-12), searching the side-out scale over [0.5, 8.0] at default K.
- The side-out unit rule: side-out scale 1.0, `k_min` and `k_max` times 1.0 / s_fit, bounds kept at 2.0 and 8.0, side-out `a` as fitted.
- Rally scale = the declared side-out scale (1.0) and rally `a` = 1.0. No seeding. Every other field at `Config()`'s default.

## The final fit

### Side-out scale

- s_fit = 1.112; at its search bound: no.
- Search range [0.5, 8.0]; matches scored in the fit window: 667.

Profile: point-share MSE at every scale the search tried.

| Scale | MSE |
| --- | --- |
| 0.5 | 0.042038497729985004 |
| 0.909249 | 0.04002280606041979 |
| 1.035714 | 0.039944450133583206 |
| 1.065569 | 0.03993792929010495 |
| 1.102471 | 0.03993429790224887 |
| 1.107855 | 0.03993426921304348 |
| 1.111182 | 0.03993412876086957 |
| 1.112 | 0.03993418506326837 |
| 1.116566 | 0.03993418304707646 |
| 1.125277 | 0.03993429039895052 |
| 1.139373 | 0.039935072637331336 |
| 1.162179 | 0.039937447974662664 |
| 1.221888 | 0.0399489988898051 |
| 1.318499 | 0.03998084378365817 |
| 1.571429 | 0.040104371836131934 |
| 2.107143 | 0.04040835021724138 |
| 2.642857 | 0.040685933034332836 |
| 2.9 | 0.04080565064182908 |
| 3.178571 | 0.040926165134182915 |
| 3.714286 | 0.041135085996551725 |
| 4.25 | 0.041318460559970016 |
| 4.785714 | 0.04148117396566717 |
| 5.321429 | 0.041626683379160416 |
| 5.857143 | 0.041757493262668664 |
| 6.392857 | 0.04187599482923538 |
| 6.928571 | 0.04198355043778111 |
| 7.464286 | 0.042081877420989504 |
| 8.0 | 0.042171842768215895 |

### Side-out win calibration

- a = 0.476; at its search bound: no; predictions used: 667.
- Fit-window Brier at the fitted a: 0.23023298364793812; at a = 1: 0.2408311593261697.

Profile: fit-window Brier at a = 0.20 to 3.00 in 0.05 steps (the engine's default search range), on the same predictions the fit used.

| a | Brier |
| --- | --- |
| 0.20 | 0.236061715182565 |
| 0.25 | 0.23398681834100635 |
| 0.30 | 0.23241518649311244 |
| 0.35 | 0.23130273847897934 |
| 0.40 | 0.23060383266277212 |
| 0.45 | 0.23027348406497222 |
| 0.50 | 0.2302687096015532 |
| 0.55 | 0.2305492825969005 |
| 0.60 | 0.23107810675012108 |
| 0.65 | 0.23182135035423565 |
| 0.70 | 0.23274843056443636 |
| 0.75 | 0.2338319042281293 |
| 0.80 | 0.23504730130033685 |
| 0.85 | 0.2363729245265636 |
| 0.90 | 0.23778963155208555 |
| 0.95 | 0.23928061081778754 |
| 1.00 | 0.2408311593261697 |
| 1.05 | 0.24242846796720494 |
| 1.10 | 0.2440614182611476 |
| 1.15 | 0.24572039294601766 |
| 1.20 | 0.24739710172790622 |
| 1.25 | 0.24908442266919348 |
| 1.30 | 0.25077625907059403 |
| 1.35 | 0.2524674112688621 |
| 1.40 | 0.25415346248687704 |
| 1.45 | 0.2558306777038851 |
| 1.50 | 0.2574959144322332 |
| 1.55 | 0.2591465442689526 |
| 1.60 | 0.2607803841165953 |
| 1.65 | 0.2623956360225892 |
| 1.70 | 0.26399083465859 |
| 1.75 | 0.2655648015424394 |
| 1.80 | 0.2671166051894414 |
| 1.85 | 0.26864552646269596 |
| 1.90 | 0.2701510284715623 |
| 1.95 | 0.27163273044141006 |
| 2.00 | 0.2730903850458288 |
| 2.05 | 0.2745238587541217 |
| 2.10 | 0.27593311480225063 |
| 2.15 | 0.27731819844469396 |
| 2.20 | 0.27867922418833097 |
| 2.25 | 0.2800163647479303 |
| 2.30 | 0.28132984149659496 |
| 2.35 | 0.28261991621407895 |
| 2.40 | 0.2838868839617077 |
| 2.45 | 0.2851310669351396 |
| 2.50 | 0.2863528091657938 |
| 2.55 | 0.2875524719588017 |
| 2.60 | 0.2887304299701311 |
| 2.65 | 0.28988706783837814 |
| 2.70 | 0.29102277729786635 |
| 2.75 | 0.2921379547093582 |
| 2.80 | 0.2932329989530741 |
| 2.85 | 0.2943083096359816 |
| 2.90 | 0.29536428557162914 |
| 2.95 | 0.2964013234962678 |
| 3.00 | 0.2974198169897517 |

## The V1 config

- V1 fingerprint: `22b53a37587d6881`.
- `Config()` fingerprint: `17f2981186edf96c`.
- Engine `openpickleball-engine` 0.3.0.dev0 at commit `e62468c508126ed46b5a26a82f175ba17dd78371`.
- The fingerprint includes `version` ('0.3.0.dev0'), so the same parameters under another package version get another fingerprint.

Every `Config` field, mapping fields per key, with its V1 value at full precision.

| Field | V1 value | Config() value | Status | Reason |
| --- | --- | --- | --- | --- |
| `version` | "0.3.0.dev0" | "0.3.0.dev0" | label | The package version. It changes no rating but is in the fingerprint, so the fingerprint moves with every release |
| `min_rating` | 2.0 | 2.0 | declared | Level and range, policy |
| `max_rating` | 8.0 | 8.0 | declared | Level and range, policy |
| `default_rating` | 3.5 | 3.5 | declared | Level and range, policy |
| `scale_by_scoring[rally]` | 1.0 | 2.9 | declared | Declared equal to the side-out scale: no rally matches, and the same unit keeps one rating meaning. Unvalidated |
| `scale_by_scoring[side_out]` | 1.0 | 2.9 | declared | Declared 1.0: a unit, not identifiable |
| `win_calibration_by_scoring[rally]` | 1.0 | 1.0 | declared | Declared 1.0: no rally matches. Unvalidated |
| `win_calibration_by_scoring[side_out]` | 0.476 | 1.0 | fitted | Fitted by fit_win_calibration; identified in every fit (engine #150 option 2) |
| `k_min` | 0.1079136690647482 | 0.12 | fitted | Default K times 1.0 / s_fit from fit_scale at default K (ratio 1:5 kept): the one identified step-size quantity |
| `k_max` | 0.539568345323741 | 0.6 | fitted | Default K times 1.0 / s_fit from fit_scale at default K (ratio 1:5 kept): the one identified step-size quantity |
| `rd_min` | 0.1 | 0.1 | default | Not identified: they shape the K schedule, and fitting them on about 1,600 matches of mostly regular players would fit noise. Revisit at the first milestone retune |
| `rd_max` | 0.8 | 0.8 | default | Not identified: they shape the K schedule, and fitting them on about 1,600 matches of mostly regular players would fit noise. Revisit at the first milestone retune |
| `new_player_rd` | 0.8 | 0.8 | default | Not identified: they shape the K schedule, and fitting them on about 1,600 matches of mostly regular players would fit noise. Revisit at the first milestone retune |
| `external_seed_rd` | 0.42 | 0.42 | default | Default 0.42, a declared choice; it only matters for seeding, and `v1` seeds no one |
| `category_seed_rd` | 0.5 | 0.5 | default | No category matches in the data |
| `rd_shrink_per_match` | 0.08 | 0.08 | default | Not identified: they shape the K schedule, and fitting them on about 1,600 matches of mostly regular players would fit noise. Revisit at the first milestone retune |
| `rd_growth_per_day` | 0.012 | 0.012 | default | Not identified: they shape the K schedule, and fitting them on about 1,600 matches of mostly regular players would fit noise. Revisit at the first milestone retune |
| `win_weight` | 0.0 | 0.0 | declared | Declared 0.0; not tested here |
| `match_weights[league]` | 0.9 | 0.9 | declared | Policy for verification level; tournament against league could be tested later |
| `match_weights[rec_verified]` | 0.75 | 0.75 | declared | Policy for verification level; tournament against league could be tested later |
| `match_weights[self_reported]` | 0.4 | 0.4 | declared | Policy for verification level; tournament against league could be tested later |
| `match_weights[tournament]` | 1.0 | 1.0 | declared | Policy for verification level; tournament against league could be tested later |
| `allowed_game_formats` | [["rally", 11], ["rally", 15], ["rally", 21], ["side_out", 11], ["side_out", 15], ["side_out", 21]] | [["rally", 11], ["rally", 15], ["rally", 21], ["side_out", 11], ["side_out", 15], ["side_out", 21]] | declared | Integrity and schedule, not accuracy |
| `game_length_weights[11]` | 1.0 | 1.0 | default | Not identified: every game is to 11, and best of 3 occurs only in tournaments, so it is confounded with match type |
| `game_length_weights[15]` | 1.1 | 1.1 | default | Not identified: every game is to 11, and best of 3 occurs only in tournaments, so it is confounded with match type |
| `game_length_weights[21]` | 1.2 | 1.2 | default | Not identified: every game is to 11, and best of 3 occurs only in tournaments, so it is confounded with match type |
| `game_count_bonus` | 0.12 | 0.12 | default | Not identified: every game is to 11, and best of 3 occurs only in tournaments, so it is confounded with match type |
| `max_information_multiplier` | 1.3 | 1.3 | default | Not identified: every game is to 11, and best of 3 occurs only in tournaments, so it is confounded with match type |
| `replay_interval_days` | 7 | 7 | declared | Integrity and schedule, not accuracy |
| `diversity_window_days` | 365 | 365 | label | Changes no rating; default kept (the connectivity cap stays off, engine #86) |
| `diversity_target` | 12 | 12 | label | Changes no rating; default kept (the connectivity cap stays off, engine #86) |
| `self_reported_requires_both_teams` | true | true | declared | Integrity and schedule, not accuracy |
| `self_reported_repeat_window_days` | 90 | 90 | declared | Integrity and schedule, not accuracy |
| `self_reported_repeat_factor` | 0.5 | 0.5 | declared | Integrity and schedule, not accuracy |
| `self_reported_max_repeats` | 4 | 4 | declared | Integrity and schedule, not accuracy |
| `reject_content_duplicates` | false | false | declared | Integrity and schedule, not accuracy |
| `import_reliability_carry_matches` | 10 | 10 | label | Changes no rating; default kept (the connectivity cap stays off, engine #86) |
| `import_reliability_max_age_days` | 365 | 365 | label | Changes no rating; default kept (the connectivity cap stays off, engine #86) |
| `established_reliability_threshold` | 60.0 | 60.0 | label | Changes no rating; default kept (the connectivity cap stays off, engine #86) |
| `established_min_verified_matches` | 3 | 3 | label | Changes no rating; default kept (the connectivity cap stays off, engine #86) |
| `connectivity_reliability_cap` | null | null | label | Changes no rating; default kept (the connectivity cap stays off, engine #86) |
| `connectivity_min_shared_contacts` | 4 | 4 | label | Changes no rating; default kept (the connectivity cap stays off, engine #86) |
| `connectivity_min_contact_days` | 3 | 3 | label | Changes no rating; default kept (the connectivity cap stays off, engine #86) |
| `connectivity_min_outside_players` | 8 | 8 | label | Changes no rating; default kept (the connectivity cap stays off, engine #86) |
| `category_feeds_parent` | true | true | default | No category matches in the data |
| `max_singles_rating_gap` | null | null | declared | Integrity and schedule, not accuracy |
| `max_doubles_partner_gap` | null | null | declared | Integrity and schedule, not accuracy |

Fields that differ from `Config()`: scale_by_scoring, win_calibration_by_scoring, k_min, k_max.

## Bounds and distribution

A match touches a bound if a player's doubles rating is within 1e-9 of the floor or the ceiling before or after it, on a live replay of every match through the freeze date.

| Engine | Matches at the floor | Matches at the ceiling | Matches at either |
| --- | --- | --- | --- |
| V1 | 0 | 0 | 0 |
| Config() (reference) | 0 | 0 | 0 |

V1 doubles ratings on 2026-11-20 for players with at least 10 matches (inclusive quartiles).

| Players | Lowest | Q1 | Median | Q3 | Highest | IQR |
| --- | --- | --- | --- | --- | --- | --- |
| 122 | 2.619 | 3.354 | 3.521 | 3.686 | 4.078 | 0.332 |

## The default-change test

Arms: `Config()` at engine `e62468c` (fixed) against V1 (the procedure above, run on each fold's training part only, `tuning_end` = the fold's training end). Both predict each block scheduled, with no burn-in, as calibrated win probability. A match is pooled only when both arms scored it; a fold whose V1 tune refused is not run for either arm.

Folds: 6, of which 6 ran.

| Fold | Training ends | Block | Block matches | Paired | Status | s_fit | a | k_min | k_max | s_fit at bound | a at bound |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| 1 | 2026-04-12 | 2026-04-19 to 2026-05-24 | 150 | 150 | ran | 0.671 | 0.403 | 0.17883755588673622 | 0.8941877794336811 | no | no |
| 2 | 2026-05-24 | 2026-05-31 to 2026-07-19 | 150 | 150 | ran | 0.814 | 0.432 | 0.14742014742014745 | 0.7371007371007372 | no | no |
| 3 | 2026-07-19 | 2026-07-26 to 2026-08-21 | 174 | 174 | ran | 0.885 | 0.468 | 0.13559322033898305 | 0.6779661016949152 | no | no |
| 4 | 2026-08-21 | 2026-08-23 to 2026-10-11 | 211 | 211 | ran | 0.951 | 0.393 | 0.12618296529968456 | 0.6309148264984228 | no | no |
| 5 | 2026-10-11 | 2026-10-18 to 2026-11-06 | 231 | 231 | ran | 0.951 | 0.427 | 0.12618296529968456 | 0.6309148264984228 | no | no |
| 6 | 2026-11-06 | 2026-11-08 to 2026-11-20 | 102 | 102 | ran | 1.079 | 0.469 | 0.11121408711770157 | 0.5560704355885079 | no | no |

Pooled scores per arm.

| Arm | Matches | Brier | Log loss | Accuracy |
| --- | --- | --- | --- | --- |
| Config() | 1018 | 0.2379 | 0.6689 | 0.6007 |
| V1 | 1018 | 0.2332 | 0.6596 | 0.6125 |

`Config()` minus V1, pooled; positive favours V1. Intervals are 95%. The two-way bootstrap decides; the date bootstrap is information only and too narrow, because players recur across dates.

| Metric | Estimate | Two-way bootstrap (decision) | Date bootstrap (too narrow) |
| --- | --- | --- | --- |
| Brier | +0.0047 | [-0.0044, +0.0139] | [+0.0005, +0.0090] |
| Log loss | +0.0093 | [-0.0106, +0.0295] | [-0.0001, +0.0186] |

Note: the two-way bootstrap interval resamples match dates and players independently. It is conservative: it counts some shared dependence twice, so it can be wider than needed.

## Decision

The rule, verbatim as fixed: "Apply the default-change rule, fixed 2026-11-22 before any run. The baseline is `Config()` at engine `e62468c`. V1 replaces the defaults if the pooled paired difference on the rolling-origin layout has a 95% lower bound above 0 on Brier or on log loss, and in either case its Brier lower bound is above -0.005. If V1 fails the -0.005 bar, keep the placeholders and find out why."

Replaces the defaults iff (Brier lower bound > 0 or log-loss lower bound > 0) and Brier lower bound > -0.005. A bound exactly at 0 or exactly at -0.005 does not pass.

**Under the rule, keep the placeholders (the `Config()` defaults are not replaced): the Brier lower bound -0.0044 is above -0.005, but neither lower bound is above 0 (Brier -0.0044, log loss -0.0106).**

The -0.005 non-inferiority bar passed (Brier lower bound -0.0044 is above -0.005); the gain bar did not (neither two-way lower bound, Brier -0.0044 or log loss -0.0106, is above 0). So under the rule the `Config()` defaults are not replaced. V1 ships as the named parameter set `v1` ([openpickleball/engine#48](https://github.com/openpickleball/engine/issues/48)), not as the default. Point estimates favour V1 (Brier +0.0047, log loss +0.0093). The date bootstrap (too narrow) excludes 0 for Brier ([+0.0005, +0.0090]), which suggests limited power. Revisit the defaults with the same rule on new data.

## Not done here

Writing `params/v1.json` or `PARAMETERS.md` ([openpickleball/engine#48](https://github.com/openpickleball/engine/issues/48)), changing any engine default and rally validation (no rally data).
