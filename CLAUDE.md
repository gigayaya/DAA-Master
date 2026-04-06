# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Project Is

This is a **Claude Code Skills plugin** that teaches Claude the **Declarative Action Architecture (DAA)** — a strict three-layer separation pattern for E2E automation testing. It is not an application; it contains no runnable code, only Markdown-based skill definitions consumed by Claude Code's skill system.

## Architecture

The plugin provides four skills, each in its own directory under `plugins/DAA-Master/skills/`:

| Skill | Purpose | Entry Point |
|-------|---------|-------------|
| `daa-core` | Foundational principles, naming conventions, anti-pattern catalog | `SKILL.md` |
| `daa-generate` | Code generation rules per layer (test, action, physical) with pre-output checklist | `SKILL.md` |
| `daa-review` | Code review checklist (38 items), severity classification (CRITICAL/WARNING/SUGGESTION), report format | `SKILL.md` |
| `daa-architect` | Project scaffolding templates, scaling patterns (Builder/Adapter/Strategy/Chain), POM migration guide | `SKILL.md` |
| `daa-install` | Installs DAA knowledge base into a project, updates CLAUDE.md/AGENTS.md/README.md with DAA rule pointers | `SKILL.md` |

Each skill directory has a `SKILL.md` (the entry point with frontmatter metadata) plus supporting `.md` files for detailed reference. Skills reference each other: `daa-generate`, `daa-review`, and `daa-architect` all require `daa-core` as prerequisite context.

## The DAA Three-Layer Model

All skills revolve around this strict hierarchy — understand it before editing any skill content:

- **Test Layer** ("What"): 100% declarative, zero logic/assertions/system calls. Tests read like plain English.
- **Action Layer** ("How" + Verification): The core layer. Every action method **must self-verify** its own success (the "Self-Verification Mandate"). Split into Atomic (single op + verify) and Composite (orchestrate Atomics into business workflows).
- **Physical Layer** ("Mechanism"): Pure "dumb" wrapper around system libraries (HTTP client, Playwright, etc.). Zero assertions, zero business logic, 1-2 line methods.

**Dependencies flow strictly downward**: Test → Action → Physical. Layer skipping is a CRITICAL violation.

## Editing Skills

- `SKILL.md` files use YAML frontmatter (`name`, `description`) — the `description` field determines when Claude Code activates the skill, so keep it precise.
- Supporting `.md` files are referenced from `SKILL.md` via relative paths (e.g., `→ Full generation rules: action-layer.md`).
- Anti-pattern codes (AP-1 through AP-8), checklist item codes (TL-*, AL-*, PL-*, CL-*), and severity levels are cross-referenced across skills — keep them consistent if renaming or adding items.
- The naming convention `verb_object_and_verify_outcome` is a core contract referenced by all four skills.

## Key Design Decisions

- **Semantic clarity over brevity**: Long method names like `upgrade_device_firmware_and_verify_version` are intentional — they provide self-documenting stack traces and act as API contracts. This explicitly overrides line-length style guides.
- **No utils/helpers pattern**: Actions must be organized by business domain (`user_actions.py`, `cart_actions.py`), never in catch-all files.
- **Self-verification is non-negotiable**: An action without verification is classified as CRITICAL (AP-2). "It works" is not sufficient — it must be provably correct.
