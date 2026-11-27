# Review: SW-589 W9, estimation stage, model weights and tools

- **Status:** group A at review (2026-11-27) · group B waiting for tickets (drafts below) · G15 after PRs 2–4 merge · **Jira:** SW-602 (In Progress)
- **Branch:** `tram/sw-589-estimation-tools` from master `b0e17b0` (PR 5)
- **Range (group A):** `sw589-W9-start..sw589-W9A-review` (2 commits)
- **Findings text:** G8, G23, G24 uncommitted, for your review.

## Group A: scope and new statuses

| Finding | Was | Now | What changed |
|---|---|---|---|
| G23 | ACCEPTED | **FIXED** | A JSON model card next to every weights file; `docs/MODEL_WEIGHTS.md` lists what is needed to reuse or recreate a model |
| G8 | ACCEPTED | **FIXED** | New `estimation` stage after training scores `estimation_pickle.pkl` with every saved model |
| G24 | ACCEPTED | **FIXED** | Storage = `model_weights/` + cards per model run (already uploaded by the results profile); scoring = G8 |

## Commits

| Hash | Finding | Summary |
|---|---|---|
| 5959236 | G23 | Save a model card next to every weights file |
| 8f04034 | G8, G24 | Add an estimation stage that scores the estimation set |
| (uncommitted) | G8, G23, G24 | Findings text |

## Files changed (riskiest first)

1. **`src/core/train.py`** (G23): `save_trained_model` now also writes `<weights>.json` (`write_model_card`, `load_model_card`, `model_card_path`). Runs for every saved model in training and holdout runs. The joblib file is unchanged.
2. **`src/core/pipeline_runner.py`** (G8): `estimation` added to `STAGE_ORDER` after `training`, depends on `training`; resume check; `_enabled` reads a missing stage as off, so runs frozen before this change still load.
3. **`src/core/stages/estimation.py`** (new, G8): the stage.
4. **`src/core/config/validate.py`, `src/core/stages/__init__.py`, `configs/pipeline/validation_only.yaml`**: stage registered, `enabled: false` by default.
5. **`docs/MODEL_WEIGHTS.md`** (new).
6. **Tests:** 3 new files.

## Test evidence

| Finding | Cases | Test file | What they cover | Red before the change? |
|---|---|---|---|---|
| G23 | 3 | `tests/unit/test_model_card.py` | each weights file gets a card; card fields match the model (feature order, threshold, metric, target, positive label, group mapping, versions) and the file is plain JSON; weights + card alone reproduce the training loop's validation scores with the columns shuffled | All 3 red (functions missing; no card written) |
| G8 | 8 | `tests/unit/test_estimation_stage.py` | off by default and planned right after training when on; asking for it alone brings in training; a run frozen without the stage still plans; scoring in card order per permutation; missing feature column is an error; summary keeps row identity, vote rate and greyzone/group labels; no model run is a clear error and marks the stage failed; resume skips only a finished estimation | All 8 red (module missing; run in a worktree at 5959236) |
| G8, G24 | 2 | `tests/system/test_estimation_e2e.py` | full synthetic run: every estimation row is scored (unused study group NOT_LC and the other breaths included), scores equal the saved weights on those rows, manifest row count, resume skips estimation; with `training.save_models: false` the stage is skipped with a warning and writes nothing | Both red (stage unknown) |
| **Total** | **13** | | | 13 red |

**Full suite:**

| | Passed | Failed | xfailed | Collection errors |
|---|---|---|---|---|
| master b0e17b0 | 475 | 1 | 6 | 1 |
| W9 group A | **488** (+13) | 1 (same) | 6 (same) | 1 (same) |

- The failure is `test_unused_study_group_is_left_out[NOT_LC-L-RAD]` (SW-611); the error is `test_protocol6_con_fact.py` (SW-610). Both on master.
- `-m invariant`: 18 passed, 6 xfailed. `-m system`: 12 passed (+2), 1 failed (same SW-611).
- **Combined with PRs 2–4** (throwaway merge of `preprocess-features` 7ff52eb, `training-defaults` 99b7c1b, `reports` 7f590ed and W9, 2026-11-27): **616 passed**, 1 failed (SW-611), 6 xfailed, 1 error (SW-610); the 13 W9 tests pass. The code merges cleanly; the only conflict is `docs/pipeline_review_findings.md` between PR 3 and PR 4. Breakdown: 475 master + 82 PR 2 + 30 PR 3 + 16 PR 4 + 13 W9.
- **Branch base (Tram, 2026-11-27):** W9 stays on master; rebase onto master after PRs 2–4 merge, then rerun and update these counts.
- **Synthetic run** (by hand, 40 patients per group, 3 groups): 192 estimation rows scored. Mean probability by group: L-RAD 0.07, LC 0.93, NOT_LC (never trained on) 0.11. Labels: 124 L-RAD, 57 LC, 11 grayzone.

## Metric comparison
Not needed. The card is an extra file and the stage is off by default; no shipped config plans a different set of stages and no model number changes.

## Decisions I made (please check)
1. **Label without a new threshold.** Each permutation votes with its own train-CV threshold (as in training). The summary label is `grayzone` when the vote rate is in `[low_acc, 1 - low_acc]`, otherwise the majority group. `low_acc` comes from `model_reports.low_acc`, so there is no new config key. The holdout script instead uses the validation operating point from `operating_points.csv`; a training run has no such file.
2. **Every row is scored,** including other sensor types and breaths. A model trained on one sensor type gives a number for another sensor's row, but it may mean little; `sensor_type`, `breath_number` and `aggregated_reason` are in the summary so they can be filtered. Should the stage score only the training sensor and breath by default? **Tram (2026-11-27): keep scoring every row; ask Avichai in the PR 5 description.** Superseded the same day by Avichai's answer, see Round 2.
3. **One output per model run, overwritten on a rerun** (`<model_run>/estimations/`), not a time-stamped folder. Model runs are already one per training.
4. **Only the latest model run per model** is scored when the stage runs on its own (`stage estimation --run-dir`).
5. **Own scoring loop** instead of `saved_model_inference.estimate_long`: that helper keys rows by `file_write_time`, but the estimation set can hold several rows per `file_write_time` (other sensors, channels). Rows are keyed by position (`row_index`) instead. The holdout helpers are unchanged.
6. **Old runs without cards:** the stage falls back to `feature_selector.json`'s feature names, which is what the existing e2e reload test uses.

## Things I noticed
- **SW-605 on master:** in the by-hand run, `--resume` reran the features stage. Estimation was skipped correctly. The fix is on the PR 4 branch.
- The `saved_model_inference.py` docstring says the feature-selector medians "were never applied in training". The features stage does apply them to all three sets (`impute_with_selector`), so the docstring is out of date. I didn't change it (it's outside W9).

## Round 2 (2026-11-27, Windows): which rows G8 scores

- **Status:** at review. Commit `49aaa88` (local tag `sw589-W9-G8r2-review`), **not pushed**. G8 findings text uncommitted.
- **Avichai:** by default score the training sensor type and breath; allow selected breaths; never other sensor types. He confirmed that invalid rows and unused study groups are still scored when they match.
- **Change:** the training selection comes from the model run's feature run (`resolved_feature_config.yaml`); new key `estimation.breath_numbers` (empty = training breaths, a list replaces them, unknown `estimation` keys rejected); manifest records sensor type, breaths and excluded rows; `row_index` still points into `estimation_pickle.pkl`.
- **Files:** `src/core/stages/estimation.py`, `src/core/config/validate.py`, `configs/pipeline/validation_only.yaml`, `docs/MODEL_WEIGHTS.md`, 2 test files.

| Cases | Test file | What they cover | Red before? |
|---|---|---|---|
| 1 | `test_estimation_stage.py` | only the training sensor and breath selected; invalid and unused-group rows kept | Yes |
| 1 | same | several breaths selected, never another sensor type | Yes |
| 2 | same | missing `sensor_type` / `breath_number` column is an error | Yes |
| 1 | same | `estimation.breath_numbers` empty by default and read from config | Yes |
| 5 | same | bad breath values rejected | Yes |
| 1 | same | no sensor type setting | Yes |
| 1 (rewritten) | `test_estimation_e2e.py` | default scores training sensor and breath; second sensor type never scored; `--set` rerun scores selected breaths, warns on an absent one, skips when none match | Yes (scored every row) |

The unit file didn't import before the change (`select_estimation_rows` missing). New: 11 cases; the W9 total goes from 13 to 24.

**Full suite on Windows:** 497 passed, 6 xfailed, 3 failed, 1 collection error. Known: SW-611 failure, SW-610 error. The other 2 failures (`test_delete_pipeline_run` symlink privilege, `test_visualization_stage` path separator) are Windows-only and also fail on `master` and `60f188f`. **Not yet run on Linux**; the combined PRs 2–5 run is not repeated.

**Please check:** a rerun overwrites `estimations/`; `--resume` skips a finished estimation even if the breaths changed (documented); a rerun that finds no matching rows leaves the earlier output files in place and records "skipped".

## Group B: follow-up tickets (drafts, not created)
See the chat message. G17 values (Avichai, Gilad), G19 database update (Avichai), G25 which dashboard (Lenny, Tony, Gilad).

## Bug tickets
- SW-603/604/605: done; not touched by W9.
- SW-611: still open (the known e2e failure).

## How to review
```bash
git log --oneline sw589-W9-start..sw589-W9A-review
git show 5959236 -- src/core/train.py
git diff sw589-W9-start..sw589-W9A-review -- src/core/pipeline_runner.py src/core/stages/estimation.py
git diff docs/pipeline_review_findings.md   # uncommitted findings text
```
