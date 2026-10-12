---
name: to-issues
description: Break a plan, spec, or the current conversation into a set of tracer-bullet issues, each declaring its blocking edges, published to the configured tracker (edges as text in one file per ticket locally, or native blocking links on a real tracker).
disable-model-invocation: true
---

# To Issues

Break a plan, spec, or conversation into a set of **local issues**: tracer-bullet vertical slices, each declaring the blocking dependencies required before it can start.

## Process

### 1. Gather Context
- Work from whatever is already in the conversation context (PRDs, specs, or plans).
- If a reference path to a spec or plan file is provided as an argument, read its full content.

### 2. Explore Codebase
- Inspect the codebase to understand the current implementation state.
- Respect project domain vocabulary (`GLOSSARY.md`) and architectural decision records (`ADR`).
- Identify prefactoring opportunities ("Make the change easy, then make the easy change").

### 3. Draft Vertical Slices
Break the work into **tracer bullet** tickets.

<vertical-slice-rules>
- Each slice cuts a narrow but COMPLETE path through all required layers (data model, business logic, API/interface, verification).
- A completed slice is functional and demoable on its own.
- Each slice fits within a single fresh context window.
- Prefactoring work should be placed in its own initial ticket.
</vertical-slice-rules>

Define **blocking edges** for each ticket: dependencies that must be completed before the ticket can start. Tickets without blockers can start immediately.

**Wide Refactors (Expansion–Contraction):**
For mechanical changes with a broad blast radius across the codebase, sequence as:
1. **Expand:** Add the new interface alongside the old one so existing code remains unbroken.
2. **Migrate:** Update call sites in batches (each batch is a ticket blocked by Expand).
3. **Contract:** Remove the deprecated interface once all call sites are migrated (blocked by all Migrate tickets).

### 4. Quiz the User
Present the proposed breakdown as a numbered list in the chat. For each ticket, show:
- **Title:** Short, descriptive title.
- **Blocked by:** List of blocking tickets (or "None").
- **What it delivers:** End-to-end behavior made functional by this ticket.

Ask the user:
- Is the granularity right?
- Are the blocking dependencies accurate?
- Should any tickets be merged or split further?

Iterate until the user approves the breakdown.

### 5. Publish Local Issues
Once approved, save each ticket as an individual Markdown file in `docs/issues/` named in dependency order (`issue-01-<slug>.md`, `issue-02-<slug>.md`, etc.).

Use the following local issue template:

<local-issue-template>

# Issue <NN>: <Ticket Title>

**What to Build:**
The end-to-end behavior delivered from the user or system perspective, rather than a layer-by-layer technical breakdown.

**Blocked by:**
List of ticket numbers/titles that gate this issue, or "None (can start immediately)".

**Status:** ready-for-agent

**Acceptance Criteria:**
- [ ] Criterion 1
- [ ] Criterion 2

**Verification ("If Available"):**
- **If test suite exists:** Run or write the corresponding test using the project's test runner.
- **If NO test suite exists:** Provide the compilation/execution command and step-by-step manual verification procedure.

</local-issue-template>

Avoid hardcoded file paths or fragile code snippets unless they represent precise design decisions (schemas, types, or state machines) produced during prototyping.
