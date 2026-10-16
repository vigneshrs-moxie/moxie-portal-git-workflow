# Personal fork -> Moxie feature branch -> staging -> main

**This is the default proposed process for new Moxie Orbit work by a two-developer team (you and Raj).** Each task has one named branch in your fork and one *matching* branch in the Moxie organization. It is intentionally three PR stages: **you self-review PR #1 (an AI reviewer is optional), Raj approves PR #2 into staging, and the release owner handles PR #3 into production**. No coding agent merges or deploys.

## Worked example: Role Template copy feature

The exact branch name in both repositories is `feature/role-template-copy-v1`. The author's fork may have a **different repository name**; only the branch name is the same.

| Stage | GitHub PR | Gate / owner | Result |
| --- | --- | --- | --- |
| Prepare | No PR yet | Developer/agent verifies fork, Moxie remote, base SHA, plan and isolation | Fork task branch and Moxie integration branch are based on the correct Moxie environment baseline |
| 1. Intake | `<personal fork>:feature/role-template-copy-v1` -> `Moxie-Pest-Control-Phoenix/moxie-portal:feature/role-template-copy-v1` | Developer's **own review**, optional AI code review, local checks; developer **manually merges** | Code moves from personal fork to matching org branch; **not staged or released** |
| 2. Staging | `Moxie-Pest-Control-Phoenix/moxie-portal:feature/role-template-copy-v1` -> `Moxie-Pest-Control-Phoenix/moxie-portal:staging` | Up-to-date CI + security, **Raj's code review and recorded approval**, then human merge | Code becomes part of staging |
| Validate | No new PR | Developer/QA tests actual staging deployment; logs results and blockers | Release candidate has evidence of working behavior |
| 3. Release | `Moxie-Pest-Control-Phoenix/moxie-portal:staging` -> `Moxie-Pest-Control-Phoenix/moxie-portal:main` | Release PR lists every change and risk; Raj/authorized release owner approves; human merges/deploys | Release includes all staged changes shown in the diff |
| Confirm | No new PR | Release owner checks deployment, migration jobs and smoke tests | Ticket and changelog updated with real production status |

## 0. Agree scope and create both branches safely

- For every task, fetch and branch from the **authoritative Moxie `main`**, not an old fork copy and never `staging`. Production bugs are reproduced against `main`; work normally integrates through staging later (PR #2). See [`bugfix-process.md`](bugfix-process.md) for the urgent exception.
- Verify `git remote -v`: use explicit names for personal fork and Moxie remote (`origin` and `upstream` are common but not guaranteed). Verify that the fork is yours and the org URL is exactly `Moxie-Pest-Control-Phoenix/moxie-portal`.
- Create one task branch and a dedicated worktree on the **personal fork** from the verified baseline. Never develop by directly committing to the Moxie organization's shared feature branch.
- Create/reserve the **identically named Moxie organization branch** from the appropriate authoritative baseline, only if it does not already belong to another task. Creating a new org ref requires write permission and team authorization. Do not overwrite an existing remote branch; verify its SHA and owner.
- If either branch is not yet created, make sure the PR's base exists before opening it. Keep one human task owner; separate Claude agents get separate worktrees, not shared branch writers.

Conceptual setup, after verifying remotes (do not paste blindly):

```bash
git remote -v                           # check actual fork and Moxie URLs
git fetch upstream --prune             # only if upstream is verified Moxie
git worktree add -b feature/role-template-copy-v1 ../orbit-role-template-copy upstream/main
cd ../orbit-role-template-copy
# Push ONLY to your personal fork after authorization:
git push -u origin feature/role-template-copy-v1
# Have an authorized team member create the matching org branch
# from the same upstream/main SHA if it does not exist.
```

The org integration branch should start from the *same authorized baseline* as the fork task branch. Before any GitHub write, inspect the target ref and PR diff. Changes can move on `staging` while the task is in progress; reconcile them explicitly, never force-push or silently reset shared branches.

## 1. Code and test entirely in the personal fork

1. Implement the approved behavior in small, reviewable commits. Do not mix unrelated issues.
2. Add or update focused automated tests and a manual scenario checklist (especially roles, offices, error/edge paths).
3. Run relevant focused tests, then `cd app && npm run check` and `cd app && npm run build` where supported. Write down exact results and any missing setup; do not claim green without running them.
4. Inspect `git diff`, `git status`, and migration/auth/external integration impacts. Confirm no secrets, production data or unrelated changes are committed.
5. **Self-review the final change set before merging PR #1:** read the full fork-to-org PR diff (not only the last commit), check intended behavior, edge/error cases, permissions, security, migrations and unintended changes. Optionally request a review from an approved AI code-review service (such as Greptile) or a separate review agent. Fix actionable findings; document dismissed findings and why. AI review is helpful but **never required or a GitHub approval**.
6. Push to your **fork branch**, not to the Moxie feature/staging/main branches.

## 2. PR #1: fork branch -> matching Moxie branch

- Open the PR **in Moxie's repo** with the *head* `YOUR_FORK_OWNER:feature/role-template-copy-v1` and *base* `Moxie-Pest-Control-Phoenix/moxie-portal:feature/role-template-copy-v1`.
- Verify **both repository owners and both ref names**, not just the label. A same-name branch in two different repos is valid.
- Include ticket/spec, changes, proof of local tests, migrations, risks and screenshots. **You are the PR #1 reviewer**: inspect the exact final diff, run or read any optional AI-review report, resolve meaningful findings, and record a short self-review outcome in the PR body/comment. When satisfied and GitHub permits, **you manually merge PR #1** into the Moxie feature branch. **Raj's approval and a second human reviewer are not prerequisites here.** Do not try to submit an APPROVED review on your own PR; it does not count as an independent GitHub approval.
- A security scan on this PR is **not an all-tests-passed signal**. Run local checks and record the results.

Example `gh` arguments, after checking repo names, permissions and branch existence:

```bash
gh pr create --repo Moxie-Pest-Control-Phoenix/moxie-portal \
  --head YOUR_GITHUB_USERNAME:feature/role-template-copy-v1 \
  --base feature/role-template-copy-v1 \
  --title 'feat(scorecards): copy role templates'
```

If the fork is a differently named repository, confirm GitHub identifies the intended fork/head and shows the right diff; use the GitHub compare UI if necessary. Never assume a local `gh` default will target the org.

## 3. PR #2: organization feature branch -> staging

- After PR #1 has **actually merged**, open a new PR with head **Moxie org** `feature/role-template-copy-v1` and base **Moxie org** `staging`. Do not accidentally raise a second fork PR to staging.
- Confirm the diff contains only the intended feature, the branch is updated safely against current staging, and relevant migration numbers are still available.
- Run `git log --oneline upstream/staging..upstream/<your-branch>`: every commit must belong to this task. If not, see [`branch-sync.md`](branch-sync.md).
- The `staging`-target PR triggers the repo's lint, typecheck, tests, build, migration number check, and security scan. Require successful, *fresh* checks (and verify which checks are technically enforced).
- Request **Raj** for code review/approval on this staging PR. **His submitted APPROVED GitHub review, on the latest substantive changes, is mandatory**, regardless of any PR #1 self-review or AI report. Obtain specialist input for complex DB/auth/office logic if appropriate. New substantive commits after his approval require a fresh review.
- **An authorized human** merges only after Raj has approved and all blockers are resolved. The agent never self-approves, merges, or marks it approved.

## 4. Test the deployed staging environment

- Wait for the **actual staging deployment**, verify its commit and any automatic migration job result, and run a task-specific smoke test. A green PR check does not prove the deployed environment works.
- Test normal paths, permissions and office/entity isolation, failure paths, and integration dependencies where applicable. Record who tested what and the result; leave known failures as release blockers unless the release owner explicitly accepts the risk.
- If you discover a bug: make the fix in your fork and re-integrate through a PR to the **same Moxie feature branch** (or use a new approved fix branch if the original branch is retired), then another staging PR. Do not push a surprise fix directly onto staging.

## 5. PR #3: staging -> main is a *release summary*

A release PR promotes the **entire tested staging diff** against main, often containing several features/fixes. The PR body must include:

- Included feature/fix PRs and ticket links; what customers will notice.
- Staging deployment commit, test evidence, reviewers/signoff, and known issues.
- All DB migrations and correct before/after-deploy order, secrets/external configuration, feature flags and dependencies.
- Release owner, production trigger (verified with GitHub/Vercel owners), rollback/roll-forward, and post-deploy smoke tests.
- Checks freshly run against current `main`. If staging has moved, update the summary and **re-approve** the expanded release contents. **The PR author cannot approve their own release PR**. With two developers, the non-author may review, while the release owner confirms the production go/no-go.

**Never assume a staging -> main merge pauses for a manual deployment approval.** The release owner checks the active deployment and migration configuration.

## 6. Verify in production and close the loop

The release owner observes deployment and migration status and tests the affected flows. Then update the Orbit ticket with the staging PR, release PR, production commit/deployment and confirmation. Recommend cleaning up merged task branches **only when no active work/PR depends on them**; an agent does not delete shared refs by itself.
