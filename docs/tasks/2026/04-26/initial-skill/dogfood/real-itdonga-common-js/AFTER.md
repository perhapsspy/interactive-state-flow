# After

The code does not need a large framework-style rewrite. A realistic refactor is to name the viewport source state, separate derived presentation work from reload policy, and avoid doing iframe DOM work immediately before navigation.

```js
const viewportState = {
  width: window.innerWidth,
  breakpoint: getBreakpoint(window.innerWidth),
};

function readViewportState() {
  const width = window.innerWidth;
  return {
    width,
    breakpoint: getBreakpoint(width),
  };
}

function applyResponsivePresentation(state) {
  makeYouTubeIframesResponsive(state.breakpoint);
}

function shouldReloadForBreakpointChange(previousState, nextState) {
  return previousState.breakpoint !== nextState.breakpoint;
}

window.addEventListener('resize', debounce(function () {
  const nextState = readViewportState();
  const previousState = {
    width: viewportState.width,
    breakpoint: viewportState.breakpoint,
  };

  viewportState.width = nextState.width;
  viewportState.breakpoint = nextState.breakpoint;

  if (!shouldReloadForBreakpointChange(previousState, nextState)) {
    return;
  }

  // If ads require a full reload across breakpoint families, make that the only
  // breakpoint-change side effect. The next page load will apply iframe styles.
  window.location.reload();
}, 500));

onDomReady(function () {
  const state = readViewportState();
  viewportState.width = state.width;
  viewportState.breakpoint = state.breakpoint;

  applyResponsivePresentation(state);
  initializeAdsForViewport(state);
});
```

If the page can support live breakpoint changes without reloading, the resize handler can instead call a single owner for presentation updates:

```js
window.addEventListener('resize', debounce(function () {
  const nextState = readViewportState();
  const breakpointChanged = nextState.breakpoint !== viewportState.breakpoint;

  viewportState.width = nextState.width;
  viewportState.breakpoint = nextState.breakpoint;

  if (breakpointChanged) {
    applyResponsivePresentation(nextState);
    refreshAdsForViewport(nextState);
  }
}, 500));
```

In either version, `makeYouTubeIframesResponsive` should accept the already-read breakpoint instead of reading `window.innerWidth` per iframe:

```js
function makeYouTubeIframesResponsive(breakpoint) {
  const isResponsiveSize = breakpoint === 'sm' || breakpoint === 'md';
  document.querySelectorAll('article iframe[src*="youtube.com"], article iframe[src*="youtu.be"]')
    .forEach(function (iframe) {
      if (isResponsiveSize) {
        const originalWidth = iframe.width || iframe.offsetWidth || iframe.getAttribute('width') || 560;
        const originalHeight = iframe.height || iframe.offsetHeight || iframe.getAttribute('height') || 315;
        iframe.style.width = '100%';
        iframe.style.height = 'auto';
        iframe.style.aspectRatio = String(originalWidth / originalHeight);
        iframe.style.maxWidth = '100%';
      } else {
        iframe.style.width = '';
        iframe.style.height = '';
        iframe.style.aspectRatio = '';
        iframe.style.maxWidth = '';
      }
    });
}
```

This keeps the code close to the current style while making the owner of viewport freshness explicit.
