---
name: tdd
description: Implement a change using test-driven development with RSpec. Guides the specify-encode-fulfill workflow.
argument-hint: [specification]
disable-model-invocation: true
---

# Test-Driven Development

## Initial Specification

$ARGUMENTS

## The Specify-Encode-Fulfill Loop

I use something called "specify-encode-fulfill":

1. **Specify**: Come up with the specifications for what you want to build
2. **Encode**: Encode those specifications as automated tests (executable specifications)
3. **Fulfill**: Write the code to fulfill the specifications

At a finer grain:

1. Write a list of the specifications within scope of the current TDD session
2. Encode one item in the list as an automated test
3. Change the code *just barely enough* to *make the current test failure go away*. Avoid "speculative coding" - if we write more code than necessary to make the current test failure go away, we risk having code never exercised by any test
4. Optionally refactor, but not before committing the behavior change. Never mix behavior changes with refactoring
5. Until the list is empty, go back to #2

This follows Kent Beck's [Canon TDD](https://tidyfirst.substack.com/p/canon-tdd).

## Clarifying Specifications

Before writing tests, follow this loop:

1. Repeat my specifications back to me in your own words
2. Ask me to confirm your articulation is correct or explain how it's wrong
3. If confirmed, proceed to writing tests; otherwise use my response and go back to step 1

Specifications should take the form: "under scenario A, X happens; under scenario B, Y happens".

For guidance on designing good specifications, see [test-design.md](test-design.md).

## Translating Specifications into Tests

Each scenario should map to an RSpec _example group_. If the specification is,
for example "when a test suite run's status is 'passed', its label says
'Passed'", then the test should look like this:

```ruby
describe "#label" do
  context "when status is 'passed'" do
    it "returns 'Passed'" do
      test_suite_run = TestSuiteRun.new(status: "passed")
      expect(test_suite_run.label).to eq("Passed")
    end
  end
end
```

Here's an example of a BAD way to write such a test:

```ruby
describe "#label" do
  it "returns the correct value" do
    test_suite_run = TestSuiteRun.new(status: "passed")
    expect(test_suite_run.label).to eq("Passed")
  end
end
```

Of course it returns the "correct" value. What else could we ever want from our
code? Never assert that a behavior "works correctly" or "works properly" or
"handles" certain scenario. What we want to specify in every scenario is _what
the correct behavior is_.

## Workflow

1. I invoke /tdd with a draft specification
2. After back-and-forth, we agree on "final" specifications
3. See if we need to "clean the kitchen before we make dinner". Refer to the
   "Cleaning the Kitchen" section below.
4. You write just one test (per Canon TDD)
5. Show me the test and ask for approval before continuing
6. Write the application code, show it to me, and ask for approval before
   committing. Refer to the "Fulfilling Test Specifications" section below.
7. I provide a new specification and we start over from step 2

### Cleaning the Kitchen

Before you write a test, picture the test you're going to write and where
you're going to put it. Does the conceptual framework of this new behavior
we're about to add slot tidily into the conceptual framework of the area of the
code where we'll be adding it? If not, is there a reconceptualizing of the
current behavior that could be done in order to make the ending result more
conceptually elegant? If such a reconceptualizing is called for, suggest it to
the user. If the user approves, abandon the current change, get to a clean
working state, and, on a new branch, perform a refactoring. "Clean the kitchen
before you make dinner." Then pause and consult the user and we'll begin again.

### Fulfilling Test Specifications

When writing the application code to fulfill a certain specification, write
ONLY ENOUGH CODE to make the current test failure go away. Never use "defensive
coding". Defensive coding is almost always just speculative coding, which means
code that's added without justification or feedback.
