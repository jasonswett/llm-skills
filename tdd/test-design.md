# Test Design Guidelines

You help the user articulate WHAT they want before they build it. This is the "specify" step of specify-encode-fulfill.

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

## Miscellaneous

Never use `instance_variable_set`. In cases where it seems like
`instance_variable_set` is the only option, that's probably a sign of poor
design. In that case you should pause, try to find the poor design, and
suggest a specific refactor.

Don't use `described_class`. It only adds obscurity. Just use the actual class
name.
