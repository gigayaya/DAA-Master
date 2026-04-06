<!-- DAA:BEGIN -->
## E2E Test Architecture

This project uses the **Declarative Action Architecture (DAA)** for E2E test automation — a strict three-layer separation pattern that eliminates false positives and maximizes test maintainability.

```
Test Layer        →  "What" the user does (100% declarative, zero logic)
Action Layer      →  "How" it's done + verification (every action self-verifies)
Physical Layer    →  The mechanism (thin wrapper around system libraries)
```

Dependencies flow strictly downward: **Test → Action → Physical**.

See `docs/daa_rules/` for detailed rules and guidelines. For a deep dive into DAA, see: [Declarative Action Architecture: A Scalable Pattern for E2E Automation](https://medium.com/@gigayaya/declarative-action-architecture-a-scalable-pattern-for-e2e-automation-1f9a10d24ee0)
<!-- DAA:END -->
