---
name: handshake
description: Generates a protocol-agnostic integration report or handoff spec for a third party based on a new implementation. Use when asked to document usage, contracts, interfaces, risks, or integration impact of a technical change.
---

# Handshake

Generates a protocol-agnostic technical handoff document designed exclusively for consumption by a third party. Omit internal architecture details and focus strictly on the usage contract, behavior, and operational risks.

## Triggers

Activate this skill when the user:

- Requests an integration report, handoff, or technical specification for a third party.
- Asks to document the usage, contract, or interface of a new implementation.
- Asks to evaluate the impact matrix or integration risks of a technical change.

## Execution Rules

1. **Input Analysis**
   - Extract context strictly from the document, code, or message provided by the user.
   - If the user provides no context, code, or reference material, DO NOT invent information or assume details; explicitly ask for clarification or the required material before proceeding.
   - Identify the interface or integration surface.

2. **Mandatory Document Structure**
   The generated document MUST include the following the sections:

   - **1. Contract & Interface:** Define inputs specifying required/optional fields, expected outputs, and an error dictionary with recommended actions.
   - **2. Behavior & Edge Cases:** Document behavior under exceptional scenarios (malformed or null data, concurrency, timeouts, rate limits, and retry policies).
   - **3. Impact & Risk Matrix:** Evaluate backward compatibility (breaking changes), cascading failure risks, and performance or caching considerations.
   - **4. Quick Adoption Guide:** Include minimal, copy-pasteable examples (snippets, cURL commands, sample JSON) ready for immediate testing.

3. **Content & Security Restrictions**
   - Redact any sensitive information, such as API keys, passwords, credentials, tokens, or personally identifiable information (PII).
   - Hide internal implementation details that do not directly affect the consumer.
   - Maintain an agnostic, neutral, and clear technical tone.

Save the generated document to the temporary directory of the user's OS (`$TMPDIR`, else `/tmp`; `%TEMP%` on Windows) rather than the current workspace to avoid project clutter.
