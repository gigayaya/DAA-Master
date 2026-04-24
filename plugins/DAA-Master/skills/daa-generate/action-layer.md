# Generating Action Layer Code

## Purpose

The Action Layer is the **heart of DAA**. It translates declarative test steps into execution logic with built-in verification. Think of it as an **internal SDK** for test authors.

## Atomic Actions (Level 1)

### Definition

An Atomic Action performs exactly ONE operation and verifies its own success.

### Structure Pattern

```
verb_object_and_verify_outcome(params):
    // 1. Delegate to Physical Layer
    result = physical_layer.operation(params)

    // 2. Self-Verification (MANDATORY)
    assert expected_condition(result), "Clear error message with context"

    // 3. Return result for potential use by callers
    return result
```

### Rules

1. **Single Responsibility**: One business operation per Atomic Action
2. **Self-Verification**: At least one assertion verifying the operation succeeded
3. **Physical Layer Only**: All system interactions go through Physical Layer — never import HTTP clients, WebDrivers, or database connectors directly
4. **Descriptive Error Messages**: Include context (URL, expected vs actual, parameters) in assertion messages
5. **Return Values**: Return the operation result so callers (Composite Actions or Tests) can use it

### Examples

```
// API Atomic Action
create_object_and_verify(url, name, data):
    payload = {name: name, data: data}
    response = physical_layer.post(url, payload)       // Delegate

    assert response.status in [200, 201],              // Self-verify
        f"POST {url} expected 200/201, got {response.status}"
    assert response.body.name == name                  // Self-verify
    assert response.body.data == data                  // Self-verify
    assert "id" in response.body                       // Self-verify
    return response.body

// Web Atomic Action
search_for_product_and_verify_result_list_not_empty(keyword):
    physical.fill(Selectors.SEARCH_INPUT, keyword)     // Delegate
    physical.click(Selectors.SEARCH_BUTTON)            // Delegate
    physical.wait_for(Selectors.RESULT_SLOT)           // Delegate

    count = physical.get_count(Selectors.RESULT_ITEM)  // Delegate
    assert count > 0,                                  // Self-verify
        f"Expected >0 results for '{keyword}', got {count}"
```

## Composite Actions (Level 2)

### Definition

A Composite Action represents a **business workflow** — a user goal composed of multiple Atomic Actions.

### Structure Pattern

```
business_workflow_and_verify(params):
    // 1. Orchestrate Atomic Actions
    step1_result = atomic_action_1(params)       // Atomic (self-verifying)
    step2_result = atomic_action_2(step1_result) // Atomic (self-verifying)
    atomic_action_3(...)                          // Atomic (self-verifying)

    // 2. Composite-Level Verification
    assert overall_business_condition(step2_result), "Business workflow verification"

    return final_result
```

### Rules

1. **Compose Atomics Only**: A Composite Action calls ONLY Atomic Actions (Level 1) or other Composites — NEVER the Physical Layer directly
2. **Business-Level Verification**: After orchestrating Atomics, verify the overall business outcome (not just individual steps)
3. **Encapsulate Business Logic**: If a business process changes (e.g., "upgrade now requires email confirmation"), update the Composite — all tests automatically inherit the change
4. **Single Point of Truth**: One Composite per business workflow. Don't duplicate the same sequence across multiple places

### Example

```
// Composite Action — device upgrade business workflow
perform_device_upgrade(url, old_device_id, new_device_name):
    """
    Business flow:
    1. Read old device data (preservation)
    2. Create new device with migrated data
    3. Delete old device (recycling)
    4. Verify old device is gone
    5. Verify new device has correct data
    """
    // Step 1: Read (Atomic — self-verifying)
    old_device = get_object_and_verify(url, old_device_id)
    old_data = get_object_data(old_device)

    // Step 2: Create (Atomic — self-verifying)
    new_device = create_object_and_verify(url, new_device_name, old_data)

    // Step 3: Delete (Atomic — self-verifying)
    delete_object_and_verify(url, old_device_id)

    // Step 4: Verify deletion (Atomic — self-verifying)
    get_object_and_expect_not_found(url, old_device_id)

    // Step 5: Composite-level business verification
    assert get_object_name(new_device) == new_device_name
    assert get_object_data(new_device) == old_data

    return new_device
```

## Helper / Accessor Methods

Small utility methods that extract data from response objects are acceptable in the Action Layer:

```
get_object_id(object_data):
    return object_data["id"]

get_object_name(object_data):
    return object_data["name"]

get_object_data(object_data):
    return object_data.get("data")
```

These don't need self-verification because they are pure data accessors, not system interactions.

## Module Organization

The principle: organize Action Layer files by **business domain**, not by technical function.
The filenames below are one way to express this — adapt them to your project's vocabulary
and language idioms. What matters is that each file represents a coherent business area, not
a grab-bag of unrelated technical operations.

```
// GOOD — domain-aligned (e.g. for an e-commerce project)
actions/
    user_actions.py          // create_user, delete_user, verify_profile
    cart_actions.py          // add_to_cart, remove_from_cart, verify_count
    checkout_actions.py      // submit_payment, verify_order_confirmation
    search_actions.py        // search_product, apply_filter, verify_results

// BAD — technical grouping (regardless of file names)
actions/
    api_helpers.py           // mix of unrelated API calls
    ui_utils.py              // mix of unrelated UI operations
    validators.py            // assertions separated from their actions
```

The exact filenames are up to the team — `users.py`, `user_flows.py`, `customer_actions.py`
are all fine if they clearly map to a business domain. The anti-pattern is mixing technical
groupings (`helpers`, `utils`, `validators`) where business meaning is lost.

## Deciding Atomic vs Composite

```
Is this a single user interaction with one verification?
    → Atomic Action (Level 1)

Is this a multi-step business workflow?
    → Composite Action (Level 2)

Am I calling Physical Layer methods directly?
    → Must be Atomic (Level 1)

Am I calling other Action methods?
    → May be Composite (Level 2)
```
