# Review: SW-589 W3, config contract and defaults

- **Status:** `[x]` approved by Tram on 2026-11-20 · **Jira:** SW-596 (Done)
- **Branch:** `tram/sw-589-pr38-findings`
- **Range:** `sw589-W3-start..sw589-W3-review` (5 commits)
- **Goal of W3:** a run's config is checked before it starts, and the defaults are chosen deliberately.
- **Closed early (Tram, 2026-11-19):** G3 and Q1 wait for the 2026-11-23 meeting and G9 waits for Eran, so W3 closes with G22 as its only fix. G3 gets its own commit after the meeting.

## Scope and new statuses

| Finding | Was | Now | What changed |
|---|---|---|---|
| G22 | ACCEPTED | **FIXED** | Per-trace settings checked when the config loads |
| G9 (+ H2) | ACCEPTED | **WAITING (Eran)** | No per-trace models from Eran on any branch; questions recorded |
| G3 | ACCEPTED | **WAITING (Gilad, Avichai, SW-606)** | Defaults to decide at the 2026-11-23 daily meeting |
| Q1 | open | open, **→ SW-606** | Same meeting as G3 |

1 FIXED; 3 waiting on others, each with an owner.

## Commits

| Hash | Finding | Summary |
|---|---|---|
| 66e81aa | G22 | Check the per-trace config keys when the config is loaded |
| 87446e0 | (all FIXED) | Show only the current state of fixed findings (Tram's rule). Docs only |
| bb69e45 | G9 | Mark as waiting for Eran's per-trace models. Docs only |
| 8fb1b35 | G22 | Explain why null stays true with clustering off. Docs only |
| 2a8ed48 | G3, Q1 | Mark as waiting for the defaults decision (SW-606). Docs only |

## Files changed (riskiest first)

1. **`src/core/config/validate.py`** (G22): new `per_trace_rows`, `resolve_allow_multi_trace`, `_validate_model_report_levels`; called from `validate_pipeline_config`. Replaces the one old check (`unit: trace` + `false`).
2. **`src/core/stages/training.py`** (G22): `training_settings_from_config` uses the shared resolver instead of its own copy of the rule.
3. **`src/core/holdout_estimation.py`** (G22): `prepare_rows` uses the resolver; it read `null` as false before.
4. **`src/core/stages/model_reports.py`** (G22): the missing trace-statistics error names the right setting.
5. **Docs:** `docs/pipeline_review_findings.md` (G22, G9, G3, Q1, and the FIXED-items rewrite).
6. **Tests:** 1 new file, 2 changed.

## Test evidence

**Test cases per finding (30 new cases, 2 changed):**

| Finding | Cases | Test file | What they cover | Red before the change? |
|---|---|---|---|---|
| G22 | 7 | `test_trace_mode_config.py::test_agreeing_keys_load_and_resolve_the_same_in_training` | every agreeing combination of unit × clustering × flag loads, and training resolves the same value | Pass both times: pins the behaviour that had to stay |
| G22 | 3 | `::test_disagreeing_multi_trace_is_rejected_at_load` | `true` on cluster rows, `false` on `unit: trace`, `false` with clustering off | 2 red (cluster + true, clustering off + false); `unit: trace` + false was already rejected |
| G22 | 4 | `::test_report_levels_that_cannot_be_built_are_rejected_at_load` | patient mixed with cluster; cluster with `source: training`; patient with `source: holdout`; trace on a cluster run | All 4 red (only failed at the report stage before) |
| G22 | 3 | `::test_report_levels_that_can_be_built_load` | auto + patient; holdout + cluster on a cluster run; holdout + trace/cluster on a trace run | Pass both times |
| G22 | 11 | `::test_every_shipped_config_still_loads` | all 11 YAMLs in `configs/pipeline/` and `experiments/` | Pass both times: no shipped config is affected |
| G22 | 1 | `::test_holdout_estimation_resolves_null_like_training` | holdout estimation gets `allow_multi_trace=True` for a per-trace run with `null` | Red (got False) |
| G22 | 1 | `::test_missing_trace_statistics_hint_names_the_real_gate` | the missing-file error mentions `allow_multi_trace` | Red |
| **Total** | **30** | | | **9 red before, all pass now** |

**Changed tests:**
- `test_training_stage.py::test_explicit_multi_trace_setting_overrides_clustering_mode` → `test_multi_trace_on_cluster_rows_is_rejected`. It asserted the mismatch G22 removes; it now expects the rejection (red before the change).
- `test_pipeline_runner.py::test_holdout_model_reports_depend_on_holdout_not_training`: asked for `trace` reports on a cluster run, which G22 rejects. It tests stage planning only, so its levels are now `["cluster"]`.

**Red check method:** `git stash` of `src/` only, run the new tests against the old code, then restore.

**Full suite:**

| | Passed | Failed | xfailed | Collection errors |
|---|---|---|---|---|
| After W2 | 443 | 3 | 6 | 1 |
| W3 review | **473** (+30) | 3 (same) | 6 (same) | 1 (same) |

- **Invariants:** 18 passed, 6 xfailed (SW-568). No XPASS.
- **System:** 10 passed; the 1 baseline failure (`test_unused_study_group_is_left_out[NOT_LC-L-RAD]`) is unchanged.
- **Synthetic run:** the system tests run the full pipeline through config loading, so every stage now passes the new check. No separate run needed: G22 changes validation only.

## Metric comparison
Not needed. G22 rejects configs that disagree and resolves `null` the same way training already did; every shipped config loads and resolves as before. No model numbers change.

## Decisions made
1. **Clustering off → `null` resolves to true** (confirmed by Tram, documented in G22, 8fb1b35). The finding proposed false; that would make holdout keep one channel per measurement.
2. **`source: auto` keeps its run-time check.** The `holdout` command picks the source only when it runs, so config loading can't know it.
3. **Holdout estimation fixed in G22.** Same rule; it read `null` as false.
4. **W3 closed early** (Tram): G3 and Q1 → SW-606, G9 → Eran.

## Follow-up Jira tickets created
- **SW-606:** decide the default config values (G3, Q1) at the 2026-11-23 daily meeting. Sub-task of SW-589, linked to SW-596.

## Questions for Tram
None open for W3. Waiting on others: G3/Q1 (meeting 2026-11-23), G9 (Eran).

## How to review
```bash
git log --oneline sw589-W3-start..sw589-W3-review
git diff --stat sw589-W3-start..sw589-W3-review
git diff sw589-W3-start..sw589-W3-review -- src/core/config/validate.py   # riskiest
git show 66e81aa                                                          # G22
```
