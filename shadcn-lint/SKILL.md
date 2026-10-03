---
name: shadcn-lint
description: >-
  Set up and use @shadcn/lint to enforce design system rules on Tailwind v4
  projects. Covers installation (ESLint or Oxlint), rule configuration,
  contracts, and fixing violations. Use when the user wants to lint Tailwind
  classes, prevent restyling components, ban arbitrary values, enforce theme
  colors, or add @shadcn/lint to a project. Also triggers for "shadcn lint",
  "design system lint", or "no-restyle".
---

# @shadcn/lint

An agent-first linter for Tailwind v4 design systems. You define what's
allowed. When code breaks a rule, the error explains what's wrong and suggests
a fix using the project's components, variants, and theme.

- Works with **React, Svelte, and Vue**.
- Available for **ESLint and Oxlint**.
- **shadcn/ui is not required** — any Tailwind v4 project works.

> **Source:** <https://github.com/shadcn-ui/lint>

---

## Setup workflow

Follow these steps in order. For detailed installation instructions, see
[setup.md](./references/setup.md).

1. **Detect the project setup.**
   - Package manager from lockfile (`package-lock.json`, `pnpm-lock.yaml`,
     `bun.lock`).
   - Single project or monorepo/workspace.
   - Existing linter: ESLint or Oxlint. If both exist, register with the
     one that checks UI files. If neither exists, set up Oxlint (ESLint for
     `.svelte` or `.vue` files).

2. **Install `@shadcn/lint`.**
   ```bash
   # npm
   npm install -D @shadcn/lint

   # pnpm
   pnpm add -D @shadcn/lint

   # bun
   bun add -D @shadcn/lint
   ```

3. **Register the plugin** in the lint config. Do NOT enable rules yet —
   only register the plugin and parser.

4. **Turn rules off inside the component directory** so components can
   style their own internals.

5. **Enable rules** one at a time, starting with `no-restyle`. See the
   rules table below.

6. **Run the linter** and fix violations.
   ```bash
   npx eslint .     # ESLint
   npx oxlint       # Oxlint
   ```

7. **Add a lint instruction to `AGENTS.md`** (recommended) so agents
   automatically lint after making changes:
   ```md
   After making changes, run `npm run lint` and fix all errors.
   ```

---

## Rules at a glance

| Rule | What it checks | Where to disable |
|------|----------------|------------------|
| [`no-restyle`](./references/rules.md#no-restyle) | Classes passed to design-system components | Component dir |
| [`no-raw-colors`](./references/rules.md#no-raw-colors) | Palette colors, undeclared tokens, raw SVG colors | Keep enabled everywhere |
| [`no-arbitrary-values`](./references/rules.md#no-arbitrary-values) | Arbitrary values like `p-[13px]` | Component dir |
| [`no-inline-styles`](./references/rules.md#no-inline-styles) | Inline `style` props and `<style>` elements | Keep enabled everywhere |
| [`no-unknown-classes`](./references/rules.md#no-unknown-classes) | Classes Tailwind can't generate (typos) | Keep enabled everywhere |
| [`require-static-classes`](./references/rules.md#require-static-classes) | Dynamically built classNames | Component dir |

For full examples and configuration options, see [rules.md](./references/rules.md).

---

## Fixing violations

Errors include actionable guidance: what broke, what to use instead, and
where to find it. Suggestions come from the project's components, variants,
and theme.

**Agent workflow:**

1. Run the lint command.
2. Read each error message — it tells you the fix.
3. Apply the suggested fix.
4. Re-run lint to confirm zero violations.

> Agents reach zero violations in one correction round across all tested
> models (Sonnet 5, Haiku 4.5, Opus 5, GPT 5.6).

---

## Advanced configuration

- **Contracts** — Set per-component policies (allow typography on CardTitle,
  spacing on CardContent). See [contracts.md](./references/contracts.md).
- **Adoption** — Gradual rollout for existing projects with many violations.
  See [adoption.md](./references/adoption.md).
- **How it works** — Component discovery, theme tokens, variant reading.
  See [how-it-works.md](./references/how-it-works.md).
