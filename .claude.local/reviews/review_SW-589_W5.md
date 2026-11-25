# Review: SW-589 W5, feature extraction and selection

- **Status:** group A `[x]` approved by Tram on 2026-11-22 · group B waiting for decisions · **Jira:** SW-598 (In Progress)
- **Branch:** `tram/sw-589-preprocess-features` (same as W4; PR 2)
- **Range (group A):** `sw589-W5-start..sw589-W5A-review` (3 commits)
- **Findings text:** committed after approval.

## Group A: scope and new statuses

| Finding | Was | Now | What changed |
|---|---|---|---|
| G7 | FIXED (test open) | **FIXED**, with test | Regression test: filtered rows go to the estimation set |
| M3 | ACCEPTED | **FIXED** | Stability uses the run's selection settings; method failures are reported |
| L14 | ACCEPTED | ACCEPTED, **3 bugs fixed** | Median fill, quota size, unknown settings rejected. MI on bootstrap duplicates stays for group B |

## Commits

| Hash | Finding | Summary |
|---|---|---|
| 1854dfa | G7 | Add a regression test for filtered rows in the estimation set |
| f376997 | M3 | Measure stability with the run's selector settings and report failures |
| 4abb5ea | L14 | Fix the poolsel median fill and quota size, reject unknown settings |
| (uncommitted) | G7, M3, L14 | Findings text |

## Files changed (riskiest first)

1. **`src/core/feature_selection_poolsel.py`** (L14): the v1 redundancy step reads raw columns so the median rule runs; the quota pass stops at the pool size. Affects only `patient_mrmr_consensus` runs (relive_beta, type1_61, lucia550 mrmr_sites).
2. **`src/core/config/validate.py`** (L14): `patient_mrmr_consensus` settings checked against what the funnel accepts, for `features` and `holdout.selection`.
3. **`src/core/feature_stability.py`, `src/core/stages/features.py`** (M3): stability gets `selection_kwargs`; failures counted, warned, written and shown.
4. **Tests:** 4 new files.

## Test evidence

| Finding | Cases | Test file | What they cover | Red before the change? |
|---|---|---|---|---|
| G7 | 1 | `test_estimation_keeps_filtered_rows.py` | other study group, other sensor, other breath and an invalid row go to `estimation_pickle`, none to train/test | Red on the code before 0df6cd8 (only the invalid row reached estimation). Run in a temporary worktree, because the test file's imports changed since |
| M3 | 3 | `test_feature_stability_settings.py` | configured `correlation_threshold` and other settings reach the selector; a method failing in every permutation is warned about, counted, written to the raw JSON and shown in the report; no warning when all succeed | The 2 behaviour cases are red; all 3 fail on the old signature |
| L14 | 7 | `test_poolsel_edge_cases.py` | v1 redundancy step sees the median fill; quota pool capped at its size with 6 sections; unchanged with 4 sections; no scored replicate raises; `correlation_threshold` rejected for poolsel; `holdout.selection` settings checked; valid poolsel settings load | 4 red; the 4-section, no-replicate and valid-settings cases pin behaviour that had to stay |
| **Total** | **11** | | | 7 red |

**Full suite:**

| | Passed | Failed | xfailed | Collection errors |
|---|---|---|---|---|
| After W4 | 507 | 3 | 6 | 1 |
| W5 group A | **518** (+11) | 3 (same) | 6 (same) | 1 (same) |

## Metric comparison
- **Default runs (`ensemble`):** no change. M3 only changes the stability report, which is off by default.
- **`patient_mrmr_consensus` runs:** can change when a TSF column has under 10% missing values (median instead of zero before the redundancy step) or when features come from more than four sections (pool capped at its size). A before/after on a real poolsel run belongs with the group B decisions, which change the same selector.

## Decisions I made (please check)
1. **M3:** stability gives every method the run's `selection_kwargs`, as training does. A method that rejects them now shows up as a reported failure instead of a silent empty result.
2. **M3:** `validate` stays off in stability. It only logs a CV score, so the selection is the same and it saves time.
3. **M3:** a failing method is warned about and reported, not raised, so one broken method doesn't stop the run.
4. **L14:** unknown poolsel settings are rejected at config load, not renamed. `correlation_threshold` and `corr_threshold` have different defaults (0.9 and 0.93), so a silent rename would change results.
5. **L14:** only the redundancy step gets the median fill. Scoring still uses the zero-filled frame; unifying the fills is G20 (group B).
6. **L14:** `recursive` having no ranking is left as is. The stability report then orders its features by selection order.

## Bug tickets
SW-603 (in PR #52), SW-604, SW-605: not touched by W5.

## Group B

Done: **H5** (Avichai confirmed yes 2026-11-25). Still waiting: M4, M5, L14 MI on bootstrap duplicates, G20 (Gilad, Eran).

### H5 (2026-11-25): TSF is the only extractor

| Finding | Was | Now | What changed |
|---|---|---|---|
| H5 | ACCEPTED | **FIXED** | `RobustSignalAnalyzerTSF` is the only extractor; `use_tsf_features` removed; default `tsf_interval_counts: [1]` |

- **Decision (Avichai, 2026-11-25):** keep only the TSF extractor, remove the original one. Default interval counts `[1]` (option 1) — reproduces today's features, renamed, so no default run changes its numbers (option 2, `[1,2]`, would add features and move results).
- **Equivalence proven:** `RobustSignalAnalyzerTSF(interval_counts=(1,))` gives each legacy feature `{section}_{desc}` as `{section}_n1_i0_{desc}`, same value. Regression test `test_default_extraction_renames_the_base_features_without_changing_values` (red before, green after).
- **Base class kept:** `RobustSignalAnalyzer` stays as the TSF parent and keeps its own unit tests (`test_feature_extraction_values.py`, `test_feature_invariants.py`) — those test the base methods, not the pipeline path, so they were unchanged.
- **Q3 resolved:** the config check that would have required `use_tsf_features=true` for poolsel is moot — poolsel always gets TSF names now. The `patient_mrmr_consensus requires use_tsf_features` validation and the holdout error hint were removed.
- **Configs:** `validation_only` → `[1]`; `feature_selection_mrmr_consensus`, `relive_beta_preprocess`, `type1_61` drop the flag and keep `[1,2]`; lucia550 comment + TYPE1_61 handoff doc reworded.
- **Full suite:** 541 passed, 6 xfailed. The one failure (`test_pipeline_e2e.py::test_unused_study_group_is_left_out[NOT_LC-L-RAD]`) and the one collection error (`test_protocol6_con_fact.py`, missing `scripts.protocol6_con_fact`) pre-date H5 — both reproduce on stashed pre-H5 src.

## Environment note (2026-11-25)
Group A ran on the Windows venv. H5 ran on a fresh Linux clone: new venv `.venv` (py3.12) from `requirements-lock-linux-py312.txt` minus torch/nccl; `pytest -p no:cacheprovider --continue-on-collection-errors -W ignore`.

## How to review
```bash
git log --oneline sw589-W5-start..HEAD
git show HEAD   # the H5 code commit
git diff docs/pipeline_review_findings.md   # uncommitted findings text (H5 FIXED)
```
