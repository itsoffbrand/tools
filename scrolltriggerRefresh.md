# ScrollTrigger Refresh Function

This module provides a robust solution for refreshing GSAP ScrollTrigger instances in response to various DOM and window events. It's designed to optimize performance by debouncing refresh calls and avoiding unnecessary updates during active scrolling.

## Table of Contents

1. [Installation](#installation)
2. [Usage](#usage)
3. [Functions](#functions)
   - [setupScrollTriggerRefresh](#setupscrolltriggerrefresh)
   - [cleanupScrollTriggerRefresh](#cleanupscrolltriggerrefresh)
4. [How It Works](#how-it-works)
5. [Lenis Scroll Variation](#lenis-scroll-variation)
6. [Full Code](#full-code)

## Installation

Ensure you have GSAP and ScrollTrigger installed in your project:

```bash
npm install gsap
```

## Usage

Import the functions in your main JavaScript file:

```javascript
import { setupScrollTriggerRefresh, cleanupScrollTriggerRefresh } from './path/to/scrollTriggerRefresh';

// Call this function to set up the refresh mechanism
setupScrollTriggerRefresh();

// Call this function when you need to clean up (e.g., on component unmount)
cleanupScrollTriggerRefresh();
```

## Functions

### setupScrollTriggerRefresh

This function sets up all the necessary observers and event listeners to trigger ScrollTrigger refreshes when needed.

### cleanupScrollTriggerRefresh

This function removes all observers and event listeners set up by `setupScrollTriggerRefresh`.

## How It Works

The module uses a combination of ResizeObserver, MutationObserver, and event listeners to detect changes that might affect ScrollTrigger animations. It includes mechanisms to:

- Detect window resizing
- Observe DOM mutations
- Handle media (image/video) loading
- Manage scroll state to avoid refreshing during active scrolling

The refresh calls are debounced to prevent performance issues from rapid successive calls.

## Lenis Scroll Variation

For sites using Lenis Smooth Scroll, you need to make the following modifications:

1. Replace the `isScrolling` function:

```javascript
function isScrolling() {
  return document.documentElement.classList.contains("lenis-scrolling");
}
```

2. Remove the `startScrollDetection` and `setupScrollDetection` functions.

3. Update the `cleanupScrollTriggerRefresh` function:

```javascript
function cleanupScrollTriggerRefresh() {
  if (resizeObserver) {
    resizeObserver.disconnect();
    resizeObserver = null;
  }
  if (mutationObserver) {
    mutationObserver.disconnect();
    mutationObserver = null;
  }
  if (resizeHandler) {
    window.removeEventListener("resize", resizeHandler);
    resizeHandler = null;
  }
  if (loadHandler) {
    window.removeEventListener("load", loadHandler);
    loadHandler = null;
  }
  if (domContentLoadedHandler) {
    document.removeEventListener("DOMContentLoaded", domContentLoadedHandler);
    domContentLoadedHandler = null;
  }
  // Remove the following line:
  // window.removeEventListener("scroll", startScrollDetection);
}
```

4. Update the `setupScrollTriggerRefresh` function:

```javascript
function setupScrollTriggerRefresh() {
  cleanupScrollTriggerRefresh();
  setupResizeObserver();
  // Remove the following line:
  // setupScrollDetection();

  loadHandler = () => {
    scheduleRefresh("Window load");
    const mediaElements = document.querySelectorAll("img, video");
    mediaElements.forEach((el) => {
      if (!el.complete) {
        el.addEventListener("load", () => scheduleRefresh("Media loaded"));
      }
    });
  };
  window.addEventListener("load", loadHandler);

  domContentLoadedHandler = () => scheduleRefresh("DOMContentLoaded");
  document.addEventListener("DOMContentLoaded", domContentLoadedHandler);

  scheduleRefresh("Initial setup");
}
```

These changes ensure that the ScrollTrigger refresh function works correctly with Lenis Smooth Scroll, relying on Lenis's own scroll detection mechanism.

## Full Code

```javascript
import { gsap } from "gsap";
import { ScrollTrigger } from "gsap/ScrollTrigger";

gsap.registerPlugin(ScrollTrigger);

let resizeObserver = null;
let mutationObserver = null;
let resizeHandler = null;
let loadHandler = null;
let domContentLoadedHandler = null;
let isRefreshing = false;
let needsRefresh = false;
let scrollTimeout = null;

function startScrollDetection() {
  document.documentElement.classList.add("is-scrolling");
  if (scrollTimeout) clearTimeout(scrollTimeout);
  scrollTimeout = setTimeout(() => {
    document.documentElement.classList.remove("is-scrolling");
  }, 150);
}

function setupScrollDetection() {
  window.addEventListener("scroll", startScrollDetection, { passive: true });
}

function isScrolling() {
  return document.documentElement.classList.contains("is-scrolling");
}

function attemptRefresh(reason) {
  if (isRefreshing) return;
  isRefreshing = true;

  if (typeof ScrollTrigger !== "undefined") {
    if (!isScrolling()) {
      ScrollTrigger.refresh();
      needsRefresh = false;
      isRefreshing = false;
    } else {
      const observer = new MutationObserver((mutationsList, observer) => {
        if (!isScrolling()) {
          observer.disconnect();
          ScrollTrigger.refresh();
          needsRefresh = false;
          isRefreshing = false;
        }
      });
      observer.observe(document.documentElement, {
        attributes: true,
        attributeFilter: ["class"],
      });
    }
  } else {
    console.warn("ScrollTrigger is not available");
    isRefreshing = false;
  }
}

function scheduleRefresh(reason) {
  needsRefresh = true;
  attemptRefresh(reason);
}

function callAfterResize(func, delay) {
  let dc = gsap.delayedCall(delay || 0.2, func).pause(),
    handler = () => dc.restart(true);
  window.addEventListener("resize", handler);
  return handler;
}

function setupResizeObserver() {
  if (resizeObserver) {
    resizeObserver.disconnect();
  }

  if ("ResizeObserver" in window) {
    let dc = gsap
      .delayedCall(0.2, () => scheduleRefresh("ResizeObserver"))
      .pause();
    resizeObserver = new ResizeObserver((entries) => {
      dc.restart(true);
    });
    resizeObserver.observe(document.body);
  } else {
    if (resizeHandler) {
      window.removeEventListener("resize", resizeHandler);
    }
    resizeHandler = callAfterResize(
      () => scheduleRefresh("Window resize"),
      0.2
    );
  }
}

function setupMutationObserver() {
  if (mutationObserver) {
    mutationObserver.disconnect();
  }

  let mutationTimeout;
  mutationObserver = new MutationObserver((mutations) => {
    let shouldRefresh = false;
    for (let mutation of mutations) {
      if (
        mutation.type === "childList" ||
        (mutation.type === "attributes" &&
          ["style", "class"].includes(mutation.attributeName))
      ) {
        shouldRefresh = true;
        break;
      }
    }
    if (shouldRefresh) {
      clearTimeout(mutationTimeout);
      mutationTimeout = setTimeout(() => {
        scheduleRefresh("DOM mutation");
      }, 100);
    }
  });

  mutationObserver.observe(document.body, {
    childList: true,
    subtree: true,
    attributes: true,
    attributeFilter: ["style", "class"],
  });
}

function cleanupScrollTriggerRefresh() {
  if (resizeObserver) {
    resizeObserver.disconnect();
    resizeObserver = null;
  }
  if (mutationObserver) {
    mutationObserver.disconnect();
    mutationObserver = null;
  }
  if (resizeHandler) {
    window.removeEventListener("resize", resizeHandler);
    resizeHandler = null;
  }
  if (loadHandler) {
    window.removeEventListener("load", loadHandler);
    loadHandler = null;
  }
  if (domContentLoadedHandler) {
    document.removeEventListener("DOMContentLoaded", domContentLoadedHandler);
    domContentLoadedHandler = null;
  }
  window.removeEventListener("scroll", startScrollDetection);
}

function setupScrollTriggerRefresh() {
  cleanupScrollTriggerRefresh();
  setupResizeObserver();
  setupScrollDetection(); // remove if lenis 
  // optional to set up mution observer (needs programmed)
  // setupMutationObserver();

  loadHandler = () => {
    scheduleRefresh("Window load");
    const mediaElements = document.querySelectorAll("img, video");
    mediaElements.forEach((el) => {
      if (!el.complete) {
        el.addEventListener("load", () => scheduleRefresh("Media loaded"));
      }
    });
  };
  window.addEventListener("load", loadHandler);

  domContentLoadedHandler = () => scheduleRefresh("DOMContentLoaded");
  document.addEventListener("DOMContentLoaded", domContentLoadedHandler);

  scheduleRefresh("Initial setup");
}

export { setupScrollTriggerRefresh, cleanupScrollTriggerRefresh };
```
