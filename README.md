# DAA Claude Plugin

A Claude Code skills plugin that teaches the **Declarative Action Architecture (DAA)** — a strict three-layer separation pattern for E2E automation testing.

## What is DAA?

**Declarative Action Architecture (DAA)** is a scalable design pattern for E2E automation testing framework that ensures reliability and clarity. For a deep dive into its philosophy and design, check out the original article:

👉 [**Declarative Action Architecture: A Scalable Pattern for E2E Automation**](https://medium.com/@gigayaya/declarative-action-architecture-a-scalable-pattern-for-e2e-automation-1f9a10d24ee0)

DAA enforces that **tests are declarative**, **actions are self-verifying**, and **physical interactions are pure execution**. This eliminates false positives and maximizes test maintainability.

```
Test Layer        →  "What" the user does (100% declarative, zero logic)
Action Layer      →  "How" it's done + verification (every action self-verifies its success)
Physical Layer    →  The mechanism (thin wrapper around HTTP client, Playwright, etc.)
```

Dependencies flow strictly downward: **Test → Action → Physical**. Layer skipping is never allowed.

## Included Skills

| Skill | Description |
|-------|-------------|
| `daa-core` | Foundational principles, naming conventions (`verb_object_and_verify_outcome`), and anti-pattern catalog (AP-1 through AP-8) |
| `daa-generate` | Code generation rules per layer with a pre-output checklist to enforce DAA compliance |
| `daa-review` | 38-item code review checklist with severity classification (CRITICAL / WARNING / SUGGESTION) and structured report format |
| `daa-architect` | Project scaffolding templates, scaling patterns (Builder, Adapter, Strategy, Chain), and POM-to-DAA migration guide |
| `daa-install` | Installs DAA knowledge base (`docs/daa_rules/`) into a project and updates CLAUDE.md, AGENTS.md, README.md with DAA rule pointers |

## Installation

```bash
/plugin marketplace add https://github.com/gigayaya/DAA-Master
/plugin install DAA-Master@DAA_Master
```

## Usage

Once installed, the skills activate automatically based on context. You can also invoke them directly:

- **Install DAA into a project**: Use `daa:daa-install` to add DAA rules and knowledge base to your project
- **Generate test code**: Ask Claude to write E2E tests and it will follow DAA layer rules
- **Review existing tests**: Ask Claude to review your test code for DAA compliance
- **Scaffold a project**: Ask Claude to design a test automation framework structure
- **Learn DAA**: Ask Claude about DAA principles, naming conventions, or anti-patterns

## Project Structure

```
skills/
├── daa-core/
│   ├── SKILL.md                # Core DAA principles
│   ├── naming-conventions.md   # verb_object_and_verify_outcome pattern
│   └── anti-patterns.md        # AP-1 through AP-8 violation catalog
├── daa-generate/
│   ├── SKILL.md                # Code generation overview + checklist
│   ├── test-layer.md           # Test Layer generation rules
│   ├── action-layer.md         # Action Layer generation rules
│   └── physical-layer.md       # Physical Layer generation rules
├── daa-review/
│   ├── SKILL.md                # Review process overview
│   ├── checklist.md            # 38-item review checklist (TL/AL/PL/CL codes)
│   ├── scoring.md              # DAA Score (1-10) rubric and report template
│   └── severity-guide.md       # CRITICAL/WARNING/SUGGESTION classification
├── daa-architect/
│   ├── SKILL.md                # Framework design overview
│   ├── project-scaffold.md     # Directory structure templates
│   └── scaling-patterns.md     # Builder, Adapter, Strategy, Chain patterns
└── daa-install/
    ├── SKILL.md                # Installation procedure
    └── templates/              # Content templates for target project
        ├── claude-md-section.md
        ├── agents-md-section.md
        ├── readme-md-section.md
        └── docs-daa-rules/     # Knowledge base files (7 files)
```

## License

[Apache License 2.0](LICENSE)
