# DAA Anti-Patterns

Common violations of the Declarative Action Architecture. If you see any of these, flag them immediately.

## Critical Anti-Patterns (Break DAA Fundamentals)

### AP-1: Logic in the Test Layer

**Symptom**: `if`, `for`, `while`, `try/catch`, or raw API/UI calls in test methods.

```
// VIOLATION — Test Layer contains logic
test_search():
    page.goto(URL)                     // Direct system call
    page.fill("#search", "iPhone")     // Direct system call
    results = page.query_all(".item")  // Direct system call
    if len(results) > 0:               // Conditional logic
        assert results[0].text != ""
    else:
        skip("No results")            // Flow control
```

**Fix**: Move ALL logic to Action Layer. Test Layer calls only action methods.

```
// CORRECT — Pure declarative
test_search():
    navigate_to_home_and_verify_title()
    search_for_product_and_verify_result_list_not_empty("iPhone")
```

### AP-2: Fire-and-Forget Actions (Missing Self-Verification)

**Symptom**: Action methods that execute an operation but never verify the outcome.

```
// VIOLATION — No verification after action
click_save_button():
    physical_layer.click("#save-btn")
    // Returns without checking if save actually happened
```

**Fix**: Every action MUST include an assertion verifying its own success.

```
// CORRECT — Self-verifying
click_save_and_verify_confirmation():
    physical_layer.click("#save-btn")
    physical_layer.wait_for_selector(".toast-success")
    assert physical_layer.is_visible(".toast-success")
```

### AP-3: Business Logic in the Physical Layer

**Symptom**: The Physical Layer makes decisions about *business meaning* — asserting expected outcomes, branching on domain semantics, or raising domain-specific errors.

**What is still allowed**: Logic that serves the Physical Layer's own scope — connection/session management, transport-level retries on transient transport errors, `try/catch` that normalizes low-level hardware/IO errors, settling waits required by the underlying protocol.

The dividing test: *would this logic exist even if the caller's business domain were completely different?* If yes, it's a Physical concern. If it encodes what the data means to the application, it belongs in the Action Layer.

```
// VIOLATION — encoding business meaning
post(url, data):
    response = http_client.post(url, json=data)
    assert response.status == 201                       // business expectation
    if response.body.error_code == "USER_EXISTS":       // domain interpretation
        raise UserAlreadyExists()
    return response
```

**Fix**: Keep the Physical Layer scoped to "make the call work." Move all business interpretation to the Action Layer.

```
// CORRECT — pure execution
post(url, data):
    return http_client.post(url, json=data)

// ALSO CORRECT — physical-scope recovery (no business interpretation)
post(url, data):
    try:
        return http_client.post(url, json=data, timeout=30)
    except ConnectionResetError:
        self._reconnect()
        return http_client.post(url, json=data, timeout=30)
```

## Warning Anti-Patterns (Hurt Maintainability)

### AP-4: Flat Procedural Test Scripts

**Symptom**: Tests directly sequence low-level operations without abstraction layers.

```
// VIOLATION — Flat, procedural
test_upgrade():
    old = http.post(URL, {"name": "iPhone 12"})
    data = http.get(URL + "/" + old.id).json()["data"]
    new = http.post(URL, {"name": "iPhone 15", "data": data})
    http.delete(URL + "/" + old.id)
    assert http.get(URL + "/" + old.id).status == 404
    assert new.json()["data"] == data
```

**Impact**: If the upgrade process changes, every test that performs an upgrade must be updated.

**Fix**: Encapsulate in a Composite Action. Tests call one method.

### AP-5: The Utils/Helpers Dumping Ground

**Symptom**: `utils.py`, `helpers.py`, `helpers_new.py`, `common.py` containing a mix of unrelated functions.

```
// VIOLATION — Dumping ground
utils.py:
    def format_date(d): ...
    def create_user(name): ...
    def click_and_wait(sel): ...
    def parse_csv(path): ...
    def retry_request(url): ...
```

**Impact**: No discoverability, no cohesion, impossible to understand the "SDK" surface area.

**Fix**: Organize actions into cohesive modules aligned with business domains.

```
// CORRECT — Cohesive, domain-aligned modules
user_actions.py:     create_user_and_verify(), delete_user_and_verify()
cart_actions.py:     add_to_cart_and_verify_toast(), verify_cart_count()
checkout_actions.py: complete_checkout_and_verify_order()
```

### AP-6: Page Object Model as Mixed-Responsibility Catch-All

**Symptom**: Page Objects containing selectors, business logic, assertions, and utility methods all in one class.

**Impact**: Page Objects become large, tangled files that are hard to maintain and test.

**Fix**: DAA replaces POM with three distinct layers. If migrating from POM:
- Selectors/constants → Physical Layer (or a constants file)
- Business logic + assertions → Action Layer
- Test scenarios → Test Layer

## Suggestion Anti-Patterns (Reduce Quality)

### AP-7: Inconsistent Naming

**Symptom**: Mix of naming conventions across actions — some with `_and_verify`, some without; some long-form, some abbreviated.

**Fix**: Apply the naming formula consistently. See `naming-conventions.md`.

### AP-8: Composite Actions Calling Physical Layer Directly

**Symptom**: A Composite Action (Level 2) bypasses Atomic Actions and calls the Physical Layer directly.

```
// VIOLATION — Composite calling Physical directly
perform_upgrade(url, old_id, new_name):
    old = physical_layer.get(url + "/" + old_id)   // Should use Atomic
    physical_layer.post(url, {name: new_name})     // Should use Atomic
```

**Fix**: Composite Actions should ONLY compose Atomic Actions (Level 1). This ensures every sub-step is self-verified.

```
// CORRECT — Composing Atomics
perform_upgrade(url, old_id, new_name):
    old = get_object_and_verify(url, old_id)       // Atomic (self-verifying)
    create_object_and_verify(url, new_name, old.data)  // Atomic (self-verifying)
```
