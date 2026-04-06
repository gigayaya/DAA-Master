# DAA Code Review Checklist

Review automation test code against DAA principles. Classify each violation by severity and provide actionable fix suggestions.

## Review Process

1. Read the code
2. Classify code into layers (Test / Action / Physical)
3. Check each layer against its rules
4. Check cross-layer boundaries
5. Assign severity to each finding
6. Calculate DAA Score and generate report

---

## Test Layer Checklist

### Logic Isolation (CRITICAL)

- [ ] **TL-1**: No `if` / `else` statements in test methods
- [ ] **TL-2**: No `for` / `while` loops in test methods
- [ ] **TL-3**: No `try` / `catch` / `except` blocks in test methods
- [ ] **TL-4**: No direct API calls (HTTP requests, gRPC, etc.) in test methods
- [ ] **TL-5**: No direct UI calls (WebDriver, Playwright, Selenium) in test methods
- [ ] **TL-6**: No direct database queries in test methods
- [ ] **TL-7**: No `assert` statements in test methods (assertions belong in Action Layer)

### Readability (WARNING)

- [ ] **TL-8**: Test method names describe the scenario, not the implementation
- [ ] **TL-9**: Test methods are short sequences of Action calls (typically 2-6 lines)
- [ ] **TL-10**: Test data uses named constants or fixtures, not inline magic values
- [ ] **TL-11**: Test class inherits from or composes the Action Layer (not Physical Layer)

---

## Action Layer Checklist

### Self-Verification (CRITICAL)

- [ ] **AL-1**: Every action method that performs a system operation contains at least one assertion
- [ ] **AL-2**: Assertions verify the operation's outcome, not just that code ran without error
- [ ] **AL-3**: Assertion error messages include context (URL, expected vs actual, parameters)

### Delegation (CRITICAL)

- [ ] **AL-4**: No direct imports of system libraries (requests, selenium, playwright, etc.)
- [ ] **AL-5**: All system interactions go through Physical Layer methods
- [ ] **AL-6**: Action Layer does not instantiate Physical Layer internals (e.g., creating browser pages)

### Composition (WARNING)

- [ ] **AL-7**: Repeated sequences of 3+ action calls across tests are extracted into Composite Actions
- [ ] **AL-8**: Composite Actions call only Atomic Actions (Level 1) — not Physical Layer directly
- [ ] **AL-9**: Composite Actions include their own business-level verification beyond the Atomics' verifications
- [ ] **AL-10**: Each Composite represents ONE business workflow (not multiple unrelated operations)

### Naming (WARNING)

- [ ] **AL-11**: Action methods follow `verb_object_and_verify_outcome` pattern
- [ ] **AL-12**: Method names include verification suffix (`_and_verify`, `_and_success`, `_and_expect`)
- [ ] **AL-13**: No generic names (`do_thing`, `helper_x`, `check_stuff`)
- [ ] **AL-14**: No abbreviated names that sacrifice clarity (`fw_upg` → `perform_firmware_upgrade_and_verify`)

### Organization (SUGGESTION)

- [ ] **AL-15**: Actions are organized by business domain, not technical function
- [ ] **AL-16**: No `utils.py`, `helpers.py`, or catch-all files
- [ ] **AL-17**: Each action module has a clear, focused responsibility

---

## Physical Layer Checklist

### Pure Execution (CRITICAL)

- [ ] **PL-1**: No `assert` / `expect` / `should` statements
- [ ] **PL-2**: No `if` / `else` / `switch` conditionals
- [ ] **PL-3**: No `for` / `while` loops
- [ ] **PL-4**: No `try` / `catch` error handling (let errors propagate)
- [ ] **PL-5**: No retry logic
- [ ] **PL-6**: No data transformation or business decisions

### Thin Wrapping (WARNING)

- [ ] **PL-7**: Each method wraps exactly ONE primitive operation
- [ ] **PL-8**: Methods are typically 1-2 lines of code (direct delegation)
- [ ] **PL-9**: The interface is technology-agnostic (could swap Selenium for Playwright)

### Selectors (SUGGESTION)

- [ ] **PL-10**: UI selectors are centralized in a constants/selectors file
- [ ] **PL-11**: No hardcoded selectors scattered across Physical Layer methods

---

## Cross-Layer Checklist

### Boundary Integrity (CRITICAL)

- [ ] **CL-1**: No layer skipping (Test → Physical without Action in between)
- [ ] **CL-2**: Dependencies flow downward only: Test → Action → Physical
- [ ] **CL-3**: Physical Layer never calls Action Layer (no upward dependency)
- [ ] **CL-4**: Test Layer never imports Physical Layer directly

### Separation of Concerns (WARNING)

- [ ] **CL-5**: Business logic lives ONLY in Action Layer
- [ ] **CL-6**: System interaction details live ONLY in Physical Layer
- [ ] **CL-7**: Test scenarios live ONLY in Test Layer
- [ ] **CL-8**: Changes to UI selectors require modifying ONLY Physical Layer / constants

---

## Severity Classification

| Level | Meaning | Action Required | Score Impact |
|-------|---------|-----------------|-------------|
| **CRITICAL** | Breaks DAA fundamentals — causes false positives or destroys test trust | Must fix before merge | -2 points |
| **WARNING** | Hurts maintainability or violates DAA best practices | Should fix; acceptable to defer with justification | -1 point |
| **SUGGESTION** | Improvement opportunity for readability or consistency | Nice to have; fix when convenient | -0.5 points (max -2 total) |

### Severity Decision Rules

1. "Can this violation cause a test to report SUCCESS when the operation actually FAILED?" → **CRITICAL**
2. "Will this violation cause pain when the test suite grows from 50 to 500 tests?" → **WARNING**
3. Neither? → **SUGGESTION**

---

## DAA Score

Every review report MUST start with a **DAA Score** (1-10).

### Score Rubric

| Score | Grade | Meaning |
|-------|-------|---------|
| 10 | Perfect | Full DAA compliance — no violations found |
| 8-9 | Excellent | Minor SUGGESTION-level issues only |
| 6-7 | Good | WARNING-level issues but no CRITICAL violations |
| 4-5 | Needs Work | 1-2 CRITICAL violations or many WARNINGs |
| 2-3 | Poor | Multiple CRITICAL violations — DAA structure is largely broken |
| 1 | Failing | Code does not follow DAA at all |

### Scoring Rules

1. **Start at 10**, then deduct per finding
2. **Floor at 1** — score never goes below 1
3. **SUGGESTION cap**: Even with 10 SUGGESTIONs, max deduction is 2 points

### Report Template

```
## DAA Review Report

### DAA Score: X / 10

### Summary
- X CRITICAL findings
- Y WARNING findings
- Z SUGGESTION findings

### Findings

#### [CRITICAL] AP-X: Title
**File**: path/to/file.py:line
**Code**: Description of the violation
**Fix**: Specific fix instruction

#### [WARNING] AP-X: Title
**File**: path/to/file.py:line
**Code**: Description of the violation
**Fix**: Specific fix instruction
```
