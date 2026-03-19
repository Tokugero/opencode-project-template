---
name: tdd
description: Implements a feature or fix using strict red-green-refactor TDD cycles. Confirms interfaces and behaviours before writing any code. Invoke with /tdd <task description>.
context: fork
allowed-tools:
  - Read
  - Glob
  - Grep
  - Bash
  - Write
  - Edit
---

# tdd

Drives implementation through test-first discipline: confirm the interface,
confirm the behaviours, write failing tests, make them pass with the minimum
code needed, then refactor. Each cycle is explicit — no code is written before
a failing test justifies it.

## When to use

Invoke with `/tdd` when:
- Implementing a new function, method, or module
- Fixing a bug (write a test that reproduces it first)
- Refactoring code that lacks test coverage (add tests before touching logic)

## Inputs

- **Task description** (required): What behaviour needs to be implemented or fixed.
- **Relevant files or modules** (optional): If known, provide paths; otherwise
  the skill will explore the codebase.

## Steps

### 1. Confirm interface changes needed

Load L0 (`.abstract.md`) and L1 (`.overview.md`). Read the relevant L2 source
files. Identify:
- The function/method/class signature that will change or be created
- Any types, schemas, or contracts it must conform to
- What callers or dependents exist

Ask the user to confirm the interface before continuing. Do not proceed until
the interface is agreed.

### 2. Confirm behaviours to test

List the behaviours the implementation must satisfy as concrete, verifiable
statements:

```
Behaviours:
1. Returns X when given input Y
2. Raises <Error> when Z is missing
3. Calls <dependency> with arguments A, B
4. Does NOT call <dependency> when condition C is false
```

Ask the user to confirm, add, or remove behaviours before writing any code.

### 3. Design testable interfaces

If the current design makes the behaviours hard to test (e.g., hidden
dependencies, no injection points), propose the minimum interface adjustment
needed. Confirm with the user before proceeding.

### 4. Red — write failing tests

For each confirmed behaviour, write one test that:
- Is named clearly after the behaviour it tests
- Fails for the right reason (not a syntax error or missing import)
- Tests only one behaviour per test case

Run the test suite to confirm all new tests fail:
```bash
<test command for this project>
```

Do not write any implementation code until all new tests are failing.

### 5. Green — write minimum implementation

Write the smallest implementation that makes all failing tests pass.
Do not add logic not covered by a test.

Run the test suite again to confirm all tests pass:
```bash
<test command for this project>
```

If any pre-existing tests break, fix them before continuing.

### 6. Refactor

With all tests green, examine the code for:
- Duplication that can be eliminated
- Names that don't reflect intent
- Abstractions that are missing or wrong
- Complexity that can be simplified

Make refactoring changes in small steps, running tests after each step to
confirm nothing broke. Do not add new behaviour during refactor.

### 7. Repeat

If additional behaviours remain unimplemented, return to Step 4 and run
another red-green-refactor cycle.

## Expected output

- All confirmed behaviours covered by passing tests
- No implementation code that isn't exercised by a test
- A clean, refactored implementation with the test suite green
- A brief summary of cycles run and behaviours covered
