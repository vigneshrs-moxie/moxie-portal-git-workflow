# Daily routine and branch sync

**One rule makes the rest work: `staging` always contains everything in `main`.** When that holds, a branch cut from `main` shows only its own commits in a PR to `staging`. When it breaks, the PR drags in other people's unreleased work. This is what happened to #80, which brought 29 unrelated files and was replaced by #92 (fork -> org) and #94 (org -> staging).

```text
main ──────●──────────●────────────────●──
            \          \ daily sync PR  ↑
staging ─────●──────────●──────●───────┘ release PR (staging -> main)
                              ↑
feature/xyz (from main) ──●──●┘ PR #1 fork -> org branch, PR #2 -> staging
```

## Who moves what

| Flow | How | Who |
| --- | --- | --- |
| `main` → `staging` | Automatic PR every weekday at 7:00 Phoenix time. **Proposed:** the Action is in [`templates/sync-main-into-staging.yml`](templates/sync-main-into-staging.yml) and the org repo owner adds it to `.github/workflows/`. Merge commit, never squash. | A human merges it. If there are conflicts, whoever owns the conflicting change resolves them |
| feature → `staging` | PR #1 and PR #2, as in [`fork-to-release.md`](fork-to-release.md) | Developer, then Raj approves |
| `staging` → `main` | Release PR (PR #3), see [`release-and-migrations.md`](release-and-migrations.md) | Release owner |
| `main`/`staging` → your feature branch | **Don't** do this routinely. Bring the target in only when there's a conflict or right before opening a PR | Developer |

## Developer routine

1. **Start of day:** `git fetch upstream --prune`. If the sync PR is open, it's fine to keep working.
2. **New task:** worktree and branch from `upstream/main` (see [`fork-to-release.md`](fork-to-release.md) §0). Don't branch from `staging`, and don't branch from another feature branch.
3. **Work:** small commits on your fork branch only.
4. **Before PR #2 (to staging), run the "only my commits" check:**
   ```bash
   git fetch upstream
   git log --oneline upstream/staging..HEAD   # every line must be your task's commit
   ```
   - If other people's commits appear, `main` is ahead of `staging`. Get the sync PR merged first, then check again. If you can't wait, rebuild the branch from `upstream/staging` by cherry-picking your commits. Ask before force-pushing anything already shared.
   - Check that the PR's **Files changed** count on GitHub matches what you expect.
   - If PR #2 shows **conflicts** with `staging`, follow [`staging-conflicts.md`](staging-conflicts.md).
5. **After merge to staging:** test on deployed staging ([`release-and-migrations.md`](release-and-migrations.md)).

## Not decided yet

- Adding the sync Action to the org repo (needs the repo owner).
- Whether the sync PR needs Raj's approval like other staging PRs, or can be merged by any developer once CI passes.
- Long term: dropping the `staging` branch in favor of Vercel preview deploys and PRs straight to `main` (trunk-based). Revisit once the team grows.
