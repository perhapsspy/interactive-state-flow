# Notes

- The skill pushed the refactor toward naming two states first: immediate source state and lagging presentation state.
- The most useful rule was requiring a presentation commit owner. In Flutter terms, that became a controller instead of letting futures call `setState` directly.
- Freshness needed both a source revision and per-operation ids because preview work and tag IO are independent lanes.
- The guidance was clear that isolate/`compute` is a boundary, not the main goal. That kept UI objects and `BuildContext` out of the background input.
- The skill is a little heavy for small screens; it would help to include a minimal version for cases where one revision token and one pending flag are enough.
