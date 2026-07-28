# Historical workflow artifact retention

> **Execution**: Use `/execute-task` to implement this plan. After implementation is complete, use `/review-task` to prepare and create the PR.

## Objective

Keep completed execution plans and resolved local issues in Git/PR history only,
not in checked-out `done/` folders, to reduce repository-search noise.

## Changes and verification

- (MODIFY) active-plan/issue README guidance and every current workflow reference to `done/`.
- (MODIFY) `tools/workflow-lint.sh` and focused checks so an active matching plan remains required, while a closeout validates the deleted plan's declared local links and external issue metadata from the diff base.
- (DELETE) `docs/exec-plan/done/**`, any `docs/issues/done/**`, and empty directories.
- Verify compliant and missing-linked-issue closeouts, active-plan enforcement, existing project checks, `git diff --check`, no current `done/` references, and historical retrieval with `git log --all -- docs/exec-plan`.
