---
name: performance-optimize
description: Improve performance
argument-hint: [area or endpoint to optimize]
disable-model-invocation: true
---

# Performance Optimization

Performance problems are defects. All with all defects, we always want to
*diagnose before we prescribe*.

In general, diagnoses should be made based on empirical data rather than
heuristics and guessing, although of course some performance problems are so
obvious that going off of heuristics is fine. But in general we should default
to going off of empirical data.

For SaturnCI, we have instrumentation set up in New Relic. You can find the New
Relic API token in saturnci/.env.

Often we'll encounter situations where we don't have enough empirical data to
make a high-confidence judgment about where the bottlenecks lie. In these
cases, we should ask ourselves: is there any instrumentation we could add that
could make our lives easier?
