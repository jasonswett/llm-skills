---
name: design
---

# Design

Always remember the following design principles:
- Don't Repeat Yourself (DRY)
- Modularity
- Loose Coupling
- Dependency Inversion (all entities should only depend on entities equally or
  more abstract than themselves)
- No Speculative Coding
- No Epicycles (https://www.codewithjason.com/no-epicycles/)

## Avoid Abbreviation

Bad:
```ruby
usr = User.first
```

Good:
```ruby
user = User.first
```

Exceptions are abbreviations that are already part of everyone's vocabulary,
such as SSN or URL.

## Be Strictly Consistent with Naming

Bad:
```ruby
last_run = @repository.test_suite_runs.first
```

Is it a "run" or is it a "test suite run"?

Good:
```ruby
last_test_suite_run = @repository.test_suite_runs.first
```

Bad:
```rust
let mut history = SnapshotHistory::new();
```

 Good:
```rust
let mut snapshot_history = SnapshotHistory::new();
```
