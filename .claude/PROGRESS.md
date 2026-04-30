# Progress Log

Reverse-chronological session log. Newest entry on top.

## 2026-04-30 — Initial context system created

- Authored `CLAUDE.md` (tech stack, coding standards, allow / not-allowed lists, branch rules, session-end requirements). Preserved the existing `@AGENTS.md` import so the Next.js version-specific rules still load first.
- Added `.claude/STATE.yml`, `.claude/BUILD_INDEX.md`, `.claude/ARCHITECTURE.md`, and `.claude/PROGRESS.md`.
- Updated `.gitignore` from `.claude/` to `.claude/*` plus negations for the four context files, so `.claude/settings.local.json` stays ignored while context is tracked.
- No application code touched (`src/`, `public/`, and config files unchanged).
