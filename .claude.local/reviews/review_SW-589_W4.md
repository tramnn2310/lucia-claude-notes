# Review: SW-589 W4, preprocessing (GRID_CLEAN, Eran's preprocessing, blip filter)

- **Status:** group A `[x]` approved by Tram on 2026-11-20 · group B in progress · group C not started · **Jira:** SW-597 (In Progress)
- **Branch:** `tram/sw-589-preprocess-features` (cut from the PR 1 branch; rebase onto master after PR #52 merges)
- **Range (group A):** `sw589-W4-start..sw589-W4A-review` (7 commits)
- **Not committed yet:** the group A text in `docs/pipeline_review_findings.md` (Tram's rule: review first).

## Group A: scope and new statuses

| Finding | Was | Now | What changed |
|---|---|---|---|
| M9 | ACCEPTED | **FIXED** | Trigger search covers its whole window (±15 really ±15) |
| M8 | ACCEPTED | **FIXED** | Plateau cut drops traces whose window runs outside the trace |
| L10 | ACCEPTED | **FIXED** | `median_from_up_start_to_down_start: false` no longer drops every trace |
| M10 | ACCEPTED | **FIXED** (already on master, 0df6cd8) | Regression test: AUX / INDEX rows kept |
| L11 | ACCEPTED | **FIXED** (already on master, 99db420) | Existing test fails before the fix. Eran's `exact_last_plateau` alias left for him (G10) |
| L7 | ACCEPTED | **FIXED** | Legacy blip total reproduced with the split off |
| L8 | ACCEPTED | **FIXED** | Descent-sample check only on same-length signals |
| H3 | → G11 | **FIXED** | Blip rejection survives the temperature override. Default-on stays with G11 |
| H4 | → G10 | **FIXED** | GRID_CLEAN preprocessing run finishes with temperature and type_3 rows. Eran's module stays with G10 |
| Q2 | open | partly answered | GRID_CLEAN finishes on synthetic data; a real-data run comes with G10/G11 |

## Commits

| Hash | Finding | Summary |
|---|---|---|
| e04d118 | M9 | Search the whole trigger window |
| 755cdc1 | M8 | Drop plateau cuts whose window runs outside the trace |
| a80f6c8 | L10 | Keep traces when the middle is measured from the rise end |
| 8ce5c5d | M10 | Add a regression test for AUX and INDEX rows |
| 444d6d0 | L7, L8 | Fix the legacy blip count and the descent-sample check |
| c1a646b | H3 | Keep a blip rejection when aggregated_reason is recomputed |
| 0960110 | H4 | Let a GRID_CLEAN run finish with temperature rows |
| (uncommitted) | all of group A | Findings-file text |

## Files changed (riskiest first)

1. **`src/core/schema_mapper.py`** (H3): `compute_aggregated_reason` returns `INVALID_blip_rule` when `blip_rule_violation` is true and no reason column is invalid. Used by every recomputation of `aggregated_reason`. No effect when the blip filter is off (the column doesn't exist).
2. **`src/core/stages/preprocessing.py`** (H4): with GRID_CLEAN on, non-SENSOR rows take the default cut. No effect with GRID_CLEAN off.
3. **`src/core/override_scheme.py`** (H4): the temperature override is a no-op on a frame with no temperature row, instead of requiring the cut column.
4. **`src/core/trigger_detection.py`** (M9): search window. Only GRID_CLEAN and Eran's preprocessing call it.
5. **`src/core/plateau_normalization.py`** (M8), **`src/core/grid_clean_preprocess.py`** (L10), **`src/core/blip_filter.py`** (L7, L8, H3 constant).
6. **Tests:** 7 new files.

## Test evidence

| Finding | Cases | Test file | What they cover | Red before the change? |
|---|---|---|---|---|
| M9 | 7 | `test_trigger_search_window.py` | a rise starting 11–14 samples into a 15-sample window is found (4); a rise at the window end is not; DB±15 finds DB−15 and DB+15 in the close window; a rise with fewer than 5 samples left in the signal is never used | 5 red; the window-end and signal-end cases pass both times (they pin the bounds) |
| M8 | 3 | `test_plateau_cut_bounds.py` | window past the trace end dropped; window before sample 0 dropped; every kept cut has the full length | 1 red (past the end). The before-0 case was already dropped by the length filter |
| L10 | 4 | `test_grid_clean_legacy_median.py` | median filter sets min/max length (true, false); traces kept by the gate survive the length filter (true, false) | 2 red (the `false` cases) |
| M10 | 2 | `test_preprocessing_keeps_aux_rows.py` | AUX and INDEX rows kept by a run with `include_sensor_types` | Both red on the stage before 0df6cd8 |
| L11 | 0 new | `test_trigger_detection_anchors.py` (existing) | DOWN landmark is the first descent sample; anchor offsets | The landmark test fails on the code before 99db420 |
| L7 | 2 | `test_blip_counts.py` | legacy total = reported total when bracketed; split mode still counts the bracketed sample | 1 red (legacy) |
| L8 | 2 | `test_blip_counts.py` | same-length change detected; different-length signals not compared | 1 red (different length) |
| H3 | 2 | `test_blip_survives_overrides.py` | temperature override keeps the blip rejection, and still updates the other trace; `compute_aggregated_reason` reports the blip only when nothing else is invalid | Both red |
| H4 | 4 | `test_grid_clean_stage_run.py` | GRID_CLEAN stage run finishes and keeps every row: sensors only, with temperature, with temperature + type_3; the temperature row gets a cut and reads Good | All 4 red |
| **Total** | **26** | | | 15 red before; the rest pin behaviour that had to stay |

**Red-check method:** `git stash` of the changed `src/` files (or the file from before the fix commit, for M10 and L11), run the new tests, restore.

**Full suite:**

| | Passed | Failed | xfailed | Collection errors |
|---|---|---|---|---|
| After W3 | 473 | 3 | 6 | 1 |
| W4 group A | **499** (+26) | 3 (same) | 6 (same) | 1 (same) |

## Metric comparison
Default runs don't change: GRID_CLEAN, Eran's preprocessing and the blip filter are off by default, and trigger detection is only called by those paths. Runs that turn them on change:
- **GRID_CLEAN runs:** both `relive_beta_preprocess.yaml` and `type1_61_crop_wp_m1_a070_pw2.yaml` enable GRID_CLEAN. They change through the trigger windows (M9), dropped out-of-bounds cuts (M8) and, most likely the biggest effect, the temperature override (H4): their AUX rows went through GRID_CLEAN with no cut, so on real data (where type_3/4/5 rows give the frame the cut column) every file with a listed PCB read NaN and was marked Bad. Worth telling Eran and Gilad: past results from these two configs may have lost those files. A before/after on a real GRID_CLEAN run belongs to the G10/G11 run.
- **Blip filter runs:** rejected traces now stay out of training (H3); legacy counts change by at most 1 for bracketed repairs (L7).

## Decisions I made (please check)
1. **M9:** both window ends are included (DB−15 .. DB+15), matching "± 15" and the existing inclusive stage window.
2. **M8:** an out-of-bounds trace is dropped, not padded. It's recorded under the existing `grid_clean_length_filter` reason.
3. **L10:** in the `false` mode the accepted cut length comes from the traces the median gate kept, so the gate is the only length filter, as in the `true` mode.
4. **L8:** when the repair changed the signal's length, the descent sample counts as not repaired (it can't be compared). The bracketed rule still counts it when repairs sit on both sides.
5. **H3:** fixed in `compute_aggregated_reason` (one place) rather than in each override.
6. **H4:** AUX / INDEX rows take the default cut in a GRID_CLEAN run.

## Bug tickets (SW-603/604/605)
- SW-603: fixed (a1ea093, PR #52), In Progress until the merge. Comment added.
- SW-604, SW-605: still open in the code, To Do. Not touched by W4.

## Next (after review)
- **Group B** (needs decisions): G11 blip filter on by default (Avichai), M6 plateau median scope and M7 persisting it (Gilad, Eran).
- **Group C** (waiting for Eran): G10, and the `exact_last_plateau` alias (L11).

## How to review
```bash
git log --oneline sw589-W4-start..sw589-W4A-review
git diff sw589-W4-start..sw589-W4A-review -- src/core/schema_mapper.py src/core/stages/preprocessing.py   # riskiest
git show <hash>
git diff docs/pipeline_review_findings.md   # uncommitted findings text
```

## Group B, part 1 (2026-11-20): changes that don't need the decisions

| Hash | Finding | Summary |
|---|---|---|
| f505390 | M6 (part) | Reject a GRID_CLEAN config with the median gate off and no layout |
| 403889d | M7 (part) | Record the GRID_CLEAN plateau layout on the run |
| (uncommitted) | M6, M7 | Findings text: both stay ACCEPTED, with what's done so far |

| Finding | Cases | Test file | What they cover | Red before the change? |
|---|---|---|---|---|
| M6 | 5 | `test_grid_clean_median_switch.py` | gate off without a fixed layout rejected at load (2 switches); gate off with a fixed layout loads; gate on loads; gate off with GRID_CLEAN disabled loads | 2 red (the rejections) |
| M7 | 3 | `test_grid_clean_layout_record.py` | measured layout on the tracker and the manifest; feeding the recorded layout back as a fixed layout gives identical signals; a default run records no layout | 2 red |
| **Total** | **8** | | | 4 red |

- **Full suite:** 507 passed (+8), 3 failed, 6 xfailed, 1 collection error (same baseline).
- **No result changes:** the M6 check only rejects configs that would fail anyway; M7 only adds a record.
- **Decisions:**
  - The M6 check runs only when GRID_CLEAN is enabled.
  - M7 reuse needs no new setting: the existing fixed-layout keys (`plateau_middle_target`, `plateau_min_len`, `plateau_max_len`) with the gate off.
- **Still waiting:** G11 (Avichai), the M6 median scope and the M7 leakage decision (Gilad, Eran), G10 (Eran).

## Group B, part 2 (2026-11-25): Eran's answer on M6 and M7

Eran: M6 A + C (no per-breath median); default band −60 / +120; pin relive_beta at +90; M7 A (layout file), B won't fix.

| Hash | Finding | Summary |
|---|---|---|
| 97bd83a | M6 | Pin relive_beta at the old GRID_CLEAN median band (+90) |
| d1ce287 | M6 | Widen the default GRID_CLEAN median band to −60 / +120 |
| 6eca34a | M6 | Update the type1_61 handoff for the new band |
| 2b8d04d | M6 | Say what `plateau_median_global: false` means in errors, yaml and docs |
| 6c89266 | M7 | Write the layout to `grid_clean_layout.json` in the run folder |
| (uncommitted) | M6, M7 | Findings text |

| Finding | Cases | Test file | What they cover | Red before? |
|---|---|---|---|---|
| M6 | 3 new | `test_grid_clean_tolerance_default.py` | default band −60 / +120 in code and `validation_only.yaml`; relive_beta pinned at +90; median gate without a configured band keeps up to +120 | All 3 red |
| M6 | 5 (reworded) | `test_grid_clean_median_switch.py` | the rejection message now says what false means | Message match updated |
| M7 | 2 new | `test_grid_clean_layout_record.py` | `grid_clean_layout.json` in the run folder with median, target, min / max, tolerances and counts before / after the gate, listed as a run artifact; a fixed layout records no gate | Both red |

- **Full suite:** 523 passed (+5), 3 failed, 6 xfailed, 1 collection error (same baseline).
- **Results:** only runs that rely on the default band change, and only if they have traces with a middle +91 to +120 above the median. Eran checked type1_61 (same 439 / 439 traces). relive_beta is pinned. Default runs (GRID_CLEAN off) don't change.
- **Decisions:** relive_beta pinned in its own commit before the default changed, so no commit moves it silently; tolerances and the after-gate count are None for a fixed layout (no gate ran).
- **Still waiting on Eran:** valid-only median (pool scope, meaning of "valid"), layout file format and `layout_file` reuse key, before / after median on the real data, the doc line on his branch, whether `eran_preprocess.py`'s own +90 default should change too.

## Group B, part 3 (2026-11-25): G11, blip filter on by default

Avichai: on by default; keep only < 5 before / < 4 after / < 10 total (max 4 / 3 / 9), type_5 exempt; mark invalid; datasets have `blip_tuple` but still to verify. Tram: pin the experiment configs off.

| Hash | Summary |
|---|---|
| 82f0972 | Pin the blip filter off in the 7 experiment configs (before the default changes) |
| 566fadb | Synthetic clinical frame gets `up_trig`, `down_trig`, `blip_tuple` (so tests run through the filter) |
| 9165c7a | Filter on by default, limits 4 / 3 / 9, warning when no sensor row has a `blip_tuple` value |
| 2a533e6 | `docs/GRID_CLEAN_PREPROCESSING.md`: default, limits, what rejection does |
| (uncommitted) | Findings text: G11 FIXED (current state only), H3 row |

| Cases | Test file | What they cover | Red before? |
|---|---|---|---|
| 16 | `test_blip_filter_default.py` | default config resolves on with 4 / 3 / 9, type_5, count on; each of the 7 experiment configs pins it off; 4 before kept / 5 rejected; 3 after kept / 4 rejected; 4 + 3 kept; type_5 and already-invalid left alone; warning with no blip values, none with some | 4 red (default, 2 rejections, warning); the experiment pins pass both times because they were committed first |

- **Full suite:** 540 passed (+16), 1 skipped, 1 failed (SW-611), 6 xfailed, 1 collection error (SW-610).
- **Results change for default runs:** traces over the limits are now invalid. The experiment configs don't change.
- **Decisions:** the `enabled` default lives in `validation_only.yaml`; settings built without a config (tests) keep it off. The warning fires only when no sensor row has a value: empty values are normal for traces with no repair, so "most rows empty" would warn on real data.
- **For Avichai:** with 4 / 3, a kept trace has at most 8 repaired samples, so the total limit (< 10) never rejects on its own.
- **Still open:** verify `blip_tuple` on the published datasets (someone with GCS access: manifest `query_type: full`, or load a pickle).

