# User-Level Local WSL CI/CD Workflow

**Scope:** repositories owned by `acrinym`  
**Authority:** account-level GitHub workflow policy  
**Execution location:** local machine, WSL2  
**Primary distro:** `Acrinym-Dev`

## Purpose

This is the canonical CI/CD workflow for `acrinym` repositories.

GitHub is the source of truth for the workflow contract and the PR receipt. The actual CI/CD execution happens locally in WSL.

GitHub-hosted Actions are not the authoritative CI/CD runner for this workflow.

## Local CI/CD workflow

### 1. Resolve the exact GitHub target

Determine the repository, branch, exact HEAD SHA, and matching open pull request when one exists.

The SHA qualified locally must be the SHA reported in the PR receipt.

### 2. Stage a disposable Linux-native copy

Run CI/CD in `Acrinym-Dev`.

Do not qualify directly against the Windows-mounted working tree when WSL Git sees line-ending or filesystem noise.

Prefer a disposable Linux-native clone or equivalent isolated staging directory at the exact target SHA.

Do not create a Git worktree for this purpose.

### 3. Provision the repository environment

Provision dependencies declared by the repository into run-scoped local CI state.

For Python repositories this normally means a disposable virtual environment using the repository's own dependency manifests.

Do not treat missing globally installed dependencies as a product failure when they are already declared by the repository.

### 4. Execute qualification

Run the repository's applicable local qualification surface.

For HTE-enabled repositories, use HTE-Code Depth 5 with execution enabled.

GUI tests that require a display must run under an appropriate local virtual display such as Xvfb. Do not silently skip GUI tests merely because CI is headless.

Capture the exact HEAD before and after, dirty-state stability, executed commands, pass/fail status, required failures, optional failures, environment blockers, and qualification level/decision when provided.

### 5. Post one PR receipt

When the target has a matching pull request, CI/CD posts a GitHub PR comment containing the final qualification result.

The receipt must identify the exact qualified SHA, summarize the material command results, and include a cleanup receipt.

Suggested shape:

```markdown
Local WSL CI/CD qualification receipt for `<sha>`

- Environment: `Acrinym-Dev` WSL2
- Qualification: <level / decision>
- <command>: PASS/FAIL
- required failures: <count>
- optional failures: <count>
- environment blockers: <count>
- HEAD stable: true/false
- dirty-state stable: true/false

Cleanup receipt:
- disposable staging removed: yes/no
- run-scoped dependency environment removed: yes/no
- qualification output/artifacts removed: yes/no
- run-specific temporary files removed: yes/no
- source checkout clean after cleanup: yes/no
```

Do not represent GitHub Actions, CodeRabbit, Devin, Copilot, or another checker as this local CI/CD result.

### 6. Clean run-scoped artifacts

After final qualification evidence has been captured and the PR receipt has been posted, remove run-scoped material created by CI/CD, including as applicable:

- disposable clones/staging directories
- run-scoped virtual environments
- HTE output directories
- build/test output created only for the run
- `.pytest_cache`
- `__pycache__`
- `*.pyc`
- coverage/test caches
- run-specific temporary scripts and logs

Do not delete reusable user-level toolchains or intentional shared caches unless the workflow explicitly created them for the run.

Verify cleanup after deletion.

### 7. Stop

**The local CI/CD workflow ends here.**

CI/CD does not automatically merge, deploy, begin another development stage, or continue into a broader orchestration loop.

PR review is a separate post-PR workflow.

## Post-PR review workflow

This begins only after a pull request exists.

### Public repositories

OpenHands and CodeRabbit may run through their normal GitHub integrations.

Do not substitute one reviewer for the other when a specific reviewer is required.

### Private / non-public repositories

Immediately after the PR is created, automatically add this PR comment:

```
@openhands, please review PR.
```

This comment is the standard OpenHands review trigger for non-public repositories.

### Review does not imply merge

OpenHands or CodeRabbit review is advisory/review workflow only.

Do not merge merely because review is green. Merge requires the user's explicit merge authorization.

## Failure handling

A qualification failure must distinguish between product/code failure, missing but declared environment provisioning, headless GUI/display requirement, CI infrastructure failure, and external GitHub/account failure.

Fix real encountered product failures before proceeding.

Environment/infrastructure failures must not be mislabeled as product failures.

## Precedence

A repository may contain an explicit repo-specific override when its architecture requires one. Otherwise this account-level workflow is the default CI/CD and post-PR review contract for repositories owned by `acrinym`.
