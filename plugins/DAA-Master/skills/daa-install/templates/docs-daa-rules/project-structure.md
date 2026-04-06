# DAA Project Structure

## Standard Directory Structure

```
project-root/
│
├── lib/                              # Framework core (non-test code)
│   │
│   ├── api/                          # API testing domain
│   │   ├── __init__.py
│   │   ├── action_layer.py           # Action Layer: API business actions
│   │   │                             # - Atomic: create_object_and_verify()
│   │   │                             # - Composite: perform_device_upgrade()
│   │   │                             # - Self-verification in every method
│   │   │
│   │   └── physical_layer.py         # Physical Layer: HTTP client wrapper
│   │                                 # - Pure execution: get(), post(), put(), delete()
│   │                                 # - No assertions, no business logic
│   │
│   ├── web/                          # Web/UI testing domain
│   │   ├── __init__.py
│   │   ├── action_layer.py           # Action Layer: UI business actions
│   │   ├── physical_layer.py         # Physical Layer: browser driver wrapper
│   │   └── constants.py              # UI selectors and URLs
│   │
│   └── __init__.py
│
├── tests/                            # Test Layer (100% declarative)
│   │
│   ├── api/
│   │   ├── __init__.py
│   │   ├── test_basic_crud.py        # Basic CRUD test scenarios
│   │   └── test_business_workflows.py # Composite workflow scenarios
│   │
│   ├── web/
│   │   ├── __init__.py
│   │   └── test_search_flows.py      # Web UI test scenarios
│   │
│   └── conftest.py                   # Shared fixtures (Physical Layer init, env config)
│
├── config/                           # Environment configuration
│   ├── dev.yaml
│   ├── staging.yaml
│   └── production.yaml
│
├── docs/
│   └── daa_rules/                    # DAA rules reference (this directory)
│
├── requirements.txt
└── README.md
```

## Layer Placement Rules

| Content Type | Belongs In | Never In |
|-------------|------------|----------|
| Test scenarios | `tests/` | `lib/` |
| Business actions + verification | `lib/*/action_layer.py` | `tests/`, `physical_layer.py` |
| System interaction wrappers | `lib/*/physical_layer.py` | `tests/`, `action_layer.py` |
| UI selectors / URLs | `lib/*/constants.py` | Scattered across any file |
| Test fixtures / setup | `tests/conftest.py` | `lib/` action files |
| Environment config | `config/` | Hardcoded in code |

## Scaling the Structure

### Small project (< 50 tests)

Single action layer file per domain is sufficient:

```
lib/
├── api/
│   ├── action_layer.py      # All API actions in one file
│   └── physical_layer.py
```

### Medium project (50-200 tests)

Split action layer by business domain:

```
lib/
├── api/
│   ├── actions/
│   │   ├── user_actions.py       # User CRUD actions
│   │   ├── order_actions.py      # Order workflow actions
│   │   └── payment_actions.py    # Payment actions
│   └── physical_layer.py
```

### Large project (200+ tests)

Add composite actions directory and shared base classes:

```
lib/
├── api/
│   ├── actions/
│   │   ├── base_action.py        # Shared action utilities
│   │   ├── user_actions.py
│   │   ├── order_actions.py
│   │   └── payment_actions.py
│   ├── composites/
│   │   ├── checkout_flow.py      # Multi-step business workflows
│   │   └── onboarding_flow.py
│   └── physical_layer.py
```

## Multi-Domain Projects

For projects testing multiple interfaces (API + Web + Mobile):

```
lib/
├── api/                    # API domain — own Action + Physical stack
│   ├── action_layer.py
│   └── physical_layer.py
├── web/                    # Web domain — own Action + Physical stack
│   ├── action_layer.py
│   ├── physical_layer.py
│   └── constants.py
├── mobile/                 # Mobile domain — own Action + Physical stack
│   ├── action_layer.py
│   ├── physical_layer.py
│   └── constants.py
└── shared/                 # Cross-domain utilities
    └── data_helpers.py     # Data generation, format conversion (no system calls)
```

Each domain has its own complete Action + Physical stack. The Test Layer can compose across domains if needed.

## Scaling Patterns

When Action Layer complexity grows, apply the appropriate pattern:

| Symptom | Pattern | Apply When |
|---------|---------|------------|
| Same action sequence repeated across tests | **Composite** | 3+ tests share the same sequence |
| Action method has 6+ parameters | **Builder** | Parameters represent a complex domain object |
| External API format doesn't match internal expectations | **Adapter** | Physical Layer needs format normalization |
| Action method has `if/else` branches | **Strategy** | Different execution paths based on type/mode |
| Steps must execute in strict order, each depending on previous output | **Chain** | Pipeline or provisioning workflows |

### Composite Pattern

Encapsulate repeated sequences of Atomic Actions into a single Composite Action:

```
// BEFORE — same sequence in many tests
test_upgrade():
    old = get_object_and_verify(url, old_id)
    new = create_object_and_verify(url, "iPhone 15", old.data)
    delete_object_and_verify(url, old_id)
    assert new.data == old.data

// AFTER — one Composite, reused everywhere
test_upgrade():
    old = create_object_and_verify(url, "iPhone 12", old_data)
    perform_device_upgrade(url, get_object_id(old), "iPhone 15")
```

### Builder Pattern

Manage complex action parameters with a builder:

```
UserBuilder:
    with_name(first, last): ...
    with_email(email): ...
    with_role(role, department): ...
    build(): return self.data

// Usage
user_data = UserBuilder().with_name("Jane", "Doe").with_email("jane@ex.com").build()
create_user_and_verify(URL, user_data)
```

### Adapter Pattern

Normalize external API formats in the Physical Layer:

```
ExternalAPIAdapter:
    get_user(user_id):
        raw = raw_client.get_user(user_id)
        return {"name": raw["user_name"], "email": raw["user_email"]}
```

### Strategy Pattern

Replace `if/else` branching with separate action methods:

```
// Instead of: submit_payment(type, ...) with if/else
submit_credit_card_payment_and_verify(card, expiry, amount): ...
submit_paypal_payment_and_verify(email, amount): ...
submit_bank_transfer_and_verify(account, routing, amount): ...
```

### Chain Pattern

Model strict sequential pipelines as Composite Actions:

```
provision_and_activate_service(config):
    instance = provision_service_and_verify(config)
    configured = configure_service_and_verify(instance.id, config.settings)
    activate_service_and_verify(configured.id)
    verify_service_is_running(configured.id)
    return configured
```

## Fixture / Configuration Patterns

Fixtures handle Physical Layer initialization and environment setup. They must NOT contain business logic.

```
// conftest.py — GOOD
fixture api_client():
    client = APIClient(base_url=config.API_URL)
    return client

fixture web_driver(playwright_page):
    driver = PlaywrightDriver(playwright_page)
    return driver
```

## Migration from Page Object Model

When converting existing POM code to DAA:

1. **Identify** selectors/locators in Page Objects → move to Physical Layer (or constants)
2. **Extract** business logic and assertions from Page Objects → move to Action Layer
3. **Simplify** test files to pure declarative calls → Test Layer
4. **Verify** no test file imports Page Objects or Physical Layer directly
5. **Add** self-verification to every Action method (the step most POM code is missing)
