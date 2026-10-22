# Feature development: personal fork to production

This is the **standard feature route**. Read [`fork-to-release.md`](fork-to-release.md) for the full three-PR sequence and example. Never skip directly from a personal fork to `staging` just because it seems faster.

## 1. Confirm behavior and scope

- Start from the feature's real Orbit ticket (ID, URL, owner, agreed scope); create/find it in Portal Feedback before any branch exists. See [README](README.md#every-new-task-starts-with-an-orbit-ticket).
- Identify the product outcome, users/roles/offices, edge cases and acceptance criteria. Inspect relevant code and existing `docs/plans/` or `docs/superpowers/` plans.
- For a small feature, record a short checklist. For substantial work, document the proposed design, tests and rollout in `docs/plans/YYYY-MM-DD-<slug>.md` and obtain human approval before implementation.
- Identify schema migrations, auth/role changes, integrations and new environment configuration in advance. Agree who owns external setup before a feature reaches staging.

## 2. Branch in your personal fork; reserve the same branch in Moxie

- Fetch the latest **Moxie `main`** as authoritative baseline (never `staging`), using verified remotes. `origin` may be the personal fork; do not infer its identity from the name.
- Create a dedicated **fork** branch/worktree, e.g. `feature/orbit-123-ticket-comments` (your real ticket number) based on that Moxie `main` SHA.
- Arrange for an authorized developer to create/reserve **Moxie org branch `feature/orbit-123-ticket-comments`** from the same `main` baseline. Check it is not owned by another task; never push development directly to this shared branch.
- Check both repositories and refs before making changes or opening PRs.

## 3. Implement and test locally

- Make small, reviewable commits that change only approved scope; add focused automated tests and meaningful error/permissions cases.
- Run targeted tests first, then (from the `app` directory):

```bash
npm run check
npm run build
```

`npm run check` currently covers typecheck, lint and Vitest. Use the README's safe environment setup if needed; never add production secrets to get a build working. Confirm manual cases across applicable roles, offices and errors. State skipped/failed tests honestly.
- Check `git diff` for generated files, credentials, schema changes, and unrelated modifications. **Perform a deliberate full-diff self-review**; optionally use an approved AI reviewer (e.g. Greptile or a separate review agent), fix its valid findings, and record your decisions. Follow `release-and-migrations.md` for any numbered SQL migration.

## 4. First PR: your fork -> Moxie feature branch

- With authorization, push the task branch to the **personal fork**.
- Open **PR #1** with head `<your personal fork>:feature/orbit-123-ticket-comments` and base `Moxie-Pest-Control-Phoenix/moxie-portal:feature/orbit-123-ticket-comments`, titled e.g. `feat(feedback): ORBIT-123 add ticket comments`.
- Register the pushed fork branch and PR #1 on the ticket (**Branches** / **Pull requests** fields, or the authorized API/MCP once available).
- Fill the template's ticket fields (primary and additional tickets, related branches/PRs, acceptance criteria/spec and HTML demo links, knowledge-base docs and requirement-approval evidence where applicable), approved plan, summary, changes, local test evidence, known risks, UI proof (if appropriate), migration notes and a short **author self-review** report (including optional AI review findings). **You are the first-stage reviewer. No Raj/second developer approval is needed for this PR.** Address issues on the fork branch.
- Once satisfied and GitHub allows it, **you manually merge PR #1**. Confirm the **Moxie integration branch** contains the expected commits. This merge does **not** deploy to staging. Coding agents never merge PRs.
- **Local checks matter here:** secret scanning on its own is not full CI.

## 5. Second PR: Moxie feature branch -> Moxie staging

- Open **PR #2** *from the organization's feature branch* to the organization's `staging`. Confirm target and diff; do not use the fork branch as head. Link PR #1 in its body and register PR #2 on the ticket.
- Required checks: the repository's lint, typecheck, tests, build, migration check and security scan, up to date with staging. Resolve conflicts safely and rerun checks. A previously green migration check can become stale as the base moves.
- **Request Raj as reviewer.** He must perform code review and submit an **APPROVED GitHub review** of the latest material changes before a human merges to staging. The first-stage author/AI review does not count. Consult specialists for DB/auth/multi-office impacts as needed.
- Update tests and PR notes after review changes; do not treat old approval as final if code has changed.

## 6. Validate deployed staging, then release

- Confirm the staging deployment commit and migration job status. Run and record end-to-end/manual QA for the agreed acceptance criteria, including permissions, bad inputs, and external dependencies.
- Any staging bug must be fixed through another reviewed branch/PR path, not an unreviewed direct staging push. Clearly mark release blockers.
- A release owner collects the finished staged work into **PR #3: Moxie `staging` -> `main`**, with a summary of all included features/fixes, staging test evidence, migrations, configuration, known issues and rollback plan. A human approves/merges and verifies production. Agents can draft the release summary but never deploy or merge it.

**Example:** ORBIT-123 -> `feature/orbit-123-ticket-comments` in fork -> `feature/orbit-123-ticket-comments` in Moxie -> `staging` after Raj's approval -> inclusion in a reviewed `staging` -> `main` release.
