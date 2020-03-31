- Start Date: 2020-03-31
- Implementation PR: (leave this empty)
- Store Discussion Issue: (leave this empty)

# Summary

Make it possible to use Tachyons values in CSS files.

# Basic example

```css
.container {
  color: var(--c-on-base);
  margin-right: var(--spacing-1);
}

@media screen and (min-width: var(--breakpoint-ns)) {
  .container {
    color: var(--c-muted-1);
    margin-right: var(--spacing-2);
  }
}
```

# Motivation

This makes CSS customizations consistent with the Design System of the store. We avoid hard-coding HEX values and breakpoint values in favor of the token values.

# Detailed design

The Styles builder uses [VTEX Tachyons Generator](https://github.com/vtex/tachyons-generator) to generate the store's Tachyons CSS files. This library have an option called `compileVars` that we can set it to `false` generating a CSS with CSS variables.

Example cherry-picking the important parts:

```css
:root { --b-action-primary: #1346d8; }
.b--action-primary { border-color: var( --b-action-primary ); }
```

Compiling the Tachyons with `compileVars: false` will make it possible to reference those variables in CSS.

Tachyons Generator needs to be changed to add some variables that are useful and not available as a CSS variable even with this option. The missing variables are:

- **Breakpoints:** `--breakpoint-s`, `--breakpoint-ns`, `--breakpoint-m`, `--breakpoint-l`, and `--breakpoint-xl`
- **Spacing:** `--spacing-1`, `--spacing-2`, `--spacing-3`, `--spacing-4`, `--spacing-5`, `--spacing-6`, `--spacing-7`, `--spacing-8`, `--spacing-9`, `--spacing-10`, and `--spacing-11`

CSS Variables does not work in IE11 so we need to add a [polyfil](https://github.com/nuxodin/ie11CustomProperties).

# Drawbacks

This will increase the size of Tachyons CSS for two main reasons:

- Size of the variable declarations, example: `:root { --b-action-primary: #1346d8; ...` 
- The usage of variables makes use of more characters then using the values directly
   - Before: `color: #ffffff;`
   - After: `color: var(--c-on-base);`

# Alternatives

We considered replacing the variables values in build time, but that is not possible because the user might change Tachyons anytime through the `/admin/cms/styles` interface.

# Adoption strategy

Adoption would be gradual. It opens a path to increase the usage of Tachyons tokens.

# How we teach this

A good documentation on the Tachyons tokens should be written so those values are more widely used throughout the store development.

# Unresolved questions

n/a
