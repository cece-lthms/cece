---
description: Create a new CeCe command mode through guided questions
---

# Wizard

Create a new command mode interactively.

## Mode Properties

| Property | Value |
|----------|-------|
| Indicator | 🧙 |
| Arguments | `[name] [scope...]` — name prefills question 1, remaining words prefill question 4 |
| Exit | Command file written, or user sends `stop` |
| Scope | Gather information and generate a command definition file |
| Persistence | None (ephemeral) |
| Resumption | Start over |

## Permissions

**Allowed:**
- Ask questions
- Write to `.claude/commands/` or `~/.claude/commands/`
- Run self-quality-assurance agent

**Forbidden:**
- Modify existing command files without confirmation

---

## Workflow

Announce:

<response>
🧙 Switching to wizard mode.
</response>

### Step 1: Parse arguments

If arguments provided:
- First word → `{name}` (prefills Step 2)
- Remaining words joined → `{scope}` (prefills Step 5)

Examples:
- `/cece:wizard` — no prefill
- `/cece:wizard focus` — name is "focus"
- `/cece:wizard focus Focus on a single question` — name is "focus", scope is
  "Focus on a single question"

If `{name}` is prefilled:
1. Validate format (lowercase letters and hyphens only)
2. Check if a command with that name already exists in both locations
3. If conflict found: warn the user immediately and ask whether to overwrite or
   choose a different name
4. If name is invalid: reject and ask for a different name

### Step 2: Name

Skip if prefilled from arguments.

Ask: "What should this command be called? (lowercase, hyphens allowed)"

Validate:
- Lowercase letters and hyphens only
- Check for conflicts in both `.claude/commands/` and `~/.claude/commands/`

If conflict: ask whether to overwrite or choose a different name.

Store as `{name}`.

### Step 3: Location

Use AskUserQuestion:
- Question: "Where should this command live?"
- Options:
  - Project — available only in this project (`.claude/commands/`)
  - Global — available in all projects (`~/.claude/commands/`)

Store as `{location}`.

### Step 4: Indicator

Use AskUserQuestion:
- Question: "Pick an indicator emoji for this command."
- Options:
  - 🔧 (tools/utility)
  - 🎯 (focused task)
  - 🔍 (investigation/search)
  - Other (let me type one)

If "Other": ask for the emoji in conversation.

Store as `{indicator}`.

### Step 5: Scope

Skip if prefilled from arguments.

Ask: "In one sentence, what does this command do? (from your perspective)"

Store as `{scope}`.

### Step 6: Arguments

Use AskUserQuestion:
- Question: "Does this command take arguments?"
- Options:
  - None — no arguments
  - Yes — I'll describe them

If "Yes": ask for argument description in conversation.

Store as `{arguments}`.

### Step 7: Exit conditions

Ask: "When is this command done? What are the exit conditions?"

Store as `{exit}`.

### Step 8: Persistence

Use AskUserQuestion:
- Question: "Where should this command store state for resumption?"
- Options:
  - None — ephemeral, no resumption needed
  - Issue — state lives in issue/ticket comments
  - File — state lives in a file in the repo
  - External — state lives elsewhere

If "None": set `{resumption}` to "Start over".

If "Issue", "File", or "External": ask how to resume from saved state.

Store `{persistence}` and `{resumption}`.

### Step 9: Permissions

Ask: "What actions should this command be allowed to do without confirmation?"

Store as `{allowed}`.

Ask: "What actions should be forbidden?"

Store as `{forbidden}`.

### Step 10: Interaction patterns

Ask the user how the command should respond to each interaction type:

- **Clarification**: The command needs missing information before it can continue.
- **Approval**: The command must pause before a significant action (creating files,
  pushing code, deleting data).
- **Blocker**: Something goes wrong (error, constraint) and the command cannot
  proceed as planned.

Use AskUserQuestion:
- Question: "What should this command do when it needs more information before
  proceeding?"
- Options:
  - Ask (default) — pause and ask
  - Continue — proceed without asking
  - Revert — undo work and stop cleanly
  - Stop — halt and save state

Store as `{clarification_action}`.

Use AskUserQuestion:
- Question: "What should this command do when it must get your approval before
  continuing?"
- Options:
  - Ask (default) — pause and ask
  - Continue — proceed without asking
  - Revert — undo work and stop cleanly
  - Stop — halt and save state

Store as `{approval_action}`.

Use AskUserQuestion:
- Question: "What should this command do when it encounters an error or cannot
  proceed?"
- Options:
  - Ask (default) — pause and ask
  - Continue — proceed without asking
  - Revert — undo work and stop cleanly
  - Stop — halt and save state

Store as `{blocker_action}`.

**Build the policy block:**
- If all three actions are "ask": omit the `<policy>` block entirely
- Otherwise: include only the lines that differ from "ask"

### Step 11: Workflow

Ask: "Describe what this command should do in sequence. Include enough detail
that I can turn it into numbered steps."

Take the user's free-form description and structure it into numbered steps.

Present the structured workflow:

<response>
Here's how I structured your workflow:

1. Step one description
2. Step two description
3. ...

Does this capture it?
</response>

If user accepts: proceed.

If user gives feedback: adjust the structure, present again.

Repeat until user accepts.

### Step 12: Generate

Generate the command file.

**Perspective transformation:**

The frontmatter description uses the original user-focused scope.
The Mode Properties Scope field uses a rewritten CeCe-perspective scope.

- User-focused: "What will this help me do?"
- CeCe-perspective: "What will I do when you invoke this command?"

Example: User writes "Help me debug code" →
- Frontmatter description: "Help me debug code" (unchanged)
- Mode Properties Scope: "Debug code by analyzing errors and suggesting fixes"

**Policy block:**

Include the `<policy>` block only if at least one action differs from "ask".
If all actions are "ask", omit the entire block.
Include only lines that differ from the default ("ask").

Example — if only blocker differs (clarification and approval are "ask"):

```xml
<policy>
  blocker: revert
</policy>
```

Example — if multiple actions differ:

```xml
<policy>
  approval: continue
  blocker: revert
</policy>
```

**Inline interaction tags:**

Insert inline interaction tags into workflow steps at the exact point where the
interaction happens:
- `<clarification>prompt</clarification>` — where the command needs info
- `<approval>prompt</approval>` — before significant actions
- `<blocker>prompt</blocker>` — where errors might occur

Not all steps need tags — only those where an interaction pattern applies.
The prompt inside each tag describes what the command needs at that point.

**Template without policy block** (all actions are "ask"):

~~~markdown
---
description: {scope}
---

# {name (title case)}

## Mode Properties

| Property | Value |
|----------|-------|
| Indicator | {indicator} |
| Arguments | {arguments} |
| Exit | {exit} |
| Scope | {scope rewritten from CeCe's perspective} |
| Persistence | {persistence} |
| Resumption | {resumption} |

## Permissions

**Allowed:**
{allowed as bullet list}

**Forbidden:**
{forbidden as bullet list}

---

## Workflow

Announce:

<response>
{indicator} Switching to {name} mode.
</response>

{workflow as numbered steps, each with a ### header}
~~~

**Template with policy block** (at least one action differs from "ask"):

~~~markdown
---
description: {scope}
---

<policy>
  {lines for actions that differ from "ask"}
</policy>

# {name (title case)}

## Mode Properties

| Property | Value |
|----------|-------|
| Indicator | {indicator} |
| Arguments | {arguments} |
| Exit | {exit} |
| Scope | {scope rewritten from CeCe's perspective} |
| Persistence | {persistence} |
| Resumption | {resumption} |

## Permissions

**Allowed:**
{allowed as bullet list}

**Forbidden:**
{forbidden as bullet list}

---

## Workflow

Announce:

<response>
{indicator} Switching to {name} mode.
</response>

{workflow as numbered steps, each with a ### header}
~~~

Replace `{indicator}` with the actual emoji character chosen in Step 4.

### Step 13: Review

Run `self-quality-assurance` on the generated content.

Apply all fixes that do not alter the user's intended meaning.

For fixes that would change meaning, ask the user before applying.

### Step 14: Write

Create the directory if needed:
- Project: `.claude/commands/`
- Global: `~/.claude/commands/`

Write the file as `{name}.md`.

### Step 15: Confirm

Announce:

<response>
🧙 Command created: /cece:{name}

Location: {location path}
Indicator: {indicator}

You can now use this command by typing /cece:{name}
</response>

Return to chat mode.
