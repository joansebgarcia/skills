---
name: blueprint
description: "Synthesize the conversation into a phased implementation specification (blueprint). Use when the user wants to plan, spec, or phase out a feature for execution."
disable-model-invocation: true
---

# Blueprint

## Summary

Generates a structured, phased implementation specification (a "blueprint") from the current conversation. This document combines high-level product vision with tactical, actionable execution phases (using checkboxes) so an agent or developer can implement it step-by-step.

## When to Use

- The user asks to create a spec, plan, or blueprint based on the current discussion.
- You have enough context about the problem and solution to map out how it should be built.
- The user wants a step-by-step phased plan to execute.

## Workflow Steps

1. **Synthesize the Context:** Gather all the information discussed. Do NOT interview the user unless strictly necessary to unblock the plan; synthesize what you already know.

2. **Determine File Location:**

Before writing the blueprint, actively explore the workspace/repo to find where specs or plans are usually stored.

- Look for directories like `docs/`, `docs/specs`, `prd`, `docs/blueprints`, or `plans/`.
- If you find an existing convention, use it. The filename should be `YYYY-MM-DD-<feature-name>.md`.
- **If and only if** you cannot find any logical place to save the file, ask the user where they want to store it before proceeding.

3. **Generate the Blueprint:** Write the blueprint using the exact template below.

4. **Save the Document:** Save the generated markdown into the identified file path. Once saved, inform the user that the blueprint is ready and they can now instruct you to "execute phase 1".

<spec-template>

## Problem Statement

The problem that the user is facing, from the user's perspective.

## Solution

The solution to the problem, from the user's perspective.

## User Stories

A numbered list of user stories covering all aspects of the feature:

1. As an <actor>, I want a <feature>, so that <benefit>

<user-story-example>
1. As a mobile bank customer, I want to see balance on my accounts, so that I can make better informed decisions about my spending
</user-story-example>

This list of user stories should be extremely extensive and cover all aspects of the feature.

## Implementation Decisions

List architectural decisions, API contracts, schema changes, and modified interfaces.

- DO NOT include specific code snippets or literal file paths unless absolutely necessary (e.g., schemas).
- Keep it focused on the "how" at a structural level.

## Phases

Break down the execution into logical phases. Each phase must be a list of actionable steps using markdown checkboxes (`- [ ]`).
_Phases represent chunks of work that can be executed and verified sequentially._

### Phase 1: [Name of Phase]

- [ ] Step 1 description
- [ ] Step 2 description

### Phase 2: [Name of Phase]

- [ ] Step 1 description

## Testing Decisions

- What makes a good test for this feature.
- Which modules will be tested.
- The highest seams where the tests will run.

## Out of Scope

A description of the things that are out of scope for this spec.

## Further Notes

Any additional behavior changes, risk assessments, or notes.

</spec-template>

## Gotchas

- Make sure the `Phases` section uses standard markdown checkboxes (`- [ ]`) so the user can easily prompt an agent to "execute phase X" and track progress.
- Do not output a giant block of code. Keep Implementation Decisions at the architecture/design level.
- **Crucial:** Always check the workspace for an existing `docs/plans` or `docs/specs` directory to determine where to save the file automatically before asking the user.
