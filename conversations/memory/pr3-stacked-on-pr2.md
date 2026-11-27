---
name: pr3-stacked-on-pr2
description: "SW-589 PR 3 is stacked on PR 2's branch; revisit it when PR 2 merges and that branch is deleted"
metadata:
  node_type: memory
  type: project
  originSessionId: 73cc5d46-ce04-4ba5-8820-a52e9597ebf1
  modified: 2026-11-27T10:22:51.420Z
---

On 2026-11-27 Tram opened PR 3 (`tram/sw-589-training-defaults`, tip `99b7c1b`, 17 commits) with destination `tram/sw-589-preprocess-features` (PR 2, tip `7ff52eb`), because reviewers were off and SW-589 was due. Tram can't change PR 2's delete-branch-on-merge setting, and said we'll come back to PR 3 if that branch gets deleted.

**Why:** Not confirmed that Bitbucket Cloud retargets a PR whose destination branch is deleted.

**How to apply:** After PR 2 merges, check PR 3: destination must be `master`, remove the "Stacked on PR 2" line, never merge it into PR 2's branch. If the branch must be restored: `git push origin <PR2-merge-hash>^2:refs/heads/tram/sw-589-preprocess-features`. Pre-rebase PR 3 is tag `backup/training-defaults-prerebase`; the handover `.claude.local/handover/20261127.md` still says PR 3 needs a rebase, which is done. Related: [[always-push-branch]].
