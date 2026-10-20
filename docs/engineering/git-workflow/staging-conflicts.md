# When PR #2 conflicts with staging

**Use this when your PR into `staging` (PR #2) shows "This branch has conflicts".** It means someone else's work reached `staging` after your branch was cut, and you both edited the same lines. The fix is to bring `staging` into your branch, resolve the conflicts on your fork, and send the fix through a small PR #1 again. The staging PR then updates itself.

Worked example: collection rate, 2026-10-20. PR #2 = #129 conflicted with the damage list (#128). The fix went through sync PR #130, and the merge commit was `537035e1`.

```text
staging ──●───────●(#128 damage list)──────────────┐
           \                                       ↓ 2. merge staging in
fork branch ●──●──●(your work)─────────────────────●(merge commit) ── 6. push to fork
                   \ #108 (PR #1)                   \ 7. sync PR (fork -> org)
org branch ─────────●───────────────────────────────●── #129 (PR #2) is now mergeable
```

Remote names: these docs say `upstream` for the Moxie org repo. Some clones call it `moxie`. Check with `git remote -v` and use whichever name points to `Moxie-Pest-Control-Phoenix/moxie-portal`. `origin` is your fork.

## Before you start

- PR #1 for the branch is **merged**, and PR #2 is open (draft is fine) and shows conflicts.
- The worktree is clean: `git status --short` prints nothing.
- See which files conflict without touching anything:
  ```bash
  git fetch upstream
  git merge-tree --write-tree --name-only upstream/staging upstream/<branch>
  ```
  The lines after the first one are the conflicting files. `git log --oneline <merge-base>..upstream/staging -- <file>` shows which staging PR touched them.

## Steps

| # | Step | Command | Why |
| --- | --- | --- | --- |
| 1 | Get up to date | `git fetch upstream && git fetch origin`<br>`git merge --ff-only upstream/<branch>` | Merging PR #1 created a merge commit on the org branch. Fast-forwarding to it keeps the fork and org history identical. `--ff-only` refuses rather than creating a new commit |
| 2 | Merge staging in | `git merge --no-ff upstream/staging` | Git applies what it can and stops on the conflicts. Nothing is committed yet. **`git merge --abort` undoes everything** if it looks wrong |
| 3 | Fix each conflict | Edit the files listed as `UU` (both changed) or `AA` (both created) in `git status` | See "Resolving a conflict" below |
| 4 | Check what Git merged on its own | `npx prisma generate` (only if `schema.prisma` came in)<br>`npm run typecheck`<br>focused tests for the area | Files without markers can still be broken. See "Things that bite" |
| 5 | Commit | `git add <fixed files>`<br>`git commit` | In the message, list each conflict and how you fixed it |
| — | Scope check | `git diff --stat upstream/staging HEAD` | Must list **only your feature's files**. If anything else shows up, stop and find out why |
| 6 | Full check, then push | `cd app && npm run check`<br>`git fetch upstream` (anything new on staging? repeat from 2)<br>`git push origin <branch>` | Normal push. **Never force-push** |
| 7 | Sync PR (PR #1 again) | Fork `<branch>` → org `<branch>` | See "The sync PR" below. You review and merge it yourself, as with any PR #1 |
| 8 | Back to PR #2 | Check that PR #2 says it can merge, and its **Files changed** count matches your feature. Remove any conflict warning from its description. Mark it ready for Raj | Raj reviews PR #2 as usual |

## Resolving a conflict

Git marks each conflict like this:

```text
<<<<<<< HEAD
your branch's lines
=======
staging's lines
>>>>>>> upstream/staging
```

Edit the block until it reads the way the **combined** code should, then delete all three marker lines. Usually you keep both sides' work:

- **Both added imports in the same spot:** keep both lists.
- **Both created the same new file (`AA`):** there's no shared original. Look at each side's whole version (`git show :2:./path` = yours, `git show :3:./path` = staging's) and write one file that keeps every test or function from both.
- **Both changed the same logic:** understand what each side meant first. If it isn't obvious, ask the author of the staging change; `git log upstream/staging -- <file>` shows who.

Before committing, check that no markers are left: `grep -nE '^(<<<<<<<|=======|>>>>>>>)' <file>`.

## Things that bite

- **Tests that count things.** If your feature adds a tile, column or button, staging's tests may count the old number. Update the count and add a comment saying why. (#130: help-cursor tiles went 7→8 and 8→9 for the new Collection tile.)
- **Shared sample data.** Staging's tests may build objects without your new fields, so typecheck fails. Add the field.
- **Cache key versions.** If both sides bumped a cache key version (for example `field-dashboard:v3` → `v4`), they now mean two different payloads under one name. Bump past both. This happened in a September 2026 merge.
- **Typecheck errors in code you never touched, right after the merge.** Staging probably changed `schema.prisma`. Run `npx prisma generate` (it only rebuilds generated code, no database) and typecheck again.
- **Staging migrations and your local database.** A migration from staging (e.g. `0137`) isn't applied to your local database, so pages that use it may error on localhost. Applying it is a separate decision.
- **The pre-commit hook runs `eslint --fix` on every staged file, including staging's.** The scope check after committing (`git diff --stat upstream/staging HEAD`) proves it didn't change them.
- **zsh and `git show`:** write `"${tree}:path"`, not `$tree:path`. zsh reads `:a`, `:h` and similar as path modifiers.

## The sync PR

Its diff looks alarming: **every staging change your branch hadn't seen yet** shows up (66 files in #130). That's expected. It's all already on `staging` and reviewed there. Say so at the top of the description, and point reviewers to the one commit that matters:

- Purpose: "merge conflict fix for PR #2 #NNN, no new feature work".
- Why the diff is large, and that PR #2 goes back to only your files once this merges.
- The hand-written changes: each conflict and how it was fixed. Reviewers can run `git show <merge commit>` instead of reading the whole diff.
- Test evidence: focused tests, `npm run check`, and the scope check result.
- Rollback: revert the merge commit on the org branch.

## Other option: GitHub's web editor

The **Resolve conflicts** button on PR #2 works for a two-line import clash. But it commits straight to the org branch, skips your local checks, and can't handle cases where both sides created the file. Prefer the steps above.
