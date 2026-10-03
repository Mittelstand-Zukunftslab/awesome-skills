# Contracts & Shared Options — @shadcn/lint

## Shared options

Every rule accepts `message`. Every rule except `require-static-classes`
also accepts `allow`, `deny`, and `contracts`.

| Option | Purpose |
|--------|---------|
| `allow` | Allows matching classes or properties through the rule |
| `deny` | Removes matches from `allow`. Without `allow`, permits everything else |
| `contracts` | Sets a per-component policy |
| `message` | Replaces the rule's default error text |

### allow / deny interaction

| Configuration | Policy |
|---------------|--------|
| Neither `allow` nor `deny` | No policy exceptions |
| `allow: [...]` | Only matching classes are exempt |
| `allow: []` | No policy exceptions |
| `deny: [...]` without `allow` | Everything except these matches is exempt |
| `deny: []` without `allow` | Everything is exempt |

### Example

```js
"shadcn/no-restyle": ["error", {
  allow: ["layout"],
  deny: ["w-*"],
}]
```

Margin passes. Width does not. Appearance classes are still reported.

---

## Contracts

A contract sets a policy for components matching a regex `pattern`. Each
contract replaces the keys it writes and inherits the rest from the
top-level rule.

### Syntax

```js
contracts: [
  {
    pattern: "^Button$",        // regex on resolved component name
    allow: ["w-full", "mt-*"],  // override allow for this component
    deny: [],                   // override deny (clears inherited denial)
    message: "Custom guidance", // optional custom error text
  },
]
```

### Pattern matching

- `pattern` is a regex on the **resolved** component name.
- `^Button$` matches only `Button`.
- `Button` (without anchors) also matches `IconButton`.
- For `no-inline-styles`, contracts match the component name as written
  (`Card.Title` in Svelte, `CardTitle` or `card-title` in Vue).
- Lowercase elements use the top-level policy, not contracts.

### Inheritance

A contract replaces only the keys it writes. Unset keys inherit from the
top-level rule:

```js
"shadcn/no-restyle": ["error", {
  allow: ["layout"],
  contracts: [
    // Inherits allow: ["layout"] and adds typography.
    { pattern: "^CardTitle$", allow: ["layout", "typography"] },
    // Only allows layout + spacing. Does NOT inherit typography from above.
    { pattern: "^CardContent$", allow: ["layout", "spacing"] },
  ],
}]
```

### Last match wins

If several contracts match the same component, only the **last** one
applies. It does not inherit from earlier contracts.

### Full example — layered policies

Allow spacing with margin. Allow full width. Keep size and shape in the
Button:

```js
"shadcn/no-restyle": ["error", {
  allow: ["layout"],
  contracts: [
    { pattern: "^Button$", allow: ["w-full", "mt-*", "mb-*"] },
  ],
}]
```

```tsx
// ✅ Allowed.
<Button size="lg" className="mt-4 md:w-full" />

// ❌ Reported — padding and shape not allowed.
<Button className="p-4 hover:rounded-full" />

// ❌ Reported — custom height and fixed width not allowed.
<Button className="md:h-12 w-48" />
```

---

## Custom messages

Replace the default error text with a static string:

```js
"shadcn/no-restyle": ["error", {
  allow: ["layout"],
  message: "Use a variant prop instead of className for appearance changes.",
}]
```

Or use a **structured message object** with category keys and placeholders:

```js
"shadcn/no-restyle": ["error", {
  allow: ["layout"],
  message: {
    spacing: "Use a {{component}} size ({{sizes}}) instead of padding. Defined in {{file}}.",
    color: "Use a variant prop for color changes on {{component}}.",
  },
}]
```

### Placeholders

| Placeholder | Replaced with |
|-------------|---------------|
| `{{component}}` | The resolved component name |
| `{{sizes}}` | The component's size variants |
| `{{variants}}` | The component's variant names |
| `{{file}}` | The file that defines the component |

Per-contract messages override the rule-level message for matching
components.

---

## Component import settings

### componentImports

Regex patterns that identify which imports are design-system components:

```js
settings: {
  shadcn: {
    componentImports: ["^@/components/ui$"],
  },
}
```

### ignoreImports

Import patterns to skip before recognition. Useful for third-party
wrappers:

```js
settings: {
  shadcn: {
    ignoreImports: ["^@radix-ui/"],
  },
}
```

### ui

The import path to the UI directory. Auto-detected from `components.json`:

```js
settings: {
  shadcn: {
    ui: "@/ds",
  },
}
```

This recognizes `@/ds` and `@/ds/button` across the rules. Only set this
when auto-detection fails.

### mergeFunctions

Additional class-merge functions the linter should recognize. The built-in
list includes `cn`, `cx`, `clsx`, `twMerge`, and `twJoin`:

```js
settings: {
  shadcn: {
    mergeFunctions: ["myMerge"],
  },
}
```

### variantFunctions

Additional variant functions beyond the built-in `cva` and `tv`:

```js
settings: {
  shadcn: {
    variantFunctions: ["myVariants"],
  },
}
```

### note

Custom text appended to every rule's error message. Useful for linking to
a design system guide:

```js
settings: {
  shadcn: {
    note: "See https://wiki.example.com/design-system for guidance.",
  },
}
```
