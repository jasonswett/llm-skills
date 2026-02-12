---
description: Iterative refactoring loop
allowed-tools: Read, Bash, Grep, Edit, AskUserQuestion
---

# Refactor: $ARGUMENTS

You guide the user through iterative refactoring. Each iteration follows a strict loop: identify a problem, get approval, propose a solution, implement if accepted.

## Before Starting

Read the relevant code. Run the existing tests to confirm they pass. If there are no tests covering the code, warn the user before proceeding.

## The Loop

Repeat until the user says to stop or no more problems are found:

### 1. Identify a Problem

Name a single, specific problem from the catalog below. Be concrete.

Good: "The `process_order` method is 45 lines long and handles validation, pricing, and email sending."
Bad: "This code could be cleaner."

### 2. Ask Permission

Ask the user: "Should I address this?" Wait for their answer. If no, move to the next problem.

### 3. Propose a Solution

Describe the specific change you'd make. Name the refactoring technique (extract method, rename, simplify conditional, introduce parameter object, etc.). Show what the result would look like.

### 4. Implement

Make the change. Run the tests. If tests fail, revert and explain what went wrong. If tests pass, show the user what changed and move to the next iteration.

## What to Look For

Work through the code looking for these problems, roughly in priority order:

### Speculative code
Code that exists for a future that hasn't arrived. YAGNI violations. Feature flags nobody toggles. Configuration for cases that don't exist. Unused parameters, dead branches, commented-out code. If it's not serving a current need, it's waste.

### Duplication
Two or more places that express the same idea. Not just identical lines — similar structures with minor variations count. If changing one requires remembering to change the other, it's duplication.

### Unclear naming
Names that don't communicate intent. Single-letter variables outside tiny blocks. Abbreviations. Generic names like `data`, `info`, `temp`, `result`, `item`. A name should tell you what something IS, not require you to read the implementation.

### Imperative code that should be declarative
Code that spells out step-by-step HOW to do something when it could instead declare WHAT it wants. Manual loops building up arrays instead of `map`/`select`/`reject`. Hand-rolled lookup logic instead of `find`/`index_by`. Multi-step conditionals assembling a value instead of a lookup table or pattern match. Declarative code is shorter, harder to get wrong, and communicates intent instead of mechanism.

## Rules

- One problem per iteration. Don't batch changes.
- Run tests after every change.
- Never change behavior. If a change would alter what the code does, it's not a refactoring.
- Keep changes small. A single extract-method is better than a full rewrite.
- Stop and warn if you notice missing test coverage for the code you're about to change.
