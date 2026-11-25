# Review: SW-589 W2, run provenance and tracker

- **Status:** `[x]` approved by Tram on 2026-11-19 · **Jira:** SW-595 (Done)
- **Branch:** `tram/sw-589-pr38-findings`
- **Range:** `sw589-W2-start..sw589-W2-review` (4 commits)
- **Goal of W2:** every run records the code that produced it and every row it removed. No change to model numbers.

## Scope and new statuses

| Finding | Was | Now | What changed |
|---|---|---|---|
| G5 | FIXED (0df6cd8) | FIXED, **now with tests** | 2 regression tests: no base `features/` folder for new runs; old runs still load |
| G13 | FIXED (0df6cd8) | FIXED, **tests confirmed** | Tests already came with the fix; checked that they catch the defect. Docs only |
| G12 | ACCEPTED | **FIXED** | Audit counts remaining visits (`n_remaining_visits`, `n_visits` per set) |
| M11 | ACCEPTED | **FIXED** | GRID_CLEAN drops recorded in the audit with their reason |
| G18 | ACCEPTED | ACCEPTED, **question for Gilad** | Not changed; the intent is unclear (Question 1) |
| L13 | ACCEPTED | ACCEPTED, **question for Gilad/Avichai** | Not changed; it's a default decision (Question 2) |

4 of 6 done (2 FIXED, 2 confirmed with tests); 2 waiting on a decision.

## Commits

| Hash | Finding | Summary |
|---|---|---|
| a754233 | G5 | Add regression tests for the base features folder |
| a075961 | G13 | Record the existing regression tests as evidence |
| c11d02b | G12 | Count remaining visits in the removal audit |
| 1a06062 | M11 | Record GRID_CLEAN drops in the removal audit |

## Files changed (riskiest first)

1. **`src/core/grid_clean_preprocess.py`, `src/core/stages/preprocessing.py`** (M11): the GRID_CLEAN entry points take an optional `tracker`, and the stage passes its own. Rows kept and dropped are unchanged; only audit records are added. Without a tracker, behaviour is exactly as before.
2. **`src/core/removal_audit.py`** (G12): one new field on `RemovalEvent`, one new column in the events table, pivot and set summary. Audit output only.
3. **Docs:** `docs/pipeline_review_findings.md` (G5, G12, G13, M11 rows).
4. **Tests:** 2 new files, 1 extended.

## Test evidence

**Test cases per finding (10 new cases):**

| Finding | Cases | Test file | What they cover | Red before the change? |
|---|---|---|---|---|
| G5 | 2 | `test_v2_pipeline_run.py` | new run (and reloaded) has no base `features/` folder; old-layout run still resolves its features from `<run>/features` | New-run test fails on 0df6cd8^ (folder still created). Old-run test passes both times: it pins that old runs keep loading |
| G13 | 0 new | `test_v2_pipeline_run.py` (existing: `test_child_runs_record_their_own_commit_and_branch`, branch-capture test) | base, feature and model runs each record their own commit + branch on manifest and tracker | Existing child-run test fails on 0df6cd8^ with `KeyError: 'identity'`, so it already catches the defect |
| G12 | 5 | `test_removal_audit_visits.py` | event counts valid patient × visit pairs; count reaches events table and pivot; `None` without a visit column; `n_visits` per set; `n_visits` for files dropped before features | All 5 red |
| M11 | 3 | `test_grid_clean_drop_audit.py` | each of the 3 drop steps recorded with its reason and counts; no event when a step drops nothing; without a tracker the output is unchanged | 2 red (the recording tests); the no-tracker case passes both times |
| **Total** | **10** | | | Each fixed finding has **at least one** test that failed before its change |

How the pre-fix checks were done (G5, G13): the test file also imports functions that don't exist before 0df6cd8, so I ran just those tests from a temporary file against `v2_pipeline_run.py` at 0df6cd8^, then restored the file and deleted the temporary test.

**Full suite:**

| | Passed | Failed | xfailed | Collection errors |
|---|---|---|---|---|
| After W1 | 427 | 3 | 6 | 1 |
| W2 review | **437** (+10) | 3 (same) | 6 (same) | 1 (same) |

- **Invariants** (`tests/invariants`): 18 passed, 6 xfailed (SW-568). No XPASS.
- **System** (`tests/system`, synthetic end-to-end runs): 8 passed. The 1 baseline failure (`test_unused_study_group_is_left_out[NOT_LC-L-RAD]`) is unchanged.
- **Synthetic run, outputs checked:** one system run's directory, inspected after the run:
  - base, feature and model run manifests: `git_commit` 1a06062, `git_branch` `tram/sw-589-pr38-findings`;
  - tracker `git` section: same pair;
  - no base `features/` folder;
  - `n_remaining_visits` filled on all 6 tracker events.
  - GRID_CLEAN is off in the synthetic config, so its drop events don't appear in this run. A stage run with GRID_CLEAN enabled comes with the GRID_CLEAN fixes (G10/H4); until then, M11 is proven by the unit tests.

## Metric comparison
Not needed. W2 changes audit and provenance output only; no row, draw, threshold or prediction changes.

## Decisions I made (please check)
1. **G13: no new test.** The tests that came with the fix already cover base, feature and model runs, and they fail on the pre-fix code. Writing another would duplicate them.
2. **G5: one of the two tests passes before and after on purpose.** It pins that old-layout runs keep loading, which the fix had to preserve.
3. **M11: drops matched on `file_write_time` + `device_channel`.** The GRID_CLEAN steps renumber rows, so the tracker's usual index-based `record_filter` can't find what was dropped. Without those two columns, only counts are recorded.
4. **G12: `set_summary` counts visits over all rows in a set,** the same way it already counts patients and files there. Removal events count only VALID rows, matching their existing patient and file counts.
5. **G18 and L13 are not implemented.** Both need a decision I shouldn't guess (questions below).

## Follow-up Jira tickets created
None.

## Questions for Tram
1. ~~**G18:** what should "a holdout should start only at feature selection" change?~~ **Answered (Tram, 2026-11-19):** the reading below is correct; implemented in Round 2.
2. ~~**L13:** keep excluded override rows in `processed.pkl`, or keep the default?~~ **Answered (Tram, 2026-11-19): the default stays.** L13 is WONTFIX; see Round 3.
3. **Pending from W1:** L1 (validation draw rounding), also for Gilad.

## How to review
```bash
git log --oneline sw589-W2-start..sw589-W2-review
git diff --stat sw589-W2-start..sw589-W2-review
git diff sw589-W2-start..sw589-W2-review -- src/core/grid_clean_preprocess.py src/core/stages/preprocessing.py   # riskiest
git show <hash>                                                                                                   # one finding
```
The quickest read is the G5, G12, G13 and M11 rows in `docs/pipeline_review_findings.md`.

## Round 2 (2026-11-19): G18

**Decision (Tram):** G18 is clear as written. A holdout run's features stage should only extract features, because each holdout selects again on its own training patients.

| Hash | Finding | Summary |
|---|---|---|
| df4416d | G18 | Extract features only when a run trains holdouts |

**What changed:**
- `run_features(extract_only=True)`: writes `features_all.pkl`, `features_all_invalid.pkl` and `feature_columns.json` to a feature run ending in `_extract`, and records `extract_only` on its manifest and on the base-run entry. Skips the blind split, whole-pool selection, selector, train/test/estimation sets, stability and lab-window reports.
- `features_extract_only_for_plan(plan)`: true when the plan has holdout but not training. The runner passes the result to the features stage.
- `resolve_feature_run(allow_extract_only=...)`: the holdout stage and site stability (they read only `features_all`) pass `True`. Training doesn't, and gets a clear error on an extract-only run.
- `docs/HOLDOUT_TRAINING.md`: describes the new behaviour.
- Unchanged: `holdout --run-dir`, the shared preprocessing/clustering/extraction, and every training path.

**Files (riskiest first):** `src/core/stages/features.py` (new extract-only branch, +195 lines, mostly the persistence helper), `src/core/v2_pipeline_run.py` (run resolution), `src/core/pipeline_runner.py`, `src/core/stages/holdout.py`, `src/core/site_leading_features.py`, `tests/unit/test_pipeline_runner.py` (the 2 test fakes accept the new keyword).

**Test cases (6 new):**

| Finding | Cases | Test file | What they cover | Red before the change? |
|---|---|---|---|---|
| G18 | 1 | `test_holdout_extract_only.py` | extract-only writes the 3 extraction files, never calls selection (stubs raise), no selection files, `_extract` ID, manifest `extract_only` | Red (no `extract_only` mode) |
| G18 | 1 | `test_holdout_extract_only.py` | `features_all`, `features_all_invalid` and `feature_columns.json` are identical to a full run's, i.e. holdout gets the same inputs | Red (no `extract_only` mode) |
| G18 | 1 | `test_holdout_extract_only.py` | holdout resolves an extract-only run; plain resolution (training) raises "extract-only" | Red |
| G18 | 1 | `test_holdout_extract_only.py` | plan rule: holdout without training → extract-only; with training or without holdout → full | Red (no helper) |
| G18 | 1 | `test_holdout_extract_only.py` | the `holdout` command's plan runs preprocessing → clustering → features → holdout, with features extract-only | Red |
| G18 | 1 | `tests/system/test_holdout_extract_only_e2e.py` | real `holdout` runs on the same synthetic data, extract-only vs. old full features stage: **all 34 holdout output tables identical** (predictions, full results, patient stats, selected-features frequency, operating points, set tracking) | Parity test: it compares the new and old paths, so "before" is the full-features run |
| **Total** | **6** | | | |

Red check: on the code before this commit, the unit test file fails at import (`features_extract_only_for_plan` doesn't exist), and `run_features` has no `extract_only` parameter.

**Full suite:** **443 passed** (was 437), 3 failed (same baseline), 6 xfailed, 1 collection error (same).

**Metric comparison:** holdout results are identical by the parity test. Training runs are unaffected (full mode, unchanged code path).

**Decisions I made (please check):**
1. **Extract-only is chosen automatically by the plan** (holdout without training), not by a config key. A run that does both holdout and training still runs the full features stage.
2. **Site stability may also use an extract-only run,** because it reads only `features_all.pkl`.
3. **Stability and lab-window reports are skipped in extract-only mode.** They depend on the whole-pool selection pool.

**Found (not changed):** the runner's resume check lists the `features` artifacts at the old base-run paths (`features/train_validation_set.pkl` …, `REQUIRED_ARTIFACTS` in `pipeline_runner.py`). Since feature files moved to `feature_runs/<id>/features/` (G5), those paths never exist, so `--resume` always reruns the features stage instead of skipping it. That's safe (never skips wrongly) but wastes time. It's outside the findings list; should it go to W2 as well, or a ticket?

**Review:** `git show df4416d`

## Round 3 (2026-11-19): L13

**Decision (Tram):** the default `retain_excluded_override_rows: false` stays.

| Hash | Finding | Summary |
|---|---|---|
| 2940910 | L13 | Status WONTFIX (by design), with the reason in its Resolution line. Docs only |

**Reason recorded:**
- The site-period / blocked-unit excluded rows are marked invalid, so they never reach training.
- Each one is still recorded in the removal audit when the override marks it.
- Only the saved `processed.pkl` leaves them out.
- A run can set the key to `true` to keep them.

**W2 now:** all 6 findings resolved (G12, M11, G18 FIXED; G5, G13 FIXED with tests confirmed; L13 WONTFIX). Tests unchanged since Round 2 (443 passed).

**Stale `--resume` feature paths:** ticketed as SW-605 (not part of W2).

**Review:** `git show 2940910`
