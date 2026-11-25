# Review: SW-589 W1, small bugs and docs

- **Status:** `[x]` approved by Tram on 2026-11-19 · **Jira:** SW-594 (Done)
- **Branch:** `tram/sw-589-pr38-findings`
- **Range:** `sw589-W1-start..sw589-W1-review` (9 commits)
- **Goal of W1:** no change to model numbers.

## Scope and new statuses

| Finding | Was | Now | What changed |
|---|---|---|---|
| L9 | ACCEPTED | **FIXED** | Regression test only; fix was already on master (aac57b7) |
| L12 | ACCEPTED | **FIXED** | `apply_sensor_fusion` defaults to `none` instead of the removed `late` |
| L5 | ACCEPTED | **FIXED** | Warning + reason when no threshold meets the sens/spec floors |
| L6 | ACCEPTED | **FIXED** | Same warning, flag and reason on the two early-return paths |
| L3 | ACCEPTED | **FIXED** | Training seed context also seeds/restores NumPy's global RNG |
| L4 | ACCEPTED | **FIXED** | Aggregate result names its model (`aggregate_model_id`), adds `results_by_model()`, warns with >1 model |
| L1 | ACCEPTED | **FIXED** | Wrong "reproduce exactly" comment corrected; draws unchanged |
| L2 | ACCEPTED | **FIXED** | `rotating` records `effective_fraction` and warns when slices ≠ proportion |
| L16 | ACCEPTED | **FIXED** | Partition audit says what it checked (`scope`, `scope_note`) |
| L19 | ACCEPTED | **FIXED** | GRID_CLEAN doc uses Pipeline V2 keys and real paths |
| G2 | WONTFIX | WONTFIX | Password check recorded; notebook unchanged (see Decisions 4) |

11 findings: 10 FIXED, 1 recorded. No INVALID, NOT REPRODUCED or WAITING.

## Commits

| Hash | Finding | Summary |
|---|---|---|
| 6312173 | L9 | Add a regression test for missing trigger landmarks |
| 83a310d | L12 | Default sensor fusion to the only mode that exists |
| 738af71 | L5, L6 | Warn when the threshold falls back to 0.5 |
| 34a7773 | L3 | Seed NumPy's global RNG in the training seed context |
| e7fc5dd | L4 | Name the model the aggregate training result describes |
| 714ceb8 | L1, L2 | Make validation draw sizes match their docs |
| aa3c7d9 | L16 | Say what the partition audit actually checks |
| 485195a | L19 | Point the GRID_CLEAN doc at Pipeline V2 config keys |
| efd171c | G2 | Record the notebook password check |

## Files changed (riskiest first)

1. **`src/core/train.py`**: `ThresholdClassifier.fit` (L5, L6). On the main training path, but the thresholds are the same; only a warning, the flag and a reason are added. The per-model table gets one new column, `threshold_fallback_reason`. Also a comment fix (L1).
2. **`src/core/stages/training.py`**: seed context (L3); `TrainingResult` gets one optional field and one method, plus a warning (L4).
3. **`src/core/partition_testing.py`, `src/visualization/partition_report.py`**: warning and attribute on `RotatingDrawer` (L2); `scope` / `scope_note` in the audit dict and the HTML report (L16).
4. **`src/core/sensor_fusion.py`**: default argument (L12). No callers anywhere.
5. **Docs:** `docs/GRID_CLEAN_PREPROCESSING.md` (L19), `docs/pipeline_review_findings.md` (all rows).
6. **Tests:** 7 new files, 2 extended.

## Test evidence

**New tests (33), each red before its fix and green after.** The exceptions are noted.

| Finding | Test | Red before? |
|---|---|---|
| L9 | `test_trigger_detection_anchors.py::test_missing_landmark_does_not_abort_the_cohort` (4 cases) | Yes, run against aac57b7^: NaN cases fail with `ValueError: cannot convert float NaN to integer` |
| L12 | `test_sensor_fusion.py` (6) | 2 default-mode tests red; 4 explicit-mode tests pin existing behaviour |
| L5, L6 | `test_threshold_fallback.py` (4) | All red |
| L3 | `test_training_seed.py` (2) | All red |
| L4 | `test_training_stage.py::test_multi_model_aggregate_*`, `::test_single_model_aggregate_does_not_warn` | Both red |
| L1, L2 | `test_partition_sizes.py` (5) | L2: 2 red. L1: 3 pin the corrected comment (doc fix, so green both times) |
| L16 | `test_partition_audit_scope.py` (4) | All red |
| L19 | `test_grid_clean_doc.py` (6) | 4 red (stale path, `process.` / `filter.` / `load.` keys) |

**Test cases per finding (33 cases from 24 test functions;** a function with several inputs counts once per input, as pytest collects them):

| Finding | Cases | Test file | What they cover | Red before the change? |
|---|---|---|---|---|
| L9 | 4 | `test_trigger_detection_anchors.py` | NaN up trigger; NaN down trigger; missing up tuple; missing down tuple | Fix was already on master, so all 4 pass there. Proven on aac57b7^: the 2 NaN cases crash (`cannot convert float NaN to integer`) and the 2 tuple cases fail |
| L12 | 6 | `test_sensor_fusion.py` | default is a valid mode (1); default call returns data unchanged (1); each removed mode still raises when asked for (4) | 2 red (the default-mode tests). The 4 removed-mode cases pass both times; they pin existing behaviour |
| L5 | 1 | `test_threshold_fallback.py` | no threshold meets the sens/spec floors | Red |
| L6 | 2 | `test_threshold_fallback.py` | single-class training set; fewer than 2 patients in a class | Red |
| L5 + L6 | 1 | `test_threshold_fallback.py` | a normal fit doesn't warn or flag a fallback (shared) | Red (the reason attribute didn't exist) |
| L3 | 2 | `test_training_seed.py` | same seed gives same Python + NumPy draws; outer random state restored | Red |
| L4 | 2 | `test_training_stage.py` | several models: names the model and warns; one model: no warning | Red |
| L1 | 3 | `test_partition_sizes.py` | matches the old count for 20% and 11.1111% (2); the documented differences are real (1) | Pass both times (the fix was only a comment; these pin the corrected claim) |
| L2 | 2 | `test_partition_sizes.py` | 30% warns (3 slices of 33%); 20% stays silent | Red |
| L16 | 4 | `test_partition_audit_scope.py` | default scope; recorded-run scope; note shown in the report; `test_partitions` audits its own plan | Red |
| L19 | 6 | `test_grid_clean_doc.py` | every path exists; every function exists; every config key exists; no `process.` / `filter.` / `load.` keys (3) | 4 red (stale path + the 3 legacy-key cases); 2 pass both times |
| G2 | 0 | — | checked and recorded only, no code change | — |
| **Total** | **33** | | | Every fix has **at least one** test that failed before its change |

Not counted above: the tuning import fix (Round 3) added no tests; it made the 8 existing tests in the 3 tuning test files collect and pass again.

**Full suite:**

| | Passed | Failed | xfailed | Collection errors |
|---|---|---|---|---|
| Baseline (master@de32631) | 386 | 3 | 6 | 4 |
| W1 review | **419** (+33) | 3 (same) | 6 (same) | 4 (same) |

- **Invariants** (`tests/invariants`): 18 passed, 6 xfailed (the SW-568 ones). No XPASS, so no SW-568 xfail changed state.
- **System** (`-m system`, synthetic end-to-end runs through `run_pipeline`, including training): 9 passed, 1 failed (the baseline `test_unused_study_group_is_left_out[NOT_LC-L-RAD]`).
- **Synthetic pipeline run:** covered by the system tests above. W1 touches no stage inputs, so I didn't do a separate `scripts/lucia_pipeline.py` run.
- `-m invariant` on its own stops at the 4 baseline collection errors, so I ran the folder instead.

## Metric comparison
Not needed. No W1 change alters a draw, a threshold or a prediction:
- **L3:** every shipped model sets `random_state: 42`, and the training path doesn't use `np.random`.
- **L1, L2:** draws are unchanged; no config uses `rotating`.
- **L5, L6:** the threshold value is the same, now with a warning.

## Decisions I made (please check)
1. **L1: fixed the comment, not the rounding.** Fixing the rounding would change validation draws for some `val_size` values (e.g. 35%), which breaks W1's "no number changes" rule. The shipped values (20, 11.1111) already match. If you want the rounding fixed too, it belongs with W6/W7.
2. **L4: kept the aggregate fields** and made them explicit instead of changing their shape, which would break `notebooks/pipeline/full/04_training.ipynb` and the model-report stage. **New issue found:** with `source: training` and several models, `model_reports` reports only the last model. I added it to W8 in the plan.
3. **L16: documented rather than rewired.** The audit now states its scope; the real run guards already exist and are named in the note.
4. **G2: notebook unchanged.** Checked by comparing values in code, never printing them. The current `password=` is a placeholder word that the Cloud SQL proxy replaces with an IAM token (the cell's own comment says so). It matches no earlier value, and removing it could break the connection. **The real exposure is an older, different hard-coded `DB_PASSWORD` in the history of `notebooks/General/algo_data_download.ipynb` (e.g. 0227e0f).** Recorded in G2's Resolution line for your conversation with Avichai.
5. **L1 + L2 and L5 + L6 share a commit each,** because they're one fix to one function or topic.

## Found during W1, outside the plan
- **The tuning modules don't import on master.** In `FeatureTuningInput` (`balanced_rf_tuning.py`), `positive_labels` has a default but comes before `feature_run_id`, which doesn't (added in 1bfe9ba). This blocks BRF/XGB/EasyEnsemble tuning and their tests, so W7 (H6, M12, M13, L15) can't be tested until it's fixed. The fix is one line (move the field).
- **`tests/unit/test_protocol6_con_fact.py` imports `scripts.protocol6_con_fact`,** which isn't on master.
- **Two baseline failures still have no known cause** (`test_unused_study_group_is_left_out[NOT_LC-L-RAD]`, `test_visualization_skips_unavailable_optional_report_inputs`). They may be Windows-only; check on the VM.

## Follow-up Jira tickets created
- SW-593 (G2, Avichai), see Round 2.

## Questions for Tram
1. ~~**G2:** was the old 4-character `DB_PASSWORD` in git history the real `postgres` password, and has it been rotated?~~ **Answered (Avichai, 2026-11-19):** it's still valid for now; the admin user will be removed so connections are IAM-only. See Round 2.
2. ~~**Tuning import bug:** fix it or ticket for Gilad?~~ **Answered: fix it myself.** See Round 3.
3. **L1:** comment fix only, or also fix the rounding / the `n − 1` cap? **Pending: Tram is asking Gilad at the daily meeting (2026-11-19).** Doesn't block W1 approval; any code change would go in W6.

## How to review
```bash
git log --oneline sw589-W1-start..sw589-W1-review
git diff --stat sw589-W1-start..sw589-W1-review
git diff sw589-W1-start..sw589-W1-review -- src/core/train.py src/core/stages/training.py   # riskiest
git show <hash>                                                                             # one finding
```
The quickest read is the L rows in the Low table of `docs/pipeline_review_findings.md`.

## Round 2 (2026-11-19)

**Feedback:** Tram relayed Avichai's answer to question 1. The old password is still valid for now. The admin user will be removed so the database accepts IAM logins only.

| Hash | Item | Summary |
|---|---|---|
| see `git log -1 sw589-W1-review` | G2 | Resolution line records the answer; open until the admin user is removed (owner: Avichai). Docs only |

- **Still open:** questions 2 (tuning import bug) and 3 (L1 rounding).
- **Follow-up ticket created (Tram's OK):** **SW-593** "SW-589 follow-up: G2 - remove the postgres admin user (IAM-only DB access)", assigned to Avichai, To Do, linked (relates to) to SW-589, SW-545 and SW-553. Before creating it I searched all tickets (one project, SW): no open ticket covered it. SW-545 listed "update postgres user password" as remaining work but was closed without it. The password value isn't in the ticket.
- **Review:** `git show sw589-W1-review`

## Round 3 (2026-11-19)

**Decision (Tram):** fix the tuning import bug myself, in W1.

| Hash | Item | Summary |
|---|---|---|
| see `git log -1 sw589-W1-review` | tuning (not a finding) | `FeatureTuningInput`: `positive_labels` moved after `feature_run_id`, so the dataclass is valid again |

**Proof:**
- The 3 tuning test files (`test_balanced_rf_tuning.py`, `test_xgboost_tuning.py`, `test_easy_ensemble_tuning.py`) failed to collect before. After the fix, their 8 tests pass.
- The only constructor call uses keywords, so no caller changes.

**Full suite now:** **427 passed** (was 419), 3 failed (same as baseline), 6 xfailed, **1 collection error** (was 4; only `test_protocol6_con_fact.py` is left, missing `scripts/protocol6_con_fact.py` on master).

**Effect on later work:** W7 tuning items (H6, M12, M13, L15) can now be tested.

**Still open:** question 3 (L1 rounding).

**Review:** `git show sw589-W1-review`

## Follow-up (2026-11-19): L1 decision

- **Gilad:** study groups have at least 5–10 patients; the real dataset has about 100. The 1-patient case only came from notebook tests.
- **Decision (Tram):** keep cap n and the current rounding. L1 status FIXED → **WONTFIX**; the corrected comment and its test stay. Commit 24f80c2 (docs only).
- **Added to W6:** warn when a study group's training pool has fewer than 5 patients (draws unchanged).
