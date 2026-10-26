# Baseline: merge everywhere (what we are moving away from)

Many teams, including ours today, run CI/CD from two environment branches, `uat` and `production`, and use plain merges for everything. This file records why that flow was rejected.

## The flow

1. `production` deploys to production. `uat` deploys to the UAT environment.
2. Tickets branch from `production`, with small agent commits.
3. Every ticket is merged into `uat` for testing.
4. A hotfix lands on `production`. Every open ticket branch, and `uat`, must merge `production` in to stay current. Each is a merge commit with no work of its own.
5. The client approves some tickets and rejects others. Merging `uat` into `production` would ship the rejected tickets too, because they are already inside `uat`. So each approved ticket is merged straight into `production`. Conflicts between tickets are fixed there by hand, in a merge that was never tested on `uat`.
6. `production` changed, so `uat` must merge it back. `uat` still contains the rejected ticket.

## What it costs

In the worked example (3 tickets, 1 hotfix, 2 approved):

- `production` holds 9 commits for 2 shipped tickets. Only 5 are real work (first commit, hotfix, three ticket commits). 4 are merge commits.
- `uat` carries 5 more merge commits of its own.

Recurring problems:

- Reverting one ticket means reverting a merge and untangling the production code merged into it.
- PR diffs mix the base branch's code with the ticket's.
- Reading the log or bisecting means walking through interleaved branches.
- Both environment branches must be synced in both directions after every release.
- Rejected code stays in `uat`, so `uat` no longer matches `production` plus what is pending.
- Choosing only some tickets to ship is hard, because their branches already contain other changes.
- Conflict fixes reach `production` untested.

## The replacement

- Trunk-based development: 1 commit per ticket and 0 merge commits on `main`. See [strategy-trunk-based.md](strategy-trunk-based.md).
- Selective promotion: same result, with `uat` reset after each release. See [strategy-selective-promotion.md](strategy-selective-promotion.md).

In both, `main` is the production branch. `uat` stays a disposable test branch where plain merges are acceptable, because it never lands anywhere.
