# Branch names, fork remotes and baselines

## Naming for new branches

Every new internal task branch carries its **primary Orbit ticket**, which must exist before the branch does (see [README: Every new task starts with an Orbit ticket](README.md#every-new-task-starts-with-an-orbit-ticket)). People write the key as `ORBIT-123`; branches use lowercase `orbit-123`. Otherwise use short, lowercase kebab-case. Always use the **same task branch name in your fork and in Moxie's repo**.

| Work | Format | Fork AND org example |
| --- | --- | --- |
| Feature | `feature/orbit-<number>-<slug>-v<N>` | `feature/orbit-123-ticket-comments-v1` |
| Bug / hotfix | `fix/orbit-<number>-<slug>` | `fix/orbit-124-comment-save` |
| CI | `ci/orbit-<number>-<slug>` | `ci/orbit-126-migration-number-check` |
| Maintenance | `chore/orbit-<number>-<slug>` | `chore/orbit-127-update-dependencies` |
| Docs | `docs/orbit-<number>-<slug>` | `docs/orbit-125-ticket-linking` |
| Refactor | `refactor/orbit-<number>-<slug>` | `refactor/orbit-128-rating-query` |

**Example numbers are illustrative only**; never reuse them. Use your real ticket's number.

- **Why the ticket goes in the name:** anyone looking at a branch, worktree or PR head can find the ticket without opening anything, and Portal Feedback can match pushed branches to tickets.
- **Why lowercase `orbit-123`:** branch names stay all-lowercase kebab-case like the rest of the repo; `ORBIT-123` remains the human-facing key in titles, PR bodies and tickets.
- **`-vN` (features only)** still means a separately scoped **feature increment**, not an application release version. Use `-v2` for a separately planned second increment, not merely the second commit or PR on `-v1`. A `-v2` increment normally has its own ticket.
- **One focused task and one primary ticket per branch.** Closely related work may cover additional tickets; list them in the PR (**Additional Orbit tickets**). Never concatenate several ticket IDs into the branch name.
- **Same name, two repositories, two branches.** Branch identity is repository + full ref: `vigneshrs-moxie/moxie-portal-vrs:feature/orbit-123-ticket-comments-v1` and `Moxie-Pest-Control-Phoenix/moxie-portal:feature/orbit-123-ticket-comments-v1` are distinct refs that deliberately share a name.
- Do not fabricate a ticket number or use a placeholder such as `ORBIT-TBD`; there is no ticketless branch path for new internal work. Avoid collisions: check that the chosen name is not already owned by another task or agent.

**Existing and external branches are unchanged.** The format applies to new internal task branches only. Do not rename, rebase, force-push, delete or recreate existing branches (for example `feature/role-template-copy-v1`, `feature/orbit-tickets-v1` or unprefixed legacy branches) to comply; link active legacy work to its ticket through the ticket's fields and the PR body instead. Keep `main`, `staging`, `mirror/*`, `dependabot/*`, `revert-*` and `feat/*` branches under their current names and owners. Do not invent a `release/*` scheme; release strategy and versioning are undecided. Internal remediation arising from vendor or bot work (for example a fix after a Dependabot bump) is a normal ticketed task branch.

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
# ORBIT-123 exists and its ID/URL are in the starting-state report
git worktree add -b feature/orbit-123-ticket-comments-v1 ../orbit-ticket-comments upstream/main
cd ../orbit-ticket-comments
git branch --show-current
git status --short
```

For a bug, replace the feature branch with the appropriate `fix/orbit-<number>-<slug>` name; the base is still `upstream/main`. The org integration branch must be reserved/created by an authorized writer from the agreed baseline and must **not** be an unrelated or existing developer's branch. Check `git ls-remote --heads upstream <branch-name>` before creation; never overwrite a remote branch. Do not create a new worktree on top of an existing task worktree; inspect its owner and state first.

When code is ready, push **only to the personal fork** (after authorization). **PR #1** uses your fork as head and the same-named Moxie branch as base; **you self-review and manually merge it** after local tests (AI review optional). **PR #2** uses that Moxie branch as head and Moxie `staging` as base; **Raj's GitHub approval is required**. GitHub URLs and owner/ref matter: same-looking branch names in different repos are not a self-merge. After each push or PR, register the branch/PR URL on the Orbit ticket (README, *Link the work back to the ticket*).

See [`fork-to-release.md`](fork-to-release.md) for the end-to-end PR sequence and `gh` example.

## Parallel agents, shared branches and cleanup

One agent owns one task branch/worktree; coordinate analysis through docs or review comments, not multiple writers sharing the same Git working tree. Do not force-push/rebase shared refs, delete the org feature branch automatically, or reset someone else's fork branch. Human maintainers can clean up old integration branches only after all dependent PRs/worktrees are done. If org `staging` advances while PRs are open, reconcile deliberately, rerun relevant checks, and refresh human review.
