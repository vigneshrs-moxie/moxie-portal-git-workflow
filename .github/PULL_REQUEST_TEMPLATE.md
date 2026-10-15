## Purpose and ticket

What changed, why, and which ORBIT ticket/spec covers it?

## PR stage (choose one)

- [ ] **#1 Fork intake:** personal fork `<branch>` -> Moxie org **same-named** task branch (**developer self-review + manual merge; AI review optional**)
- [ ] **#2 Staging:** Moxie org task branch -> Moxie org `staging` (**Raj review required**)
- [ ] **#3 Release:** Moxie org `staging` -> Moxie org `main` (list **all** included changes below)
- [ ] **Approved urgent production hotfix:** Moxie org fix branch -> `main`, with reviewed `main` -> `staging` sync
- [ ] Other (CadreAI mirror / Dependabot / legacy PR): explain

**Head repository + ref:**
**Base repository + ref:**
**Earlier-stage PR(s) / reason for exception:**

## Changes / implementation

- What this PR changes:
- Any non-obvious design choices:
- Screenshots/video (for UI changes):

## Test evidence (only check if actually passed)

- [ ] Focused automated/regression tests (commands/results below)
- [ ] `cd app && npm run check`
- [ ] `cd app && npm run build`
- [ ] Applicable GitHub CI + security jobs passed on current base
- [ ] Deployed staging smoke test / acceptance scenarios (for staging signoff and release)
- [ ] Checks skipped/blocked (explain)

**Commands, results and tested environments:**

> Local test evidence is required for PR #1; secret scanning alone is not full CI.

## Risk / rollout / rollback

- [ ] No DB schema/data change
- [ ] DB changes listed below, with migration IDs and **pre-/post-deploy order**
- [ ] Auth/role/entity/office visibility affected
- [ ] OAuth, secrets, external integrations, flags or deployment config affected

**Dependencies and config changes:**
**Known limitations / release blockers:**
**Rollback or roll-forward plan:**

## Review and approval gates

- **PR #1 (fork -> org):** **I reviewed my own full diff**, checked test/build results and addressed or documented findings. An AI code reviewer (e.g. Greptile) is **optional**, not a formal GitHub approval. **I will manually merge** once satisfied; Raj/another developer is not required for this stage.
- **PR #2 (org -> staging):** **Raj must review and submit GitHub APPROVED** on the latest material changes, after current CI/security checks pass. PR #1 self-review or AI comments do not meet this gate.
- **PR #3 (staging -> main):** release owner confirms deployed staging testing, release contents and risks; current `main` ruleset requires **one eligible GitHub approval from someone other than the PR author** before a human merges.

**PR #1 developer self-review (what I checked / outcome):**

**Optional code-review agent + findings (used or not used; fixed/accepted):**

**Raj's staging approval / staging QA evidence (when applicable):**

## Release-only summary (PR #3)

- All included feature/fix PRs/tickets and user-visible changes:
- Staging deployment commit, QA testers/results and blockers:
- Production migration/config order, owner and rollback:
- Production verification checklist:
