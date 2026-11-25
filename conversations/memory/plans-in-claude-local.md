---
name: plans-in-claude-local
description: Save every plan to .claude.local/plans/ in the lucia repo as plan_<jira-ticket>_<task-name>_<plan-version>.md; implement only from that file
metadata:
  node_type: memory
  type: feedback
  originSessionId: 01c6ba1c-6b43-45ed-99e5-afaca4a762f5
  modified: 2026-11-19T05:53:35.366Z
---

Save every plan I generate to `.claude.local/plans/` in the lucia repo (git-ignored, local-only), named `plan_<jira-ticket>_<task-name>_<plan-version>.md` (e.g. `plan_SW-589_pr38-review-findings_v1.md`). The plan file is the source of truth: when something changes, update the plan first, then implement from it.

**Why:** The user reviews plans later and wants a durable, local record that the code follows; `.claude.local/` holds all Claude-related files and is never committed.

Workstream review notes (`.claude.local/reviews/`) must include a per-finding table of test cases: count as pytest collects them, file, what each covers, and whether it failed before the change (Tram's request, 2026-11-19).

**How to apply:** For a new task, write the plan there (not only in ~/.claude/plans). Keep `.claude.local/` in `.gitignore` on the working branch. Branch names follow the repo's `<owner>/<ticket>-<desc>` format (e.g. `tram/sw-589-pr38-findings`).
