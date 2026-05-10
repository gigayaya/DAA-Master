# DAA Review Checklist — Full Version

## How to Use

Review each item. Mark as PASS, FAIL, or N/A. Any CRITICAL FAIL must be resolved before code is accepted.

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
  - Good: `test_guest_user_checkout_journey`
  - Bad: `test_post_then_get_api`
- [ ] **TL-9**: Test methods are short sequences of Action calls (typically 2-6 lines)
- [ ] **TL-10**: Test data uses named constants or fixtures, not inline magic values
- [ ] **TL-11**: Test class inherits from or composes the Action Layer (not Physical Layer)

---

## Action Layer Checklist

### Self-Verification (CRITICAL)

- [ ] **AL-1**: Every action method that performs a system operation contains at least one assertion
- [ ] **AL-2**: Assertions verify the operation's outcome, not just that code ran without error
  - Good: `assert response.status == 201`
  - Bad: `assert response is not None` (this only checks something was returned)
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

### Business Logic Boundary (CRITICAL)

- [ ] **PL-1**: No `assert` / `expect` / `should` statements (assertions encode business expectations — belong in Action Layer)
- [ ] **PL-2**: No branching on domain semantics (e.g., `if response.body.role == "admin"`)
- [ ] **PL-3**: No domain-specific exceptions raised (e.g., `raise UserAlreadyExists()`)
- [ ] **PL-4**: No business-level data transformation or interpretation
- [ ] **PL-5**: Any `try` / `catch`, retry, or conditional serves only Physical Layer's own scope (connection management, transport recovery, settling waits) — never business state
- [ ] **PL-6**: Passes the dividing test: this logic would exist even if the caller's business domain were completely different

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
