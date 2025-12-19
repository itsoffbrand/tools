# Deferred Programmatic JS Loading System

This system is an adaption of [Timothy Ricks's Component JS solution](https://www.timothyricks.com/resources/javascript-inside-webflow-components).

Our upgrade provides a method for deferred and programmatic JavaScript loading large component heavy websites.
It allows for efficient loading of JavaScript functions based on viewport visibility and other customizable options.

## Table of Contents

1. [Overview](#overview)
2. [Installation](#installation)
3. [Usage](#usage)
   - [Head Section](#head-section)
   - [Footer Section](#footer-section)
   - [Methods](#methods)
   - [Component Usage](#component-usage)
     - [Immediate Function](#immediate-function)
     - [Deferred Function](#deferred-function)
     - [Persistent Function (SPA)](#persistent-function-spa)
4. [API Reference](#api-reference)
5. [Examples](#examples)

## Overview

The system uses the Intersection Observer API to defer the execution of JavaScript functions until specific elements enter the viewport. This approach can significantly improve page load performance by loading and executing JavaScript code only when it's needed.

## Installation

To use this system, you need to include the necessary scripts in your HTML file. There are three main parts to be added:

1. A script in the `<head>` section
2. A script in the footer
3. Function definitions in your components

## Usage

### Head Section

Add the following script to the `<head>` section of your HTML:

```html
<!-- OB Programmatic JS Functions -->
<script>
(function(global) {
  global.pageFunctions = global.pageFunctions || {
    executed: {},
    functions: {},
    debug: false,

    log: function(message) {
      if (this.debug) {
        console.log('[pageFunctions] ' + message);
      }
    },

    addFunction: function(id, fn, options) {
      if (!this.functions[id]) {
        this.functions[id] = { 
          fn: fn, 
          options: options || {},
          observer: null
        };
      }
    },

    removeFunction: function(id) {
      if (this.functions[id]) {
        // Disconnect observer if it exists
        if (this.functions[id].observer) {
          this.functions[id].observer.disconnect();
          this.log('Disconnected observer for: ' + id);
        }
        
        // Remove function and execution state
        delete this.functions[id];
        delete this.executed[id];
        
        this.log('Removed function: ' + id);
        return true;
      }
      return false;
    },

    cleanup: function() {
      var idsToRemove = [];
      
      for (var id in this.functions) {
        var funcObj = this.functions[id];
        
        // Disconnect observer if it exists
        if (funcObj.observer) {
          funcObj.observer.disconnect();
          funcObj.observer = null;
        }
        
        // Mark non-persistent functions for removal
        if (!funcObj.options.persistent) {
          idsToRemove.push(id);
        }
      }
      
      // Remove non-persistent functions
      for (var i = 0; i < idsToRemove.length; i++) {
        delete this.functions[idsToRemove[i]];
        delete this.executed[idsToRemove[i]];
      }
      
      // Reset execution state for persistent functions (so they can re-run)
      for (var id in this.functions) {
        delete this.executed[id];
      }
      
      // Reset active flag
      this.active = false;
      
      this.log('Cleanup complete. Removed ' + idsToRemove.length + ' non-persistent functions. ' + 
               Object.keys(this.functions).length + ' persistent functions retained.');
    },

    executeFunctions: function() {
      if (this.active) return;
      this.active = true;
      
      var self = this;

      var executeAll = function() {
        for (var id in self.functions) {
          if (!self.executed[id]) {
            try {
              var funcObj = self.functions[id];
              var fn = funcObj.fn;
              var options = funcObj.options;
              
              // Check if immediate execution is requested
              if (options.immediate) {
                fn();
                self.executed[id] = true;
                self.log('Executed immediately: ' + id);
                continue;
              }

              var element = document.querySelector(options.selector);
              if (element) {
                var observerOptions = {
                  root: options.root || null,
                  rootMargin: options.rootMargin || '0px',
                  threshold: options.threshold !== undefined ? options.threshold : 0
                };

                // Use IIFE to capture id in closure
                (function(functionId, funcObject) {
                  var observer = new IntersectionObserver(function(entries, obs) {
                    entries.forEach(function(entry) {
                      if (entry.isIntersecting) {
                        funcObject.fn();
                        self.executed[functionId] = true;
                        obs.unobserve(entry.target);
                        funcObject.observer = null;
                        self.log('Executed on intersection: ' + functionId);
                      }
                    });
                  }, observerOptions);

                  observer.observe(element);
                  funcObject.observer = observer;
                  self.log('Observer created for: ' + functionId);
                })(id, funcObj);
                
              } else {
                console.error('[pageFunctions] Element not found for function: ' + id + ' (selector: ' + options.selector + ')');
              }
            } catch (e) {
              console.error('[pageFunctions] Error setting up function ' + id + ':', e);
            }
          }
        }
      };

      if (document.readyState === 'loading') {
        document.addEventListener('DOMContentLoaded', executeAll);
      } else {
        executeAll();
      }
    },

    handlePageTransition: function() {
      this.cleanup();
      this.executeFunctions();
      this.log('Page transition handled');
    }
  };
})(window);
</script>
```

### Footer Section

Add the following script to the footer of your HTML:

```html
<!-- OB Programmatic JS Functions -->
<script>
  pageFunctions.executeFunctions();
</script>
```

### Methods

Initialise
```html
pageFunctions.executeFunctions();
```

Destroy / Clean up (SPA)
```html
pageFunctions.cleanup();
```

Handle SPA page transition
```html
pageFunctions.handlePageTransition();
```

Remove a specific function
```html
pageFunctions.removeFunction('functionId');
```

Enable debug logging
```html
pageFunctions.debug = true;
```

### Component Usage

#### Immediate Function

For functions that need to run immediately on page load:

```html
<script>
pageFunctions.addFunction('immediateSetup', function() {
  // Code that needs to run on page load
  console.log("Immediate setup completed.");
}, { immediate: true });
</script>
```

#### Deferred Function

For functions that should run when a specific element is about to enter the viewport:

```html
<script>
pageFunctions.addFunction('lazyLoadComponent', function() {
  // Code that runs when the component is about to enter the viewport
  console.log("Lazy-loaded component initialized.");
}, {
  selector: '[data-function="target"]',
  rootMargin: '0px 0px 50% 0px',
  threshold: 0
});
</script>
```

#### Persistent Function (SPA)

For functions that should survive SPA page transitions and re-run on new content:

```html
<script>
pageFunctions.addFunction('globalAnalytics', function() {
  // Code that should run on every page
  console.log("Analytics tracking initialized.");
}, {
  immediate: true,
  persistent: true
});
</script>
```

## API Reference

### `pageFunctions.addFunction(id, fn, options)`

Adds a function to be executed either immediately or when a specified element enters the viewport.

- `id` (string): A unique identifier for the function.
- `fn` (function): The function to be executed.
- `options` (object):
  - `immediate` (boolean): If true, executes the function immediately.
  - `selector` (string): CSS selector for the target element (for deferred execution).
  - `root` (Element|null): The root element for intersection calculations.
  - `rootMargin` (string): Margin around the root element.
  - `threshold` (number|array): Percentage of target's visibility at which to trigger execution.
  - `persistent` (boolean): If true, function survives `cleanup()` and can re-run on SPA transitions.

### `pageFunctions.removeFunction(id)`

Removes a specific function and disconnects its observer if active.

- `id` (string): The unique identifier of the function to remove.
- Returns: `true` if the function was found and removed, `false` otherwise.

### `pageFunctions.executeFunctions()`

Initializes the execution of all added functions. This should be called once in the footer of your HTML.

### `pageFunctions.cleanup()`

Disconnects all observers, removes non-persistent functions, and resets execution state for persistent functions. Used for SPA transitions.

### `pageFunctions.handlePageTransition()`

Convenience method that calls `cleanup()` followed by `executeFunctions()`. Use this on SPA route changes.

### `pageFunctions.debug`

Boolean flag. Set to `true` to enable console logging for debugging purposes.

## Examples

### Immediate Execution

```html
<script>
pageFunctions.addFunction('headerSetup', function() {
  // Initialize header components
  console.log("Header setup completed.");
}, { immediate: true });
</script>
```

### Deferred Execution

```html
<script>
pageFunctions.addFunction('lazyLoadImages', function() {
  // Load images in this section
  console.log("Images lazy-loaded.");
}, {
  selector: '.image-gallery',
  rootMargin: '0px 0px 200px 0px',
  threshold: 0.1
});
</script>
```

### SPA with Persistent Global Functions

```html
<script>
// This runs once and persists across page transitions
pageFunctions.addFunction('smoothScroll', function() {
  // Initialize smooth scrolling library
  console.log("Smooth scroll initialized.");
}, {
  immediate: true,
  persistent: true
});

// This will be cleaned up and re-registered by components on each page
pageFunctions.addFunction('pageSpecificSlider', function() {
  // Initialize a slider specific to this page
  console.log("Slider initialized.");
}, {
  selector: '.hero-slider',
  rootMargin: '0px 0px 100px 0px'
});
</script>
```

### SPA Route Change Handler

```html
<script>
// Example with a router library
router.on('navigate', function() {
  pageFunctions.handlePageTransition();
});
</script>
```
