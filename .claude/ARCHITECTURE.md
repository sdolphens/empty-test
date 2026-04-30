# Architecture

Snapshot of the current scaffolded state. Refresh whenever files or routes change.

## Top-level layout

```
empty-test/
├── .claude/                # Context system (this folder)
│   ├── ARCHITECTURE.md
│   ├── BUILD_INDEX.md
│   ├── PROGRESS.md
│   └── STATE.yml
├── .github/
│   └── workflows/
│       └── ci.yml          # Lint + type check + build on push / PR to main
├── .husky/
│   └── pre-commit          # Runs `npx lint-staged`
├── public/                 # Static assets
│   ├── file.svg
│   ├── globe.svg
│   ├── next.svg
│   ├── vercel.svg
│   └── window.svg
├── src/
│   └── app/                # Next.js App Router root
│       ├── favicon.ico
│       ├── globals.css     # Tailwind layer + global styles
│       ├── layout.tsx      # Root layout (Geist + Geist_Mono fonts)
│       └── page.tsx        # Default landing page
├── .prettierignore
├── .prettierrc.json        # Prettier defaults (semi, double quotes, printWidth 100)
├── AGENTS.md               # Next.js version-specific rules (imported by CLAUDE.md)
├── CLAUDE.md               # Project guide for Claude
├── README.md
├── eslint.config.mjs       # ESLint 9 flat config (Next + TS + prettier-disable)
├── next-env.d.ts           # Next-managed TS env (do not edit)
├── next.config.ts          # Next.js config (no overrides yet)
├── package.json            # `lint-staged` config block lives here
├── package-lock.json
├── postcss.config.mjs      # Tailwind v4 via @tailwindcss/postcss
└── tsconfig.json           # strict; "@/*" → "./src/*"
```

## Routes

| Path | File | Type | Description |
| --- | --- | --- | --- |
| `/` | `src/app/page.tsx` | Server Component | Default `create-next-app` landing page |

## Entry points

- **Root layout:** `src/app/layout.tsx` — sets `<html>`/`<body>`, loads Geist fonts via `next/font/google`, applies `h-full antialiased` and a flex column body.
- **Global styles:** `src/app/globals.css`.
- **Path alias:** `@/*` resolves to `./src/*` (see `tsconfig.json`).

## Not yet present

- Supabase client / server helpers (`src/lib/supabase/*`).
- Stripe checkout, customer portal, or webhook handlers.
- API route handlers (`src/app/api/`).
- Middleware (`src/middleware.ts`).
- Test setup or test files.
- Environment variable schema / loader.
