# Branch names, fork remotes and baselines

## Naming for new branches

Use short, lowercase kebab-case. The `-vN` suffix identifies a separately scoped **feature increment** (not an application release). Always use the **same task branch name in your fork and in Moxie's repo**.

| Work | Format | Fork AND org example |
| --- | --- | --- |
| Feature | `feature/<slug>-v<N>` | `feature/role-template-copy-v1` |
| Bug with ticket | `fix/<ticket-id>-<slug>` | `fix/orbit-123-rating-submit` |
| Bug without ticket | `fix/<slug>` | `fix/talent-ratings-save` |
| CI | `ci/<slug>` | `ci/migration-number-check` |
| Maintenance | `chore/<slug>` | `chore/update-dependencies` |
| Docs | `docs/<slug>` | `docs/git-workflow` |
| Refactor | `refactor/<slug>` | `refactor/rating-query` |

Do not fabricate a ticket number. Use `-v2` for a separately planned second increment, not merely the second commit or PR on `-v1`. Avoid collisions: check that the chosen name is not already owned by another issue or agent. Keep existing `main`, `staging`, `mirror/*`, `dependabot/*`, `revert-*`, `feat/*` and unprefixed legacy branches unchanged.

## Choose starting baseline and destination

| Case | Starting Moxie ref | First PR base | Second PR base | Last stage |
| --- | --- | --- | --- | --- |
| Normal feature | `main` | **Matching org `feature/*` branch** | `staging` | Release PR `staging` -> `main` |
| Nonurgent production fix | `main` | **Matching org `fix/*` branch** | `staging` (after testing both baselines as appropriate) | Release PR `staging` -> `main` |
| Staging-only fix | `main` (reproduce on staging; ask the human if the affected code is not on `main` yet) | **Matching org `fix/*` branch** | `staging` | Next approved release |
| Urgent production fix | `main` | **Matching org `fix/*` branch** | `main` with human emergency approval | Reviewed sync from `main` -> `staging` |

Why everything starts on `main`: `staging` can contain unreviewed or unreleased changes, so it is not a valid baseline, and numbered migrations collide when branches start from it. Nonurgent fixes still normally pass through staging for integration testing. The release owner handles the urgent exception.

## Which repository owns what?

- **Personal fork:** developer/agent writes code, commits and runs tests on its task branch.
- **Moxie repo:** an identically named integration branch receives code only via the **developer-self-reviewed, manually merged** fork -> org PR; then that org branch is the head of the next PR into staging for **Raj's code review and approval**.
- **Environment branches:** `staging` and `main` accept only authorized, reviewed merges (agents never push or merge directly).
- Fork repository *names* may differ (`vigneshrs-moxie/moxie-portal-vrs` is a historical example). What matches is the branch ref, not necessarily the repo name.

## Verify Git remotes and use a dedicated worktree

Never assume `origin` points to Moxie; ideally `origin` = personal fork and `upstream` = Moxie, but **verify URLs** before acting. For a normal feature (commands are examples, not blindly executable automation):

```bash
git status --short
git remote -v
git worktree list
# Verify upstream points to Moxie-Pest-Control-Phoenix/moxie-portal
# and origin points to YOUR personal fork.
git fetch upstream --prune
git worktree add -b feature/role-template-copy-v1 ../orbit-role-template-copy upstream/main
cd ../orbit-role-template-copy
git branch --show-current
git status --short
```

For a bug, replace the feature branch with the appropriate `fix/...` name; the base is still `upstream/main`. The org integration branch must be reserved/created by an authorized writer from the agreed baseline and must **not** be an unrelated or existing developer's branch. Check `git ls-remote --heads upstream <branch-name>` before creation; never overwrite a remote branch. Do not create a new worktree on top of an existing task worktree; inspect its owner and state first.

When code is ready, push **only to the personal fork** (after authorization). **PR #1** uses your fork as head and the same-named Moxie branch as base; **you self-review and manually merge it** after local tests (AI review optional). **PR #2** uses that Moxie branch as head and Moxie `staging` as base; **Raj's GitHub approval is required**. GitHub URLs and owner/ref matter: same-looking branch names in different repos are not a self-merge.

See [`fork-to-release.md`](fork-to-release.md) for the end-to-end PR sequence and `gh` example.

## Parallel agents, shared branches and cleanup

One agent owns one task branch/worktree; coordinate analysis through docs or review comments, not multiple writers sharing the same Git working tree. Do not force-push/rebase shared refs, delete the org feature branch automatically, or reset someone else's fork branch. Human maintainers can clean up old integration branches only after all dependent PRs/worktrees are done. If org `staging` advances while PRs are open, reconcile deliberately, rerun relevant checks, and refresh human review.
