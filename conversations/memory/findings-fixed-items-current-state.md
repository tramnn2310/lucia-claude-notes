---
name: findings-fixed-items-current-state
description: "In docs/pipeline_review_findings.md, a FIXED finding shows only its current state (status + short resolution + test); remove the reviewer's original text and Where line"
metadata:
  node_type: memory
  type: feedback
  originSessionId: 01c6ba1c-6b43-45ed-99e5-afaca4a762f5
  modified: 2026-11-19T13:24:49.993Z
---

In `docs/pipeline_review_findings.md` (SW-589), once a finding is FIXED its section shows only the current state: Status line, then a short Resolution saying what the code does now, any part still open, and the test that proves it. Remove Gilad's original finding text, the `Where:` line and the W0 triage history. Open findings keep the review text.

**Why:** Tram (2026-11-19): keeping the original finding text next to a fix is confusing for Avichai (tech lead) and Gilad (data scientist); they need to see what is fixed now.

**How to apply:** Do this in the same change that marks a finding FIXED. Write it in plain language for those two readers, and keep it short. Don't commit findings-file wording changes until Tram has reviewed them. Related: [[jira-ticket-style]], [[plans-in-claude-local]].
