# Freshness Contract Guard

## Goal

- Add a narrow shipped-skill guard so `interactive-state-flow` does not treat code shape alone as proof that a user-visible stale, laggy, or race-prone interaction is fixed.

## Scope

- Update `skills/interactive-state-flow/SKILL.md` and its frontmatter-free Korean pair.
- Keep README/frontmatter unchanged so trigger scope does not broaden.
- Use deep-review feedback to check the wording before closeout.

## Current Understanding

- The existing skill already owns source state, presentation state, async freshness, owner-held identity, and route/session/selection freshness checks.
- The missing guard is closeout discipline: name the behavior contract and verify it with the smallest appropriate evidence before declaring an interaction fixed.
- The skill must not become generic UI debugging, product-policy, or browser-verification ownership.

## Current State

- Deep review accepted the narrow body placement and requested one Korean wording fix so the skill does not appear to decide product policy.
- Final wording now covers `Primary Flow`, `Hard Rules`, and `Final Checklist`.
- Validation passed for YAML frontmatter parsing, diff whitespace, and project-context runtime shape.

## Next Step

- Reopen only if the wording needs publishing, installed-skill sync, or another review pass.

## Working Boundary

- `skills/interactive-state-flow/SKILL.md`
- `skills/interactive-state-flow/SKILL.ko.md`
