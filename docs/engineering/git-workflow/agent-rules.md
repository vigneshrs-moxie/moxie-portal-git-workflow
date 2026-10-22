# Claude Code agent rules for the fork-first workflow

**These are agent instructions, not GitHub enforcement.** Human review, branch protection and deployment permissions must enforce the important boundaries. Read [`fork-to-release.md`](fork-to-release.md) before pushing branches or creating PRs.

## Report the starting state before editing

```text
Primary Orbit ticket: ORBIT-<number> + ticket URL (real ticket; never invented or ORBIT-TBD)
Additional Orbit tickets: IDs + URLs, or None
Task owner: ...
Agreed scope (ticket does not itself authorize it): ...
Kind: feature | production bug | staging-only bug | urgent incident | CI | maintenance | docs | refactor
Moxie authoritative repository: Moxie-Pest-Control-Phoenix/moxie-portal
Verified personal fork repo + remote: ...
Verified Moxie remote: ...
Based on: Moxie/main @ SHA
Fork worktree / task branch: ...
Matching Moxie integration branch: ... (exists? owner? base SHA?)
Next PR: personal fork:branch -> Moxie:branch
Uncommitted changes and related existing PRs: ...
Approved scope / test cases: ...
```

If there is no real ticket yet, finish read-only preparation and stop for the ticket (manual Portal Feedback today; the documented API/MCP once it exists). Never scrape around missing permissions. Never use `origin/main` or `origin/staging` as the authoritative baseline until the remote URL is verified. Do not overwrite existing org refs or modify another developer/agent's worktree.

## Default responsibilities

**Agent/developer can do, within approved task scope:**

- Inspect tickets, docs, code and history; propose acceptance criteria, tests and a plan.
- Once the Portal Feedback API/MCP is available and authorized, create/find tickets and register pushed branches and created PRs through it. Until then, give the human the exact URLs to paste into the ticket's **Branches** and **Pull requests** fields.
- Create a personal-fork task branch in a dedicated worktree, implement changes, write tests, and make local commits.
- Run safe local tests/builds, inspect diffs, **perform and document an AI-assisted code review when requested**, prepare clear PR bodies and release notes. Flag findings for the developer; do not decide on behalf of the human that the review gate is satisfied.
- With specific authorization, push to the **personal fork** and open PR #1 for the **developer's own review** (optional independent code-review agent). The developer may manually merge it if checks pass and GitHub allows. After it merges, prepare PR #2 from the **org integration branch** to `staging` and request Raj's review. Draft PR #3 release notes without merging.

**Explicit human authorization is needed for:**

- Scope/design decisions beyond a minor implementation detail; branch creation in Moxie's org; pushing/publishing PRs if not already requested.
- DB migrations/data updates, auth/RBAC/office scope, production credentials, external write-capable integrations, Vercel/GitHub configuration, CI, branch rules and shared-environment operations.
- Stacking PRs, branching from an unusual base, selecting an urgent direct-to-main path, or any production rollout.

**Agent must never do autonomously:**

- Directly push/merge to `staging` or `main`, **submit GitHub approval on behalf of any human**, or report Raj approved without a real submitted APPROVED GitHub review. Reviewing code with an AI agent is allowed; using that review as a substitute for human authorization is not.
- Merge **any** PR stage (#1, #2, #3), bypass checks, force-push/reset/delete shared branches, or modify another person's commits/worktree.
- Deploy, apply migrations to staging/production, change live production data, disclose secrets or work around security restrictions.
- Mark tickets 'deployed' because a branch or PR merged; actual deployment and verification are separate.
- Advance a ticket's status or post a shared ticket comment as a side effect of registering a branch/PR link. Linking, merging, requirement approval and production availability are separate events.

## Three handoffs

1. **PR #1: fork -> matching org task branch.** Agent shows exact repositories/refs, local test evidence and a full-diff review report. **The developer personally reviews, handles optional AI-review findings and manually merges; Raj is not required.**
2. **PR #2: org task branch -> staging.** Agent gathers complete test/diff/migration evidence and requests Raj's code review. Fresh passing CI/security and Raj's **submitted GitHub approval** are required **before human merge**. A change after approval needs renewed review.
3. **PR #3: staging -> main.** Agent may draft release summary from the actual diff, but human release owner authorizes merge/deployment after documented staging QA and migration/deploy review.

An urgent production bug may change **PR #2 target to `main`**, but only on explicit human incident/release authorization, with later reviewed `main` -> `staging` synchronization.

## Standard task flow

1. Confirm task type; the baseline is always the latest Moxie `main` (never `staging`).
2. Confirm the real Orbit ticket (ID, URL, owner, scope); check duplicate/already fixed tickets, relevant product docs and affected code.
3. Get approval on scope; choose the `<type>/orbit-<number>-<slug>` branch name, reserve identical org branch via human authorization, create isolated fork worktree.
4. Implement minimum complete change; tests for happy path, errors and access boundaries.
5. Run the checks; examine the **full diff** for secrets, sensitive DB/role/office changes, scope expansion and generated files; prepare a review report and invite an approved AI review agent if the developer chooses.
6. Prepare PR #1 and stop for authorization if pushing/opening wasn't requested. The **developer reviews and may manually merge** after documenting their findings; the coding agent cannot. Do not mistake a green secret scan for full CI.
7. After the developer has merged PR #1, prepare PR #2 to staging, request **Raj's GitHub code review and approval**, and wait for approval/CI. Don't click merge.
8. After staging deployment, help record test outcomes and prepare release notes; never claim production deployed until verified.

## Multi-agent coordination

One task = one owned fork branch = one owned org integration branch = one writer/worktree. Helpers can research or prepare patches, but must not independently push conflicting commits to the same branch. Ask a human to resolve pre-existing branches that appear owned by someone else.

## Stop signs

- **Migration:** verify current four-digit numbering, stale-base collisions and *before/after deploy* order. The timestamp migration redesign is not adopted yet.
- **RBAC/entity scope:** test forbidden-office/role access, not just successful access.
- **Orbit tickets:** internal notes and assignee data must not leak to reporters; check attachment and notification failure modes.
- **Third-party integrations:** validate idempotency, retries, timeouts and service credential scope.
- **Releases:** do not infer automatic vs gated Vercel deployment solely from workflow YAML.

## Final agent handoff format

```text
Primary Orbit ticket: ORBIT-<number> / ticket URL
Additional Orbit tickets: IDs + URLs, or None
Fork branch URL / worktree: ...
Moxie integration branch URL: ...
Ticket links registered: Branches [...], Pull requests [...] (via API/MCP, or pasted by human, or still to do)
Authoritative base: main/staging @ SHA
Changed files and expected behavior: ...
Tests/checks: passed [...], failed/skipped [...] with reasons
PR #1 (fork -> org): URL / status / developer self-review / optional AI findings / manual-merge decision
PR #2 (org -> staging or authorized main hotfix): URL / status / Raj review / CI
PR #3 (staging -> main): URL / status / staging QA / release owner
Risks, migrations and unresolved decisions: ...
Production deployed and verified? no / confirmed by owner with reference
```
