---
name: test-design-review
description: Review tests for design quality using test design guidelines.
---

# Test Design Review

When invoked, review the specified tests (or the diff if none specified) against the guidelines in this document.

For each violation found, show the offending code and suggest a fix. Group by guideline.

## Core Principle

Tests are executable specifications. A specification answers: "In scenario X, what should happen?"

## Specification Format

Good: "When the user submits an empty form, display a validation error."
Good: "When the API returns 500, show a graceful error message."
Good: "When no records exist, display 'No results found'."

Bad: "It works correctly." (What does 'correctly' mean?)
Bad: "It handles errors." (Which errors? How?)
Bad: "It validates input." (What validation? What happens on failure?)

## Test Behavior, Not Implementation Details

Bad:
```rust
#[cfg(test)]
mod tests {
    use super::*;

    mod tick {
        use super::*;

        #[test]
        fn marks_the_particles_position_blue() {
            let mut world = World::new(10, 10);

            world.tick();

            assert_eq!(world.color_at(5, 0), 0x0000FF);
        }
    }
}
```

  Good:
```rust
mod when_a_particle_touches_a_grid_cell {
    use super::*;

    #[test]
    fn the_cell_turns_the_particles_color() {
        let mut world = World::new(10, 10);
        let particle_color = world.particle_color();
        let particle_position = world.particle_position();

        world.tick();

        assert_eq!(
            world.color_at(particle_position.0, particle_position.1),
            particle_color
        );
    }
}
```

## When Capturing Scenarios, Describe the Essence

Bad:
```
describe "scope=failed" do
```

Good:
```
describe "rerunning only failed tests" do
```

## Avoid Arbitrariness

### Avoid .first and .last in Tests

Using `.first` or `.last` to retrieve records in tests is fragile because it depends on ordering, which can change unexpectedly. Instead, use explicit queries with `change` and `where`:

Bad:
```ruby
post repositories_path, params: { repo_full_name: "jasonswett/ductwork" }
repository = Repository.last
expect(repository.github_account).to eq(github_account_jasonswett)
```

Good:
```ruby
expect { post repositories_path, params: { repo_full_name: "jasonswett/ductwork" } }
  .to change { Repository.where(github_account: github_account_jasonswett).count }.by(1)
```

## Make Assertions About What's Essential, Not What's Incidental

Only assert what matters. Don't assert things that are:
- Implied by other assertions (if checking response body, don't also check `be_successful` - if it wasn't successful, the body check would fail)
- Implementation details rather than behavior
- Just noise that makes the test longer without adding meaning

Bad:
```ruby
expect(response).to be_successful  # redundant noise
expect(response.body).not_to include("deleted_item")
```

Good:
```ruby
expect(response.body).not_to include("deleted_item")
```

If the response wasn't successful, the body assertion tells you something went wrong. The `be_successful` check adds nothing.

## Don't Mix Levels of Abstraction

Bad:
```ruby
describe "Rerun test suite run", type: :system do
  # ... existing tests ...

  describe "Rerun Failed button" do
    context "when the test suite run has failed tests" do
      let!(:test_suite_run) { create(:test_suite_run, :with_failed_run) }
      let!(:failed_test_case_run) { create(:test_case_run, task: test_suite_run.tasks.first, status: "failed") }

      before do
        allow(ENV).to receive(:fetch).and_call_original
        allow(ENV).to receive(:fetch).with("NOVA_K8S_API_URL").and_return("https://k8s.example.com")
        allow(ENV).to receive(:fetch).with("NOVA_K8S_TOKEN").and_return("test-token")
        allow(ENV).to receive(:fetch).with("NOVA_K8S_CA_CERT").and_return("test-ca-cert")
        allow_any_instance_of(User).to receive(:can_access_repository?).and_return(true)
        login_as(test_suite_run.repository.user)
      end

      it "displays the Rerun Failed button" do
        visit repository_test_suite_run_path(id: test_suite_run.id, repository_id: test_suite_run.repository.id)
        expect(page).to have_button("Rerun Failed")
      end
    end
  end
end
```

Good:
```ruby
describe "Rerun Failed button", type: :system do
  context "when the test suite run has failed tests" do
    let!(:test_suite_run) { create(:test_suite_run, :with_task) }
    let!(:test_case_run) { create(:test_case_run, task: test_suite_run.tasks.first, status: "failed") }

    before do
      login_as(test_suite_run.repository.user)
    end

    it "displays the Rerun Failed button" do
      visit repository_test_suite_run_path(id: test_suite_run.id, repository_id: test_suite_run.repository.id)
      expect(page).to have_button("Rerun Failed")
    end
  end
end
```

## Avoid forward reference

In the below example, `task_id` is referred to before it's "defined".

Bad:
```ruby
describe '#docker_compose_project_name' do
  let!(:executor) { instance_double(Executor, task_id: task_id) }
  let!(:worker) { Worker.new(executor) }

  context 'when task_id is a non-empty string' do
    let!(:task_id) { '123' }

    it 'returns "task-" followed by the task_id' do
      expect(worker.docker_compose_project_name).to eq('task-123')
    end
  end
end
```

Better to do it the other way around.

Better:
```ruby
describe '#docker_compose_project_name' do
  context 'when task_id is a non-empty string' do
    let!(:task_id) { '123' }
    let!(:executor) { instance_double(Executor, task_id: task_id) }
    let!(:worker) { Worker.new(executor) }

    it 'returns "task-" followed by the task_id' do
      expect(worker.docker_compose_project_name).to eq('task-123')
    end
  end
end
```

Even better: just hard-code it.
```ruby
describe '#docker_compose_project_name' do
  context 'when task_id is a non-empty string' do
    let!(:executor) { instance_double(Executor, task_id: '123') }
    let!(:worker) { Worker.new(executor) }

    it 'returns "task-" followed by the task_id' do
      expect(worker.docker_compose_project_name).to eq('task-123')
    end
  end
end
```

## Don't Use have_current_path

`have_current_path` is the wrong level of abstraction -- it's too tightly coupled to the implementation. Instead, assert on what the user sees on the page.

Bad:
```ruby
it "redirects to the repositories page" do
  visit root_path
  expect(page).to have_current_path(repositories_path)
end
```

Good:
```ruby
it "redirects to the repositories page" do
  visit root_path
  expect(page).to have_content("Repositories")
end
```

## Assert on Observable Outcomes, Not Method Calls

When testing whether something happened (or didn't happen), assert on the observable end result rather than on whether a specific method was called. Mock-based assertions like `expect(x).to have_received(:foo)` test means (was this method called?) rather than ends (did the thing actually happen?).

Bad:
```ruby
it "queues the task" do
  worker_pool = instance_double(WorkerPool, queue_task: nil)
  allow(WorkerPool).to receive(:new).and_return(worker_pool)

  QueueUnqueuedTasksJob.new.perform

  expect(worker_pool).to have_received(:queue_task).with(task)
end
```

Good:
```ruby
it "queues the task" do
  expect { QueueUnqueuedTasksJob.new.perform }
    .to change { TaskEvent.where(name: "queued").count }.by(1)
end
```

The bad version tests that a specific method was called on a specific object -- pure implementation. The good version tests the observable outcome: a "queued" event was created. If the implementation changes (different class, different method name), the good test still works as long as the end result is the same.

Stub only what you must (external services like k8s calls), and let the real code run so you can assert on real outcomes.

## Test Ends, Not Means

When testing performance optimizations like caching, don't assert on the caching mechanism (an implementation detail). Assert on the observable difference: fewer database queries.

Bad:
```ruby
it "caches the result" do
  test_suite_run.duration
  expect(Rails.cache.read("test_suite_run/#{test_suite_run.id}/duration")).not_to be_nil
end
```

Good:
```ruby
it "does not query the database on subsequent calls" do
  test_suite_run.duration

  query_count = 0
  callback = ->(*) { query_count += 1 }
  ActiveSupport::Notifications.subscribe("sql.active_record", callback)
  test_suite_run.duration
  ActiveSupport::Notifications.unsubscribe(callback)

  expect(query_count).to eq(0)
end
```

The bad version is coupled to the caching mechanism (`Rails.cache`). If you switch to memoization, a database column, or a different cache store, the test breaks even though the behavior is the same. The good version tests the essential outcome: no redundant database queries.

## Maintain an Appropriately High Level of Abstraction

Bad:
```ruby
context "when called twice" do
  it "queries the database only once" do
    dispatcher = TestSuiteExecution::TestSuiteRunDispatcher.new(cluster_cpu_headroom_millicores: 72000)

    query_count = 0
    callback = lambda { |*, _| query_count += 1 }
    ActiveSupport::Notifications.subscribed(callback, "sql.active_record") do
      dispatcher.test_suite_runs_with_undispatched_tasks
      query_count_after_first_call = query_count
      dispatcher.test_suite_runs_with_undispatched_tasks
      expect(query_count).to eq(query_count_after_first_call)
    end
  end
end
```

What's happening in this test? I'm assaulted with a dense block of incidental
details! The following is much clearer. It hides incidental details and shows
me the essence of the test.

Good:
```ruby
context "when called twice" do                                                                                                               
  it "queries the database only once" do                                                                                                     
    dispatcher = TestSuiteExecution::TestSuiteRunDispatcher.new(cluster_cpu_headroom_millicores: 72000)                                      
                                                                                                                                             
    first_call_count = count_queries { dispatcher.test_suite_runs_with_undispatched_tasks }                                                  
    second_call_count = count_queries { dispatcher.test_suite_runs_with_undispatched_tasks }                                                 
                                                                                                                                               
    expect(second_call_count).to eq(0)                                                                                                       
  end                                                                                                                                        
                                                                                                                                               
  def count_queries(&block)                                                                                                                  
    count = 0                                                                                                                                
    callback = lambda { |*, _| count += 1 }                                                                                                  
    ActiveSupport::Notifications.subscribed(callback, "sql.active_record", &block)                                                           
    count                                                                                                                                    
  end                                                                                                                                        
end
```

Note carefully that the helper method `count_queries` appears *after* the test,
since the helper itself is an incidental detail, not part of the essential
meaning of the test.

## Don't Use Hacks to Test Private Methods

Never use `#send` or `#public_send` to test private methods. If you feel
yourself wanting to test a method directly but you can't because it's private,
just make the method public. It's usually a quite acceptable price to pay.

## Miscellaneous

Never use `instance_variable_set`. In cases where it seems like
`instance_variable_set` is the only option, that's probably a sign of poor
design. In that case you should pause, try to find the poor design, and
suggest a specific refactor.

Don't use `described_class`. It only adds obscurity. Just use the actual class
name.
