- Start Date: 2020-03-31
- Implementation PR: (leave this empty)
- Store Discussion Issue: (leave this empty)

# Summary

This RFC proposes a solution to use the values defined in VTEX Tachyons in JavaScript-land.

# Basic example

```tsx
import {
  getBreakpoint,
  getBackgroundColor,
  getTextColor,
  getBorderColor,
  getOnColor,
  getSpacing,
  getBorderWidth,
} from 'vtex.styles'

// Breakpoint API
const minWidth = getBreakpoint('m') // 40

// Colors API
const bgBaseColor = getBackgroundColor('base')
const cMuted1Color = getTextColor('muted-1')
const bPrimaryActiveColor = getBorderColor('action-primary', 'active')
const onDangerHoverColor = getOnColor('danger', 'hover')

// Spacing API
const space = getSpacing(2) // 0.25

// Border Width API
const border = getBorderWidth(1) // 0.125
```

# Motivation

Apps like [vtex.device-detector](https://github.com/vtex-apps/device-detector) uses some [hard-coded values](https://github.com/vtex-apps/device-detector/blob/86076b0886bac7ddeada03625d2035407b19834f/react/useDevice.tsx#L19-L23) for breakpoints. With this new API those values would be the ones in effect in the store.

# Detailed design

The `styles-graphql` would expose a new API that will respond a JSON with 4 properties: `breakpoints`, `spacing`, `borderWidths` and `semanticColors`. Example:

```json
{
  "breakpoints": {
    "s": 20,
    "ns": 40,
    "m": 40,
    "l": 64,
    "xl": 80,
  },
  "spacing": [0.125, 0.25, 0.5, 0.75, 1, 1.5, 2, 3, 4, 8, 16],
  "borderWidths": [0, 0.125, 0.25, 0.5, 1, 2],
  "semanticColors": {
    "background": {
      "base": "#ffffff",
      "base--inverted": "#3f3f40",
      "action-primary": "#134cd8",
      "action-secondary": "#eef3f7",
      "emphasis": "#f71963",
      "disabled": "#f2f4f5",
      "success": "#8bc34a",
      "success--faded": "#eafce3",
      "danger": "#ff4c4c",
      "danger--faded": "#ffe6e6",
      "warning": "#ffb100",
      "warning--faded": "#fff6e0",
      "muted-1": "#727273",
      "muted-2": "#979899",
      "muted-3": "#cacbcc",
      "muted-4": "#e3e4e6",
      "muted-5": "#f2f4f5"
    },
    "hover-background": {
      "action-primary": "#0c389f",
      "action-secondary": "#dbe9fd",
      "emphasis": "#dd1659",
      "success": "#8bc34a",
      "success--faded": "#eafce3",
      "danger": "#e13232",
      "danger--faded": "#ffe6e6",
      "warning": "#ffb100",
      "warning--faded": "#fff6e0",
      "muted-1": "#727273",
      "muted-2": "#979899",
      "muted-3": "#cacbcc",
      "muted-4": "#e3e4e6",
      "muted-5": "#f2f4f5"
    },
    "active-background": {
      "action-primary": "#0c389f",
      "action-secondary": "#dbe9fd",
      "emphasis": "#dd1659",
      "success": "#8bc34a",
      "success--faded": "#eafce3",
      "danger": "#ff4c4c",
      "danger--faded": "#ffe6e6",
      "warning": "#ffb100",
      "warning--faded": "#fff6e0",
      "muted-1": "#727273",
      "muted-2": "#979899",
      "muted-3": "#cacbcc",
      "muted-4": "#e3e4e6",
      "muted-5": "#f2f4f5"
    },
    "text": {
      "action-primary": "#134cd8",
      "action-secondary": "#eef3f7",
      "link": "#134cd8",
      "emphasis": "#f71963",
      "disabled": "#979899",
      "success": "#79B03A",
      "success--faded": "#eafce3",
      "danger": "#ff4c4c",
      "danger--faded": "#ffe6e6",
      "warning": "#E19D00",
      "warning--faded": "#fff6e0",
      "muted-1": "#727273",
      "muted-2": "#979899",
      "muted-3": "#cacbcc",
      "muted-4": "#e3e4e6",
      "muted-5": "#f2f4f5"
    },
    "visited-text": {
      "link": "#0c389f",
    },
    "hover-text": {
      "action-primary": "#0c389f",
      "action-secondary": "#dbe9fd",
      "link": "#0c389f",
      "emphasis": "#dd1659",
      "success": "#79B03A",
      "success--faded": "#eafce3",
      "danger": "#e13232",
      "danger--faded": "#ffe6e6",
      "warning": "#E19D00",
      "warning--faded": "#fff6e0"
    },
    "active-text": {
      "link": "#0c389f",
      "emphasis": "#dd1659",
      "success": "#79B03A",
      "success--faded": "#eafce3",
      "danger": "#ff4c4c",
      "danger--faded": "#ffe6e6",
      "warning": "#E19D00",
      "warning--faded": "#fff6e0"
    },
    "border": {
      "action-primary": "#134cd8",
      "action-secondary": "#eef3f7",
      "emphasis": "#f71963",
      "disabled": "#e3e4e6",
      "success": "#79B03A",
      "success--faded": "#eafce3",
      "danger": "#ff4c4c",
      "danger--faded": "#ffe6e6",
      "warning": "#E19D00",
      "warning--faded": "#fff6e0",
      "muted-1": "#727273",
      "muted-2": "#979899",
      "muted-3": "#cacbcc",
      "muted-4": "#e3e4e6",
      "muted-5": "#f2f4f5"
    },
    "hover-border": {
      "action-primary": "#0c389f",
      "action-secondary": "#dbe9fd",
      "emphasis": "#dd1659",
      "success": "#79B03A",
      "success--faded": "#eafce3",
      "danger": "#e13232",
      "danger--faded": "#ffe6e6",
      "warning": "#E19D00",
      "warning--faded": "#fff6e0",
      "muted-1": "#727273",
      "muted-2": "#979899",
      "muted-3": "#cacbcc",
      "muted-4": "#e3e4e6",
      "muted-5": "#f2f4f5"
    },
    "active-border": {
      "action-primary": "#0c389f",
      "action-secondary": "#dbe9fd",
      "emphasis": "#dd1659",
      "success": "#79B03A",
      "success--faded": "#eafce3",
      "danger": "#ff4c4c",
      "danger--faded": "#ffe6e6",
      "warning": "#E19D00",
      "warning--faded": "#fff6e0",
      "muted-1": "#727273",
      "muted-2": "#979899",
      "muted-3": "#cacbcc",
      "muted-4": "#e3e4e6",
      "muted-5": "#f2f4f5"
    },
    "on": {
      "base": "#3f3f40",
      "base--inverted": "#ffffff",
      "action-primary": "#ffffff",
      "action-secondary": "#134cd8",
      "emphasis": "#ffffff",
      "disabled": "#979899",
      "success": "#ffffff",
      "success--faded": "#3f3f40",
      "danger": "#ffffff",
      "danger--faded": "#3f3f40",
      "warning": "#ffffff",
      "warning--faded": "#1a1a1a",
      "muted-1": "#ffffff",
      "muted-2": "#ffffff",
      "muted-3": "#3f3f40",
      "muted-4": "#3f3f40",
      "muted-5": "#3f3f40"
    },
    "hover-on": {
      "action-primary": "#ffffff",
      "action-secondary": "#134cd8",
      "emphasis": "#ffffff",
      "success": "#ffffff",
      "success--faded": "#3f3f40",
      "danger": "#ffffff",
      "danger--faded": "#3f3f40",
      "warning": "#ffffff",
      "warning--faded": "#1a1a1a"
    },
    "active-on": {
      "action-primary": "#ffffff",
      "action-secondary": "#134cd8",
      "emphasis": "#ffffff",
      "success": "#ffffff",
      "success--faded": "#3f3f40",
      "danger": "#ffffff",
      "danger--faded": "#3f3f40",
      "warning": "#ffffff",
      "warning--faded": "#1a1a1a"
    }
  }
}
```

Render-server would use this API to add this JSON to the HTML like so:

```js
window.__STYLES__ = {{ JSON above }}
```

A new app called `vtex.styles` would read the variable `window.__STYLES__` and expose the values through a JavaScript API.

## API

### Breakpoint

Returns a numeric value of min-width of the media-query in `em` unit.

```ts
type Breakpoint = 's' | 'ns' | 'm' | 'l' | 'xl'
getBreakpoint(breakpoint: Breakpoint): number
```

Example:

```ts
const minWidth = getBreakpoint('m') // 40
```

### Colors

Returns the value of the color as defined in the JSON (HEX or RGBA).

```ts
type Color = string

type BackgroundModifier = 'hover' | 'active'
getBackgroundColor(token: BackgroundToken, modifier: BackgroundModifier): Color

type TextModifier = 'hover' | 'active' | 'visited'
getTextColor(token: TextToken, modifier: TextModifier): Color

type BorderModifier = 'hover' | 'active'
getBorderColor(token: BorderToken, modifier: BorderModifier): Color

type OnModifier = 'hover' | 'active'
getOnColor(token: OnToken, modifier: OnModifier): Color
```

Example:

```ts
const color = getOnColor('base') // '#3f3f40'
```

### Spacing

Get the spacing value as a numeric value in 'rem' unit.

```ts
getSpacing(index: number): number
```

Example:

```ts
const spacing = getSpacing(2) // 0.25
```

### Border Width

Get the border width value as a numeric value in 'rem' unit.

```ts
getBorderWidth(index: number): number
```

Example:

```ts
const border = getBorderWidth(1) // 0.125
```

## Usage

In the vtex.device-detector example the code using this new API would be:

```ts
const mBreakpoint = getBreakpoint('m') // 40
const lBreakpoint = getBreakpoint('l') // 64
const isScreenMedium = useMedia({ minWidth: `${mBreakpoint}rem` })
const isScreenLarge = useMedia({ minWidth: `${lBreakpoint + 0.1}rem` })
```

# Drawbacks

Adding `window.__STYLES__` would add around 632 bytes to the payload of the page.

# Alternatives

Compiling Tachyons keeping CSS variables and getting the value of those tokens through:

```js
getComputedStyle(document.documentElement)
  .getPropertyValue('--c-on-base');
```

This approach would not work in SSR since it depends on `document`.

---

Maybe we can mix and match both solutions:
- At SSR: use `global.__STYLES__`
- At client: use `getPropertyValue` and not add `window.__STYLES__` to the page

# Adoption strategy

That's a new feature, no need to rush adopting it.

# How we teach this

We need to document how to better use the Tachyons config.

# Unresolved questions

n/a