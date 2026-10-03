# Setup — @shadcn/lint

## Requirements

- **Node.js 20.19** or later
- **ESLint 9.30** or later (if using ESLint)
- **Oxlint 1.80** or later (if using Oxlint)
- **Tailwind CSS v4**

---

## Inspect the project

Before installing, gather this information:

1. **Package manager** — Check for `package-lock.json` (npm), `pnpm-lock.yaml`
   (pnpm), or `bun.lock` (bun).
2. **Single project or monorepo** — Look for `workspaces` in `package.json` or
   `pnpm-workspace.yaml`. In a monorepo, find the apps, shared UI packages,
   lint configs, and task runner.
3. **Existing linter** — Check for `eslint.config.*` (ESLint flat config) or
   `oxlintrc.json` / `.oxlintrc.json` (Oxlint).
   - If both exist, register the plugin with the one that checks UI files.
   - If neither exists, set up **Oxlint** by default. Use **ESLint** when the
     UI files are `.svelte` or `.vue`.
4. **Component directories** — Check `components.json` for the UI path. Without
   it, look in `components/ui` or `src/components/ui`.
5. **Tailwind v4 theme** — Find the CSS file that imports Tailwind and declares
   `@theme` tokens.

---

## Install

```bash
# npm
npm install -D @shadcn/lint

# pnpm
pnpm add -D @shadcn/lint

# bun
bun add -D @shadcn/lint
```

For ESLint, also install the parser if not already present:

```bash
npm install -D @typescript-eslint/parser
```

In a monorepo, install dependencies in the package that owns the lint
configuration.

---

## Register the plugin — ESLint (flat config)

Create or update `eslint.config.mjs`:

```js
import { plugin as shadcn } from "@shadcn/lint"
import tsParser from "@typescript-eslint/parser"

export default [
  {
    files: ["**/*.{ts,tsx}"],
    languageOptions: { parser: tsParser },
    plugins: { shadcn },
    settings: {
      shadcn: {
        // Optional: set if your UI is not at @/components/ui
        // ui: "@/ds",
      },
    },
    rules: {
      "shadcn/no-restyle": ["error", { allow: ["layout"] }],
      "shadcn/no-raw-colors": "error",
    },
  },
  // Turn rules OFF inside the component directory
  {
    files: ["components/ui/**"],
    rules: {
      "shadcn/no-restyle": "off",
    },
  },
]
```

### Svelte

```js
import { plugin as shadcn } from "@shadcn/lint"
import svelteParser from "svelte-eslint-parser"
import tsParser from "@typescript-eslint/parser"

export default [
  {
    files: ["**/*.svelte"],
    languageOptions: {
      parser: svelteParser,
      parserOptions: { parser: tsParser },
    },
    plugins: { shadcn },
    rules: {
      "shadcn/no-restyle": ["error", { allow: ["layout"] }],
    },
  },
]
```

### Vue

```js
import { plugin as shadcn } from "@shadcn/lint"
import vueParser from "vue-eslint-parser"
import tsParser from "@typescript-eslint/parser"

export default [
  {
    files: ["**/*.vue"],
    languageOptions: {
      parser: vueParser,
      parserOptions: { parser: tsParser },
    },
    plugins: { shadcn },
    rules: {
      "shadcn/no-restyle": ["error", { allow: ["layout"] }],
    },
  },
]
```

---

## Register the plugin — Oxlint

Create or update `oxlintrc.json`:

```json
{
  "jsPlugins": ["@shadcn/lint"],
  "rules": {
    "shadcn/no-restyle": ["error", { "allow": ["layout"] }],
    "shadcn/no-raw-colors": "error"
  },
  "overrides": [
    {
      "files": ["components/ui/**"],
      "rules": {
        "shadcn/no-restyle": "off"
      }
    }
  ]
}
```

---

## Settings

Configure `settings.shadcn` when the project uses a non-standard setup:

| Setting | Default | Purpose |
|---------|---------|---------|
| `ui` | `@/components/ui` | Import path to the UI directory |
| `componentImports` | Auto-detected | Regex patterns for component import paths |
| `ignoreImports` | `[]` | Import patterns to skip before recognition |
| `mergeFunctions` | Built-ins (`cn`, `cx`, `clsx`, `twMerge`, etc.) | Additional class-merge functions the linter should recognize |
| `variantFunctions` | `cva`, `tv` | Additional variant functions beyond the built-ins |
| `note` | `""` | Custom text appended to every rule's error message |

With `components.json` present, the linter reads the UI path, theme CSS,
and import aliases automatically. Only set these when the auto-detection
is wrong.

---

## Monorepo setup

In a monorepo, install `@shadcn/lint` in the package that owns the lint
configuration. Configure the UI path with a workspace package prefix:

```js
settings: {
  shadcn: {
    ui: "@workspace/ui/components",
  },
}
```

Override the component directory to match the package layout:

```js
// Turn rules off inside the UI package's component directory.
{
  files: ["packages/ui/src/components/**"],
  rules: {
    "shadcn/no-restyle": "off",
  },
},
```

Make sure the lint command runs from the workspace root, or configure
the linter's working directory in your task runner.

---

## Verify setup

Run the linter to confirm the plugin is registered:

```bash
# ESLint
npx eslint .

# Oxlint
npx oxlint
```

If configuration is correct, you'll see `shadcn/` prefixed diagnostics for
any violations. If the UI alias cannot be resolved, the linter warns with
the setting to fix.
