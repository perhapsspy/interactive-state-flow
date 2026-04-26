# Notes

## What The Skill Helped With

- It framed `window.innerWidth` and breakpoint as source state, while iframe styles and ad DOM insertion are presentation/effect work.
- It highlighted that the resize handler currently combines source update, derived DOM mutation, and reload in one opaque flow.
- It made the implicit freshness policy visible: ad placement is fresh because the page reloads, not because the ad code can respond to viewport changes.
- It helped avoid over-focusing on debounce itself. The larger issue is ownership and whether work before reload is useful.

## Where The Skill Could Overreach

- This file is small and old-school DOM scripting, so introducing operation ids, cancellation tokens, workers, or test harnesses would be excessive.
- The expensive-work concern is mild. The stronger practical issue is coupling and wasted presentation mutation before reload.
- The ad vendor may require reload semantics. The skill should not push a live-refresh architecture without confirming vendor constraints.

## Suggested SKILL.md Change

This case suggests adding a short example for resize/layout flows where navigation or reload is the actual freshness boundary. The guidance could say: if a reload is intentionally used to reinitialize third-party layout or ads, avoid doing presentation DOM work immediately before the reload, and name the reload policy separately from responsive presentation updates.
