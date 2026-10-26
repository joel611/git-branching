# Common principles

These apply to both strategies.

## 1. One worktree and one branch per ticket

Each agent works in its own folder, created from the latest base branch:

```bash
git worktree add -b ticket/101-login ../worktrees/101 origin/main
```

- Real filesystem isolation lets several agents run in parallel without overwriting each other.
- All worktrees share one repository and one history.
- Automate creation and cleanup.

## 2. Short-lived ticket branches

A branch exists to land one ticket. The longer it lives, the further it drifts from its base. Keep branches current while they are active (see principle 5).

## 3. Pull requests are the integration gate

CI and review run on the PR. Nothing reaches a shared branch without passing it.

## 4. Squash merge agent work

Agents commit in many small, exploratory steps. A squash merge turns them into one readable commit per ticket. The squash commit does not have the ticket's commits as parents, so the ticket branch is finished once the PR merges (see principle 6).

## 5. Keep a branch current by rebasing, never by merging the base in

```bash
git fetch origin
git rebase origin/<base>
# on conflict: fix files, then
git add . && git rebase --continue
git push --force-with-lease
```

- Rebase replays the ticket's commits on top of the current base, so the branch contains only the ticket's changes.
- Conflicts are resolved in the branch, while the ticket is fresh.
- `--force-with-lease` is needed because rebase rewrites commits. It is safe on a branch only one agent owns.

**The common mistake:** running `git merge origin/main` in the ticket branch.

- It adds a merge commit that carries main's code (and the conflict fix) into the branch history.
- The PR diff and history mix the base's code with the ticket's, which makes review harder.
- A squash merge does not record that the base code inside the branch is the same as the real commits on the base. The branch's own commits stay unrelated to the squash commit, so conflicts you already resolved can come back, and any follow-up branch built on it collides with the base again.

The final content of a squash can still come out the same. The harm is noisy review, repeated conflicts, and broken follow-up work, not guaranteed wrong code.

## 6. Critical rule: never commit to a branch after its PR has merged

After a squash merge, the branch's old commits are unrelated to the base. More commits on that branch make git treat the old work as new. The next PR re-includes code that already shipped and conflicts with the squash commit.

- Delete the branch and remove the worktree when the PR merges:

  ```bash
  git worktree remove ../worktrees/101
  git branch -D ticket/101-login
  git push origin --delete ticket/101-login
  ```

- For follow-up work, create a new branch and worktree from the latest base, for example `ticket/101-login-fix`.
- The same applies to a release branch after its PR merges (see the selective promotion strategy).

## 7. Naming

Use consistent names: `ticket/<id>-short-desc` or `agent/<id>-...`. Release branches are `release/YYYY-MM-DD`.

## 8. Avoid plain merges for integration

Plain merges create merge commits everywhere and entangle branches. See [baseline-merge-everywhere.md](baseline-merge-everywhere.md).
