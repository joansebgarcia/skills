---
"@joansebgarcia/skills": major
---

Rename the `blueprint` skill to `staging`. Invoke it with `/staging` instead of `/blueprint`. Plans are now split into stages, so run them with "execute stage N" instead of "execute phase N".

The skill now writes for an executor who has not seen the conversation, self-reviews the plan before saving it (coverage, empty lines, naming consistency), and requires a `Done when` line on every stage.
