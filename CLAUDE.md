# Moxie Orbit - instructions for coding agents

**Status: proposed; team review required before adoption.** This file is behavioral guidance. Repository/branch permissions must enforce the security gates.

Read `docs/engineering/git-workflow/README.md` and **`fork-to-release.md`** first, then `branch-naming.md`, `agent-rules.md`, and the appropriate feature/bug process.

## Required path for new development

0. **Orbit ticket first.** Every new internal task (feature, bug, urgent fix, code-producing investigation, CI, chore, refactor, docs) needs a real Orbit ticket before its branch exists. Record ticket ID (`ORBIT-123`), ticket URL, task owner and agreed scope in the starting-state report. Until the Portal Feedback API/MCP exists the human creates/finds it manually in Portal Feedback; afterwards use only that documented, authorized interface. Never invent an ID or use `ORBIT-TBD`; without a ticket, stay read-only. A ticket does not authorize scope, execution, PR publication, merge or release.
1. Verify the Moxie organization's repo, personal fork, `git remote -v`, current branch, worktree, clean state and authoritative starting SHA. Do not assume `origin` is Moxie.
2. **Code in a dedicated worktree and branch in the developer's personal fork.** Create/reserve the **same exact branch name** in Moxie's repo from the correct base: `feature/orbit-<number>-<slug>` for features, `fix/orbit-<number>-<slug>` for bugs/hotfixes, and `ci/`, `chore/`, `docs/`, `refactor/` + `orbit-<number>-<slug>` (see `branch-naming.md`). Branch creation on the org requires authorized ownership; never overwrite an existing branch.
3. **Every task branch starts from the latest Moxie `main`, never `staging`** (features, bugs, CI/chore/docs). Production bugs are reproduced against `main` with production-like data; staging-only bugs are reproduced on staging, but the fix branch still starts from `main` (if the affected code is not on `main` yet, ask the human how to route it). `staging` is the integration target for PR #2, not a starting point.
4. Get the scope/plan approved, implement focused changes and tests, check diff for secrets and unrelated files. Run `cd app && npm run check` and `cd app && npm run build` where the local environment permits; state what was actually run. **Before PR #1 is merged, supply a code-review report:** inspect the entire diff, optionally use an approved AI review tool (e.g. Greptile), fix or explicitly disposition findings, and hand the report to the developer for their own review.
5. With authorization, push the task branch to the **personal fork**, then register the branch URL on the ticket (authorized API/MCP when available; until then the human pastes it into the ticket's **Branches** field). Do the same for each PR in the **Pull requests** field. Registering a link never changes ticket status or posts a shared comment. Prepare **PR #1: personal fork branch -> matching Moxie org branch**. **The developer reviews their own PR, optionally assisted by a code-review agent, and may manually merge it into the org task branch after checks and self-review. Raj is NOT required at this first stage.** Do not mistake secret scanning for passing tests. Self-review is not a formal GitHub APPROVED review.
6. Once the developer has merged PR #1, prepare **PR #2: Moxie org branch -> Moxie `staging`** (normal route). Require up-to-date CI/security checks and **Raj's recorded GitHub code review/approval** before a human merges. Never bypass this reviewer gate or auto-merge.
7. Record manual validation on the deployed staging environment. A human release owner prepares **PR #3: Moxie `staging` -> `main`** with every included change, test evidence, migrations, risks and rollback. `main` currently requires at least one eligible **GitHub approval from someone other than the PR author**; the release owner coordinates approval, merge and production verification.
8. **Urgent production hotfix exception:** record the incident ticket promptly (short intake) before creating the fix branch; keep the `main` baseline, still use fork -> matching org fix branch, but the human may approve an org-fix -> `main` PR after safe preview testing; then sync `main` back into `staging` by reviewed PR. Agents never choose this shortcut unilaterally.

## Non-negotiable safeguards

- One task and one primary Orbit ticket per named branch, one writer per worktree. Existing branches remain grandfathered; don't rename, rebase, force-push, delete or recreate them to fit this proposal, and keep `main`, `staging`, `mirror/*`, `dependabot/*` and revert branches as they are.
- Stop for human input before changing DB migrations, roles/auth/office visibility, shared-environment data, secrets, production infrastructure, or an unapproved scope.
- Don't silently modify GitHub rules, activate deployments, run shared-database migrations, push into `staging`/`main`, merge PRs, approve PRs or deploy.
- Human authorization is required before publishing a PR unless specifically authorized by the user's request. At PR #1, the developer's documented self-review (with optional AI review) is the human go/no-go decision; the coding agent never merges. At PR #2, **Raj personally reviews and approves in GitHub**. A positive agent review or author self-review never substitutes for Raj's staging approval.
- Check PR **head repository + ref** and **base repository + ref** at every stage, especially when fork/org branches have matching names.
- Treat a release PR as a production operation; repository CI contains production migration automation, and live Vercel deployment behavior needs owner verification.
- On completion, report the Orbit ticket ID/URL (plus additional tickets), exact fork and Moxie branch URLs, exact base SHA, checks and results, three PR links/statuses (if applicable), blockers, risks and what human action remains.

## Supporting guidance

- `docs/engineering/git-workflow/fork-to-release.md` - PR #1/#2/#3 and human gates.
- `docs/engineering/git-workflow/branch-naming.md` - naming, forks, worktrees, and remotes.
- `docs/engineering/git-workflow/feature-process.md` and `bugfix-process.md` - task-specific workflow.
- `docs/engineering/git-workflow/pull-requests.md` - evidence and review requirements.
- `docs/engineering/git-workflow/release-and-migrations.md` - production and migration risks.
- `docs/engineering/git-workflow/agent-rules.md` - agent stop conditions and handoff.
