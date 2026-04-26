# Before

Relevant pseudocode from `app/static/js/common.js`:

```js
let currentBreakpoint = getBreakpoint(window.innerWidth);

window.addEventListener('resize', debounce(function () {
  const newBreakpoint = getBreakpoint(window.innerWidth);
  if (newBreakpoint !== currentBreakpoint) {
    currentBreakpoint = newBreakpoint;
    makeYouTubeIframesResponsive();
    window.location.reload();
  }
}, 500));

onDomReady(function () {
  const windowWidth = window.innerWidth;
  makeYouTubeIframesResponsive();

  document.querySelectorAll('div.ad-container').forEach(function (adContainer) {
    const adId = getAdIdByBreakpoint(adContainer, windowWidth);
    const displayType = getAdDisplayType(adContainer, windowWidth);
    const insertionTarget = findAdInsertionTarget(...);
    const adElement = createAdElement(adId, displayType);
    insertAdElement(...);
    window.top.__vm_add.push(adElement);
  });
});
```

## Suspected Risks

- The source state for viewport width is implicit. The resize handler records only `currentBreakpoint`, while ad setup captures `windowWidth` once during DOM ready.
- The resize path mixes source update, presentation mutation, and navigation side effect in one handler.
- `makeYouTubeIframesResponsive()` runs immediately before `window.location.reload()`, so the iframe DOM work is likely wasted on breakpoint changes.
- If reload is required for ad vendor behavior, that policy is not named separately from YouTube iframe presentation work.
- If reload is not strictly required, resizing across breakpoints can discard page state and repeat ad/iframe initialization instead of applying only the needed presentation changes.
- Ad placement is only fresh for the width captured on DOM ready. The reload currently acts as the freshness boundary, but that coupling is implicit.
