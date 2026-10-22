# Git workflow for Moxie Orbit

**Status: proposal for Raj/team review (2026-10-12; ticket rule added 2026-10-22). Not yet enforced.** Nothing in GitHub was changed by this document.

## The normal route: three PR stages

Every new development task starts with an **Orbit ticket**, then a dedicated branch in the developer's **personal fork**. The organization holds an identically named integration branch. Each stage is a separate GitHub PR:

```text
Moxie main (starting baseline for every task)
    |                         Same branch name in TWO repositories
    |   (Orbit ticket ORBIT-123 exists first)
    +--> personal fork: feature/orbit-123-ticket-comments-v1
    |       code + local checks
    |       PR #1: fork feature/... -> Moxie feature/...  (developer self-review; optional AI review)
    +--> Moxie org:   feature/orbit-123-ticket-comments-v1
            PR #2: Moxie feature/... -> Moxie staging  (CI + Raj's code review/approval)
          Moxie staging
            deploy + documented staging QA/signoff
            PR #3: Moxie staging -> Moxie main  (release summary + human approval)
          Moxie main
            verify deployment/migrations + production smoke test
```

**No agent merges any of the three PRs.** Humans authorize merges and production deployment. The two matching branch names live in *different repositories*; the PR's full head/base repository and branch must always be verified.

Detailed steps, commands and gates: **[`fork-to-release.md`](fork-to-release.md)**.

## Every new task starts with an Orbit ticket

**Rule (2026-10-22):** every new internal Orbit development task needs a **real Orbit ticket before its task branch is created or implementation begins**. That covers features, bugs, urgent fixes, investigations that produce code, CI, maintenance, refactors and documentation.

- Record the canonical ticket ID (`ORBIT-123`), the ticket URL, the task owner and the agreed scope in the starting-state report ([`agent-rules.md`](agent-rules.md)).
- A ticket records the work. It does **not** authorize the proposed scope, execution, PR publication, merge or release; those keep their existing gates.
- **Until the Portal Feedback API/MCP is available,** find or create the ticket manually in Portal Feedback. **Once it is available,** use only its documented, authorized interface. Never invent a ticket number, use `ORBIT-TBD`, or work around missing permissions. With no ticket yet, finish read-only preparation, get the ticket, then create the branch.
- **Urgent incidents:** keep intake short, but record the incident ticket before creating the fix branch. Creating a ticket neither replaces nor bypasses Raj's emergency release authority.
- **Link the work back to the ticket.** After a branch is pushed or a PR is opened, register it: through the authorized API/MCP once it exists, until then by pasting the URLs into the ticket's separate **Branches** and **Pull requests** fields in Portal Feedback. Registering a link never advances the ticket's status or posts a shared comment by itself. *Linked*, *merged*, *requirement approved* and *available in production* are four separate events.
- **Compatibility:** the rule applies to new internal task branches only. Do not rename, rebase, force-push, delete or recreate existing branches to comply; associate active legacy work through ticket links and PR fields. `main`, `staging`, `mirror/*`, `dependabot/*`, revert branches and other historical or external branches keep their names and owners. Sync and release PRs get traceability in their summaries, not ticket IDs in environment branch names. New internal remediation arising from vendor or bot work uses the normal ticket-bearing task branch.
- **Adoption:** required now for the workflow owner's own new work. Wider team and CadreAI adoption needs their agreement; these docs do not change CadreAI's independent review or production permissions.

Branch formats: [`branch-naming.md`](branch-naming.md). PR fields: [`pull-requests.md`](pull-requests.md).

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

1. **Fork -> matching org branch (your gate):** Development complete for the ticketed scope; focused checks plus `cd app && npm run check` and `npm run build` performed where feasible. **The developer inspects their own PR diff**, optionally runs an approved code-review agent (Greptile or equivalent), resolves or documents findings, and records test results. The developer may **manually merge** PR #1 into the org task branch if GitHub permissions/checks permit. **No separate human reviewer or Raj approval is required at PR #1.**
2. **Org branch -> `staging` (Raj's gate):** PR is up to date with the integration target; CI and security checks pass; **Raj performs code review and submits a GitHub APPROVED review on the latest material changes** before an authorized human merges. Test the resulting staging deployment, including auth/office and DB scenarios where relevant. Your PR #1 self-review or AI review is *not* a substitute.
3. **`staging` -> `main` (release gate):** A single release-summary PR inventories **all** included feature/fix PRs, staging test evidence, migrations, external configuration, known limitations, rollback/roll-forward and production verification. The release owner authorizes promotion; `main` requires at least one eligible GitHub approval **from someone other than the PR author**. An authorized human merges and supervises deployment. No feature code is written directly in the release PR.

This is the **proposed two-developer review policy**: the author owns the PR #1 self-review (AI assistance optional); Raj is the required reviewer for PR #2; release approval is a separate production gate. GitHub does not let a PR author satisfy required approval by approving their own PR.

## Read in this order

1. [`fork-to-release.md`](fork-to-release.md) - full personal-fork-to-production walkthrough.
2. [`branch-naming.md`](branch-naming.md) - naming, matching branch names, fork/upstream remotes and worktrees.
3. [`feature-process.md`](feature-process.md) or [`bugfix-process.md`](bugfix-process.md) - task-specific steps.
4. [`pull-requests.md`](pull-requests.md) - each PR's reviewers, checks and merge method.
5. [`release-and-migrations.md`](release-and-migrations.md) - staging, production and DB safety.
6. [`branch-sync.md`](branch-sync.md) - daily routine, keeping `staging` in step with `main`, and the "only my commits" check before PR #2.
   - [`staging-conflicts.md`](staging-conflicts.md) - what to do when PR #2 shows conflicts with `staging`, step by step.
7. [`agent-rules.md`](agent-rules.md) and root [`CLAUDE.md`](../../../CLAUDE.md) - agent permissions and stop conditions.
8. [`claude-prompts.md`](claude-prompts.md) - sample agent prompts.

## Scope

The project is not adopting GitFlow, separate release branches (no `release/*` scheme while release strategy and versioning are undecided), a merge queue, autonomous agent merges, GitHub enforcement of ticket IDs, or new migration naming through this documentation change.
