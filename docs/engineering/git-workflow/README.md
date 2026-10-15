# Git workflow for Moxie Orbit

**Status: proposal for Raj/team review (2026-10-12). Not yet enforced.** Nothing in GitHub was changed by this document.

## The normal route: three PR stages

Every new development task starts in the developer's **personal fork**, on a dedicated branch. The organization holds an identically named integration branch. Each stage is a separate GitHub PR:

```text
Moxie main (starting baseline for every task)
    |                         Same branch name in TWO repositories
    +--> personal fork: feature/role-template-copy-v1
    |       code + local checks
    |       PR #1: fork feature/... -> Moxie feature/...  (developer self-review; optional AI review)
    +--> Moxie org:   feature/role-template-copy-v1
            PR #2: Moxie feature/... -> Moxie staging  (CI + Raj's code review/approval)
          Moxie staging
            deploy + documented staging QA/signoff
            PR #3: Moxie staging -> Moxie main  (release summary + human approval)
          Moxie main
            verify deployment/migrations + production smoke test
```

**No agent merges any of the three PRs.** Humans authorize merges and production deployment. The two matching branch names live in *different repositories*; the PR's full head/base repository and branch must always be verified.

Detailed steps, commands and gates: **[`fork-to-release.md`](fork-to-release.md)**.

## Choosing a baseline

**Rule (decided 2026-10-12): every fork and org task branch starts from the latest Moxie `main`, never `staging`.** `staging` can carry unreviewed or unreleased work, and numbered migrations collide when branches start from it (this matches `app/prisma/migrations/README.md`: "Branch off `main`. Not off `staging`"). `staging` is only the PR #2 integration target.

| Work | Fork + Moxie integration branches start from | Normal promotion path | Exception |
| --- | --- | --- | --- |
| New feature | Current Moxie `main` | fork -> matching Moxie branch -> `staging` -> `main` | Authorized cross-team integrations |
| Nonurgent production bug | Current Moxie `main` after reproducing the production behavior | fork -> matching Moxie `fix/*` -> `staging` -> `main` | Validate against the main baseline too; staging may differ |
| Staging-only bug | Current Moxie `main` (reproduce on staging; if the affected code is not on `main` yet, ask the human how to route the fix) | fork -> matching Moxie `fix/*` -> `staging` -> later release | None normally |
| Urgent production hotfix | Current Moxie `main` | fork -> matching Moxie `fix/*` -> `main` with emergency human approval | Sync the result back to `staging` by reviewed PR |
| CI, chore, docs | Current Moxie `main` | same fork -> org branch intake; normally then `staging` | Explicitly documented exceptions |
| CadreAI mirror, Dependabot, revert | Existing owner-maintained process | As authorized | Do not rename or route these blindly |

`main` represents the production baseline; `staging` contains changes not necessarily in production. Do not branch production bug fixes from staging just because it is ahead. **Never infer that merging to `main` is only a Git operation:** repo CI runs production migration jobs and Vercel may auto-deploy.

## The three review gates

1. **Fork -> matching org branch (your gate):** Development complete; focused checks plus `cd app && npm run check` and `npm run build` performed where feasible. **The developer inspects their own PR diff**, optionally runs an approved code-review agent (Greptile or equivalent), resolves or documents findings, and records test results. The developer may **manually merge** PR #1 into the org task branch if GitHub permissions/checks permit. **No separate human reviewer or Raj approval is required at PR #1.**
2. **Org branch -> `staging` (Raj's gate):** PR is up to date with the integration target; CI and security checks pass; **Raj performs code review and submits a GitHub APPROVED review on the latest material changes** before an authorized human merges. Test the resulting staging deployment, including auth/office and DB scenarios where relevant. Your PR #1 self-review or AI review is *not* a substitute.
3. **`staging` -> `main` (release gate):** A single release-summary PR inventories **all** included feature/fix PRs, staging test evidence, migrations, external configuration, known limitations, rollback/roll-forward and production verification. The release owner authorizes promotion; `main` requires at least one eligible GitHub approval **from someone other than the PR author**. An authorized human merges and supervises deployment. No feature code is written directly in the release PR.

This is the **proposed two-developer review policy**: the author owns the PR #1 self-review (AI assistance optional); Raj is the required reviewer for PR #2; release approval is a separate production gate. GitHub does not let a PR author satisfy required approval by approving their own PR.

## Read in this order

1. [`fork-to-release.md`](fork-to-release.md) - full personal-fork-to-production walkthrough.
2. [`branch-naming.md`](branch-naming.md) - naming, matching branch names, fork/upstream remotes and worktrees.
3. [`feature-process.md`](feature-process.md) or [`bugfix-process.md`](bugfix-process.md) - task-specific steps.
4. [`pull-requests.md`](pull-requests.md) - each PR's reviewers, checks and merge method.
5. [`release-and-migrations.md`](release-and-migrations.md) - staging, production and DB safety.
6. [`agent-rules.md`](agent-rules.md) and root [`CLAUDE.md`](../../../CLAUDE.md) - agent permissions and stop conditions.
7. [`claude-prompts.md`](claude-prompts.md) - sample agent prompts.

## Scope

The project is not adopting GitFlow, separate release branches, a merge queue, autonomous agent merges, or new migration naming through this documentation change.
