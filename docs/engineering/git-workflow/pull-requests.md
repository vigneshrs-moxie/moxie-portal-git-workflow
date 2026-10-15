# Pull requests: three distinct stages and explicit owners

A pull request should show **what changed, why, what ran, and what could fail**. Read [`fork-to-release.md`](fork-to-release.md) before opening one. The destination is never guessed from the local checkout's default remote.

## PR #1: personal fork -> matching Moxie integration branch

| Field | Required value |
| --- | --- |
| Head | `<personal fork repo>:<task branch>` |
| Base | `Moxie-Pest-Control-Phoenix/moxie-portal:<exact same task branch>` |
| Review owner | **PR author (developer)**: full-diff self-review; optional AI code review (Greptile or another approved tool). **No second human reviewer or Raj approval required.** |
| Merge actor | The developer may **manually merge** once satisfied and permitted by GitHub; never the coding agent |
| Minimum evidence | Approved scope/ticket, local test/build results, regression tests, documented self-review and disposition of AI findings (if used), diff/security/DB notes |

Verify the organization feature/fix branch **exists**, is based on the agreed baseline, and is not someone else's work. Matching **branch names** do not mean matching repositories. **Self-review means you examine the PR and document your decision; GitHub does not let a PR author satisfy a required approving review on their own PR.** Do not set a mandatory independent approval rule on the org feature branch unless the team deliberately changes this stage.

**Local checks:** run local checks for PR #1 and label incomplete validation plainly; a secret scan alone is not full CI. Resolve important security/code-review findings before the developer merges PR #1; document accepted exceptions. AI tools are optional and must be approved for access to this private repository.

## PR #2: Moxie integration branch -> Moxie staging

| Field | Required value |
| --- | --- |
| Head | `Moxie-Pest-Control-Phoenix/moxie-portal:<task branch>` |
| Base | `Moxie-Pest-Control-Phoenix/moxie-portal:staging` |
| Required reviewer | **Raj** must code-review and submit an approving review on the latest material changes; domain reviewers additionally where needed |
| Merge actor | Authorized human after Raj approval and green, current checks |
| Minimum evidence | First-stage PR link, test results, change/DB/security summary, smoke-test plan, screenshots if relevant |

Ensure both integration and staging branch heads are current. If conflicts or migration numbers change, resolve safely and re-run checks **against the new base**. If code changes after Raj's approval, request another review. Resolve outstanding review conversations before merge. **The developer's PR #1 self-review (and any AI review) does not count as Raj's GitHub approval.** A green CI badge from yesterday is not proof that migrations are collision-free now.

## PR #3: Moxie staging -> Moxie main

| Field | Required value |
| --- | --- |
| Head / base | `Moxie:staging` -> `Moxie:main` |
| Reviewer | An eligible reviewer **other than the PR author**; Raj/release owner also authorizes production go/no-go |
| Type | Release summary encompassing **every** change in the staging-vs-main diff |
| Minimum evidence | Included PRs/tickets, staging deployment and QA signoff, current CI/security, migration/configuration plan, known issues, owner, rollback and production test plan |

Do not sneak new code into the release PR. `staging` may include other people's work: use a fresh compare to enumerate it, and don't merge until every included change is ready. When staging advances after release review, refresh the summary and get approval for the expanded diff. Prefer a **merge commit** for staging -> main to preserve ancestry; do not squash away staging ancestry without an explicitly agreed release strategy.

A main merge can trigger real deployments and automated migrations. The author cannot provide their own required GitHub approval on PR #3; with this two-developer team, the non-author can review (subject to repo permissions) and Raj/release owner coordinates the production decision. The owner verifies the release configuration before clicking merge.

## PR titles, review and merge style

- Title: readable and specific, e.g. `feat(scorecards): copy role templates`, `fix(ratings): prevent duplicate submission`, or `release: staging to main (2026-10-12)`.
- Body: use the [PR template](../../../.github/PULL_REQUEST_TEMPLATE.md), or equivalent clear sections: scope, ticket/spec, exact head/base repos, checks, risks, DB migration order, screenshots, human reviewer.
- Regular single-feature/fix integration PRs may be **squashed** if the team wants concise histories; choose consistently. Keep important mirror/upstream history where required by its owner.
- Avoid mixing multiple independent tasks in a feature branch. Stacked agent PRs need explicit approval and clear diff/CI handling, not an improvised chain.
- When an agent is authorized to open PRs, it may prepare/update the body, but it must **not submit an approval, merge, bypass protection, dismiss human objections, or deploy**.

## Checklist that applies to every stage

1. Confirm **head repository/ref AND base repository/ref**, plus the task owner and corresponding ticket.
2. Inspect `git status`, commits and final diff for accidental files, secrets and unrelated scope.
3. Record *actual* focused tests, `cd app && npm run check`, `cd app && npm run build`, and manual tests; name anything skipped/blocked.
4. Check auth/roles/office scope, API integration writes, user-visible errors, and four-digit DB migration ordering as applicable.
5. Apply the **correct gate**: PR #1 = developer-documented self-review plus local checks (AI review optional); PR #2 = **Raj's recorded GitHub approval** plus fresh CI/security; PR #3 = independent GitHub approval required by `main` plus release-owner go/no-go and staging QA. Never treat a CI pass as a substitute for deployed staging tests or product acceptance.
6. After merge, verify the intended target ref and only then open the next PR. No agent automatically starts the next merge.

## Production hotfix exception

For a human-approved urgent production incident, PR #1 remains fork -> matching org `fix/*`, but PR #2 goes from org `fix/*` -> **main** after production-baseline reproduction, focused tests and authorized review. Follow with a **reviewed `main` -> `staging` sync**. See [`bugfix-process.md`](bugfix-process.md); an agent cannot decide to bypass staging itself.
