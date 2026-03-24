# Claude Code Audit Setup

A [Claude Code](https://docs.anthropic.com/en/docs/claude-code) skill that performs a comprehensive audit of your Claude Code configuration — CLAUDE.md files, skills, settings, MCP configs, hooks, memory files, and more.

Run `/audit-setup` to eliminate bloat, resolve conflicts, surface stale rules, and get a prioritised changelist you can act on.

## What it audits

| Source | What it checks |
|---|---|
| `CLAUDE.md` | Project-level and global instruction files |
| `settings.json` / `settings.local.json` | Both layers, including conflicts between them |
| Skills & commands | Every `SKILL.md` and slash command |
| Hooks | Hook configurations |
| MCP config | `.mcp.json` (project) and `~/.claude/mcp.json` (global) |
| Memory files | `MEMORY.md` index and all referenced memory files |
| Other instruction files | `.cursorrules`, `.windsurfrules`, agent instructions in READMEs |

## What it evaluates

Every rule, instruction, and preference is assessed against six questions:

1. **Already default behaviour?** — Is Claude doing this without being told?
2. **Conflicts with another rule?** — Contradictions between files or layers?
3. **Redundant / duplicate?** — Same intent covered elsewhere?
4. **Bandaid fix?** — Added to fix one bad output rather than improve outputs overall?
5. **Too vague to be actionable?** — Would it be interpreted differently every time?
6. **Stale or orphaned?** — References tools, paths, or frameworks that no longer exist?

## What it outputs

1. **Setup summary** — File count, rule count, estimated token footprint
2. **Conflicts between files** — With recommendations on which to keep
3. **Recommended cuts** — Sorted by impact (high first), citing which question each fails
4. **Recommended merges** — Overlapping rules consolidated into single rules
5. **Recommended rewrites** — Better wording for rules with good intent but poor phrasing
6. **Global vs. project layer issues** — Rules in the wrong layer
7. **Memory health** — Stale, redundant, orphaned, or broken memory entries
8. **Changelist for CLAUDE.md** — Diff-style remove/merge/reword/reorder

## Installation

Copy the skill into your Claude Code skills directory:

```bash
mkdir -p ~/.claude/skills/audit-setup
cp .claude/skills/audit-setup/SKILL.md ~/.claude/skills/audit-setup/SKILL.md
```

## Usage

```
/audit-setup
```

The skill is configured with `disable-model-invocation: true` so it only runs when you explicitly invoke it — it won't fire mid-task.

## Credits

This skill is based on the original work by [Jarod Taylor](https://github.com/jarodtaylor), published as a [GitHub Gist](https://gist.github.com/jarodtaylor/3f9ab15c62a83dadbc72898ae598d0ad). The concept was shared by [Ole Lehmann](https://x.com/itsolelehmann) in [this post](https://x.com/itsolelehmann/status/2036065138147471665).

This fork adds:
- **Memory file auditing** — scans `MEMORY.md` index and all referenced memory files for staleness, redundancy, orphaned files, and broken index entries
- **`settings.local.json` scanning** — checks both settings layers for conflicts and duplication
- **Memory health output section** — dedicated report section for memory issues

## Licence

MIT
