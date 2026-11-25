# Plan: SW-589, work through the PR 38 review findings (v1)

- **Jira:** SW-589, "Implement changes from 'PR 38' file" (In Progress, due 2026-11-24, reporter Avichai Toubi)
- **Branch:** `tram/sw-589-pr38-findings` (from master at `de32631`)
- **Plan location:** `.claude.local/plans/` (git-ignored, local only)

## How this plan is used
- This file is the source of truth for the task. Claude implements only what the plan says.
- If anything changes (a finding turns out different, a new decision), update this plan first, then implement.
- Each section shows its status: `[ ]` todo · `[~]` in progress · `[r]` committed, awaiting Tram's review · `[x]` done (approved) · `[w]` waiting on a follow-up ticket.

## Context
SW-589 says: branch from current master, go through every item in "the PR 38 file", and for each one:
1. Validate its status.
2. Fix it if needed.
3. Record what changed.

The task is done when every item is resolved and a PR with the updates is complete. The ticket also asks for ideas that would help the team work better in this repo.

"The PR 38 file" is `docs/pipeline_review_findings.md`. Commit `de32631` turned it into a "Full review of the model pipeline" and added G19–G25. Existing statuses didn't change. It now has **69 findings** plus **5 questions for the author**:

| Group | Items | Status breakdown |
|---|---|---|
| Gilad's notes | G1–G25 | 5 FIXED (G1, G4, G5, G7, G13), 1 WONTFIX (G2), 19 ACCEPTED |
| High | H1–H6 | H1–H4 point to G14, G9, G11, G10. H5, H6 ACCEPTED |
| Medium | M1–M19 | all ACCEPTED |
| Low / Info | L1–L19 | all ACCEPTED, no Verified column |
| Questions | Q1–Q5 | open |

`docs/PR38.pptx` is a separate how-to deck for `scripts/publish_clinical_dataset.py`, not the findings list.

## Steps

### 0. Confirm scope with Avichai `[x]`
**Result (2026-11-18):** Avichai confirmed I implement the items marked for him and the items waiting for Eran too. Nothing in the file is status-only.

What this means for those items:
- **Avichai items (G8, G11, G19, H5):** I implement them. Use Avichai's existing work as the reference: branches such as `avichai/eran-reproduction-check` and `avichai/signal_humid_combination`. Ask him where that work lives when I need it.
- **Eran items (G6, G9, G10):** I implement them, but each still needs input from Eran:
  - G10: his module `src/core/eran_preprocess.py` can be recovered from commit `e1a9cbd`.
  - G9: needs his trace-based model results (run and metric).
  - G6: needs a finished model run to check.
  When I reach one of these items, follow the guideline in 0b: check master, open an assigned follow-up ticket if anything is needed, update the findings file, and keep working. Don't guess values.

**Defaults** (Avichai's answer didn't cover these; change them here if he says otherwise):
- G19–G25 are in scope (they're in the file).
- The task-style items (G15–G17, G23–G25) are in scope.
- Several PRs split by scope (see step 7), with one commit per finding or tightly related group (see "Commit strategy" in step 4). Confirm the PR split with Avichai.

### 0b. Guideline: items that depend on other members' work
Some items use or wait on someone else's work. **Follow this process when you reach such an item while implementing a workstream.** Don't do it all up front.

1. **Check whether their work is on master.** Run `git fetch` first, then check `origin/master`, not local branches:
   - Does the file or function exist? (`git ls-tree -r --name-only origin/master | grep <file>`)
   - Is it wired into the pipeline, or is it just present? (`git grep <symbol> origin/master -- src configs`)
   - Is there newer work on an unmerged branch? (`git log origin/master..origin/<branch>`, `git diff --stat origin/master...origin/<branch>`)

   Record what you find, with the commit hash and date.
2. **If it's on master and does what the finding needs:** build on it. Cite the commit in the Resolution line.
3. **If it's only on an unmerged branch, or a confirmation or input is needed from the owner** (which ref is final, values such as hyperparameters, a DB procedure, agreement on a default):
   - **Create a Jira ticket in the SW project, assigned to that person:**
     - Title: `SW-589 follow-up: <ID> - <what is needed>`.
     - Link it to SW-589 ("relates to").
     - Description: the finding ID and text, what was checked on master (hash and date), the branch involved, and the exact question or input needed.
   - **Don't wait.** Continue with the rest of the workstream, and with any part of this item that doesn't depend on the answer.
   - When the answer comes, finish the item.
   - Never guess values that the owner has to supply.
4. **When reusing their code:** bring it in with `git cherry-pick -x` or `git checkout <ref> -- <file>`, so authorship and the source commit stay in history.
5. **At the same time, update `docs/pipeline_review_findings.md`.** It's the single source of truth. Every status change above goes into the finding's Status and Resolution right away, in the same commit as the related work, or in its own small commit if there's no code change. Examples:
   - `WAITING (Eran, SW-xxx)`, with the Resolution line "checked origin/master@<hash> on <date>: <result>; asked: <question>".
   - When the answer arrives: switch to the real status (FIXED / INVALID / ...) and keep the ticket key in the Resolution line.

**Items known to depend on others** (a starting point from the 2026-11-18 check; re-check step 1 when implementing):

| Item(s) | Owner | What may be needed | Found on 2026-11-18 |
|---|---|---|---|
| G10 + H4 (W4) | Eran | Eran's preprocessing integrated into the stage | `src/core/eran_preprocess.py` and `tests/unit/test_eran_preprocess_path.py` are on master (last change `40ef11a`, 2026-11-12, PR #50), but nothing in `src/` or `configs/` calls them. Newer unmerged work: `eran/new-trig-eran-cliff-share` (14 commits, 2026-11-08). Ask which version is final. |
| G9 + H2 (W3) | Eran | Per-trace vs. per-cluster model defaults, and their source | `balanced_rf_trace_based_legacy` and `xgboost_trace_based_legacy` exist under `configs/pipeline/models/` (`aab77d3`), but they're commented out in `validation_only.yaml`. Ask whether these are his final trace-based values and which run they came from. |
| G6 (W7) | Eran | Confirm training is per trace | Check on my own run. The ticket asks Eran to confirm against his run. |
| H5, M3–M5 (W5) | Eran | Final TSF / mRMR consensus code | TSF name check is on master (`feature_selection_poolsel.py:94, 830`). Unmerged: `eran/mrmr-consensus-master-ready` (2026-09-08), `eran/pipeline-v2-oof-tsf-mrmr-anchors-master-ready` (2026-09-10). |
| G8 + G24 (W9) | Avichai | The estimation work G8 refers to | No estimation stage on master (`src/core/stages/` has no estimation file). Branch unknown: ask. |
| G11 + H3 (W4) | Avichai | Intent and review | Limits are on master in `validation_only.yaml:65-68` but `enabled: false`. Implement from the finding; no ticket unless something is unclear. |
| G19 (W9) | Avichai | DB update procedure and access for 00300026 | Not code. |
| G15 (W9) | Avichai, Tony | Which manual to extend | `docs/PIPELINE_V2_USER_GUIDE.md` is on master (`6045305`). Avichai's unmerged `avichai/signal_humid_combination` changes it. |
| G25 (W9) | Lenny, Tony | Which dashboard version is current | `model_dashboard/` is on master. `tony/unified-model-dashboard-cloudbuild` (3 commits) isn't merged. |
| G3, M12, M13, W7 metrics | Gilad, Avichai | Agreement on defaults and metric-changing fixes | Raised through the PR (they're the fixed reviewers). Make a ticket only if a decision is needed before the PR. |

### 1. Local setup and branch `[x]`
- [x] Branch `tram/sw-589-pr38-findings` created from an up-to-date master (`<owner>/<ticket>-<desc>` format).
- [x] `.claude.local/plans/` created.
- [x] `.claude.local/` added to `.gitignore` (not committed yet; goes in the first commit).
- [x] Plan saved as `.claude.local/plans/plan_SW-589_pr38-review-findings_v1.md`.

### 2. Add tracking to the findings file `[x]` (W0, approved 2026-11-18)
Use `docs/pipeline_review_findings.md` itself as the tracker.
- Add a **Resolution** column to each section table (G / H / M / L).
- Under each finding, add a one-line `**Resolution:**` entry: the commit hash, files changed, test added, or why no change was needed.
- Extend "How to read this page". Keep ACCEPTED / WONTFIX / FIXED and add:
  - **WAITING (owner, SW-xxx)**: waiting on a follow-up Jira ticket assigned to that person (see 0b).
  - **INVALID**: the claim is shown to be wrong.
  - **NOT REPRODUCED**: couldn't trigger it; waiting for the reviewer to confirm.

### 3. Validation pass (read-only, all 69 items) `[x]` (W0, approved 2026-11-18)
For every finding, open its `Where:` locations on current master and classify it.

- **FIXED (G1, G4, G5, G7, G13):** confirm the fix actually landed on master, since several notes say "uncommitted". For example, G4 says `grid_clean_preprocessing.yaml` is removed, but G11 still cites it. Configs now live under `configs/pipeline/` (G18, G22), so re-resolve every `configs/pipeline_v2/` path.
  - These five also contain "still open" parts. Those are handled in step 4: G1 (ReLIVE report) in W8, G5 (reader fallback) in W2, G13 (via G18) in W2.
- **WONTFIX (G2):** confirm with the reporter. The open parts it lists (scheduler, Secret Manager password, hard-coded `DB_PASSWORD` in `algo_data_download.ipynb`) go into the team-notes list. A hard-coded password is a security issue, so remove it in W1 even though G2 is WONTFIX.
- **ACCEPTED:** confirm the problem still exists on master. Later merges (#49 parallel holdouts, #50 plateau norm) may have touched it. If it's already gone, mark it FIXED and cite the commit.
- **ACCEPTED but not verified in the repo:** reproduce before fixing. How depends on the Verified value:
  - `yes`: re-read the code on master and confirm it's unchanged.
  - `sub-review`, `(synthetic)` or `(real CSV)` (H4, H6, M3–M13, M15–M19): the evidence was tests in `/tmp` that aren't in the repo. Write a failing test in `tests/` that goes through the real code path.
  - `no` (G6, G8, G9, G10): G6 is itself a check (is training done on traces?). G8, G9 and G10 are feature requests, so confirm the gap still exists.
  - `—` (G15–G19, G23–G25): these are tasks. Write a short definition of done for each and add it to step 4 before starting.
  - L1–L19 have no Verified column, so treat them like `sub-review`.
  - **Before deciding,** match the finding's conditions: config flags (`grid_clean` / `blip_filter` enabled), sensor types, data shape (for example, many traces per patient). A bug on a path that's off by default is still a bug.
  - **Pick one result:**
    - **Reproduced:** set Verified to `yes (<test or file:line>)`, then fix it in step 4.
    - **Already fixed by a later commit:** **FIXED**, citing the commit.
    - **INVALID:** the claim is shown to be wrong, with a code line that already handles the case or a passing test of the exact scenario. Record the evidence.
    - **NOT REPRODUCED:** tried, didn't happen, can't rule it out. Record the config, data and test. Don't change the code. Ask Gilad to confirm before closing.
- **Pointer items (H1–H4):** they're resolved together with their target finding, in the same workstream.

Commit the filled-in table on its own (`SW-589: Validate review findings against master`) so the reviewer sees the triage before any fixes.

### 4. Fix pass, by workstream `[~]`
Related findings touch the same code, so they're fixed together. Work through the workstreams in order: the low-risk ones come first, and the ones that change model numbers or depend on others come last. Each fix needs a test that fails before the change (when it's a bug).

#### Commit strategy
Two levels: **the workstream decides which branch and PR a commit goes into; the finding decides what goes into one commit.**

- **One commit per finding**, or per pointer group fixed together (e.g. G11 + H3, G13 + G18, M1 + H6). Each commit holds:
  - the code change,
  - the test that proves it,
  - that finding's Status and Resolution row in `docs/pipeline_review_findings.md`.

  The finding, its proof and its tracker update then land together and can be reverted together.
- **Never mix workstreams in one commit.** A commit belongs to exactly one workstream, so a workstream can move to a different PR without rewriting commits.
- **No single squashed commit per workstream.** It would hide which change fixed which finding and make `git bisect` useless when a metric moves. The workstream is still visible as a unit: its commits sit together in the history, and its PR is what reviewers see.
- **Per-workstream commits are allowed only for shared groundwork**, e.g. one "add test fixtures for preprocessing" commit at the start of W4, used by later finding commits.
- **Housekeeping commits are separate from behavior commits:**
  - the `.gitignore` change,
  - the step 2 tracker columns,
  - the step 3 validation table,
  - doc-only fixes.
- **Before every commit:** `pytest` passes, the diff contains only that finding's files, and nothing from `.claude.local/`, data, PKLs or notebook outputs.
- **Message format:** `SW-589 <ID>: <imperative summary>`, e.g. `SW-589 M19: Stop the loop variable shadowing prevalence`. The `SW-589` key lets Bitbucket link the commit to Jira. The body says what was wrong, what changed, and how it was tested. **No co-author or "Generated with" attribution lines** (Tram's rule).
- **Commit order:** the order of the workstream list, so each commit builds on fixed inputs.

**W1. Small, clear-cut bugs and docs** (Jira SW-594, Done) (no change to model numbers) `[x]` (approved by Tram 2026-11-19; note: `.claude.local/reviews/review_SW-589_W1.md`)
- L9: NaN up_trig/down_trig crashes detection. The fix is already on master (aac57b7), so only add the regression test, then set L9 to FIXED.
- L12: late fusion is the default but raises NotImplementedError.
- L5, L6: the 0.5 threshold is used silently. Add a warning.
- L3: seed NumPy as well as Python random.
- L4: aggregate TrainingResult keeps only the last model.
- L1, L2: partition sizes don't match their docs (fix the docs or the rounding).
- L16: `verify_partitions` is a no-op check (document it or make it audit the real splits).
- L19: stale refs in GRID_CLEAN_PREPROCESSING.md.
- G2 follow-up: remove the hard-coded DB password from the notebook.
- Found in W1 (not a finding; Tram decided 2026-11-19 to fix it here): the tuning modules don't import on master. In `FeatureTuningInput` (`balanced_rf_tuning.py`), `positive_labels` has a default and comes before `feature_run_id`, which has none (1bfe9ba). Move the field. Proof: the 3 tuning test files collect and pass. This unblocks the W7 tuning tests (H6, M12, M13, L15).

**W2. Run provenance and tracker** (Jira SW-595, Done) `[x]` (approved by Tram 2026-11-19; note: `.claude.local/reviews/review_SW-589_W2.md`)
- G13 + G18: git commit and branch on the tracker and on every run manifest.
- G18: holdouts start at feature selection. Decided with Tram (2026-11-19): the `holdout` command (no training in the plan) runs the features stage in **extract-only** mode. It writes `features_all.pkl`, `features_all_invalid.pkl`, `feature_columns.json`, the tracker and the manifest (`extract_only: true`), and skips the blind split, whole-pool selection, selector, train/test/estimation sets and stability. The runner's required artifacts become mode-aware. `resolve_feature_run` accepts an extract-only run for holdout; training on one fails with a clear error. Feature run id gets an `_extract` tag. Proof: a parity test (identical holdout results with and without whole-pool selection), a plan test, a training guard test, and the full suite. No model-number change.
- G5: keep the reader fallback for old runs. Already FIXED on master (0df6cd8); add a regression test (new runs have no base `features/` folder, old runs still load).
- G13: already FIXED on master (0df6cd8); add a regression test (commit and branch on the manifest and tracker). Same rule as M1/H6/L9: FIXED is confirmed by a test.
- G12: `n_remaining_visits`.
- M11: GRID_CLEAN drops recorded in the removal audit.
- L13: `retain_excluded_override_rows` default. Decided 2026-11-19: default `false` stays; L13 WONTFIX (2940910).

**W3. Config contract and defaults** (Jira SW-596, Done) `[x]` (approved by Tram 2026-11-20; note: `.claude.local/reviews/review_SW-589_W3.md`; G3/Q1 → SW-606, G9 → Eran)
- [x] G22: validate the per-trace keys at config load. FIXED in 66e81aa (30 test cases, 9 red before). Design (2026-11-19):
  - One resolver, `resolve_allow_multi_trace(config)` in `src/core/config/validate.py`, used by config validation, `training_settings_from_config` and `holdout_estimation.prepare_rows` (which today reads null as false).
  - Rows are per trace when `features.unit: trace` **or clustering is disabled** (clustering off passes SENSOR rows through, one per channel). Per-trace rows: `allow_multi_trace` true or null (null → true); false is rejected. Cluster rows (unit cluster, clustering on): false or null (null → false); true is rejected.
  - **Deviation from the finding text** (flag in the review): the finding says null must stay false when clustering is off. That would make holdout drop all but one channel per `file_write_time`, the failure the finding itself describes, so clustering-off keeps multi-trace on.
  - `model_reports.levels`: `patient` can't be combined with `cluster`/`trace` (different sources); `trace` needs per-trace rows; explicit `source: training` allows only `patient`, explicit `source: holdout` rejects `patient`. `source: auto` keeps its run-time check (the `holdout` command decides the source at run time).
  - Fix the missing trace-statistics hint in `model_reports.py`.
  - Proof: unit tests for every combination, every shipped config still loads, the old `test_explicit_multi_trace_setting_overrides_clustering_mode` now expects rejection.
- G3: new defaults, at most 5 default YAMLs. Agree the values with Avichai or Gilad. **Waiting:** to decide with Gilad and Avichai at the daily meeting on 2026-11-23 (SW-606, with Q1). Gilad: choose the defaults against every input in the run's `resolved_config.yaml`.
- H2 → G9: trace vs. cluster model defaults, with Eran's source recorded. **WAITING (Eran)** (2026-11-19): no per-trace models from Eran on any branch; the `*_trace_based_legacy` files are Gilad's (0df6cd8), no source. Questions for Eran in `daily-reports/2026-11-20.md`.
- Q1: where the shipped hyperparameters came from.

**G9 round (2026-11-25, on `tram/sw-589-training-defaults`, PR 3)** `[r]` (6543d7b, tag `sw589-G9-review`, 17 test cases, suite 548 passed; note `.claude.local/reviews/review_SW-589_G9.md`): Eran answered; Tram approved the approach.
- Mechanism: `training.default_models: {cluster: [...], trace: [...]}`; `training.selected_models: []` in `validation_only.yaml`. At config load an empty list is filled from the run type (G22 rule: trace when `features.unit: trace` or clustering off) and `training.selected_models_source` is recorded (`default_cluster` / `default_trace` / `run_config`). Both lists = today's trio (Eran's decision), so no run changes. Unknown models in `default_models` rejected at load.
- Provenance: a `provenance:` block beside `model:` in each model YAML (not hashed, so model hashes stay). Legacy trio: notebook; adapted trio: FO1 `adapted_final_all_breath` (TYPE1-61); trace_based_legacy: no recorded source, pending Gilad.
- Adapted trio stays optional; fix its comment in `validation_only.yaml`.
- Add `xgboost_depth2_mds` (XGBd2) from Eran's branch as an optional model.
- Tests: resolution per run type, explicit list wins, unknown model rejected, every shipped config resolves to today's list, model YAMLs keep their hashes, XGBd2 loads.
- Stays open: Gilad to confirm Eran's default (vs his adapted interim); EE30, RF29, XGBd3 definitions (Eran); trace_based_legacy keep/deprecate (Gilad); provenance table format (Eran); run-type rule (Eran); per-case guidance (Gilad, Eran); model name `balanced_random_forest_lucia_legacy` vs `balanced_rf_aggregated_tuned_lucia_legacy` (Tram asks Eran).

**W4. Preprocessing: GRID_CLEAN, Eran's preprocessing, blip filter** (Jira SW-597, In Progress) `[~]` (started 2026-11-20 on `tram/sw-589-preprocess-features`, cut from the PR 1 branch; rebase onto master after PR #52 merges)
- **Triage (2026-11-20):** the default path doesn't use GRID_CLEAN, Eran's preprocessing or the blip filter (all off), and `trigger_detection` is only called by those two paths, so most of W4 doesn't change default runs.
  - **A. Clear bugs, fix now** `[x]` (approved by Tram 2026-11-20; tag `sw589-W4A-review`, 7 commits + findings text, 26 test cases): M8, M9, M10, L10, L11, L7, L8, H3 (temperature override resets blip-rejected rows), and the H4 crashes (AUX rows, mixed sensor types, NaN temperature). M10 changes runs that filter sensor types (2 experiments).
  - **B. Need a decision:** G11 (turn the blip filter on by default: changes default runs; owner Avichai), M6 (plateau median scope), M7 (persist the median for inference).
  - **C. Waiting for Eran:** G10 (final `eran_preprocess.py` and the md Gilad asked for). The module is on master already, not wired in.
- G10 + H4: bring in `eran_preprocess.py` from `e1a9cbd`, call it from the stage, and make GRID_CLEAN finish a realistic run.
- M6, M7: plateau median scope; persist it for inference.
- M8: end-bound check on the cut.
- M9: search-window off-by-4.
- M10: keep AUX/INDEX rows.
- L10: `median_from_up_start_to_down_start: false`.
- L11: `first_descent` naming.
- G11 + H3: enable the blip filter with the limits and actually drop rows. Make sure the temperature override doesn't reset the rows to VALID.
- L7, L8: blip counts and repair-index checks.
- Q2: was this ever run end to end? Answered by the run-through in step 5.

**W5. Feature extraction and selection** (Jira SW-598, In Progress) `[~]` (started 2026-11-20, tag `sw589-W5-start`, same branch as W4)
- **Triage (2026-11-20):** default runs use `selection_method: ensemble` with `use_tsf_features: false`; `patient_mrmr_consensus` (poolsel) is used by 3 experiments (relive_beta, type1_61, lucia550 mrmr_sites).
  - **A. Fix now, no change to model results:** G7 test; M3 (stability uses the configured `selection_kwargs`, records method failures instead of hiding them); L14 bug parts (median imputation never fires, consensus all-1.0 when no replicate scored, quota pool over target, `correlation_threshold` vs `corr_threshold` crash); Q3 config check if Avichai/Gilad agree.
  - **B. Need a decision (change selection, so results):** M4 (pre-ranking inside the bootstrap), M5 (patient-level relevance/redundancy), L14 MI on bootstrap duplicates, G20 (one fill for selection and training), H5 (TSF naming always; owner Avichai).
- H5: keep the TSF naming convention so `patient_mrmr_consensus` works.
- M3: stability uses the configured selector and doesn't swallow errors.
- M4: pre-ranking inside the bootstrap.
- M5: patient-level relevance and redundancy.
- L14: poolsel edge cases.
- G20: imputation vs. selection fill.
- G7 check: estimation rows include the filtered rows.
- Q3.

**W6. Splits, holdouts, leakage** (Jira SW-599: to move to Done, Jira was unavailable on 2026-11-23) `[x]` (approved by Tram 2026-11-23; tag `sw589-W6-review`; note `.claude.local/reviews/review_SW-589_W6.md`) (started 2026-11-23 on `tram/sw-589-training-defaults`, cut from the W4/W5 branch because PR #52 isn't merged; tag `sw589-W6-start`)
- **Design (2026-11-23):**
  - **M2:** stability runs on the training/validation pool after the blind split, not on the whole selection pool. Changes the stability report only.
  - **G14:** the features section of the tracker records every holdout: blind-test patient IDs by study group, and the estimation set's membership (patient, file, why it's there: not selected for training, or invalid). The membership goes to a new `holdout_membership` sheet of the audit workbook. Called `blind_test` to keep it apart from the holdout stage's test sets.
  - **H1:** an unbiased estimate already exists: the holdout stage selects features inside every holdout (G18). The default `blind_test_percent: 0` is a defaults question (SW-606). Add a warning and a tracker flag when a run validates on the same pool its features were selected on (no blind test).
  - **L1 follow-up:** warn before drawing when a study group's training pool has fewer than 5 patients.
  - **G16:** definition of done: test-only patients and holdout test groups go through the same `holdout` workflow, each population is tagged and scored separately in the outputs, and `mode: test_only` runs the same path. Prove it with a synthetic end-to-end run.
  - None of these change model results.
- G14 + H1: record holdout membership; unbiased estimate when `blind_test_percent` > 0.
- M2: stability runs after the blind split.
- G16: test-set-only and holdout-test workflows share one path.
- L1 follow-up (decided 2026-11-19, Gilad + Tram): keep `_n_to_draw` as it is (cap at n, current rounding); L1 is WONTFIX. Instead, add a check before drawing: **warn when a study group's training pool has fewer than 5 patients** (Gilad: real groups have at least 5–10; ~100 patients in the real dataset). Warning only, no change to draws. Test: a pool with a 4-patient group warns, a 5-patient group doesn't, draws identical with and without the warning.

**W7. Training, threshold and tuning** (Jira SW-600) (changes model numbers; agree each one first)
- G6: verify training is per trace.
- M1 + H6: patient-grouped threshold folds, matching tuning. The fix is already on master (1bfe9ba), so only add the regression test (no `patient_id` in two inner folds), then set M1 and H6 to FIXED.
- M12: tuning doesn't search the clinical floors.
- M13: repeated CV / 1-SE rule.
- L15: tuning reproducibility; one pinned xgboost version.
- Q4.

**W8. Reports** (Jira SW-601)
- M14: RBC sign.
- M15: count patients, not visit/site rows.
- M16: confounder tests stratified, with multiple-testing correction.
- M17: AUC on the full cohort.
- M18: don't fit the threshold on test by default.
- M19: prevalence shadowing.
- L17: greyzone, labels, age, CIs, picking CSVs by mtime.
- L18: production_date warning, default bucket.
- G21: tests for the confounding report.
- G1 open part + Q5: build the ReLIVE confounding report when `study: relive`.
- L4 follow-up (found in W1): with `source: training` and several `selected_models`, `model_reports` reports only the last model (it uses `TrainingResult.output_dir`). Build one report per model run. Jira: **SW-604**.
- **SW-605** (added 2026-11-23, Tram): `--resume` always reruns the features stage. `REQUIRED_ARTIFACTS['features']` in `src/core/pipeline_runner.py` lists base-run paths (`features/train_validation_set.pkl`, ...) that haven't existed since feature files moved to `feature_runs/<id>/features/`. Check the latest completed feature run instead (and accept an extract-only run when the plan only needs `features_all`). Test: a resume after a completed features stage skips it; the existing resume test's fake stage writes the real paths. No change to results. Move SW-605 to In Progress when W8 starts, Done when its PR merges.

**W9. New stages, models, data and tools** (Jira SW-602)
- G8 + G24: an estimation stage that scores `estimation_pickle.pkl`, plus model weight storage (depends on G7).
- G23: the minimal parameters needed to recreate a model's weights. Feeds G24.
- G17: DCU, tempr and period overrides for ReLIVE.
- G19: update the SQL database with 00300026 (beta). This is a database change, not a code change. Confirm the procedure and access with Avichai before touching the DB.
- G25: integrate Lenny's model dashboard (`model_dashboard/`) as a pipeline tool.
- G15: pipeline user manual. Write it last so it reflects all the changes.

**Coverage check:** all 69 findings and Q1–Q5 are assigned above. FIXED items (G1, G4, G5, G7, G13) get validated in step 3, and their open parts are in W2, W5 and W8. G2 is validated in step 3, with its follow-up in W1. Pointers H1–H4 sit with their targets.

### 4a. Review checkpoint after every workstream `[ ]`
**Rule: after I commit a workstream, I stop. I don't start the next workstream until Tram explicitly approves the current one** (e.g. "W1 approved"). Silence, or approving a different workstream, isn't approval. This also covers steps 2 and 3, which together form checkpoint **W0** (tracker columns and validation).

**At the start of each workstream:**
- Tag the starting point with a local tag: `git tag sw589-W<n>-start`.
- Mark the workstream `[~]` in this plan.

**At the end of each workstream (the checkpoint):**
1. All of the workstream's commits are in, and the workstream gate (4b) passes.
2. Tag the end with a local tag: `git tag sw589-W<n>-review`. Tags aren't pushed, so they stay local.
3. Write a review note at `.claude.local/reviews/review_SW-589_W<n>.md`:
   - **Scope:** the findings covered, and each finding's new status (FIXED / INVALID / NOT REPRODUCED / WAITING + ticket).
   - **Commits:** a table of hash, finding and one-line summary.
   - **Files changed:** grouped by area, with the riskiest changes flagged first.
   - **Test evidence:**
     - new and changed tests,
     - **a per-finding table of test cases** (number of cases as pytest collects them, test file, what each covers, and whether it failed before the change), so Tram can see exactly what proves each fix (Tram's request, 2026-11-19),
     - xfail → pass transitions,
     - full-suite counts before and after,
     - the `-m invariant` / `-m system` results,
     - the synthetic run result.
   - **Metric comparison:** for workstreams that change numbers.
   - **Decisions I made,** and anything I wasn't sure about.
   - **Follow-up Jira tickets created.**
   - **Questions for Tram.**
   - **How to review:**
     - `git log --oneline sw589-W<n>-start..sw589-W<n>-review` lists the commits.
     - `git diff --stat sw589-W<n>-start..sw589-W<n>-review` shows the size.
     - `git diff sw589-W<n>-start..sw589-W<n>-review -- <path>` shows one area.
     - `git show <hash>` shows one finding.
4. Mark the workstream `[r]` (awaiting review) in this plan, then report in chat: the review note's path, a short summary, and "waiting for your approval".
5. **Stop.** No more code changes and no pushes until the workstream is approved.

**When Tram reviews:**
- **Approved:**
  - mark it `[x]`, with the approval date, in this plan and in the review note;
  - push the branch, if Tram says to;
  - start the next workstream.
- **Changes requested:**
  - Add the feedback to the review note.
  - Fix it with new commits. Don't rewrite or force-push history that has been reviewed.
  - Re-run the gate and move the review tag: `git tag -f sw589-W<n>-review`.
  - Add a "Round 2" section to the note listing only the new commits (`git diff <old review hash>..sw589-W<n>-review`), and stop again.
- **Rejected or redo:**
  - Discuss the approach first and update this plan.
  - Rolling back to `sw589-W<n>-start` rewrites history, so it needs Tram's explicit OK.

**Pushing and PRs:**
- Nothing is pushed until the workstream is approved.
- A PR (step 7) is opened only after every workstream in it is approved, and Gilad and Avichai's review comes on top of Tram's.

**Status markers:** `[r]` means awaiting Tram's review. It's added to the legend at the top.

### 4b. Test and evaluation method `[ ]`
This builds on the repo's existing conventions: `pytest.ini` markers `unit` / `invariant` / `system` / `slow`; synthetic data only, through the `make_clinical_frame` / `clinical_pickle` fixtures in `tests/conftest.py`, never real clinical files; and `xfail(strict=True)` to record confirmed defects, as SW-568 did.

**The red → green cycle for every bug finding:**
1. **Reproduce (red).** Write the smallest test that shows the defect through the real code path. Use the conditions the finding names (config flags, sensor types, traces per patient). Put it in the right folder:
   - `tests/unit/` for one function or stage,
   - `tests/invariants/` for a rule that must hold for any input (patient-level splits, row conservation, determinism),
   - `tests/system/` for end-to-end behaviour.
2. **Record it.** Mark the test `xfail(strict=True, reason="SW-589 <ID>: <defect>")`. If the fix will be in a later PR, commit the test like this, so the defect is documented in the suite and the findings file links to the test.
3. **Fix (green).** Change the code and remove the `xfail` marker. A strict xfail that suddenly passes fails the suite, so the marker can't be forgotten.
4. **Check the fix is complete and doesn't break anything else:**
   - edge cases the finding implies (NaN, empty groups, one row per class, more than one trace per patient);
   - the default path, with the flag off, is unchanged;
   - the full suite passes.
5. **If the defect won't reproduce:** keep the test as a passing regression test, and mark the finding INVALID or NOT REPRODUCED (step 3).

**What counts as proof, by type of finding:**

| Type | Examples | Proof it's done right |
|---|---|---|
| Crash or wrong value | L9, M8, M9, M14, M19, L10 | Unit test with a known input and a hand-calculated expected output. For M14, check the sign against a case where group A is clearly higher. |
| Silent behaviour → explicit | L5, L6, L12, M3 | Test asserts the warning or error is raised (`pytest.warns` / `pytest.raises`), and the message names the cause. |
| Data leakage / split integrity | M1, H6, M2, M4, M5, G14/H1, G16 | Invariant test: no `patient_id` appears in two folds or splits; the blind-test patients aren't used before the split. Test with several rows per patient, so a row-level split visibly fails. |
| Rows kept, dropped or recorded | G7, G11/H3, M10, M11, L13, G12 | Row-conservation invariant: rows in = kept + dropped by recorded reason. Every drop shows in the removal audit, and the counts match. For H3, check the temperature override doesn't flip blip-rejected rows back to VALID. |
| Tracker / provenance | G13/G18, G14, G12 | Run a small synthetic pipeline in `tmp_path` and assert the manifest and audit workbook have the new fields (commit, branch, holdout IDs, `n_remaining_visits`). |
| Config contract | G22, G3, G9 | `test_pipeline_config.py`-style tests: invalid combinations are rejected at load time with a clear message; valid ones resolve to the expected values; every shipped YAML in `configs/pipeline/` still loads. |
| Ported work (others' code) | G10, H5, G8 | Bring their tests along (e.g. `test_eran_preprocess_path.py`). Add a parity test: same input through their original function and the wired-in stage gives the same output. |
| Methodology (changes numbers) | W7, W8, M12, M13, M16, M17 | Unit test of the mechanism, plus a before/after comparison (below). Agreed by Gilad and Avichai in the PR. |
| Report correctness | M15, M17, M18, L17, G21 | Build the report from a synthetic run where the right answer is known (e.g. 26 patients with 30 rows must report n=26), and assert the table values. G21 tests the contract in `docs/HOLDOUT_CLUSTER_CONFOUNDING_REPORT.md`. |
| Docs / tasks | L19, G15, G23, G25 | Every path, command and config key in the doc exists. For G15, follow the manual from a fresh clone and it runs. G23: a model rebuilt from the saved parameters produces the same predictions. |

**Workstream gate** (must pass before the workstream counts as done):
- **Full suite:** `pytest` is green. Compare the counts with the baseline taken in step 5; the number of tests should only go up.
- **Invariants:** `pytest -m invariant` and `pytest -m system` pass.
- **xfails:** none of the SW-568 strict xfails changed state by accident. If one passes, that's an unexpected side effect; investigate it.
- **One synthetic pipeline run** through `scripts/lucia_pipeline.py`, with the stages that workstream touched, finishes without errors. For W4, run once with `grid_clean` and `blip_filter` enabled.

**Before/after evaluation for changes that move numbers** (PR 2, 3 and 4):
- **Same data both times:** the same published dataset ID and config, on master (before) and on the branch (after). Use fixed seeds and the same machine, so the only difference is the code.
- **Compare:**
  - row and patient counts at every removal step (from the audit);
  - selected features;
  - per model: AUC, sensitivity and specificity at the threshold, and balanced accuracy on validation and blind test;
  - report tables.
- **Expected vs. unexpected changes:**
  - For each change, write down beforehand which direction it should go. Example: fixing leakage (M1, M4) should lower optimistic validation scores.
  - A metric that moves in the wrong direction, or something changing that the fix shouldn't touch, means stop and investigate before continuing.
- **Recording:** put the comparison table in the PR description, and summarise it in the finding's Resolution line.

**Definition of done for one finding:**
- A test proves it: it failed before the fix and passes after. For tasks, the check from the table above.
- The full suite passes.
- The workstream gate passes.
- Its Status, Verified and Resolution in `docs/pipeline_review_findings.md` point to the commit and the test.

### 5. Verify `[ ]`
**Test environment (2026-11-18):**
- A venv at `C:\Users\82104\venvs\lucia`. It's outside OneDrive so the package files don't sync.
- Installed from `requirements-lock-linux-py312.txt` minus `nvidia-*` and `torch` (drive C: is nearly full). Tests that import torch are skipped, and the baseline says so.
- Plan to move to running workstream tests on the team VM over SSH (needs the host, the user and the key authorised).

- `pytest` passes after every commit, compared with the baseline: **386 passed, 3 failed, 6 xfailed, 4 collection errors** on master@de32631 (details in `.claude.local/reviews/review_SW-589_W0.md`). The tuning modules can't be imported on master (1bfe9ba dataclass field order), which blocks the W7 tuning tests (H6, M12, M13, L15) until it's fixed.
- **Pipeline runs:** do a small `scripts/lucia_pipeline.py` run after W2, W3, W4 and W6, and check the tracker/audit workbook and outputs show the changes.
- **W4:** do one run with `grid_clean` and `blip_filter` enabled. That answers Q2.
- **W7 and W8:** compare metrics before and after on the same feature run, and record the difference in the PR.
- **Before the PR:** every finding has a final Status and a Resolution, and every question has an answer or is marked open.

### 6. Team-improvement notes `[~]` (running list)
- Findings are tracked in a .md file with no owner or ticket column.
- Config paths drift (`configs/pipeline_v2` vs `configs/pipeline`), so file references go stale.
- Fixes get marked FIXED while still uncommitted.
- No stage test runs with `grid_clean` or `blip_filter` enabled.
- The findings file grew (G19–G25) after the ticket was written; it needs a version marker.
- G2 is still open: no weekly publish scheduler, DB password not in Secret Manager.

These go in the PR description or a Jira comment on SW-589.

### 7. Pull requests and ticket `[ ]`
**Decision: several PRs, split by how risky the changes are. Not one combined PR.**

One PR with all 69 findings would mix safe fixes with changes that move model metrics. It would be too large to review, and one disputed change (e.g. G3 defaults) would hold up everything else. It would also need every reviewer (Gilad, Eran, Avichai) to read all of it. Splitting by scope lets safe work merge first and sends each PR to the people who know that code.

**Reviewers:** **Gilad and Avichai are required reviewers on every PR.** Others are optional, added when the PR touches their work.

| PR | Contents | Changes model numbers? | Optional reviewers | Branch |
|---|---|---|---|---|
| **PR 1: triage and safe fixes** | `.gitignore`; step 2 tracker columns; step 3 validation table; W1; W2; G22 from W3 | No | none | `tram/sw-589-pr38-findings` (current) |
| **PR 2: preprocessing and features** | W4, W5 | Yes (inputs change) | Eran | `tram/sw-589-preprocess-features` |
| **PR 3: splits, training and defaults** | W6, W7, G3 + G9/H2 from W3 | Yes | Eran | `tram/sw-589-training-defaults` |
| **PR 4: reports** | W8 | Report numbers only | Eran | `tram/sw-589-reports` |
| **PR 5: new stages and tools** | W9 (G8/G24, G23, G17, G25, G15). G19 isn't code; it's tracked in the doc only | Adds outputs | Lenny, Tony | `tram/sw-589-estimation-tools` |

**Rules:**
- **Merge in order (PR 1 → 5).** Each later branch is cut from master after the previous PR merges (or rebased onto it). Every PR edits `docs/pipeline_review_findings.md`, so merging in order keeps conflicts small, and each PR is tested on fixed inputs.
- **No stacked PRs open at once,** except PR 4 (reports), which only depends on PR 1 and can run alongside PR 2 and PR 3 if time is tight.
- **Each PR description includes:**
  - the findings it closes, with their final status;
  - INVALID and NOT REPRODUCED items highlighted for Gilad;
  - before/after metrics on the same feature run (PR 2 and PR 3);
  - the dependency sources used, and the follow-up tickets with their state (from 0b);
  - the team-improvement notes, in the last PR.
- **Title format:** `SW-589 (PR n/5): <scope>`.
- **No Claude attribution:** no "Generated with Claude Code" line in any PR description (Tram's rule).
- **Waiting items** stay listed in the PR they belong to, as WAITING with their owner and ticket key. They don't hold that PR back.
- **Jira:** suggest to Avichai that PRs 2–5 become sub-tasks of SW-589, with PR 5 possibly its own ticket. Most of PR 2–5 likely won't fit before the due date (2026-11-24). After each merge, comment on SW-589 with the PR link and what it closed. Move SW-589 to done only after the last PR merges, or after the remaining items move to follow-up tickets.

**If Avichai wants one PR anyway:** keep the same commit order. The workstream boundaries can still be split into PRs later, because no commit mixes workstreams.

## Files involved
- `docs/pipeline_review_findings.md` (the tracker), `.gitignore`, `docs/GRID_CLEAN_PREPROCESSING.md`, `docs/HOLDOUT_CLUSTER_CONFOUNDING_REPORT.md`
- **Core:** `src/core/removal_audit.py`, `v2_pipeline_run.py`, `pipeline_runner.py`, `train.py`, `feature_extraction.py`, `feature_selection_poolsel.py`, `feature_stability.py`, `partition_testing.py`, `blip_filter.py`, `override_scheme.py`, `sensor_fusion.py`, `balanced_rf_tuning.py`
- **Stages:** `src/core/stages/{preprocessing,features,training,holdout,model_reports}.py`
- **Preprocessing:** `grid_clean_preprocess.py`, `plateau_normalization.py`, `trigger_detection.py`, `eran_preprocess.py` (new, from `e1a9cbd`)
- **Reports:** `src/visualization/model_report.py`, `feature_and_patient_score_report.py`, `unit_report.py`
- **Config, scripts, tools:** `configs/pipeline/validation_only.yaml`, `configs/pipeline/models/`, `scripts/lucia_pipeline.py`, `notebooks/General/algo_data_download.ipynb`, `model_dashboard/`
- `tests/`

## Changelog
- **v1 (2026-11-18):** First version. Reordered so local setup and the branch (step 1) run while scope is being confirmed (step 0). Added the INVALID / NOT REPRODUCED statuses. Included G19–G25 from `de32631`.
- **v1, update 1 (2026-11-18):**
  - Step 0 done: Avichai confirmed I implement the Avichai and Eran items too. Recorded defaults for the questions his answer didn't cover.
  - Reviewed coverage: 17 findings weren't placed anywhere (M6, M7, M10, L1–L4, L7, L8, L10, L11, L13–L18), and open parts of G1, G2 and G5 were missing.
  - Rebuilt step 4 into 9 workstreams (W1–W9) that cover all 69 findings and Q1–Q5.
  - Added a security fix for the hard-coded DB password (G2 follow-up).
  - Added before/after metric checks for W7 and W8.
- **v1, update 2 (2026-11-18):**
  - Added 0b, dependencies on other members' work: items, owners, where their work is now (checked against remote branches) and how each is handled.
  - Added the commit strategy: one commit per finding, never mixing workstreams, no single squashed commit per workstream, shared-groundwork commits allowed.
  - Replaced step 7 with the PR plan: 5 PRs by scope and risk, merged in order, with reviewers per PR.
- **v1, update 3 (2026-11-18):**
  - Gilad and Avichai are required reviewers on every PR; others are optional.
  - 0b rewritten as an implementation guideline: check `origin/master` first; open an assigned Jira follow-up ticket for any confirmation or input instead of blocking; update `pipeline_review_findings.md` at the same time.
  - BLOCKED renamed to WAITING (owner, ticket).
  - Dependency table corrected after checking master: `eran_preprocess.py` is on master but not wired in; trace-based model YAMLs exist but are commented out; the user guide exists.
- **v1, update 5 (2026-11-18):** Added 4a, a review checkpoint after every workstream: start/review tags, a review note in `.claude.local/reviews/`, the `[r]` status, a hard stop until explicit approval, a change-request loop without rewriting history, and no push before approval.
- **v1, update 6 (2026-11-18):** W0 review: Tram decided a finding is FIXED only once its regression test lands, not on code reading alone. M1, H6 and L9 went back to ACCEPTED. Their tests are in W7 (M1, H6) and W1 (L9).
- **v1, update 7 (2026-11-18):** W0 approved. Baseline recorded (386 passed, 3 failed, 6 xfailed, 4 collection errors). W1 done and at review: 10 findings FIXED, G2 recorded. Found: the tuning modules can't be imported on master (blocks W7 tuning tests); `model_reports` reports only the last model with `source: training` (added to W8).
- **v1, update 8 (2026-11-19):** G2 answered (password still valid; admin user to be removed) and tracked in SW-593 (Avichai). Tram decided to fix the tuning import bug in W1.
- **v1, update 9 (2026-11-19):** W1 approved. Tram asked not to start W2 yet.
- **v1, update 10 (2026-11-19):** Jira: one sub-task of SW-589 per workstream (W1–W9), assigned to Tram; no W0 ticket. Keep the ticket text short (counts, not per-finding detail; details stay in `pipeline_review_findings.md`) and don't name workstreams the reviewers haven't seen. W2 adds regression tests for G5 and G13.
- **v1, update 11 (2026-11-19):** Jira sub-tasks of SW-589 created: SW-594 (W1, Done, closing comment posted) and SW-595 (W2, To Do), both assigned to Tram. Create the sub-task for each later workstream the same way when it starts (draft in chat first). Jira status: To Do → In Progress → Done (no review status in this project).
- **v1, update 12 (2026-11-19):** Jira sub-tasks created for W3–W9 (SW-596 to SW-602), To Do, assigned to Tram. Every workstream now has its ticket.
- **v1, update 13 (2026-11-19):** W2 at review: G12 and M11 FIXED; G5 got regression tests; G13 already had tests (confirmed they catch the defect). G18 (intent) and L13 (default) need a decision from Gilad/Avichai. Full suite 437 passed.
- **v1, update 14 (2026-11-19):** Tram confirmed the G18 reading; implement it as W2 Round 2 (extract-only features stage for holdout runs).
- **v1, update 15 (2026-11-19):** W2 Round 2: G18 FIXED (df4416d), extract-only features stage for holdout runs, parity test shows 34 identical holdout tables. Full suite 443 passed. Found: resume check uses stale base-run feature paths (asked Tram).
- **v1, update 16 (2026-11-19):** L13 WONTFIX (default kept). All 6 W2 findings resolved.
- **v1, update 17 (2026-11-19):** L1 decided: keep cap n and the current rounding, L1 WONTFIX (Gilad: groups have 5–10+ patients, so the cap never applies on real data). Added to W6: warn when a study group's training pool has < 5 patients. Bugs found during SW-589 ticketed: SW-603 (tuning import, fixed), SW-604 (model reports, last model only), SW-605 (`--resume` stale feature paths).
- **v1, update 18 (2026-11-19):** W2 approved (6 findings: G5, G12, G13, M11, G18 FIXED; L13 WONTFIX). SW-595 Done. The stale `--resume` paths are tracked in SW-605. W3 not started.
- **v1, update 19 (2026-11-19):** W3 started (`sw589-W3-start`). G22 design written; G3, G9/H2 and Q1 need values from Avichai/Gilad/Eran, to ask Tram after G22.
- **v1, update 20 (2026-11-19):** G22 FIXED (66e81aa); full suite 473 passed, baseline failures unchanged. SW-596 In Progress. Asked Tram about G3 values and file count, G9/H2 model-set selection, Q1 provenance. Noted: the default `selected_models` are already the notebook models since 7730944 (H2's model part); n_features 20, L-RAD and permutations 200 still differ from the notebook.
- **v1, update 21 (2026-11-19):** Fixed findings rewritten to show only their current state (87446e0). G9 → WAITING (Eran) after a search of all branches. Open questions for 2026-11-20 listed in the daily report.
- **v1, update 22 (2026-11-19):** G3 and Q1 to be decided at the 2026-11-23 daily meeting; SW-606 created (sub-task of SW-589, linked to SW-596, due 2026-11-23).
- **v1, update 23 (2026-11-19):** Tram closed W3 early: G22 FIXED, G9 WAITING (Eran), G3 WAITING (SW-606), Q1 → SW-606. G3 status drafted for review; tag `sw589-W3-review` after that commit. W3 at review.
- **v1, update 24 (2026-11-19):** G3 status committed (2a8ed48); tag `sw589-W3-review` set. W3 waiting for Tram's review.
- **v1, update 25 (2026-11-20):** W3 approved. PR 1 description drafted for Tram's review (`.claude.local/prs/pr1_description.md`); master unchanged (de32631), branch has 27 commits.
- **v1, update 26 (2026-11-20):** PR 1 description approved (short version). Branch `tram/sw-589-pr38-findings` pushed to origin (tags stay local). PR to be opened by Tram in Bitbucket.
- **v1, update 27 (2026-11-20):** Gilad confirmed the synthetic-dataset request. Created SW-607 (Task, Tram; links SW-589, SW-568; mentions Gilad, Tony, Avichai). Added finding G26 to the findings file (Gilad's notes now G1–G26), not committed yet. Not on PR 1's branch scope list; decide where G26's commit goes.
- **v1, update 28 (2026-11-20):** W4 started (`sw589-W4-start`, branch `tram/sw-589-preprocess-features`, SW-597 In Progress). Triage into A (fix now), B (decision), C (Eran).
- **v1, update 29 (2026-11-20):** Tram: SW-603, SW-604, SW-605 are hers. Check them in every workstream and keep their Jira status current. Status now: SW-603 fixed (a1ea093, in PR #52; tuning tests 8 passed); SW-604 open (planned W8, `model_reports` still reads one `output_dir`); SW-605 open (`REQUIRED_ARTIFACTS['features']` still lists base-run paths). SW-605 has no workstream yet: proposal W8 with SW-604 (runner + reports), or its own small commit in PR 2.
- **v1, update 30 (2026-11-20):** W4 group A done: M9, M8, L10, L7, L8, H3, H4 FIXED; M10 (0df6cd8) and L11 (99db420) confirmed with tests. Suite 499 passed. Waiting for Tram's review before group B.
- **v1, update 31 (2026-11-20):** Group A approved; findings text committed. Group B started: gather the decisions for G11, M6, M7.
- **v1, update 32 (2026-11-20):** Gilad took G9: WAITING (Gilad), he and Eran decide the model per work case (dataset size, group proportions, per trace / per cluster). Interim: adapted models for both per-cluster and per-trace training (not applied yet; it changes the default model list, which overlaps G3 / SW-606). The G9 questions for Eran are no longer needed.
- **v1, update 33 (2026-11-20):** G9 text committed. Tram: the interim adapted-model default waits for the 2026-11-23 meeting (with G3, SW-606).
- **v1, update 34 (2026-11-20):** Group B part 1: M6 config check (f505390), M7 layout record (403889d), 8 test cases, suite 507 passed. Findings text uncommitted for review. M6 scope, M7 leakage, G11 still wait for answers.
- **v1, update 35 (2026-11-20):** Group B part 1 approved; M6/M7 text committed. W4 now waits only on others (G11 Avichai; M6 scope, M7 leakage Gilad/Eran; G10 Eran).
- **v1, update 36 (2026-11-20):** W5 started (`sw589-W5-start`), SW-598 In Progress. Triage into A (fix now) and B (decision); waiting for Tram to approve the split.
- **v1, update 37 (2026-11-20):** W5 group A done (tag `sw589-W5A-review`): G7 test, M3 FIXED, L14 3 bugs fixed. 11 test cases, suite 518 passed. Findings text uncommitted; waiting for Tram's review.
- **v1, update 38 (2026-11-23):** W5 group A approved (2026-11-22). W6 started (branch `tram/sw-589-training-defaults`, tag `sw589-W6-start`, SW-599 In Progress). Design written for M2, G14, H1, L1 warning, G16.
- **v1, update 39 (2026-11-23):** W6 at review: M2, G14, H1, G16 FIXED; L1 warning added. 13 test cases, suite 531 passed. Findings text uncommitted.
- **v1, update 40 (2026-11-23):** W6 approved; findings text committed. SW-599 not moved yet: Jira sign-in server unavailable; retry.
- **v1, update 41 (2026-11-23):** SW-605 added to W8 with SW-604 (Tram). SW-603 is fixed in W1 (PR #52).
- **v1, update 42 (2026-11-25):** Eran answered G9 (keep the legacy trio as the silent default for both run types; adapted trio optional; 7-model catalog; provenance next to models). G9 round planned on `tram/sw-589-training-defaults`; open items listed under it.
- **v1, update 43 (2026-11-25):** G9 round done (6543d7b): default models per run type, provenance blocks, XGBd2. No run changes (22 config/run-type cases identical, hashes unchanged). Findings text uncommitted; waiting for Tram's review.
- **v1, update 44 (2026-11-25):** G9 findings text committed (c20fd44). Tram asked Eran to confirm the model name (`balanced_random_forest_lucia_legacy` = `balanced_rf_aggregated_tuned_lucia_legacy`?) and the run-type rule (clustering off counts as per trace). Waiting for his answer.
- **v1, update 38 (2026-11-22):** W5 group A approved; findings text committed. Group B waits for Gilad, Eran (M4, M5, L14 MI, G20) and Avichai, Gilad (H5, Q3).
- **v1, update 4 (2026-11-18):** Added 4b, the test and evaluation method: the red → green cycle using the repo's `xfail(strict=True)` convention, proof by type of finding, the workstream gate, the before/after metric comparison, and a definition of done per finding.
- **v1, update (2026-11-25):** Eran answered M6/M7. Done on the W4 branch: band −60 / +120 (relive_beta pinned +90), `plateau_median_global` wording, `grid_clean_layout.json`; M7 B won't fix. 5 commits, suite 523 passed. Waiting on Eran: valid-only pool and meaning of valid, file format / `layout_file` key, before/after median on real data.
- **v1, update (2026-11-25):** Baseline failures explained: 2 Windows-only test issues (symlink rights, path separator), 2 real bugs ticketed: SW-610 (missing `scripts/protocol6_con_fact.py`, Avichai) and SW-611 (`control_group` ignored when `control_groups` is set, Tram; links SW-589, SW-606).
- **v1, update (2026-11-25):** G11 decided (Avichai): blip filter on by default; keep a trace only with < 5 repaired samples before the descent, < 4 after, < 10 in total (max 4 / 3 / 9), type_5 exempt; rejected traces marked invalid (stay in audit and estimation set); datasets have `blip_tuple` but still to verify. Tram: pin the 7 experiment configs off so they reproduce past results. Steps: (1) pin experiments off, (2) synthetic test data gets `blip_tuple`, (3) default on + limits + warning when most `blip_tuple` values are empty + tests, (4) docs, (5) findings text uncommitted for review.
- **v1, update (2026-11-25):** G11 implemented (82f0972, 566fadb, 9165c7a, 2a533e6), 16 test cases, suite 540 passed. Findings text uncommitted. Open: dataset check for `blip_tuple`.
- **v1, update (2026-11-25):** Avichai: H5 yes (keep only the TSF extractor, remove the original one); PR split is fine as planned (5 PRs; not his call). G11 dataset check and the total-limit note: Tram confirms with him at today's daily meeting.
- **v1, update (2026-11-25):** H5 implemented on a fresh Linux clone (new `.venv` from the py3.12 lock minus torch). `RobustSignalAnalyzerTSF` is the only extractor; `use_tsf_features` removed from feature_extraction, features stage, validate and holdout; default `tsf_interval_counts: [1]` (option 1 — reproduces today's features renamed, no result change). Q3 resolved (poolsel always gets TSF names; the mrmr-requires-tsf check removed). Configs: validation_only → [1]; mrmr_consensus / relive_beta / type1_61 drop the flag, keep [1,2]. Base `RobustSignalAnalyzer` class kept for its own unit tests. Regression test red→green; full suite 541 passed, 6 xfailed; the 1 failure (SW-611 control_group) and 1 collection error (SW-610 missing module) pre-date H5. Code committed as one H5 commit; findings text uncommitted for Tram. Stopping for review.
