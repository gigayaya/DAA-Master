# DAA Naming Conventions

## Core Principle

**Intent over brevity.** Names are contracts with the user (test author). A name should tell the reader exactly what the action does AND what it verifies, without reading the implementation.

## The Naming Formula

```
verb_object_and_verify_outcome
```

### Components

| Part | Purpose | Examples |
|------|---------|----------|
| **verb** | The primary operation | `create`, `delete`, `navigate_to`, `search_for`, `click` |
| **object** | What is being acted upon | `user`, `object`, `product`, `home`, `cart` |
| **and_verify** | Explicit verification contract | `and_verify`, `and_expect`, `and_success` |
| **outcome** | What is verified | `results_not_empty`, `title`, `not_found`, `toast` |

### Patterns

```
// Atomic Actions — verb_object_and_verify_outcome
create_object_and_verify(...)
delete_object_and_verify(...)
get_object_and_expect_not_found(...)
request_by_get_and_success(...)
navigate_to_home_and_verify_title()
search_for_product_and_verify_result_list_not_empty(keyword)
add_first_result_to_cart_and_verify_toast()

// Composite Actions — verb_business_workflow
perform_device_upgrade(...)
complete_checkout_flow_and_verify_order_confirmation(...)
onboard_new_customer_and_verify_welcome_email(...)

// Test Methods — test_scenario_description
test_upgrade_old_phone_to_new_phone()
test_search_for_existing_product()
test_proceed_to_checkout_with_empty_cart()
```

## Why Long Names Are Acceptable

### 1. Semantic Stack Traces

When a test fails, a long name provides complete context without diving into code:

```
AssertionError: Expected payment success message was not found.
  at verify_payment_success_message_is_visible (actions/payment.py:42)
  at submit_credit_card_payment_and_verify_success (actions/payment.py:28)
  at complete_checkout_flow_and_verify_order (actions/checkout.py:15)
  at test_guest_user_checkout_journey (tests/e2e/test_checkout.py:10)
```

Without reading source code, you immediately know: "Guest checkout → payment submission → payment verification failed."

### 2. IDE Fuzzy Search

Modern IDEs handle long names well. Type `upg fw` or `ver cart` to instantly locate the target function. The length is a non-issue in practice.

### 3. Suffixes Are Contracts

The `_and_verify` suffix is not redundant — it is a **contract** with the test author:
- It guarantees the Action contains verification logic
- The test author does NOT need to add assertions in the Test Layer
- Removing the suffix means removing the guarantee

## Anti-Patterns in Naming

| Bad | Why | Good |
|-----|-----|------|
| `fw_upg` | Cryptic, no semantic context | `perform_device_firmware_upgrade_and_verify_version` |
| `do_login` | No verification contract | `login_with_credentials_and_verify_dashboard` |
| `click_save` | Fire-and-forget, no outcome | `click_save_and_verify_confirmation_toast` |
| `helper_create_user` | "helper" prefix adds no meaning | `create_user_and_verify` |
| `utils.check_thing` | Dumping ground module | `user_actions.verify_user_profile_is_visible` |

## Coding Style Trade-Off

If a coding style guideline (e.g., PEP 8's 79-character line limit) conflicts with semantic clarity:

**Semantic clarity wins.** A name like `upgrade_device_firmware_successfully_and_verify_version` exceeds 79 characters, but its clarity for test authors and stack trace readability is worth the trade-off. The Action Layer is an SDK — its naming standard is "Is this clear to the user?" not "Does it fit a line length rule?"
