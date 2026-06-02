# Full SCSS → UnoCSS Mapping Reference

Lookup table for every SCSS variable in `client/assets/styles/_variables.scss` (and the most common helpers in `_sizes.scss`, `_global.scss`) mapped to its closest UnoCSS utility under the `@paradoxai/ui` preset.

When the legacy hex code matches a design-token value (within ~1 hex unit, or after rounding), use the token utility. When it doesn't match anything in `@paradoxai/design-tokens` (e.g. `$color-electric-violet: #7549fa`), fall back to an arbitrary utility (`text-[#7549fa]`). Don't pick a vaguely-similar token — visual mismatches are worse than honest arbitraries.

**Variable not in this file?** If an SCSS variable has no row below, do not guess a mapping. Resolve its value from source, apply the best token or arbitrary fallback, and add `//TODO` on the line immediately before the migrated utility in the Vue file (see [SKILL.md — Unmapped SCSS variables](../SKILL.md#unmapped-scss-variables--add-todo)). After review, add the variable to this table and remove the `//TODO`.

## Table of contents

- [1. Brand / semantic colors](#1-brand--semantic-colors)
- [2. Text colors](#2-text-colors)
- [3. Border colors](#3-border-colors)
- [4. Background colors](#4-background-colors)
- [5. One-off named colors](#5-one-off-named-colors-no-token-match)
- [6. Border radius](#6-border-radius)
- [7. Font sizes](#7-font-sizes)
- [8. Font weights](#8-font-weights)
- [9. Line heights](#9-line-heights)
- [10. Spacing scale](#10-spacing-scale)
- [11. Breakpoints](#11-breakpoints)
- [12. Box shadow](#12-box-shadow)
- [13. Layout constants](#13-layout-constants)
- [14. Durations](#14-durations)
- [15. Common SCSS mixins](#15-common-scss-mixins--functions)

---

## 1. Brand / semantic colors

| SCSS variable | Hex | Utility (text/bg/border) | Notes |
|---|---|---|---|
| `$color-primary` | `var(--color-primary, #25c9d0)` | `text-primary-500` | Themable runtime var; token matches default |
| `$color-success` | `var(--color-success, #39d279)` | `text-success-500` | |
| `$color-warning` | `var(--color-warning, #f5a623)` | `text-warning-500` (closest) or `text-[#f5a623]` | Token `warning-500` is `#f9bc4f` — use arbitrary if exact match required |
| `$color-danger` | `var(--color-danger, #e52d2d)` | `text-danger-500` | |
| `$color-less-danger` | `var(--color-less-danger, #da5656)` | `text-[#da5656]` | No token match |
| `$color-danger-dark` | `#bf1818` | `text-danger-700` | |
| `$color-danger-medium-dark` | `#b91e1e` | `text-[#b91e1e]` | Between 700 and 800 |
| `$color-bubble` | `var(--color-bubble, #45d0d6)` | `text-[var(--color-bubble,#45d0d6)]` | Themable; preserve var |
| `$color-warning-dark` | `#c5964d` | `text-[#c5964d]` | |
| `$color-link` | `var(--color-link, #0bb4ba)` | `text-primary-700` | `#0bb4ba` matches `primary-700` |
| `$color-link-hover` | `var(--color-link-hover, #2a80b0)` | `text-[#2a80b0]` | |

## 2. Text colors

| SCSS variable | Hex | Utility | Notes |
|---|---|---|---|
| `$color-text` | `#444` | `text-grey-950` | |
| `$color-text-primary` | `var(--color-text, #555)` | `text-grey-900` | |
| `$color-text-secondary` | `var(--color-text-secondary, #a2a2a2)` | `text-grey-500` | `#a9a9a9` is `grey-500`; `#a2a2a2` is close enough |
| `$color-text-lighter` | `var(--color-text-lighter, #a9a9a9)` | `text-grey-500` | Exact match |
| `$color-text-extra-light` | `#dadce0` | `text-grey-400` | |
| `$color-text-placeholder` | `#a9a9a9` | `text-grey-500` | Same as `$color-text-lighter` |
| `$color-text-danger` | `#e52d2d` | `text-danger-500` | |
| `$color-text-transparent` | `#adadad` | `text-[#adadad]` | |
| `$color-text-disable` | `#c8c8c8` | `text-[#c8c8c8]` | |
| `$color-text-error` | `#a94442` | `text-[#a94442]` | |
| `$color-text-light-gray` | `#bbb` | `text-[#bbb]` | |

## 3. Border colors

| SCSS variable | Hex | Utility |
|---|---|---|
| `$border-color-danger` | `#fdc8c8` | `border-[#fdc8c8]` |
| `$border-color-base` | `#cbced3` | `border-grey-400` (close: `#dadce0`) or `border-[#cbced3]` |
| `$border-color-medium` | `#dadce0` | `border-grey-400` |
| `$border-color-light` | `#ededed` | `border-grey-300` |
| `$border-color-lighter` | `#ebeef5` | `border-[#ebeef5]` |
| `$border-color-extra-light` | `#f2f6fc` | `border-[#f2f6fc]` |
| `$border-color-extra-danger` | `#e52d2d` | `border-danger-500` |
| `$border-button-base` | `rgba(0,0,0,0.15)` | `border-[rgba(0,0,0,0.15)]` |
| `$border-color-box` | `#dadce0` | `border-grey-400` |
| `$border-color-bubble` | `#bdf4f2` | `border-[#bdf4f2]` |
| `$border-color-light-gray` | `#e4e4e4` | `border-[#e4e4e4]` |
| `$border-color-light-blue-gray` | `#d2d7df` | `border-[#d2d7df]` |
| `$border-color-silver` | `#e1e1e1` | `border-[#e1e1e1]` |
| `$border-color-caution` | `#f9bc4f` | `border-warning-500` |
| `$border-color-gray` | `#8991a5` | `border-[#8991a5]` |
| `$bolder-color-blue` | `#2765cf` | `border-[#2765cf]` |
| `$border-color-editor` | `#ccc` | `border-[#ccc]` |
| `$border-color-warning` | `#fbd288` | `border-warning-400` |
| `$border-color-mint-green` | `#cef3dd` | `border-[#cef3dd]` |
| `$color-border-line` | `#e0e0e0` | `border-[#e0e0e0]` |

## 4. Background colors

| SCSS variable | Hex | Utility |
|---|---|---|
| `$background-color-danger` | `#ffe3e2` | `bg-[#ffe3e2]` |
| `$background-color-base` | `#f7f7f7` | `bg-grey-100` (`#f8f8f8` ≈) |
| `$background-color-white` | `#fff` | `bg-white` |
| `$background-hover` | `#f8f8f8` | `bg-grey-100` |
| `$background-color-bubble` | `#f7ffff` | `bg-primary-50` |
| `$background-color-disable` | `#bde6e8` | `bg-primary-400` |
| `$background-color-hover` | `#e4e4e4` | `bg-[#e4e4e4]` |
| `$background-color-light` | `#ededed` | `bg-grey-300` |
| `$background-dropdown` | `#fcfcfc` | `bg-grey-50` |
| `$background-light-gray` | `#dfdfdf` | `bg-[#dfdfdf]` |
| `$background-gray` | `#aaa` | `bg-[#aaa]` |
| `$background-danger-exit` | `#e52d2d` | `bg-danger-500` |
| `$background-danger-light` | `#fdeded` | `bg-danger-100` |
| `$background-hover-exit` | `#bf1818` | `bg-danger-700` |
| `$background-dragging` | `rgba(255,255,255,0.8)` | `bg-white/80` |
| `$background-olivia-blue-300` | `#ccf4f3` | `bg-primary-300` |
| `$background-color-tranparent` | `rgba(0,0,0,0)` | `bg-transparent` |
| `$background-color-caution` | `#fef6e7` | `bg-warning-100` |
| `$background-color-light-black` | `rgba(0,0,0,0.5)` | `bg-black/50` |
| `$background-color-light-red` | `rgba(232,75,75,0.05)` | `bg-[rgba(232,75,75,0.05)]` |

Generic gray scale (semantic-token-aligned):

| SCSS variable | Hex | Utility |
|---|---|---|
| `$color-gray` | `#dfe2e6` | `bg-[#dfe2e6]` |
| `$color-pale-gray` | `#e3e3e3` | `bg-[#e3e3e3]` |
| `$color-light-gray` | `#fafafa` | `bg-[#fafafa]` (no exact token; `grey-100` is `#f8f8f8`) |
| `$color-dark-gray` | `#a2a2a2` | `text-grey-500` |
| `$color-darker-gray` | `var(--color-darker-gray, #999)` | `text-[#999]` |
| `$color-lighter-gray` | `#a9a9a9` | `text-grey-500` |
| `$color-state-grey` | `#757575` | `text-grey-800` |
| `$color-disco-grey` | `#dde1e8` | `bg-[#dde1e8]` |
| `$color-platinum` | `#e5e5e5` | `bg-[#e5e5e5]` |
| `$color-gainsboro` | `#e2e2e2` | `bg-[#e2e2e2]` |
| `$color-light-gainsboro` | `#d8d8d8` | `bg-[#d8d8d8]` |
| `$color-white-smoke` | `#f3f3f3` | `bg-[#f3f3f3]` |
| `$color-anti-flash-white` | `#f1f1f1` | `bg-[#f1f1f1]` |
| `$color-alto` | `#dbdbdb` | `bg-[#dbdbdb]` |
| `$skeleton-color` | `#f2f2f2` | `bg-grey-200` |
| `$skeleton-to-color` | `#e6e6e6` | `bg-[#e6e6e6]` |
| `$icon-color` | `#979797` | `text-[#979797]` |
| `$icon-color-disable` | `#d1d1d1` | `text-[#d1d1d1]` |

## 5. One-off named colors (no token match)

These are decorative palette colors. They have no design-token analogue — always use arbitrary utilities.

| SCSS variable | Hex | Utility |
|---|---|---|
| `$color-black-darker` | `#1c2330` | `text-[#1c2330]` |
| `$color-black-lighter` | `#222` | `text-[#222]` |
| `$color-black-extra-lighter` | `#272d39` | `text-[#272d39]` |
| `$color-dark` | `#233d4d` | `text-[#233d4d]` (matches `$color-blue-dianne`, `navyBlue-500`) |
| `$color-blue-dianne` | `#233d4d` | `text-navyBlue-500` |
| `$color-william` | `#395e66` | `text-secondary-500` |
| `$color-electric-violet` | `#7549fa` | `text-[#7549fa]` |
| `$color-medium-purple` | `#7756da` | `text-[#7756da]` |
| `$color-japonica` | `var(--color-japonica, #dd7373)` | `text-cardinalRed-500` |
| `$color-stella-pink` | `#e69d99` | `text-[#e69d99]` |
| `$color-matisse` | `#2360a7` | `text-[#2360a7]` |
| `$color-cupid` | `var(--color-cupid, #fac4c4)` | `text-danger-400` |
| `$color-medium-pink` | `#c961aa` | `text-pricklyPearMagenta-500` |
| `$color-medium-orange` | `#ff9871` | `text-[#ff9871]` |
| `$color-notify-warning` | `#f9bc4f` | `text-warning-500` |
| `$color-bismark` | `#427887` | `text-[#427887]` |
| `$color-begonia` | `#fe6d73` | `text-desertRed-500` |
| `$color-shamrock` | `#3bceac` | `text-cactusGreen-500` |
| `$color-can-can` | `#d596c2` | `text-[#d596c2]` |
| `$color-atomic-tangerine` | `#ff9b71` | `text-duneOrange-500` |
| `$color-dull-lavender` | `#ad8ce2` | `text-purple-500` |
| `$color-casablanca` | `#f9bc4f` | `text-warning-500` |
| `$color-bondi-blue` | `#00aab5` | `text-[#00aab5]` |
| `$color-ziggurat` | `#b2dadc` | `text-[#b2dadc]` |
| `$color-cerulean` | `#0bb4ba` | `text-primary-700` |
| `$color-picton-blue` | `#37a9e9` | `text-skyBlue-500` |
| `$color-mint-tulip` | `#ccf4f3` | `text-primary-300` |
| `$color-white-lilac` | `#efe8f9` | `text-purple-200` |
| `$color-sea-blue` | `#126892` | `text-skyBlue-700` |
| `$color-hawkes-blue` | `#d7eefb` | `text-skyBlue-200` |
| `$color-colombia-blue` | `#c6e2f1` | `text-[#c6e2f1]` |
| `$color-yellow-dark` | `#e08f00` | `text-yellow-700` |
| `$color-royal-blue` | `#3c5b9a` | `text-[#3c5b9a]` |
| `$color-desert-red-light` | `#ffe2e3` | `text-desertRed-200` |
| `$color-dark-red` | `rgba(232,75,75,0.9)` | `text-[rgba(232,75,75,0.9)]` |
| `$color-soft-azure-haze` | `#c8e1f7` | `text-[#c8e1f7]` |
| `$color-icy-azure` | `#e3f2fb` | `text-[#e3f2fb]` |
| `$color-midnight-blue` | `#1d559d` | `text-[#1d559d]` |
| `$color-steel-blue` | `#337ab7` | `text-[#337ab7]` |
| `$color-shadow` | `#0000002e` | `text-[#0000002e]` |
| `$color-light-red` | `#d06161` | `text-[#d06161]` |
| `$color-light-green` | `#64d17e` | `text-[#64d17e]` |
| `$color-gold` | `#ffbc0d` | `text-[#ffbc0d]` |
| `$color-tooltip-shadow` | `#0003` | `text-[#0003]` |
| `$color-emerald-green` | `#27aa5d` | `text-success-700` |
| `$color-light-teal-green` | `#e9faf0` | `text-success-100` |
| `$color-vanilla-cream` | `#fdebca` | `text-[#fdebca]` |
| `$color-light-pink` | `#f7c0c0` | `text-[#f7c0c0]` |
| `$color-mint-green` | `#c4f1d7` | `text-[#c4f1d7]` |
| `$color-salmon-pink` | `#f56c6c` | `text-[#f56c6c]` |

## 6. Border radius

| SCSS variable | Value | Utility |
|---|---|---|
| `$border-radius-base` | `2px` | `rounded-sm` (`0.125rem` = `2px`) |
| `$border-radius-medium` | `4px` | `rounded` (`0.25rem` = `4px`) |
| `$border-radius-large` | `8px` | `rounded-lg` (`0.5rem` = `8px`) |
| `$border-radius-larger` | `10px` | `rounded-[10px]` |
| `$border-radius-extra-large` | `12px` | `rounded-xl` (`0.75rem` = `12px`) |
| `$rounded-full` | `9999px` | `rounded-full` |

Token name: `borderRadius` from `uno.mjs` (`base, sm, md, lg, xl, full, xxl`).

## 7. Font sizes

| SCSS variable | Value | Utility (with line-height pair) |
|---|---|---|
| `$font-size-extra-large` | `20px` | `text-xl` (`1.25rem`) |
| `$font-size-large` | `18px` | `text-lg` (`1.125rem`) |
| `$font-size-medium` | `16px` | `text-md` (`1rem`) |
| `$font-size-base` | `14px` | `text-base` (`0.875rem`) |
| `$font-size-small` | `12px` | `text-sm` (`0.75rem`) |
| `$font-size-extra-small` | `10px` | `text-xs` (`0.625rem`) |
| `$font-size-h1` | `~36px` | `text-h1` |
| `$font-size-h2` | `~30px` | `text-h2` |
| `$font-size-h3` | `~24px` | `text-h3` |
| `$font-size-h4` | `~18px` | `text-h4` |
| `$font-size-h5` | `14px` | `text-h5` |
| `$font-size-h6` | `~12px` | `text-h6` |

## 8. Font weights

| SCSS variable | Value | Utility |
|---|---|---|
| `$font-weight-base` | `400` | `font-base` (alias for normal) |
| `$font-weight-primary` | `600` | `font-primary` (alias for semibold) |
| `$font-weight-semi-bold` | `700` | `font-bold` |
| `$font-weight-bold` | `bold` | `font-bold` |
| `$font-weight-bolder` | `bolder` | `font-bolder` (or `font-[bolder]`) |

## 9. Line heights

| SCSS variable | Value | Utility |
|---|---|---|
| `$line-height-extra-small` | `1.4` | `leading-[1.4]` |
| `$line-height-small` | `1.4167` | `leading-sm` (`1.0625rem`) — note: token uses rem, not unitless |
| `$line-height-base` | `1.4286` | `leading-base` (`1.25rem`) |
| `$line-height-medium` | `1.375` | `leading-md` (`1.375rem`) |
| `$headings-line-height` | `1.1` | `leading-[1.1]` |
| `$line-height-computed` | `~20px` | `leading-base` |

## 10. Spacing scale

> **Foundation rule:** `1rem = 16px`. UnoCSS unit `1` = `0.25rem` = `4px`.
> Therefore: **px ÷ 4 = UnoCSS unit** (and **rem × 4 = UnoCSS unit**).
> All sizing utilities (`p-*`, `m-*`, `w-*`, `h-*`, `gap-*`, `top-*`, etc.) follow this scale.

| SCSS spacing token | Value | Utility |
|---|---|---|
| `spacing.xs` | `0.125rem` (`2px`) | `p-0.5` / `m-0.5` |
| `spacing.sm` | `0.25rem` (`4px`) | `p-1` / `m-1` |
| `spacing.base` | `0.5rem` (`8px`) | `p-2` / `m-2` |
| `spacing.md` | `1rem` (`16px`) | `p-4` / `m-4` |
| `spacing.lg` | `1.5rem` (`24px`) | `p-6` / `m-6` |
| `spacing.xl` | `2rem` (`32px`) | `p-8` / `m-8` |
| `spacing.2xl` | `2.5rem` (`40px`) | `p-10` / `m-10` |
| `spacing.3xl` | `3rem` (`48px`) | `p-12` / `m-12` |
| `spacing.4xl` | `3.5rem` | `p-14` |
| `spacing.5xl` | `4rem` | `p-16` |
| `spacing.6xl` | `4.5rem` | `p-18` |
| `spacing.7xl` | `5rem` | `p-20` |
| `spacing.8xl` | `5.5rem` | `p-22` |
| `spacing.9xl` | `8rem` | `p-32` |

For pixel values in legacy SCSS, divide by 4 to get the UnoCSS unit. Examples:

| SCSS | Utility |
|---|---|
| `padding: 4px` | `p-1` |
| `padding: 8px` | `p-2` |
| `padding: 12px` | `p-3` |
| `padding: 16px` | `p-4` |
| `padding: 24px` | `p-6` |
| `padding: 32px` | `p-8` |
| `gap: 14px` | `gap-3.5` |
| `margin-top: 30px` | `mt-[30px]` (off-grid → arbitrary) |

## 11. Breakpoints

| SCSS breakpoint | Width | UnoCSS variant prefix |
|---|---|---|
| `xxs` | `375px` | not in tokens — write `[@media(min-width:375px)]:` if needed |
| `xs` | `576px` | `xs:` |
| `sm` | `768px` | `sm:` |
| `md` | `992px` | `md:` |
| `lg` | `1200px` | `lg:` |
| `xl` | `1440px` | `xl:` |

For "less than X" queries, use the `lt-` prefix (`lt-md:` = max-width 991.99px). For mobile-first device checks the codebase prefers reading `$device?.isMobile` from `useNuxtApp()` and switching classes via `:class` ternary — see `form-exp-review-section.vue`.

## 12. Box shadow

Tokens (`boxShadow`) come from `uno.mjs`:

| Token | Value | Utility |
|---|---|---|
| `lighter` | `0 1px 3px 0 rgba(0,0,0,0.10)` | `shadow-lighter` |
| `light` | `0 2px 10px 2px rgba(0,0,0,0.10)` | `shadow-light` |
| `base` | `0 2px 8px 0 rgba(0,0,0,0.20)` | `shadow-base` (or `shadow`) |
| `md12` | `0 6px 12px 0 rgba(0,0,0,0.18)` | `shadow-md12` |
| `md16` | `0 4px 16px 0 rgba(0,0,0,0.28)` | `shadow-md16` |
| `dark` | `0 3px 9px 0 rgba(0,0,0,0.50)` | `shadow-dark` |

Legacy `$shadow-base` (`rgba(99,99,99,0.2) 0 2px 8px 0`) → closest token is `shadow-base` (uses `rgba(0,0,0,0.2)`). Use `shadow-[0_2px_8px_0_rgba(99,99,99,0.2)]` if exact gray-tint is required.

## 13. Layout constants

| SCSS variable | Value | Utility |
|---|---|---|
| `$header-height` | `64px` | `h-16` or `h-[var(--header-height,4rem)]` |
| `$toolbar-height` | `48px` | `h-12` |
| `$mobile-toolbar-height` | `64px` | `h-16` |
| `$container-width` | `1120px` | `max-w-[1120px]` |
| `$width-drawer-large` | `864px` | `w-[864px]` |
| `$zindex-header` | `1030` | `z-[1030]` or `z-[calc(var(--z-index-header))]` |
| `$zindex-menu` | `1029` | `z-[1029]` |

Note: `$header-height` is exposed as the CSS variable `--header-height` in `_global.scss`. Pages and components consume it as `var(--header-height,4rem)`. Keep this pattern when migrating — don't bake `64px` into utilities; the variable lets the layout shrink on mobile.

## 14. Durations

| SCSS variable | Value | Utility |
|---|---|---|
| `$duration-faster` | `100ms` | `duration-100` |
| `$duration-fast` | `200ms` | `duration-200` |
| `$duration-base` | `300ms` | `duration-300` |
| `$duration-slow` | `500ms` | `duration-500` |
| `$duration-slower` | `700ms` | `duration-700` |

## 15. Common SCSS mixins → UnoCSS shortcuts

These mixins live in `client/assets/styles/base/_helpers.scss` and similar files. Translate them to the repo's preset shortcuts in `packages/ui/src/preset/shortcuts/common.ts` and `typography.ts`:

| SCSS mixin / pattern | UnoCSS shortcut |
|---|---|
| `@include flexBox()` | `flex` |
| `@include flexBox($align: center)` | `fi` |
| `@include flexBox($justify: center)` | `fc` |
| `@include flexBox($align: center, $justify: center)` | `fcc` (or `f-c`) |
| `@include flexBox($justify: space-between, $align: center)` | `fbc` |
| `@include flexBox($justify: space-around, $align: center)` | `fac` |
| `@include flexBox($direction: column)` | `f-col` |
| `@include flexBox($direction: column, $col-gap: 8px)` | `f-col gap-x-2` |
| `@include flexBox($direction: column, $row-gap: 16px)` | `f-col gap-y-4` |
| `@include size(60%, 700px)` (width, height) | `w-[60%] h-[700px]` |
| `@include size(100%)` | `size-full` |
| `position: absolute; inset: 0` | `pa inset-0` |
| `position: absolute; top: 50%; left: 50%; transform: translate(-50%, -50%)` | `p-c` |

Typography combos:

| SCSS pattern | UnoCSS shortcut |
|---|---|
| `font-size: $font-size-base; line-height: $line-height-base; font-weight: 400` | `typo-body` |
| `font-size: $font-size-base; line-height: $line-height-base; font-weight: 600` | `typo-subtitle` |
| `font-size: $font-size-small; line-height: $line-height-small; font-weight: 600; text-transform: uppercase` | `typo-subtitle-mini` |
| Button text (14px / 600) | `typo-button` |
| Button text small (12px / 600) | `typo-button-sm` |
| Tooltip (12px / 400) | `typo-tooltip` |

See `packages/ui/src/preset/shortcuts/typography.ts` for the full list.

---

## Quick decision: token vs arbitrary

When you're unsure whether to use a token or an arbitrary utility, apply this rule:

1. Is the SCSS value referenced by a *semantic* variable (`$color-primary`, `$color-danger`, `$font-size-base`)? → **Token**.
2. Is it a *named decorative* color (`$color-electric-violet`, `$color-japonica`)? → Check if hex matches a token within ~1 unit. If yes, token; if no, **arbitrary**.
3. Is it a raw inline hex (`#7549fa`) or one-off pixel (`30px`)? → **Arbitrary**.
4. Is it a CSS custom property (`var(--header-height)`)? → **Preserve the var**: `pt-[var(--header-height)]`.

When the user reviews your migration diff, arbitrary utilities are easy to upgrade to tokens later if they decide to add the missing token. Wrong-token migrations are silent visual regressions — much harder to catch.

5. Is the SCSS **variable name** missing from this file entirely? → Resolve value, apply fallback utility, and add `//TODO` before it in the migrated component.
