# Git Branching for Agentic Development

Branching rules for teams that run many AI coding agents in parallel, each in its own git worktree, with every change landing through a pull request.

An interactive lesson that walks through all of this lives in [`../index.html`](../index.html) (published through GitHub Pages).

## Contents

| File | What it covers |
|------|----------------|
| [principles.md](principles.md) | Rules shared by every strategy |
| [strategy-trunk-based.md](strategy-trunk-based.md) | Trunk-based development, for internal products and continuous deployment |
| [strategy-selective-promotion.md](strategy-selective-promotion.md) | Selective promotion, for client work with UAT and partial releases |
| [baseline-merge-everywhere.md](baseline-merge-everywhere.md) | The `uat` / `production` plain-merge flow we are moving away from |
| [decisions.md](decisions.md) | Every decision made, with the reason and the alternative rejected |

## Choosing a strategy

Choose by project constraints, not preference.

- Can the client reject individual tickets after testing them on UAT? Use **selective promotion**.
- Otherwise, if you own the product and can deploy often, use **trunk-based**.

Classic Git Flow (long-lived `develop`, release and hotfix branches) is not used. It adds too much overhead and merge pain when many agents produce changes at once.
