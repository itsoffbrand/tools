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
   - [Component Usage](#component-usage)
     - [Immediate Function](#immediate-function)
     - [Deferred Function](#deferred-function)
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
    addFunction: function(id, fn, options) {
      if (!this.functions[id]) {
        this.functions[id] = { fn: fn, options: options };
      }
    },
    executeFunctions: function() {
      if (this.added) return;
      this.added = true;
      document.addEventListener('DOMContentLoaded', () => {
        for (const id in this.functions) {
          if (!this.executed[id]) {
            try {
              const funcObj = this.functions[id];
              const fn = funcObj.fn;
              const options = funcObj.options || {};
              
              // Check if immediate execution is requested
              if (options.immediate) {
                fn();
                this.executed[id] = true;
                continue; // Skip setting up observer
              }

              const element = document.querySelector(options.selector);
              if (element) {
                const observerOptions = {
                  root: options.root || null,
                  rootMargin: options.rootMargin || '0px',
                  threshold: options.threshold !== undefined ? options.threshold : 0
                };

                const observer = new IntersectionObserver((entries, observer) => {
                  entries.forEach(entry => {
                    if (entry.isIntersecting) {
                      fn();
                      this.executed[id] = true;
                      observer.unobserve(entry.target);
                    }
                  });
                }, observerOptions);

                observer.observe(element);
              } else {
                console.error(`Element not found for function ${id}`);
              }
            } catch (e) {
              console.error(`Error setting up IntersectionObserver for function ${id}:`, e);
            }
          }
        }
      });
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

### `pageFunctions.executeFunctions()`

Initializes the execution of all added functions. This should be called once in the footer of your HTML.

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
