# Plan: understand the LUCIA codebase structure (v1)

Ticket: TBD (rename the file when a Jira ticket exists)
Date: 2026-11-19
Reference map: https://claude.ai/artifact/FhBeQ8kcaQJsZtG5wvBaVD (Part 1 only; Parts 2-5 are out of scope here)

## Scope

In scope: what the blocks are, how one run flows through them, where each kind of change belongs.
Out of scope: improving the code, reorganising docs, understanding each algorithm's math.

## Definition of Done (whole task)

The task is done when all of these are true:

1. You can draw the block diagram (interfaces, config, orchestration, stages, algorithms,
   reporting, data/storage, dashboard) from memory and name 2 key files in each block.
2. You can trace one run from YAML config to published results, naming the stage order
   and one artifact each stage writes.
3. You have run the pipeline yourself at least as a dry-run plus the end-to-end test,
   and found the run directory it produced.
4. You can say where these 3 changes would go: a new config key, a new feature
   in feature extraction, a new stage.
5. You can tell active code from legacy/personal code by folder.
6. You answer at least 8 of the 10 check questions at the bottom without looking.
7. You have written a 1-page summary in your own words
   (`.claude.local/notes/codebase-summary.md`).

## Sub-tasks

Do them in order. Each one builds on the previous.

### S1. Big picture
Read: `README.md`, `docs/PIPELINE_V2_USER_GUIDE.md` sections "What the pipeline does"
and "Repository map", `.cursor/rules/pipeline-team-standards.mdc`.

Done when:
- [ ] You can say in 3 sentences what the pipeline does.
- [ ] You can name the 3 interfaces (CLI, thin notebooks, full notebooks) and the folder
      they all call (`src/core/stages/`).
- [ ] You can state the gist of the 9 team rules, one line each.

### S2. Run it
Read: `docs/PIPELINE_V2_QUICKSTART.md` sections 1-2.
Do: activate the venv, run
`python scripts/lucia_pipeline.py run --config configs/pipeline/example_run_lucia.yaml --dry-run`,
then `pytest tests/system tests/unit -q`.

Done when:
- [ ] The dry-run prints a stage plan and you can read it.
- [ ] Tests run; you have written down pass/skip counts and why tests were skipped.
- [ ] You found where local runs are written (`LUCIA_RUNS_DIR`, default `~/lucia-runs`).

### S3. Configuration
Read: `src/core/config/` (`loader.py`, `merge.py`, `model.py`), `configs/pipeline/validation_only.yaml`,
`configs/pipeline/example_run_lucia.yaml`, `docs/pipeline_v2_configuration.md` "Configuration layers".

Done when:
- [ ] You can explain the 3 config layers and which one wins.
- [ ] You traced one key (e.g. `features.n_features`) from YAML to the line of stage code that reads it.
- [ ] You can name the 3 allowed environment variables.

### S4. Orchestration
Read: `src/core/pipeline_runner.py` (top constants, `resolve_stage_plan`, `should_skip_stage`,
`run_pipeline`), `src/core/v2_pipeline_run.py` (`PipelineRun`, `record_artifact`),
`Tracker` in `src/core/removal_audit.py`.

Done when:
- [ ] You can draw the stage dependency graph (7 stages).
- [ ] You can explain how a run is resumed and when a stage is skipped.
- [ ] You can say what the run manifest and the Tracker record.

### S5. Stages, one at a time
Read: each file in `src/core/stages/` and the algorithm module it calls.
Do it in 4 sittings: (a) preprocessing + clustering, (b) features, (c) training + holdout,
(d) visualization + model_reports.

Done when:
- [ ] You filled this table for all 7 stages:

  | Stage | Reads | Calls (algorithm module) | Writes | Key config section |
  |---|---|---|---|---|

- [ ] You can explain the difference between the `training` and `holdout` stages.
- [ ] You know which stage enforces patient-level separation (and why it matters, rule 7).

### S6. Data in and out
Read: `docs/shared_datasets.md`, `src/datasets/__init__.py`, `src/core/results_store.py`,
`model_dashboard/README.md`.

Done when:
- [ ] You can draw the path Cloud SQL -> published dataset -> local cache -> run dir
      -> GCS results -> dashboard bundle.
- [ ] You know the 3 top-level GCS folders (`datasets/`, `results/`, `models/`) and what goes in each.
- [ ] You can run `python scripts/lucia_pipeline.py runs list` (or know why you cannot: GCS access).

### S7. Active vs legacy
Read: `docs/NOTEBOOK_MIGRATION.md`, folder listing of `notebooks/`, `scripts/`, `src/utils/`.

Done when:
- [ ] Every top-level folder and `notebooks/` subfolder is labelled active, legacy or personal.
- [ ] You know which folders must not be used for new work (`notebooks/gilad/`, any `archive/`).

### S8. Wrap-up
Do: close all files, draw the block diagram on paper, answer the check questions,
write the 1-page summary.

Done when:
- [ ] Whole-task DoD items 1, 4, 6 and 7 are met.
- [ ] Open questions you still have are listed at the end of the summary.

## Check questions

1. Which file decides the stage order and dependencies?
2. Which 3 files must preprocessing leave for the stage to count as complete?
3. In which order are config layers applied?
4. Which stage functions do the CLI and notebooks call, and from which package?
5. What is the difference between thin and full notebooks?
6. What does the `holdout` stage do that `training` does not?
7. Where in GCS do published run results go?
8. What does the `Tracker` record, and why is it shared across stages?
9. Which folders are frozen legacy and must not be used for new work?
10. A new config knob is needed. What must it have, according to team rule 4?

## Answer key (check only after answering)

1. `src/core/pipeline_runner.py` (`STAGE_ORDER`, `STAGE_DEPENDENCIES`).
2. `processed.pkl`, `tracker.pkl`, `section_landmarks.json`.
3. `validation_only.yaml` -> run YAML -> CLI `--set` overrides (last wins).
4. `run_*` functions from `src/core/stages/`.
5. Same stage APIs; full notebooks add read-only diagnostics, no second algorithm.
6. Several patient-disjoint holdouts (or k-fold), feature selection inside each holdout,
   pooled test predictions per patient.
7. `gs://nanose_ml/datasets_models/lucia/results/<dataset>/<selection>/<owner>/<experiment>/<run-id>/`.
8. Rows removed at each step and why; later stages read it (e.g. features reads clustering settings).
9. `notebooks/gilad/` and any `archive/` folder (e.g. `notebooks/lenny/archive/`).
10. A safe default that keeps current behaviour for existing configs.
