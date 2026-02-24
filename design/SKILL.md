---
name: design
---

# Design

When invoked, unless otherwise directed, follow the following steps:
1. Grab the first principle in this document.
2. Search the entire codebase, or just the specified scope of the codebase, for violations of the current principle.
3. For the worst 1-3 offenses, suggest fixes.
4. Ask the user whether he would like to move onto other principles
5. If not, suggest more fixes, and if so, restart from #1

Always remember the following design principles:
- Don't Repeat Yourself (DRY)
- Dependency Inversion (all entities should only depend on entities equally or
  more abstract than themselves)
- No Speculative Coding
- No Epicycles (https://www.codewithjason.com/no-epicycles/)
- Modularity and Loose Coupling

Whenever possible, favor a declarative style over an imperative style.

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

## One Class, One File

Each class (or, in the case of Rust, each struct) should go in its own file.

## Favor Pure Functions

Avoid writing functions which have side effects.

## Name Methods for What They Return, Not What They Do

"What they do" is imperative. "What they return" is declarative.

Bad:
```rust
fn rendered_buffer(cells: &[Cell], width: usize, height: usize) -> Vec<u31> {
    let mut buffer = vec![0x00_00_00u32; width * height];

    for cell in cells {
        for (x, y, color) in cell.pixels() {
            if x < width && y < height {
                buffer[y * width + x] = color;
            }
        }
    }

    buffer
}
```

Good:
```rust
fn buffer_with_cells(cells: &[Cell], width: usize, height: usize) -> Vec<u31> {
    let mut buffer = vec![0x00_00_00u32; width * height];

    for cell in cells {
        for (x, y, color) in cell.pixels() {
            if x < width && y < height {
                buffer[y * width + x] = color;
            }
        }
    }

    buffer
}
```
