---
on:
  pull_request:
    types: [labeled]
    forks: ["*"]
checkout: false
labels: [auto-format-needed]
if: github.event.label.name == 'auto-format-needed'
concurrency:
  group: "gh-aw-${{ github.workflow }}-${{ github.event.pull_request.number }}"
  cancel-in-progress: true
description: "Auto-fix code formatting issues in management SDK pull requests"
permissions:
  contents: read
  pull-requests: read
strict: false
tools:
  github:
    toolsets: [context, repos, pull_requests]
  bash: true
safe-outputs:
  push-to-pull-request-branch:
    max: 1
    protected-files: allowed
    allowed-files: ["sdk/"]
  remove-labels:
    max: 1
    target: "${{ github.event.pull_request.number }}"
  add-comment:
    max: 1
    target: "${{ github.event.pull_request.number }}"
    hide-older-comments: false
    footer: false
  messages:
    run-started: "⚡ Format auto-fix workflow started for PR #${{ github.event.pull_request.number }}..."
    run-success: "⚡ Format auto-fix workflow completed. ✅"
    run-failure: "⚡ Format auto-fix workflow {status}. ❌"
timeout-minutes: 15

---

# Management SDK Format Auto-Fix

You are a format auto-fix assistant. Your only task is to run `pnpm format` on the affected package and push the formatted result back to the PR branch.

> **Security note**: This workflow runs with the `pull_request` trigger (not `pull_request_target`), so it has no access to repository secrets. It is safe to checkout and execute PR code here.

## Steps

### Step 1 — Identify the package

List the files changed in this PR using the GitHub API and identify the affected package directory (e.g., `sdk/compute/arm-compute`).

### Step 2 — Run format

```bash
cd <package-dir>
pnpm install --frozen-lockfile=false
pnpm format
```

### Step 3 — Push changes

If any files were modified by the format command, push them to the PR branch via `push-to-pull-request-branch`.

If no files were modified (code was already formatted), skip to Step 4 without pushing.

### Step 4 — Update labels and post comment

1. Remove the `auto-format-needed` label via `remove-labels`.
2. If files were pushed, post a comment:

   > ✅ **Auto-formatted**: `pnpm format` has been applied. Please pull the latest changes to sync your local branch.

   If no files were modified, do not post a comment.
