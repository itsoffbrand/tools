# Fluid CSS Methodology

A responsive design approach that scales elements proportionally with viewport size while respecting design breakpoints.

## Table of Contents

- [Introduction](#introduction)
- [Key Features](#key-features)
- [Getting Started](#getting-started)
- [Usage](#usage)
- [Configuration](#configuration)
- [Breakpoints](#breakpoints)
- [Advanced Techniques](#advanced-techniques)
- [Browser Support](#browser-support)

## Introduction

The Fluid CSS Methodology creates a responsive design system that automatically scales UI elements based on viewport dimensions without relying on numerous breakpoints. It's inspired by [Willy Brauner's approach](https://willybrauner.com) and provides a consistent, fluid experience across all device sizes.

Unlike traditional responsive approaches that use fixed breakpoints for everything, this methodology creates a fluid unit that adapts to the viewport, combined with strategic breakpoints for major layout shifts.

## Key Features

- **Truly Fluid Scaling**: Elements scale smoothly with viewport size
- **Design-to-Code Simplicity**: Transfer values directly from Figma/design tools
- **Reduced Breakpoint Complexity**: Less media query overrides needed
- **Maximum Size Control**: Prevent elements from growing too large on big screens
- **Responsive Ratio Adjustment**: Configure width/height influence on scaling
- **Design Value Updates**: Reference different design mockup dimensions at each breakpoint
- **Balanced Proportions**: Maintains design intent across all device sizes

## Getting Started

### Installation

Simply copy the CSS variables into your project's stylesheet:

```css
:root {
  /* Base design values for desktop */
  --design-width: 1440;  /* Width of your design in Figma (px) */
  --design-height: 900;  /* Height of your design in Figma (px) */

  /* Maximum dimensions where scaling stops */
  --max-viewport-width: 1920px;  /* Scaling stops at this width */
  --max-viewport-height: 1080px;  /* Scaling stops at this height */

  /* Fluid ratio controls */
  --vw-ratio: 0.5;  /* How much the width affects scaling (0-1) */
  --vh-ratio: 0.3;  /* How much the height affects scaling (0-1) */

  /* The fluid calculation with max limit */
  --vw-contribution: calc(var(--vw-ratio) * (min(100vw, var(--max-viewport-width)) / var(--design-width)));
  --vh-contribution: calc(var(--vh-ratio) * (min(100vh, var(--max-viewport-height)) / var(--design-height)));

  /* Final fluid unit (equivalent to 1px in a fluid context) */
  --fluid: calc((var(--vw-contribution) + var(--vh-contribution)) / (var(--vw-ratio) + var(--vh-ratio)));
}
```

### Basic Setup

1. Update the `--design-width` and `--design-height` to match your design mockup dimensions
2. Set the `--max-viewport-width` and `--max-viewport-height` if you want to cap scaling
3. Adjust the `--vw-ratio` and `--vh-ratio` to control how much width vs height influences scaling

## Usage

Use the `--fluid` variable in your CSS by multiplying it with pixel values from your design:

```css
h1 {
  font-size: calc(var(--fluid) * 72);  /* 72px in design becomes fluid */
  margin-bottom: calc(var(--fluid) * 24);  /* 24px margin becomes fluid */
}

.container {
  width: calc(var(--fluid) * 1200);  /* 1200px width becomes fluid */
  padding: calc(var(--fluid) * 20);  /* 20px padding becomes fluid */
}

.card {
  border-radius: calc(var(--fluid) * 8);  /* 8px border radius becomes fluid */
}
```

## Configuration

### Key Variables

| Variable | Description | Default |
|----------|-------------|---------|
| `--design-width` | Width of your design mockup (px) | 1440 |
| `--design-height` | Height of your design mockup (px) | 900 |
| `--max-viewport-width` | Maximum width where scaling stops (px) | 1920px |
| `--max-viewport-height` | Maximum height where scaling stops (px) | 1080px |
| `--vw-ratio` | How much viewport width influences scaling (0-1) | 0.5 |
| `--vh-ratio` | How much viewport height influences scaling (0-1) | 0.3 |

### Adjusting Ratios

- **Higher `--vw-ratio`**: Elements scale more with width changes
- **Higher `--vh-ratio`**: Elements scale more with height changes
- **Equal ratios**: Balanced influence of width and height
- **Ratios sum to 1**: Creates a weighted average between width and height

## Breakpoints

The most important enhancement is updating design values at each breakpoint to reference device-specific mockups:

```css
/* Base desktop values */
:root {
  --design-width: 1440;  /* Desktop mockup width */
  --design-height: 900;  /* Desktop mockup height */
  --vw-ratio: 0.5;
  --vh-ratio: 0.3;
}

/* Tablet breakpoint */
@media only screen and (max-width: 991px) {
  :root {
    /* Update to match tablet mockup dimensions */
    --design-width: 768;   /* Tablet mockup width */
    --design-height: 1024; /* Tablet mockup height */
    --vw-ratio: 0.7;
    --vh-ratio: 0.2;
  }
}

/* Mobile landscape breakpoint */
@media screen and (max-width: 767px) {
  :root {
    /* Update to match mobile landscape mockup dimensions */
    --design-width: 667;   /* Mobile landscape mockup width */
    --design-height: 375;  /* Mobile landscape mockup height */
    --vw-ratio: 0.8;
    --vh-ratio: 0.1;
  }
}

/* Mobile portrait breakpoint */
@media screen and (max-width: 497px) {
  :root {
    /* Update to match mobile portrait mockup dimensions */
    --design-width: 375;   /* Mobile portrait mockup width */
    --design-height: 667;  /* Mobile portrait mockup height */
    --vw-ratio: 0.9;
    --vh-ratio: 0.05;
  }
}
```

### Why Update Design Values?

When a designer creates different mockups for different devices (desktop, tablet, mobile), these mockups often have different dimensions. Updating the design reference values at each breakpoint ensures your fluid scaling calculations correctly reference the appropriate "source of truth" for that device category.

This maintains the exact proportions intended in the design at each breakpoint while allowing smooth scaling between them.

## Advanced Techniques

### Adding Common Fluid Values

For convenience, you can create common fluid values:

```css
:root {
  /* Base fluid calculation */
  --fluid: calc((var(--vw-contribution) + var(--vh-contribution)) / (var(--vw-ratio) + var(--vh-ratio)));
  
  /* Common values */
  --fluid-2: calc(var(--fluid) * 2);
  --fluid-4: calc(var(--fluid) * 4);
  --fluid-8: calc(var(--fluid) * 8);
  --fluid-16: calc(var(--fluid) * 16);
  --fluid-24: calc(var(--fluid) * 24);
  --fluid-32: calc(var(--fluid) * 32);
  --fluid-48: calc(var(--fluid) * 48);
  --fluid-64: calc(var(--fluid) * 64);
}
```

### JavaScript Integration

For animations or dynamic elements, you can access the fluid value in JavaScript:

```javascript
// Helper function to convert design pixels to fluid values
function fluid(pixelValue) {
  const fluidValue = parseFloat(
    getComputedStyle(document.documentElement).getPropertyValue('--fluid')
  );
  return pixelValue * fluidValue;
}

// Use in animations or dynamic positioning
element.style.transform = `translateX(${fluid(100)}px)`;
```

### GSAP Integration

For animations with GSAP:

```javascript
// Helper function for fluid values in GSAP
function fluid(pixelValue) {
  return pixelValue * parseFloat(
    getComputedStyle(document.documentElement).getPropertyValue('--fluid')
  );
}

// Use in GSAP animations
gsap.to(".element", {
  x: fluid(200),
  y: fluid(50),
  width: fluid(300),
  duration: 1
});
```

### Accessibility Considerations

For better text zoom accessibility, use a hybrid approach:

```css
/* Don't override the root font size with fluid units */
html {
  /* Let the browser's font size be the base (default is usually 16px) */
  /* font-size remains at browser default or user setting */
}

/* Use rem for text elements to respect browser zoom */
h1 {
  font-size: 4.5rem; /* Will scale with browser zoom */
  margin-bottom: calc(var(--fluid) * 24); /* Layout can still use fluid units */
}

/* Use fluid units for layout elements */
.container {
  width: calc(var(--fluid) * 1200);
  padding: calc(var(--fluid) * 20);
}
```

## Browser Support

The Fluid CSS Methodology relies on:

- CSS Custom Properties (Variables)
- CSS calc() function
- CSS min() function

These features are supported in all modern browsers:
- Chrome 49+
- Firefox 42+
- Safari 9.1+
- Edge 15+

For legacy browser support, consider using a CSS preprocessor like Sass with a PostCSS plugin to compile the fluid calculations into static values.

---

## License

MIT

*Based on the fluid layout methodology by Willy Brauner.*
