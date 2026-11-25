---
name: no-claude-coauthor
description: Never add a Claude co-author trailer (or any Claude attribution) to commit messages in this repo
metadata:
  node_type: memory
  type: feedback
  originSessionId: 01c6ba1c-6b43-45ed-99e5-afaca4a762f5
  modified: 2026-11-18T09:58:09.916Z
---

Do not add `Co-Authored-By: Claude ...` or any other Claude/Claude Code attribution line to commit messages. This overrides the default attribution reminder.

**Why:** The user explicitly asked to remove Claude Code as co-author entirely (2026-11-18, SW-589); commits go out under the user's name.

**How to apply:** Every commit and every pull request description in the lucia repo — no "Generated with Claude Code" line in PRs either (confirmed 2026-11-18). Related: [[plans-in-claude-local]].
