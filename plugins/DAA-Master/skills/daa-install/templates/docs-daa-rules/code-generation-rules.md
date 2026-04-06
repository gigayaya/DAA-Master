# DAA Code Generation Rules

When generating E2E test automation code, follow the Declarative Action Architecture strictly. Every piece of code belongs to exactly ONE of the three layers, and must follow that layer's rules.

## Pre-Output Checklist

Before outputting ANY generated test code, verify ALL items:

- [ ] **Test Layer has ZERO logic?** No if/for/while/try, no raw system calls, no assertions
- [ ] **Every Action self-verifies?** Every action method contains at least one assertion
- [ ] **Physical Layer has ZERO assertions?** No assert, no business decisions
- [ ] **Naming follows semantic convention?** `verb_object_and_verify_outcome` pattern
- [ ] **Composite Actions compose Atomics?** Level 2 calls Level 1, not Physical Layer directly
- [ ] **Layers don't skip?** Test → Action → Physical (never Test → Physical)

If ANY item fails, fix the code before outputting.

---

## Test Layer Rules

### Structure Pattern

```
TestClass(inherits ActionLayer):

    CONSTANTS:
        BASE_URL = "..."
        TEST_DATA = {...}

    test_scenario_description():
        // Setup (using Action methods only)
        action_setup_step()

        // Execute (using Action methods only)
        action_primary_step()

        // NO assertions here — they live in Action Layer
```

### R1: No Logic — Absolute Zero

| Forbidden | Why |
|-----------|-----|
| `if / else` | Conditional paths belong in Action Layer |
| `for / while` | Loops belong in Action Layer |
| `try / catch` | Error handling belongs in Action Layer |
| `assert` | Verification belongs in Action Layer |
| Raw API calls | System interaction belongs in Physical Layer |
| Variable manipulation | Data processing belongs in Action Layer |

**Exception**: Assigning return values from Action methods to variables for passing to subsequent Action methods is acceptable.

### R2: Test Method Naming

Test methods describe the **scenario**, not the implementation:

```
// GOOD — describes the scenario
test_upgrade_old_phone_to_new_phone()
test_search_for_existing_product()
test_proceed_to_checkout_with_empty_cart()

// BAD — describes implementation details
test_post_and_delete_api()
test_fill_form_click_submit()
```

### R3: Inheritance / Composition

The Test class must access Action Layer methods through inheritance or composition:

```
// Inheritance approach (simpler)
TestCheckout(inherits CheckoutActionLayer):
    test_guest_checkout():
        self.navigate_to_home_and_verify_title()
        self.search_for_product_and_verify_results("MacBook")

// Composition approach (more flexible)
TestCheckout:
    actions = CheckoutActionLayer()

    test_guest_checkout():
        actions.navigate_to_home_and_verify_title()
        actions.search_for_product_and_verify_results("MacBook")
```

### R4: Test Data

Test data should be defined as class-level constants or fixtures, not inline magic values.

### Complete Test Layer Example

```
TestAmazonSearch(inherits AmazonActionLayer):
    """Test Layer: E2E tests for product search functionality."""

    test_search_for_existing_product():
        navigate_to_home_and_verify_title()
        search_for_product_and_verify_result_list_not_empty("iPhone 15")

    test_search_for_non_existent_product():
        navigate_to_home_and_verify_title()
        search_for_product_and_expect_no_results("xyz_nonexistent_random_123")

    test_product_detail_consistency():
        navigate_to_home_and_verify_title()
        search_for_product_and_verify_result_list_not_empty("Apple MacBook Air")
        title, price = get_first_result_title_and_price()
        click_first_result_and_verify_detail_page_matches(title, price)
```

---

## Action Layer Rules

### Atomic Actions (Level 1)

An Atomic Action performs exactly ONE operation and verifies its own success.

```
verb_object_and_verify_outcome(params):
    // 1. Delegate to Physical Layer
    result = physical_layer.operation(params)

    // 2. Self-Verification (MANDATORY)
    assert expected_condition(result), "Clear error message with context"

    // 3. Return result for potential use by callers
    return result
```

**Rules:**
1. **Single Responsibility**: One business operation per Atomic Action
2. **Self-Verification**: At least one assertion verifying the operation succeeded
3. **Physical Layer Only**: All system interactions go through Physical Layer
4. **Descriptive Error Messages**: Include context (URL, expected vs actual, parameters)
5. **Return Values**: Return the operation result so callers can use it

### Composite Actions (Level 2)

A Composite Action represents a **business workflow** composed of multiple Atomic Actions.

```
business_workflow_and_verify(params):
    // 1. Orchestrate Atomic Actions
    step1_result = atomic_action_1(params)
    step2_result = atomic_action_2(step1_result)

    // 2. Composite-Level Verification
    assert overall_business_condition(step2_result), "Business workflow verification"

    return final_result
```

**Rules:**
1. **Compose Atomics Only**: Never call Physical Layer directly
2. **Business-Level Verification**: Verify the overall business outcome beyond individual steps
3. **Encapsulate Business Logic**: If a process changes, update the Composite — all tests inherit the change
4. **Single Point of Truth**: One Composite per business workflow

### Helper / Accessor Methods

Small utility methods that extract data from response objects are acceptable:

```
get_object_id(object_data):
    return object_data["id"]
```

These don't need self-verification because they are pure data accessors, not system interactions.

### Module Organization

Organize Action Layer files by **business domain**, not by technical function:

```
// GOOD — domain-aligned
actions/
    user_actions.py
    cart_actions.py
    checkout_actions.py
    search_actions.py

// BAD — technical grouping
actions/
    api_helpers.py
    ui_utils.py
    validators.py
```

### Deciding Atomic vs Composite

- Single user interaction with one verification? → **Atomic Action (Level 1)**
- Multi-step business workflow? → **Composite Action (Level 2)**
- Calling Physical Layer methods directly? → Must be **Atomic (Level 1)**
- Calling other Action methods? → May be **Composite (Level 2)**

---

## Physical Layer Rules

### R1: Pure Execution — Zero Intelligence

The Physical Layer must contain:
- **NO assertions** (`assert`, `expect`, `should`, etc.)
- **NO conditionals** (`if`, `switch`, ternary)
- **NO loops** (`for`, `while`)
- **NO error handling** (`try/catch`) — let errors propagate to Action Layer
- **NO business logic** (no data transformation, no decision-making)
- **NO retry logic** — retries are a business decision for the Action Layer

### R2: Thin Wrapping

Each method should be a direct, 1-2 line delegation to the underlying library.

### R3: One Method Per Primitive

Each Physical Layer method wraps exactly one primitive operation. Don't combine multiple operations.

### R4: Technology Swappability

The Physical Layer is the ONLY place where the underlying technology is referenced. Switching from Selenium to Playwright or `requests` to `httpx` requires only Physical Layer changes. Action Layer and Test Layer remain untouched.

### Constants / Selectors

UI selectors and URLs should be extracted into a constants file, separate from the Physical Layer:

```
Selectors:
    BASE_URL = "https://example.com"
    SEARCH_INPUT = "#twotabsearchtextbox"
    SEARCH_BUTTON = "#nav-search-submit-button"
    RESULT_ITEM = "[data-component-type='s-search-result']"
```
