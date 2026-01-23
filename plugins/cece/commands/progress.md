---
description: Execute work on an issue with an existing plan
---

<policy>
  clarification: ask
  approval: continue
  blocker: ask
</policy>

# Progress Mode

## Mode Properties

| Property | Value |
|----------|-------|
| Indicator | 🔥 |
| Arguments | `<issue-ref>` — issue number or URL (required) |
| Exit | Work session end, task completion, or user sends `stop` |
| Scope | Execute planned work independently |
| Persistence | Updates Plan comment; can post comments |
| Resumption | Re-invoke with same issue-ref |

## Principles

**Requirements are commitments.** Once the user approves the plan, every Definition
of Done item is a promise. NEVER drop, weaken, or partially implement a requirement
without explicit user approval. If you cannot deliver exactly what was agreed,
raise it as a blocker — do not silently reduce scope.

**Persistence over convenience.** When implementation is difficult, explore
alternative approaches before concluding something is impossible. Only raise a
blocker after genuine effort to solve the problem.

---

## Permissions

Execute freely without asking for approval once work begins.

**Allowed:**
- Read files, search code
- Write/edit files
- Run tests
- Create branches (per naming convention)
- Commit to your branches
- Push per `## Git Strategy` in `.cece/config.md`
- Create/update PRs
- Post issue comments
- Edit own Plan comment (to check off completed PRs)

**NEVER:**
- Commit to `main` or `master`
- Push to unauthorized remotes
- Close issues
- Merge PRs
- Check off Definition of Done boxes (only the user does that)
- Edit issue description (use `/cece:scope` for that)
- Edit Design comment (use `/cece:design` for that)
- Violate an Architectural Decision without raising a blocker
- Trigger CI pipelines autonomously (CI can be expensive or slow; only the user
  decides when to run pipelines)

---

## Artifacts

### Goal

The introduction of the issue — the opening text before any sections. Created by
`/cece:scope`. This explains the problem, context, and desired outcome.

**Read-only:** Only `/cece:scope` creates or modifies this section.

### Definition of Done

A `## Definition of Done` section in the issue description. Created by
`/cece:scope`. These define what "done" means.

**Read-only:** Only `/cece:scope` modifies this section. If you discover that a
requirement needs changing, tell the user to run `/cece:scope`.

### Design Comment

A comment on the issue containing Approach, Architectural Decisions, and Q&A.
Created by `/cece:design`.

**Read-only:** Only `/cece:design` modifies this comment. If you discover
constraints that should be recorded in Q&A or require changes to Architectural
Decisions, tell the user to run `/cece:design`.

### Plan Comment

A comment on the issue with the `## Plan` heading. Created by `/cece:plan`,
updated by `/cece:progress`.

**Update when:**
- PR is completed (check it off, add link)
- PR scope changes based on review feedback

### Comments

Posted on the issue or PRs during execution.

**Where to post:**
- Blockers → on the PR if one exists, otherwise on the issue
- Progress updates → on the issue (optional, for long tasks)
- Review responses → on the PR threads

---

## Workflow

### Usage

```
/cece:progress <issue-ref>
```

Argument is required. The issue must have:
- Definition of Done (from `/cece:scope`)
- Design comment (from `/cece:design`)
- Plan comment (from `/cece:plan`)

### Step 1: Load context

1. Read `## Project Management` in `.cece/config.md` to determine the platform
2. If the URL's tracker does not match your configured tracker:
   <clarification>This issue is on a different tracker than configured — should
   I proceed or stop?</clarification>
3. Fetch the issue (content, comments, labels, linked PRs)
4. Read the Definition of Done section from the issue description
5. Find the Design comment posted by your configured account (from `## Identity`
   in `.cece/config.md`)
6. Find the Plan comment posted by your configured account

**If Definition of Done is missing:**

<response>
🔥 This issue has no Definition of Done. Run `/cece:scope <issue-ref>` first.
</response>

Return to chat mode.

**If Design comment is missing:**

<response>
🔥 This issue has no design. Run `/cece:design <issue-ref>` first.
</response>

Return to chat mode.

**If Plan comment is missing:**

<response>
🔥 No plan found for this issue. Run `/cece:plan <issue-ref>` first.
</response>

Return to chat mode.

### Step 2: Validate and resume

1. Read the Approach, Architectural Decisions, and Q&A from the Design comment
2. Read the Plan comment (task, test plan, planned PRs)
3. Parse current state: which PRs are done, pending, any blockers
4. Check open PRs for unaddressed reviews
5. **Check CI status** for all open PRs:
   - Query CI/pipeline status via platform API (e.g., `gh pr checks`)
   - If any PR has failing CI, note the failures for the summary
   - Treat CI failures like review feedback: attempt to fix them during execution
6. Present summary to user: what's planned, done, remaining, pending reviews, CI status
7. Announce:

<response>
🔥 Resuming progress on issue.
</response>

8. Proceed to Step 3. (Step 4 applies when reviews arrive during execution.)

### Step 3: Execution

**Before any implementation:**

1. Extract every Definition of Done item from the issue description
2. Create a todo list in conversation with one item per Definition of Done requirement
3. These todos track requirement coverage — mark each complete only after the
   code is committed (and tests pass, unless waived)

Work through each planned PR:

1. **Git setup**: Read `## Git Strategy` from `.cece/config.md` and prepare
2. **Upstream info**: Spawn the `git-upstream-info` agent. It returns `upstream_remote`
   and `default_branch`. Use these values in substeps 3, 4, 8, and when creating PRs.
   If the agent returns an error, raise a <blocker>I couldn't determine the
   target of the PR — [agent error message]</blocker>
3. **Branch**: Create or checkout branch per naming convention in `.cece/config.md`.
   For new branches, create from `<upstream_remote>/<default_branch>`:
   `git checkout -b <branch-name> <upstream_remote>/<default_branch>`
4. **Freshness check** (existing branches only, skip for new branches):
   - Determine the base ref: if this PR depends on another (marked in Plan),
     use that PR's branch (or `<upstream_remote>/<default_branch>` if merged).
     Otherwise use `<upstream_remote>/<default_branch>`.
   - Spawn the `git-sync-branch` agent with the base ref. If it returns a
     conflict, raise a <blocker>Rebase conflict — [agent message]</blocker>
5. **Implement**: Write code to implement the planned PR, committing as you progress
6. **Test**: Execute the test plan. If tests fail, fix before proceeding.
   - If test plan says "User approved: no tests", skip testing for this PR
   - If test plan cannot be executed for other reasons, raise as blocker
7. **PR**: When PR scope is complete:
   - **Gate**: Before creating the PR, confirm which Definition of Done items this
     PR implements. Verify the PR fully implements those items. If incomplete,
     either complete the missing work, split across multiple PRs, or raise a
     blocker if a constraint prevents completion.
   - Create PR targeting `<default_branch>`, linking to the issue ("Fixes #N" or "Part of #N")
   - Assign user as reviewer
   - Update Plan comment: check off completed PR, add link
8. **Rebase dependents**: If this PR has dependent branches (marked in Plan),
   spawn the `git-rebase-dependents` agent. If it returns conflicts, raise a
   <blocker>Rebase conflict in dependent — [agent message]</blocker>
9. **Repeat** for remaining PRs

### Step 4: Handling Reviews

When PR reviews come in, evaluate each comment:

1. Does it change what "done" means? → <clarification>This review feedback changes the Definition of Done — should I implement it?</clarification>
2. Would it violate an Architectural Decision? → <blocker>This review feedback conflicts with an Architectural Decision — how should I proceed?</blocker>
3. Does it add work beyond the planned scope? → <clarification>This review feedback adds work beyond the planned scope — should I implement it?</clarification>
4. Otherwise → Implement the change

After addressing comments:

5. Push fixes to your branch per `## Git Strategy` in `.cece/config.md`
6. **Rebase dependents**: If this PR has dependent branches (marked in Plan),
   spawn the `git-rebase-dependents` agent after pushing your fixes.
7. In each thread, explain what you changed or why you declined the feedback (with user approval)
8. Update the Plan comment if PR scope changed based on review
9. If review requires changes to Definition of Done, tell the user to run `/cece:scope`
10. If review requires changes to Approach, Architectural Decisions, or Q&A, tell the
    user to run `/cece:design`

### Step 5: Blockers

A blocker is anything that prevents full implementation of a requirement:
- Tests fail unexpectedly
- Design question emerges
- Missing information
- Technical constraints that force a compromise
- Implementation would violate an Architectural Decision

When blocked:

1. Post blocker as comment (on PR if exists, otherwise on issue)
2. <blocker>Cannot implement the requirement as specified — what constraint prevents completion and how should I proceed?</blocker>
3. Present options when possible
4. Once user approves an option, continue
5. If the decision should be recorded, tell the user to run `/cece:design` to
   update Q&A

### Step 6: Work Session End

A **work session** ends when you have completed all addressable work and are
waiting for external input. Exit to chat mode when:

- All planned PRs are created, pushed, and waiting for review
- All review feedback has been addressed and pushed
- All fixable CI failures have been addressed and pushed
- You hit a blocker that requires user decision

When exiting at a work session boundary (not task completion):

<response>
🐱 Pausing progress on #<N>. [Brief summary of what was done]

Re-run `/cece:progress <issue-ref>` to continue.
</response>

### Step 7: Task Completion

When all planned PRs are created and the task is fully complete:

1. Verify all PRs are checked off in the Plan comment
2. Run the full test plan to verify all PRs work together (skip if "User approved: no tests")
3. For each Definition of Done item, confirm the implementation meets it exactly.
   If any item is unmet, raise a blocker — do not declare completion.
4. Present final summary: what was delivered, how each Definition of Done item was met

<response>
🐱 All work complete for issue #<N>. [Summary of what was delivered]
</response>
