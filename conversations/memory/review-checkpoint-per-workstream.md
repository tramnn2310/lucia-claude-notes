---
name: review-checkpoint-per-workstream
description: "For multi-workstream tasks, stop after committing each workstream and wait for explicit user approval before starting the next"
metadata:
  node_type: memory
  type: feedback
  originSessionId: 01c6ba1c-6b43-45ed-99e5-afaca4a762f5
  modified: 2026-11-18T09:44:15.748Z
---

When a plan splits work into workstreams, commit one workstream, tag it (`<ticket>-W<n>-start` / `-review`, local tags), write a review note in `.claude.local/reviews/review_<ticket>_W<n>.md` (commits, files, test evidence, decisions, questions, git commands to review), then STOP. Start the next workstream only after the user explicitly approves (e.g. "W1 approved"). Don't push before approval; address feedback with new commits, not history rewrites.

**Why:** The work touches many files; the user wants to review each workstream before more changes pile on.

**How to apply:** Applies to SW-589 and similar large tasks. Related: [[plans-in-claude-local]].
