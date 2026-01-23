---
name: git-sync-branch
description: Syncs the current branch with a base ref (fetch, rebase if needed, force-push). Use when resuming work on an existing branch to ensure it's up to date.
tools: Bash, Read
model: haiku
---

You sync the current branch with a specified base ref.

## Input

You receive:

- `base_ref` — the ref to sync with (e.g., `origin/main` or a branch name)
- `remote` — the remote to fetch from and push to
- `push_remote` — the remote to push to (may differ from fetch remote in fork workflows)

## Steps

1. Fetch the base ref: `git fetch <remote> <ref>`
2. Check if branch is up to date: `git merge-base --is-ancestor <base-ref> HEAD`
3. If exit code 0: branch is up to date — skip to Output
4. If exit code 1: branch is behind or diverged — rebase:
   a. Run `git rebase <base-ref>`
   b. If conflicts occur:
      - Read the conflicting files
      - Attempt to resolve the conflicts
      - Run `git add <resolved-files>` then `git rebase --continue`
   c. If conflicts persist after one retry:
      - Run `git rebase --abort`
      - Return error with conflict details
5. Force-push the rebased branch: `git push <push_remote> HEAD --force-with-lease`

## Output Format

On success:

```
status: synced
action: <none|rebased>
```

On conflict that couldn't be resolved:

```
status: conflict
files: <comma-separated list of conflicting files>
message: <brief description of the conflict>
```

## Notes

- Always use `--force-with-lease` for force pushes
- Only attempt automatic conflict resolution once
- If the base ref doesn't exist, return an error
