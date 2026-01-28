# Styling Guide

Modern styling patterns for Vue/Nuxt components using UnoCSS with `:uno:` prefix and design tokens.

---

## UnoCSS Color Tokens

### Available Color Palettes

The project uses design tokens from `@paradoxai/design-tokens` with the following color palettes:

**Semantic Colors:**
- `primary` - Primary brand color (#25c9d0)
- `success` - Success states (#39d279)
- `danger` - Error/danger states (#e52d2d)
- `warning` - Warning states (#f5a623)
- `info` - Informational states

**Base Colors:**
- `grey` - Neutral grayscale colors
- `blue` - Blue color palette
- `red` - Red color palette
- `green` - Green color palette
- `yellow` - Yellow color palette

### Color Shades

Each color palette includes shades from 50 (lightest) to 900 (darkest):
- `50`, `100`, `200`, `300`, `400`, `500` (default), `600`, `700`, `800`, `900`

### Color Usage Examples

```vue
<template>
  <!-- Background colors -->
  <div class=":uno: bg-primary">Primary background</div>
  <div class=":uno: bg-primary-500">Primary 500 shade</div>
  <div class=":uno: bg-grey-100">Light grey background</div>
  <div class=":uno: bg-success-50">Light success background</div>
  <div class=":uno: bg-danger-600">Danger background</div>
  <div class=":uno: bg-warning-100">Warning background</div>

  <!-- Text colors -->
  <div class=":uno: text-primary">Primary text</div>
  <div class=":uno: text-grey-900">Dark grey text</div>
  <div class=":uno: text-grey-400">Medium grey text</div>
  <div class=":uno: text-success">Success text</div>
  <div class=":uno: text-danger-600">Danger text</div>
  <div class=":uno: color-grey-900">Alternative text color syntax</div>

  <!-- Border colors -->
  <div class=":uno: border border-grey-400">Grey border</div>
  <div class=":uno: border-primary">Primary border</div>
  <div class=":uno: border-danger-500">Danger border</div>
  <div class=":uno: border-t border-grey-300">Top border only</div>

  <!-- Icon colors -->
  <i class=":uno: i-ol-icon icon:text-primary">Primary icon</i>
  <i class=":uno: i-ol-icon icon:text-grey-900">Dark icon</i>
  <i class=":uno: i-ol-icon icon:color-grey-500">Grey icon</i>
</template>
```

### Complete Color Token Reference

| Color | Shades Available | Common Usage |
|-------|----------------|--------------|
| `primary` | 50-900 | Brand color, links, primary actions |
| `success` | 50-900 | Success messages, positive states |
| `danger` | 50-900 | Errors, destructive actions |
| `warning` | 50-900 | Warnings, caution states |
| `info` | 50-900 | Informational messages |
| `grey` | 50-900 | Text, borders, backgrounds |
| `blue` | 50-900 | Blue accents |
| `red` | 50-900 | Red accents |
| `green` | 50-900 | Green accents |
| `yellow` | 50-900 | Yellow accents |

**Examples:**
- `bg-primary`, `bg-primary-100`, `bg-primary-500`, `bg-primary-900`
- `text-grey-50`, `text-grey-400`, `text-grey-900`
- `border-success-300`, `border-danger-600`
- `icon:text-primary`, `icon:color-grey-500`

---

## UnoCSS Usage Patterns

### Basic Usage with `:uno:` Prefix

All UnoCSS classes must be prefixed with `:uno:` in Vue templates:

```vue
<template>
  <div class=":uno: p-4 mb-3 flex items-center gap-2">
    <span class=":uno: text-grey-900 font-semibold">Title</span>
    <button class=":uno: bg-primary text-white px-4 py-2 rounded">
      Click me
    </button>
  </div>
</template>
```

### Responsive Styles

```vue
<template>
  <div class=":uno: p-2 md:p-4 lg:p-6">
    <div class=":uno: flex-col md:flex-row">
      <div class=":uno: w-full md:w-1/2">Responsive content</div>
    </div>
  </div>
</template>
```

### Pseudo-Selectors and States

```vue
<template>
  <div class=":uno: p-4 hover:bg-grey-100 active:bg-grey-200">
    <button class=":uno: bg-primary hover:bg-primary-600 focus:ring-2 focus:ring-primary-500">
      Interactive Button
    </button>
  </div>
</template>
```

### Child Selectors with `all-[]`

```vue
<template>
  <div class=":uno: all-[.el-button]:bg-primary all-[.el-button]:text-white">
    <button class="el-button">Styled button</button>
  </div>
</template>
```

---

## Inline vs Separate Styles

### Decision Threshold

**<100 lines: Inline styles in component**

For Vue components, prefer UnoCSS utility classes with `:uno:` prefix. If you need complex styles, use inline style objects or computed classes:

```vue
<script setup lang="ts">
const containerClass = computed(() => [
  ':uno: p-4',
  ':uno: flex',
  ':uno: flex-col',
  isActive ? ':uno: bg-primary-50' : ':uno: bg-white',
])
</script>

<template>
  <div :class="containerClass">
    Content
  </div>
</template>
```

**>100 lines: Separate style module or composable**

For complex styling logic, extract to a composable or use CSS modules:

```vue
<!-- MyComponent.vue -->
<script setup lang="ts">
import { useComponentStyles } from './useComponentStyles'

const styles = useComponentStyles()
</script>

<template>
  <div :class="styles.container">
    Content
  </template>
</template>
```

---

## Common Style Patterns

### Flexbox Layout

```vue
<template>
  <!-- Flex row -->
  <div class=":uno: flex items-center gap-2">
    <span>Item 1</span>
    <span>Item 2</span>
  </div>

  <!-- Flex column -->
  <div class=":uno: flex flex-col gap-4">
    <div>Row 1</div>
    <div>Row 2</div>
  </div>

  <!-- Space between -->
  <div class=":uno: flex justify-between items-center">
    <span>Left</span>
    <span>Right</span>
  </div>
</template>
```

### Spacing

```vue
<template>
  <!-- Padding -->
  <div class=":uno: p-4">All sides (16px)</div>
  <div class=":uno: px-4 py-2">Horizontal and vertical</div>
  <div class=":uno: pt-2 pr-4 pb-2 pl-4">Specific sides</div>
  <div class=":uno: p-x-12.5 p-y-10">Custom spacing (50px, 40px)</div>

  <!-- Margin -->
  <div class=":uno: m-4 mx-2 my-3 mt-2 mb-4">Margin variants</div>
  <div class=":uno: mb-4">Bottom margin</div>
</template>
```

### Positioning

```vue
<template>
  <div class=":uno: relative">
    <div class=":uno: absolute top-0 right-0">Absolute positioned</div>
  </div>

  <div class=":uno: sticky top-0 z-10">Sticky header</div>
</template>
```

### Borders and Rounded Corners

```vue
<template>
  <div class=":uno: border border-grey-400 rounded">Basic border</div>
  <div class=":uno: border-t border-grey-300">Top border only</div>
  <div class=":uno: rounded-1 rounded-2 rounded-lg">Border radius variants</div>
  <div class=":uno: border-1 border-t-1">Border width</div>
</template>
```

### Shadows

```vue
<template>
  <div class=":uno: shadow-[0_0_8px_rgba(0,0,0,0.05),0_2px_4px_rgba(0,0,0,0.1)]">
    Custom shadow
  </div>
</template>
```

---

## Typography

### Font Sizes

```vue
<template>
  <div class=":uno: text-xs">Extra small</div>
  <div class=":uno: text-sm">Small</div>
  <div class=":uno: text-base">Base</div>
  <div class=":uno: text-md">Medium</div>
  <div class=":uno: text-lg">Large</div>
  <div class=":uno: text-xl">Extra large</div>
  <div class=":uno: text-2xl">2XL</div>
  <div class=":uno: text-3xl">3XL</div>
  <div class=":uno: text-[16px]">Custom size</div>
</template>
```

### Font Weights

```vue
<template>
  <div class=":uno: font-normal">Normal</div>
  <div class=":uno: font-semibold">Semibold</div>
  <div class=":uno: font-bold">Bold</div>
  <div class=":uno: font-primary">Primary font family</div>
</template>
```

### Text Utilities

```vue
<template>
  <div class=":uno: text-center">Centered</div>
  <div class=":uno: truncate">Truncated text</div>
  <div class=":uno: uppercase">Uppercase</div>
  <div class=":uno: whitespace-nowrap">No wrap</div>
</template>
```

---

## Component Styling Examples

### Card Component

```vue
<template>
  <div class=":uno: bg-white rounded-lg border border-grey-300 p-4 shadow-sm">
    <h3 class=":uno: text-xl font-semibold text-grey-900 mb-2">Card Title</h3>
    <p class=":uno: text-grey-600">Card content</p>
  </div>
</template>
```

### Button Variants

```vue
<template>
  <!-- Primary button -->
  <button class=":uno: bg-primary text-white px-4 py-2 rounded hover:bg-primary-600">
    Primary
  </button>

  <!-- Secondary button -->
  <button class=":uno: bg-grey-200 text-grey-900 px-4 py-2 rounded hover:bg-grey-300">
    Secondary
  </button>

  <!-- Danger button -->
  <button class=":uno: bg-danger-600 text-white px-4 py-2 rounded hover:bg-danger-700">
    Delete
  </button>

  <!-- Disabled button -->
  <button class=":uno: bg-grey-300 text-grey-500 px-4 py-2 rounded cursor-not-allowed" disabled>
    Disabled
  </button>
</template>
```

### Form Inputs

```vue
<template>
  <input
    class=":uno: border border-grey-400 rounded px-3 py-2 focus:border-primary focus:ring-2 focus:ring-primary-500"
    placeholder="Enter text"
  />
</template>
```

---

## Code Style Standards

### Indentation

**2 spaces** for Vue templates and TypeScript (project standard)

```vue
<template>
  <div class=":uno: p-4">
    <span>Content</span>
  </div>
</template>
```

### Quotes

**Single quotes** for strings (project standard)

```vue
<script setup lang="ts">
const className = ':uno: p-4'
const message = 'Hello world'
</script>
```

### Trailing Commas

**Always use trailing commas** in objects and arrays

```vue
<script setup lang="ts">
const styles = {
  container: ':uno: p-4',
  header: ':uno: mb-2',  // Trailing comma
}

const items = [
  'item1',
  'item2',  // Trailing comma
]
</script>
```

---

## What NOT to Use

### ❌ Inline Style Objects (unless necessary)

```vue
<!-- ❌ AVOID - Use UnoCSS classes instead -->
<div :style="{ padding: '16px', backgroundColor: '#fff' }">
  Content
</div>

<!-- ✅ PREFERRED -->
<div class=":uno: p-4 bg-white">
  Content
</div>
```

### ❌ CSS Modules for Simple Styles

```vue
<!-- ❌ AVOID - For simple utility classes -->
<div :class="$style.container">
  Content
</div>

<!-- ✅ PREFERRED - Use UnoCSS -->
<div class=":uno: p-4 bg-white rounded">
  Content
</div>
```

### ✅ Use CSS Modules Only For

- Complex animations
- Third-party library overrides
- Global style resets
- Component-specific complex selectors

---

## Advanced Patterns

### Conditional Classes

```vue
<script setup lang="ts">
const isActive = ref(false)
const variant = ref<'primary' | 'secondary'>('primary')

const buttonClass = computed(() => [
  ':uno: px-4 py-2 rounded',
  isActive.value ? ':uno: bg-primary-600' : ':uno: bg-primary',
  variant.value === 'secondary' ? ':uno: bg-grey-200 text-grey-900' : '',
])
</script>

<template>
  <button :class="buttonClass">Button</button>
</template>
```

### Dynamic Color Classes

```vue
<script setup lang="ts">
const status = ref<'success' | 'danger' | 'warning'>('success')

const statusClass = computed(() => {
  const colorMap = {
    success: ':uno: bg-success-50 text-success-900 border-success-300',
    danger: ':uno: bg-danger-50 text-danger-900 border-danger-300',
    warning: ':uno: bg-warning-50 text-warning-900 border-warning-300',
  }
  return colorMap[status.value]
})
</script>

<template>
  <div :class="[':uno: p-4 rounded border', statusClass]">
    Status message
  </div>
</template>
```

### Icon Styling

```vue
<template>
  <!-- Icon with color -->
  <i class=":uno: i-ol-icon icon:text-primary"></i>
  <i class=":uno: i-ol-icon icon:color-grey-900"></i>
  <i class=":uno: i-ol-icon icon:(w-6 h-6 text-primary)"></i>

  <!-- Icon with background -->
  <i class=":uno: i-ol-icon !bg-primary text-white"></i>
</template>
```

---

## Summary

**Styling Checklist:**
- ✅ Use `:uno:` prefix for all UnoCSS classes
- ✅ Use color tokens: `primary`, `success`, `danger`, `warning`, `info`, `grey`, `blue`, `red`, `green`, `yellow`
- ✅ Use color shades: `50`, `100`, `200`, `300`, `400`, `500`, `600`, `700`, `800`, `900`
- ✅ 2 space indentation
- ✅ Single quotes
- ✅ Trailing commas
- ✅ Prefer UnoCSS utilities over inline styles
- ❌ Avoid inline style objects for simple styling

**Color Token Quick Reference:**
- Background: `bg-{color}-{shade}` (e.g., `bg-primary-500`, `bg-grey-100`)
- Text: `text-{color}-{shade}` or `color-{color}-{shade}` (e.g., `text-grey-900`, `color-primary`)
- Border: `border-{color}-{shade}` (e.g., `border-grey-400`, `border-primary`)
- Icon: `icon:text-{color}-{shade}` or `icon:color-{color}-{shade}` (e.g., `icon:text-primary`, `icon:color-grey-500`)

**See Also:**
- [component-patterns.md](component-patterns.md) - Component structure
- [complete-examples.md](complete-examples.md) - Full styling examples
