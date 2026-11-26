# W8 review — Reports (Jira SW-601)

Branch: `tram/sw-589-preprocess-features` · start tag `sw589-W8-start` (faa731b)
Jira: SW-601 In Progress, SW-604 In Progress, SW-605 In Progress.

W8 is split into groups with a stop for review after each. This note covers
**Group A** (the two runner/stage bugs that don't change report methodology).
The report-number findings (M14, M15, M16, M17, M18, L17) and the rest follow
in later groups; M16/M17/M18/L17 need Eran's methodology decision first.

## Group A — commits

| Commit | Ticket | Change |
| --- | --- | --- |
| 1c19711 | SW-605 | Resume the features stage from the latest completed feature run |
| c2b0fae | SW-604 | Build one confounding report per trained model |

### SW-605 — `--resume` always re-ran the features stage
`should_skip_stage` checked `REQUIRED_ARTIFACTS['features']` at the base-run root
(`features/train_validation_set.pkl`, ...). Feature files live under
`feature_runs/<id>/features/` since the feature-run split, so those paths never
exist and the stage never skipped. Now the features resume check resolves the
artifacts through the recorded feature-run lineage (latest completed run) and
accepts an extract-only run when the plan needs only `features_all` (holdout
without training). The stale base-run `features` entry is removed from
`REQUIRED_ARTIFACTS`. No change to results; it only stops re-extracting on resume.

### SW-604 — reports covered only the last model
`run_model_reports` (source `training`) built from `TrainingResult.output_dir`,
the last model trained, so other `selected_models` got no report. It now fans
out over `TrainingResult.model_results` — one report per model, each under its
own model-run directory. First report keeps the original artifact keys; later
models get suffixed keys. The aggregate return still points at the last model.
A standalone report run (no in-process training result) is unchanged.

## Test cases

| Ticket | Test | File | Covers | Red before? |
| --- | --- | --- | --- | --- |
| SW-605 | `test_resume_skips_features_from_completed_feature_run` | tests/unit/test_pipeline_runner.py | resume skips features when a completed feature run exists under `feature_runs/<id>/` | **Yes** (old: never skips) |
| SW-605 | `test_resume_extract_only_run_does_not_satisfy_full_split` | tests/unit/test_pipeline_runner.py | extract-only run can't skip a full-split plan, can skip an extract-only plan | **Yes** (old: `extract_only` kwarg absent) |
| SW-605 | `test_resume_reruns_features_when_feature_run_missing` | tests/unit/test_pipeline_runner.py | status=completed but no feature run recorded → don't skip (guard) | No (guard, green on old) |
| SW-605 | `test_second_runner_execution_resumes_all_verified_stages` (updated) | tests/unit/test_pipeline_runner.py | the e2e resume path now writes the real feature-run layout | n/a (existing, rewired) |
| SW-604 | `test_run_model_reports_builds_one_report_per_trained_model` | tests/unit/test_model_reports_stage.py | two `model_results` → two reports, two excel artifacts, two manifest entries | **Yes** (old: builds only the last) |

4 new tests + 1 rewired. 3 red-before (the two guards aside).

## Suite

- `test_pipeline_runner.py` 11 passed · `test_model_reports_stage.py` 20 passed
- `test_holdout_stage.py` + `test_v2_pipeline_run.py` + the two above: 53 passed together
- `test_pipeline_e2e.py`: 30 passed, 1 pre-existing failure
  `test_unused_study_group_is_left_out[NOT_LC-L-RAD]` — fails identically on the
  pre-W8 code (baseline, likely SW-611 `control_group` handling), not from W8.

---

# Group B — unambiguous report-number bugs (approved)

Approved: Group A on 2026-11-26 (tag `sw589-W8A-approved`).
Approved: Group B on 2026-11-26 (tag `sw589-W8B-approved`, findings commit `bdb8eef`).

Two report-number findings that are clear bugs (no methodology judgment). M15
was reclassified out of Group B into Group C: how to combine a patient's
several visits (majority vote? keep separate?) is a methodology decision that
changes numbers, so it waits on Eran with M16/M17/M18/L17.

| Commit | Finding | Change |
| --- | --- | --- |
| 66b99ca | M14 | Fix the inverted rank-biserial correlation sign |
| c1ae7a6 | M19 | Stop the High Risk Summary leaking prevalence into later models |

### M14 — RBC sign inverted
`_mann_whitney_rbc` computed `rbc = 1 - 2U/(n_a*n_b)`. With scipy's
`mannwhitneyu(a, b)`, U counts a>b pairs, so that form is negative when group A
scores higher — the opposite of the report legend ("Positive means group A tends
to have higher scores"). Switched to the Kerby form `2U/(n_a*n_b) - 1`.
Magnitudes and p-values (|RBC|, bands) are unchanged.

### M19 — prevalence leaked across models
In `build_lucia_confounding_factor_report`, the per-model loop's High Risk
Summary used `for risk_label, risk_flag, prevalence in [...]`, shadowing the
function's `prevalence` parameter. After the first model it stayed 0.002, so
later models' per-patient PPV/NPV used 0.002 regardless of config. Renamed the
loop variable to `risk_prevalence`. The per-band values (0.01 / 0.002) are
unchanged.

This commit also adds a **reusable synthetic fixture** for the report builder
(`write_confounding_report_inputs` in `test_lucia_confounding_report.py`): a
16-patient eCRF + patient-level statistics CSV that drives the real builder
end-to-end. Group C (M15/M16/M17/M18/G21/L17/G1-open) will reuse it.

## Test cases — Group B

| Finding | Test | File | Covers | Red before? |
| --- | --- | --- | --- | --- |
| M14 | `test_rbc_is_positive_when_group_a_scores_higher` | tests/unit/test_feature_and_patient_score_report.py | A higher → rbc = +1 | **Yes** (old: -1) |
| M14 | `test_rbc_flips_sign_with_group_order` | " | swapping A/B flips the sign | **Yes** |
| M14 | `test_study_group_reference_row_is_positive_when_lc_scores_higher` | " | LC-vs-NOT_LC reference row reads positive | **Yes** |
| M19 | `test_prevalence_is_not_leaked_across_models` | tests/unit/test_lucia_confounding_report.py | two identical models → matching per-patient NPVs | **Yes** (old: 0.727 vs 0.999) |

4 new tests, all red-before. Suites: `test_feature_and_patient_score_report` 3,
`test_lucia_confounding_report` 1, with `test_model_reports_stage` 24 passed.

## Findings file (drafted, not committed — awaiting review)
M14 and M19 rewritten to FIXED current-state form (original text + Where line
dropped), per the findings-file rule. Will commit after your review.

---

# Group C′ — non-Eran-blocked report work (item 1 at review)

The Eran-blocked methodology items (M15, M16, M17, M18, and the greyzone/age/CI
parts of L17) are drafted as questions for Eran in
`.claude.local/notes/eran-questions_SW-589_report-methodology.md` (sent via Tram
2026-11-26). In parallel, the report work that needs no methodology call is being
done as Group C′, each item with its own review stop.

## Item 1 — L17 mtime sub-item (standalone bug)

| Commit | Change |
| --- | --- |
| b8c9813 | Resolve the patient-level statistics CSV by selection mode, not mtime |

`resolve_patient_level_statistics_path`'s fallback (no explicit path, no
in-process training result) picked the newest CSV by mtime from all
`patient_level_statistics_*_vs_*.csv` in the run dir. A run dir can hold CSVs for
several selection modes, so when `mode_hint`/`test_selection_mode` is known the
newest file can be the wrong mode. The fallback now filters to CSVs whose encoded
mode matches `test_selection_mode` (new helper `_statistics_mode_matches`),
keeping mtime only as a tiebreaker among same-mode CSVs, and falls back to all
choices if none match (so a dir with only off-mode CSVs behaves as before). The
`training_result` and explicit-path branches are unchanged.

### Test cases — Group C′ item 1

| Test | File | Covers | Red before? |
| --- | --- | --- | --- |
| `test_resolve_patient_level_statistics_prefers_mode_over_mtime` | tests/unit/test_model_reports_stage.py | mode hint wins over a newer wrong-mode CSV | **Yes** (old: picks the newer L-RAD CSV) |
| `test_resolve_patient_level_statistics_falls_back_to_mtime_without_mode` | tests/unit/test_model_reports_stage.py | no mode hint → newest-by-mtime unchanged | No (guard, green on old) |

2 new tests, 1 red-before. Suite `test_model_reports_stage.py` 22 passed (was 20).

### Findings file (L17) — drafted, NOT written to the file yet
L17 is a bundle (greyzone boundary, labels, age, CIs, CSV-by-mtime). Only the
mtime sub-item is fixed now; the rest wait on Eran. Proposed note to add under
L17, to be written and committed after review (held out of the file for now
because the findings doc is being committed concurrently on this branch —
c66c53a landed a G10/G20 findings edit mid-turn):

> **Partially addressed.** CSV-by-mtime: FIXED — `resolve_patient_level_statistics_path`
> now resolves the patient-level statistics CSV by selection mode, with mtime only
> a same-mode tiebreaker (b8c9813; test `test_model_reports_stage.py`). Greyzone
> boundary, age-at-sample-date, and CIs remain open pending Eran.

### Environment note (this machine)
This Windows machine had no Python test env; set up `.venv` (torch-free scientific
stack) to run the red-before check. WSL2 unavailable (virtualization off in
firmware). Branch advanced to c66c53a (G10/G20) concurrently during the turn;
HEAD == origin, no overlap with the report change.

## Item 2 — G21 (confounding-report contract tests)

| Commit | Change |
| --- | --- |
| daed926 | Lock the holdout confounding-report pooling, labels and filename contract (test-only) |

The existing tests did not lock the holdout cluster path: `test_holdout_reports_build_one_report_per_level` replaces the builder with a fake, and `test_collapse_with_device_channel_keeps_one_row_per_trace` only checks one-row-per-trace and summed `times_tested`. Five new tests lock the four contracts the finding names, directly on `collapse_cluster_level_statistics` and the label/filename helpers (no end-to-end workbook build, no src change):

| Test | Covers |
| --- | --- |
| `test_collapse_weighted_pooling_sums_counts_not_averages_accuracy` | weighted pooling sums counts, recomputes `accuracy_rate` = correct/(correct+incorrect) = 160/400 = 0.40 (not the 0.3833 mean of per-holdout rates), weighted mean 0.50, pooled std 0.0751665, `n_holdouts` 3; a single-fold `test` set stays as that fold (`n_holdouts` 1) |
| `test_collapse_weighted_pooling_differs_from_mean_pooling` | weighted (0.40) vs `pooling="mean"` (0.3833) differ — guards the default against regressing to averaging |
| `test_greyzone_band_is_on_accuracy_rate` | `_categorize_accuracy`: 0.487 and 0.513 greyzone, 0.532 decisive; `_categorize_accuracy_rate` correct/incorrect/greyzone |
| `test_relive_patient_group_2_is_renamed_to_f3` | `_normalize_relive_patient_group`: 2/"2"→F3, 1→F1 |
| `test_holdout_confounding_filename_encodes_threshold_and_band` | `confounding_factors_cluster_th-manual0d475_48_52.xlsx` (tag + band round) |

G21 is a coverage finding, so there is no old code to be red against. Non-vacuousness proven by a **mutation check**: temporarily setting the weighted `accuracy_rate` to the averaged value made both pooling tests fail (0.3833 vs 0.40); reverted. Suite `test_model_reports_stage.py` 27 passed (was 22).

**Note for Tram:** the finding cites `docs/HOLDOUT_CLUSTER_CONFOUNDING_REPORT.md` as the contract, but that doc does not exist in the repo. The tests encode the contract from the finding's worked example + the code. Decide whether that doc should also be authored (separate task) or the finding's "Where" reference dropped.

**Findings file (G21) — drafted, NOT written yet (held for review):**
> **Status:** FIXED · **Verified:** yes
> **Resolution:** Five tests lock the holdout confounding-report contract on `collapse_cluster_level_statistics` and the label/filename helpers (`test_model_reports_stage.py`): weighted pooling sums counts and recomputes `accuracy_rate` from them (not averaging per-holdout rates), `n_holdouts` counts pooled rows, the greyzone band is on `accuracy_rate`, RELIVE class 2 → F3, and the workbook filename encodes the threshold tag and band. Non-vacuousness confirmed by a mutation check.

## Item 3 — G1 (open part) + Q5 (ReLIVE report from a training run)

| Commit | Change |
| --- | --- |
| fd26e43 | Build the ReLIVE confounding report from a RELIVE training run |

The storage split for G1 was already fixed (35de2a3) and the **holdout** path
already picks the ReLIVE builder when `study: relive` (1bfe9ba). The open part
(and question 5) was the **training** path: `run_model_reports` (source
training) always called `build_lucia_confounding_factor_report`, so a RELIVE
training run got a LUCIA report. `_build_one_report` now branches on
`resolved.study`: `relive` → `build_relive_confounding_factor_report` (F1/F3
fixed, so no `pos_label`/`neg_label`), else the LUCIA builder as before. The
ReLIVE builder writes to `output_path` and names HTML `<stem>_<model>.html`
(no `_report` suffix) and returns a frame, so the ReLIVE branch resolves
`built_path` from `output_path` and globs `<stem>_*.html` (the LUCIA branch is
unchanged — still uses the return value and `<stem>_*_report.html`). Labels
still flow from the stats filename (relive → F1/F3), so only the builder choice
and HTML glob differ. The `study` default is `lucia`, so non-RELIVE runs are
untouched.

### Test cases — Group C′ item 3

| Test | File | Covers | Red before? |
| --- | --- | --- | --- |
| `test_run_model_reports_builds_relive_report_when_study_relive` | tests/unit/test_model_reports_stage.py | study=relive training run calls the ReLIVE builder (not LUCIA), without pos/neg labels, and discovers `<stem>_<model>.html` | **Yes** (mutation: forcing the LUCIA branch gives lucia=1/relive=0) |

1 new test, red-before shown by a mutation check (there is no pre-change code to
stash cleanly because the fix also adds the import; forcing the LUCIA branch
reproduces the old behavior and the test fails). Existing training-path and
holdout tests unchanged. Suites together: `test_model_reports_stage` 28 +
`test_lucia_confounding_report` 1 + `test_feature_and_patient_score_report` 3 =
32 passed.

### Findings file (G1 open part + Q5) — drafted, NOT written yet (held for review)
> **G1 — Resolution (append):** The open part is fixed — model reports from a
> training run now build the ReLIVE report when `study: relive`
> (`run_model_reports` branches on study, mirroring the holdout path; fd26e43).
> Test: `test_model_reports_stage.py::test_run_model_reports_builds_relive_report_when_study_relive`.
> Drop the "Still open" line.
>
> **Q5 — answer:** Yes. The model-report stage calls
> `build_relive_confounding_factor_report` when `study: relive`, on both the
> holdout (1bfe9ba) and training (fd26e43) paths.

## Item 4 — Group D / L18 (WONTFIX)

| Commit | Change |
| --- | --- |
| e595308 | Mark the unit-report changes WONTFIX (intentional, confirmed) |

L18 flagged three changes from Avichai's `89b488d` rewrite. All confirmed
intentional by Tram (2026-11-26), no code change:
- default GCS bucket `nanose_clinical_data` → `nanose_ml`: correct, stays;
- `production_date` uniqueness warning removed: stays removed (the `unit_type`
  uniqueness warning is still present);
- 16-slot diagram: deliberately replaced by the 32-channel sensor summary.

## W8 status after Group D

Done / approved (non-Eran): **Group A** (SW-605, SW-604 — Jira Done),
**Group B** (M14, M19), **Group C′** (L17 mtime, G21, G1-open/Q5),
**Group D** (L18 WONTFIX). All committed and pushed on
`tram/sw-589-preprocess-features`.

**Only Group C remains, fully blocked on Eran** — M15, M16, M17, M18, and the
greyzone-boundary / age-at-sample-date / CI parts of L17. Questions sent via
Tram (`.claude.local/notes/eran-questions_SW-589_report-methodology.md`). SW-601
stays In Progress until these resolve.

SW-604/SW-605 are bug tickets tracked in Jira, not findings entries.
