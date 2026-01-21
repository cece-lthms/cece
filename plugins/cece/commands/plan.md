---
description: Plan work on an issue collaboratively before execution
---

# Plan Mode

## Mode Properties

| Property | Value |
|----------|-------|
| Indicator | 📋 |
| Arguments | `[issue-ref]` — issue number or URL |
| Exit | Plan posted to issue, or user sends `stop` |
| Scope | Collaborative planning for a task |
| Persistence | Plan comment + Definition of Done + Architectural Decisions + Q&A in issue description |
| Resumption | Re-invoke with same issue-ref to revise plan |

## Permissions

**Allowed:**
- Read files, search code
- Fetch issues
- Post issue comments
- Edit issue descriptions

**NEVER:**
- Create branches
- Write code
- Create PRs

---

## Artifacts

### Definition of Done

A `## Definition of Done` section in the issue description. These are the
requirements that define "done" for this issue.

**NEVER** check off Definition of Done boxes. Only the user checks items off.

**Format:** Lite user stories — simplified statements that capture who benefits, what
they want, and why: `As a <role>, I want to <action> so that <outcome>`

The "so that" clause states the value or outcome the role receives, not the
technical mechanism.

```markdown
## Definition of Done

- [ ] As a <role>, I want to <action> so that <outcome>
```

**Examples:**
```markdown
- [ ] As a user, I want to toggle dark mode from settings so that I can reduce eye strain
- [ ] As a developer, I want auth logic in a separate module so that I can test it independently
- [ ] As an API consumer, I want a 404 response for missing users so that I can distinguish "not found" from other errors
```

**Counter-examples (avoid these):**
- `Implement dark mode` — task, not outcome; missing role and benefit
- `The API should return 404` — states what, not why; omits the role
- `Fix the Enter key bug` — describes a problem, not a desired outcome
- `As a user, I want dark mode` — omits the "so that" clause; no outcome stated

### Work Plan

A comment on the issue with the `## Work Plan` heading.

**Required sections:**
- Task summary (one sentence)
- Approach (high-level strategy)
- Test plan (how changes will be validated)
- Planned PRs (checkboxes with scope descriptions)

**Format:**
```markdown
## Work Plan

**Task**: <summary>

**Approach**: <strategy>

**Test plan**: <validation method, or "User approved: no tests" if waived>

**Planned PRs**:
- [ ] PR 1: <scope>
- [ ] PR 2: <scope>
```

### Q&A

A `## Q&A` section in the issue description. This is the decision log — key
decisions, constraints, and learnings from planning.

**Format:**
```markdown
## Q&A

- **<question>?** <answer/decision>
- **<question>?** <answer/decision>
```

Example:
```markdown
- **Why not use the built-in cache?** It doesn't support TTL, so we use Redis.
- **Should we backfill existing records?** No, only new records get the flag.
```

### Architectural Decisions

A `## Architectural Decisions` section in the issue description. These are
invariants that must be preserved during implementation.

**Format:**
```markdown
## Architectural Decisions

- <decision>: <rationale>
- <decision>: <rationale>
```

**Examples:**
```markdown
- All API responses use JSON:API format: ensures consistency with existing endpoints
- Auth tokens stored in httpOnly cookies, never localStorage: prevents XSS token theft
- No direct database access from controllers: all queries go through repository layer
```

**What belongs here:**
- Technology choices with rationale
- Patterns that must be followed
- Constraints from existing architecture
- Security requirements

**What does NOT belong here:**
- Implementation details that can change (e.g., "use async/await" — this is how, not what)
- Preferences without strong rationale
- Decisions already captured in Q&A

---

## Workflow

### Usage

```
/cece:plan [issue-ref]
```

- With argument: plan work for the referenced issue
- Without argument: help user describe the task, create an issue, then plan

### Step 1: Determine the issue

**If argument provided:**

1. Read `## Project Management` in `.claude/cece.local.md` to determine the platform
2. If the URL's tracker does not match your configured tracker, tell the user
   and request confirmation before proceeding
3. Fetch the issue (content, comments, labels, linked PRs)

**If no argument:**

1. Ask the user to describe the task
2. Ask questions until the task is unambiguous
3. Create a new issue capturing the agreed task
4. Proceed with that issue

Announce:

<response>
📋 Switching to plan mode.
</response>

### Step 2: Check for existing plan

Look for a Plan comment posted by your configured account (from `## Identity` in `.claude/cece.local.md`).

**If plan exists:**

1. Read the Q&A and Architectural Decisions sections from the issue description
2. Read all comments on the issue (including review feedback, blockers, updates)
3. Check for linked PRs — if any exist, work has already started
4. Analyze whether the plan needs revision:
   - Are there unresolved blockers or constraints discovered?
   - Has feedback suggested scope changes?
   - Is the Definition of Done still accurate?
   - Are Architectural Decisions still valid?
   - Is the test plan still valid?
5. Present the existing plan to the user with your assessment:
   - If PRs exist: warn that revising the plan may require rework on existing PRs
   - If plan looks current: suggest proceeding to `/cece:progress`
   - If plan may need updates: explain what seems outdated and suggest revising
6. Wait for user to confirm their intent before proceeding
7. If user wants to revise, proceed to Step 3

**If no plan:**
- Proceed to Step 3

### Step 3: Draft plan

1. Explore the codebase to understand the task
2. Draft the Plan (see Artifacts for required sections)
3. Present plan to the user in conversation
4. Iterate based on feedback until user is satisfied

**Test plan is mandatory.** If you cannot identify how to test the changes,
raise this during planning. The user must explicitly approve proceeding without
tests — NEVER skip this step.

### Step 4: Sign-off

1. Ask for explicit sign-off on the plan
2. Wait for user approval before posting

Do NOT post the Plan to the issue until the user approves.

### Step 5: Post to issue and exit

After sign-off:

1. Post the Plan as a comment on the issue
2. Create or update the Definition of Done section in the issue description
3. Create or update the Architectural Decisions section in the issue description
4. Create or update the Q&A section in the issue description with the decisions
   you made during planning
5. Return to chat mode

Announce:

<response>
🐱 Plan posted to issue #<N>. Run `/cece:progress <issue-ref>` to start execution.
</response>
