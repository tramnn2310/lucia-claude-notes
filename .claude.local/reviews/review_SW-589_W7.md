# Review: SW-589 W7, training, threshold and tuning

- **Status:** `[x]` done, approved by Tram on 2026-11-26 · **Jira:** SW-600 (Done)
- **Branch:** `tram/sw-589-training-defaults` (PR 3: W6, W7, G3, G9)
- **Range:** `sw589-W7-start..sw589-W7A-review` (4 commits), then G9: b7472b6, 33ab0c2. The H6 and L15 tuning commits were dropped before pushing (Tram, 2026-11-25); backup tag `sw589-W7-before-drop`, same final files.
- **Findings text:** committed in 3e5a280 (2026-11-26); branch pushed.

## Group A: scope and new statuses

| Finding | Was | Now | What changed |
|---|---|---|---|
| M1 | ACCEPTED (fix in 1bfe9ba) | **FIXED** | Regression test added. Note: 1bfe9ba already came with two tests for it (fit side); the new test adds the validation side |
| H6 | ACCEPTED (fix in 1bfe9ba) | **WONTFIX** | Tuning scripts removed (78608d6); only the production fit remains |
| Q4 | open | **answered** | Yes: patient-grouped folds in production (1bfe9ba); the tuning copy is removed |
| G6 | ACCEPTED (waiting for Eran) | **FIXED** (verified) | Test proves per-trace training fits on traces; no code change. Eran's confirmation on a real run still welcome |
| L15 | ACCEPTED | **FIXED** | Tuning parts removed with the scripts; one xgboost pin (2.1.1) everywhere |
| M12, M13 | ACCEPTED | **WONTFIX** | Tuning scripts removed (unused; Gilad tunes with grid search). Requirements carried to Gilad's tuning-review ticket (key to fill in) |

## Commits

| Hash | Finding | Summary |
|---|---|---|
| 9415cf3 | M1 | Add a regression test for patient-grouped threshold folds |
| ded7c8e | G6 | Verify that per-trace training fits on traces |
| 78608d6 | M12, M13, H6, L15 | Remove the unused Optuna tuning scripts (search ranges kept in the message) |
| c5875e2 | L15 | Pin one xgboost version in every requirements file |
| b7472b6 | G9 | Default per-trace runs to the adapted models |
| 33ab0c2 | G9 | Keep device_channel when clustering is off |
| 3e5a280 | M1, H6, Q4, G6, L15, M12, M13, G9 | Findings text |

## Files changed (riskiest first)

1. **`requirements.txt`, `requirements_pipeline.txt`** (L15): xgboost 2.1.1 everywhere; scikit-learn 1.6.1 -> 1.5.2 in `requirements.txt`, capped `<1.6` in `requirements_pipeline.txt`. Changes the environment for anyone installing from `requirements.txt`.
2. **Removed** (M12, M13, H6, L15): 3 tuning scripts, 3 `src/core/*_tuning.py` modules, 3 tuning configs, 3 tuning test files, `optuna` from the requirements, tuning sections of 4 docs.
3. **Tests:** 3 new files (M1, G6, L15 pins).

Training, shipped model YAMLs and default configs are untouched.

## Test evidence

| Finding | Cases | Test file | What they cover | Red before the change? |
|---|---|---|---|---|
| M1 | 1 | `test_threshold_patient_folds.py` | 20 patients x 6 rows, 5 inner folds: no patient on both the fit and the validation side of a fold; every patient validated once | Fails on `train.py` before 1bfe9ba (temporary worktree): with the old `fit(X, y)` 13 to 18 of 20 patients were on both sides of each fold |
| H6 | 2 | `test_threshold_patient_folds.py` | tuning's threshold, fall-back flag and `min_pos_rate` equal production's (`min_pos_rate` 0 and 0.45) | Both red. Before, tuning seeded its folds with `random_seed + fold`: thresholds 0.6182 / 0.4772 vs training's 0.5493 in 4 of 5 outer folds |
| G6 | 3 | `test_training_fits_on_traces.py` | final fit of each permutation sees every training trace (3 per exam); a patient never in two splits per permutation; patient stats = exam votes (mean channel probability) aggregated | Pass before and after: a verification, nothing to fix |
| L15 | 2 | `test_requirements_pins.py` | every requirements file pins the lock's xgboost; scikit-learn pins stay below 1.6 | Both red |
| **Total** | **6** | | | 3 red; G6's 3 are a verification. The H6 (2) and L15 tuning (3) tests went with the scripts |

**Full suite** (both runs on 2026-11-25):

| | Passed | Failed | xfailed | Collection errors |
|---|---|---|---|---|
| W7 start (`sw589-W7-start`) | 548 | 3 | 6 | 1 |
| W7 group A | 557 (+9) | 3 (same) | 6 (same) | 1 (same) |
| After tuning removal (78608d6) | 544 (-13 tuning tests) | 3 (same) | 6 (same) | 1 (same) |
| + L15 pin | **546** (+2; the 2 new tests run, full suite not rerun: only requirements files changed) | | | |

The 3 failures and the collection error are the known ones (SW-611, the Windows symlink test, the visualization test, SW-610).

## Metric comparison
Not needed for group A: no shipped model, default config or training path changes. Tuning results can differ from before (H6: thresholds follow production's fold seed), but no model was re-tuned.

## Decisions I made (please check)
1. **M1:** kept the new test although 1bfe9ba already has two. Theirs check the fit side only; the new one checks the validation side and that every patient is validated.
2. **G6:** marked FIXED as "verified" with the test as proof, and noted that Eran's confirmation on a real per-trace run is still welcome.
3. **L15:** xgboost 2.1.1 (the lock file's), not 3.0.2, because the lock is the documented install and the tested environment.
4. **H6:** WONTFIX rather than FIXED: the tuning copy is gone, nothing was fixed.

## Tuning removal and L15 pin (2026-11-25)
- Gilad: the Optuna scripts are unused (grid search instead); a tuning review replaces them. Searched all 94 branches: nothing outside the scripts, their tests and docs used them. The `*_tuned_lucia_legacy` model YAMLs are the production defaults and stay.
- xgboost: 2.1.1 everywhere (lock, venv, tests). 2.1.1 crashes with scikit-learn 1.6.1 (checked in a clean venv), so `requirements.txt` moves scikit-learn 1.6.1 -> 1.5.2 and `requirements_pipeline.txt` caps it `<1.6`. Anyone who installed from `requirements.txt` (3.0.2 / 1.6.1) gets older versions on reinstall.
- Gilad's tuning-review ticket key: Tram fills in the `SW-___` placeholder in M12/M13 later.

## G9: defaults per run type (2026-11-26)
- **Status:** WAITING -> **FIXED** (verified). Gilad (2026-11-25, SW-606): per cluster keeps the legacy trio, per trace defaults to the adapted trio; a run is per trace when clustering is off (`features.unit: trace` only for testing). Follow-up questions kept in G9 (EE30/RF29/XGBd3 definitions, `*_trace_based_legacy`, provenance table format, model guidance per work case).
- **b7472b6:** `training.default_models.trace` = `random_forest_adapted`, `easy_ensemble_adapted`, `xgboost_adapted`. Only per-trace runs with an empty `selected_models` change; all 11 shipped configs resolve the same models as before. `test_default_models.py`: 18 cases, 3 red before.
- **33ab0c2:** bug found while checking b7472b6: every clustering-off run failed in training (`allow_multi_trace=True requires column 'device_channel'`), with or without G9, because the features stage kept `device_channel` only for `features.unit: trace`. It now uses `per_trace_rows`, the rule training uses. System test `test_clustering_off_run_trains_per_trace`: red before, passes after; the run trains the 3 adapted models and writes trace-level statistics.
- **Before/after** (synthetic, `features.unit: trace`, 40 patients per group, 3 permutations, mean test AUC): legacy BRF 0.479 / EE 0.553 / XGB 0.533 -> adapted RF 0.504 / EE 0.553 / XGB 0.555. Synthetic signal is weak; this shows the run works, not model quality.
- **Suite:** 547 passed after b7472b6 (known 3 failures + 1 collection error). After 33ab0c2, related tests: 129 passed, 1 known failure (SW-611).
- **Decision to check:** fixing the clustering-off bug inside G9 rather than a separate ticket, since it blocked the per-trace default Gilad described.

## Bug tickets
SW-603 (in PR #52), SW-604, SW-605 (planned for W8), SW-611 (with SW-606): none in W7 scope, not touched.

## How to review
```bash
git log --oneline sw589-W7-start..sw589-W7A-review
git diff sw589-W7-start..sw589-W7A-review -- requirements.txt requirements_pipeline.txt
git show --stat sw589-W7A-review~1          # the tuning removal
git diff docs/pipeline_review_findings.md   # uncommitted findings text
```
