# CLAUDE.md

This file provides guidance to Claude Code when working with code in this repository.

## Project Overview

A SvelteKit + Skeleton v4 allergen tracker webapp. Single-page app for tracking allergen tolerance levels.

## Commands

```bash
npm run dev      # start dev server
npm run build    # production build → build/
npm run preview  # preview the build locally
npm run check    # svelte-check type checking
```

Node 24 LTS (v24.15.0) via fnm.

## Stack

- **SvelteKit 2** with `@sveltejs/adapter-static` (fully prerendered SPA)
- **Svelte 5** — runes mode enforced for all non-library files (`svelte.config.js`)
- **Tailwind CSS v4** via `@tailwindcss/vite`
- **Skeleton v4** — two packages:
  - `@skeletonlabs/skeleton` — CSS-only design system (tokens, themes, utilities)
  - `@skeletonlabs/skeleton-svelte` — Svelte component library (Progress, etc.)

## Styling

Skeleton is imported in `src/routes/layout.css`:
```css
@import 'tailwindcss';
@import '@skeletonlabs/skeleton';
@import '../../node_modules/@skeletonlabs/skeleton/src/themes/catppuccin.css';
```
The theme import uses a direct `node_modules` path because Tailwind v4's CSS importer doesn't resolve wildcard package exports.

Theme and dark mode are set on `<html>` in `src/app.html`:
```html
<html lang="en" data-theme="catppuccin" class="dark">
```

Use Skeleton's surface color tokens in CSS: `rgb(var(--color-surface-800) / 1)` etc. Tailwind utility classes (`btn`, `preset-filled-primary-500`, `preset-tonal-surface`) are available from Skeleton.

## Vite config

`@skeletonlabs/skeleton-svelte` ships `.svelte` files in its dist folder. Without `ssr.noExternal`, Node throws `ERR_UNKNOWN_FILE_EXTENSION` during prerender. The fix in `vite.config.js`:
```js
ssr: { noExternal: ['@skeletonlabs/skeleton-svelte'] }
```

## Conventions

- Svelte 5 runes only — no `$:`, no `export let`, no `on:event`. Use `$props()`, `$derived()`, `$state()`, `onclick=`.
- No comments unless logic is non-obvious.
