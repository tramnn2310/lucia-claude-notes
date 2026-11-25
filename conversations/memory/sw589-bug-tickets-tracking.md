---
name: sw589-bug-tickets-tracking
description: "SW-603/604/605/611 (bugs found during SW-589) are Tram's; check them in every workstream and keep their Jira status current"
metadata:
  node_type: memory
  type: project
  originSessionId: 01c6ba1c-6b43-45ed-99e5-afaca4a762f5
  modified: 2026-11-20T05:01:17.747Z
---

SW-603 (tuning import), SW-604 (model reports use only the last model), SW-605 (`--resume` stale feature paths) and SW-611 (`control_group` ignored when `control_groups` is set; fails the system test `test_unused_study_group_is_left_out[NOT_LC-L-RAD]`, linked to SW-606) are Tram's tasks, linked to SW-589 (Tram, 2026-11-20 / 2026-11-25). SW-610 (missing `scripts/protocol6_con_fact.py`, collection error) is Avichai's; just note when it's resolved.

**Why:** Tram wants these bugs tracked alongside the review findings, not forgotten as side notes.

**How to apply:** In every SW-589 workstream, check whether each one is fixed (code + regression test), and update its Jira status to match: To Do → In Progress when work starts, Done once the fix is merged, with a short comment naming the commit or PR. SW-603's fix (a1ea093) is in PR #52. Related: [[jira-ticket-style]], [[plans-in-claude-local]].
