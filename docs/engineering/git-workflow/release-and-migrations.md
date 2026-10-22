# Staging, production and migrations

**A merge into `main` can affect production.** Do not treat staging-to-main as a harmless Git cleanup. The actual Vercel and GitHub deployment settings must be confirmed by the release owner before enabling autonomous PR creation or any new release process.

## Normal route and exception

```text
Normal feature:  fork feature/orbit-<n>-xyz -> org feature/orbit-<n>-xyz -> org staging -> org main
                 PR #1: developer self-review + local checks (optional AI reviewer)
                 PR #2: full CI + Raj's submitted GitHub approval
                 PR #3: staging QA + release summary + non-author GitHub approval

Nonurgent bug:   fork fix/orbit-<n>-xyz -> org fix/orbit-<n>-xyz -> org staging -> org main
                 (bug reproduced from authoritative main, also validate against main baseline)

Urgent prod bug: fork fix/orbit-<n>-xyz -> org fix/orbit-<n>-xyz -> org main -> reviewed sync into staging
                 (incident ticket recorded first; only with human incident/release-owner authorization)
```

**Release PR** `Moxie staging -> Moxie main` should summarize *all staged changes* and every included Orbit ticket, not one feature. Prefer preserving staging ancestry with a merge commit, rather than squashing the entire staging history. No fresh feature implementation belongs directly in the release PR.

## Deployment configuration

The release owner confirms how deployments and migration jobs are triggered before each release. Agents do not change deployment or CI configuration as part of adopting this documentation.

## Migration safeguards

1. **No naming change yet.** Current migration workflows and `.github/ci/check-migration-numbers.sh` still assume four-digit sequential directory prefixes. The timestamp-naming redesign is a different project requiring explicit approval and coordinated tooling changes.
2. **Numbers can collide after a green PR.** The numbering check compares against a PR's base at the time it runs. As `staging` or `main` advances, another merged PR may take the same number. Before merging PR #2 and before release, re-compare with the current target and refresh validation. A previously green check is not proof after new merges.
3. **Read-before-create risk:** if migrations are applied **after** the corresponding deploy becomes available, App code that expects a new column immediately can fail during that window. Use backward-compatible expand/contract changes or have a human approve a safe pre-deployment migration runbook.
4. **Rollback:** reverting a code commit does not automatically revert a database schema/data change. For destructive changes, document recovery, backups, feature flags, dual-read/write compatibility, and roll-forward as needed.
5. **Agents:** may inspect SQL and run safe isolated local DB tests; may not apply migrations to staging or production, copy identifiable production data, or access production credentials without approved procedures.

## Human staging QA checklist

- [ ] Confirm PR #2 merged with Raj's submitted review; identify actual staging deployment SHA.
- [ ] Confirm staging CI, security and auto-migration job outcome; check logs, not only a PR badge.
- [ ] Run explicit functional tests/acceptance criteria on **deployed staging**, including role/office and negative cases.
- [ ] Check external OAuth/integration permissions, required secrets, storage and feature flags in staging.
- [ ] Record tester, results and known defects in PR/ticket; mark release blockers.

## Human release PR (`staging` -> `main`) checklist

- [ ] Compare current **Moxie `main` vs `staging`** and list *every* included org->staging PR/ticket (the latest feature may not be the only change).
- [ ] Ensure all included changes have staging QA signoff and meet product acceptance; re-review if staging changed since approval.
- [ ] Ensure current CI/security/migration checks pass against latest main, and reconcile branch movement safely.
- [ ] List SQL migration folders, pre/post-deploy ordering, non-DB configuration changes, fallbacks, and any production-only setup.
- [ ] Confirm verified deployment mechanism, release approver, rollback/roll-forward and production smoke-test owner.
- [ ] Obtain an eligible **GitHub approval from someone other than the PR author**, plus the release owner's go/no-go; authorized human merges the PR and observes production deployment/migration jobs.
- [ ] Smoke test affected production flows; attach commit/deployment reference to tickets and incident notes.

## Changes intentionally excluded from this docs proposal

No CI workflow rewrite, merge queue, GitHub permission change, Vercel config change, timestamp migration rollout or shared DB operation is applied just by reviewing this document. Approve and implement those as separate tasks.
