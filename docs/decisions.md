# Decision log

Each entry gives the decision, the reason, and the alternative rejected.

| # | Decision | Reason | Rejected alternative |
|---|----------|--------|----------------------|
| 1 | One worktree and one branch per ticket | Filesystem isolation lets agents run in parallel | Several agents sharing one checkout |
| 2 | Squash merge agent-generated work | Hides exploratory commits, one readable commit per ticket | Merging every agent commit |
| 3 | Offer two strategies and choose by project constraints | Internal products and client contracts need different controls | One model for everything; classic Git Flow, which has too much overhead for parallel agents |
| 4 | Trunk-based: `main` is the only long-lived branch and stays releasable | Maximum speed, minimal ceremony | Long-lived `develop` or `uat` for internal products |
| 5 | Incomplete work in trunk-based is hidden behind feature flags | Keeps branches short and `main` releasable | Long-lived feature branches |
| 6 | Keep branches current by rebasing onto the base and push with `--force-with-lease` | Branch holds only the ticket's changes, conflicts are settled in the branch before the squash, and the squash applies cleanly | Merging the base into the feature branch |
| 7 | Treat "merge main into the feature branch, then squash" as a known common mistake | It mixes base code into branch history, adds noisy diffs, and can bring conflicts back after the squash. The final content can still come out the same, so the harm is review noise and broken follow-ups | None. Documented as a warning |
| 8 | Never commit to a branch after its PR has merged; delete it and branch again from the base | The squash commit is unrelated to the branch's commits, so new commits re-include shipped work and conflict | Reusing the merged branch for follow-ups |
| 9 | Selective promotion: tickets merge into `uat` with plain merges | `uat` is disposable and never lands on `main` | Squashing or rebasing into `uat` |
| 10 | Cut `release/*` from `main`, not from `uat` | Keeps rejected tickets out of the release | Merging `uat` into `main`, or deleting rejected tickets from `uat` |
| 11 | Ticket conflicts are not fixed in ticket branches before release | With two features on the same line, each fix would need the other ticket's code, and merging one into the other ties them together | Merging tickets into each other early |
| 12 | Ticket conflicts are not fixed in `uat` | `uat` is disposable and not a release | Treating the `uat` merge as the resolution |
| 13 | Ticket conflicts are not fixed directly on `main` | Hand fixes risk human mistakes and are untested compared with `uat` | Separate PRs from each ticket straight to `main` |
| 14 | Ticket conflicts are resolved in the PR into the release branch, by rebasing the ticket onto `release/*` | Both tickets are confirmed to ship at that point, CI tests the combination, and `main` is untouched | Merging `release/*` into the ticket |
| 15 | Each approved ticket gets its own PR into the release branch, squash merged | Satisfies "each feature has its own PR" with CI, review and an audit trail per feature | Local `git merge --squash` by the PIC with no ticket PR. Valid, but less traceable. The cost of the chosen way is that conflict fixes are owned by each ticket's author and order matters |
| 16 | One final PR from `release/*` to `main` | Client requirement that the final PR lands on `main` | Several PRs to `main` |
| 17 | The final PR uses Rebase and merge, never squash | Appends each ticket commit as it is, giving N commits and no merge commit | Squash (collapses all tickets into one) or a merge commit (adds noise) |
| 18 | Never reuse a release branch after its final PR merges | Rebase and merge creates new hashes, so the release branch no longer matches `main` | Reusing the branch for the next release |
| 19 | Run CI and a quick UAT check on the release branch before the final PR | The release branch is the first place the combined, resolved code exists | Testing only on `uat` |
| 20 | Rebase the release branch onto `main` first if `main` moved during the release | Avoids a surprise conflict at the final merge | Resolving it in the final PR |
| 21 | Reset `uat` to `main` after each release | Stops rejected tickets from accumulating | Leaving `uat` as it is |
| 22 | Restrict merge methods per branch with rulesets where the host supports it | Turns the merge-method rule into something enforced, not remembered | Relying on memory |
| 23 | Use `ticket/<id>-short-desc` and `release/YYYY-MM-DD` naming | Consistency, easy automation | Free-form names |
| 24 | The `uat` / `production` plain-merge flow is the baseline we move away from | Merge commits everywhere, two-way syncing, untested conflict fixes, rejected code stuck in `uat` | Keeping it |

## Open items

These are not yet decided:

- How and when the team moves from a branch called `production` to `main`, and how CI triggers change.
- Whether the host's rulesets can enforce the merge methods in decision 22. Confirm in your Git host's current settings.
- Whether ticket → `release/*` PRs stay the default, or the PIC uses local `git merge --squash` for small releases.
