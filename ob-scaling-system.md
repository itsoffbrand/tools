# OFF+BRAND Scaling System v0.0.2

A fluid typography and scaling system that maintains design fidelity while ensuring accessibility. This system provides smooth scaling between viewport widths without layout shifts or the need for breakpoint adjustments.

## Features

- Fluid typography scaling
- Maintains design proportions across viewports
- Accessibility-first approach
- Browser zoom support
- High contrast mode support
- No layout shifts
- Single source of truth for scaling values

## Installation

### 1. Add the CSS variables and scaling logic to your stylesheet:

```css
/* Base values */
:root {
  --min-width: 991;
  --max-width: 2560;
  --design-width: 1440;
  --min-font: 0.6881944444444444;
  --max-font: 1.777777777777778;
  --user-font-size: 16px;
  
  --fluid-bp: calc((100vw - var(--min-width) * 1px) / (var(--max-width) - var(--min-width)));
  --fluid-scale: calc(var(--min-font) + (var(--max-font) - var(--min-font)) * var(--fluid-bp));
}

/* Apply scaling to html element */
html {
  font-size: clamp(
    calc(var(--min-font) * 16px),
    calc(var(--fluid-scale) * 16px),
    calc(var(--max-font) * 16px)
  );
}

/* Reset body font-size */
body {
  font-size: 1rem;
}

/* Support Windows High Contrast mode */
@media screen and (forced-colors: active) {
  html {
    font-size: var(--user-font-size);
  }
}

/* Container setup */
.container {
  max-width: min(90em, 95vw);
  margin-inline: auto;
  padding-inline: max(1rem, 2vw);
}
```

### 2. Add the JavaScript to handle resize events:

```javascript
document.addEventListener('DOMContentLoaded', () => {
  const resizeObserver = new ResizeObserver(entries => {
    for (const entry of entries) {
      const width = entry.contentRect.width;
      
      const minWidth = parseFloat(getComputedStyle(document.documentElement)
        .getPropertyValue('--min-width'));
      const maxWidth = parseFloat(getComputedStyle(document.documentElement)
        .getPropertyValue('--max-width'));
      
      if (width < minWidth || width > maxWidth) {
        const baseFontSize = parseFloat(getComputedStyle(document.documentElement).fontSize);
        document.documentElement.style.setProperty('--user-font-size', `${baseFontSize}px`);
        
        document.documentElement.style.setProperty(
          '--fluid-bp',
          Math.max(0, Math.min(1, (width - minWidth) / (maxWidth - minWidth)))
        );
      }
    }
  });

  resizeObserver.observe(document.documentElement);

  window.addEventListener('resize', () => {
    const baseFontSize = parseFloat(getComputedStyle(document.documentElement).fontSize);
    document.documentElement.style.setProperty('--user-font-size', `${baseFontSize}px`);
  });
});
```

## Configuration

Adjust the CSS variables to match your design requirements:

- `--min-width`: Viewport width where scaling starts (default: 991)
- `--max-width`: Viewport width where scaling stops (default: 2560)
- `--design-width`: Your design's reference width (default: 1440)
- `--min-font`: Minimum font size in rem (default: 0.6881944444444444)
- `--max-font`: Maximum font size in rem (default: 1.777777777777778)

## Converting Design Values

Convert your design's pixel values to rem using this formula:

```
design-pixels ÷ 16 = rem value
```

Example conversions:
- 100px → 6.25rem
- 200px → 12.5rem
- 300px → 18.75rem
- 400px → 25rem
- 500px → 31.25rem

## Implementation Example

Instead of using pixel values:

```css
/* ❌ Don't do this */
.element {
  width: 300px;
  margin: 24px;
  padding: 16px;
}
```

Use rem values:

```css
/* ✅ Do this */
.element {
  width: 18.75rem;    /* 300px ÷ 16 */
  margin: 1.5rem;     /* 24px ÷ 16 */
  padding: 1rem;      /* 16px ÷ 16 */
}
```

## How It Works

1. The system calculates a fluid scaling factor based on viewport width
2. This scaling is applied to the html element's font-size
3. All rem units in your CSS will scale proportionally
4. The scaling is smooth between your minimum and maximum viewport widths
5. JavaScript handles edge cases and accessibility features

## Accessibility Features

The OFF+BRAND Scaling System includes several accessibility features:

- Respects user's browser zoom settings
- Maintains readability in Windows High Contrast mode
- Supports screen readers
- Honors browser minimum font size settings
- Maintains proper spacing when zoomed

## Browser Support

- Chrome 79+
- Firefox 69+
- Safari 14+
- Edge 79+

## Troubleshooting

### Common Issues

1. **Layout Shifts**
   - Ensure all measurements are in rem units
   - Check that the html element has the scaling applied (not :root)
   - Verify body font-size is set to 1rem

2. **Scaling Issues**
   - Verify your `--design-width` matches your design file
   - Check min/max width values
   - Ensure all pixel values are converted to rem
   - Confirm the JavaScript is properly initialized

3. **Browser Zoom Problems**
   - Check if the ResizeObserver is properly initialized
   - Verify the `--user-font-size` is being updated
   - Test zooming at different viewport widths

4. **High Contrast Mode**
   - Test with Windows High Contrast mode
   - Verify media query support
   - Check font-size fallbacks

### Testing Checklist

- [ ] Test at minimum viewport width (991px)
- [ ] Test at design width (1440px)
- [ ] Test at maximum viewport width (2560px)
- [ ] Verify browser zoom functionality
- [ ] Check high contrast mode display
- [ ] Test with screen readers
- [ ] Verify smooth scaling between viewport sizes

## License

MIT License - Feel free to use in personal and commercial projects.

---

Created by OFF+BRAND - Fluid design precision for Webflow experts

Version 0.0.2 - Updated 2025
