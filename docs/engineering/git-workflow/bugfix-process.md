# Bug-fix workflow: production baseline, fork-first integration

Use the same **personal fork -> matching Moxie `fix/*` branch** intake as features. The starting baseline differs depending on where the bug exists. Read [`fork-to-release.md`](fork-to-release.md) for PR stages and gates.

## 1. Triage before editing

1. Capture the ORBIT ticket (if available), environment, user role/office, impact, reproduction, expected and actual behavior, and logs/links with secrets redacted.
2. Check for duplicate tickets, existing PRs, already-merged fixes and *whether a fix is actually deployed*. Do not assume a closed ticket means production is fixed.
3. Reproduce production bugs against current authoritative Moxie **`main`** and a safe, production-like schema/data baseline. A bug only in staging is reproduced on staging, but the fix branch still starts from **`main`**; if the affected code is not on `main` yet, ask the human how to route the fix.
4. Explain cause, smallest fix, regression tests and deployment risk; obtain approval for nontrivial or sensitive changes.

## 2. Create matching branches in personal fork and org

- Start a new isolated worktree and `fix/<ticket-id>-<slug>` branch in your personal fork, based on the correct Moxie SHA. With no ticket, use `fix/<short-slug>`; do not invent an ID.
- Have an authorized writer reserve the **identically named branch in Moxie's repo** from that same baseline. Don't push development directly to the org integration branch.
- Use one task owner/writer; never change a second agent's worktree or silently include unreleased staging features in a production fix.

## 3. Write the regression test and verify the fix

- Prefer a regression test that fails before the fix. Make the smallest safe code change and cover key roles/offices, error cases and DB interactions.
- Run targeted tests, `cd app && npm run check`, and `cd app && npm run build` where supported. **Self-review the whole diff**, optionally with an approved AI code-review agent, resolve or document findings, and record the outcome. Include a safe local/preview reproduction and a clear rollback/roll-forward plan for sensitive changes.
- Stop for human approval if a fix changes migration schema/data, authentication, role/office boundaries, secrets, shared environments or external write behavior.

## 4. PR #1, followed by the integration PR

1. **PR #1** — personal fork `fix/*` -> **matching Moxie org `fix/*`**. Supply reproduction, evidence of local checks, regression coverage, deployment risks and a **documented self-review** (optionally AI-assisted). **The developer manually merges after their own review**, if GitHub allows; no independent human or Raj approval is required at this stage. Do not treat a security-only green badge as full CI.
2. **Normal PR #2** — Moxie org `fix/*` -> **Moxie `staging`**. Require fresh CI/security checks, **Raj's code review and submitted GitHub approval**, and a human merge. For production bugs, ensure the fix was validated against the **main baseline** even though it is being integrated into staging.
3. Verify the deployed staging fix and include it in a later **PR #3: `staging` -> `main`** release summary; check after production deployment and update the ticket with actual commit/deployment.

## Urgent production hotfix exception

A severe production incident may require shipping without waiting for unrelated unreleased staging changes. **Only a human incident/release owner** may approve this exception:

- Still start on Moxie `main`, implement in a **personal fork**, and merge through **PR #1 to the matching Moxie org fix branch**.
- After targeted validation in an isolated preview/main-like environment, open a reviewed **org `fix/*` -> `main`** PR (instead of `staging`) with explicit urgency, CI, safe migration ordering and rollback. A human approves and merges, then monitors production.
- **Synchronize the fix back to `staging` in a reviewed PR**; carefully avoid overwriting other staging work. Do not simply merge all of staging into a production hotfix branch.

Urgency removes optional scheduling, not testing/approval/production safety. A staging-only fix never uses the direct-to-main exception.
