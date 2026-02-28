---
name: performance
description: Identify, compare and evaluate performance improvement opportunities before implementing any changes.
argument-hint: [area or endpoint to optimize]
disable-model-invocation: true
---

# Performance Optimization

## Target

$ARGUMENTS

## Approach

Do not jump straight to implementing a fix. The goal is to build a complete
picture of where time is spent, then compare all improvement opportunities
against each other before choosing what to work on.

## Steps

### 1. Clarify the goal

Restate the performance optimization goal back to the user in your own words.
Ask for confirmation. If the user corrects your understanding, restate again.
Repeat until the user confirms you have it right. Do not proceed until this
loop completes.

### 2. Measure the baseline

Get a clear, reproducible measurement of current performance. Record P50 and
P95 duration, DB call count, and sample size. Note the time window so it can
be compared later.

### 2. Break down where time is spent

Profile the request to understand its components. For each component (query,
template render, association load, etc.), determine how much time it consumes.
Use whatever tools are available: New Relic spans, rack-mini-profiler, Rails
logs, or custom instrumentation.

### 4. Build an opportunity table

Enumerate every possible improvement. For each one, estimate:

| Opportunity | Est. time saved | Complexity | Worth it? |
|-------------|----------------|------------|-----------|
| ...         | ...            | ...        | ...       |

- **Est. time saved**: The ceiling. If a component takes 20ms, the max savings
  is 20ms. Be honest about what's achievable.
- **Complexity**: How invasive is the change? A one-line fix is low. Restructuring
  multiple files or adding new abstractions is high.
- **Worth it?**: Best ratio of time saved to complexity added.

### 5. Present the table and discuss

Show the table to the user. Don't start implementing until we agree on which
opportunity (or opportunities) to pursue. The table makes trade-offs visible
so we can make a deliberate choice rather than chasing the first idea that
comes to mind.

### 6. Implement, measure, repeat

After implementing a change, measure again using the same method as the
baseline. Update the table with actual results. Then decide whether to
pursue the next opportunity or stop.
