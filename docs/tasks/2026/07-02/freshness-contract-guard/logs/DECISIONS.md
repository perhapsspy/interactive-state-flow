**2026-07-02**
- Background: Deep review found the current skill already covers the state and async freshness mechanics, but does not explicitly prevent completion claims based only on refactor shape.
- Decision: Add the guard to the shipped skill body, not to frontmatter or README.
- Why: Body wording improves execution discipline without broadening activation or making the public summary heavier.
- Impact: `SKILL.md` and `SKILL.ko.md` carry the rule; README/frontmatter stay unchanged.
