# Source

Evaluated source file: `app/static/js/common.js`

The file contains shared frontend behavior:

- breakpoint definitions and resize handling
- clipboard button feedback
- YouTube iframe responsive styling inside `article`
- ad placement initialization for `div.ad-container`

## Interaction Path Evaluated

The evaluated path is the browser resize and initial page setup flow around breakpoint-dependent DOM work:

1. `currentBreakpoint` is initialized from `window.innerWidth`.
2. A debounced `resize` listener waits 500 ms, reads the new breakpoint, and compares it to `currentBreakpoint`.
3. When the breakpoint changes, it updates `currentBreakpoint`, calls `makeYouTubeIframesResponsive()`, then calls `window.location.reload()`.
4. On DOM ready, `makeYouTubeIframesResponsive()` runs again, then ad containers are scanned, breakpoint-specific ad ids/display types are selected from the initial `window.innerWidth`, ad elements are inserted, and pushed to `window.top.__vm_add`.

The main question is whether resize, ad setup, and YouTube iframe DOM work are mixing immediate viewport state, derived presentation, and page reload side effects in a way that risks stale or unnecessary work.
