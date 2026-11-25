---
name: jira-ticket-style
description: "Jira tickets/sub-tasks for SW-589-style work must be short summaries (counts), not per-item detail; don't mention internal workstreams reviewers don't know"
metadata:
  node_type: memory
  type: feedback
  originSessionId: 01c6ba1c-6b43-45ed-99e5-afaca4a762f5
  modified: 2026-11-19T05:47:34.709Z
---

Keep Jira ticket and sub-task text short: state how many items were solved / are planned, and point to `docs/pipeline_review_findings.md` for the per-finding detail. Don't repeat finding-by-finding tables in Jira. Don't mention internal plan names (other workstreams like W7, PR numbering, side fixes) that Gilad and Avichai haven't been told about.

**Why:** Tram reviewed draft sub-tasks for SW-589 (2026-11-19): the findings tables were "too long", details already live in the findings file, and naming W7 "may be confusing to them".

**How to apply:** Draft Jira content in chat for Tram's review before creating it ([[plans-in-claude-local]] workflow). Lead with counts and the goal; one line per open decision at most.
