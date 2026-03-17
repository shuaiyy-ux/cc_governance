# cc_governance

A Claude Code skill that audits and fixes your agent configuration.

Most Claude Code problems are not model problems — they are configuration problems. Context layering is wrong, CLAUDE.md has grown into a wiki nobody reads, skill descriptors are too vague to trigger reliably, hooks are doing things they shouldn't. These issues compound silently until the agent stops being useful.

`cc_governance` scans your project, finds the specific issues, and walks you through fixing them. It shows you exactly what it wants to change before touching anything.

---

## What it does

1. **Scan** — reads your actual `CLAUDE.md`, `.claude/settings.json`, `rules/`, `skills/`, and hooks config
2. **Diagnose** — checks six dimensions: CLAUDE.md quality, context layering, skill descriptor clarity, hook boundaries, verification coverage, session continuity
3. **Propose** — groups findings by priority (fix now / structural / nice to have), with file names and line numbers
4. **Confirm** — shows a before/after diff for every proposed change and waits for your approval
5. **Apply** — writes only what you confirmed, reads back to verify, flags any downstream impact

Nothing is overwritten without explicit confirmation. If a proposed change conflicts with existing content, it shows both versions and asks which to keep.

---

## Install

**Claude Code**
```bash
npx skills add shuaiyy-ux/cc_governance
```

**Manual**
```bash
# Project-level (checked into your repo)
cp -r cc_governance .claude/skills/

# Personal (available across all your projects)
cp -r cc_governance ~/.claude/skills/
```

**claude.ai**

Settings → Features → Skills → upload `cc_governance.skill`

---

## Usage

Once installed, run `/cc_governance` in any Claude Code session, or just describe the problem — Claude will load the skill automatically when you mention issues with your setup.

Works on:
- New projects with no Claude Code config yet
- Existing projects that have accumulated config debt
- Projects mid-refactor where you want to audit before the next phase

---

## What it checks

| Area | What it looks for |
|---|---|
| `CLAUDE.md` | Contract vs wiki, Compact Instructions, NEVER list, line count |
| Context loading | Rules split by path, MCP server overhead |
| Skills | Descriptor trigger clarity, single-responsibility, high-risk workflow guards |
| Hooks | Deterministic-only use, output length, three-layer stack coverage |
| Verification | Acceptance criteria defined, verifier commands bound to task types |
| Session continuity | HANDOFF.md pattern, Compact Instructions present |

---

## Design principles

**Additive, not destructive.** This skill guides Claude to make surgical edits. It does not generate full-file replacements or apply opinionated templates.

**Evidence-based.** Every finding references the specific file and location. No generic advice.

**Confirm before write.** The skill will never modify a file without showing the diff and waiting for a yes.

**No hardcoded values.** Token limits, command syntax, and API behavior change across Claude Code releases. The skill uses web search to verify current behavior before citing specifics.

---

## Background

Built from the engineering principles in [《你不知道的 Claude Code：架构、治理与工程实践》](https://tw93.fun/en/2026-03-12/claude.html) by [@HiTw93](https://x.com/HiTw93) — a deep dive into why Claude Code setups degrade and what the highest-leverage fixes are.

---

## License

MIT
