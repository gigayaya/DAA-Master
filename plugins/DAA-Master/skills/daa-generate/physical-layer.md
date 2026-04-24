# Generating Physical Layer Code

## Purpose

The Physical Layer is a **thin, "dumb" wrapper** around the underlying system interaction library (HTTP client, WebDriver, Playwright, database driver, etc.). It knows HOW to talk to the system but knows NOTHING about business logic or expected outcomes.

## Structure Pattern

```
PhysicalLayerClass:
    driver = underlying_library_instance

    operation(params):
        return driver.operation(params)
```

## Rules

### R1: Pure Execution — Zero Intelligence

The Physical Layer must contain:
- **NO assertions** (`assert`, `expect`, `should`, etc.)
- **NO conditionals** (`if`, `switch`, ternary)
- **NO loops** (`for`, `while`)
- **NO error handling** (`try/catch`) — let errors propagate to Action Layer
- **NO business logic** (no data transformation, no decision-making)
- **NO retry logic** — retries are a business decision for the Action Layer

### R2: Thin Wrapping

Each method should be a direct, 1-2 line delegation to the underlying library. The method names below mirror the underlying library's API — your own names should reflect your library's vocabulary and your team's conventions, not copy these examples verbatim.

```
// API Physical Layer — wrapping an HTTP client (e.g. requests, httpx, axios)
APIClient:
    http = requests_library

    get(url, params=None, headers=None):
        return http.get(url, params=params, headers=headers)

    post(url, data=None, headers=None):
        return http.post(url, json=data, headers=headers)

    put(url, data=None, headers=None):
        return http.put(url, json=data, headers=headers)

    delete(url, headers=None):
        return http.delete(url, headers=headers)
```

```
// Web Physical Layer — wrapping a browser driver (e.g. Playwright, Selenium, Cypress)
// Method names here match Playwright's API; adapt to your driver's semantics
BrowserDriver:
    page = browser_page_instance

    navigate(url):              // Playwright calls this goto(); Selenium uses get()
        page.goto(url)

    type_into(selector, text):  // Could also be fill(), send_keys(), setValue()
        page.fill(selector, text)

    click(selector):
        page.click(selector)

    read_text(selector):        // Could also be get_text(), text_content(), innerText()
        return page.text_content(selector)

    element_visible(selector):
        return page.is_visible(selector)

    count_elements(selector):
        return page.locator(selector).count()

    wait_for(selector):
        page.wait_for_selector(selector)
```

### R3: One Method Per Primitive

Each Physical Layer method wraps exactly one primitive operation. Don't combine multiple operations:

```
// BAD — multiple operations in one method
type_and_submit(selector, text, submit_button):
    page.fill(selector, text)
    page.click(submit_button)    // This is a second operation

// GOOD — one operation per method
type_into(selector, text):
    page.fill(selector, text)

click(selector):
    page.click(selector)
```

### R4: Technology Swappability

The Physical Layer is the ONLY place where the underlying technology is referenced. This means:

- Switching from Selenium to Playwright? Only Physical Layer changes.
- Switching from `requests` to `httpx`? Only Physical Layer changes.
- Action Layer and Test Layer remain untouched.

Design your Physical Layer's interface to be technology-agnostic — the Action Layer should not need to know which library is underneath. When the two libraries have different method names for the same primitive, your wrapper is what normalizes them:

```
// Two implementations exposing the same interface to the Action Layer
// The wrapper name (type_into) is your team's choice — the underlying call differs

SeleniumDriver:
    type_into(selector, text):
        self.driver.find_element(By.CSS_SELECTOR, selector).send_keys(text)

PlaywrightDriver:
    type_into(selector, text):
        self.page.fill(selector, text)
```

## Constants / Selectors

UI selectors and URLs should be extracted into a constants file, separate from the Physical Layer:

```
// Constants file — centralized selector management
Selectors:
    BASE_URL = "https://example.com"
    SEARCH_INPUT = "#twotabsearchtextbox"
    SEARCH_BUTTON = "#nav-search-submit-button"
    RESULT_ITEM = "[data-component-type='s-search-result']"
    ADD_TO_CART = "#add-to-cart-button"
    CART_COUNT = "#nav-cart-count"
```

This ensures that when a UI element's selector changes, you update ONE file — not scattered references across the Physical Layer.

## Complete Example

These examples use Python + requests/Playwright naming conventions. The method names here (`goto`, `fill`, `click`) closely mirror Playwright's own API — in your project, use names that make sense for your library and team.

```
// Physical Layer — API (Python + requests)
APIClient:
    """Pure execution wrapper for HTTP operations."""

    __init__():
        self.http = requests_library

    get(url, params=None, headers=None):
        return self.http.get(url, params=params, headers=headers)

    post(url, data=None, headers=None):
        return self.http.post(url, json=data, headers=headers)

    delete(url, headers=None):
        return self.http.delete(url, headers=headers)


// Physical Layer — Web (Playwright)
BrowserDriver:
    """Pure execution wrapper for browser operations."""

    __init__(page):
        self.page = page

    navigate(url):
        self.page.goto(url)

    type_into(selector, text):
        self.page.fill(selector, text)

    click(selector):
        self.page.click(selector)

    read_text(selector):
        return self.page.text_content(selector)

    element_visible(selector):
        return self.page.is_visible(selector)

    count_elements(selector):
        return self.page.locator(selector).count()

    wait_for(selector):
        self.page.wait_for_selector(selector)
```

Notice: every method is a 1-line delegation. No logic, no assertions, no decisions.
