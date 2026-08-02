---
name: agents-generator
description: "Generate a complete, project-specific AGENTS.md compatible with the agents.md standard (60k+ open-source projects). Also generates platform-specific files (.cursorrules, Copilot instructions, GEMINI.md) for detected platforms. Use when setting up AI agent rules, creating project conventions for Claude Code, Cursor, Copilot, OpenCode, or any agent that reads AGENTS.md. Supports three modes: full (rich AGENTS.md + companion rule files + multi-platform output), minimal (single 30-line AGENTS.md following the agents.md standard), and update (diff existing, regenerate only changed). Detects monorepos for nested files. Extracts design tokens from config files. Includes confidence scoring, managed blocks, audit logging, and state memory. Every command and convention is derived from the project's real package.json, config files, and directory structure — never generic templates."
license: MIT
allowed-tools: Read Write Edit Bash(ls:*) Bash(git:*) Bash(tree:*) Bash(find:*) Grep Glob WebFetch
metadata:
  author: OJPalenzuela
  version: "1.0.0"
---

# Skill: agents-generator

Generates a tailored AGENTS.md + `.agents/rules/*.md` for the target project — not a template with placeholders, but a living document that matches the project's real toolchain.

## What you get

From a project that uses **Bun + Next.js 16 + Tailwind + Vitest + Server Actions**, the skill produces:

```
AGENTS.md
├── Comandos Esenciales: bun dev, bun run build, bun run test:run, bun doctor
├── Ciclo de Verificación:  bunx tsc --noEmit → bun run lint → bun run test:run → bun doctor
├── Convenciones: "Bun siempre. Sin librería de validación externa — TypeScript types + guards manuales."
└── Arquitectura → .agents/rules/architecture.md

.agents/rules/
├── architecture.md       ← ASCII diagram with real directories, exact versions from package.json
├── frontend-patterns.md  ← Component rules, state locations, toast system, analytics hook
├── server-actions.md     ← downloadVideo() flow, DownloadResult type, rate limiter 10req/60s
├── testing.md            ← "74 tests in 5 files", vitest commands, mock patterns
├── git-workflow.md       ← Conventional commits, "Sin Co-Authored-By ni atribución a IA"
└── sdd-workflow.md       ← Preflight defaults, post-apply verification cycle
```

Rules NOT generated: `backend.md` (no NestJS), `database.md` (no ORM), `i18n.md` (hardcoded Spanish, no i18n library), `forms.md` (manual inputs, no react-hook-form), `styling.md` (Tailwind already in frontend rules).

## Activation Contract

Generate AGENTS.md (+ `.agents/rules/*.md` in full mode) for the target project. Never guess — read the project's actual files first.

### Mode selection

Ask which mode, or infer from the user's phrasing. Three modes exist:

| User says                                                              | Mode        | Output                                                             |
| ---------------------------------------------------------------------- | ----------- | ------------------------------------------------------------------ |
| "simple AGENTS.md", "just the basics", "minimal", "agents.md standard" | **Minimal** | Single `AGENTS.md` (~30 lines, no rule files)                      |
| "full AGENTS.md", "with rules", "complete", or default                 | **Full**    | `AGENTS.md` + `.agents/rules/*.md` (project-specific rule files)   |
| "update AGENTS.md", "refresh", "my stack changed", "I added X"         | **Update**  | Diff existing files, regenerate only what changed, merge new rules |

If the project is a monorepo, **always** offer nested AGENTS.md files for each sub-package after generating the root file.

### Dry-run mode

If the user asks to "preview", "show what would change", "dry-run", or "what will be generated", run all detection steps but **do not write any files**. Instead, display:

- Detection summary (all 16 categories with detected values)
- List of files that would be created (full paths, relative to project root)
- List of files that would be updated (with a summary of changes)
- List of rules skipped (with reason)
- Sample of the first 10 lines of the generated AGENTS.md

### Fallback when detection fails

If critical files are missing (no `package.json`, no config files, empty directory):

| Missing                                       | Action                                                                                                                                                          |
| --------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| No `package.json`                             | Generate only AGENTS.md with a `## Setup commands` section asking the user to fill it in. Note "No package.json found — please add your project's commands."    |
| No TypeScript config                          | Skip `tsc --noEmit` from verification cycle. Skip no-any rule.                                                                                                  |
| No test runner                                | Skip `testing.md`. Remove test from verification cycle.                                                                                                         |
| No framework detected                         | Generate generic AGENTS.md. Ask user to confirm the framework.                                                                                                  |
| Empty directory (no source files)             | Generate minimal AGENTS.md with placeholder sections. Tell user "This project appears empty. The AGENTS.md has placeholders — fill them in once code is added." |
| Detection returns "unknown" for 3+ categories | Flag to user: "I could only detect X of 16 categories. The AGENTS.md may be incomplete. Review manually."                                                       |

## Hard Rules

- **Read before writing.** Read `package.json`, all config files, and directory structure before generating anything.
- **Apply `references/decision-matrix.md`.** 16 detection steps. Run all, skip absent signals.
- **Generate only what applies.** No backend rules for frontend-only. No database rules without ORM. No i18n without i18n library.
- **Validate commands.** Every command in output must exist as a script key in `package.json`. Missing scripts → substitute with raw invocation or skip.
- **No placeholders.** Post-generation scan: reject output containing `{{`, `TODO`, `add here`, or `...`.
- **Backup first.** If AGENTS.md or `.agents/rules/*.md` exist, copy to `.agents/backups/` with timestamp. Report to user.
- **Calibrate quality.** Read `references/example-output/README.md` before generating. Match that specificity.

## Execution Steps

### Common (all modes)

1. `git rev-parse --show-toplevel` → project root.
2. Read `package.json` (scripts, deps, workspaces). Save scripts for validation.
3. Read config: `tsconfig.json`, framework config, test config, CSS config, lint config, ORM config, i18n config, monorepo config.
4. **Extract design tokens**: if `tailwind.config.*`, `globals.css`, `theme.ts/tokens.ts` exist, extract colors, fonts, breakpoints, spacing. These go into the AGENTS.md as concrete values, not generic references.
5. Explore directory structure: router type, CSS approach, server actions, route files. Also detect platform files: `.cursorrules`, `.github/copilot-instructions.md`, `GEMINI.md`, `.windsurfrules`.
6. **JIT skill retrieval**: scan `.agents/skills/`, `.claude/skills/`, `.cursor/skills/` for installed skills. Match their descriptions against the project's detected stack. Integrate relevant skill references into AGENTS.md (2-5 max, pointer-based, not full content).
7. **Select mode**: ask if ambiguous, or infer from user's phrasing (see Activation Contract).

### Dry-run (if requested)

6. Run detection (steps 2-4 above) and decision matrix.
7. Display: detection summary, files that would be created/updated, skipped rules, sample output.
8. **STOP** — do not write any files. Wait for user confirmation to proceed with write mode.

### Full mode

8. **Calibrate**: read `references/example-output/README.md`.
9. **Backup**: timestamped copies to `.agents/backups/` if overwriting.
10. Run decision matrix → list of rule files to generate.
11. **Confidence scoring**: assign 0-100 score based on detection success rate (each of 16 categories = ~6 points). If score < 80, flag uncertain categories and ask developer to confirm. If score < 50, warn that the AGENTS.md may be incomplete.
12. **Confirm with developer** (interactive/update mode only): show detected stack, confidence score, and which rule files will be generated. Ask: "Does this look right?" Wait for confirmation.
13. **Validate**: cross-check all planned commands against `package.json` scripts.
14. Generate `AGENTS.md` from `assets/agents-template.md` — wrap all generated sections in `<!-- AGENTS-GENERATED-START -->` / `<!-- AGENTS-GENERATED-END -->` markers to preserve human edits on update. Generate each rule file from its template with same managed block wrappers.
15. If Claude Code detected: generate thin `CLAUDE.md` from `assets/claude-template.md`.
16. **Multi-platform**: for each platform detected in step 5, generate platform-specific file from `assets/platform-template.md` (.cursorrules, Copilot instructions, GEMINI.md, .windsurfrules).
17. Generate per-package nested files if monorepo.
18. **Global baseline**: if `.agents/global-baseline.md` doesn't exist, generate it with cross-project defaults (instruction scope, coding behavior baseline, done-when checklist).

### Update mode

6. **Backup**: timestamped copies of ALL existing files to `.agents/backups/`.
7. Read existing `AGENTS.md` and `.agents/rules/*.md`.
8. Re-run full detection (steps 2-4) to get the new state of the project.
9. **Diff**: compare old detections vs new. Identify:
   - New categories (e.g., project added Zod since last run) → generate new rule files
   - Removed categories (e.g., project removed Prisma) → flag for removal, ask user
   - Changed values (e.g., test runner switched from jest to vitest) → update affected sections
   - Unchanged categories → leave existing files as-is
10. Show the user a summary of what changed and what will be updated.
11. Apply changes — merge new sections into existing AGENTS.md, add new rule files, update changed sections.
12. **Do not** regenerate files for unchanged categories.

### Minimal mode

6. **Backup**: timestamped copy of existing `AGENTS.md` if present.
7. Generate single `AGENTS.md` from `assets/agents-minimal-template.md` — no rule files, under 30 lines.

### Nested (monorepo, all modes)

- If monorepo detected: explore each `packages/*/` and `apps/*/`. For each sub-directory with a `package.json`:
  - Read the sub-package's `package.json` for scripts, name, description.
  - Generate `AGENTS.md` from `assets/agents-nested-template.md`.
  - For full/update mode: also generate/update applicable rule files at sub-package root.
  - Report which sub-packages received nested files.

### Post-generation (all write modes)

- **Scan**: search output for `{{`, `TODO`, `add here`, `...`. Fix any found.
- **Line limit**: if AGENTS.md > 300 lines, warn. If > 500 lines, move content to `.agents/rules/` or use `@import` for existing docs.
- **Validate commands** (full/update): cross-check against package.json scripts.
- **Fallback check**: if 3+ detection categories returned "unknown", flag to user.
- **Audit logging**: write `.agent/logs/log_{timestamp}_{session}.json` with confidence score, detection summary, files generated, matched skills, and verification status.
- **State memory**: write `.agent/memory/project_state.md` with current phase, detected profile, recent skills, and last generated files. This is the System Prompt injected into future sessions.
- **Session lifecycle**: write `docs/handoff/HANDOFF.md` with session summary for the next agent session. If resuming, read the latest handoff first.
- Save summary to Engram: `{project}/agents-config`.

## Output Contract

Return:

- Mode used (full / minimal) and why
- Files created or modified (with diff if overwriting), including nested files if monorepo
- Detection summary (all 16 categories)
- Which rule files were generated and which were skipped (with reason)
- Any edge cases encountered (multiple frameworks, hybrid router, no config, etc.)

## References

| Priority                    | File                                                  | Purpose                                                                                            |
| --------------------------- | ----------------------------------------------------- | -------------------------------------------------------------------------------------------------- |
| **Read first**              | `references/example-output/README.md`                 | Quality benchmark — real full-mode output for a Next.js/Bun project                                |
| **Read second**             | `references/decision-matrix.md`                       | Full 16-step detection logic, rule selection, edge cases                                           |
| **Apply during generation** | `references/template-filling-guide.md`                | How to fill each placeholder with real project data                                                |
| Full mode                   | `assets/agents-template.md`                           | Rich AGENTS.md template — commands, verification cycle, conventions, PR instructions, rule files   |
| Minimal mode                | `assets/agents-minimal-template.md`                   | Single AGENTS.md (~30 lines) following agents.md standard — setup, code style, testing, PR         |
| Nested (monorepo)           | `assets/agents-nested-template.md`                    | Per-package AGENTS.md — lightweight, package-specific commands                                     |
| Templates                   | `assets/*.md` (10 rule templates + 3 agent templates) | Architecture, frontend, backend, server actions, styling, forms, database, i18n, testing, git, SDD |
| Claude Code                 | `assets/claude-template.md`                           | Thin CLAUDE.md — only generated if Claude detected                                                 |
