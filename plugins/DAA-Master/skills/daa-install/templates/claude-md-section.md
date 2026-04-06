<!-- DAA:BEGIN — DO NOT EDIT THIS BLOCK MANUALLY. Managed by daa-install. -->
## DAA (Declarative Action Architecture)

This project follows **DAA** for all E2E test automation. Before writing or modifying test code, read the relevant rule file from `docs/daa_rules/`:

| Task | Read |
|------|------|
| Understand DAA fundamentals | `docs/daa_rules/core-principles.md` |
| Name action methods | `docs/daa_rules/naming-conventions.md` |
| Avoid common mistakes | `docs/daa_rules/anti-patterns.md` |
| Write new test code | `docs/daa_rules/code-generation-rules.md` |
| Review existing test code | `docs/daa_rules/code-review-checklist.md` |
| Scaffold or restructure the framework | `docs/daa_rules/project-structure.md` |

**Key rules (always apply, no need to read files for these):**
- Test Layer: ZERO logic, ZERO assertions, ZERO system calls
- Action Layer: every method MUST self-verify its outcome
- Physical Layer: pure execution wrapper, ZERO assertions
- Dependencies: Test → Action → Physical (never skip layers)
- Naming: `verb_object_and_verify_outcome`
<!-- DAA:END -->
