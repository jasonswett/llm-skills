---
name: design
---

# Design

When invoked, unless otherwise directed, follow the following steps:
1. Unless otherwise specified, set your scope to the diff between the current
   branch and master/main.
2. Grab the first principle in this document, and look for violations of it.
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
- Cohesion

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

## Call Things What They Are

Bad:
```ruby
def self.generate(github_installation_id)
  fail "Installation ID is missing" if github_installation_id.blank?

  retries = 0
  begin
    token(github_installation_id)
  rescue Octokit::InternalServerError
    retries += 1
    retry if retries < 3
    raise
  end
end
```

The number 2 is not a "retry", it's a retry COUNT.

Bad:
```ruby
def self.generate(github_installation_id)
  fail "Installation ID is missing" if github_installation_id.blank?

  attempts = 0
  begin
    token(github_installation_id)
  rescue Octokit::InternalServerError
    attempts += 1
    retry if attempts < 3
    raise
  end
end
```

The number 2 is also not an "attempt" it's an attempt COUNT. "Attempt" is just
a synonym for "retry".

Good:
```ruby
def self.generate(github_installation_id)
  fail "Installation ID is missing" if github_installation_id.blank?

  retry_count = 0
  begin
    token(github_installation_id)
  rescue Octokit::InternalServerError
    retry_count += 1
    retry if retry_count < 3
    raise
  end
end
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

## No Magic Numbers

Bad:
```ruby
def self.generate(github_installation_id)
  fail "Installation ID is missing" if github_installation_id.blank?

  retry_count = 0
  begin
    token(github_installation_id)
  rescue Octokit::InternalServerError
    retry_count += 1
    retry if retry_count < 3
    raise
  end
end
```

Good:
```ruby
RETRY_LIMIT = 3

def self.generate(github_installation_id)
  fail "Installation ID is missing" if github_installation_id.blank?

  retry_count = 0
  begin
    token(github_installation_id)
  rescue Octokit::InternalServerError
    retry_count += 1
    retry if retry_count < RETRY_LIMIT
    raise
  end
end
```
