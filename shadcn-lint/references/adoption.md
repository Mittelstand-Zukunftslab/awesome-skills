# Adoption — @shadcn/lint

A gradual strategy for adding `@shadcn/lint` to existing projects with
many violations. Do NOT enable `error` severity on a codebase with
hundreds of violations — it will break CI.

---

## Step 1: Start with warnings

Set `no-restyle` to `warn` in the main rules:

```js
"shadcn/no-restyle": ["warn", { allow: ["layout"] }]
```

Keep the component-directory override that turns the rule off, then run:

```bash
npx eslint .    # ESLint
npx oxlint      # Oxlint
```

Look for **repeated patterns** (e.g., padding on buttons). Fix those
patterns first before working through individual files.

---

## Step 2: Cap the warning count

Use the measured count in the lint script to prevent regressions. For
example, if the project has 287 warnings:

```json
{
  "scripts": {
    "lint": "eslint . --max-warnings 287"
  }
}
```

Run this in CI. It fails when the total increases. Lower the cap as you
fix findings. Oxlint supports the same `--max-warnings` flag.

### ESLint bulk suppressions

For a per-file, per-rule baseline, ESLint supports
[bulk suppressions](https://eslint.org/docs/latest/use/suppressions):

```bash
# Record existing violations.
npx eslint --suppress-all .

# Remove resolved entries.
npx eslint --prune-suppressions .
```

These apply to errors, not warnings.

---

## Step 3: Keep new code strict

Enforce the rule in new code while leaving legacy code at `warn`. Add
these config objects after the main config and before the component
directory override:

```js
// New code — strict.
{
  files: ["app/**", "features/**"],
  rules: { "shadcn/no-restyle": ["error", { allow: ["layout"] }] },
},

// Legacy code — warnings only.
{
  files: ["legacy/**"],
  rules: { "shadcn/no-restyle": ["warn", { allow: ["layout"] }] },
},
```

To exempt a folder entirely:

```js
{ files: ["marketing/**"], rules: { "shadcn/no-restyle": "off" } }
```

---

## Step 4: Add more rules

Roll out rules one at a time in this order:

| Order | Rule | Why |
|-------|------|-----|
| 1 | `no-restyle` | Biggest impact — prevents component restyling |
| 2 | `no-raw-colors` | Enforces theme tokens |
| 3 | `no-unknown-classes` | Catches typos (start at `warn`) |
| 4 | `no-arbitrary-values` | Enforces design scale |
| 5 | `no-inline-styles` | Removes inline styles |
| 6 | `require-static-classes` | Ensures linter can read all classNames |

When enabling `no-arbitrary-values` and `require-static-classes`, turn
them off in the component directory too:

```js
// Component directory override — add new rules here.
{
  files: ["components/ui/**"],
  rules: {
    "shadcn/no-restyle": "off",
    "shadcn/no-arbitrary-values": "off",
    "shadcn/require-static-classes": "off",
  },
},
```

---

## Verification

After each rollout step:

1. Run the linter.
2. Confirm CI passes with the current `--max-warnings` cap.
3. Fix the most common violation pattern.
4. Lower the cap.
5. Repeat until warnings reach zero, then switch to `error`.
