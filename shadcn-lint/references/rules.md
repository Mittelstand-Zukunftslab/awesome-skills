# Rules — @shadcn/lint

Enable each rule at `warn` or `error`. Start with `no-restyle`, then add
more rules as needed.

---

## no-restyle

Use component variants for appearance. `className` is only for changes
your policy allows. The error lists the component's variants and the file
that defines them.

### Setup

```js
"shadcn/no-restyle": ["error", { allow: ["layout"] }]
```

Turn this rule **off** inside the component directory.

### Examples

```tsx
// ✅ Allowed — layout classes and variants.
<Button className="mt-4 w-full">Save changes</Button>
<Button size="lg" variant="destructive">Delete account</Button>

// ❌ Reported — padding, colors, and shape override the component.
<Button className="p-4 bg-pink-500 rounded-full">Save changes</Button>
```

Spacing errors suggest sizes defined by the component. Appearance errors
list its variants. The linter reads those values from code.

### Per-component contracts

```js
"shadcn/no-restyle": ["error", {
  allow: ["layout"],
  contracts: [
    { pattern: "^CardTitle$", allow: ["layout", "typography"] },
    { pattern: "^CardContent$", allow: ["layout", "spacing"] },
  ],
}]
```

```tsx
// ✅ Allowed.
<CardTitle className="text-lg">Account settings</CardTitle>
<CardContent className="p-6">Profile details</CardContent>

// ❌ Reported — color is not allowed by either contract.
<CardTitle className="text-pink-500">Account settings</CardTitle>
```

See [contracts.md](./contracts.md) for the full contract API.

---

## no-raw-colors

Use colors from your theme. Reports raw Tailwind palette colors, undeclared
color tokens, and literal colors in SVG attributes. Suggests nearby theme
colors when it can resolve their values.

### Setup

```js
"shadcn/no-raw-colors": "error"
```

Keep this rule enabled **everywhere**, including inside the component directory.

### Examples

Given a theme that declares `primary` and `muted-foreground`:

```tsx
// ✅ Allowed — declared theme tokens.
<div className="bg-primary text-muted-foreground">Account settings</div>

// ❌ Reported — raw palette color.
<div className="bg-pink-500">Account settings</div>

// ❌ Reported — no highlight token in theme.
<div className="bg-highlight">Account settings</div>
```

### SVG attributes

```tsx
// ✅ Allowed.
<svg className="text-primary" fill="currentColor" />
<svg fill="var(--color-primary)" />

// ❌ Reported — literal color values.
<svg fill="#ec4899" />
<path stroke="red" />
```

Accepted color names: `white`, `black`, `transparent`, `current`, `inherit`.

---

## no-arbitrary-values

Use theme tokens and scale values instead of arbitrary values like
`p-[13px]`. Suggests scale equivalents when they exist.

### Setup

```js
// Check appearance values, allow layout arbitrary values.
"shadcn/no-arbitrary-values": ["error", { allow: ["layout"] }]

// Or check everything (including layout):
"shadcn/no-arbitrary-values": "error"
```

Turn this rule **off** inside the component directory (components may need
structural values).

### Examples

```tsx
// ✅ Allowed.
<div className="p-4 rounded-lg bg-primary">Account settings</div>
<div className="w-[320px]">Sidebar</div>  // ← only with allow: ["layout"]

// ❌ Reported.
<div className="p-[13px] rounded-[10px] bg-[#333]">Account settings</div>
```

The error includes scale replacements:
```
"p-[13px]" hardcodes an off-token value. Use "p-3.25" instead (same value, on the scale).
```

### What is NOT an arbitrary value

- Arbitrary variants: `data-[state=open]:flex`
- Variable shorthands: `bg-(--brand)`

---

## no-inline-styles

Use classes for styling. Pass dynamic values through CSS custom properties.
Reports inline `style` props, hardcoded colors in custom properties,
unreadable style objects, and `<style>` elements.

### Setup

```js
"shadcn/no-inline-styles": "error"
```

Keep this rule enabled **everywhere**.

### Examples

```tsx
// ✅ Allowed — class-based styling and CSS variable for dynamic values.
<div className="text-primary">Account settings</div>
<div
  className="w-(--panel-width)"
  style={{ "--panel-width": `${width}px` } as React.CSSProperties}
/>

// ❌ Reported — ordinary inline property.
<div style={{ color: "var(--color-primary)" }}>Account settings</div>

// ❌ Reported — hardcoded color in a custom property.
<div style={{ "--label-color": "#ec4899" }}>Account settings</div>
```

### Style objects

Same-file objects and lookup values one hop deep are resolved:

```tsx
const colors = { accent: "#ec4899" }

// ❌ Reported — the lookup contains a raw color.
<div style={{ "--label-color": colors.accent }} />

// ❌ Reported — imported object cannot be checked.
import { panelStyle } from "./styles"
<div style={panelStyle} />
```

---

## no-unknown-classes

Catch class names that produce no CSS. Checks your installed Tailwind v4
with your theme, custom utilities, variants, and plugins. Suggests spelling
corrections.

### Setup

```js
// Start with warnings to find external class exceptions.
"shadcn/no-unknown-classes": "warn"
```

Keep this rule enabled **everywhere**.

### Examples

```tsx
// ✅ Allowed.
<div className="flex-col rounded-lg hover:flex" />

// ❌ Reported.
<div className="flex-cols rounded-huge hovr:flex" />
```

Error message:
```
"flex-cols" is not a class this project's Tailwind knows. Did you mean "flex-col"?
```

### External stylesheets

Allow classes from CSS outside the theme's import graph:

```js
"shadcn/no-unknown-classes": ["error", {
  allow: ["editor-root", "editor-toolbar"],
}]
```

---

## require-static-classes

Keep component class values readable by the linter. Reports dynamically
built classNames that the other rules can't check.

### Setup

```js
"shadcn/require-static-classes": "error"
```

Turn this rule **off** inside the component directory (components often
call their own variant functions).

### Examples

```tsx
// ✅ Allowed — static strings and conditional choices.
<Button className="mt-4">Save changes</Button>
<Button className={wide ? "w-full" : "w-auto"}>Save changes</Button>
<Button className={cn("mt-4", wide && "w-full")}>Save changes</Button>

// ❌ Reported — dynamically built class.
<Button className={`bg-${color}`}>Save changes</Button>

// ❌ Reported — imported object cannot be checked.
<Button className={buttonClasses}>Save changes</Button>
```

Same-file constants and never-reassigned variables are read one hop deep
and pass.
