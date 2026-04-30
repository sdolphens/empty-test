# Progress Log

Reverse-chronological session log. Newest entry on top.

## 2026-04-30 — Code quality tooling added

- Installed Prettier 3.8.3 and `eslint-config-prettier` 10.1.8. Wired the flat preset (`eslint-config-prettier/flat`) into `eslint.config.mjs` so formatting rules don't conflict with ESLint.
- Authored `.prettierrc.json` (semi, double quotes, trailing-comma all, printWidth 100, tabWidth 2, lf endings) and `.prettierignore` (ignores `.next`, `out`, `build`, `node_modules`, `coverage`, `package-lock.json`, `next-env.d.ts`, `public`, `.claude`).
- Installed Husky 9.1.7 (`npx husky init`) and lint-staged 16.4.0. Pre-commit hook at `.husky/pre-commit` runs `npx lint-staged`; lint-staged config lives in `package.json` and runs `eslint --fix` + `prettier --write` on JS/TS files and `prettier --write` on JSON/CSS/MD/YAML files.
- Added `lint:fix`, `format`, `format:check`, and `prepare` (= `husky`) scripts to `package.json`.
- Verified: `npm run lint` clean, `npm run build` succeeds, and the pre-commit hook fired on the chore commit (lint-staged ran the configured tasks against the staged config files).

## 2026-04-30 — Initial context system created

- Authored `CLAUDE.md` (tech stack, coding standards, allow / not-allowed lists, branch rules, session-end requirements). Preserved the existing `@AGENTS.md` import so the Next.js version-specific rules still load first.
- Added `.claude/STATE.yml`, `.claude/BUILD_INDEX.md`, `.claude/ARCHITECTURE.md`, and `.claude/PROGRESS.md`.
- Updated `.gitignore` from `.claude/` to `.claude/*` plus negations for the four context files, so `.claude/settings.local.json` stays ignored while context is tracked.
- No application code touched (`src/`, `public/`, and config files unchanged).
