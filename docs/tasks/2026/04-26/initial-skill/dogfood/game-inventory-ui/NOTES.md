# Notes

- The skill pushed the refactor to name source state and presentation state separately before choosing scheduling tools.
- The most useful rule was treating async results as untrusted until an owner checks screen key, generation, current intent, and visibility.
- The game/realtime framing mapped well to frame input, job system work, async asset loading, and render commits.
- The skill was a little heavy for deciding exactly when reusable assets like icons should be generation-gated versus visibility-gated.
- It was unclear how much stale presentation should remain visible while filtering catches up; the guidance says to be explicit, but the exact UX policy still needs product judgment.
