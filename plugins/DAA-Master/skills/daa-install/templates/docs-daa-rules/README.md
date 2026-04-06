# DAA Rules — Knowledge Base Index

This directory contains the **Declarative Action Architecture (DAA)** rules for E2E test automation in this project.

## When to Read Which File

| Task | Read This File |
|------|---------------|
| Understanding DAA fundamentals | `core-principles.md` |
| Naming action methods | `naming-conventions.md` |
| Avoiding common mistakes | `anti-patterns.md` |
| Writing new test code | `code-generation-rules.md` |
| Reviewing existing test code | `code-review-checklist.md` |
| Scaffolding or restructuring the framework | `project-structure.md` |

## Quick Reference

DAA enforces a strict three-layer separation:

```
Test Layer        →  "What" the user does (100% declarative, zero logic)
Action Layer      →  "How" it's done + verification (every action self-verifies)
Physical Layer    →  The mechanism (thin wrapper around system libraries)
```

Dependencies flow strictly downward: **Test → Action → Physical**. Layer skipping is never allowed.

For the full DAA philosophy, see: [Declarative Action Architecture: A Scalable Pattern for E2E Automation](https://medium.com/@gigayaya/declarative-action-architecture-a-scalable-pattern-for-e2e-automation-1f9a10d24ee0)
