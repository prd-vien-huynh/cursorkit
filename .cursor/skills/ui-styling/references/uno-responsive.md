# UnoCSS Responsive Design

Mobile-first breakpoints, responsive utilities, and adaptive layouts using UnoCSS in Nuxt 3.

---

## Mobile-First Approach

UnoCSS uses mobile-first responsive design. Base styles apply to all screen sizes, then use breakpoint prefixes to override at larger sizes.

```vue
<template>
  <!-- Base: 1 column (mobile)
       sm: 2 columns (tablet)
       lg: 4 columns (desktop) -->
  <div class=":uno: grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-4">
    <div>Item 1</div>
    <div>Item 2</div>
    <div>Item 3</div>
    <div>Item 4</div>
  </div>
</template>
```

**Note:** The `:uno:` prefix is used to compile UnoCSS classes. See `areas/common/uno.config.js` for configuration.

---

## Breakpoint System

**Project breakpoints (custom):**

| Prefix | Min Width | CSS Media Query | Use Case |
|--------|-----------|-----------------|----------|
| `xs:` | 576px | `@media (min-width: 576px)` | Extra small devices |
| `sm:` | 768px | `@media (min-width: 768px)` | Small devices (tablets) |
| `md:` | 992px | `@media (min-width: 992px)` | Medium devices |
| `lg:` | 1200px | `@media (min-width: 1200px)` | Large devices (desktop) |
| `xl:` | 1440px | `@media (min-width: 1440px)` | Extra large devices |

**Reference:** See `packages/ui/stories/base/breakpoint.story.md` and `src/static/site/sass/_media-queries.scss` for breakpoint definitions.

---

## Responsive Patterns

### Layout Changes

```vue
<template>
  <!-- Vertical on mobile, horizontal on desktop -->
  <div class=":uno: flex flex-col lg:flex-row gap-4">
    <div>Left</div>
    <div>Right</div>
  </div>

  <!-- 1 column -> 2 columns -> 3 columns -->
  <div class=":uno: grid grid-cols-1 md:grid-cols-2 xl:grid-cols-3 gap-6">
    <div>Item 1</div>
    <div>Item 2</div>
    <div>Item 3</div>
  </div>
</template>
```

**Reference:** See `areas/common/app/components/page/page.vue` for layout patterns.

### Visibility

```vue
<template>
  <!-- Hide on mobile, show on desktop -->
  <div class=":uno: hidden lg:block">
    Desktop only content
  </div>

  <!-- Show on mobile, hide on desktop -->
  <div class=":uno: block lg:hidden">
    Mobile only content
  </div>

  <!-- Different content per breakpoint -->
  <div class=":uno: lg:hidden">Mobile menu</div>
  <div class=":uno: hidden lg:flex">Desktop navigation</div>
</template>
```

### Typography

```vue
<template>
  <!-- Responsive text sizes -->
  <h1 class=":uno: text-2xl md:text-4xl lg:text-6xl font-bold">
    Heading scales with screen size
  </h1>

  <p class=":uno: text-sm md:text-base lg:text-lg">
    Body text scales appropriately
  </p>
</template>
```

### Spacing

```vue
<template>
  <!-- Responsive padding -->
  <div class=":uno: p-4 md:p-6 lg:p-8">
    More padding on larger screens
  </div>

  <!-- Responsive gap -->
  <div class=":uno: flex gap-2 md:gap-4 lg:gap-6">
    <div>Item 1</div>
    <div>Item 2</div>
  </div>
</template>
```

**Reference:** See `areas/common/app/components/page/page-body.vue` for spacing patterns.

### Width

```vue
<template>
  <!-- Full width on mobile, constrained on desktop -->
  <div class=":uno: w-full lg:w-1/2 xl:w-1/3">
    Responsive width
  </div>

  <!-- Responsive max-width -->
  <div class=":uno: max-w-sm md:max-w-2xl lg:max-w-4xl mx-auto">
    Centered with responsive max width
  </div>
</template>
```

---

## Max-Width Queries

Apply styles only below certain breakpoint using `max-*:` prefix or `<*:` syntax:

```vue
<template>
  <!-- Only on mobile and tablet (below 992px) -->
  <div class=":uno: max-md:text-center">
    Centered on mobile/tablet, left-aligned on desktop
  </div>

  <!-- Only on mobile (below 768px) -->
  <div class=":uno: max-sm:hidden">
    Hidden only on mobile
  </div>

  <!-- Alternative: Less than syntax -->
  <div class=":uno: <sm:min-h-12">
    Minimum height only on mobile
  </div>
</template>
```

**Available max-width prefixes:**
- `max-xs:` - Below 576px
- `max-sm:` - Below 768px
- `max-md:` - Below 992px
- `max-lg:` - Below 1200px
- `max-xl:` - Below 1440px

**Alternative less-than syntax:**
- `<xs:` - Less than 576px
- `<sm:` - Less than 768px
- `<md:` - Less than 992px
- `<lg:` - Less than 1200px
- `<xl:` - Less than 1440px

**Reference:** See `areas/event/app/components/filter/item/filter-by-date.vue` for `<sm:` usage.

---

## Variant Groups

UnoCSS supports variant groups with parentheses for cleaner syntax:

```vue
<template>
  <!-- Group multiple utilities at same breakpoint -->
  <div class=":uno: sm:(flex flex-col gap-4 p-6)">
    Content with grouped responsive utilities
  </div>

  <!-- Multiple breakpoint groups -->
  <div class=":uno: sm:(flex flex-col) lg:(flex-row gap-8)">
    Different groups per breakpoint
  </div>
</template>
```

**Reference:** See `areas/common/app/components/page/page-content.vue` for variant group patterns.

---

## Common Responsive Layouts

### Sidebar Layout

```vue
<template>
  <div class=":uno: flex flex-col lg:flex-row min-h-screen">
    <!-- Sidebar: Full width on mobile, fixed on desktop -->
    <aside class=":uno: w-full lg:w-64 bg-grey-100 p-4">
      Sidebar
    </aside>

    <!-- Main content -->
    <main class=":uno: flex-1 p-4 md:p-8">
      Main content
    </main>
  </div>
</template>
```

**Reference:** See `areas/common/app/components/page/page-content.vue` for sidebar layout patterns.

### Card Grid

```vue
<template>
  <div class=":uno: grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-3 xl:grid-cols-4 gap-4 md:gap-6">
    <div class=":uno: bg-white rounded-lg shadow p-6">Card 1</div>
    <div class=":uno: bg-white rounded-lg shadow p-6">Card 2</div>
    <div class=":uno: bg-white rounded-lg shadow p-6">Card 3</div>
    <div class=":uno: bg-white rounded-lg shadow p-6">Card 4</div>
  </div>
</template>
```

### Hero Section

```vue
<template>
  <section class=":uno: py-12 md:py-20 lg:py-32">
    <div class=":uno: container mx-auto px-4">
      <div class=":uno: flex flex-col lg:flex-row items-center gap-8 lg:gap-12">
        <div class=":uno: flex-1 text-center lg:text-left">
          <h1 class=":uno: text-4xl md:text-5xl lg:text-6xl font-bold mb-4">
            Hero Title
          </h1>
          <p class=":uno: text-lg md:text-xl mb-6">
            Hero description
          </p>
          <button class=":uno: px-6 py-3 md:px-8 md:py-4">
            CTA Button
          </button>
        </div>
        <div class=":uno: flex-1">
          <img src="hero.jpg" class=":uno: w-full rounded-lg" />
        </div>
      </div>
    </div>
  </section>
</template>
```

### Navigation

```vue
<template>
  <nav class=":uno: bg-white shadow">
    <div class=":uno: container mx-auto px-4">
      <div class=":uno: flex items-center justify-between h-16">
        <div class=":uno: text-xl font-bold">Logo</div>

        <!-- Desktop navigation -->
        <div class=":uno: hidden md:flex gap-6">
          <a href="#">Home</a>
          <a href="#">About</a>
          <a href="#">Services</a>
          <a href="#">Contact</a>
        </div>

        <!-- Mobile menu button -->
        <button class=":uno: md:hidden">
          <svg class=":uno: w-6 h-6">...</svg>
        </button>
      </div>
    </div>
  </nav>
</template>
```

---

## Range Queries

Apply styles between breakpoints:

```vue
<template>
  <!-- Only on tablets (between md and lg) -->
  <div class=":uno: md:block lg:hidden">
    Visible only on tablets
  </div>

  <!-- Between sm and xl -->
  <div class=":uno: sm:grid-cols-2 xl:grid-cols-4">
    2 columns on tablet, 4 on extra large
  </div>
</template>
```

---

## Container Queries

Style elements based on parent container width:

```vue
<template>
  <div class=":uno: @container">
    <div class=":uno: @md:grid-cols-2 @lg:grid-cols-3">
      Responds to parent width, not viewport
    </div>
  </div>
</template>
```

Container query breakpoints: `@xs:` `@sm:` `@md:` `@lg:` `@xl:`

---

## Responsive State Variants

Combine responsive with hover/focus:

```vue
<template>
  <!-- Hover effect only on desktop -->
  <button class=":uno: lg:hover:scale-105">
    Scale on hover (desktop only)
  </button>

  <!-- Different hover colors per breakpoint -->
  <a class=":uno: hover:text-blue-600 lg:hover:text-purple-600">
    Link
  </a>
</template>
```

---

## Advanced Patterns

### Conditional Grid Columns

```vue
<script setup lang="ts">
const isWide = ref(true)
</script>

<template>
  <!-- Dynamic grid columns based on state -->
  <div 
    class=":uno: sm:(grid grid-cols-$grid-cols)"
    :class="{ 'grid-cols-none': !isWide }"
  >
    Content
  </div>
</template>
```

**Reference:** See `areas/candidate-shared/app/components/candidate-board/board-tabs.vue` for conditional grid patterns.

### Responsive Padding with Variant Groups

```vue
<template>
  <!-- Grouped responsive padding -->
  <div class=":uno: p-md sm:px-xl">
    Responsive padding with groups
  </div>

  <!-- Complex grouped utilities -->
  <div class=":uno: max-sm:all-[.ol-drawer\\_\\_header-wrapper]:(justify-center max-sm:p-4! max-sm:h-16!)">
    Nested responsive utilities
  </div>
</template>
```

**Reference:** See `areas/scheduling/app/components/my-calendar/edit-history/edit-history-detail-drawer.vue` for complex responsive patterns.

### Less Than Breakpoint Syntax

```vue
<template>
  <!-- Less than small breakpoint -->
  <div class=":uno: <sm:min-h-12">
    Minimum height only below 768px
  </div>

  <!-- Less than with variant groups -->
  <div class=":uno: <sm:(px-8 py-6)">
    Padding only on mobile
  </div>
</template>
```

**Reference:** See `areas/event/app/components/filter/item/filter-by-date.vue` for `<sm:` usage.

---

## Best Practices

### 1. Mobile-First Design

Start with mobile styles, add complexity at larger breakpoints:

```vue
<template>
  <!-- Good: Mobile first -->
  <div class=":uno: text-base md:text-lg lg:text-xl">
    Text scales up
  </div>

  <!-- Avoid: Desktop first -->
  <div class=":uno: text-xl lg:text-base">
    Text scales down (confusing)
  </div>
</template>
```

### 2. Consistent Breakpoint Usage

Use same breakpoints across related elements:

```vue
<template>
  <div class=":uno: grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-4 md:gap-6 lg:gap-8">
    Spacing scales with layout
  </div>
</template>
```

### 3. Test at Breakpoint Boundaries

Test at exact breakpoint widths (576px, 768px, 992px, 1200px, 1440px) to catch edge cases.

### 4. Use Variant Groups for Cleaner Code

```vue
<template>
  <!-- Good: Grouped utilities -->
  <div class=":uno: sm:(flex flex-col gap-4 p-6)">
    Clean and readable
  </div>

  <!-- Avoid: Repetitive prefixes -->
  <div class=":uno: sm:flex sm:flex-col sm:gap-4 sm:p-6">
    Repetitive and harder to read
  </div>
</template>
```

### 5. Progressive Enhancement

Ensure core functionality works on mobile, enhance for larger screens:

```vue
<template>
  <!-- Core layout works on mobile -->
  <div class=":uno: p-4">
    <!-- Enhanced spacing on desktop -->
    <div class=":uno: lg:p-8">
      Content
    </div>
  </div>
</template>
```

### 6. Avoid Too Many Breakpoints

Use 2-3 breakpoints per element for maintainability:

```vue
<template>
  <!-- Good: 2 breakpoints -->
  <div class=":uno: grid-cols-1 md:grid-cols-2 lg:grid-cols-4">
    Simple and maintainable
  </div>

  <!-- Avoid: Too many breakpoints -->
  <div class=":uno: grid-cols-1 xs:grid-cols-2 sm:grid-cols-3 md:grid-cols-4 lg:grid-cols-5 xl:grid-cols-6">
    Too complex
  </div>
</template>
```

### 7. Use `:uno:` Prefix Consistently

Always use the `:uno:` prefix for UnoCSS classes in Vue templates:

```vue
<template>
  <!-- Correct: With :uno: prefix -->
  <div class=":uno: flex flex-col sm:flex-row">
    Content
  </div>

  <!-- Also works but less explicit -->
  <div class="flex flex-col sm:flex-row">
    Content
  </div>
</template>
```

**Reference:** See `areas/common/uno.config.js` for UnoCSS configuration including the `:uno:` prefix transformer.

---

## Common Responsive Utilities

### Responsive Display

```vue
<template>
  <div class=":uno: block md:flex lg:grid">
    Changes display type per breakpoint
  </div>
</template>
```

### Responsive Position

```vue
<template>
  <div class=":uno: relative lg:absolute">
    Positioned differently per breakpoint
  </div>
</template>
```

### Responsive Order

```vue
<template>
  <div class=":uno: flex flex-col">
    <div class=":uno: order-2 lg:order-1">First on desktop</div>
    <div class=":uno: order-1 lg:order-2">First on mobile</div>
  </div>
</template>
```

### Responsive Overflow

```vue
<template>
  <div class=":uno: overflow-auto lg:overflow-visible">
    Scrollable on mobile, expanded on desktop
  </div>
</template>
```

---

## Project-Specific Patterns

### Page Layout Pattern

```vue
<template>
  <!-- Common page layout with responsive grid -->
  <div class=":uno: flex flex-col flex-1 gap-2 p-md lt-sm:p-0 sm:(grid grid-cols-$nav-cols gap-4)">
    <aside class=":uno: flex flex-col">
      Sidebar
    </aside>
    <main class=":uno: flex flex-col flex-1 overflow-x-hidden">
      Main content
    </main>
  </div>
</template>
```

**Reference:** See `areas/common/app/components/page/page.vue` for page layout patterns.

### Responsive Body Padding

```vue
<template>
  <!-- Responsive padding with max-width queries -->
  <div class=":uno: px-md sm:px-xl h-full">
    Content with responsive horizontal padding
  </div>
</template>
```

**Reference:** See `areas/common/app/components/page/page-body.vue` for body padding patterns.

### Conditional Grid Based on State

```vue
<script setup lang="ts">
const isWide = ref(true)
</script>

<template>
  <div 
    class=":uno: sm:(grid grid-cols-$grid-cols)"
    :class="{ 'grid-cols-none': !isWide }"
  >
    Grid that adapts to state
  </div>
</template>
```

**Reference:** See `areas/candidate-shared/app/components/candidate-board/board-tabs.vue` for conditional grid patterns.

---

## Testing Checklist

- [ ] Test at 320px (small mobile)
- [ ] Test at 576px (xs breakpoint)
- [ ] Test at 768px (sm breakpoint - tablet)
- [ ] Test at 992px (md breakpoint)
- [ ] Test at 1200px (lg breakpoint - desktop)
- [ ] Test at 1440px (xl breakpoint - large desktop)
- [ ] Test landscape orientation
- [ ] Verify touch targets (min 44x44px)
- [ ] Check text readability at all sizes
- [ ] Verify navigation works on mobile
- [ ] Test with browser zoom
- [ ] Test at breakpoint boundaries (575px, 767px, 991px, 1199px, 1439px)

---

## Summary

**UnoCSS Responsive Checklist:**
- ✅ Mobile-first approach (base styles, then breakpoints)
- ✅ Use project breakpoints: `xs:`, `sm:`, `md:`, `lg:`, `xl:`
- ✅ Use `max-*:` or `<*:` for max-width queries
- ✅ Use variant groups `sm:(...)` for cleaner code
- ✅ Use `:uno:` prefix consistently
- ✅ Test at all breakpoint boundaries
- ✅ Progressive enhancement (mobile → desktop)
- ✅ 2-3 breakpoints per element maximum

**See Also:**
- `areas/common/uno.config.js` - UnoCSS configuration
- `packages/ui/stories/base/breakpoint.story.md` - Breakpoint definitions
- `areas/common/app/components/page/` - Page layout components
- `src/static/site/sass/_media-queries.scss` - Legacy breakpoint definitions
