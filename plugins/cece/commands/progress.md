---
description: Execute work on an issue with an existing plan
---

# Progress Mode

## Mode Properties

| Property | Value |
|----------|-------|
| Indicator | 🔥 |
| Arguments | `<issue-ref>` — issue number or URL (required) |
| Exit | Task completion, or user sends `stop` |
| Scope | Execute planned work independently |
| Persistence | Updates Plan comment + Q&A in issue description (reads Definition of Done + Architectural Decisions) |
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
- Push per `## Git Strategy` in `.claude/cece.local.md`
- Create/update PRs
- Post issue comments

**NEVER:**
- Commit to `main` or `master`
- Push to unauthorized remotes
- Close issues
- Merge PRs
- Violate an Architectural Decision without raising a blocker

---

## Artifacts

### Definition of Done

A `## Definition of Done` section in the issue description. Created by
`/cece:plan`. These define what "done" means.

**Format:** Lite user stories — simplified statements: `As a <role>, I want to <action> so that <outcome>`

**Examples:**
```markdown
- [ ] As a user, I want to toggle dark mode so that I can reduce eye strain
- [ ] As a developer, I want auth in a separate module so that I can test it independently
```

**NEVER** check off Definition of Done boxes. Only the user checks items off.

### Work Plan

A comment on the issue with the `## Work Plan` heading. Created by `/cece:plan`.

**Update when:**
- PR is completed (check it off, add link)
- PR scope changes based on review feedback

### Q&A

A `## Q&A` section in the issue description. This is the decision log — key
decisions, constraints, and learnings.

**Purpose:** Preserve context across sessions. When resuming, read this first and
refer to prior decisions before proposing changes to settled approaches.

**What belongs:**
- Constraints discovered during implementation ("API doesn't support batch ops")
- Clarifications from user that affect approach
- Learnings from PR reviews that apply to remaining work

**Update when:**
- After blocker resolution (constraints discovered)
- After significant review feedback (learnings)

### Architectural Decisions

A `## Architectural Decisions` section in the issue description. Created by
`/cece:plan`. These are invariants that must be preserved.

**NEVER** violate an architectural decision. If implementation requires breaking
one, raise a blocker immediately — do not proceed.

**Read-only:** Only `/cece:plan` creates or modifies this section. If you discover
a new constraint during implementation, raise it as a blocker — never modify this
section directly.

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

Argument is required. The issue must have an existing Plan (created by
`/cece:plan`).

### Step 1: Load context

1. Read `## Project Management` in `.claude/cece.local.md` to determine the platform
2. Fetch the issue (content, comments, labels, linked PRs)
3. Read the Definition of Done section from the issue description
4. Read the Architectural Decisions section from the issue description
5. Find the Plan comment posted by your configured account (from `## Identity` in `.claude/cece.local.md`)

**If no plan exists:**

<response>
🔥 No plan found for this issue. Run `/cece:plan <issue-ref>` first.
</response>

Return to chat mode.

### Step 2: Validate and resume

1. Read the Q&A section from the issue description
2. Validate completeness:
   - Definition of Done section exists in issue description
   - Architectural Decisions section exists in issue description
   - Plan comment has task, approach, test plan, and planned PRs
   - If anything is missing, ask user whether to run `/cece:plan` first
3. Parse current state: which PRs are done, pending, any blockers
4. Check open PRs for unaddressed reviews
5. Present summary to user: what's planned, done, remaining, pending reviews

Announce:

<response>
🔥 Resuming progress on issue.
</response>

6. If reviews are pending, proceed to Step 4. Otherwise, proceed to Step 3.

### Step 3: Execution

**Before any implementation:**

1. Extract every Definition of Done item from the issue description
2. Create a todo item for each
3. These todos track requirement coverage — mark each complete only after the
   code is committed (and tests pass, unless waived)

Work through each planned PR:

1. **Branch**: Create or checkout branch per naming convention in `.claude/cece.local.md`
2. **Git setup**: Read `## Git Strategy` from `.claude/cece.local.md` and prepare
3. **Implement**: Write code, commit freely
4. **Test**: Execute the test plan. If tests fail, fix before proceeding.
   - If test plan says "User approved: no tests", skip testing for this PR
   - If test plan cannot be executed for other reasons, raise as blocker
5. **PR**: When scope is complete:
   - **Gate**: Before creating the PR, list which Definition of Done items this PR
     addresses. If you have not fully implemented all Definition of Done items
     this PR addresses, return to the Implement step to complete the missing
     work, or raise a blocker if a constraint prevents completion.
   - Create PR linking to the issue ("Fixes #N" or "Part of #N")
   - Assign user as reviewer
   - Update Plan: check off completed PR, add link
6. **Repeat** for remaining PRs

### Step 4: Handling Reviews

When PR reviews come in, evaluate each comment:

1. Does it change what "done" means? → Ask user before implementing
2. Would it violate an Architectural Decision? → Raise blocker, ask user before implementing
3. Otherwise → Implement the change

NEVER decline review feedback without user approval. If you believe a comment
should not be addressed, present your reasoning to the user and request approval
before declining.

After addressing comments:

4. Push fixes
5. In each thread, explain what you changed or why you declined the feedback (with user approval)
6. Update the Plan comment if PR scope changed based on review
7. Update Q&A if review revealed significant learnings
8. If review requires changes to Definition of Done or Architectural Decisions,
   tell the user to run `/cece:plan` to revise — do not modify these sections

### Step 5: Blockers

A blocker is anything that prevents full implementation of a requirement:
- Tests fail unexpectedly
- Design question emerges
- Missing information
- Technical constraints that force a compromise
- Implementation would violate an Architectural Decision

**NEVER silently compromise.** If you cannot implement exactly what was asked,
raise it as a blocker. Partial solutions require explicit user approval.

When blocked:

1. Post blocker as comment (on PR if exists, otherwise on issue)
2. Ask user for clarification in conversation
3. Present options when possible
4. Once user approves an option, continue
5. Update Q&A with the constraint and decision

### Step 6: Completion

When all planned PRs are created:

1. **Pre-check**: Re-fetch the Definition of Done from the issue description. For
   each item, identify which code covers it (and tests, unless waived). If you
   cannot point to concrete implementation, the item is not met — raise a blocker
   before proceeding.
2. Verify all PRs are checked off in the Plan comment
3. Run the full test plan to verify all PRs work together (skip if "User approved: no tests")
4. **Review each Definition of Done item:**
   - Confirm the implementation meets the requirement exactly
   - If any item is not fully satisfied, raise a blocker
   - NEVER declare completion with unmet requirements
5. Return to chat mode
6. Present final summary: what was delivered, how each Definition of Done item was met
7. Ask user what to do next

**NEVER** mark Definition of Done checkboxes complete — only the user does that.

NEVER close issues; closure happens automatically when PRs merge.
