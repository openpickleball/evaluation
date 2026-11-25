# Declared side-out scale with K tuned for accuracy

**Status: part of the initial evaluation, not pre-registered.** This tests openpickleball/engine#149's proposal to declare the side-out scale and tune K instead. The method and decision rule were written down before any result was computed.

## Provenance

- Engine: `openpickleball-engine` 0.3.0.dev0 at commit `e62468c508126ed46b5a26a82f175ba17dd78371`.
- Data version: `28bcab9de3af`.
- Bootstrap: 2000 resamples, seed 20261114, 95% percentile intervals, resampling match dates.

## Step 1: the scale from spread alone (tuning part only)

For each candidate scale, K is chosen as in step 2, ratings are replayed to 2026-08-23, and the IQR is taken over players with at least 10 matches. The chosen scale gives the IQR nearest 0.8. K multipliers are 2^(i/4) for i = -4 to 8 (0.5 to 4) times the default k_min and k_max; a grid edge is a chosen multiplier of 0.5 or 4.

| Scale | K multiplier | k_min | k_max | a | Tuning Brier | IQR | Players | Grid edge | Distance from 0.8 |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| 1.30 | 2.000 | 0.2400 | 1.2000 | 0.358 | 0.23430 | 0.657 | 81 | no | 0.143 |
| 1.74 | 2.000 | 0.2400 | 1.2000 | 0.411 | 0.23449 | 0.680 | 81 | no | 0.120 |
| 2.20 | 2.000 | 0.2400 | 1.2000 | 0.444 | 0.23539 | 0.678 | 81 | no | 0.122 |
| 2.90 | 2.000 | 0.2400 | 1.2000 | 0.478 | 0.23703 | 0.707 | 81 | no | 0.093 |

## Step 2: the declared engine

- Chosen scale 2.9, K multiplier 2.000 (k_min 0.2400, k_max 1.2000), win calibration a 0.478 (at bound: no, n = 494, tuning Brier 0.23703).
- Current tuned engine: fitted scale 0.868, a 0.402, default K (k_min 0.12, k_max 0.6).

Expected point share of the stronger team at a team-rating gap, at each engine's scale:

| Engine | Scale | 0.5 gap | 1 gap |
| --- | --- | --- | --- |
| Declared-scale engine | 2.900 | 59.8% | 68.9% |
| Current tuned engine | 0.868 | 79.0% | 93.4% |

## Step 3: test part

| Engine | Matches | Brier [95%] | Log loss [95%] | Accuracy [95%] |
| --- | --- | --- | --- | --- |
| Declared-scale engine | 285 | 0.2257 [0.2153, 0.2364] | 0.6418 [0.6189, 0.6655] | 0.604 [0.559, 0.655] |
| Current tuned engine | 285 | 0.2224 [0.2127, 0.2312] | 0.6340 [0.6123, 0.6546] | 0.628 [0.579, 0.687] |

Paired Brier difference, current tuned engine minus declared-scale engine, on the 285 test matches both score: -0.0033 [-0.0069, -0.0003] (negative favours the current engine).

Rule: PASS if the interval's lower end is above -0.005, else FAIL. Lower end above -0.005: no.

Verdict: **FAIL**

## By experience (test part, no burn-in)

Declared-scale engine:

| Fewest earlier matches | Matches | Engine Brier [95%] | Engine accuracy |
| --- | --- | --- | --- |
| 0 | 46 | 0.2534 [0.2256, 0.2803] | 0.533 |
| 1 to 4 | 183 | 0.2329 [0.2181, 0.2447] | 0.607 |
| 5 to 9 | 88 | 0.2249 [0.1901, 0.2494] | 0.648 |
| 10 to 19 | 97 | 0.2306 [0.2170, 0.2448] | 0.567 |
| 20+ | 100 | 0.2217 [0.2029, 0.2386] | 0.600 |

Current tuned engine:

| Fewest earlier matches | Matches | Engine Brier [95%] | Engine accuracy |
| --- | --- | --- | --- |
| 0 | 46 | 0.2602 [0.2238, 0.2944] | 0.424 |
| 1 to 4 | 183 | 0.2345 [0.2154, 0.2497] | 0.612 |
| 5 to 9 | 88 | 0.2202 [0.1854, 0.2444] | 0.670 |
| 10 to 19 | 97 | 0.2221 [0.2069, 0.2342] | 0.619 |
| 20+ | 100 | 0.2246 [0.2008, 0.2448] | 0.600 |

## Rating distributions

Players with at least 10 matches; inclusive quartiles, the same method as step 1's IQR.

| Distribution | Players | Min | Q1 | Median | Q3 | Max |
| --- | --- | --- | --- | --- | --- | --- |
| Declared-scale engine at 2026-08-23 | 81 | 2.00 | 3.22 | 3.51 | 3.92 | 4.89 |
| Declared-scale engine at the last match | 122 | 2.01 | 3.16 | 3.55 | 3.92 | 4.89 |
| Current tuned engine at 2026-08-23 | 81 | 2.73 | 3.35 | 3.53 | 3.69 | 4.04 |
| Current tuned engine at the last match | 122 | 2.61 | 3.33 | 3.52 | 3.70 | 4.09 |

Players ending at or below 2.2, and matches touching a rating bound (a player's rating within 1e-9 of the floor or ceiling, before or after the match, on a live replay of every match):

| Engine | Players at or below 2.2 | Of them with 10+ matches | Matches at the floor | Matches at the ceiling | Matches at either |
| --- | --- | --- | --- | --- | --- |
| Declared-scale engine | 5 | 2 | 60 | 0 | 60 |
| Current tuned engine | 0 | 0 | 0 | 0 | 0 |

