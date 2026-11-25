# Rolling-origin splits (initial evaluation, whole dataset)

Part of the initial evaluation (not pre-registered), for information only: nothing was tuned or chosen on these results. The folds cover the whole dataset, so the last two overlap the main test part. Each fold tunes the engine and every baseline on every match before its test block, then scores the block. Dates are league nights and tournament days. Read this next to `technical-report.md`.

Folds: 4, of which 4 ran. A fold whose strict tune refused, or that had nothing to score, is not run and is not retried.

| Fold | Training | Test block | Scored | Side-out scale | a | Elo K | Status |
|---|---|---|---|---|---|---|---|
| 1 | 28 nights, 673 matches, to 2026-05-03 | 2026-05-17 to 2026-07-12 (6 nights, 180 matches) | 132 | 0.752 | 0.425 | 32 | ran |
| 2 | 34 nights, 853 matches, to 2026-07-12 | 2026-07-19 to 2026-08-21 (6 nights, 204 matches) | 153 | 0.854 | 0.503 | 24 | ran |
| 3 | 40 nights, 1057 matches, to 2026-08-21 | 2026-08-23 to 2026-10-18 (6 nights, 253 matches) | 149 | 0.951 | 0.393 | 12 | ran |
| 4 | 46 nights, 1310 matches, to 2026-10-18 | 2026-10-23 to 2026-11-20 (6 nights, 291 matches) | 166 | 0.868 | 0.431 | 16 | ran |

## Spread across folds

Each fold's estimate, and the mean, minimum and maximum over the folds that ran. Criterion 1's measure passes its bar when its interval's lower end is above -0.005, and criterion 2's when its absolute value is at most 0.015. There is no pooled interval across folds: the folds share training data and players, so they are not independent. Each fold's intervals are below.

| Model | Measure | Fold 1 | Fold 2 | Fold 3 | Fold 4 | Mean | Min | Max | Folds that ran |
|---|---|---|---|---|---|---|---|---|---|
| Criterion 1 measure | Brier difference, Elo minus engine, paired set | +0.0305 | +0.0011 | +0.0220 | +0.0128 | +0.0166 | +0.0011 | +0.0305 | 4 |
| Criterion 2 measure | Point-share bias, favourite's side | -0.0119 | -0.0273 | +0.0373 | -0.0383 | -0.0101 | -0.0383 | +0.0373 | 4 |
| Engine, tuned | Brier | 0.2199 | 0.2584 | 0.2214 | 0.2244 | 0.2310 | 0.2199 | 0.2584 | 4 |
| Engine, tuned | Log loss | 0.6298 | 0.7169 | 0.6353 | 0.6353 | 0.6543 | 0.6298 | 0.7169 | 4 |
| Engine, tuned | Accuracy | 0.6364 | 0.5817 | 0.6913 | 0.5904 | 0.6249 | 0.5817 | 0.6913 | 4 |
| Engine, default config | Brier | 0.2408 | 0.2690 | 0.2295 | 0.2211 | 0.2401 | 0.2211 | 0.2690 | 4 |
| Engine, default config | Log loss | 0.6742 | 0.7376 | 0.6534 | 0.6282 | 0.6734 | 0.6282 | 0.7376 | 4 |
| Engine, default config | Accuracy | 0.5985 | 0.5490 | 0.6174 | 0.6084 | 0.5933 | 0.5490 | 0.6174 | 4 |
| Engine, default config | Brier difference, row minus engine | +0.0209 | +0.0106 | +0.0081 | -0.0033 | +0.0091 | -0.0033 | +0.0209 | 4 |
| Elo, tuned | Brier | 0.2504 | 0.2596 | 0.2433 | 0.2372 | 0.2476 | 0.2372 | 0.2596 | 4 |
| Elo, tuned | Log loss | 0.6952 | 0.7127 | 0.6797 | 0.6670 | 0.6887 | 0.6670 | 0.7127 | 4 |
| Elo, tuned | Accuracy | 0.5303 | 0.4771 | 0.5906 | 0.5904 | 0.5471 | 0.4771 | 0.5906 | 4 |
| Elo, tuned | Brier difference, row minus engine | +0.0305 | +0.0011 | +0.0220 | +0.0128 | +0.0166 | +0.0011 | +0.0305 | 4 |
| Elo, default | Brier | 0.2504 | 0.2625 | 0.2421 | 0.2352 | 0.2475 | 0.2352 | 0.2625 | 4 |
| Elo, default | Log loss | 0.6952 | 0.7190 | 0.6773 | 0.6626 | 0.6885 | 0.6626 | 0.7190 | 4 |
| Elo, default | Accuracy | 0.5303 | 0.5163 | 0.5906 | 0.5843 | 0.5554 | 0.5163 | 0.5906 | 4 |
| Elo, default | Brier difference, row minus engine | +0.0305 | +0.0040 | +0.0207 | +0.0108 | +0.0165 | +0.0040 | +0.0305 | 4 |
| Glicko-2, tuned | Brier | 0.2281 | 0.2437 | 0.2303 | 0.2283 | 0.2326 | 0.2281 | 0.2437 | 4 |
| Glicko-2, tuned | Log loss | 0.6481 | 0.6805 | 0.6530 | 0.6479 | 0.6574 | 0.6479 | 0.6805 | 4 |
| Glicko-2, tuned | Accuracy | 0.6515 | 0.5850 | 0.6309 | 0.6235 | 0.6227 | 0.5850 | 0.6515 | 4 |
| Glicko-2, tuned | Brier difference, row minus engine | +0.0082 | -0.0147 | +0.0089 | +0.0039 | +0.0016 | -0.0147 | +0.0089 | 4 |
| Glicko-2, default | Brier | 0.2281 | 0.2437 | 0.2310 | 0.2296 | 0.2331 | 0.2281 | 0.2437 | 4 |
| Glicko-2, default | Log loss | 0.6481 | 0.6805 | 0.6544 | 0.6507 | 0.6584 | 0.6481 | 0.6805 | 4 |
| Glicko-2, default | Accuracy | 0.6515 | 0.5850 | 0.6242 | 0.6114 | 0.6180 | 0.5850 | 0.6515 | 4 |
| Glicko-2, default | Brier difference, row minus engine | +0.0082 | -0.0147 | +0.0096 | +0.0052 | +0.0021 | -0.0147 | +0.0096 | 4 |
| TrueSkill, tuned | Brier | 0.2412 | 0.2530 | 0.2252 | 0.2356 | 0.2388 | 0.2252 | 0.2530 | 4 |
| TrueSkill, tuned | Log loss | 0.6773 | 0.7049 | 0.6484 | 0.6622 | 0.6732 | 0.6484 | 0.7049 | 4 |
| TrueSkill, tuned | Accuracy | 0.6061 | 0.5752 | 0.6711 | 0.5964 | 0.6122 | 0.5752 | 0.6711 | 4 |
| TrueSkill, tuned | Brier difference, row minus engine | +0.0214 | -0.0054 | +0.0038 | +0.0112 | +0.0077 | -0.0054 | +0.0214 | 4 |
| TrueSkill, default | Brier | 0.2343 | 0.2485 | 0.2254 | 0.2313 | 0.2349 | 0.2254 | 0.2485 | 4 |
| TrueSkill, default | Log loss | 0.6665 | 0.6940 | 0.6487 | 0.6509 | 0.6650 | 0.6487 | 0.6940 | 4 |
| TrueSkill, default | Accuracy | 0.6364 | 0.6209 | 0.6644 | 0.5904 | 0.6280 | 0.5904 | 0.6644 | 4 |
| TrueSkill, default | Brier difference, row minus engine | +0.0144 | -0.0100 | +0.0040 | +0.0068 | +0.0038 | -0.0100 | +0.0144 | 4 |
| Higher-rated team wins | Brier | 0.3636 | 0.4183 | 0.3087 | 0.4096 | 0.3751 | 0.3087 | 0.4183 | 4 |
| Higher-rated team wins | Accuracy | 0.6364 | 0.5817 | 0.6913 | 0.5904 | 0.6249 | 0.5817 | 0.6913 | 4 |
| Higher-rated team wins | Brier difference, row minus engine | +0.1437 | +0.1599 | +0.0873 | +0.1852 | +0.1440 | +0.0873 | +0.1852 | 4 |
| Coin flip | Brier | 0.2500 | 0.2500 | 0.2500 | 0.2500 | 0.2500 | 0.2500 | 0.2500 | 4 |
| Coin flip | Log loss | 0.6931 | 0.6931 | 0.6931 | 0.6931 | 0.6931 | 0.6931 | 0.6931 | 4 |
| Coin flip | Accuracy | 0.5000 | 0.5000 | 0.5000 | 0.5000 | 0.5000 | 0.5000 | 0.5000 | 4 |
| Coin flip | Brier difference, row minus engine | +0.0301 | -0.0084 | +0.0286 | +0.0256 | +0.0190 | -0.0084 | +0.0301 | 4 |

## Each fold

### Fold 1

Criterion measures in this fold, with 95% night-bootstrap intervals within the block. The bars are shown for information only; they decide nothing here.

| Criterion | Bar | Result | Bar met in this fold |
|---|---|---|---|
| Engine not worse than tuned Elo on holdout Brier | 95% CI of (Elo minus engine) Brier lower bound above -0.005 | 0.0305 [0.0203, 0.0423] (n_paired = 132) | yes |
| Holdout point-share bias, favourite's side | \|bias\| <= 0.015 | -0.0119 [-0.0314, +0.0068] | yes |

Baselines: every row is scored on the same matches, the tuned engine's scored test-block matches that every row also scores (132 matches, 6 league nights). Only the tuned engine row is calibrated. Intervals are 95% cluster-bootstrap intervals, resampling league nights (2000 resamples, seed 20261114).

| Model | Settings | Brier | Log loss | Accuracy |
|---|---|---|---|---|
| Engine, tuned | side-out scale 0.752, a = 0.425, scheduled replays | 0.2199 [0.2080, 0.2320] | 0.6298 [0.6033, 0.6573] | 0.636 [0.600, 0.672] |
| Engine, default config | Config(): side-out scale 2.9, a = 1, scheduled replays | 0.2408 [0.2253, 0.2533] | 0.6742 [0.6375, 0.7044] | 0.598 [0.538, 0.654] |
| Elo, tuned | K = 32, tuned on the tuning period | 0.2504 [0.2362, 0.2613] | 0.6952 [0.6656, 0.7176] | 0.530 [0.476, 0.590] |
| Elo, default | K = 32, start 1500, untuned | 0.2504 [0.2362, 0.2613] | 0.6952 [0.6656, 0.7176] | 0.530 [0.476, 0.590] |
| Glicko-2, tuned | tau 0.3, initial RD 350, rating period 7 days, volatility 0.06, tuned on the tuning period | 0.2281 [0.2179, 0.2375] | 0.6481 [0.6263, 0.6680] | 0.652 [0.624, 0.669] |
| Glicko-2, default | tau 0.5, initial RD 350, rating period 7 days, volatility 0.06, untuned | 0.2281 [0.2179, 0.2375] | 0.6481 [0.6263, 0.6680] | 0.652 [0.624, 0.669] |
| TrueSkill, tuned | beta 2 x 25/6, tau 0 x 25/300, draw probability 0, tuned on the tuning period | 0.2412 [0.2224, 0.2579] | 0.6773 [0.6340, 0.7155] | 0.606 [0.561, 0.656] |
| TrueSkill, default | beta 1 x 25/6, tau 1 x 25/300, draw probability 0, untuned | 0.2343 [0.2131, 0.2546] | 0.6665 [0.6122, 0.7248] | 0.636 [0.601, 0.680] |
| Higher-rated team wins | hard call from the tuned engine's expected share | 0.3636 [0.3284, 0.4000] | not defined | 0.636 [0.600, 0.672] |
| Coin flip | 0.5 for every match | 0.2500 (constant, no interval) | 0.6931 (constant, no interval) | 0.500 (constant, no interval) |

Log loss is not defined for Higher-rated team wins: hard 0 or 1 calls make it infinite on any miss.

Coin flip: constant, no interval. The metrics are the same on any set of matches (Brier 0.25, log loss ln 2, accuracy 0.5), so there is nothing to resample. The paired difference below still has an interval.

Paired Brier difference against the tuned engine, on the common set: the row's Brier minus the engine's, paired by match (positive favours the engine), for every row except the tuned engine itself. 95% intervals resample league nights (2000 resamples, seed 20261114). For information only; no success criterion depends on them (criterion 1 is decided on the paired set).

| Model | Brier difference, row minus engine | 95% interval |
|---|---|---|
| Engine, default config | +0.0209 | [+0.0143, +0.0279] |
| Elo, tuned | +0.0305 | [+0.0203, +0.0423] |
| Elo, default | +0.0305 | [+0.0203, +0.0423] |
| Glicko-2, tuned | +0.0082 | [-0.0012, +0.0143] |
| Glicko-2, default | +0.0082 | [-0.0012, +0.0143] |
| TrueSkill, tuned | +0.0214 | [+0.0069, +0.0333] |
| TrueSkill, default | +0.0144 | [-0.0037, +0.0283] |
| Higher-rated team wins | +0.1437 | [+0.1093, +0.1765] |
| Coin flip | +0.0301 | [+0.0180, +0.0420] |

### Fold 2

Criterion measures in this fold, with 95% night-bootstrap intervals within the block. The bars are shown for information only; they decide nothing here.

| Criterion | Bar | Result | Bar met in this fold |
|---|---|---|---|
| Engine not worse than tuned Elo on holdout Brier | 95% CI of (Elo minus engine) Brier lower bound above -0.005 | 0.0011 [-0.0130, 0.0168] (n_paired = 153) | no |
| Holdout point-share bias, favourite's side | \|bias\| <= 0.015 | -0.0273 [-0.0382, -0.0172] | no |

Baselines: every row is scored on the same matches, the tuned engine's scored test-block matches that every row also scores (153 matches, 6 league nights). Only the tuned engine row is calibrated. Intervals are 95% cluster-bootstrap intervals, resampling league nights (2000 resamples, seed 20261114).

| Model | Settings | Brier | Log loss | Accuracy |
|---|---|---|---|---|
| Engine, tuned | side-out scale 0.854, a = 0.503, scheduled replays | 0.2584 [0.2459, 0.2690] | 0.7169 [0.6871, 0.7434] | 0.582 [0.552, 0.617] |
| Engine, default config | Config(): side-out scale 2.9, a = 1, scheduled replays | 0.2690 [0.2601, 0.2779] | 0.7376 [0.7167, 0.7580] | 0.549 [0.517, 0.588] |
| Elo, tuned | K = 24, tuned on the tuning period | 0.2596 [0.2500, 0.2690] | 0.7127 [0.6930, 0.7321] | 0.477 [0.387, 0.543] |
| Elo, default | K = 32, start 1500, untuned | 0.2625 [0.2521, 0.2731] | 0.7190 [0.6973, 0.7409] | 0.516 [0.445, 0.577] |
| Glicko-2, tuned | tau 0.3, initial RD 350, rating period 7 days, volatility 0.06, tuned on the tuning period | 0.2437 [0.2322, 0.2570] | 0.6805 [0.6568, 0.7077] | 0.585 [0.503, 0.662] |
| Glicko-2, default | tau 0.5, initial RD 350, rating period 7 days, volatility 0.06, untuned | 0.2437 [0.2322, 0.2570] | 0.6805 [0.6568, 0.7077] | 0.585 [0.503, 0.662] |
| TrueSkill, tuned | beta 1 x 25/6, tau 4 x 25/300, draw probability 0, tuned on the tuning period | 0.2530 [0.2329, 0.2732] | 0.7049 [0.6594, 0.7520] | 0.575 [0.500, 0.644] |
| TrueSkill, default | beta 1 x 25/6, tau 1 x 25/300, draw probability 0, untuned | 0.2485 [0.2302, 0.2658] | 0.6940 [0.6533, 0.7340] | 0.621 [0.561, 0.675] |
| Higher-rated team wins | hard call from the tuned engine's expected share | 0.4183 [0.3831, 0.4483] | not defined | 0.582 [0.552, 0.617] |
| Coin flip | 0.5 for every match | 0.2500 (constant, no interval) | 0.6931 (constant, no interval) | 0.500 (constant, no interval) |

Log loss is not defined for Higher-rated team wins: hard 0 or 1 calls make it infinite on any miss.

Coin flip: constant, no interval. The metrics are the same on any set of matches (Brier 0.25, log loss ln 2, accuracy 0.5), so there is nothing to resample. The paired difference below still has an interval.

Paired Brier difference against the tuned engine, on the common set: the row's Brier minus the engine's, paired by match (positive favours the engine), for every row except the tuned engine itself. 95% intervals resample league nights (2000 resamples, seed 20261114). For information only; no success criterion depends on them (criterion 1 is decided on the paired set).

| Model | Brier difference, row minus engine | 95% interval |
|---|---|---|
| Engine, default config | +0.0106 | [-0.0013, +0.0190] |
| Elo, tuned | +0.0011 | [-0.0130, +0.0168] |
| Elo, default | +0.0040 | [-0.0106, +0.0189] |
| Glicko-2, tuned | -0.0147 | [-0.0270, -0.0026] |
| Glicko-2, default | -0.0147 | [-0.0270, -0.0026] |
| TrueSkill, tuned | -0.0054 | [-0.0227, +0.0095] |
| TrueSkill, default | -0.0100 | [-0.0263, +0.0046] |
| Higher-rated team wins | +0.1599 | [+0.1205, +0.1913] |
| Coin flip | -0.0084 | [-0.0190, +0.0041] |

### Fold 3

Criterion measures in this fold, with 95% night-bootstrap intervals within the block. The bars are shown for information only; they decide nothing here.

| Criterion | Bar | Result | Bar met in this fold |
|---|---|---|---|
| Engine not worse than tuned Elo on holdout Brier | 95% CI of (Elo minus engine) Brier lower bound above -0.005 | 0.0220 [0.0124, 0.0320] (n_paired = 149) | yes |
| Holdout point-share bias, favourite's side | \|bias\| <= 0.015 | +0.0373 [-0.0060, +0.0784] | no |

Baselines: every row is scored on the same matches, the tuned engine's scored test-block matches that every row also scores (149 matches, 6 league nights). Only the tuned engine row is calibrated. Intervals are 95% cluster-bootstrap intervals, resampling league nights (2000 resamples, seed 20261114).

| Model | Settings | Brier | Log loss | Accuracy |
|---|---|---|---|---|
| Engine, tuned | side-out scale 0.951, a = 0.393, scheduled replays | 0.2214 [0.2087, 0.2327] | 0.6353 [0.6072, 0.6610] | 0.691 [0.614, 0.760] |
| Engine, default config | Config(): side-out scale 2.9, a = 1, scheduled replays | 0.2295 [0.2064, 0.2477] | 0.6534 [0.5982, 0.6980] | 0.617 [0.588, 0.667] |
| Elo, tuned | K = 12, tuned on the tuning period | 0.2433 [0.2376, 0.2469] | 0.6797 [0.6683, 0.6870] | 0.591 [0.509, 0.669] |
| Elo, default | K = 32, start 1500, untuned | 0.2421 [0.2303, 0.2503] | 0.6773 [0.6536, 0.6937] | 0.591 [0.509, 0.677] |
| Glicko-2, tuned | tau 0.3, initial RD 350, rating period 14 days, volatility 0.06, tuned on the tuning period | 0.2303 [0.2189, 0.2376] | 0.6530 [0.6295, 0.6675] | 0.631 [0.577, 0.691] |
| Glicko-2, default | tau 0.5, initial RD 350, rating period 7 days, volatility 0.06, untuned | 0.2310 [0.2180, 0.2410] | 0.6544 [0.6277, 0.6745] | 0.624 [0.551, 0.698] |
| TrueSkill, tuned | beta 1 x 25/6, tau 0 x 25/300, draw probability 0, tuned on the tuning period | 0.2252 [0.1963, 0.2465] | 0.6484 [0.5822, 0.6955] | 0.671 [0.599, 0.754] |
| TrueSkill, default | beta 1 x 25/6, tau 1 x 25/300, draw probability 0, untuned | 0.2254 [0.1966, 0.2460] | 0.6487 [0.5832, 0.6953] | 0.664 [0.593, 0.735] |
| Higher-rated team wins | hard call from the tuned engine's expected share | 0.3087 [0.2403, 0.3864] | not defined | 0.691 [0.614, 0.760] |
| Coin flip | 0.5 for every match | 0.2500 (constant, no interval) | 0.6931 (constant, no interval) | 0.500 (constant, no interval) |

Log loss is not defined for Higher-rated team wins: hard 0 or 1 calls make it infinite on any miss.

Coin flip: constant, no interval. The metrics are the same on any set of matches (Brier 0.25, log loss ln 2, accuracy 0.5), so there is nothing to resample. The paired difference below still has an interval.

Paired Brier difference against the tuned engine, on the common set: the row's Brier minus the engine's, paired by match (positive favours the engine), for every row except the tuned engine itself. 95% intervals resample league nights (2000 resamples, seed 20261114). For information only; no success criterion depends on them (criterion 1 is decided on the paired set).

| Model | Brier difference, row minus engine | 95% interval |
|---|---|---|
| Engine, default config | +0.0081 | [-0.0032, +0.0169] |
| Elo, tuned | +0.0220 | [+0.0124, +0.0320] |
| Elo, default | +0.0207 | [+0.0096, +0.0323] |
| Glicko-2, tuned | +0.0089 | [+0.0015, +0.0161] |
| Glicko-2, default | +0.0096 | [+0.0030, +0.0150] |
| TrueSkill, tuned | +0.0038 | [-0.0163, +0.0170] |
| TrueSkill, default | +0.0040 | [-0.0160, +0.0173] |
| Higher-rated team wins | +0.0873 | [+0.0256, +0.1537] |
| Coin flip | +0.0286 | [+0.0173, +0.0413] |

### Fold 4

Criterion measures in this fold, with 95% night-bootstrap intervals within the block. The bars are shown for information only; they decide nothing here.

| Criterion | Bar | Result | Bar met in this fold |
|---|---|---|---|
| Engine not worse than tuned Elo on holdout Brier | 95% CI of (Elo minus engine) Brier lower bound above -0.005 | 0.0128 [0.0005, 0.0218] (n_paired = 166) | yes |
| Holdout point-share bias, favourite's side | \|bias\| <= 0.015 | -0.0383 [-0.0671, -0.0131] | no |

Baselines: every row is scored on the same matches, the tuned engine's scored test-block matches that every row also scores (166 matches, 6 league nights). Only the tuned engine row is calibrated. Intervals are 95% cluster-bootstrap intervals, resampling league nights (2000 resamples, seed 20261114).

| Model | Settings | Brier | Log loss | Accuracy |
|---|---|---|---|---|
| Engine, tuned | side-out scale 0.868, a = 0.431, scheduled replays | 0.2244 [0.2129, 0.2385] | 0.6353 [0.6083, 0.6659] | 0.590 [0.531, 0.642] |
| Engine, default config | Config(): side-out scale 2.9, a = 1, scheduled replays | 0.2211 [0.2048, 0.2456] | 0.6282 [0.5908, 0.6819] | 0.608 [0.509, 0.650] |
| Elo, tuned | K = 16, tuned on the tuning period | 0.2372 [0.2302, 0.2445] | 0.6670 [0.6527, 0.6823] | 0.590 [0.522, 0.643] |
| Elo, default | K = 32, start 1500, untuned | 0.2352 [0.2228, 0.2459] | 0.6626 [0.6364, 0.6857] | 0.584 [0.520, 0.641] |
| Glicko-2, tuned | tau 1.2, initial RD 350, rating period 14 days, volatility 0.06, tuned on the tuning period | 0.2283 [0.2155, 0.2362] | 0.6479 [0.6217, 0.6638] | 0.623 [0.557, 0.696] |
| Glicko-2, default | tau 0.5, initial RD 350, rating period 7 days, volatility 0.06, untuned | 0.2296 [0.2138, 0.2376] | 0.6507 [0.6181, 0.6674] | 0.611 [0.563, 0.688] |
| TrueSkill, tuned | beta 0.75 x 25/6, tau 1 x 25/300, draw probability 0, tuned on the tuning period | 0.2356 [0.2120, 0.2495] | 0.6622 [0.6128, 0.6900] | 0.596 [0.534, 0.688] |
| TrueSkill, default | beta 1 x 25/6, tau 1 x 25/300, draw probability 0, untuned | 0.2313 [0.2121, 0.2431] | 0.6509 [0.6118, 0.6737] | 0.590 [0.544, 0.676] |
| Higher-rated team wins | hard call from the tuned engine's expected share | 0.4096 [0.3583, 0.4692] | not defined | 0.590 [0.531, 0.642] |
| Coin flip | 0.5 for every match | 0.2500 (constant, no interval) | 0.6931 (constant, no interval) | 0.500 (constant, no interval) |

Log loss is not defined for Higher-rated team wins: hard 0 or 1 calls make it infinite on any miss.

Coin flip: constant, no interval. The metrics are the same on any set of matches (Brier 0.25, log loss ln 2, accuracy 0.5), so there is nothing to resample. The paired difference below still has an interval.

Paired Brier difference against the tuned engine, on the common set: the row's Brier minus the engine's, paired by match (positive favours the engine), for every row except the tuned engine itself. 95% intervals resample league nights (2000 resamples, seed 20261114). For information only; no success criterion depends on them (criterion 1 is decided on the paired set).

| Model | Brier difference, row minus engine | 95% interval |
|---|---|---|
| Engine, default config | -0.0033 | [-0.0111, +0.0104] |
| Elo, tuned | +0.0128 | [+0.0005, +0.0218] |
| Elo, default | +0.0108 | [-0.0022, +0.0198] |
| Glicko-2, tuned | +0.0039 | [-0.0094, +0.0113] |
| Glicko-2, default | +0.0052 | [-0.0090, +0.0122] |
| TrueSkill, tuned | +0.0112 | [-0.0211, +0.0260] |
| TrueSkill, default | +0.0068 | [-0.0214, +0.0192] |
| Higher-rated team wins | +0.1852 | [+0.1401, +0.2402] |
| Coin flip | +0.0256 | [+0.0115, +0.0371] |

## Limits

- The tuning segment holds an earlier plan's holdout, which was seen before, and every fold tests on nights from the same league and players. These splits show variation over time within these data; they are not an independent test.
- Intervals resample league nights within each block. Players carry over from night to night, so nights are not fully independent and the intervals may be too narrow.
