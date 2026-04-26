# Skill Content Review

## Review Criteria

1. Trigger clarity
- The frontmatter description should identify when the skill should load without relying on the body.
- It should be broad enough for web, mobile, desktop, and game UI, but narrow enough not to trigger for ordinary async or backend work.

2. Origin alignment
- The skill should preserve the origin thesis: source state now, presentation when useful, async results only when fresh.
- It should stay independent from `structure-first` and `project-context` inside `SKILL.md`.

3. Operational usefulness
- The body should guide an agent through concrete decisions, not only state principles.
- Failure smells, flow, rules, do-not rules, and contract tests should be actionable during code changes.

4. Platform neutrality
- Platform APIs should remain examples, not the center of the design.
- The shared vocabulary should be interaction path, source state, presentation state, freshness gate, execution context, and owning boundary.

5. Complexity control
- The skill should prevent over-engineering when code is small or the boundary cost exceeds the work.
- It should not encourage workers, scheduling, or extra state layers by default.

6. Testability
- Contract tests should protect behavior, not scheduler implementation details.
- The test categories should be specific enough to turn into actual test cases.

7. Progressive disclosure
- `SKILL.md` should be dense enough to be useful but not so long that it becomes a general essay.
- Repeated or secondary explanation should live outside the shipped skill.

## Findings

### 1. Main gap: classification does not yet produce a clear tool choice

Lines 118-133 correctly tell the agent to classify follow-up work before choosing tools. Lines 55-62 and 120 list possible actions: defer, cancel, batch, prioritize, virtualize, move to another execution context.

What is weaker: after classification, the skill does not give a compact decision mapping.

Useful missing shape:

- source state or user intent -> commit immediately through the source owner
- expensive but visible presentation -> defer, virtualize, or progressive presentation
- expensive but not visible -> skip, idle, or precompute only when likely useful
- async result with stale risk -> operation id, cancellation, freshness gate
- heavy CPU work -> background only when transfer and ownership are clear
- IO-bound work -> cancellation, request identity, screen/session guard

Impact: agents may understand the principles but still jump straight from classification to a favorite tool.

### 2. Frontmatter is accurate but dense

Line 3 covers the right triggers and constraints, but it is long and includes overlapping terms: "async follow-up work" and "background results" partly cover the same mental space.

It is valid and passes validation, but the trigger could be tighter without losing meaning.

Possible direction: keep the trigger centered on interactive code where source state must update immediately while expensive presentation, derived work, IO, or async results need deferred execution, cancellation, prioritization, or freshness gates.

### 3. Source commit immediacy could clarify render coupling

Lines 55-56 say record user intent and commit source state immediately. This is the right thesis.

Potential ambiguity: in some frameworks, committing source state can synchronously trigger expensive rendering if derived presentation is coupled to that same state path. The skill implies the fix elsewhere, but it could say more explicitly that immediate source commit should not require immediate expensive presentation commit.

Impact: an agent might treat "commit source state immediately" as enough, without separating derived subscribers or render-heavy projections.

### 4. Test section is well scoped but could include stale-result race examples

Lines 230-275 define useful contract categories. They are behavior-focused and match the origin.

Missing practical edge: the Freshness Contract would be stronger with one compact example pattern, such as "request A starts, request B starts, B commits, A resolves later and is ignored." This would make the most important failure mode more concrete without turning the skill into a framework guide.

### 5. Platform mapping is balanced

Lines 277-291 are platform-neutral and keep APIs as examples. This matches the origin.

No action needed unless the list grows. If it grows, move platform-specific details to reference files rather than expanding `SKILL.md`.

### 6. Over-engineering guardrails are present

Lines 34-41, 178-186, and 220-228 clearly guard against unnecessary scheduling and background execution.

No major issue. This is one of the stronger parts of the skill.

### 7. Korean pair should be updated whenever English changes

`SKILL.ko.md` currently tracks the English content closely enough. If the English skill gains a decision mapping or race example, the Korean pair should be updated in the same pass.

## Recommended Changes

1. Add a short "Decision Hints" or "Selection Guide" section after "Classify Follow-Up Work Before Choosing Tools".
2. Tighten the frontmatter description only if trigger brevity becomes a priority.
3. Add one sentence clarifying that source state commits must not force expensive presentation to commit synchronously.
4. Add one stale async race example under Freshness Contract.
5. Keep platform mapping as-is for now.

## Applied Revision

- Added a compact "Choose the lightest matching action" mapping inside Rule 3 instead of adding a new top-level section.
- Tightened frontmatter wording while preserving trigger coverage.
- Clarified that immediate source-state commit must not force expensive presentation to commit synchronously.
- Added one stale-result race example to the Freshness Contract.
- Compressed source-state and presentation-state examples from bullet lists into inline examples, reducing `SKILL.md` from 304 to 292 lines while adding the missing guidance.
- Mirrored the same content changes in `SKILL.ko.md`.

## Later Simplification

- Adopted the `recorded promptly` wording to avoid implying synchronous rendering.
- Added an explicit owner definition.
- Reframed expensive work around justified execution and fresh commit, not "run only when fresh."
- Compressed Do Not Rules into Hard Rules, and shortened Contract Tests and Platform Mapping.
- After an additional compression pass, current `SKILL.md` is 163 lines while keeping trigger, decision flow, hard rules, contract tests, and platform-neutral guidance.
- Applied final external review refinements: trigger now leads with laggy input/stale async/mixed-flow failure modes, Do Not Use avoids treating "small and clear" as a standalone exclusion, Primary Flow commit goes through the owner, and Rule 7 wording is smoother.
