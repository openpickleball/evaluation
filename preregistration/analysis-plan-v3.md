# Analysis plan v3: pre-registered test of engine 1.0.0 and parameter set v1 on future play

Status: **DRAFT. Not locked.** The owner confirmed every item in "Owner decisions" below on 2026-11-23 ([issue #26](https://github.com/GeoBradDev/open-engine-evaluation/issues/26#issuecomment-7119500806)). Nothing in this file is fixed until the owner fills in the sign-off and runs `v3-lock` (section 12). The plan is not externally reviewed (owner decision 14, as amended on 2026-11-24); it is made public at the lock, so anyone can critique it against its fixed sha256. Locking and any registry submission (for example OSF) are owner steps; neither has happened.

Issue: [#26](https://github.com/GeoBradDev/open-engine-evaluation/issues/26). Roadmap: `docs/plans/2026-11-22-engine-v1-roadmap.md`, section 4.6 and owner decision D8. Power simulation: `reports/v3-power-2026-11/v3-power.md`.

**Internal check (2026-11-24).** A fresh reviewer with no part in the draft checked this plan and its commands for errors only (owner decision 14, as amended). It reported 2 blockers, 6 should-fix findings and some nits. All were addressed in the PR that adds this note. It is an internal check, not an external review.

## Owner decisions

Each item below was a choice this draft made for the owner to confirm or change before the lock. On 2026-11-23 the owner accepted every recommendation ([issue #26](https://github.com/GeoBradDev/open-engine-evaluation/issues/26#issuecomment-7119500806)). Each item names the section it affects, the recommendation as drafted, and what the plan now says. On 2026-11-24 the owner replaced item 14 ([issue #26](https://github.com/GeoBradDev/open-engine-evaluation/issues/26#issuecomment-7131147762)). Nothing else in the plan is meant to need a decision. The owner steps left are the sign-off and running `v3-lock` (section 12, "Lock-day procedure").

1. **Lock date and test start** (sections 2 and 13). The test window starts on the first match date after the lock's UTC date. The power simulation assumed a lock on 2026-12-01. *Recommendation:* lock as soon as items 2 to 14 are settled and the pin of item 3 is merged; every week of delay turns a week of play into history instead of test. **Confirmed 2026-11-23.** The lock date stays open: lock once #11's pin and the PR that adds section 12's commands are merged, the internal check's fixes are merged, and the owner has signed off (item 14 as amended on 2026-11-24).
2. **Which V1 is tested** (section 3). This draft tests the published parameter set `Config.published("v1")` exactly as engine 1.0.0 ships it (fitted through 2026-11-20, #40), with no retune. Issue #26's scope said "tune on every match through the lock"; that would test a config nobody ships. *Recommendation:* test the published set as is. **Confirmed 2026-11-23:** V1 is `Config.published("v1")` as engine 1.0.0 ships it, with no retune (section 3).
3. **Engine pin** (section "Engine"). The lock waits until the evaluation pins `openpickleball-engine==1.0.0` from PyPI (#11's pin PR), and the pin then does not move until the last `score` run (#21's rule). *Recommendation:* confirm, and record the wheel sha256 in the sign-off. **Confirmed 2026-11-23:** the lock waits for the 1.0.0 pin, the pin does not move until the last `v3-score` run, and the wheel sha256 goes in the sign-off. `v3-lock` refuses any other engine (section 12).
4. **Stopping rule** (section 2). D8 as decided: the first match date on which burn-in-eligible matches reach 400, or 2027-06-30, whichever is first. The power simulation (section 11) puts the stopping date at about late April 2027 for a lock on 2026-12-01, and shows that the rule gives almost no power to the default-change gain test at #40's effect size. *Recommendation:* keep D8 (it is the owner's decision, and a longer window delays the published claim), and state the power in the plan as section 11 does. A larger target would not rescue the gain test: if the interval narrows as one over the square root of the number of matches, an even chance of passing at #40's Brier gain needs about 3,500 matches (416 x (0.0136 / 0.0047)^2, about 3,480). **Confirmed 2026-11-23:** D8 stays, and the power is stated as section 11 states it.
5. **The non-inferiority margins against the two-way interval** (section 9, criteria 1 and 3). At about 400 matches the two-way interval of a Brier difference is about +/-0.014 wide, so a lower bound above -0.005 needs V1's point estimate to be ahead by about +0.009. The -0.005 bars therefore behave like "V1 is clearly better", not "V1 is not worse": if V1 and tuned Elo were truly equal, criterion 1 would almost never pass. It does pass reliably if V1 keeps the tuned engine's lead over tuned Elo in the initial evaluation (+0.0175). *Recommendation:* keep -0.005 (it is v1's and v2's bar, and changing it after seeing the power invites the charge of moving the goalposts), and state in the plan what it takes to pass. The alternatives are a wider margin, or the date bootstrap as the decision interval (narrower, but #28 found it too narrow). **Final choice: owner.** **Confirmed 2026-11-23:** the -0.005 margins stay with the two-way interval, and section 9 states what it takes to pass (V1 about +0.009 ahead at about 400 matches).
6. **The favourite-side bias bar** (section 9, criterion 2). The roadmap's two options: (1) restate it as "interval includes 0 and point estimate within 0.015", or (2) keep "point estimate within 0.015" and say in the plan that it is underpowered. *Recommendation:* option 2. At this sample size the two options decide every sample the same way (section 11), so option 1 adds a condition that changes nothing now and would only make the bar stricter on a larger sample. **Final choice: owner.** **Confirmed 2026-11-23:** option 2. Section 9 keeps "point estimate within 0.015" and says that it is underpowered.
7. **Burn-in for the default-change test** (section 9, criterion 3). #40 used no burn-in. This draft scores criterion 3 on the same burn-in-5 matches as the other criteria and as D8's count, and shows the no-burn-in result for information. *Recommendation:* burn-in 5, so one set of matches decides everything and the count that stops the test is the count that is scored. **Confirmed 2026-11-23:** burn-in 5, with the no-burn-in result shown for information.
8. **What criterion 3 decides** (section 9). It is #40's rule on new data: whether a later engine release may propose V1 as `Config()`'s defaults. Changing a default changes every rating computed with `Config()`, so the engine maintainers decide which release carries it. *Recommendation:* confirm that the v3 result is evidence for that decision, not the decision itself. **Confirmed 2026-11-23:** the v3 result is evidence for a later engine decision on `Config()`'s defaults, not the decision itself.
9. **No minimum number of tournaments** (section 2). The roadmap asked for at least two tournaments in the window "if seeding ships". Seeding did not ship (#24), so there is no new-player bar and the requirement has nothing left to protect. *Recommendation:* drop it, and report how many tournaments the window covers. Keeping it would make the stopping point depend on the calendar a second way. **Confirmed 2026-11-23:** no minimum; the report gives the count.
10. **Tuned Elo** (section 7). K from v2's grid, chosen by Brier on every match through the lock date, burn-in 5. *Recommendation:* confirm. **Confirmed 2026-11-23:** tuned Elo as in section 7. `v3-lock` records its K.
11. **The external rating** (section 10). No bar refers to it (#10, roadmap contradiction C3). The issue listed a comparison with the rating players registered with; under #10 and #35 any such comparison is private only. *Recommendation:* run none in v3. **Confirmed 2026-11-23:** no external-rating comparison in v3.
12. **Count-only monitoring** (section 2). Someone has to watch the count to know when the window ends. *Recommendation:* a `v3-count` command, written and reviewed before the lock, that reads only who played and when (never a score or a winner) and prints the running count. Who runs it and how often: owner. **Confirmed 2026-11-23:** `v3-count` (section 12). The owner runs it weekly.
13. **Commands that must exist before the lock** (section 12). The `score` path for the processed dataset under the lock (issue #26 scope), item 12's count command, and a v3 `lock` that needs no tuned config. *Recommendation:* all three land, with tests on the stand-in, before the lock. **Confirmed 2026-11-23:** all three land with tests on the stand-in before the lock. They are written: `v3-count`, `v3-lock` and `v3-score` (section 12).
14. **Review before the lock.** The roadmap asked for an external review of this draft before the lock (#13), and on 2026-11-23 the owner confirmed sending it. **Replaced 2026-11-24** ([issue #26](https://github.com/GeoBradDev/open-engine-evaluation/issues/26#issuecomment-7131147762)): there is no external reviewer. The plan is not externally reviewed before the lock and says so under Limitations (section 14). An independent internal check runs instead: a fresh reviewer with no part in the draft, reporting errors only. It ran on 2026-11-24 and its findings are fixed (the note under the issue line above). The plan is made public at the lock, so anyone can critique it against its fixed sha256.

## Why v3

The initial evaluation (`reports/initial-2026-11`) and the V1 work (#24, #36, #40) were not pre-registered: the methods were fixed before results were computed, but not locked, and the data include v1's holdout. Plan v2 was never locked (rescoped 2026-11-21). v3 is the one pre-registered test: engine 1.0.0 with parameter set v1, fixed before any test match is played, scored once on league nights and tournaments played after the lock.

What v3 can and cannot show:

- It can show whether V1's accuracy on new play is not worse than tuned Elo's, whether its point shares are biased toward the favourite, and whether V1 beats the placeholder defaults on new data.
- It cannot show anything about other clubs, about new-player seeding (not shipped), or about rally scoring (none expected). Every limit in section 14 applies.

## Engine

- **Package:** `openpickleball-engine==1.0.0`, as published on PyPI.
- **Tag and commit:** `v1.0.0`, commit `be2a0ff59065f7273b5dc13e395e5ec25ee01d3c`.
- **Wheel:** `openpickleball_engine-1.0.0-py3-none-any.whl`, sha256 `403201b8d2e543726e322adef2d96dadeb58d067fd703bb7f02f71766133c9c3` (checked against PyPI's file listing on 2026-11-23).
- **Installed files:** the wheel's `open_engine` package files have the digest `b3452fe1ae75a39e676638fd18fabf0b5f4c5ba777e92c8a874dc92f656de4f9` (sha256 over the sorted lines "path,sha256", one per file, byte-code caches left out; computed from the wheel above on 2026-11-24). `v3-lock` compares the installed files with it, so the install must come from that wheel (`uv sync --locked`), not only the `uv.lock` entry.
- **Parameter set:** `Config.published("v1")`, parameter fingerprint (`parameter_fingerprint()`) `f9149f3c38a01efc`. Its full-config fingerprint at 1.0.0 (`fingerprint()`, which also covers the package version) is `fbee4568a6b0a21c`. Values: side-out scale 1.0, rally scale 1.0, side-out `a` 0.476, rally `a` 1.0, `k_min` 0.1079136690647482, `k_max` 0.539568345323741, every other field at its default. These are #40's V1 values.
- **Baseline config:** `Config()` at 1.0.0, the placeholder defaults (side-out and rally scale 2.9, `a` 1.0, `k_min` 0.12, `k_max` 0.6), parameter fingerprint `1058623faf5d9367`.
- **Freeze.** The lock hashes the installed engine code, and the pin does not move between the lock and the last `score` run. A patch release for the backend in that window is allowed; the evaluation keeps its pin (roadmap 4.6). A move is a deviation (section 13).
- **Today's pin.** #11 (merged 2026-11-24) pins `openpickleball-engine==1.0.0` from PyPI (owner decision 3). `v3-lock` refuses to run on any engine but exactly 1.0.0 with V1's parameter fingerprint `f9149f3c38a01efc`. The power simulation ran under `e62468c`. It fits its own V1 on synthetic data and uses no 1.0.0-only API, so the pin does not change what it measures.

## 1. Data

- **Source.** The processed dataset written by `clean` (`data/processed`): the league exports and the tournament exports in one ledger, with the initial evaluation's cleaning and exclusion rules (`evaluation/clean.py`, `evaluation/README.md`). Its data version is recorded at the lock and again after the stopping point. Before the lock and before the score run, `clean` is rerun into a new folder on exports covering every play date through the lock date (or the stopping date); `v3-lock` and `v3-score` refuse a processed folder whose manifest does not record exactly the input files now in the raw folder.
- **Owner files.** `clean` also reads four private owner files (`clean.OWNER_FILES`: the name map, `identity_aliases.csv`, `excluded_names.csv` and the rating corrections). The lock records each one's sha256 (or that it is absent); `v3-score` compares them, and any change after the lock is a deviation (section 13).
- **Self-reported matches.** A self-reported match counts in the engine only with both teams' confirmation, which the processed dataset does not hold. By rule, every self-reported match is removed from the ledger before anything is counted, rated or scored, by every model and in the lock's history; `v3-count`, the lock and the report state how many were removed. (`clean` writes none today.)
- **Through the freeze.** Data version `28bcab9de3af`: 1,601 doubles matches, 2025-08-10 to 2026-11-20, 46 league nights and 6 tournaments. V1 was fitted on exactly these (#40).
- **After the stopping point.** The same `clean` run on the new exports. Every match on or before the lock must be unchanged (the history check, section 2).
- **Names.** The processed dataset holds real player names and stays private under `data/`. Every published output holds aggregates only and passes the name check (`_check_no_names`, which checks every name token of any length since #39) and the external-name check (`check_no_external_name`) before it is written.
- **Scope.** Doubles, side-out. Rally or singles matches, if any appear, stay in the ledger as rating history and are not scored (section 4); the report flags any rally match (roadmap 4.4).

## 2. Segments and the stopping rule

- **History:** every match on or before the lock's UTC date. Replayed for ratings by every model, never scored. V1 saw only the matches through 2026-11-20; matches from 2026-11-21 to the lock are history that no engine fit saw (tuned Elo's K uses them, section 7).
- **Lock day.** Match dates are local dates; the lock date is a UTC date. The lock is taken on a day with no play, at a time when the local date and the UTC date are that same day (so the day has no play in both local time and UTC), after a fresh `clean` covering every play date through it. `v3-lock` refuses when the local date is not the UTC date (locked on a local evening west of UTC, the next local day's play would be dated on the lock date and become history), refuses data with any match on the lock date, refuses without the owner's `--exports-complete` confirmation, and prints the last match date for the owner to confirm.
- **Test start:** the first match date after the lock's UTC date (league night or tournament).
- **Burn-in-eligible match:** a doubles, side-out match on or after the test start, in a game format the engine accepts, with a win model for its format (section 5), whose four players each have at least 5 earlier matches in the ledger. Earlier matches are counted in ledger order in the match's own rating format; a category match also counts toward its parent format (overall doubles) while `category_feeds_parent` is on, as the engine counts it. Whether a match is eligible depends only on who played and when, never on who won. The stop uses this count. The engine can still decline a match for a reason that depends on its score (a content duplicate), so the scored set is the overlap of the counted matches with the matches V1 predicted; any match on which the count and V1's predictions disagree is flagged in the report as a deviation.
- **Stopping date (owner decision D8):** the first match date on which the number of burn-in-eligible matches since the test start reaches **400**, or **2027-06-30**, whichever is first. Every match on the stopping date is in the window, so the window can hold somewhat more than 400.
- **Dropped:** matches after the stopping date are never read for any result.
- **How the end is known without peeking.** `v3-count` (owner decision 12, section 12) prints the running count from the match list. It reads no score and no winner. The owner runs it weekly. The stopping date is the first date on which the count reached 400, even if the exports arrive late; nothing played after it is scored. Because a missing export would move the stopping date later, the score run waits until exports covering every play date through the stopping date are in and a fresh `clean` has run, and the owner confirms that with `--exports-complete` before the run (section 12).
- **No minimum number of tournaments** (owner decision 9). The roadmap's requirement of at least two tournaments applied only if seeding shipped, and it did not (#24). The report states how many tournaments and tournament matches the window holds.
- **History check.** After the stopping point, the ledger through the lock is compared with the one recorded at the lock (match count, dates, and a sha256 over the history matches in ledger order). A difference is a deviation (section 13).

## 3. The configs

v3 tunes nothing for the engine.

- **V1:** `Config.published("v1")` at engine 1.0.0, exactly as published (owner decision 2).
- **`Config()`:** the 1.0.0 defaults, the baseline of criterion 3.
- **Tuned Elo** and the information-only baselines: section 7.

## 4. Prediction

The initial evaluation's protocol, which is v2's:

- **Scheduled replays.** Every 7 days (`replay_interval_days`) from the scheduled anchor, the first doubles match date in the ledger. Each match is predicted from the latest replay strictly before its date; matches up to the next replay date are then predicted and recorded live in ledger order (`prequential_predictions`, scheduled mode). V1 and `Config()` use the same anchor and the same ledger.
- **Starting ratings.** Every player starts at 3.5. No seeding: the engine never sees the external rating.
- **What is scored.** The burn-in-eligible matches in the window (section 2) that V1 predicted (the overlap; disagreements are flagged). A match V1 scores but `Config()` or tuned Elo does not is left out of that comparison only; the report gives each comparison's paired count.

## 5. Win probability

The engine's own (`open_engine.winprob`): the expected point share turned into a game and match win probability under independent points, then calibrated with the parameter set's side-out `a` (`sigmoid(a x logit(p))`). V1 uses its `a` 0.476; `Config()` uses `a` 1.0. Matches with no win model for their format are not eligible (section 2).

## 6. Metrics

As in v2, section 6: Brier score, log loss (probabilities clipped to [0.000001, 0.999999]), accuracy, the reliability table, point-share mean squared and absolute error, and the favourite-side point-share bias (the mean of actual minus expected share from the predicted favourite's side; negative means overconfident).

## 7. Baselines

- **Tuned Elo** (criterion 1). v2's Elo (v2, section 7): team rating the mean of the players, start 1500, win or loss only, burn-in 5 in Elo's own history. K from (8, 12, 16, 20, 24, 32, 40, 48, 64, 80, 96, 128, 160) by the lowest Brier score on every match through the lock (owner decision 10); a tie picks the smaller K. It then runs over the whole ledger.
- **Information only:** default Elo (K 32), Glicko-2 and TrueSkill (tuned with v2's grids on every match through the lock date, and at their defaults), higher-rated team wins, and coin flip, all on the common set of matches every row scores. Glicko-2's and TrueSkill's tunes run inside `v3-score`, on the matches through the lock date only; they are not recorded in the lock. None decides anything.

## 8. Intervals

- **Decision intervals:** #28's two-way bootstrap (`layout.TWO_WAY_BOOTSTRAP`): match dates and players resampled independently, additive weights, 2,000 resamples, seed 20261114, 95% percentile interval. It is conservative: it counts some shared dependence twice.
- **Shown beside it:** the date bootstrap (the match date as the cluster, same resamples and seed), labelled too narrow because players recur.
- Every metric in section 6 that is a mean over matches gets a two-way interval; reliability bins get date-bootstrap intervals, as in v2.

## 9. Success criteria

Three criteria. None refers to an external rating. There is no new-player criterion, because seeding did not ship (#24). Every lower-bound inequality is strict, as in #40; criterion 2's bar is "at most".

1. **V1 not worse than tuned Elo.** PASS if the two-way 95% lower bound of (tuned Elo Brier minus V1 Brier), paired by match, is above -0.005. This is v1's and v2's criterion 1, with the two-way interval (owner decision 5). What it takes to pass: at about 400 matches the two-way half-width is about 0.014, so V1's point estimate must be ahead of tuned Elo by about +0.009. If V1 and tuned Elo are truly equal, this criterion almost never passes; it passes reliably only if V1 keeps the tuned engine's lead in the initial evaluation (+0.0175). A FAIL therefore does not show that V1 is worse.
2. **Favourite-side bias** (owner decision 6, option 2). PASS if the absolute value of V1's favourite-side point-share bias is at most 0.015. This bar is underpowered: with no true bias it passes in only about 80% to 90% of samples this size, and with a true bias of -0.0143 (the initial evaluation's estimate) in about half (section 11). The two-way 95% interval is reported beside it and decides nothing. (Option 1, which also required the interval to include 0, was not chosen; at this sample size it decides every sample the same way.)
3. **Default change, #40's rule on post-lock play.** With the difference `Config()` minus V1, paired by match (positive favours V1): PASS if (the two-way 95% lower bound of the Brier difference is above 0, or that of the log-loss difference is above 0) and the Brier lower bound is above -0.005. Burn-in 5 (owner decision 7); the no-burn-in result is shown for information. The outcomes are #40's: both bars pass, and V1 has the evidence to become the defaults (owner decision 8); only the -0.005 bar passes, and the defaults stay; the -0.005 bar fails, and the placeholders stay while the cause is found. Its gain part passes on Brier or on log loss, so it has two chances to pass and its false-pass rate is above the nominal 2.5% of one one-sided bound. This is #40's rule, kept unchanged.

The report prints PASS or FAIL for each and publishes all three whatever they show. It also prints, unscored, whether criterion 1's lower bound is above 0 (V1 clearly better than tuned Elo).

**No overall verdict.** There is no composite verdict and no "V1 passed v3". Each criterion is reported on its own, in these words: criterion 1, "V1 was not worse than tuned Elo by more than 0.005 in Brier score" (PASS) or "the data did not show that V1 is not worse than tuned Elo" (FAIL, which does not show that V1 is worse); criterion 2, "V1's favourite-side bias was within 0.015" or "was not within 0.015" (an underpowered bar either way); criterion 3, the outcome sentence above. One criterion passing is never described as V1 passing v3.

## 10. Information only

Pre-registered here so that none is chosen after the fact; none is a criterion:

- the baseline table (section 7), with paired differences against V1;
- the experience groups: matches whose fewest earlier matches among the four players is 0, 1 to 4, or 5 or more (every earlier match in the ledger counts, as in the initial evaluation), scored with no burn-in, V1 against `Config()` and tuned Elo;
- live against frozen within each tournament day (#25's comparison: V1 live against V1 frozen at a replay through the day before the match date), split the same way, plus all tournament matches as in #25;
- convergence and rating stability, as in the initial evaluation, under V1 on every match through the stopping date;
- the reliability and point-share calibration tables, as in v2: V1's win-probability reliability and point-share calibration on the scored window; tuned Elo's on the matches V1 and tuned Elo both score; tuned Glicko-2's and TrueSkill's on the baseline table's common set (tuned at score time on matches through the lock date, section 7). All are information only;
- v2's stopping-rule diagnostic (the bias against the exact expected share of a game's points).

No published result uses the external rating players registered with (#10, #35), and this draft runs no comparison with it (owner decision 11).

## 11. Power simulation (recorded before the lock)

`uv run python -m evaluation v3-power --seeds 100 --first-seed 1 --lock 2026-12-01 --jobs 20 --out reports/v3-power-2026-11`, run under the current pin (engine `e62468c`). Full tables: `reports/v3-power-2026-11/v3-power.md`. Information only; no criterion depends on it.

**Method.** 100 synthetic clubs (seeds 1 to 100). Each plays the synthetic stand-in's history unchanged through 2026-11-20, fits V1 by #40's procedure on it, and then continues on four assumed calendars, each locked on 2026-12-01: *central* (half the Sundays are league nights, a tournament every 6 weeks), *recent pace* (the real data's last quarter: 6 league nights in 13 weeks, a tournament every 3 weeks), *league only* (half the Sundays, no tournament) and *slow* (a quarter of the Sundays, a tournament every 8 weeks). Every run applies section 2's stopping rule from counts only and section 9's bars with the two-way bootstrap. The stand-in is not the real club (its fitted side-out scale is about 2.6, not 1.1), so its pass rates describe the stand-in. What carries over is the size of the intervals at the stopping rule's sample size.

**The window.**

| Scenario | Stopped by 400 / by the end date | Median stopping date | Median scored matches | Median tournaments in the window | Runs with 2+ tournaments |
|---|---|---|---|---|---|
| central | 97 / 3 | 2027-04-25 | 416 | 4 | 100 of 100 |
| recent pace | 98 / 2 | 2027-04-23 | 412 | 7 | 100 of 100 |
| league only | 91 / 9 | 2027-05-16 | 416 | 0 | 0 of 100 |
| slow | 15 / 85 | 2027-06-30 | 321 | 4 | 100 of 100 |

At the central pace the window closes after about 4.7 months, as the roadmap expected. Any calendar with a tournament every two months or more often gives at least two tournaments in the window; that is now information only (owner decision 9).

**Interval widths (central; recent pace and league only are within about 5%, slow is about 10% to 20% wider).** Two-way 95% half-widths at about 416 matches: Brier, tuned Elo minus V1, +/-0.014; Brier, `Config()` minus V1, +/-0.014; log loss, `Config()` minus V1, +/-0.039; favourite-side bias, +/-0.040. The roadmap's "about +/-0.03 at 285 matches" for the bias was a league-night bootstrap interval, which #28 found too narrow; the two-way interval is wider. Across simulated clubs the estimates vary about half as much as these half-widths imply (the ratio of half-width to 1.96 times the spread is 1.7 to 2.6), which fits #28's note that the two-way bootstrap is conservative.

**Power.** A lower-bound bar passes when the estimate exceeds the bar plus the half-width. With the central scenario's widths:

- *Criterion 1, V1 not worse than tuned Elo:* passes when V1's estimated lead is above about +0.009. If V1 keeps the initial evaluation's lead (+0.0175), power is about 100% (92% in the slow scenario). If V1 and tuned Elo are truly equal, it is about 0% (owner decision 5).
- *Criterion 3, default change:* the gain bar needs a Brier gain above about +0.014 or a log-loss gain above about +0.039. At #40's estimates (Brier +0.0047, log loss +0.0093) power is about 0%; at a Brier gain of +0.010 it is about 12%. In the stand-in world, where V1's median Brier gain is +0.004, the rule passed in 0 to 1 of 100 runs per scenario. The Brier non-inferiority part alone needs a gain above about +0.009.
- *Criterion 2, bias:* with no true bias the estimate stays within +/-0.015 in about 80% to 90% of runs (82% central). At a true bias of -0.0143 the bar passes about half the time, and at -0.0200 about a third. Options 1 and 2 gave the same outcome in every run of every scenario, because the interval's half-width (about 0.040) is far wider than 0.015: whenever the estimate is within +/-0.015 the interval includes 0. Option 1 would only differ once the half-width fell below 0.015, which needs several times more matches.

These powers use the spread of the estimates across simulated clubs, which also holds the differences between clubs, so a power below 50% is an upper bound and one above 50% a lower bound.

**What this means for the plan.** The criterion 1 bar can be met only if V1 is clearly ahead of tuned Elo, as the tuned engine was in the initial evaluation. The default-change test will almost certainly keep the placeholders unless V1's true gain is several times #40's estimate; a FAIL there says little. The bias bar is underpowered in the plain sense: a well-calibrated V1 fails it in one sample in five or so.

## 12. Commands

Written with tests on the synthetic stand-in before the lock (owner decision 13), in `evaluation/v3.py`. The lock's code hash covers them. Usage is in `evaluation/README.md`.

- `v3-count`: the running count of burn-in-eligible matches since the test start, and whether D8's stop has been reached. It reads only who played, when and the match format from `matches.csv` and never uses the `games` column, so it never sees a score or a winner. It takes the lock date from the v3 lock (or, before the lock, from `--lock-date` as a preview). The owner runs it weekly (owner decision 12).
- `v3-lock`: needs no tuned config. It refuses to run unless the installed engine is exactly `openpickleball-engine==1.0.0` (and, when the install records a commit, commit `be2a0ff`), `Config.published("v1")` has parameter fingerprint `f9149f3c38a01efc`, `uv.lock` resolves the engine to the wheel sha256 in "Engine", and the installed engine files have the digest in "Engine". It refuses without `--exports-complete`, refuses a processed folder not cleaned from the raw folder's current files, refuses data with a match after the lock's UTC date or on it, refuses when the local date is not the UTC date (section 2), and never overwrites a lock or its summary. It records this file's sha256, a sha256 over `evaluation/*.py` and the installed `open_engine` code, the evaluation commit, the UTC time and date and the local date, the engine version, commit, wheel sha256 and installed-files digest, the parameter set's name and parameter fingerprint, the data version, the owner files' sha256, the number of self-reported matches removed, the history check of section 2, and tuned Elo's K (section 7), in a private lock file under `data/v3/`. Beside it, it writes a public summary of every lock field (`plan-v3-lock-summary.md`: hashes, dates and counts only, no player data), checked by the name check and the external-name check. It prints the last match date for the owner to confirm.
- `v3-score`: reads the v3 lock and refuses to run if this file has changed, if the installed engine version, the installed engine files or V1's parameter fingerprint differ from the lock, if D8's stop has not been reached, without `--exports-complete` (the owner's confirmation that the exports cover every play date through the stopping date), or on a processed folder not cleaned from the raw folder's current files. It reports whether the code, the owner files and the history still match the lock (a difference is a deviation, section 13). It finds the stopping date from the same count as `v3-count`, scores the window once (criteria 1 to 3 with the two-way bootstrap, the date bootstrap beside it, the no-burn-in default-change result, the unscored "clearly better" line, the tournament count and any rally match), produces every information-only table of section 10 on the same window (`evaluation/v3info.py`), writes the report and its run record through the name check and the external-name check into a new folder for that run (`data/v3/report/run-<N>`, never overwritten; it refuses if the folder exists), and appends every run, with its folder and the report's sha256, to a score log under `data/v3/`. It reads no external rating file.

### Lock-day procedure

The owner follows these steps in order. Steps 1 and 2 happen before the lock day; nothing in this file changes after step 2.

1. Check that #11's 1.0.0 pin is on `main`, then run `uv sync --locked` in a clean checkout of `main`.
2. Fill in the sign-off below (inputs only), commit it to `main`, pull, and confirm `git status` is clean.
3. On the lock day (a day with no play, at a time when the local date and the UTC date are both that day), export every source through that day into `data/`, and run `uv run python -m evaluation clean --data-dir data --out data/v3/processed-lock` (a new folder; `clean` refuses one that is not empty).
4. Optionally preview the count: `uv run python -m evaluation v3-count --processed data/v3/processed-lock --lock-date <the lock day>`.
5. Run the sign-off's command line. Confirm the last match date it prints is the last play date before the lock.
6. Publish `data/v3/plan-v3-lock-summary.md` right away beside the public plan (for example as `docs/evaluation/analysis-plan-v3-lock.md`, committed to `main`); a registry entry, if the owner chooses one, holds the plan's sha256 from it. Publishing does not change this file or `evaluation/*.py`, so the lock stays valid.
7. Each week, export the new play, run `clean` into a new dated folder (for example `data/v3/processed-<date>`), and run `uv run python -m evaluation v3-count --processed <that folder> --lock data/v3/plan-v3.lock`. Once it reports the stop, collect the exports through the stopping date, run `clean` into a new folder once more, and run `uv run python -m evaluation v3-score --processed <that folder> --raw data --exports-complete` once. That first logged run is the result.

## 13. Deviations

v2's rules (v2, section 12) apply. Every `score` run is logged and kept in its own folder. The first logged `v3-score` run is the result; any later run is a deviation, reported beside the first with its reason, and never replaces it. Relocking is a deviation. A bug found after scoring is fixed, the window rerun, and both results reported. A change to this plan after the lock, a pin move, a changed history, a changed owner file, or scoring with a different stopping date is a deviation, listed with its reason and with the results under both versions where both exist. Any analysis not named here is exploratory.

## 14. Known limitations

- **One club.** One club in one region, a mostly closed pool. Results may not carry over to other pools.
- **Small sample.** About 400 scored matches. Section 11 gives the expected interval widths; the default-change test in particular has little power.
- **V1 was fitted on this club.** V1's two fitted numbers come from the same pool's earlier play. v3 tests them on later play only.
- **No seeding.** New players start at 3.5. The experience groups show what that costs, for information.
- **Side-out approximation.** Points are modelled as independent, and one fitted `a` corrects overall sharpness only.
- **Order within a night.** Row order approximates the order of play.
- **Dependence.** Players recur across dates. The two-way interval allows for it, conservatively.
- **No external review.** The plan was not externally reviewed before the lock (owner decision 14, as amended on 2026-11-24). It had only an independent internal check (2026-11-24). It is made public at the lock, so it can be critiqued against its fixed sha256 afterwards.

## Owner sign-off

Fill in every field before the lock (lock-day procedure, step 2). Every field is an input known before `v3-lock` runs. Nothing else in this file changes after this point.

- Owner decisions 1 to 14 (item 14 as amended on 2026-11-24) confirmed, or the changes made: `Decisions 1 to 13 confirmed 2026-11-23; 14 as amended 2026-11-24.`
- Non-inferiority margin and decision interval (decision 5): `-0.005, two-way bootstrap.`
- Bias bar option (decision 6): `Option 2.`
- Engine pin in `pyproject.toml` and `uv.lock` (`openpickleball-engine==1.0.0`, wheel sha256 as in "Engine"): `openpickleball-engine==1.0.0, wheel sha256 403201b8d2e543726e322adef2d96dadeb58d067fd703bb7f02f71766133c9c3`
- Lock command line, run from the repository root:

```
uv run python -m evaluation v3-lock --plan docs/evaluation/analysis-plan-v3.md --processed data/v3/processed-lock --raw data --lock data/v3/plan-v3.lock --exports-complete
```

**What the lock records instead.** The values that exist only once `v3-lock` has run are not written here, because writing them would change this file's sha256 and `v3-score` would then refuse to run, and a commit cannot hold its own hash. They are in the lock file and its public summary: the lock time, UTC date and local date; the test start (the first match date after the lock date, known only once play resumes, and printed by `v3-count`); the data version; the history count, first and last date and sha256; tuned Elo's K; the evaluation commit; the code sha256; the owner files' sha256; and the engine record. The summary is published right after the lock, beside this plan (lock-day procedure, step 6).

The lock file itself is private and lives only on the owner's machine; its contents (hashes, dates and counts, no player data) are public through the summary. A public registration (for example on OSF), if the owner chooses one, is the independent record of when this plan was fixed.
