---
name: agents-generator
description: "Generate a complete, project-specific AGENTS.md compatible with the agents.md standard (60k+ open-source projects). Also generates platform-specific files (.cursorrules, Copilot instructions, GEMINI.md) for detected platforms. Use when setting up AI agent rules, creating project conventions for Claude Code, Cursor, Copilot, OpenCode, or any agent that reads AGENTS.md. Supports three modes: full (rich AGENTS.md + companion rule files + multi-platform output), minimal (single 30-line AGENTS.md following the agents.md standard), and update (diff existing, regenerate only changed). Detects monorepos for nested files. Extracts design tokens from config files. Includes confidence scoring, managed blocks, audit logging, and state memory. Every command and convention is derived from the project's real package.json, config files, and directory structure — never generic templates."
license: MIT
allowed-tools: Read Write Edit Bash(ls:*) Bash(git:*) Bash(tree:*) Bash(find:*) Grep Glob WebFetch
metadata:
  author: OJPalenzuela
  version: "1.1.2"
---

# Generate AGENTS.md for this project

Read the project's package.json, config files, and directory structure. Then generate the files below. **Use real data from the project — never placeholders or guesses.**

## Step 1 — Detect the project

### CRITICAL — Detect package manager FIRST

Check for lockfiles in the project root. This determines ALL commands in the output:

```
bun.lock or bun.lockb  → bun   → commands: bun install, bun dev, bun run lint
pnpm-lock.yaml         → pnpm  → commands: pnpm install, pnpm dev, pnpm lint
package-lock.json      → npm   → commands: npm install, npm run dev, npm run lint
yarn.lock              → yarn  → commands: yarn install, yarn dev, yarn lint
```

**NEVER default to npm.** If no lockfile exists, check the `packageManager` field in `package.json`. If that's also absent, ask the user.

### Then detect everything else
| Framework | `next`→Next.js (check `app/` vs `pages/` for Router type), `@nestjs/core`→NestJS, `vite`→Vite |
| Language | `tsconfig.json`→TypeScript, check `strict: true` |
| CSS | `tailwindcss`→Tailwind, `components.json`→shadcn/ui, `.module.css`→CSS Modules |
| Testing | `vitest`→Vitest, `jest`→Jest, `playwright`→Playwright, `cypress`→Cypress |
| ORM | `prisma`→Prisma (read schema for provider), `drizzle-orm`→Drizzle |
| Validation | `zod`→Zod, `yup`→Yup, `class-validator`→class-validator |
| State | `zustand`→Zustand, `@reduxjs/toolkit`→Redux, `jotai`→Jotai |
| Monorepo | `workspaces` in package.json, `turbo.json`→Turborepo, `nx.json`→Nx |
| Platform files | `.cursorrules`, `.github/copilot-instructions.md`, `GEMINI.md`, `.windsurfrules` |
| Auth | `next-auth`→NextAuth, `@clerk/nextjs`→Clerk, `lucia`→Lucia |
| i18n | `next-intl`→next-intl, `react-i18next`→react-i18next |
| Forms | `react-hook-form`, `formik`, `@tanstack/react-form` |
| API pattern | `"use server"` in files→Server Actions, `@trpc/*`→tRPC, `graphql`→GraphQL |
| Claude | `.claude/` directory or `CLAUDE.md` exists |

**Confidence score**: each detected category = ~6 points (16 categories = 96 max). If < 80, flag uncertain ones.

## Step 2 — Generate AGENTS.md

Write `AGENTS.md` at project root. Wrap all generated content in `<!-- AGENTS-GENERATED-START -->` / `<!-- AGENTS-GENERATED-END -->`. Use EXACTLY this structure, filling each section with real project data:

```markdown
# AGENTS.md

> Compatible with the [agents.md](https://agents.md) standard. Specific rules in `.agents/rules/`.

## Project Overview

[One sentence. From package.json description or inferred.]

## Setup commands

- Install deps: `[pm] install`
- Start dev: `[pm] dev`
- Run tests: `[pm] test:run` (or `[pm] test` if no :run script)
- Lint: `[pm] lint`
[Add doctor/format if scripts exist]

## Source Files

[List source directories with one-line purpose per file. Read the actual files to describe what they do.]

```
src/
├── app/layout.tsx    # Root layout, metadata, fonts
├── app/page.tsx      # Home page
├── lib/utils.ts      # cn() helper
```

## Where to edit

| Task | Files |
|------|-------|
[5-8 rows mapping common tasks to exact file paths]

## Essential Commands

| Command | Purpose | When |
|---------|---------|------|
[Rows from package.json scripts. 3 columns: command, purpose, when to use it.]

## Verification Cycle

After every code change. Run in this order (lint FIRST, then typecheck, then build/test):
```
[lint cmd] → [typecheck cmd] → [build cmd] → [test cmd]
```
[Add doctor if react-doctor exists.]

## Before committing

- Run verification cycle. Fix failures before committing.
- If dependencies changed: run `[pm] install` and verify lockfile.

## Code Style

[2-4 project-specific rules. Only what's NOT obvious from reading the code.]
- TypeScript strict mode — zero `any` types.
- [Package manager] always. No [alternatives].
- [UI language rule if not English]

## Hard Rules

[2-3 numbered, non-negotiable rules.]
1. Never commit secrets, tokens, or credentials.
2. Every behavior change needs a test.
[If monorepo: 3. Keep contracts synchronized across packages.]

## Prohibitions

- **Never** create README or markdown docs unless explicitly asked.
- **Never** bump version numbers in feature PRs.
[If TypeScript: - **Never** use `any` types.]
[If build output: - **Never** edit generated files by hand.]

## Boundaries

**Ask first:**
- Large cross-package refactors.
- New dependencies with broad impact.

**Never:**
- Commit secrets, credentials, or tokens.
- Use destructive git operations unless explicitly requested.

## Global Conventions

[5-8 bullets from detected stack:]
- [PM] always. No [alternatives].
- TypeScript strict mode.
- [Validation approach: Zod schemas OR plain TypeScript types + guards]
- Import convention: `@/*` alias.
- [Component rules: server default, "use client" only with hooks]
- [Styling rules from CSS detection]
- Code in English. [UI language rule if different.]
- No `any` types.

## Definition of Done

A change is complete when ALL are true:
1. Typecheck, lint, and tests pass.
2. New code has tests covering the change.
3. Documentation updated if behavior changed.
4. No new warnings or errors.

## Gotchas

[If platform-specific issues: NTFS, Docker, sandbox, Windows]

## PR instructions

- Conventional commits: `feat:`, `fix:`, `refactor:`, `test:`, `chore:`.
- No "Co-Authored-By" or AI attribution in commits.
[If PR template exists: - Follow PULL_REQUEST_TEMPLATE.md.]
```

## Step 3 — Companion rule files (full mode)

**Generate these files in `.agents/rules/`.** Each file must contain project-specific content — never generic templates. Skip any file whose topic is not detected in the project.

Create `.agents/rules/` directory if it doesn't exist. Then generate:

| File | Generate when | Key sections |
|------|--------------|-------------|
| `architecture.md` | Always | Stack table (exact versions), routes list, ASCII data flow with real function names |
| `frontend-patterns.md` | Has React/Next.js/Vue | File structure, component rules, state locations, toast system, trust boundaries |
| `server-actions.md` | Has server actions or API routes | Entry points, flow steps, response types, error handling, rate limiting |
| `testing.md` | Has test runner | Runner info, commands table, test file tree, test count, patterns, specific test commands |
| `git-workflow.md` | Always | Approval rules, commit convention, pre-commit cycle, branches |
| `sdd-workflow.md` | Always | Preflight defaults, post-apply verification, spec locations |
| `styling.md` | Tailwind/CSS Modules/styled-components | Approach description, rules, theme tokens, animations |
| `forms.md` | react-hook-form/formik | Library description, form example, rules, validation, errors |
| `database.md` | Prisma/Drizzle/Knex | ORM name, schema location, migrations, conventions, query patterns |
| `i18n.md` | next-intl/react-i18next | Library, languages, file structure, usage example, rules |

## Step 4 — Multi-platform (if detected)

For each detected platform, generate the companion file. Keep under 200 lines. Include `> Full rules in @AGENTS.md`.

| Detection | File | Format |
|-----------|------|--------|
| `.cursorrules` or `.cursor/` | `.cursorrules` | Progressive disclosure, severity levels, BAD/GOOD examples |
| `.github/copilot-instructions.md` | `.github/copilot-instructions.md` | Flat instructions, short |
| `GEMINI.md` or `.gemini/` | `GEMINI.md` | System prompt style, second person |
| `.windsurfrules` or `.windsurf/` | `.windsurfrules` | Progressive disclosure |

## Step 5 — Claude (if detected)

If `.claude/` or `CLAUDE.md` exists, generate `CLAUDE.md`:

```markdown
# CLAUDE.md

See @AGENTS.md
```

Only add Claude-specific sections if `.claude/rules/`, `.claude/commands/`, or `.claude/agents/` exist.

## Step 6 — Validate and enforce

**Before declaring the task complete, the agent MUST:**

- Scan all generated files for `{{`, `TODO`, `add here`, `...` → fix any found.
- Every command in AGENTS.md must exist as a script key in package.json.
- If AGENTS.md > 300 lines, warn. If > 500, move content to rule files.
- Read package.json `scripts` and detect the CORRECT package manager from lockfiles. Never default to npm.
- Run `[format cmd]` on changed files if formatter exists.
- Update README if features, setup steps, or developer ergonomics changed.
- Summarize changes in conventional commit format.
- Resolve any open questions before declaring done.

## Step 7 — Report

Tell the user:
- What was detected (all categories with values)
- What was generated (files + line counts)
- What was skipped (with reason)
- Confidence score
- Backup location if any files were overwritten
