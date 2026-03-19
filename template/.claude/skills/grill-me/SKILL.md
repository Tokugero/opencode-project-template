---
name: grill-me
description: Conducts a relentless interview about every aspect of a plan or idea, walking down each branch of the design tree until shared understanding is reached. Invoke with /grill-me before starting any significant design or implementation work.
context: fork
allowed-tools:
  - Read
  - Glob
  - Grep
---

# grill-me

Drives an exhaustive design interview before implementation begins. Surfaces
ambiguities, forces decisions on every branch of the design tree, and resolves
dependencies between decisions one at a time — so that implementation work
starts from a shared understanding rather than unchecked assumptions.

## When to use

Invoke with `/grill-me` before starting any significant feature, architecture
decision, or refactor. Also useful when a plan exists but feels underspecified.

## Inputs

- **Plan or idea** (required): Describe what you want to build or decide.
  Can be vague — the interview will sharpen it.

## Steps

1. Ask the user to describe the plan or idea if not already provided.

2. Interview relentlessly about every aspect of the plan until a shared
   understanding is reached:

   > "Interview me relentlessly about every aspect of this plan until we reach
   > a shared understanding. Walk down each branch of the design tree, resolving
   > dependencies between decisions one by one. And finally, if a question can
   > be answered by exploring the codebase, explore the codebase instead."

3. Work through the design tree systematically:
   - Identify all major decision branches (data model, interfaces, error
     handling, performance, security, rollback, testing strategy, etc.)
   - For each branch, ask one focused question at a time
   - Resolve each answer before moving to the next branch
   - When a question is answerable from the codebase, read the relevant files
     (L0 → L1 → L2) instead of asking

4. After all branches are resolved, produce a concise **Design Summary**:
   - The decision made on each branch
   - Any assumptions that remain (explicitly flagged)
   - Recommended next step (e.g., `/write-a-prd`, implementation task)

## Expected output

A shared understanding of the full design, with all major decision branches
resolved and a written Design Summary the user can reference during implementation.
