---
name: performance-optimize
description: >
  Diagnose and fix performance bottlenecks using empirical profiling data
  (New Relic, query logs, flame graphs) before applying optimizations.
  Covers database query tuning, N+1 detection, endpoint latency reduction,
  memory profiling, and caching strategies.

  Use when: the app is slow, an endpoint has high latency, a page takes too
  long to load, queries are slow, memory usage is high, or you need to
  optimize a specific area of the codebase.
---

# Performance Optimization

Performance problems are defects. As with all defects, *diagnose before prescribe*.

## Workflow

1. **Gather data.** Check New Relic (API token in `saturnci/.env`) for transaction traces, slow queries, and throughput metrics. If instrumentation is missing, add it first.
2. **Identify the bottleneck.** Use empirical data (not heuristics) to find the slowest component: database, application code, external API, or rendering.
3. **Reproduce locally.** Write a benchmark or use a profiler to confirm the bottleneck in development.
4. **Fix the root cause.** Apply the smallest change that addresses the bottleneck (query optimization, caching, eager loading, pagination, etc.).
5. **Verify the improvement.** Compare before/after metrics. Confirm no regressions in other areas.

## Common Bottlenecks

| Symptom | Likely cause | Fix |
|---|---|---|
| Endpoint slow on list pages | N+1 queries | Add eager loading (`.includes` / `.preload`) |
| Single query > 500ms | Missing index or full table scan | Add index, rewrite query |
| High memory on large datasets | Loading all records into memory | Use pagination or batching |
| Slow external API calls | Synchronous blocking calls | Move to background job or add caching |

## Example: N+1 Query Fix

```ruby
# BAD: N+1 — one query per task
tasks.each { |t| t.user_name = User.find(t.user_id).name }

# GOOD: eager load in one query
users = User.where(id: tasks.map(&:user_id)).index_by(&:id)
tasks.each { |t| t.user_name = users[t.user_id].name }
```

## Profiling Commands

```bash
# Find slow queries in PostgreSQL
SELECT query, mean_exec_time, calls FROM pg_stat_statements ORDER BY mean_exec_time DESC LIMIT 10;

# Profile a Rails request
RACK_MINI_PROFILER=1 rails server   # then visit /?pp=flamegraph
```

## Error Recovery

- If New Relic data is insufficient, add custom instrumentation before optimizing.
- If a fix improves one metric but degrades another, revert and find a more targeted approach.
- If the bottleneck is unclear, add logging/tracing to narrow down the hot path before making changes.
