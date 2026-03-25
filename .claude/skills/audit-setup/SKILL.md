---
name: audit-setup
description: Comprehensive audit of your Claude Code configuration — CLAUDE.md, skills, context files, settings, MCP configs, and hooks. Use when you want to clean up, deduplicate, or optimize your Claude Code setup. Invoke this skill whenever you mention auditing, reviewing, cleaning up, or optimizing your Claude Code configuration, memory files, or instruction files.
disable-model-invocation: true
---

# Claude Code Setup Audit

You are performing a comprehensive audit of the user's Claude Code configuration. The goal is to eliminate bloat, resolve conflicts, surface stale rules, and produce a prioritized changelist the user can act on.

## Phase 1: Discovery

Read everything before forming any opinions. Do not respond until you have completed this full scan.

### Scan these locations (in order):

1. **Project-level config** (`.claude/` in the current project root)
   - `CLAUDE.md` (project instructions)
   - `settings.json` and `settings.local.json`
   - `skills/` (every SKILL.md and any bundled references)
   - `commands/` (slash commands)
   - `hooks/`
   - Any other `.md` or config files in `.claude/`

2. **Global config** (`~/.claude/`)
   - `CLAUDE.md` (global instructions)
   - `settings.json` and `settings.local.json` (check both — local overrides can conflict with or duplicate shared settings)
   - `skills/` (every SKILL.md and any bundled references)
   - `commands/`
   - `hooks/`

3. **MCP configuration**
   - `.mcp.json` in the project root
   - `~/.claude/mcp.json` (global MCP config)

4. **Memory files**
   - `~/.claude/projects/*/memory/MEMORY.md` (index file for the current project)
   - All memory files referenced by the index
   - Check for: stale memories that reference things that no longer exist, memories that duplicate CLAUDE.md rules, orphaned memory files not listed in the index, and index entries pointing to files that no longer exist

5. **Other instruction files**
   - `.cursorrules`, `.windsurfrules`, or similar if present
   - Any `README.md` sections that appear to contain agent instructions
   - Any other `*.md` files in the project root that look like they contain rules or preferences

Build a mental inventory of every rule, instruction, convention, and preference you find across all files before proceeding.

## Phase 2: Analysis

For **each** rule, instruction, or preference you found, evaluate it against these six questions:

### Q1 — Already Default Behavior?
Is this something Claude already does without being told? If Claude's base behavior already covers it, the rule is dead weight consuming context tokens for no benefit.

### Q2 — Conflicts With Another Rule?
Does this contradict or tension with another rule elsewhere in the setup? Pay special attention to conflicts **between layers** (global vs. project-level) since project-level should override global, but contradictions still cause confusion.

### Q3 — Redundant / Duplicate?
Does this repeat something already covered by a different rule or file? Look for both exact duplicates and semantic duplicates (same intent, different wording).

### Q4 — Bandaid Fix?
Does this read like it was added to fix one specific bad output rather than improve outputs overall? These tend to be overly narrow, reference a specific incident, or micromanage a single behavior.

### Q5 — Too Vague to Be Actionable?
Is this so vague that it would be interpreted differently on every invocation? Examples: "be more natural," "use a good tone," "write clean code," "be thorough." If you can't objectively verify compliance, the rule is noise.

### Q6 — Stale or Orphaned?
Does this reference tools, file paths, frameworks, APIs, dependencies, or workflows that no longer exist in the project? Check whether referenced paths actually exist and whether mentioned tools are still configured.

## Phase 3: Token Cost Assessment

Estimate the approximate token cost of the user's full configuration (all loaded CLAUDE.md content, skill descriptions, and always-loaded instruction files). Note:
- What percentage of the setup is actionable vs. dead weight
- Which files or sections are the biggest offenders
- Whether the skill description budget (~2% of context window) is being pressured
- **Per-item token estimates**: for each issue you identify (cuts, merges, rewrites, memory cleanup), estimate the tokens that would be saved. These feed into the savings table in the output.

## Phase 4: Output

Present your findings in this exact structure:

### 1. Setup Summary
A brief overview of what you found: how many files, how many distinct rules/instructions, and the estimated token footprint.

### 2. Conflicts Between Files
List every conflict found between any two files. For each conflict:
- **File A**: [path] — the rule
- **File B**: [path] — the contradicting rule
- **Recommendation**: which to keep and why

### 3. Recommended Cuts
A numbered list of everything you'd remove. For each item:
- **Location**: file path and the specific rule
- **Reason**: one-line explanation citing which question (Q1–Q6) it fails
- **Impact**: low / medium / high (how much context is being wasted)

Sort by impact (high first).

### 4. Recommended Merges
Rules that aren't individually wrong but should be consolidated. For each:
- **Rules to merge**: list the overlapping rules and their locations
- **Proposed single rule**: the merged version

### 5. Recommended Rewrites
Rules that have the right intent but are too vague, too narrow, or poorly worded. For each:
- **Current**: the rule as-is
- **Proposed**: the improved version
- **Why**: what's better about the new version

### 6. Global vs. Project Layer Issues
Any rules that are in the wrong layer (e.g., project-specific rules in global config, or generic preferences buried in a project-level file that should be global).

### 7. Memory Health
- **Stale memories**: memories referencing files, tools, or patterns that no longer exist
- **Redundant memories**: memories that simply restate a CLAUDE.md rule (the rule is already loaded; the memory wastes tokens)
- **Orphaned files**: memory files on disk not listed in MEMORY.md
- **Broken index entries**: MEMORY.md entries pointing to files that no longer exist
- For each issue, recommend: update, delete, or merge into another memory

### 8. Changelist for CLAUDE.md
Rather than a full rewrite, provide a **diff-style changelist**:
- Lines/sections to **remove** (with reason)
- Lines/sections to **merge** (with proposed replacement)
- Lines/sections to **reword** (with proposed replacement)
- Suggested **reordering** if the current structure buries important rules

This applies to both global and project CLAUDE.md if both exist.

### 9. Savings Table

A markdown table summarising the token cost of every recommended change. This gives the user a clear picture of the ROI for each action.

| # | Action | File | Tokens Saved | % of Config |
|---|--------|------|-------------|-------------|
| 1 | [short description of change] | [file path] | ~[n] | [x%] |
| ... | ... | ... | ... | ... |
| | **Total** | | **~[n]** | **[x%]** |

- One row per recommended cut, merge, rewrite, or memory cleanup
- Sort by tokens saved (largest first)
- Include a totals row at the bottom
- Token estimates don't need to be exact — order-of-magnitude is fine. Use word count as a proxy (1 word ≈ 1.3 tokens).

### 10. Executive Summary

This is the last thing the user sees — it must be immediately actionable. Maximum 10 lines.

Format as a numbered list of actions in priority order. Each item is a single line:
`[n]. [verb] [what] in [file] (~[tokens] saved)`

End with a total line:
`Total estimated savings: ~[n] tokens ([x%] of your loaded config)`

Frame as direct instructions ("Remove…", "Merge…", "Delete…"), not findings ("I found…", "There are…").

## Guidelines

- Be direct. If a rule is useless, say so — the user asked for honesty.
- Preserve intent. When proposing merges or rewrites, don't lose the user's actual preferences.
- Respect layers. Global config = cross-project defaults. Project config = overrides for this specific codebase.
- Don't invent new rules. Only work with what exists.
- If a rule is good and well-written, skip it silently. Only surface problems.
- When in doubt about whether something is "default behavior," err on the side of keeping it — false positives are more annoying than a slightly heavy config.
