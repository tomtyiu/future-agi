# Pull Request Review, Fix, Push, and Verify

Review and fix the **current GitHub Pull Request** using the actual repository as the source of truth.

The complete operation is:

```text
Inspect PR
→ Review changed application code
→ Identify confirmed P0/P1 defects
→ Apply minimal fix
→ Run affected tests
→ Push the fix to the current PR branch
→ Wait for GitHub PR checks to complete
→ Verify the final PR state
→ Report results
→ STOP
```

Do not start a second review cycle after pushing the fix.

---

## 1. Scope

Inspect only the application/project code, dependencies, configuration, and tests relevant to the current PR.

Completely ignore:

```text
.github/**
.github/workflows/**
.github/actions/**
```

Do not inspect, modify, execute, or report findings involving:

* GitHub Actions
* CI/CD configuration
* workflow files
* workflow triggers
* workflow permissions
* action versions
* workflow security

`.github/**` is permanently out of scope.

---

## 2. Dependency and Lockfile Protection

Treat these files as **read-only unless the PR intentionally changes dependencies and the change is required to fix a confirmed P0/P1 defect**:

```text
package.json
package-lock.json
```

Never run commands that intentionally update or regenerate dependencies:

```bash
npm install
npm update
npm audit fix
npm dedupe
npm shrinkwrap
```

Do not modify `package.json` or `package-lock.json` merely to make tests pass.

Do not regenerate `package-lock.json`.

For dependency security auditing, use a read-only command such as:

```bash
npm audit --package-lock-only
```

Inspect package scripts before running them.

Do not blindly execute lifecycle scripts such as:

```text
preinstall
install
postinstall
prepare
```

Only execute repository test, lint, type-check, or build commands that are actually required for PR validation.

---

## 3. Review

Review only changes introduced by the current PR.

Report only **confirmed P0/P1 issues**.

### Correctness

Check for:

* logic errors
* incorrect state handling
* edge cases
* regressions
* broken API behavior
* incorrect database behavior
* serious error-handling defects

### Security

Check for:

* SQL injection
* XSS
* command injection
* path traversal
* authentication bypass
* authorization bypass
* improper access control
* secret exposure
* unsafe handling of untrusted input
* confirmed dependency vulnerabilities

### Performance

Check for:

* N+1 queries
* excessive requests
* significant unnecessary computation
* serious resource/concurrency issues
* clearly required missing indexes

### Maintainability

Only report issues that create a material correctness, reliability, security, or operational risk.

Do not report:

* style
* formatting
* naming preferences
* documentation
* speculative issues
* minor refactoring opportunities

---

## 4. Tests

Discover the actual project configuration before running commands.

Determine:

* package manager
* available package scripts
* test framework
* lint configuration
* type-check configuration
* build configuration

Run the smallest relevant tests first.

Run the full test suite when practical.

Run applicable:

* Vitest tests
* Playwright tests
* lint
* type-check
* build
* dependency/security checks

Do not assume scripts or paths. Use the actual repository configuration.

---

## 5. Fixes

Fix only:

* confirmed P0/P1 defects introduced by the PR
* test failures directly caused by the PR

Make the smallest possible production-code change.

Do not modify tests just to make them pass.

Do not refactor unrelated code.

Do not modify `.github/**`.

Do not modify dependency files unless absolutely required for a confirmed P0/P1 fix.

---

## 6. Validate Before Push

After applying a fix:

1. Run the directly affected tests.
2. Run relevant lint/type-check/build checks.
3. Run broader PR validation when practical.
4. Check the working tree.
5. Inspect the final diff.
6. Confirm only intended files changed.

Run:

```bash
git status --short
git diff --check
git diff -- package.json package-lock.json
```

If `package.json` or `package-lock.json` changed unexpectedly:

* revert the unintended dependency-file changes
* do not regenerate the lockfile
* continue only with intended application changes

---

## 7. Push the Fix to the PR Branch

If a confirmed P0/P1 defect was fixed, push the fix to the **existing PR head branch**.

Determine the PR head repository, branch, and SHA from GitHub rather than assuming the branch name.

Before pushing:

```bash
git status --short
git diff --check
```

Create a concise commit containing only the fix.

Use the repository's existing Git identity where available.

Push only to the PR's existing head branch.

Do not push to the default/base branch.

Do not create a new branch unless the repository specifically requires it.

Prefer the workflow's `GITHUB_TOKEN` for the push so that the push does not recursively create another workflow run.

Do not use `workflow_dispatch`, `repository_dispatch`, or a PAT/GitHub App token to intentionally trigger another reviewer workflow.

GitHub documents that events generated using `GITHUB_TOKEN` generally do not create a new workflow run, which prevents recursive workflow execution.

---

## 8. Wait for GitHub PR Checks

After successfully pushing the fix:

1. Detect the new PR head SHA.
2. Wait for GitHub to register checks for that SHA.
3. Monitor the PR's checks/statuses.
4. Wait until the applicable checks reach a terminal state.
5. Do not start another code review.
6. Do not make another fix solely because a check is still running.

The agent must distinguish between:

```text
queued
in_progress
completed
```

Do not treat a queued or running check as a failure.

Wait for the checks associated with the **new pushed commit**, not the previous commit.

When checks complete:

* determine whether required checks passed
* record failed checks
* do not automatically modify code because an unrelated check failed
* do not retry indefinitely

If a check fails because of the fix and the failure is clearly attributable to the changed application code:

1. Diagnose the failure.
2. Make the smallest necessary fix.
3. Re-run affected local validation.
4. Push one additional corrective commit.
5. Wait for checks on the new SHA.
6. Stop after successful verification or after a clearly non-resolvable failure.

Do not enter an unbounded fix/recheck loop.

Maximum automated fix iterations: **2**.

After the second fix attempt, stop and report the remaining failure.

---

## 9. Loop Prevention

This is one PR operation.

The agent must never recursively restart itself because its own commit was pushed.

The workflow must follow:

```text
PR event
→ Review
→ Fix
→ Local validation
→ Push
→ Wait for PR checks
→ Verify
→ Report
→ STOP
```

Never:

```text
Review
→ Push
→ synchronize
→ Review again
→ Push
→ synchronize
→ ...
```

Do not trigger another workflow manually.

Do not use a token that intentionally creates recursive workflow events.

Do not change workflow files to solve the problem.

Do not continuously poll after all relevant checks reach a terminal state.

---

## 10. Final Verification

After GitHub checks complete, verify:

* PR head SHA matches the commit just pushed
* intended fix is present
* no unintended files changed
* `package.json` and `package-lock.json` were not unexpectedly modified
* applicable PR checks passed

Do not make further changes after successful verification.

---

## 11. Final Response

### Code Review

List confirmed P0/P1 findings only, or:

> No P0/P1 issues found. The pull request is clean.

### Security Audit

List confirmed security findings only, or:

> No security findings identified.

### Tests

List only commands actually executed and their results.

### Changes Made

Describe only fixes actually committed and pushed.

If none:

> No changes were required.

### PR Validation

Report:

* pushed commit SHA
* PR branch
* GitHub PR checks result
* any failed required checks

### Scope

> Only application/project code relevant to the Pull Request was reviewed. `.github/**` and GitHub Actions were completely excluded. Dependency files were protected from unintended modification. Confirmed P0/P1 defects were fixed minimally, validated locally, pushed to the existing PR branch, and the resulting GitHub PR checks were awaited and verified. The operation completed without recursively starting another review.

**Use the actual repository and current PR metadata as the source of truth. Review, fix confirmed defects, validate, push, wait for the new PR checks, verify the final SHA and checks, report, and stop.**
