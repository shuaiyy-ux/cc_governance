---
name: cc-governance
description: >
  Audit and improve Claude Code project configuration. Use this skill whenever
  the user wants to check their Claude Code setup, fix issues with CLAUDE.md,
  Skills, Hooks, context drift, or rules not being followed. Also trigger when
  starting a new project that will use Claude Code, or when the user says things
  like "Claude keeps ignoring my rules", "context is getting messy", "my skills
  aren't triggering", or "set up my project for Claude Code".
---

# Claude Code Governance

## How This Skill Works

1. **Scan** — read the current project's Claude Code config as-is
2. **Diagnose** — identify specific issues with evidence
3. **Propose** — show exactly what to add/change, explain why
4. **Confirm** — wait for user approval before touching anything
5. **Apply** — make only the confirmed changes, preserving everything else

Never overwrite existing content. Always edit surgically.

---

## Step 1: Scan Current State

Read these files and directories if they exist. Note what's missing.

```
CLAUDE.md                        (root contract)
.claude/settings.json            (permissions, allowedTools)
.claude/rules/                   (path-specific rules)
.claude/skills/                  (custom skills)
.claude/hooks/  OR  settings.json hooks key
```

For each file found, read it fully before forming any opinion.
For each file missing, note it but don't assume it's a problem yet—some are optional.

Also run these diagnostic commands if in Claude Code:
```
/context    → reveals token breakdown; MCP overhead is often the hidden culprit
/memory     → confirms which CLAUDE.md actually loaded
```

---

## Step 2: Diagnose Against These Principles

Check each area. Collect evidence before scoring.

### CLAUDE.md health
- Is it acting as a **contract** (commands, constraints) or a **wiki** (explanations, background)?
- Does it have a `## Compact Instructions` section? If not, compression will silently drop architecture decisions.
- Is it under ~150 lines? If longer, check whether content belongs in a skill or rule instead.
- Does it have a `NEVER` list for high-risk operations?

### Context loading
- Are `.claude/rules/` files used for path/language-specific constraints, or is everything crammed into root CLAUDE.md?
- How many MCP servers are connected? Each server adds tool definitions to every request. If the user has /context output available, check the overhead.

### Skills
- For each skill found: does its `description` say **when to invoke it** or just **what it does**?
- Are any skills covering multiple unrelated concerns in one file?
- Are high-risk workflow skills (deploy, migrate) missing `disable-model-invocation: true`?

### Hooks
- Are hooks being used for deterministic tasks (format, lint, notify) or for things requiring judgment?
- Is hook output unbounded? Long output pollutes context.

### Verification
- Is there any definition of "done" for recurring task types in CLAUDE.md or skills?
- Can the user tell you what commands must pass before a task is considered complete?

### Session continuity
- Is there a HANDOFF.md or equivalent? If not and the project is active, sessions are losing context on restart.
- Are Compact Instructions present? (See CLAUDE.md health above.)

---

## Step 3: Present Findings

Group issues by impact. Use this format:

```
## Audit Results

### 🔴 Fix Now (breaks reliability)
[issue] — [specific evidence from their files] — [one-line reason it matters]

### 🟡 Structural Issues (degrades over time)
[issue] — [specific evidence] — [one-line reason]

### 🟢 Nice to Have (low priority)
[issue] — [specific evidence] — [one-line reason]

---
Which of these would you like me to address?
You can say "all of them", "just the red ones", or pick by number.
```

Be specific. Don't say "your CLAUDE.md is too long." Say:
"Lines 45–89 of your CLAUDE.md contain API documentation that Claude can read
directly from your codebase. Moving it to a skill would save ~30 lines of
always-resident context."

---

## Step 4: Propose Changes (Before Touching Anything)

For each confirmed issue, show the exact change first:

**For additions** — show the block to be added and where:
```
I'll add this section to the end of your CLAUDE.md:

## Compact Instructions
When compressing, preserve in priority order:
1. Architecture decisions
2. Modified files and key changes
3. Verification status (pass/fail)
4. Open TODOs and rollback notes

Confirm? (yes / skip / modify)
```

**For edits** — show before/after diff, never the full file:
```
In your release-check skill, I'll change:

BEFORE:
description: Helps with releases

AFTER:
description: Use before cutting a release to verify build, tests, and changelog.

Confirm? (yes / skip / modify)
```

**For new files** — show full content, explain why it doesn't conflict with existing setup.

Always wait for explicit confirmation before writing.

---

## Step 5: Apply Changes

Make only what was confirmed. After each change:
- Read the file back to verify the edit landed correctly
- Report what changed in one line
- Note if anything downstream needs attention (e.g., "this skill's description
  changed—if you have other skills that reference it, check those too")

If a change touches an existing section, preserve all surrounding content exactly.

---

## Edge Cases to Handle Gracefully

**New project, no Claude Code files yet:**
Don't generate a full template. Ask what the project does and what the user wants
Claude Code to help with. Build the minimal CLAUDE.md from that conversation.
A CLAUDE.md that reflects the actual project is better than a complete one that doesn't.

**Existing project with working setup:**
Err on the side of fewer suggestions. If something is working, don't fix it.
Only surface issues with clear evidence of actual problems.

**User wants to batch-apply everything:**
Still show the full list of changes before any write. Confirm once for the batch,
then apply in order, reporting each change as it lands.

**Conflicting existing content:**
Never silently overwrite. If a proposed addition conflicts with something already
in the file, show both versions and ask the user which to keep.

---

## What This Skill Does Not Do

- Does not hardcode values like token limits or command syntax—these change with
  Claude Code releases. If a specific number or command is needed, use web search
  to verify the current behavior first.
- Does not apply any change without explicit user confirmation.
- Does not restructure or reformat files beyond the specific issue being addressed.