# Review: SW-589 G9, default models per run type (after Eran's answer)

- **Status:** `[r]` awaiting Tram's review · **Jira:** SW-596 (W3, Done) · G9 stays WAITING (Gilad, Eran)
- **Branch:** `tram/sw-589-training-defaults` (PR 3)
- **Range:** `sw589-G9-start..sw589-G9-review` (1 commit)
- **Findings text:** committed as c20fd44 (after the review tag).

## What changed

| Hash | Summary |
|---|---|
| 6543d7b | Choose default models by run type and record each model's source |
| c20fd44 | G9 findings text |

1. **`configs/pipeline/validation_only.yaml`:** `selected_models: []` and a new `default_models` block with `cluster` and `trace` lists, both the notebook trio. Comments now describe the optional models (adapted trio, XGBd2, trace_based_legacy). `xgboost_depth2_mds` registered in `model_config_files`.
2. **`src/core/config/validate.py`:** `resolve_selected_models` (explicit list wins; empty list takes the run type's default; run type = the G22 rule) and `default_models` validation (keys `cluster`/`trace`, non-empty lists, every model registered).
3. **`src/core/config/loader.py`:** fills `selected_models` and records `selected_models_source` before validation. A resolved config that is reloaded with overrides re-resolves the default, in case the override changed the run type.
4. **`src/core/stages/training.py`:** uses the same resolver.
5. **Model YAMLs (9):** a `provenance:` block beside `model:` (source, metric, date). Not loaded or hashed. `xgboost_depth2_mds.yaml` copied unchanged from Eran's d484e29.

## Test evidence

| Cases | What they cover | Red before? |
|---|---|---|
| 3 | empty selection takes the default: cluster run, `features.unit: trace`, clustering off | Red |
| 1 | the two run types can have different defaults; training settings agree | Red |
| 1 | an explicit list wins, source `run_config` | Red |
| 1 | an unknown model in `default_models` is rejected at load | Red |
| 1 | a resolved config clears the default list so it re-resolves under overrides | Red |
| 9 | every model file keeps its hash and has a `provenance` source and metric | Red (no provenance block; the hashes themselves matched before) |
| 1 | XGBd2 is registered and builds | Red |
| **17** | | **all 17 red before** |

**No change to results:**
- Every shipped config, each loaded as a cluster and as a trace run (22 cases), trains exactly the same models as before (compared with a list saved before the change).
- All 9 model hashes are unchanged, so recorded `model_config_hashes` still match.

**Full suite:** 548 passed (531 after W6, +17), 3 failed, 6 xfailed, 1 collection error. The failures and the collection error are the same pre-existing ones.

## Decisions I made (please check)
1. **Run type uses the G22 rule** (trace when `features.unit: trace` or clustering is off). Eran wrote "by `features.unit`"; to confirm with him.
2. **`provenance:` sits outside `model:`** so hashes don't change. The fields (source, metric, recorded) are a first version until Eran sends his format.
3. **Legacy trio source** is written as "production notebook values, metric not recorded": the repo has no run or metric for them.
4. **One commit** for the whole round. The test file covers all three parts, so splitting it would leave intermediate commits with failing tests.

## Still open (unchanged from the plan)
Gilad: confirm the default (vs his adapted interim), and keep or deprecate trace_based_legacy. Eran: EE30, RF29, XGBd3 definitions; the provenance format; the run-type rule. Both: per-case guidance. Tram: confirm the model name with Eran.

## How to review
```bash
git show 6543d7b --stat
git show 6543d7b -- src/core/config/validate.py src/core/config/loader.py configs/pipeline/validation_only.yaml
git diff docs/pipeline_review_findings.md   # uncommitted G9 text
```
