---
name: always-push-branch
description: Always push the branch (with tags) after committing — Tram works on two machines
metadata:
  node_type: memory
  type: feedback
  originSessionId: f886a291-2d62-40f7-b16a-c6cc812d1c44
  modified: 2026-11-26T10:40:57.253Z
---

After every commit, push the branch to `origin` automatically (without asking), including tags (`git push --tags` or push the tags alongside). Tram works on two machines, so the branch must always be on the remote to sync between them.

**Why:** Unpushed local commits can't be picked up on the other machine; keeping `origin` current is how Tram moves work between machines.

**How to apply:** Commit, then push branch + tags right away. The PR is opened separately and later — pushing the branch does not require or imply opening a PR. Still no Claude co-author in commits per [[no-claude-coauthor]], and the review-checkpoint stop per [[review-checkpoint-per-workstream]] happens before the next workstream, not before the push.
