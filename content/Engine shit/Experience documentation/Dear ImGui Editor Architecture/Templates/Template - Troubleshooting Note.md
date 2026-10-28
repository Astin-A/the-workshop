---
title: "{{title}}"
tags: [troubleshooting, bug]
status: resolved
---

# {{title}}

> [!bug] I Hit This Issue
> [Personal narrative: When I hit this, what it felt like, how long it took to debug, and the immediate symptom.]

## Symptom
[Neutral, observable description of the crash or bug. E.g., "The window froze for 5 seconds and the OS flagged it as Not Responding."]

## Root Cause
[Technical explanation. E.g., "The main thread was blocked by a `sleep_for(5)` call inside the splash screen loop."]

## Our Fix
[What we changed and why.]

```diff
- old_code_that_caused_bug();
+ new_code_that_fixed_it();
```

## Verification
[How we confirmed it was fixed. E.g., "We ran a 50-project load test and the UI maintained 60 FPS."]

## Relation to Other Systems
- [[Related Note]]