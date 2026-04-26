# Dogfood Plan

## Goal

Check whether `interactive-state-flow` works beyond React search examples and whether it avoids over-structuring small code.

## Coverage Matrix

- Web/search: already covered by `dogfood/react-search/`.
- Web async race: already covered by `dogfood/async-race/`.
- Mobile declarative UI: Flutter or Android-style screen with expensive derived presentation and async follow-up.
- Native mobile lifecycle: Android/iOS-style async work where screen/session ownership matters.
- Desktop UI: event-dispatch/UI-thread flow with file preview, parsing, sorting, or filtering.
- Realtime/game UI: frame-sensitive input/render path with async asset or inventory presentation.
- Negative control: small clear interactive code where the skill should recommend minimal or no added boundary.

## Execution Rules

- Use separate subagents with no inherited context.
- Give each subagent only the shipped skill path, the raw task, and one write scope.
- Do not pass existing review findings or prior dogfood results.
- Each run writes `BEFORE.md`, `AFTER.md`, and `NOTES.md`.
- Parent agent synthesizes common findings into `DOGFOOD-RESULTS.md` and only changes `SKILL.md` for cross-platform lessons.

## New Runs

- `dogfood/flutter-expensive-preview/`
- `dogfood/android-lifecycle-race/`
- `dogfood/desktop-file-preview/`
- `dogfood/game-inventory-ui/`
- `dogfood/minimal-counter-control/`

