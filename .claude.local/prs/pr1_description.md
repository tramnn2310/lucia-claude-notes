**Title:** SW-589 (PR 1/5): Review findings triage and safe fixes

**Branch:** `tram/sw-589-pr38-findings` into `master`
**Reviewers:** Gilad, Avichai

## What this PR does
Goes through the PR 38 review findings in `docs/pipeline_review_findings.md` and lands the fixes that don't change model results. Every finding now has a Resolution line. No model numbers change.

## Findings closed in this PR
| Status | Findings |
|---|---|
| Fixed here (12) | G12, G18, G22, M11, L2, L3, L4, L5, L6, L12, L16, L19 |
| Already fixed on master, confirmed (6) | G1, G4, G5, G7, G13, L9 |
| Won't fix (3) | G2, L1, L13 |
| Waiting (2) | G3 (meeting on 2026-11-23, SW-606), G9 (Eran) |

Each fix has a test that fails on the old code. The other findings come in later PRs.

## Changes reviewers should know about
- Configs whose per-trace settings contradict each other are now rejected before the run starts (G22). All shipped configs still load.
- A holdout run without training now only extracts features and skips the whole-pool selection (G18). Holdout results are identical.
- The removal audit counts visits (G12) and records GRID_CLEAN drops (M11).
- New warnings when the threshold falls back to 0.5 (L5, L6) and when several models are trained (L4).
- Tuning modules import again (SW-603).

## Tests
- 473 passed (386 on master). The 3 failures and 1 collection error also fail on master.
- A system test runs holdout with and without whole-pool selection and checks that all 34 output tables match.

## Found during this work
- SW-603: tuning modules failed to import (fixed here).
- SW-604: model reports from a training run cover only the last model.
- SW-605: `--resume` always reruns the features stage.

## How to review
Start with the Resolution lines in `docs/pipeline_review_findings.md`. There is one commit per finding, so `git show <hash>` shows one fix and its test. The largest changes are in `src/core/stages/features.py` (G18) and `src/core/config/validate.py` (G22).
