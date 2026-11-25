# Review: SW-589 W6, splits, holdouts, leakage

- **Status:** `[x]` approved by Tram on 2026-11-23 · **Jira:** SW-599 (still In Progress: Jira unavailable, move to Done on retry)
- **Branch:** `tram/sw-589-training-defaults` (cut from the W4/W5 branch, since PR #52 isn't merged; PR 3, to rebase after PR 1 and PR 2 merge)
- **Range:** `sw589-W6-start..sw589-W6-review` (6 commits)
- **Findings text:** committed after approval.

## Scope and new statuses

| Finding | Was | Now | What changed |
|---|---|---|---|
| M2 | ACCEPTED | **FIXED** | Feature stability runs after the blind split |
| G14 | ACCEPTED | **FIXED** | Every features-stage holdout recorded: blind-test patient IDs by group, and the estimation set row by row with its reason |
| H1 | → G14 | **FIXED** (default → SW-606) | Runs without a blind test warn that validation is optimistic, and say so on the tracker |
| G16 | ACCEPTED (task) | **FIXED** (already worked) | End-to-end test: test-only patients and holdout test groups share one workflow |
| L1 follow-up | WONTFIX | WONTFIX + warning | Warn when a study group's training pool has fewer than 5 patients |

## Commits

| Hash | Finding | Summary |
|---|---|---|
| edf6faa | M2 | Run feature stability after the blind split |
| 823329f | G14 | Record every features-stage holdout on the tracker |
| 417b9d0 | L1 | Warn when a study group's training pool has fewer than 5 patients |
| d759c01 | H1 | Flag runs whose validation has no blind test |
| 7ff1d6b | G16 | Test that test-only and holdout test patients share one workflow |
| (uncommitted) | all | Findings text |

## Files changed (riskiest first)

1. **`src/core/stages/features.py`** (M2, G14, H1): stability input; membership and holdout summary on the tracker; the optimistic-validation warning and flag.
2. **`src/core/removal_audit.py`** (G14): `holdout_membership` table and a new sheet (and CSV) in the audit workbook. Older tracker pickles without it still export (empty sheet).
3. **`src/core/partition_testing.py`** (L1): `warn_small_training_pools`, called from `make_validation_drawer` (every partition method).
4. **Tests:** 5 new files.

## Test evidence

| Finding | Cases | Test file | What they cover | Red before the change? |
|---|---|---|---|---|
| M2 | 1 | `test_stability_after_blind_split.py` | with blind patients p6, p7, stability sees only p0–p5 | Red (p6, p7 were in the stability pool) |
| G14 | 3 | `test_holdout_membership.py` | each held-out row with set and reason (blind test, not selected, invalid); blind-test IDs by study group and estimation counts on the tracker; `holdout_membership` sheet in the exported workbook | All 3 red |
| L1 | 5 | `test_small_group_warning.py` | a 4-patient group warns; 5 doesn't; draws identical to the drawer built directly, for random, balanced_coverage and rotating | 1 red (the warning); the others pin that draws don't change |
| H1 | 2 | `test_validation_estimate_flag.py` | no blind test: warning + `optimistic` flag; blind test: flag says so, no warning | Both red |
| G16 | 2 | `tests/system/test_holdout_test_only_workflow.py` | synthetic `holdout` runs: kfold with 4 test-only patients (they get holdout −1, others ≥ 0; `operating_points.csv` has `test_holdout` and `test_only`); `mode: test_only` (only the test-only patients are tested; `test_only` scored, no `test_holdout`) | Pass both times: this already worked, the test pins it |
| **Total** | **13** | | | 7 red |

**Full suite:**

| | Passed | Failed | xfailed | Collection errors |
|---|---|---|---|---|
| After W5 group A | 518 | 3 | 6 | 1 |
| W6 review | **531** (+13) | 3 (same) | 6 (same) | 1 (same) |

## Metric comparison
None needed: no change to which rows train, which features are chosen or how models are fit. M2 changes the stability report (off by default); the rest adds records and warnings.

## Decisions I made (please check)
1. **G14:** blind-test patient IDs go on the tracker config; the full estimation membership goes in its own audit sheet, since it can have thousands of rows. The feature run manifest still keeps counts only, as before.
2. **G14:** the set is named `blind_test`, to keep it apart from the holdout stage's test groups (the finding's "notice the difference between holdout test and 'test'").
3. **G14:** "add the holdout dataset from features to reports" is covered by the audit workbook (the features stage's report). The model reports read holdout-stage results, which already tag their test populations (G16).
4. **H1:** a warning and a tracker flag only; the default `blind_test_percent` is left for the SW-606 meeting. Default runs now print this warning, which is intended.
5. **L1:** the check sits in `make_validation_drawer`, so every partition method and both callers (training and the partition test) get it.

## Bug tickets
SW-603 (in PR #52, In Progress), SW-604, SW-605 (To Do): not touched by W6.

## How to review
```bash
git log --oneline sw589-W6-start..sw589-W6-review
git diff sw589-W6-start..sw589-W6-review -- src/core/stages/features.py src/core/removal_audit.py
git diff docs/pipeline_review_findings.md   # uncommitted findings text
```
