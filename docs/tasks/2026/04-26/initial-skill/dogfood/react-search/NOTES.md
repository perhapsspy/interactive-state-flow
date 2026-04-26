# Notes

What the skill made me do:

- Keep the text input value as immediate source state instead of debouncing it.
- Name separate source and presentation state so lag is intentional and visible.
- Add operation identity, abort ownership, and a freshness gate before committing results.
- Move filtering, sorting, scoring, and highlight generation out of the input handler.
- Treat virtualization and row limits as presentation choices, not source truth.

Where the skill was unclear or heavy:

- It gives the right ownership questions but not much guidance on when React transitions are enough versus when to introduce a worker.
- The distinction between stale display, pending display, and cached display could use a small UI example.
- The checklist is useful but broad for a small component; a short "minimum viable boundary" example would help.
