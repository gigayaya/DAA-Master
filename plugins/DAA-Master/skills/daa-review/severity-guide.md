# DAA Violation Severity Guide

## Severity Levels

### CRITICAL — Must Fix Before Merge

Violations that **break DAA fundamentals** and directly lead to false positives, untraceable failures, or destroy team trust in the test suite.

| Code | Violation | Why Critical | Example |
|------|-----------|--------------|---------|
| AP-1 | Logic in Test Layer | Tests become procedural scripts; lose readability and declarative value | `if page.title != "Home": skip()` in test method |
| AP-2 | Missing self-verification | Actions that don't verify success cause **false positives** — tests pass but nothing was actually validated | `click_save()` without checking save succeeded |
| AP-3 | Business logic in Physical Layer | Physical Layer becomes untestable and tightly coupled; breaks swappability | `assert response.status == 201` or `if body.error_code == "USER_EXISTS": raise ...` in HTTP wrapper (note: transport-level retries on transient connection errors are NOT a violation) |
| CL-1 | Layer skipping | Bypassing Action Layer means no self-verification — equivalent to AP-2 | Test method calling `http.post()` directly |
| PL-1 | Assertions in Physical Layer | Assertions in wrong layer create confusing failures and break separation | `assert response.ok` in HTTP client |

**Decision rule**: If the violation can cause a test to report SUCCESS when the operation actually FAILED, it is CRITICAL.

### WARNING — Should Fix; Defer with Justification

Violations that **hurt maintainability**, readability, or scalability but don't immediately cause false positives.

| Code | Violation | Why Warning | Example |
|------|-----------|-------------|---------|
| AP-4 | Flat procedural tests | Multiple tests duplicate the same sequence; one business change requires updating N tests | 5 tests each containing the same 6-step upgrade sequence |
| AP-5 | Utils/helpers dumping ground | No discoverability; new team members can't find existing actions; duplication grows | `utils.py` with 50 unrelated functions |
| AP-6 | POM mixed responsibilities | Page Objects become large, tangled files; partial migration to DAA creates confusion | LoginPage class with selectors + business logic + assertions |
| AP-8 | Composite calling Physical directly | Sub-steps aren't self-verified; defeats the purpose of the Atomic/Composite hierarchy | `perform_upgrade()` calling `http.delete()` instead of `delete_and_verify()` |
| AL-7 | Missing Composite extraction | Code duplication across tests; maintenance burden grows linearly | Same 5-action sequence in 20 tests |
| AL-11 | Non-semantic naming | Action names don't communicate intent; test readability suffers | `do_login()` instead of `login_and_verify_dashboard()` |

**Decision rule**: If the violation makes the codebase harder to maintain or understand but tests still report correct results, it is WARNING.

### SUGGESTION — Nice to Have

Improvements that enhance **readability, consistency, or developer experience** but have minimal impact on correctness or maintainability.

| Code | Violation | Why Suggestion | Example |
|------|-----------|----------------|---------|
| AL-15 | Domain misalignment | Actions work fine but are harder to discover in large codebases | All actions in one large `actions.py` instead of domain-split files |
| AL-17 | Broad module scope | Module does too many things; harder to navigate | `api_actions.py` covering users, carts, AND payments |
| PL-10 | Scattered selectors | Selector changes require searching multiple files | `"#login-btn"` hardcoded in 3 Physical Layer methods |
| TL-10 | Inline magic values | Tests are slightly harder to read; data isn't reusable | `create_user("John", "john@test.com")` instead of named constants |

**Decision rule**: If fixing it would be nice but skipping it has no practical impact on the team, it is SUGGESTION.

## Escalation

When in doubt about severity:

1. **Ask**: "Can this violation cause a test to lie (report success when it should fail)?"
   - Yes → CRITICAL
   - No → Continue

2. **Ask**: "Will this violation cause pain when the test suite grows from 50 to 500 tests?"
   - Yes → WARNING
   - No → SUGGESTION

## Impact on DAA Score

Each severity level has a fixed impact on the DAA Score (1-10, starting at 10):

| Severity | Deduction per finding | Category cap |
|----------|----------------------|-------------|
| CRITICAL | -2 points | No cap |
| WARNING | -1 point | No cap |
| SUGGESTION | -0.5 points | -2 max total |

Example: 1 CRITICAL + 2 WARNINGs + 3 SUGGESTIONs = 10 - 2 - 2 - 1.5 = 4.5 → **Score: 5 / 10**
