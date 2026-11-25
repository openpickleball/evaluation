# How well does the OPEN rating engine predict matches? Initial results

November 2026. Initial evaluation, not a final verdict. This is a snapshot written before engine 1.0.0 shipped; `writeup.md` at the repository root has the current reading.

## The short version

- **The engine picks the winner about 63% of the time** on matches it had not seen, when every player already has some history. The standard rating systems we tested it against picked 58% (Elo), 62% (Glicko-2) and 62% (TrueSkill). The engine was ahead of Elo, though by much less in some stretches of weeks than others. The gaps over Glicko-2 and TrueSkill are small and could be chance.
- **Updating ratings during a tournament did not help.** Ratings frozen at the start of a tournament predicted about as well as ratings updated after every match, and perhaps slightly better.
- **The engine struggles with brand-new players.** Everyone starts at 3.5. In matches with a player the engine had never seen, it picked only 42% of winners. Its prediction error there was worse than a coin flip's, though on only 46 matches and within the margin of chance. This is its clearest weakness.
- **Ratings are steady.** A player who plays a week typically moves about 0.03. A player who does not play does not move. Week-to-week rankings barely change.

## What we tested

We used every match we have from one club: 1,601 doubles matches and 189 players, from August 2025 to November 2026. They were played on 52 dates: league nights and one-day tournaments.

We split the matches by date:

- **Before 2026-08-24 (1,087 matches):** the engine's settings were tuned on these.
- **From 2026-08-24 (514 matches):** the test. Every one was predicted before its result was used, so the engine never saw the future.

We compared the engine with three standard rating systems (Elo, Glicko-2 and TrueSkill) and with a coin flip. Each rating system was tuned on the same early matches.

## Results

### Picking winners

On 285 test matches where all four players had at least five earlier matches:

| System | Winners picked | Prediction error (lower is better) |
|---|---|---|
| OPEN engine | 63% | 0.222 |
| Glicko-2 | 62% | 0.229 |
| TrueSkill | 62% | 0.233 |
| Elo | 58% | 0.240 |
| Coin flip | 50% | 0.250 |

The prediction error (Brier score) also rewards being honest about uncertainty, not just picking the right side. The comparison is not like for like: only the engine had its confidence tuned (see below), and only the engine uses the score of each game, while the other systems see only who won. Before that tuning, the engine's prediction error was 0.235, behind Glicko-2 and TrueSkill.

Its probabilities match how often teams actually win: teams it gave about a 64% chance won 64% of the time (`reliability.png`). It got there only after tuning shrank its raw confidence by more than half. Untuned, the engine is overconfident.

### Updating during a tournament

On 219 tournament matches from the last four test tournaments (every test tournament after the first), we compared two ways of running the engine:

| OPEN engine | Winners picked | Prediction error (lower is better) |
|---|---|---|
| Using ratings from before the event | 57% | 0.234 |
| Updating after each match of the event | 56% | 0.239 |

Updating during the event made the prediction error 0.006 worse. That is inside the margin of chance (anywhere from 0.007 better to 0.019 worse), so updating did not help, and may have hurt slightly. Brand-new players are included here, unlike in the main test above, so these numbers should not be compared with it.

### New players

| Fewest earlier matches among the four players | Winners picked by the engine |
|---|---|
| 0 (someone brand new) | 42% |
| 1 to 4 | 61% |
| 5 to 9 | 67% |
| 10 or more | 60% to 62% |

Once a player has about 5 matches, the engine treats them as well as anyone else. For half of the regular players, the rating settled within 0.25 of where it ended up, and stayed there, within 6 matches. For a quarter, it took 18 or more (`convergence.png`).

### Over time

The engine beat Elo in every one of the three test months (`accuracy-over-time.png`). Results do swing from one stretch of weeks to the next. In one stretch of six match dates in July and August (`rolling-splits.md`, fold 2), the engine's prediction error was slightly worse than a coin flip's, within the margin of chance, and Glicko-2 beat it.

## Limits of this test

- **One club, about 190 players.** Results may not hold elsewhere.
- **Not pre-registered.** We fixed the method before looking at results, but did not lock it in advance, and some of these matches were looked at in an earlier round. Treat this as a strong first look, not proof.
- **Small numbers.** The tournament comparison rests on four tournament days. The new-player numbers rest on 46 matches.
- **Order of play is approximate.** Within a night, matches are in the order they were entered.
- **Names were matched by hand.** Tournament players listed by first name were matched to full names by the organizer.

## What this meant before the 1.0.0 release

Written before 1.0.0 shipped. Since then, 1.0.0 declared what a rating gap means (see `writeup.md`, section 1). For players it knows, the engine predicted about as well as the established systems we tested. Two things stood out before calling it 1.0:

1. **New players start blind.** Every new player starts at 3.5, and predictions are weakest for their first matches.
2. **The rating scale needs a decision.** Ratings are squeezed into a narrow band: the middle half of regular players sits between 3.3 and 3.7 (`rating-distribution.png`). What a rating number means has to be stated before it is published.

A pre-registered test on matches played after a locked plan would turn this first look into confirmatory evidence.

Details, intervals and method: `technical-report.md`. Data version `28bcab9de3af`, engine tag `eval-2026-11` (commit `e62468c`).
