---
name: separating-pure-from-io
description: Use when writing code that mixes computation with I/O (database, API, files, logging) - separates pure deterministic logic from side effects for testability, clarity, and maintainability
---

# Separating Pure from I/O

## Overview

**Separate pure logic from I/O. Always.**

Code becomes dramatically easier to test and understand when deterministic calculations are isolated from external operations. The core does math; the shell does I/O.

**Core principle:** If a function touches the outside world (files, network, database, logs), it cannot also contain business logic.

## When to Use

Use when writing ANY code that involves both:
- Computation (calculations, transformations, validation)
- I/O (database, API calls, file operations, logging, notifications)

**This includes:**
- "Utility scripts"
- "Quick fixes"
- "Minimal changes"
- One-off data processing

**No exceptions for simplicity or time pressure.**

## The Rule

Every function is either:

1. **Pure** - Takes data in, returns data out. No I/O. No side effects. Deterministic.
2. **Impure** - Performs I/O. Contains no business logic. Thin wrapper.

The **shell** orchestrates: calls impure functions to get data, passes to pure functions, calls impure functions to store results.

## Quick Reference

| Function Type | Does | Doesn't |
|---------------|------|---------|
| **Pure (Core)** | Transform data, calculate, validate | Touch files, call APIs, query DB, log |
| **Impure (I/O)** | Read/write external systems | Calculate, validate, transform |
| **Shell** | Orchestrate pure + impure | Contain logic OR I/O directly |

## Pattern

```python
# PURE: Business logic - no I/O
def calculate_nav_delta(previous: Decimal, current: Decimal) -> tuple[Decimal, Decimal]:
    delta = current - previous
    delta_pct = (delta / previous * 100) if previous else Decimal(0)
    return delta, delta_pct

def should_alert(delta_pct: Decimal, threshold: Decimal = Decimal(5)) -> bool:
    return abs(delta_pct) > threshold

# IMPURE: I/O wrappers - no logic
def fetch_previous_nav(db, fund_id: str) -> Decimal:
    row = db.execute("SELECT nav FROM nav_history WHERE fund_id = ? ORDER BY date DESC LIMIT 1", [fund_id])
    return Decimal(row[0]) if row else Decimal(0)

def fetch_positions(api, fund_id: str) -> list[dict]:
    return api.get(f"/funds/{fund_id}/positions").json()

def save_nav(db, fund_id: str, nav: Decimal) -> None:
    db.execute("INSERT INTO nav_history (fund_id, nav) VALUES (?, ?)", [fund_id, nav])

def send_alert(slack, message: str) -> None:
    slack.post(message)

# SHELL: Orchestration - no logic, no direct I/O
def reconcile_nav(fund_id: str, db, api, slack) -> dict:
    # Get data (impure)
    previous = fetch_previous_nav(db, fund_id)
    positions = fetch_positions(api, fund_id)

    # Calculate (pure)
    current = sum(Decimal(p["quantity"]) * Decimal(p["price"]) for p in positions)
    delta, delta_pct = calculate_nav_delta(previous, current)

    # Act on results (impure)
    save_nav(db, fund_id, current)
    if should_alert(delta_pct):
        send_alert(slack, f"NAV changed {delta_pct}%")

    return {"previous": previous, "current": current, "delta": delta}
```

## Common Mistakes

**Mistake: "Classes = separation"**
```python
# WRONG: Class doesn't mean separation
class MetricsCalculator:
    def __init__(self, db):
        self.db = db  # I/O injected into "calculator"

    def calculate(self, fund_id):
        data = self.db.query(...)  # I/O in calculator
        return sum(data)  # Logic mixed with I/O
```

**Mistake: "Logging is harmless"**
```python
# WRONG: Logging is I/O
def calculate_fee(amount, rate):
    fee = amount * rate
    logger.info(f"Calculated fee: {fee}")  # Side effect in pure function
    return fee
```

**Mistake: "It's just a utility script"**
Utility scripts need this separation MORE, not less. They're often the hardest to debug.

## Red Flags - Refactor Immediately

- Function has both `db.query()` AND arithmetic
- Function has both `api.get()` AND validation logic
- Function logs AND returns calculated values
- "Just a quick script" with 50+ lines mixing everything
- Calculation functions that need mocks to test

## Rationalizations That Mean You're About to Fail

| Excuse | Reality |
|--------|---------|
| "It's just a utility script" | Utility scripts need separation MORE - they're hardest to debug |
| "No need for abstractions" | Pure functions aren't abstractions - they're just functions without I/O |
| "Keep it simple" | Mixed I/O is complex. Separation IS simplicity. |
| "Minimal changes requested" | Extract pure logic first, then make minimal change to pure function |
| "Not production code" | Non-production code becomes production code. Do it right. |
| "One function is simpler" | One function doing I/O + logic is untestable. That's not simple. |

## Applying to Bug Fixes

When fixing a bug in mixed code:

1. **Extract** the pure calculation that has the bug
2. **Write test** for extracted pure function showing bug
3. **Fix** the pure function
4. **Verify** test passes
5. Leave I/O wrappers and shell untouched

Don't fix bugs by adding more logic to mixed functions.

## The Test

**Can you test your business logic with zero mocks?**

- Yes → You have a functional core
- No → Extract pure functions until you can
