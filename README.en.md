# Interactive State Flow

[한국어](README.md) | [English](README.en.md)

## Summary

`interactive-state-flow` separates **source-of-truth state that must be recorded promptly** from **expensive presentation, computation, and async work** in interactive applications.

Record user intent and source state promptly. Defer, cancel, prioritize, virtualize, or move expensive derived work to a suitable execution context. Commit async results only when they are still fresh and useful.

`Spec:` Agent Skills / SKILL.md | `License:` MIT | `Agents:` Codex, ChatGPT, and Agent Skills-compatible tools

## Quick Start

**Install**

```bash
npx skills add perhapsspy/interactive-state-flow
```

Or copy `skills/interactive-state-flow` directly into an agent skill directory.

**Use**

```text
Use $interactive-state-flow to refactor this search/filter UI so input state, derived rendering, and async result commits are handled clearly.
```

## Use When

- Input, selection, route, or scroll feedback is delayed by expensive work.
- Source input state is debounced just to reduce rendering cost.
- Stale async results can overwrite newer state or presentation.
- Large lists, charts, previews, parsing, validation, or layout preparation block the interaction path.
- A worker, isolate, background queue, or async boundary exists but result ownership and freshness are unclear.

## More

- Skill rules: [English](skills/interactive-state-flow/SKILL.md) | [한국어](skills/interactive-state-flow/SKILL.ko.md)
- Skill origin context: [docs/reference/origin.md](docs/reference/origin.md)

## Support

[![Buy Me A Coffee](https://cdn.buymeacoffee.com/buttons/v2/default-yellow.png)](https://www.buymeacoffee.com/perhapsspy)

## License

[MIT](LICENSE)
