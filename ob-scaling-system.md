# OFF+BRAND Fluid Scaling System

A lightweight, CSS-driven fluid scaling system that maintains design proportions across viewports. Perfect for Webflow projects that need precise scaling behavior.

## Features

- Pure CSS scaling (no runtime calculations)
- Maintains exact design proportions
- Smooth scaling between breakpoints
- Easy to implement
- Zero dependencies
- Minimal setup required

## Installation

### 1. Add the Base CSS

```css
:root {
    /* Core settings */
    --min-width: 992px;      /* Width where scaling starts */
    --max-width: 2560px;     /* Width where scaling stops */
    --design-width: 1440;    /* Your design's reference width */
    
    /* Scaling calculations */
    --fluid-container: clamp(var(--min-width), 100vw, var(--max-width));
    --fluid-font: calc(var(--fluid-container) / var(--design-width) * 16);
}

html {
    font-size: var(--fluid-font);
}
```

### 2. Configure Your Settings

Adjust the core variables to match your design:

```css
:root {
    --min-width: 992px;      /* Your minimum width */
    --max-width: 2560px;     /* Your maximum width */
    --design-width: 1440;    /* Your design width */
}
```

## Usage

### Converting Design Values

To maintain proper scaling, convert your design's pixel values to rem using this formula:

```
design-pixels ÷ 16 = rem value
```

For example:
```css
/* 160px in design */
.element {
    width: 10rem;      /* 160 ÷ 16 = 10 */
    height: 10rem;
    margin: 1rem;      /* 16 ÷ 16 = 1 */
    padding: 1.25rem;  /* 20 ÷ 16 = 1.25 */
}
```

Quick Reference:
- 100px → 6.25rem
- 160px → 10rem
- 200px → 12.5rem
- 300px → 18.75rem
- 400px → 25rem
- 500px → 31.25rem

### How Scaling Works

At different viewport widths:
- **At design width (1440px)**: Elements match exact design sizes
- **Between min and max**: Elements scale proportionally
- **Below min (992px)**: Elements maintain minimum size
- **Above max (2560px)**: Elements maintain maximum size

### Example

```html
<div class="card">
    <img class="card-image" src="image.jpg" alt="Card image">
    <div class="card-content">
        <h2 class="card-title">Title</h2>
        <p class="card-text">Content</p>
    </div>
</div>

<style>
.card {
    width: 25rem;          /* 400px in design */
    padding: 1.25rem;      /* 20px in design */
    margin-bottom: 1.5rem; /* 24px in design */
}

.card-image {
    height: 12.5rem;       /* 200px in design */
}

.card-title {
    font-size: 1.5rem;     /* 24px in design */
    margin-bottom: 1rem;   /* 16px in design */
}

.card-text {
    font-size: 1rem;       /* 16px in design */
    line-height: 1.5;
}
</style>
```

## Webflow Implementation

1. Add the scaling CSS to your project settings
2. Use rem units for all size values
3. Design at 1440px width
4. Test at various viewport sizes

## Best Practices

1. **Always Use REM Units**
   ```css
   /* ✅ Correct */
   .element { width: 10rem; }
   
   /* ❌ Incorrect */
   .element { width: 160px; }
   ```

2. **Maintain Proportions**
   ```css
   /* ✅ Correct */
   .container { max-width: 90rem; }
   
   /* ❌ Incorrect */
   .container { max-width: 1440px; }
   ```

3. **Font Sizes**
   ```css
   /* ✅ Correct */
   .title { font-size: 2rem; }
   
   /* ❌ Incorrect */
   .title { font-size: 32px; }
   ```

## Browser Support

- Chrome 79+
- Firefox 69+
- Safari 14+
- Edge 79+

## Common Issues

### Elements Not Scaling

✅ Solution: Ensure you're using rem units for all measurements

```css
/* Not scaling */
.element { width: 100px; }

/* Will scale properly */
.element { width: 6.25rem; }
```

### Layout Shifts

✅ Solution: Check for mixed units

```css
/* Might cause shifts */
.container {
    width: 90%;
    padding: 20px;
}

/* Consistent scaling */
.container {
    width: 90%;
    padding: 1.25rem;
}
```

### Font Size Issues

✅ Solution: Use rem for typography

```css
/* Inconsistent scaling */
h1 { font-size: 32px; }

/* Consistent scaling */
h1 { font-size: 2rem; }
```

## Tips

1. Use a calculator for px to rem conversions
2. Test at various viewport widths
3. Maintain consistent units throughout your project
4. Design at 1440px for optimal results
5. Consider browser zoom behavior

## License

MIT License - Free to use in personal and commercial projects.

---

Created by OFF+BRAND - Fluid design precision for Webflow experts
