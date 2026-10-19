# @joansebgarcia/skills

## 1.1.0

### Minor Changes

- [#13](https://github.com/joansebgarcia/skills/pull/13) [`1d7542f`](https://github.com/joansebgarcia/skills/commit/1d7542f21dccc358d99c9dc755aac86a9c364947) Thanks [@joansebgarcia](https://github.com/joansebgarcia)! - Deprecate the `staging` and `handshake` skills by moving them to `skills/deprecated/`.

## 1.0.1

### Patch Changes

- [#11](https://github.com/joansebgarcia/skills/pull/11) [`9edae66`](https://github.com/joansebgarcia/skills/commit/9edae6616bd8c004f1da91c182ef3c34d4370853) Thanks [@joansebgarcia](https://github.com/joansebgarcia)! - `handshake` now names the generated document `YYYY-MM-DD-handshake-<feature-name>.md` and follows an existing file location convention when one is found.

## 1.0.0

### Major Changes

- [#9](https://github.com/joansebgarcia/skills/pull/9) [`ead55c5`](https://github.com/joansebgarcia/skills/commit/ead55c531795906e9fbe484525165406b2bf8ed1) Thanks [@joansebgarcia](https://github.com/joansebgarcia)! - Rename the `blueprint` skill to `staging`. Invoke it with `/staging` instead of `/blueprint`. Plans are now split into stages, so run them with "execute stage N" instead of "execute phase N".

  The skill now writes for an executor who has not seen the conversation, self-reviews the plan before saving it (coverage, empty lines, naming consistency), and requires a `Done when` line on every stage.

## 0.3.0

### Minor Changes

- [#7](https://github.com/joansebgarcia/skills/pull/7) [`85ec0a1`](https://github.com/joansebgarcia/skills/commit/85ec0a1c3bdc992390109e042fbadfcc41a9ce32) Thanks [@joansebgarcia](https://github.com/joansebgarcia)! - Add the `handshake` skill: generates a protocol-agnostic integration report or handoff spec for a third party, focused on the usage contract, behavior, and operational risks of a new implementation.

## 0.2.0

### Minor Changes

- [#4](https://github.com/joansebgarcia/skills/pull/4) [`45ac29c`](https://github.com/joansebgarcia/skills/commit/45ac29c7bc96bee2dc03746f2ef2db15c8bb4a38) Thanks [@joansebgarcia](https://github.com/joansebgarcia)! - Añade la skill `blueprint`: sintetiza la conversación en una especificación de implementación por fases, con checkboxes accionables para que un agente o desarrollador la ejecute paso a paso.
