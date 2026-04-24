# DAA Project Scaffold Template

## Example Directory Structure

The filenames and directory names below are one way to structure a DAA project — adapt them to your project's conventions and language idioms. What matters is that the three layers are **clearly separated** into distinct files or directories, with dependencies flowing only downward.

```
project-root/
│
├── lib/                              # Framework core (non-test code)
│   │                                 # (could also be: src/, framework/, support/)
│   │
│   ├── api/                          # API testing domain
│   │   ├── action_layer.py           # Action Layer: API business actions
│   │   │                             # - Atomic: create_object_and_verify()
│   │   │                             # - Composite: perform_device_upgrade()
│   │   │                             # - Self-verification in every method
│   │   │
│   │   └── physical_layer.py         # Physical Layer: HTTP client wrapper
│   │                                 # - Pure execution only
│   │                                 # - No assertions, no business logic
│   │
│   ├── web/                          # Web/UI testing domain
│   │   ├── action_layer.py           # Action Layer: UI business actions
│   │   │                             # - navigate_to_home_and_verify_title()
│   │   │                             # - search_and_verify_results_not_empty()
│   │   │
│   │   ├── physical_layer.py         # Physical Layer: browser driver wrapper
│   │   │                             # - Pure execution only
│   │   │                             # - No assertions, no business logic
│   │   │
│   │   └── constants.py              # UI selectors and URLs
│   │                                 # (could also be: selectors.py, locators.py, pages.py)
│   │                                 # - Centralized selector management
│   │
│   └── __init__.py
│
├── tests/                            # Test Layer (100% declarative)
│   │
│   ├── api/
│   │   ├── test_basic_crud.py        # Basic CRUD test scenarios
│   │   └── test_business_workflows.py # Composite workflow scenarios
│   │
│   ├── web/
│   │   └── test_search_flows.py      # Web UI test scenarios
│   │
│   └── conftest.py                   # Shared fixtures
│                                     # - Physical Layer initialization
│                                     # - Environment configuration
│                                     # - No business logic
│
├── config/                           # Environment configuration (optional)
│
├── requirements.txt                  # Dependencies
└── README.md                         # Project documentation
```

## Layer Placement Rules

The key question for any piece of code is: which layer's responsibility does this belong to?

| Content Type | Layer | Must NOT appear in |
|-------------|-------|--------------------|
| Test scenarios | Test Layer | Action or Physical files |
| Business actions + self-verification | Action Layer | Test files or Physical Layer |
| System interaction wrappers | Physical Layer | Test files or Action Layer |
| UI selectors / URLs | Constants file (part of Physical domain) | Scattered across multiple files |
| Test fixtures / setup | Test configuration (e.g. conftest) | Action Layer files |
| Environment config | Config directory | Hardcoded in code |

## Scaling the Structure

### Small project (< 50 tests)

A single file per layer per domain is sufficient. No need for subdirectories.

```
lib/api/
├── actions.py           # All API actions (atomic + composite) in one file
└── client.py            # Physical Layer: HTTP client wrapper
```

### Medium project (50-200 tests)

Split the Action Layer by business domain as it grows:

```
lib/api/
├── actions/
│   ├── user_actions.py       # User CRUD actions
│   ├── order_actions.py      # Order workflow actions
│   └── payment_actions.py    # Payment actions
└── client.py                 # Physical Layer stays as one file
```

### Large project (200+ tests)

Separate composite workflows from atomic actions, and add shared base classes:

```
lib/api/
├── actions/
│   ├── base.py               # Shared action utilities
│   ├── user_actions.py
│   ├── order_actions.py
│   └── payment_actions.py
├── workflows/                # (or: composites/, flows/)
│   ├── checkout_flow.py      # Multi-step business workflows
│   └── onboarding_flow.py
└── client.py                 # Physical Layer
```

## Fixture / Configuration Patterns

### Fixture Responsibilities

Fixtures handle Physical Layer initialization and environment setup. They must NOT contain business logic.

```
// conftest.py — GOOD
fixture api_client():
    """Provides an initialized API client."""
    client = APIClient(base_url=config.API_URL)
    return client

fixture web_driver(playwright_page):
    """Provides an initialized browser driver."""
    driver = PlaywrightDriver(playwright_page)
    return driver
```

### Base Test Class Pattern

Use a base class to wire fixtures into the Action Layer:

```
// action_layer.py
BaseAPITest:
    fixture setup(api_client):
        self.api_client = api_client

    // Actions can now use self.api_client
    create_object_and_verify(url, name, data):
        response = self.api_client.post(url, {name, data})
        assert response.status == 201
        return response.body
```

```
// test_crud.py — Test Layer inherits from Action Layer
TestCRUD(BaseAPITest):
    URL = "https://api.example.com/objects"

    test_create_and_retrieve():
        obj = self.create_object_and_verify(URL, "Widget", {"color": "red"})
        self.get_object_and_verify(URL, obj.id, expected_name="Widget")
```

## Multi-Domain Projects

For projects testing multiple interfaces (API + Web + Mobile), each domain gets its own complete Action + Physical stack. The Test Layer can compose across domains.

```
lib/
├── api/                    # API domain — own Action + Physical stack
├── web/                    # Web domain — own Action + Physical stack
├── mobile/                 # Mobile domain — own Action + Physical stack
└── shared/                 # Cross-domain data helpers (no system calls)
```

File names within each domain follow the same principle: one file (or directory) clearly identifiable as the Action Layer, one as the Physical Layer. The exact names (`actions.py`, `client.py`, `driver.py`, `browser.py`) are up to the team — consistency within a project matters more than following a universal naming scheme.
