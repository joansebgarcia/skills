---
name: staging
description: "Synthesize the conversation into a staged implementation specification. Use when the user wants to plan, spec, or break a feature into stages for execution."
disable-model-invocation: true
---

# Staging

## Summary

Generates a structured, staged implementation specification from the current conversation. This document combines high-level product vision with tactical, actionable execution stages (using checkboxes) so an agent or developer can implement it step-by-step.

Write for an engineer or agent who has not seen this conversation. Stages are often executed in a fresh session or after context has been compacted, so capture every decision made in the discussion; the executor cannot recover what isn't written.

## When to Use

- The user asks to create a spec, plan or prd based on the current discussion.
- You have enough context about the problem and solution to map out how it should be built.
- The user wants a step-by-step staged plan to execute.

## Workflow Steps

1. **Synthesize the Context:** Gather all the information discussed. Do NOT interview the user unless strictly necessary to unblock the plan; synthesize what you already know.

2. **Determine File Location:**

Before writing the staging plan, actively explore the workspace/repo to find where specs or plans are usually stored.

- Look for directories like `docs/`, `docs/specs`, `prd`, `docs/plans`, or `plans/`.
- If you find an existing convention, use it. The filename should be `YYYY-MM-DD-<feature-name>.md`.
- **If and only if** you cannot find any logical place to save the file, ask the user where they want to store it before proceeding.

3. **Generate the Staging Plan:** Write the staging plan using the exact template below.

4. **Self-Review:** Before saving, check the staging plan against the conversation and fix any issue inline:
   - **Coverage:** Every user story is implemented by at least one stage. Add a stage or step for any gap.
   - **Empty lines:** No step says something that decides nothing ("TBD", "handle edge cases", "add appropriate validation", "write tests"). Replace each with the concrete decision.
   - **Consistency:** Entities, endpoints, modules, and terms use the same names across every section and stage.
   - **Done criteria:** Every stage has a `Done when` line that is observable or runnable.

5. **Save the Document:** Save the generated markdown into the identified file path. Once saved, inform the user that the staging plan is ready and they can now instruct you to "execute stage 1".

<staging-template>

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

## Stages

Break down the execution into logical stages. Each stage must be a list of actionable steps using markdown checkboxes (`- [ ]`).
_Stages represent chunks of work that can be executed and verified sequentially._

Each stage ends with a `Done when` line: a command to run or a behavior to observe that proves the stage works on its own. The executor must not start the next stage until it holds.

### Stage 1: [Name of Stage]

- [ ] Step 1 description
- [ ] Step 2 description

**Done when:** [Observable result]

### Stage 2: [Name of Stage]

- [ ] Step 1 description

**Done when:** [Observable result]

## Testing Decisions

- What makes a good test for this feature.
- Which modules will be tested.
- The highest seams where the tests will run.

## Out of Scope

A description of the things that are out of scope for this spec.

## Further Notes

Any additional behavior changes, risk assessments, or notes.

</staging-template>

## Gotchas

- Make sure the `Stages` section uses standard markdown checkboxes (`- [ ]`) so the user can easily prompt an agent to "execute stage X" and track progress.
- A stage without a `Done when` line is unfinished: the executor has no way to know when to move on.
- Do not output a giant block of code. Keep Implementation Decisions at the architecture/design level.
- **Crucial:** Always check the workspace for an existing `docs/plans` or `docs/specs` directory to determine where to save the file automatically before asking the user.
