---
name: daa-install
description: Use when installing DAA rules into a project, setting up DAA conventions, or bootstrapping a project to follow Declarative Action Architecture for E2E test automation
---

# DAA Project Installer

## Overview

This skill installs DAA (Declarative Action Architecture) rules and conventions into the user's current project. It creates a knowledge base directory for on-demand reference and updates existing configuration files to point agents to those rules.

## What Gets Installed

| Target | Action | Content |
|--------|--------|---------|
| `docs/daa_rules/` | **Created** (always) | 7 knowledge base files covering DAA principles, naming, anti-patterns, code generation, code review, and project structure |
| `CLAUDE.md` | **Modified** (only if exists) | Minimal DAA section with pointer table to `docs/daa_rules/` |
| `AGENTS.md` | **Modified** (only if exists) | Minimal DAA section with pointer table to `docs/daa_rules/` |
| `README.md` | **Modified** (only if exists) | Brief DAA architecture section |

## Installation Procedure

Follow these steps IN ORDER. Do not skip steps.

### Step 1: Safety Check

Determine the user's current working directory. This is the project where DAA will be installed.

**STOP if** the current directory is inside the DAA plugin repository itself (contains `plugins/DAA-Master/skills/`). Warn the user: "This skill should be invoked from your own project, not the DAA plugin repo."

### Step 2: Survey Existing Files

Check which of these files exist in the project root:
- `CLAUDE.md`
- `AGENTS.md`
- `README.md`
- `docs/daa_rules/` directory

For each existing file, check if it already contains the marker `<!-- DAA:BEGIN`. This determines whether this is a fresh install or an update.

### Step 3: Show Installation Plan

Before making any changes, tell the user exactly what will be done:

**Always created/updated:**
- `docs/daa_rules/` — 7 knowledge base files (overwritten if they exist)

**Only if the file exists:**
- For each of CLAUDE.md, AGENTS.md, README.md:
  - File exists + no DAA marker → **APPEND** DAA section
  - File exists + DAA marker found → **REFRESH** (replace DAA block with latest)
  - File does not exist → **SKIP** (report to user)

Ask the user to confirm before proceeding.

### Step 4: Create Knowledge Base

Create the `docs/daa_rules/` directory in the project root. Copy the content of each file from this skill's `templates/docs-daa-rules/` directory:

1. `docs/daa_rules/README.md` — Index file with file-to-task mapping
2. `docs/daa_rules/core-principles.md` — Three-layer model, self-verification mandate
3. `docs/daa_rules/naming-conventions.md` — `verb_object_and_verify_outcome` formula
4. `docs/daa_rules/anti-patterns.md` — AP-1 through AP-8 catalog
5. `docs/daa_rules/code-generation-rules.md` — Layer-specific generation rules + pre-output checklist
6. `docs/daa_rules/code-review-checklist.md` — 38-item checklist + severity + DAA Score
7. `docs/daa_rules/project-structure.md` — Scaffold templates + scaling patterns

### Step 5: Update Configuration Files

For each of CLAUDE.md, AGENTS.md, README.md — **only if the file exists**:

1. Read the template content from this skill's `templates/` directory:
   - CLAUDE.md → `templates/claude-md-section.md`
   - AGENTS.md → `templates/agents-md-section.md`
   - README.md → `templates/readme-md-section.md`

2. Apply the template:
   - **If the file has no `<!-- DAA:BEGIN` marker**: Append the template content at the end of the file, preceded by a blank line.
   - **If the file already has `<!-- DAA:BEGIN` marker**: Replace everything from `<!-- DAA:BEGIN` through `<!-- DAA:END -->` (inclusive) with the new template content.

3. **NEVER create** CLAUDE.md, AGENTS.md, or README.md if they don't exist. Skip and report.

### Step 6: Report Results

Show the user a summary:

```
DAA Installation Complete
─────────────────────────
✓ docs/daa_rules/  — [created / updated] (7 files)
✓ CLAUDE.md        — [appended / refreshed / skipped (file not found)]
✓ AGENTS.md        — [appended / refreshed / skipped (file not found)]
✓ README.md        — [appended / refreshed / skipped (file not found)]

DAA rules are now available in this project.
AI agents will read docs/daa_rules/ on demand when working with test code.
```

## Idempotency

This skill is safe to run multiple times:
- `docs/daa_rules/` files are overwritten with latest content on each run
- The `<!-- DAA:BEGIN -->` / `<!-- DAA:END -->` markers in config files ensure only the DAA block is replaced, leaving all user-written content untouched

## What This Skill Does NOT Do

- Does NOT create CLAUDE.md, AGENTS.md, or README.md — only modifies existing ones
- Does NOT modify any source code files
- Does NOT scaffold a test framework structure — use `daa:daa-architect` for that
- Does NOT install any dependencies or packages
