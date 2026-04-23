# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
pnpm install                  # install dependencies
pnpm dev                      # dev server at http://localhost:3000
pnpm build                    # production build
pnpm preview                  # preview production build
pnpm lint                     # ESLint check
pnpm exec eslint . --fix      # ESLint auto-fix (resolves most errors)
pnpm typecheck                # vue-tsc type check
```

This project uses **pnpm** exclusively — do not use npm or yarn.

**Local dev env vars:** Copy `.env.example` to `.env` and fill in real values before running `pnpm dev`. Required:
```
NUXT_PUBLIC_SUPABASE_URL=https://your-project.supabase.co
NUXT_PUBLIC_SUPABASE_KEY=your-legacy-anon-key-here
```

**ESLint rules of note:** `commaDangle: 'never'`, `braceStyle: '1tbs'`, max 1 statement per line, unused function args must be prefixed with `_` (e.g. `_to` not `to`). CI runs lint + typecheck on every push — fix all errors before pushing.

## Architecture

**Stack:** Nuxt 4 · Nuxt UI 4 (Tailwind CSS v4) · @nuxtjs/supabase · TypeScript

**App directory layout** (`app/`):
- `pages/` — file-based routing: `index.vue` (public landing), `login.vue`, `dashboard.vue`, `edit.vue`, `auth/confirm.vue`
- `middleware/auth.ts` — named middleware that redirects unauthenticated users to `/login`; applied per-page via `definePageMeta({ middleware: 'auth' })`
- `types/database.types.ts` — placeholder for Supabase-generated DB types (currently empty, to be generated)
- `assets/css/main.css` — global styles: Tailwind + Nuxt UI imports, custom CSS token system (see below)

**Supabase auth flow:**
- `@nuxtjs/supabase` handles session management automatically
- Login/signup on `/login` using `supabase.auth.signInWithPassword` / `signUp`
- Email confirmation lands on `/auth/confirm`, which calls `getSession()` then redirects to `/dashboard`
- `nuxt.config.ts` sets `redirectOptions`: unauthenticated users are sent to `/login`; `/` and `/login` are excluded from the redirect guard

**Supabase database:**
The app reads/writes a `sessions` table with columns: `id`, `title`, `blocks` (JSON array), `user_id`, `updated_at`. All queries are scoped to the authenticated user.

**The builder (`public/builder/index.html`):**
The core editing experience is a standalone vanilla HTML/JS app served as a static file. `edit.vue` embeds it in a full-page `<iframe src="/builder/index.html">`. Communication between the Nuxt shell and the iframe is done by directly reading/writing `iframe.contentWindow.blocks` and calling `iframe.contentWindow.renderCanvas()` — there is no `postMessage` abstraction. Theme synchronisation (dark mode) is handled by injecting a `<style id="__nuxt-theme">` element into the iframe's `<head>` from `edit.vue`.

**Design system:**
All colours use CSS custom properties (`--sage`, `--parchment`, `--ink`, `--terracotta`, `--gold`, etc.) defined in `main.css`. Light and dark mode are both defined there — dark mode activates via `html.dark`. The Nuxt UI primary colour is `green` and neutral is `stone`, mapped to custom earthy-toned palettes defined in `@theme static`. Typography: **Lora** (serif, headings) and **DM Sans** (sans-serif, body), loaded from Google Fonts.

**Page layout pattern:**
All pages use `definePageMeta({ layout: false })` — there is no shared Nuxt layout file. Each page manages its own header/nav directly.

## Deployment

**Platform:** Cloudflare Workers (not Pages) — live at `https://interactive-session-builder.alvintan-nc.workers.dev`

**Nitro preset:** `cloudflare-module` (in `nuxt.config.ts`) — generates a Worker module at `.output/server/index.mjs` with static assets at `.output/public/`. Do not switch to `cloudflare-pages` preset; it generates Pages-style output incompatible with the Workers deployment model used here.

**Wrangler config (`wrangler.jsonc`):**
```jsonc
{
  "name": "interactive-session-builder",
  "main": ".output/server/index.mjs",
  "compatibility_date": "2025-01-15",
  "compatibility_flags": ["nodejs_compat"],
  "assets": { "directory": ".output/public" }
}
```

**CI/CD:** GitHub Actions (`.github/workflows/ci.yml`) runs lint + typecheck on every push. Cloudflare Workers CI/CD auto-deploys on push to `main` via the GitHub integration — build command `pnpm run build`, deploy command `npx wrangler deploy`.

**Environment variables** (set in Cloudflare Workers → Settings → Variables and Secrets):
- `NUXT_PUBLIC_SUPABASE_URL` — Supabase project URL (Secret)
- `NUXT_PUBLIC_SUPABASE_KEY` — Supabase legacy anon key (Secret). Use the **legacy anon key**, not the newer Publishable key — `@nuxtjs/supabase` v2 was built around the JWT-based anon format.

The `NUXT_PUBLIC_` prefix is required (not plain `SUPABASE_URL`) because Nuxt only allows runtime overrides of `runtimeConfig.public` values via that prefix. Plain `SUPABASE_URL` is only read at build time.

**Build token:** Managed in Cloudflare → Workers → Build settings → API token. If it shows as "deleted or rolled", create a new one from that same screen. Do not set `CLOUDFLARE_API_TOKEN` as a runtime env var — Cloudflare injects it automatically from the build token and setting it manually causes auth conflicts.

**Cloudflare Access:** Two Zero Trust Access applications were deleted to allow public access. If the login-code prompt reappears, go to Cloudflare Zero Trust → Access → Applications and delete any entries for this Worker.
