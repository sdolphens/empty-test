@AGENTS.md

# Project Guide for Claude

## Tech Stack
- **Framework:** Next.js 16.2.4 (App Router) — see `AGENTS.md`; APIs differ from training data, consult `node_modules/next/dist/docs/` before writing Next.js code
- **Runtime:** React 19.2.4
- **Language:** TypeScript 5 (`strict: true`)
- **Styling:** Tailwind CSS 4 via `@tailwindcss/postcss`
- **Auth + DB:** Supabase (planned, not wired yet)
- **Payments:** Stripe (planned, not wired yet)
- **Hosting:** Vercel
- **Lint:** ESLint 9 flat config (`eslint-config-next/core-web-vitals` + `eslint-config-next/typescript`)
- **Path alias:** `@/*` → `./src/*`

## Coding Standards
- Strict TypeScript. Avoid `any`; if unavoidable, leave a one-line note explaining why.
- App Router only (`src/app/`). No `pages/` directory.
- Server Components by default; mark Client Components with `"use client"` at the top of the file.
- Tailwind utility classes for styling. No CSS-in-JS, no `*.module.css` unless a feature requires it.
- Use the `@/` alias for cross-directory imports.
- Routes use kebab-case folder names; React component files use PascalCase.
- Run `npm run lint` and `npm run build` before reporting work as complete.
- Keep changes focused — no drive-by refactors in feature branches.

## Allowed
- Read and edit files in `src/`, `public/`, project config (`next.config.ts`, `tsconfig.json`, `eslint.config.mjs`, `postcss.config.mjs`), and the `.claude/` context files listed below.
- Run `npm run dev`, `npm run build`, `npm run lint`.
- Read anything inside `node_modules/next/dist/docs/` for reference.
- Add npm dependencies the user has explicitly approved.

## Not Allowed
- Modify `.git/` internals or rewrite history without an explicit user instruction.
- Force-push, hard-reset shared branches, or delete branches.
- Commit `.env*` files, secrets, API keys, or Supabase / Stripe credentials.
- Rely on memory of older Next.js versions — defer to `node_modules/next/dist/docs/` for any non-trivial Next.js code.
- Install Supabase or Stripe SDKs, or scaffold those integrations, until the user requests it.
- Edit `next-env.d.ts` (Next-managed).

## Branch Rules
- `main` is protected — never commit directly.
- One feature or fix per branch. Naming: `feat/<short-name>`, `fix/<short-name>`, `chore/<short-name>`, `docs/<short-name>`.
- Push the branch and open a PR against `main`. Do not merge without user approval.
- Prefer one logical change per commit; conventional-commit prefixes (`feat:`, `fix:`, `chore:`, `docs:`).

## Session-End Requirements
Before reporting a session done:
1. Update `.claude/STATE.yml` (current status, last session summary, next focus, blockers).
2. Append a dated entry to `.claude/PROGRESS.md`.
3. Refresh `.claude/ARCHITECTURE.md` if files or routes changed.
4. Refresh `.claude/BUILD_INDEX.md` if features were added, changed, or removed.
5. Commit on a feature branch, push, and open (or update) a PR.
