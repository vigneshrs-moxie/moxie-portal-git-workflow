# Claude Code prompts: fork-first Git workflow

These prompts require the root `CLAUDE.md` and `docs/engineering/git-workflow/` to be present. They distinguish analysis, implementation, PR #1, PR #2 and release PR #3. Agents do not approve, merge or deploy.

## A. Plan a feature (read-only)

```text
Read CLAUDE.md, docs/engineering/git-workflow/README.md and fork-to-release.md. Plan this Moxie Orbit feature: [DESCRIPTION]. Do not edit code or create branches yet.

Verify the Moxie repo and my personal fork remotes. Inspect current Moxie main, code, related tickets and plans. Suggest a unique feature/<slug>-v1 name to use in BOTH repositories. Give concise scope, acceptance tests, risk, migrations/config changes, and exact proposed fork -> org branch -> staging -> main PR route. Stop for my plan approval.
```

## B. Implement the approved feature in the personal fork

```text
Implement approved feature [PLAN/TICKET] using the Moxie Git workflow. Read CLAUDE.md, fork-to-release.md, branch-naming.md, feature-process.md and agent-rules.md.

First show verified fork and Moxie remote URLs, current branch/worktree, latest authoritative Moxie main SHA, and a branch name such as feature/role-template-copy-v1. Create an isolated worktree on that branch in MY PERSONAL FORK from Moxie main. Do not write feature code directly onto any organization branch. Confirm the matching Moxie branch exists or ask for authorization to reserve it; never overwrite an existing branch.

Implement only approved behavior with automated tests. Run focused tests, cd app && npm run check, and cd app && npm run build as appropriate. Review the FULL diff and migrations, report code-quality/security risks, and optionally request a separate AI code review if I have approved its use. Prepare PR #1 FROM my fork branch INTO the matching Moxie organization branch (same exact branch name). Present the self-review checklist/findings for MY final review and manual merge. Ask before pushing/opening unless already authorized. Do not merge or deploy.
```

## C. Prepare PR #1 (personal fork -> matching Moxie branch)

```text
Review the current personal-fork branch. Verify its exact repository/ref and the matching Moxie organization branch repository/ref are different repositories but have the SAME task branch name. Compare with the authorized baseline. Report actual local tests, full diff, risks, secrets and migrations. Fill in the PR template and propose PR #1 with head my fork:task-branch and base Moxie:task-branch. Do not aim this PR at staging.

Do not claim that full CI passed unless it actually ran. Perform an independent full-diff code-review pass (not merely a summary), prioritize findings and suggest fixes. If I choose to use Greptile or another **approved** code-review agent, help me incorporate its findings; do not assume it is connected or required. With my authorization, publish PR #1 and show me exactly what I must inspect and the merge decision. **I self-review and manually merge PR #1; Raj's approval is not required until PR #2.** You must never submit approval or merge.
```

## D. Prepare PR #2 (Moxie integration branch -> staging)

```text
The human has merged PR #1 into Moxie's task branch: [PR #1 URL]. Read fork-to-release.md and pull-requests.md. Verify the exact Moxie organization integration branch and newest staging ref. Inspect the complete org-branch -> staging diff, current migration numbers and dependencies. Prepare PR #2 with Moxie task branch as head and Moxie staging as base; never use my fork as the head.

Run/check full CI and security on this staging-targeted PR. **Request Raj for code review and a submitted APPROVED GitHub review** on the latest code; my PR #1 self-review and agent reviews cannot substitute. Add domain reviewers when needed. If the branch or target moves, refresh checks and request re-approval. Do not merge PR #2 or modify staging directly.
```

## E. Triage a production bug (read-only)

```text
Read CLAUDE.md and bugfix-process.md. Investigate production issue [TICKET/DETAILS] without editing. Fetch the authoritative Moxie main baseline, find duplicates and prior fixes, and reproduce safely with main-compatible schema/data. Report reproduction, probable root cause, smallest fix, regression tests, deployment risk and suggested fix/<ticket>-<slug> (same name for my fork and Moxie integration branch). Explain normal staging release vs emergency main hotfix; do not choose emergency routing without human approval.
```

## F. Implement an approved bug fix

```text
Implement approved fix [TICKET/PLAN]. Work ONLY in an isolated branch/worktree in my personal fork from the verified Moxie main SHA (for every bug, including staging-only bugs; ask me if the affected code is not on main yet). Use the same fix/<ticket>-<slug> branch name for the Moxie integration branch. Add regression coverage; run focused tests, cd app && npm run check, and cd app && npm run build. Identify DB/auth/office risks. Inspect the full diff and provide a severity-ranked code review, optionally including an approved second AI review, for my own review.

Prepare PR #1 from fork fix branch -> matching Moxie fix branch. I will self-review and manually merge it after local checks; no separate human approval at this first stage. Normal PR #2 will be Moxie fix branch -> staging after Raj's approval and CI; an urgent direct main PR is possible only with explicit release-owner authorization. Ask before publishing if not already requested. Never merge/deploy.
```

## G. Build a staging-to-main release summary (read-only)

```text
Read fork-to-release.md, pull-requests.md and release-and-migrations.md. Prepare PR #3 from Moxie staging INTO Moxie main only after staging QA signoff. Do not merge/deploy. Enumerate ALL changes and merged PRs in staging not in main, not just the feature that triggered this request. Include Raj's staging approvals, actual staging deployment/test evidence, latest CI/security results, pending migrations/order, production config dependencies, known issues, rollback/roll-forward and production smoke tests.

Distinguish verified GitHub facts from Vercel settings you cannot inspect. Flag every blocker. Generate a review-ready release PR body for Raj/release owner and stop for human authorization. Remember that PR #3 into `main` needs an eligible GitHub approval from **someone other than the PR author**.
```
