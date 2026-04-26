# Notes

What the skill made me do:

- Separate immediate source state (`query`, `activeSearchId`) from presentation state (`shownForQuery`, `status`, `results`).
- Add an explicit freshness owner with an operation id and abort handle.
- Treat async completion as an input to a commit owner, not as permission to mutate UI state.
- Make pending and stale presentation explicit instead of pretending old results are current.
- Think in behavior contracts: immediate state, stale result rejection, cancellation, and owner-only presentation commit.

Where the skill was unclear or heavier than needed:

- It gives the right ownership model, but leaves framework-specific state closure issues to the implementer. In React, the freshness gate should avoid relying on possibly stale `source` values unless mirrored in a ref or reducer.
- The checklist is broad for a small async race. For this example, background execution and virtualization were not relevant.
- "Presentation commit owner" is useful, but the skill could show one compact example of what that owner looks like in a component, reducer, or controller.
