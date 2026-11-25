# Review: SW-589 W0, tracker and validation pass

- **Status:** `[x]` approved by Tram on 2026-11-18
- **Branch:** `tram/sw-589-pr38-findings`
- **Range:** `sw589-W0-start..sw589-W0-review`

## Scope
Steps 2 and 3 of the plan:
- Add the tracking fields to `docs/pipeline_review_findings.md`.
- Validate all 69 findings against master@de32631.

No pipeline code changed.

## Commits
| Hash | Item | Summary |
|---|---|---|
| 2d17928 | setup | Ignore `.claude.local/` |
| 28b1024 | W0 | Resolution column + line for all 69 findings; new statuses; question tags |

## Files changed
- `.gitignore`: one entry.
- `docs/pipeline_review_findings.md`:
  - status vocabulary;
  - a Resolution column in the 4 tables;
  - 69 `**Resolution:**` lines;
  - SW-589 workstream tags on Q1–Q5.

## Validation results

### Status changes (3)
Already fixed by commits merged after the review:

| Finding | Was | Now | Evidence |
|---|---|---|---|
| M1 | ACCEPTED | FIXED | 1bfe9ba: `ThresholdClassifier.fit` uses `StratifiedGroupKFold` on patient ids (`train.py:90-121`) |
| H6 | ACCEPTED | FIXED | 1bfe9ba: production threshold folds now patient-grouped, same as tuning |
| L9 | ACCEPTED | FIXED | aac57b7: `_finite_index` / `_finite_tuple` return None for NaN landmarks |

### Confirmed FIXED (5)
- G4, G5, G7, G13: all in 0df6cd8.
- G1: storage part (35de2a3). Its open part is ReLIVE reports on the training-source path → W8.

### Partly done (status left as ACCEPTED, remaining part noted)
| Finding | What's on master | What's left |
|---|---|---|
| G1 | ReLIVE builder used for holdout reports (1bfe9ba) | Training-source path still LUCIA-only → W8 |
| G8, G24 | Holdout weight saving and re-scoring (ceab262, `holdout_estimation.py`) | No estimation stage / weight store for training runs → W9 |
| G9 | Trace-based model YAMLs (0df6cd8) | Commented out; not selected by unit → W3 |
| G10 | `eran_preprocess.py` + tests (40ef11a) | Not called by any stage → W4 |
| G18 | Git commit/branch on tracker (0df6cd8) | Holdout with no run starts at preprocessing → W2 |
| G22 | Rejects `unit: trace` + `allow_multi_trace: false` | `levels` not checked against `unit` → W3 |
| L5 | `threshold_fallback_` flag recorded | No warning → W1 |
| L12 | `pipeline.py` removed (1bfe9ba) | `sensor_fusion.py` still defaults to `late`, which raises → W1 |

### Still open
Every other ACCEPTED item still shows the problem at its location on master. Each one's Resolution line names its workstream. For the `sub-review` items, the failing test that reproduces the problem is written at the start of their workstream, as the 4b red step.

## Decisions I made (please check)
1. **Used "partly" without changing the status.** Partly done items stay ACCEPTED, with the done/left split written in the Resolution line, rather than inventing a "PARTLY FIXED" status.
2. **Changed three statuses based on code reading.** M1, H6 and L9 are marked FIXED from reading the code, before the regression tests exist. The tests come in W7 (M1, H6) and W1 (L9). If you'd rather keep them ACCEPTED until the tests are in, it's a one-line change each.
3. **Moved G2's credential concern to W1.** `notebooks/avichai/algo_data_download.ipynb` (cell 6) passes a literal 6-character `password=` alongside IAM auth. I haven't printed or copied the value. It may be a dummy, but it needs confirming, so it's in W1.
4. **Corrected two plan paths:**
   - The notebook moved to `notebooks/avichai/` (531cade); the plan said `notebooks/General/`.
   - `src/core/pipeline.py` no longer exists.

## Test evidence
- **Baseline `pytest`:** recorded after approval, see "Baseline" below.
- **W0 changed docs only,** so no new tests.

## Follow-up Jira tickets created
None. No item in W0 needed someone else's input yet.

## Questions for Tram
1. ~~Should M1, H6 and L9 stay FIXED now (decision 2), or go back to ACCEPTED until their tests land?~~ **Answered: back to ACCEPTED** (see Round 2).
2. Do you know whether the notebook's `password=` value is real? If not, I'll open a follow-up ticket for Avichai in W1.

## How to review
```bash
git log --oneline sw589-W0-start..sw589-W0-review
git diff --stat sw589-W0-start..sw589-W0-review
git diff sw589-W0-start..sw589-W0-review -- docs/pipeline_review_findings.md
```
The quickest read is the four tables in `docs/pipeline_review_findings.md`, which now carry a Resolution column.

## Round 2: Tram's feedback (2026-11-18)

**Feedback:** M1, H6 and L9 should stay ACCEPTED until their tests land (question 1 / decision 2).

**Change:**

| Hash | Item | Summary |
|---|---|---|
| d3d33a4 | M1, H6, L9 | Status back to ACCEPTED. The fix commit stays in each Resolution line as evidence, with "FIXED once test lands" → W7 (M1, H6) / W1 (L9) |

**Rule from now on:** a finding is FIXED only once a regression test proves it. "Already fixed on master" items get the test in their workstream, then flip to FIXED. The plan is updated (update 6).

**Status changes in W0 are now zero.** The validation pass changes no statuses. The 3 "already fixed" items are noted in the Resolution column only.

**Still open:** question 2 (is the notebook's `password=` value real?).

**Review:** `git show d3d33a4` (docs only, 9 lines).

**Note:** the `sw589-W0-review` tag was missing from round 1. It's now created at d3d33a4. Round 1 ended at 28b1024.

## Baseline (recorded at W1 start, 2026-11-18)

**Baseline `pytest` on master@de32631** (2026-11-18, local Windows venv, no torch): **386 passed, 3 failed, 6 xfailed (all SW-568), 4 collection errors.** These existed before SW-589; W0 changed docs only.

| Problem | Cause | Kind |
|---|---|---|
| `test_balanced_rf_tuning.py`, `test_easy_ensemble_tuning.py`, `test_xgboost_tuning.py` don't import | `FeatureTuningInput` in `balanced_rf_tuning.py` has `positive_labels` (with a default) before `feature_run_id` (no default). Added in 1bfe9ba (2026-11-09). Every tuning module fails to import | **Real bug on master** |
| `test_protocol6_con_fact.py` doesn't import | Imports `scripts.protocol6_con_fact`, which isn't on master (test added in cf21806) | **Real: missing file on master** |
| `test_resolve_run_dir_accepts_a_home_relative_lucia_runs_path` | `WinError 1314`: creating a symlink needs admin rights on Windows | Environment only |
| `test_unused_study_group_is_left_out[NOT_LC-L-RAD]` | Got `{L-RAD, LC}`, expected `{LC, NOT_LC}` | Not yet known: check on the VM/Linux |
| `test_visualization_skips_unavailable_optional_report_inputs` | `assert False` at line 217 | Not yet known: check on the VM/Linux |

The workstream gate compares against this: no new failures, and the passed count only goes up.
