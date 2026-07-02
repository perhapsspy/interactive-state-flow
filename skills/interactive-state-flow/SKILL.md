---
name: interactive-state-flow
description: Use for interactive app code where typing, selection, navigation, search/filter, command palettes, previews, streaming/tool output, realtime updates, or file/diff/log views are laggy, stale, or race-prone because user intent/source state, derived presentation, async IO, scheduling, or background work are mixed. Keep source state prompt, move only justified expensive work off the urgent path, and commit async or presentation results through freshness-owning boundaries.
---

# Interactive State Flow

## Purpose

Preserve responsiveness and maintainability by separating prompt source-state recording from expensive presentation and async work.

This is not framework-specific performance tuning. Use it to keep user intent current, keep expensive work off the urgent interaction path, and commit only results that still belong to the current interaction context.

## Core Thesis

Record user intent and source-of-truth state promptly.

Run expensive presentation, derived computation, IO, and async follow-up work only when the cost and execution path are justified.

Commit results only after the owner accepts them as fresh and useful. Owner means the smallest boundary that can answer whether a result still belongs to the current user intent, source state, screen context, and presentation policy.

Prompt source-state recording does not mean synchronous rendering. It means user intent must not be delayed just to reduce expensive presentation work.

## Use / Do Not Use

Use this skill when:

- input, touch, scroll, focus, gesture, navigation, route, or selection feedback is delayed
- expensive derived presentation follows source state too eagerly
- async results can arrive out of order and overwrite newer state
- one component, screen, handler, or flow mixes intent, IO, derivation, scheduling, and presentation mutation
- large lists, charts, previews, parsing, validation, layout prep, or transformations block interaction
- background execution exists but ownership, cancellation, ordering, failure behavior, or freshness is unclear
- humans or agents must trace a large interactive flow with minimal context

Do not use this skill when:

- the change is static or non-interactive
- the current implementation is small, clear, responsive, and free of expensive derived work or async ordering risk
- the problem is mainly backend throughput rather than interactive responsiveness
- extra scheduling, background execution, or state boundaries would cost more than the work itself

## Common Agent Cases

Reach for this skill when a Codex task mentions:

- search, typeahead, command palette, filter, or date input where the source value is debounced, delayed, or overwritten by old results
- preview, diff, file viewer, log viewer, map/canvas, or chart rendering that blocks selection, typing, scrolling, tab changes, or mode changes
- streaming, tool-output, upload, parse, fetch, realtime, SSE, socket, refresh, or background results that can land in the wrong run, route, tab, screen, session, or selected item
- URL, route, focus, selection, remount, cache, fallback, or quiet-refresh behavior where displayed state can look current while source state, freshness, or ownership says otherwise

Do not use it just because code has async work, effects, workers, memoization, or caching. Require an interactive lag, stale-result, mixed-responsibility, or user-visible freshness risk.

## Failure Smells

- Actual input, selection, route, or intent state is debounced to reduce rendering cost.
- Async or background completion mutates UI state without checking current intent, screen, or operation.
- Rendering, preview generation, parsing, or heavy derivation runs in the same path that records user intent.
- Expensive derived data is treated as source-of-truth state.
- Background work has no owner for cancellation, ordering, failure handling, or final commit.
- Presentation is built for all source state when only a visible range, cached view, or progressive preview is useful.
- One opaque handler captures input, starts IO, derives data, schedules work, and mutates presentation.

## Primary Flow

1. Name the user-visible behavior contract: what intent/source state updates promptly, what presentation may lag, which owner accepts freshness, and what current, stale, pending, cached, or progressive state the user may see.
2. Record user intent promptly.
3. Commit source-of-truth state through the source owner.
4. Classify follow-up work by urgency, cost, visibility, and freshness risk.
5. Keep the interaction path light.
6. Defer, cancel, batch, prioritize, virtualize, skip, cache, or move expensive work only when justified.
7. Commit async or presentation results only through the owner when they are still fresh and useful.
8. Test behavior contracts, not scheduler internals.

## Rules

### 1. Separate Source State From Presentation State

Source state records current truth: current input, latest intent, selected item, active route/screen/context, operation id, and source-of-truth model.

Presentation state describes how truth is shown: visible rows, chart model, preview output, rendered range, pending/stale/cached display, or progressive output.

Keep source state current. Let presentation lag only when lag preserves responsiveness and does not mislead the user. Do not delay source state just to reduce rendering; defer expensive derived work instead.

### 2. Keep the Interaction Path Light

The path that handles input, navigation, selection, and immediate feedback should stay small and predictable.

Avoid: `input -> update source -> filter/sort/build/render everything -> feedback appears`.

Prefer: `input -> record intent -> commit source -> show feedback -> schedule expensive work -> discard stale work -> commit fresh presentation`.

### 3. Classify Before Choosing Tools

Do not start with debounce, memoization, virtualization, transitions, idle work, background execution, batching, or cancellation.

First ask: intent or source state? derived work? presentation? async IO? expensive computation? can it become stale? is it visible or useful now? can it be cancelled? does it need a separate execution context?

Then choose the lightest matching action:

- intent or source state -> record promptly through the source owner
- expensive visible presentation -> defer, virtualize, cache, or show progressive output
- expensive hidden presentation -> skip, idle, or precompute only when likely useful
- stale-risk async or IO -> use operation identity, cancellation, and a freshness gate
- heavy CPU work -> move execution only when transfer, ownership, cancellation, ordering, and failure behavior are clear

### 4. Freshness-Gate Async Results

A completed result is not automatically valid. Before commit, check whether it still matches current intent, operation/generation, cancellation state, screen/route/session/context, usefulness, and user-visible meaning.

Use explicit identity when needed: request id, generation id, operation token, cancellation token, abort handle, input snapshot, screen key, or session key.

Compare against current owner-held identity, not a stale captured value.

### 5. Treat Background Execution as a Boundary

Move work to another execution context only when the interaction path needs protection and the boundary has clear input, output, ownership, cancellation, ordering, and failure behavior.

Good candidates include parsing, sorting, filtering, aggregation, validation, indexing, preview generation, layout model prep, large IO, and expensive transformation.

Avoid moving work when it is small, depends directly on UI objects, costs more to transfer than compute, has unclear result ownership or commit ordering, cannot be cancelled safely, or makes the code harder to reason about.

### 6. Commit Presentation Through an Owner

Async or background work should return results to the owner, not mutate UI state directly.

The owner decides whether the result is fresh, useful, screen-current, safe to bind to source state, and whether UI should show current, stale, pending, cached, skeleton, or progressive output.

Use the smallest owner that can answer these questions. If no owner can answer them, do not add an async or background boundary yet.

### 7. Respect Screen Context

Do not produce all presentation just because state exists. Ask whether it is visible now, likely to be visible soon, needed during active interaction, safe to show as stale content, suitable for progressive output, limited to a visible range, and still fresh.

Presentation should be useful, not merely complete. Stale output may remain visible, but never silently present stale output as current when the distinction matters.

If navigation, reload, unmount, or screen replacement is the freshness boundary, skip presentation work that will be discarded before the next active screen.

## Hard Rules

- Do not delay user intent or source state just to reduce presentation cost.
- Do not debounce actual intent state when only expensive derived work should be deferred.
- Do not commit async results without freshness checks.
- Do not move work to another execution context just because it is expensive.
- Do not mix intent, IO, derived data, scheduling, and presentation mutation in one opaque flow.
- Do not choose performance tools before classifying responsibility.
- Do not let background work directly own UI mutation.
- Do not make small, clear, responsive code harder just to introduce scheduling boundaries.
- Do not silently present stale output as current when the distinction matters.
- Do not declare a user-visible stale, laggy, or race-prone interaction fixed from refactor shape alone; verify the behavior contract with the project's smallest appropriate evidence.
- Do not use this skill as generic UI debugging, product-policy, or browser-verification ownership. If stale, cached, pending, or verification policy is undefined, surface the decision instead of choosing it inside this skill.

## Contract Tests

Test behavior contracts instead of scheduler internals:

- Immediate state: intent and source state are recorded promptly; operation identity is visible when freshness matters.
- Freshness: stale, cancelled, inactive-screen/session, or obsolete-generation results cannot overwrite newer state or presentation. If request A starts before B but resolves after B, A must not overwrite B.
- Interaction: urgent feedback is not blocked by expensive work, and delayed non-urgent work does not corrupt source state.
- Presentation: expensive presentation lags only where safe; stale or pending presentation is explicit when needed; only useful or visible presentation is produced.
- Execution boundary: background work has clear input/output, commits through the owner, handles failures safely, and cannot commit cancelled or obsolete results.

## Platform Mapping

Use the platform's local equivalent of interaction path and execution boundary. Do not make any specific API the center of the design.

Examples include web workers or deferred rendering, React Native UI/JS paths, Flutter isolates, Android coroutine dispatchers, iOS background queues, desktop worker pools, and game job systems.

The platform names differ, but the rule is the same: protect the interaction path and commit only fresh, useful results through the owner that understands the current interaction context.

## Final Checklist

Before finishing, confirm:

- user-visible behavior and freshness contracts were named and verified with the smallest appropriate project evidence
- user intent and source state are recorded promptly
- expensive derived work is not forced onto the urgent interaction path
- presentation lags only where safe and understandable
- async results have freshness identity and an owner
- cancelled or obsolete work cannot mutate state or presentation
- execution boundaries have clear input, output, ownership, cancellation, ordering, and failure behavior
- tests cover immediate state, freshness, interaction, presentation, and execution-boundary contracts
