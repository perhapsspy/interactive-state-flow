# Notes

- The skill pushed the refactor toward naming the source owner first: `selectedPhotoId` plus `generation`.
- It made the presentation commit path explicit. Async work returns prepared data, but the ViewModel decides whether it is still fresh enough to publish.
- Cancellation became a local ownership rule instead of a lifecycle afterthought.
- The guidance was clear for freshness and commit ownership. It was less specific about how much stale UI to preserve, so the example uses a simple `isStale` flag and keeps matching cached presentation only for the same photo.
- The checklist is useful, but for small Android screens it can feel heavier than the actual code change unless treated as contracts rather than required machinery.
