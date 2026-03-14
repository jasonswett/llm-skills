---
name: design
---

# Design

**IMPORTANT:** When this skill is invoked, ALWAYS delegate the entire review to a sub-agent using the Agent tool. Pass the full content of this skill file as the prompt so the agent has all the design principles. This ensures the review runs with a fresh context, uninfluenced by the current conversation.

When invoked, unless otherwise directed, follow the following steps:
1. Unless otherwise specified, set your scope to the diff between the current
   branch and master/main.
2. Grab the first principle in this document, and look for violations of it.
3. For the worst 1-3 offenses, suggest fixes.
4. Ask the user whether he would like to move onto other principles
5. If not, suggest more fixes, and if so, restart from #1

Always remember the following design principles:
- No Hacks, No Workarounds
- Don't Be Stupid, Lazy or Dishonest
- Don't Repeat Yourself (DRY)
- Dependency Inversion (all entities should only depend on entities equally or
  more abstract than themselves)
- No Epicycles (https://www.codewithjason.com/no-epicycles/)
- No Broken Windows
- Modularity and Loose Coupling
- Cohesion

Whenever possible, favor a declarative style over an imperative style.

## No Speculative Coding

Don't write application code which is not strictly needed in order to satisfy
an existing test.

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

## Dependency Inversion Principle

Bad:
```ruby
class ApplicationController < ActionController::Base
  def update_last_visited_page
    repository_id = @repository&.id || params[:repository_id]
    UpdateLastVisitedPageJob.perform_later(current_user.id, request.path, repository_id: repository_id)
  end
end
```

In the above example, the repository ID is only present and relevant SOME of
the time. Whenever someone looks at
`ApplicationController#update_last_visited_page`, they have to pay the
cognitive price not just of what `update_last_visited_page` does in general,
but also for some incidental side detail that's only relevant an extreme
minority of the time.

Good:
```ruby
class ApplicationController < ActionController::Base
  def update_last_visited_page
    UpdateLastVisitedPageJob.perform_later(current_user.id, request.path)
  end
end

class RepositoriesController < ApplicationController
  def update_last_visited_repository
    user_preference = UserPreference.find_or_initialize_by(user_id: current_user.id)
    user_preference.update!(last_visited_repository: @repository)
  end
end
```

The second version is better because we only see the repository-updating code
when we actually care about it.

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

## No Premature Optimization

Don't assign a contrivantly-named temp var just to avoid calling a method
multiple times.

Bad:
```ruby
to_dispatch = dispatchable_test_suite_runs(cluster: cluster)
to_dispatch.each do
  ...
end
```

Good:
```ruby
dispatchable_test_suite_runs(cluster: cluster).each do
  ...
end
```
