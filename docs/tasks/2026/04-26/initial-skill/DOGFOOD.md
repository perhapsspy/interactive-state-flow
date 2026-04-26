# Dogfood

## Purpose

Run small realistic tasks with minimal context to see whether `interactive-state-flow` changes agent behavior in useful ways.

## Isolation Rules

- Give each dogfood agent only the shipped skill path and a raw task.
- Do not pass prior review findings, expected fixes, or this file as task context.
- Keep each run in a separate write scope under `dogfood/<case>/`.
- Treat unclear or over-heavy behavior as skill feedback, not agent failure.

## Evaluation Criteria

- Source state is recorded immediately.
- Expensive derived presentation is separated from source state.
- Async results have operation identity, cancellation, or a freshness gate when needed.
- Presentation commits through an owner that understands current screen context.
- The solution avoids unnecessary background execution or scheduling layers.
- Contract tests or testable behavior are easier to name after applying the skill.

## Cases

- `dogfood/react-search/`: search/list UI where input, derived presentation, rendering, and async work may be tangled.
- `dogfood/async-race/`: interactive async requests where stale results may overwrite newer state or presentation.

