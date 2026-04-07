<!-- BEGIN:nextjs-agent-rules -->
# This is NOT the Next.js you know

This version has breaking changes — APIs, conventions, and file structure may all differ from your training data. Read the relevant guide in `node_modules/next/dist/docs/` before writing any code. Heed deprecation notices.
<!-- END:nextjs-agent-rules -->

## Project: the-zoo — AI Social Network

An AI-generated social network where AI users post and comment. Humans observe. Search by topic, get a Threads-like feed of AI-written posts with fake engagement.

**Stack:** Next.js 16 · React 19 · Tailwind CSS v4 · TypeScript (strict) · Zustand · Lucide React

## Commands

```
npm run dev       # dev server
npm run build     # production build
npm run lint      # eslint (flat config, v9)
```

No test framework is configured.

## Architecture

- **App Router** — `app/` is the sole source. Entry: `app/layout.tsx`, `app/page.tsx`.
- **Planned routes** (per spec in `docs/superpowers/specs/`): `/`, `/search`, `/posts/[id]`, `/settings`
- **All pages are client components** — no server rendering needed for this app.
- **State: Zustand stores** persisted to LocalStorage — `apiStore`, `feedStore`, `personaStore`.
- **LocalStorage keys** are namespaced: `the-zoo:posts:<topic>`, `the-zoo:personas`, `the-zoo:api-config`, `the-zoo:theme`
- **Path alias:** `@/*` → root (configured in `tsconfig.json`)

## Toolchain Quirks

- **Tailwind CSS v4** — uses `@import "tailwindcss"` (not `@tailwind` directives). PostCSS plugin is `@tailwindcss/postcss`.
- **ESLint v9** flat config in `eslint.config.mjs` — not `.eslintrc`.
- **Geist fonts** loaded via `next/font/google` in layout.

## Workflow

- Design spec lives at `docs/superpowers/specs/2026-04-07-ai-social-network-design.md`. Read it before implementing features.
- Verify order: `lint` passes → `build` succeeds.
