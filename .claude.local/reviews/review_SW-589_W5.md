# Review: SW-589 W5, feature extraction and selection

- **Status:** group A `[x]` approved by Tram on 2026-11-22 · group B: H5, M4, M5, L14 committed; M4, M5, L14 defaults wait for the campaign comparison; G20 waiting on SW-613 · **Jira:** SW-598 (In Progress)
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
| d18bd70 | G7, M3, L14 | Findings text |

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

Done: **H5** (Avichai confirmed yes 2026-11-25); **M4, M5, L14** (Eran answered 2026-11-25, all implemented as opt-in config options, default off, for the campaign comparison). Waiting: **G20** (Eran deferred the missing-value policy to Gilad and Avichai; split out to SW-613).

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

### M4, M5 (2026-11-25): two opt-in selection axes

Eran's reply set the rule: all four W5 group B items change which features are selected, so each ships as a config option that is off by default, to be compared on one campaign while the published runs stay reproducible. M4, M5 and L14 (below) are implemented on that basis; G20 waits on the missing-value policy (SW-613).

| Finding | Was | Now | What changed |
|---|---|---|---|
| M4 | ACCEPTED (sub-review) | ACCEPTED, **option implemented** | `pool_in_bootstrap` (default off): rebuild the ranking + pool inside each replicate from in-bag patients |
| M5 | ACCEPTED (sub-review) | ACCEPTED, **option implemented** | `patient_balanced_scoring` (default off): weight each row by 1/(patient row count) in the RF relevance and the redundancy |

- **Commits:** `a32c711` (code + test), `b200ac1` (findings text).
- **One file:** `src/core/feature_selection_poolsel.py`. The rank-gate-pool block is factored into `_rank_gate_pool` so the once-over path and the per-replicate path share it; `bootstrap_consensus` gained `patient_balanced` + a `pool_builder`; `ensemble_blend_scores` / `_abs_spearman` / `select_mrmr_soft` / `order_hybrid` take optional sample weights; new `inverse_patient_row_weights`, `_weighted_corrcoef`.

**Test evidence** — `tests/unit/test_poolsel_patient_and_pool.py` (8 cases):

| Case | Pins |
|---|---|
| defaults off == unflagged run | the two flags don't touch the default funnel |
| inverse-count weights | every patient's rows sum to the same total; mean weight 1 |
| weighted Spearman == plain on balanced rows | the weighting is a no-op when row counts are equal |
| `patient_balanced_scoring` on imbalanced cohort | changes the consensus |
| `pool_in_bootstrap` records flag + returns 6 | the per-replicate path runs |
| pool is the union of replicate picks | the pool is rebuilt, not the fixed 29 |
| `pool_in_bootstrap` changes the consensus | OOB/consensus differ from default |
| both flags on the signature | they are new (old signature would `TypeError`) |

- **Red check:** the flags are new parameters, so the old code can't accept them; the default-equivalence is proven by red/green — stashed `feature_selection_poolsel.py`, ran the default funnel on a synthetic cohort, and the selected features + consensus are byte-identical to the new code with the flags off.
- **Metric impact:** none on any shipped run — all shipped `patient_mrmr_consensus` configs leave both flags unset. Numbers move only when a flag is turned on.

**Decisions I made (please check):**
1. **M4 final pool:** when `pool_in_bootstrap` is on, the final greedy ordering runs over the union of features the replicates actually selected (ranked once on all training patients for the mRMR term), since there is no longer one fixed 29-pool.
2. **M5 scope:** weighting reaches the RF relevance (60% of the blend) and the Spearman redundancy. MI and F take no sample weight (Eran's caveat), so those two 20% slots stay unweighted, and the upstream ~900-column correlation gate stays unweighted (structural de-dup, and weighting it per fold is costly).
3. **Status:** both stay **ACCEPTED**, not FIXED — the option is built and tested, but whether to adopt it as the default is the campaign decision Eran asked for.

**Still owed / open from Eran's reply:**
- Give Eran what the 0.69 noise figure was measured on (cohort size, feature count, class ratio, shuffled labels vs synthetic) — likely Gilad.
- Campaign comparison to pick the M4, M5 and L14 defaults (both L14 de-dup methods are built).
- G20 missing-value policy (SW-613) and the M5 training-weight question → Gilad + Avichai.

### L14 MI duplicates (2026-11-25): opt-in bootstrap schemes

Eran offered two ways to stop the bootstrap's duplicate rows inflating the mutual-information term and said to build both and keep the better on a campaign. Both are implemented as one opt-in `bootstrap_scheme` on the selector.

| Scheme | What it does |
|---|---|
| `with_replacement` (default) | today's draw; duplicate rows in both selection and fit — byte-identical to before |
| `score_unique` (yours) | still draw with replacement, but select on each drawn patient once; duplicates stay only in the model fit |
| `rotating` (Eran's) | draw without replacement in rotating per-class slices; no duplicates, every patient left out the same number of times |

- **Commits:** `79a0e50` (code + test), `dae8cc3` (findings text).
- **Design points (please check):**
  1. The loop now splits the rows used for selection from the rows used for the model fit, so `score_unique` can de-duplicate selection while the out-of-bag AUC keeps its bootstrap character. In `with_replacement` the two are the same rows, which is why the default stays byte-identical.
  2. `rotating` picks its fold count from the cohort — capped so each per-class slice stays at/above the two-class out-of-bag floor (`min_oob_per_class=2`), because `n_bootstrap=15` on a small cohort would make every slice too small to score. On the real ~100-patient cohort the cap does nothing. Rotating then runs one replicate per slice.
- **Test:** `tests/unit/test_poolsel_bootstrap_scheme.py` (8 cases): default == unflagged; each scheme returns six features, scores, and moves the consensus; duplicating a patient's rows shown to change the MI-based blend (the mechanism); `rotating_oob_slices` leaves every patient out exactly once with both classes in each slice; unknown scheme rejected; the flag is on the signature.
- **Metric impact:** none on any shipped run (all leave `bootstrap_scheme` unset).
- **Status:** L14 stays **ACCEPTED** — the bug fixes (median fill, quota cap, unknown-setting rejection) are done, and the MI-duplicate options are built and tested; adoption is the campaign decision. Only `recursive` having no ranking is left as is.

## Environment note (2026-11-25)
Group A ran on the Windows venv. H5 ran on a fresh Linux clone: new venv `.venv` (py3.12) from `requirements-lock-linux-py312.txt` minus torch/nccl; `pytest -p no:cacheprovider --continue-on-collection-errors -W ignore`.

## How to review
```bash
git show 9cbfcab faa731b      # H5: code, findings text
git show a32c711 b200ac1      # M4, M5: code, findings text
git show 79a0e50 dae8cc3      # L14 schemes: code, findings text
```
