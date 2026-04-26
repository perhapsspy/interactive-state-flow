# Notes

The skill helped separate the two interactive paths:

- Editor `change` -> textarea sync is source-state mirroring and should remain immediate.
- Image upload -> editor callback is async presentation mutation and deserves a freshness owner.

The skill's freshness-gate guidance was useful for spotting that `callback(...)` commits into the editor after IO without checking screen context or response validity.

The skill would overreach if it pushed this file toward debounce, workers, transitions, or broader scheduling. The real case is small; a closure-based upload owner is enough.

Suggested `SKILL.md` change: add a tiny "small vanilla JavaScript widget" example where the right answer is only operation/screen ownership around an async callback, not a new execution boundary. This would reinforce the existing "choose the lightest matching action" rule with a concrete minimal pattern.
