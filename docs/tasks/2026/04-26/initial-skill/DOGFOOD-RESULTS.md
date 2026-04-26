# Dogfood Results

## Runs

- `react-search`: refactored a search/list UI where input state, expensive filtering/sorting/highlighting, async results, and rendering were tangled.
- `async-race`: refactored a typeahead UI where older async results could overwrite newer presentation.
- `flutter-expensive-preview`: refactored a Flutter editor preview where text input, preview rendering, tag IO, and async callbacks were tangled.
- `android-lifecycle-race`: refactored a native mobile screen where selection, lifecycle/session ownership, async loading, and presentation commit were tangled.
- `desktop-file-preview`: refactored a desktop file preview where selection state, file IO, parsing, sorting/filtering, preview generation, and UI thread updates were tangled.
- `game-inventory-ui`: refactored a realtime inventory UI where frame input, filtering/sorting, async asset loading, and render commits were tangled.
- `minimal-counter-control`: checked a small responsive counter/toggle case with no expensive derived work or async freshness risk.
- `real-itdonga-toast-editor`: inspected real `itdonga-cms` admin editor JavaScript; editor change mirroring was fine, async image upload needed a small owner/freshness guard.
- `real-itdonga-common-js`: inspected real `itdonga-cms` common JavaScript; resize handling mixed viewport source state, iframe presentation work, and reload policy.

Each run was delegated to a separate subagent with only the shipped skill path, a raw task, and an isolated write scope.

## What Worked

- Both runs stopped debouncing source state and committed user input immediately.
- Both separated source state from presentation state.
- Both introduced operation identity plus cancellation or freshness gates for stale async results.
- Both moved presentation commits behind an owner instead of mutating UI state directly from async callbacks.
- Both avoided treating background execution as the default answer.
- Non-web runs mapped cleanly to controller, ViewModel, window owner, and frame/render owner boundaries.
- Desktop and Flutter runs used background execution only when expensive work justified it.
- The negative-control run correctly made no structural change.
- Real-project runs showed the skill can distinguish "no issue" paths from real async/presentation risks inside the same file.
- The real resize case exposed navigation/reload as a freshness boundary: presentation work immediately before reload is likely wasted.

## Friction Found

- The skill was broad for small examples; not every checklist category applied.
- The React search run wanted more guidance on choosing transitions/deferred values versus workers.
- The async race run exposed a general freshness issue: gates must compare against current owner-held identity, not stale captured state.
- Both runs benefited from the owning-boundary idea, but the skill could say more directly that async/background work returns data to the owner rather than mutating UI itself.
- Several runs wanted a sizing rule for how much owner/state machinery is enough.
- Android and game runs showed that stale/cached presentation policy is a product or UX decision, but binding cached presentation to the current UI still needs owner approval.
- The real toast-editor case suggested a small vanilla-JavaScript example could be useful, but adding examples to shipped skill would increase length.
- The real common.js case suggested reload/navigation policy should be named separately from responsive presentation work.

## Skill Changes Applied

- Added a Rule 4 sentence requiring freshness checks against current owner-held identity instead of stale captured values.
- Added a Rule 6 sentence saying async/background work should return data to the owner, not call UI mutation directly.
- Added a Rule 6 sentence to use the smallest owner that can answer freshness/usefulness/screen-context questions.
- Added a Rule 7 sentence saying cached or stale presentation may remain visible, but binding it to current UI needs owner approval and an explicit policy.
- Added a Rule 7 sentence saying navigation, reload, unmount, or screen replacement can be the freshness boundary, so discarded presentation work should be skipped.

## Deferred

- Do not add React-specific transition/deferred-value guidance to `SKILL.md` yet.
- Do not add platform-specific examples to `SKILL.md` yet. Non-web runs passed, but examples would make the shipped skill longer.
- Keep the checklist broad for now; it is acceptable if small tasks use only the relevant parts.
- Consider a later separate examples/reference document if repeated real tasks need framework-specific patterns.
- Do not add the real project snippets to shipped `SKILL.md`; keep them as dogfood artifacts.
