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

## Remaining groups

- **Group C (needs Eran's methodology decisions + shared fixture):** M15 (how to
  count a patient with several visits/sites), M16 (which multiple-testing
  correction), M17 (AUC on full cohort vs greyzone-omitted), M18 (default
  `roc_curve_set`), L17. Plus G21 (confounding-report tests) and G1 open part +
  Q5 (ReLIVE report from a training run), which reuse the fixture.
- **Group D:** L18 (production_date warning + default bucket).

SW-604/SW-605 are bug tickets tracked in Jira, not findings entries.
