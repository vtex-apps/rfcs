- Start Date: 2020-03-31
- Implementation PR: (leave this empty)
- Store Discussion Issue: (leave this empty)

# Summary

Add support for `composes: ___ from styles` in CSS files.

# Basic example

```css
.element {
  border: 1px solid red;
  composes: heading-1 from styles;
}
```

# Motivation

Make it possible to compose CSS classes composing with Tachyons classes.

This will increase the usage of a well defined Design System through `styles/config/style.json`.

# Detailed design

This RFC proposes enabling the usage of the [`composes` syntax of CSS Modules](https://github.com/css-modules/css-modules#composition) in CSS files with some limitations:

1. It will **not be possible** to compose using another CSS Module file, example: `composes: className from "./style.css";`
2. It will **not be possible** to compose using `from global`, example: `composes: globalClassName from global;`
3. It will **not be possible** to compose local selectors, example: `composes: className`
4. Addition of a new API `from styles` to allow the usage of Tachyons classes, example: `composes: heading-1 from styles;`

## Why blocking composing with another file

This feature will make it harder for scenarios of themes.

Example of a theme that have the following folder structure inside the `styles` directory:

```
.
└── css
    ├── vtex.carousel.css
    ├── vtex.flex-layout.css
    └── vtex.store-header.css
```

If the user added to the `vtex.flex-layout.css`:

```css
.flexRowContent--menu-link {
  color: red;
  composes: containerImg from './vtex.carousel.css';
}
```

This would generate the following class in the HTML:

```html
<div className="vtex-flex-layout-0-x-flexRowContent--menu-link vtex-carousel-0-x-containerImg">
```

This would might lead to weird bugs, making it harder to debug.

## Why blocking composing from global

This feature will make it harder for scenarios of themes.

Example of a theme that have the following folder structure inside the `styles` directory:

```
.
└── css
    ├── vtex.carousel.css
    ├── vtex.flex-layout.css
    └── vtex.store-header.css
```

If the user added to the `vtex.flex-layout.css`:

```css
.flexRowContent--menu-link {
  color: red;
  composes: vtex-carousel-0-x-containerImg from global;
}
```

This would generate the following class in the HTML:

```html
<div className="vtex-flex-layout-0-x-flexRowContent--menu-link vtex-carousel-0-x-containerImg">
```

This would might lead to weird bugs, making it harder to debug.

## Why blocking composing with local selectors

Example of a theme that have the following folder structure inside the `styles` directory:

```
.
└── css
    └── vtex.flex-layout.css
```

If the user added to the `vtex.flex-layout.css`:

```css
.flexRowContent--menu-link {
  color: red;
  composes: flexRow--deals;
}

.flexRow--deals {
  background-color: #a7afbd;
  padding: 14px 0px;
}
```

This would generate the following class in the HTML:

```html
<div className="vtex-flex-layout-0-x-flexRowContent--menu-link vtex-flex-layout-0-x-flexRow--deals">
```

This would lead to adding a new CSS Handle to a HTML element, making it harder to debug and creating inconsistencies.

## Why creating `from styles`

We need to foster the use of the Design System. That might be possible if we let users use Tachyons classes in their custom CSS.

Example:

```css
.flexRowContent--menu-link {
  color: red;
  composes: heading-1 from styles;
}
```

This would generate the following class in the HTML:

```html
<div className="vtex-flex-layout-0-x-flexRowContent--menu-link heading-1">
```

The new syntax (`from styles`) needs to be created to be easier to validate if it's a valid Tachyons class.

# Drawbacks

This might not solve cases where the HTML element already define a Tachyons class if the composed Tachyons class might come earlier in the Tachyons CSS.

Example of an element using Tachyons and having a CSS Handle:

```html
<div class="pt1 vtex-example-0-x-container">
```

User code:

```css
.container {
  composes: pt0 from styles;
}
```

Leading to the HTML:

```html
<div class="pt1 vtex-example-0-x-container pt0">
```

This **would not** change the `padding-top` to zero because Tachyons CSS defines those properties in this order:

```css
.pt0 { padding-top: 0; }
.pt1 { padding-top: .125rem; }
```

# Alternatives

One could use the CSS Variables proposed in [CSS Variables RFC](https://github.com/vtex-apps/rfcs/pull/3), but not all values are available, example: typography tokens.

# Adoption strategy

If an app uses [vtex.css-handles](https://github.com/vtex-apps/css-handles) with a CSS Modules class, the css-handles app should deal with a composed class (example: `vtex-example-0-x-container heading-1`), given that today is not expected to have a Tachyons class in the middle of a CSS Handle.

# How we teach this

Improving the [Using CSS Handles for store customization
](https://vtex.io/docs/recipes/style/using-css-handles-for-store-customization) doc.

# Unresolved questions

n/a