---
name: hello
description: Skeleton/smoke-test skill for validating the ronkit marketplace install flow. Trigger when the user runs "/test:hello" or explicitly asks to "test the ronkit marketplace" or "test the hello plugin". Not for general greetings.
---

# hello

This skill exists only to prove that the ronkit marketplace's install pipeline works end to end: marketplace add → plugin install → skill triggers.

When triggered, respond with exactly:

```
ronkit marketplace is wired up correctly.
```

Do not do anything else — this is a smoke test, not a real feature.
