# Git Rules

## Configuration

- `cece.name`: Your display name (required)
- `cece.email`: Your email for authorship (required)

---

## Hard Constraints

**NEVER:**
- Commit to `main` or `master`
- Reset or stash changes you did not create
- Discard uncommitted changes without asking

**ALWAYS:**
- Work in a fork owned by your configured account (from `.claude/cece.local.md`)
- Commit with your identity as author, user as committer
- Follow branch naming from `.claude/cece.local.md`
- Alert the user when uncommitted changes exist that you did not make

---

## Commit Identity

Every commit:

| Field     | Value                        |
|-----------|------------------------------|
| Author    | `cece.name <cece.email>`     |
| Committer | `user.name <user.email>`     |

```bash
GIT_COMMITTER_NAME="$(git config user.name)" \
GIT_COMMITTER_EMAIL="$(git config user.email)" \
git commit --author="$(git config cece.name) <$(git config cece.email)>" \
  -m "commit message"
```

---

## Branches

**Protected:** `main`, `master`
- Never commit to these branches

**Your branches:**
- Named per `.claude/cece.local.md` convention
- Commit freely in autonomous mode
- In peer mode, ask permission before committing

---

## Remotes and Forks

**Core requirement:**
- Always work in a fork owned by your configured account
- The account is specified in `.claude/cece.local.md` (e.g., `gh: cece-lthms`)
- Never push to repositories you don't own

**Setup:**
1. Fork the repository to your configured account
2. Add your fork as a remote (typically named `cece`)
3. Set your branch to track your fork, not upstream

```bash
# Fork and add as remote
gh repo fork --remote-name=cece

# Or manually
git remote add cece https://github.com/your-account/repo.git

# Always push to your fork
git push cece branch-name
```

**Verification:**
Before pushing, verify the remote points to your fork:
```bash
git remote get-url cece
# Should show: https://github.com/your-account/repo.git
```

---

## Commit History

Write commits that each represent one logical change.

**Commit messages:**
- Explain what changed and why
- NEVER use generic messages like "Address PR review feedback"
- Write each message so it explains the change without requiring context from
  other commits

**Handling PR reviews:**
- Use fixup commits to address review feedback
- Before requesting another review, squash fixups into the commits they fix
- Rewriting your own branch history between review rounds is allowed

---

## Destructive Operations

If uncommitted changes exist that you did not make:

1. Stop
2. Alert the user
3. Ask how to proceed
4. Never discard automatically

