---
name: sw589-bug-tickets-tracking
description: "SW-603/604/605/611 (bugs found during SW-589) are Tram's; check them in every workstream and keep their Jira status current"
metadata:
  node_type: memory
  type: project
  originSessionId: 01c6ba1c-6b43-45ed-99e5-afaca4a762f5
  modified: 2026-11-26T14:00:00.689Z
---

SW-603 (tuning import), SW-604 (model reports use only the last model), SW-605 (`--resume` stale feature paths) and SW-611 (`control_group` ignored when `control_groups` is set; fails the system test `test_unused_study_group_is_left_out[NOT_LC-L-RAD]`, linked to SW-606) are Tram's tasks, linked to SW-589 (Tram, 2026-11-20 / 2026-11-25). SW-610 (missing `scripts/protocol6_con_fact.py`, collection error) is Avichai's; just note when it's resolved.

**Why:** Tram wants these bugs tracked alongside the review findings, not forgotten as side notes.

**How to apply:** In every SW-589 workstream, check whether each one is fixed (code + regression test), and update its Jira status to match: To Do → In Progress when work starts, **Done once the group containing the fix is approved by Tram** (she confirmed 2026-11-26: approval, not PR merge, is the Done trigger — do not wait for merge). Add a short comment naming the commit/PR when useful. Done transitionId = 31; In Progress = 21; cloudId 299e5b8c-9331-4e3f-83c7-fa3bed1c3069. Status log: SW-603 fixed (a1ea093, PR #52); SW-604 Done 2026-11-26 (c2b0fae, W8 group A approved); SW-605 Done 2026-11-26 (1c19711, W8 group A approved). SW-611 still open. **Eran-blocked W8 report items split into their own tickets 2026-11-26 (assigned to Tram, To Do, WAITING on Eran) so SW-601 can close: SW-614 (M15), SW-615 (M16), SW-616 (M17), SW-617 (M18), SW-618 (L17 remaining).** Tracking = these tickets + the WAITING status in the findings file + Tram's emails to Eran; no separate reminder needed. Related: [[jira-ticket-style]], [[plans-in-claude-local]].
