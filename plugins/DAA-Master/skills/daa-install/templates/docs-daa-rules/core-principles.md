# DAA Core Principles

## What is DAA

DAA (Declarative Action Architecture) is a strict three-layer separation pattern for E2E automation testing. It enforces that **tests are declarative**, **actions are self-verifying**, and **physical interactions are pure execution** — eliminating false positives and maximizing maintainability.

## The Three Layers

### Layer 1: Test Layer (The "What")

The Test Layer is **100% declarative**. It describes WHAT the user is doing, not HOW.

**Iron Rules:**
- **ZERO logic**: No `if`, no `for`, no `while`, no `try/catch`
- **ZERO direct system calls**: No raw HTTP requests, no WebDriver calls, no database queries
- **ZERO assertions**: All verification lives in the Action Layer
- Tests read like plain-English documentation — a Product Manager should understand them

```
// GOOD — Pure declarative
test_user_checkout_journey():
    navigate_to_home_and_verify_title()
    search_for_product_and_verify_results_not_empty("iPhone 15")
    add_first_result_to_cart_and_verify_toast()

// BAD — Logic leaked into test
test_user_checkout_journey():
    page.goto(URL)
    if page.title() != "Home":
        raise Error("Wrong page")
    page.fill("#search", "iPhone 15")
    page.click("#submit")
    results = page.query_all(".result")
    assert len(results) > 0
```

### Layer 2: Action Layer (The "How" + Verification)

This is the **heart** of DAA. It translates declarative test steps into execution + verification.

**Iron Rules:**
1. **Self-Verification Mandate**: Every action method MUST verify its own success. You never just "click button"; you "click button and verify modal opens." This eliminates false positives.
2. **No Direct System Access**: Delegates all system interaction to the Physical Layer.
3. **Compose, Don't Repeat**: Build complex workflows by composing smaller actions.

#### Action Hierarchy

| Level | Type | Description | Example |
|-------|------|-------------|---------|
| Level 1 | **Atomic Action** | Single operation + self-verify | `create_user_and_verify()` |
| Level 2 | **Composite Action** | Business workflow = multiple Atomics | `perform_device_upgrade()` |

```
// Atomic Action — does ONE thing, verifies it
create_object_and_verify(url, name, data):
    response = physical_layer.post(url, payload={name, data})  // Delegate
    assert response.status == 201                               // Self-verify
    assert response.body.name == name                           // Self-verify
    return response.body

// Composite Action — orchestrates Atomics into business flow
perform_device_upgrade(url, old_id, new_name):
    old_device = get_object_and_verify(url, old_id)             // Atomic
    new_device = create_object_and_verify(url, new_name, old_device.data)  // Atomic
    delete_object_and_verify(url, old_id)                       // Atomic
    get_object_and_expect_not_found(url, old_id)                // Atomic
    assert new_device.name == new_name                          // Composite self-verify
    return new_device
```

### Layer 3: Physical Layer (The Mechanism)

The Physical Layer is a **"dumb" driver**. It knows HOW to talk to the system but knows NOTHING about business logic.

**Iron Rules:**
- **Pure execution**: No assertions, no business logic, no conditionals
- **Thin wrapper**: Direct delegation to underlying library (HTTP client, WebDriver, Playwright)
- **One method per primitive**: Each method wraps exactly one operation (1-2 lines)

```
// Physical Layer — pure wrapper, zero logic
get(url, params=None):
    return http_client.get(url, params=params)

fill(selector, text):
    driver.find(selector).fill(text)

click(selector):
    driver.find(selector).click()
```

## The Self-Verification Mandate

This is non-negotiable. Every Action MUST verify its own success because:

1. **Eliminates False Positives**: A test that "passes" without verifying the action actually succeeded is worthless — it destroys team trust in automation.
2. **Correctness > Speed**: Extra verification calls cost machine time but save engineering time. Approaching 100% test correctness is worth the marginal cost.
3. **Stack Trace Clarity**: When a self-verifying Atomic Action fails inside a Composite, the stack trace points to the exact failure — no "black box" debugging.

## The Product Mindset

The Action Layer is not a bag of utility functions. It is an **internal SDK** you are responsible for:

- **Treat it like a product**: Ask "Will others understand this 6 months from now?" not "Does it work for me right now?"
- **Stable interfaces**: Action names are contracts. Suffixes like `_and_verify` explicitly tell callers what guarantees the action provides.
- **No dumping grounds**: No `utils.py`, no `helpers_new.py`. Every action belongs to a cohesive, well-named module.

## Dependency Rules

Dependencies flow **strictly downward**:

```
Test Layer → Action Layer → Physical Layer
```

- Test Layer calls ONLY Action Layer
- Action Layer calls ONLY Physical Layer (for system interaction) and other Actions
- Physical Layer calls ONLY the underlying system library
- **Layer skipping is a CRITICAL violation** (e.g., Test calling Physical directly)
