---
name: migrate-scss-to-unocss
description: Migrate Vue 2 single-file component scoped SCSS (the `<style lang="scss" scoped>` block) into Vue 3 / Nuxt 3 UnoCSS utility classes using the `@paradoxai/ui` preset and `@paradoxai/design-tokens`. Use when migrating a `client/` component to `areas/`, when the user asks to convert SCSS to UnoCSS, when porting `_variables.scss` colors to design tokens, when removing a `<style>` block in favor of inline `:class=":uno: ..."`, when the user mentions Nuxt 3 / UnoCSS / `@paradoxai/ui` / `@paradoxai/design-tokens`, or when the user wants to replace `$color-primary`, `$font-size-base`, `$border-radius-medium`, `$line-height-base`, `@include flexBox`, `@include size`, `:global(...)`, etc. with utility classes.
---

# Migrate legacy SCSS to UnoCSS (Nuxt 3)

You are converting a Vue 2 SFC's scoped SCSS block (or its sidecar `*.scss` file) into Vue 3 + Nuxt 3 UnoCSS utility classes that match the conventions used in `areas/*` and `packages/ui` of this repo.

The end state for a migrated component is:
- No `<style lang="scss" scoped>` block (or only a tiny one, with a justification comment)
- Styles expressed inline via `:class="[':uno: ...']"` or `class=":uno: ..."`
- Design-token utilities (e.g. `text-primary-500`, `bg-grey-100`, `text-base`, `rounded-lg`) preferred over arbitrary values
- Arbitrary values (`text-[#444]`, `w-[234px]`) only when the legacy color/size doesn't map cleanly to a token
- CSS custom properties (`var(--color-primary)`, `var(--header-height)`) preserved when the original uses them, since they're already runtime-themable

**You are not designing new styles.** You are translating existing pixels into utilities — match the legacy output as closely as possible.

## Workflow

Track progress with this checklist (copy into your reply once, then update):

```
- [ ] Step 1: Read the source SCSS (in-SFC or external)
- [ ] Step 2: Read the source template so class names line up with selectors
- [ ] Step 3: Build a per-selector mapping plan
- [ ] Step 4: Apply utilities to template; remove the <style> block
- [ ] Step 5: Self-check (no SCSS leftovers, tokens used, deep selectors handled)
```

### Step 1 — Read the source

Read the full SCSS *and* the full `<template>` together. You need both to know which selector applies to which element. If SCSS lives in a sidecar file (e.g. `offer-letter.scss`), search for the corresponding `.vue` that imports it.

### Step 2 — Map selectors to template nodes

For each SCSS rule, find the element it targets in the template (by class name, by `:global(...)`, by tag, by descendant chain). Note any rule that targets:
- A child component's internal slot via `:global(...)` / `:deep(...)` — this needs the `all-[selector]:(...)` UnoCSS form (see Edge Cases below).
- A pseudo-class like `&:hover`, `&:focus`, `&.disabled` — these become variant prefixes (`hover:`, `focus:`, `aria-disabled:`) on the same element.
- Media queries — these become breakpoint prefixes (`sm:`, `md:`, `lg:`).

If a selector has no match in the template, it's dead style — flag it for the user rather than silently dropping it.

### Step 3 — Plan token mappings

Using the table in the [Mapping Reference](#mapping-reference) section below, translate each SCSS declaration to its utility. Prefer in this order:

1. **Token utility** — e.g. `$color-primary` → `text-primary-500` / `bg-primary-500`
2. **CSS variable utility** — e.g. when source uses `var(--header-height)`, keep it: `pt-[var(--header-height)]`
3. **Arbitrary value** — e.g. `#fafafa` (no token match) → `bg-[#fafafa]`

For values that aren't on any token scale (random hex codes from `_variables.scss` like `$color-electric-violet: #7549fa`), arbitrary values are correct — don't invent a token name.

For deep file-by-file token lookups (the full SCSS-variable → utility table), see [references/token-mapping.md](references/token-mapping.md).

### Step 4 — Apply and clean

Apply the utilities to the template. Then **delete the `<style>` block entirely** (or its `.scss` sidecar import). If a small block must remain (rare — e.g. `@keyframes` not yet expressible as a UnoCSS animation), keep only that fragment with a one-line `// keep: <reason>` comment so the next reviewer understands why.

If the file uses CSS Modules (`.docReviewWrapper { ... }` referenced as `:class="$style.docReviewWrapper"`), the migrated version drops `$style` and the module file too.

### Step 5 — Self-check

Before reporting done, verify:
- [ ] No `lang="scss"` or `lang="sass"` blocks remain (unless explicitly justified)
- [ ] No `@import`, `@include`, or SCSS variable references (`$color-...`) anywhere in the file
- [ ] Every utility class string is prefixed with `:uno:` so the extractor picks it up (this repo's convention — see `packages/ui/src/preset/extractors`)
- [ ] No invented design-token names that don't exist in `@paradoxai/design-tokens` — when unsure, fall back to arbitrary values
- [ ] Hover/focus/active/disabled/responsive variants from `&:hover` etc. all preserved

## The `:uno:` prefix

Every class string in this repo's Vue 3 areas starts with `:uno:`. It is a marker for the custom extractor at `packages/ui/src/preset/extractors/` and tells UnoCSS to scan that string. Always include it; otherwise utilities won't be generated.

```
class=":uno: flex flex-col gap-2"
:class="[':uno: text-primary-500 hover:text-primary-700']"
:class="[isActive ? ':uno: bg-primary-100' : ':uno: bg-grey-100']"
```

## Mapping Reference

These are the most common conversions you'll need. The full table — every variable in `client/assets/styles/_variables.scss` mapped to its UnoCSS equivalent — is in [references/token-mapping.md](references/token-mapping.md). Read that file when you encounter a variable you don't recognize.

### Foundation: 1 rem = 16px

This is the single most important conversion to keep in mind — every UnoCSS spacing/sizing utility resolves through it.

| Pixels | rem | UnoCSS unit | Utility example |
|---|---|---|---|
| `4px` | `0.25rem` | `1` | `p-1`, `m-1`, `gap-1` |
| `8px` | `0.5rem` | `2` | `p-2`, `m-2` |
| `12px` | `0.75rem` | `3` | `p-3`, `m-3` |
| `16px` | `1rem` | `4` | `p-4`, `m-4`, `text-md` |
| `24px` | `1.5rem` | `6` | `p-6`, `m-6` |
| `32px` | `2rem` | `8` | `p-8`, `m-8` |
| `64px` | `4rem` | `16` | `p-16`, `h-16` (`$header-height`) |

Rules of thumb:
- **px → UnoCSS unit:** divide pixels by 4 (`12px / 4 = 3` → `p-3`).
- **rem → UnoCSS unit:** multiply rem by 4 (`1.5rem × 4 = 6` → `p-6`).
- **Off-grid pixel?** If `px / 4` doesn't yield a whole or `.5` value (e.g. `30px`, `13px`, `234px`), use an arbitrary utility: `mb-[30px]`, `w-[234px]`. Don't round — that's a silent visual change.
- **Font sizes use the same math:** `$font-size-base: 14px` → `0.875rem` → `text-base`. The token's *unit* is rem, but the *visual* result is still 14px because root font-size is 16px (set in `packages/ui/src/preset/theme/preflight.ts`).

When in doubt, do the conversion in your head before picking a utility. A migration that gets the rem math wrong is a migration that ships a visual regression.

### Colors (`_variables.scss` → token utility)

The token names live in `@paradoxai/design-tokens/uno` (re-exported by `node_modules/@paradoxai/design-tokens/dist/js/default/uno.mjs`). The two-tone naming pattern is **`<role>-<scale>`** where `scale ∈ {50,100,…,950}` and `role` is one of `primary`, `secondary`, `success`, `danger`, `warning`, `info`, `grey`, plus the marketing palettes (`desertRed`, `duneOrange`, `pricklyPearMagenta`, `cactusGreen`, `midnightTeal`, `navyBlue`, `skyBlue`, `purple`, `red`, `green`, `blue`, `yellow`, `cardinalRed`).

| SCSS variable | Closest token utility | Notes |
|---|---|---|
| `$color-primary` | `text-primary-500` / `bg-primary-500` | Originally `var(--color-primary, #25c9d0)` — `primary` token is the same hex |
| `$color-success` | `text-success-500` / `bg-success-500` | |
| `$color-danger` / `$color-text-danger` | `text-danger-500` / `bg-danger-500` | |
| `$color-warning` | `text-warning-500` / `bg-warning-500` | |
| `$color-text-primary` | `text-grey-900` | `#555` is closest to `grey-900` |
| `$color-text-secondary` / `$color-text-lighter` | `text-grey-500` | `#a2a2a2` / `#a9a9a9` ≈ `grey-500` |
| `$color-text` | `text-grey-950` | `#444` |
| `$border-color-base` | `border-grey-400` | `#cbced3` ≈ `grey-400` |
| `$border-color-medium` / `$color-text-extra-light` | `border-grey-400` | `#dadce0` |
| `$border-color-light` / `$background-color-light` | `border-grey-300` / `bg-grey-300` | `#ededed` |
| `$color-white` | `text-white` / `bg-white` | |
| `$color-black` | `text-black` / `bg-black` | |
| `$skeleton-color` / `$background-color-base` | `bg-grey-200` | `#f2f2f2` / `#f7f7f7` |

**Anything not in the table:** if it's a random one-off color (`$color-electric-violet: #7549fa`, `$color-japonica`, etc.), use an arbitrary utility — `text-[#7549fa]` — rather than picking a vaguely-similar token. Token mismatches are visually worse than honest arbitrary values.

### Sizing & spacing

| SCSS | UnoCSS | Notes |
|---|---|---|
| `padding: 12px` | `p-3` | 1 spacing unit = 0.25rem = 4px |
| `padding: 0 32px` | `px-8 py-0` or just `px-8` | |
| `margin-top: 12px` | `mt-3` | |
| `margin: 10px 0` | `my-2.5 mx-0` | |
| `width: 100%` | `w-full` | |
| `width: 234px` | `w-[234px]` | Pixel widths off-grid → arbitrary |
| `height: 56px` | `h-14` | 14 × 4 = 56 |
| `height: 60px` | `h-15` | UnoCSS allows `h-15` even though Tailwind's default scale doesn't |
| `min-height: 72px` | `min-h-18` | |
| `gap: 0.5rem` / `column-gap: 8px` | `gap-2` | |

For spacing tokens that *are* on the SCSS scale (`$spacing-md` = `1rem`), prefer the matching utility (`p-4`, `m-4`) rather than `p-[1rem]`.

### Typography

| SCSS | UnoCSS | Notes |
|---|---|---|
| `font-size: $font-size-base` (`14px`) | `text-base` | base = `0.875rem` |
| `font-size: $font-size-medium` (`16px`) | `text-md` | |
| `font-size: $font-size-large` (`18px`) | `text-lg` | |
| `font-size: $font-size-extra-large` (`20px`) | `text-xl` | |
| `font-size: $font-size-small` (`12px`) | `text-sm` | |
| `font-size: $font-size-extra-small` (`10px`) | `text-xs` | |
| `font-weight: $font-weight-base` (`400`) | `font-base` | |
| `font-weight: $font-weight-primary` (`600`) | `font-primary` | semibold |
| `font-weight: bold` / `$font-weight-bold` | `font-bold` | |
| `line-height: $line-height-base` | `leading-base` | |

Group typography is also available as shortcuts (`typo-body`, `typo-button`, `typo-subtitle`, …) — see `packages/ui/src/preset/shortcuts/typography.ts`. Prefer a shortcut when the SCSS is doing the same combo (size + weight + line-height + tracking).

### Borders & radius

| SCSS | UnoCSS |
|---|---|
| `border: 1px solid $border-color-base` | `border border-grey-400` |
| `border-radius: $border-radius-medium` (`4px`) | `rounded` |
| `border-radius: $border-radius-large` (`8px`) | `rounded-lg` |
| `border-radius: 50%` | `rounded-full` |
| `border-radius: 12px` | `rounded-xl` |

### Display & flex

The repo defines aliases in `packages/ui/src/preset/shortcuts/common.ts`. Prefer them when they fit:

| SCSS / `@include` | UnoCSS shortcut |
|---|---|
| `@include flexBox($align: center)` | `fi` (= `flex items-center`) |
| `@include flexBox($direction: column, $col-gap: 8px)` | `f-col gap-y-2` |
| `display: flex; justify-content: center; align-items: center` | `f-c` or `fcc` |
| `display: flex; justify-content: space-between; align-items: center` | `fbc` |
| `position: relative` | `pr` (yes, `pr` means `position: relative` here, not padding-right) |
| `position: absolute; top: 50%; left: 50%; transform: translate(-50%, -50%)` | `p-c` |

### Hover / focus / disabled / pseudo-classes

SCSS's `&:hover`, `&:focus`, `&.disabled` etc. become variant prefixes:

```scss
.btn {
  color: $color-primary;
  &:hover { color: $color-primary-700; }
  &.disabled { opacity: 0.6; cursor: not-allowed; }
}
```

→

```html
<button :class="[':uno: text-primary-500 hover:text-primary-700', { ':uno: opacity-60 cursor-not-allowed': isDisabled }]">
```

### Media queries

Use the breakpoint prefixes (`xs:`, `sm:`, `md:`, `lg:`, `xl:`) — these match `tokens.breakpoints`. For runtime device checks the codebase uses `$device?.isMobile` (see `form-exp-review-section.vue`), and that's fine to mix with utility classes.

### Deep / global selectors

Vue 2 scoped SCSS used `:deep(...)`, `::v-deep`, or `:global(...)` to reach into child components. UnoCSS expresses this as `all-[<selector>]:(...)`:

```scss
.wrapper :deep(.el-dialog__body) {
  flex: 1;
  padding: 0;
}
```

→

```html
<div class=":uno: all-[.el-dialog__body]:(flex-1 p-0)">
```

For element targeting via `data-testid` (common in `areas/*`):

```html
<div :class="[
  ':uno: !all-[[data-testid=\'section-content\']]:(gap-8 flex flex-col)',
]">
```

The leading `!` is `!important` — needed when overriding a child component's own utility.

### Element-Plus and other library overrides

When SCSS targets `.el-button`, `.el-input__inner`, etc., these are runtime DOM not under your component's scope. Use the `all-[.el-...]` pattern as in form-exp-review-section.vue:

```html
<div class=":uno: all-[.el-dialog__body]:(flex-1 p-0)">
```

If the override is sprawling (many child selectors, lots of pseudo-classes) it may be worth keeping a small `<style>` block specifically for it — note this exception in a comment.

## Examples

**Example 1 — simple block migration**

Before:
```vue
<template>
  <div class="docReviewWrapper">
    <span class="requireDot">*</span>
    <p class="displayBodyText">Hello</p>
  </div>
</template>

<style scoped lang="scss">
.docReviewWrapper {
  display: flex;
  flex-direction: column;
}
.displayBodyText {
  width: 100%;
  margin-top: 12px;
  margin-bottom: 30px;
  word-break: break-word;
}
.requireDot {
  color: $color-text-danger;
}
</style>
```

After:
```vue
<template>
  <div class=":uno: flex flex-col">
    <span class=":uno: text-danger-500">*</span>
    <p class=":uno: w-full mt-3 mb-[30px] break-words">Hello</p>
  </div>
</template>
```

Note `mb-[30px]` is arbitrary — `30px` ≠ any spacing token (`xl` is `2rem`/`32px`). Don't round to `mb-8`; that's a visual change.

**Example 2 — hover, conditional, deep**

Before:
```scss
.displayBody {
  background-color: $color-white;
  border: 1px solid $border-color-base;
  border-radius: 4px;
  color: $color-text-primary;
  cursor: pointer;
  &:hover { color: $color-text-primary; }
  &.disabled { opacity: 0.6; cursor: not-allowed; }
}
.assessmentIframeWrapper :global(.el-dialog__body) {
  flex: 1;
  padding: 0;
}
```

After (template):
```html
<div :class="[
  ':uno: bg-white border border-grey-400 rounded text-grey-900 cursor-pointer hover:text-grey-900',
  { ':uno: opacity-60 cursor-not-allowed': disabled },
]">
  ...
</div>

<div class=":uno: all-[.el-dialog__body]:(flex-1 p-0)">
  ...
</div>
```

**Example 3 — dynamic CSS variable kept as-is**

When the SCSS uses `var(--header-height)`, preserve it — these variables are themable at runtime and breaking them would change behaviour:

Before:
```scss
.body { padding-top: var(--header-height, 4rem); }
```

After:
```html
<div class=":uno: pt-[var(--header-height,4rem)]"></div>
```

This is the pattern used in `areas/employee-experience/app/pages/employee-experience/index.vue`.

## Edge cases & traps

**1. SCSS math (`floor`, `ceil`, `math.div`).** The legacy file uses `floor($font-size-base * 2.6)` for `$font-size-h1`. Don't try to recompute; the design tokens already have `font-size-h1: 2.25rem` (`text-h1`). Use the token, not the original computed pixels.

**2. `!important` in SCSS.** Translate to the `!` prefix: `color: red !important;` → `!text-red-500`. Use sparingly — required when overriding library defaults (Element-Plus) but a code smell otherwise.

**3. CSS Modules (`.module.scss`).** Templates reference these as `:class="$style.foo"`. After migration, drop both the import and the `$style` references — replace each with the inline utility string.

**4. Animations / `@keyframes`.** UnoCSS has `animate-...` shortcuts (see `packages/ui/src/preset/theme/animation`). For one-off keyframes, leave a tiny `<style>` block specifically for the `@keyframes` and use `animate-[name_duration]` to invoke it.

**5. `@include flexBox($direction: column, $col-gap: 8px)`.** This mixin sets `display: flex`, `flex-direction: column`, and `column-gap` — translate to `f-col gap-x-2`. Note `$col-gap` is `column-gap`, not just `gap`; check the sass mixin if unsure.

**6. The `--ol-` prefix.** The preset namespaces CSS variables as `--ol-...` (see `packages/ui/src/preset/theme/index.ts` → `variablePrefix: 'ol-'`). Don't write `var(--ol-...)` by hand — use the token utilities; UnoCSS resolves them.

**7. `breakpoints` are not generated as CSS variables.** They're consumed only by the variant system (`sm:`, `md:`, etc.). Never write `media (max-width: $breakpoints-md)` — use `lt-md:` (less-than-md) or `md:` directly.

**8. Vue 2 `<style scoped>` slot-scoped selectors.** `>>>`, `/deep/`, `::v-deep`, and `:deep()` all collapse into the `all-[selector]:(...)` form in this codebase. Pick one consistent representation per file.

## Reporting your work

After migration, summarize for the user:
- Files changed (template, removed `<style>`, removed `.scss`)
- Any selector that had no template match (you flagged it as dead style)
- Any value that fell back to an arbitrary utility because no token matched
- Any block deliberately *not* migrated (with reason — e.g. `@keyframes` left in place)

This makes the diff easy to review and surfaces anything that needs a designer's eye.

## Additional resources

- [references/token-mapping.md](references/token-mapping.md) — full SCSS-variable → UnoCSS-utility lookup, including every color, spacing, font, and breakpoint variable from `client/assets/styles/_variables.scss`. Read when you hit a variable not covered in the inline tables above.
