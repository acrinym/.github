# acrinym GitHub Policy

This repository is the account-level source of truth for workflow policy shared across repositories owned by `acrinym`.

## CI/CD authority

For `acrinym` repositories, CI/CD is **local-first and runs in WSL**, not on GitHub-hosted Actions.

The canonical workflow is documented in [CI_CD_WORKFLOW.md](CI_CD_WORKFLOW.md) and mirrored in machine-readable form at [workflows/local-wsl-ci.contract.yml](workflows/local-wsl-ci.contract.yml).

When inspecting CI/CD for an `acrinym` repository, treat this account-level contract as authoritative unless that repository contains a more specific explicit override.

## PR review authority

PR review is a separate post-PR workflow:

- **Public repositories:** OpenHands and CodeRabbit may run through their normal GitHub integrations after the PR exists.
- **Private / non-public repositories:** after creating the PR, automatically add the comment `@openhands, please review PR.` to invoke OpenHands.

Review does not authorize merge. Merge remains a separate explicit action.

## CI/CD end condition

A local CI/CD run:

1. qualifies the exact repository head locally in WSL,
2. captures the qualification evidence,
3. cleans its run-scoped artifacts and temporary files,
4. verifies cleanup,
5. posts one final qualification + cleanup receipt into the matching GitHub pull request,
6. **ends**.

It does not automatically merge, deploy, or continue into another development stage.
