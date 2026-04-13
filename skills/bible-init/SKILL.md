---
name: init-bible
description: Bootstrap a new Bible for this project — scans codebase, detects tech stack, generates comprehensive documentation, and creates .bible.yaml config. Use when starting TheBible in a new project.
---

# Bible Init

Bootstrap a comprehensive single-source-of-truth document (the "Bible") for any codebase. This skill scans the project, detects its tech stack and architecture, generates substantive documentation for each relevant section, and writes a `.bible.yaml` config file.

The output is a Bible that enables any developer to understand the entire system without asking questions.

---

## Phase 1: Pre-flight Check

Before doing anything, verify the project state.

1. **Check for existing config.** Look for `.bible.yaml` in the project root.
   - If found, warn the user: "A Bible config already exists. Reinitialize? (This won't overwrite the Bible file itself unless you confirm.)"
   - If the user declines, stop.

2. **Check for existing Bible files.** Search for `*BIBLE*.md` and `*bible*.md` in root, `docs/`, `Docs/`, `documentation/`.
   - If found, report the path and ask: "Found existing Bible at [path]. Use this path or generate a new one?"

3. **Read bootstrapping files** if they exist:
   - `README.md` -- extract project name, description, purpose, links
   - `CLAUDE.md` -- extract operational patterns, architecture notes, gotchas
   - Any `CONTRIBUTING.md`, `ARCHITECTURE.md`, or similar

4. **Determine project name.** Priority order:
   - `name` field from `package.json`, `Cargo.toml`, `pyproject.toml`, `go.mod`, etc.
   - Git remote repository name (from `git remote get-url origin`)
   - Directory name of the project root

---

## Phase 2: Codebase Scan

Gather the raw material needed to write a substantive Bible.

1. **Detect tech stack.** Read `references/tech-detectors.md` and check for each signature file. Record ALL detected stacks (projects are often polyglot).

2. **Recent activity.** Run:
   ```bash
   git log --oneline -20
   ```
   Note the development pace and recent focus areas.

3. **Directory structure.** Scan the top 2 levels, excluding standard ignores (`.git`, `node_modules`, `.build`, `target`, `__pycache__`, `venv`, `.venv`, `dist`, `build`, `.next`, `.nuxt`, `Pods`, `DerivedData`):
   ```bash
   find . -maxdepth 2 -type d ! -path '*/\.*' ! -path '*/node_modules/*' ! -path '*/target/*' ! -path '*/__pycache__/*' ! -path '*/venv/*' ! -path '*/.venv/*' ! -path '*/dist/*' ! -path '*/build/*' ! -path '*/Pods/*' ! -path '*/DerivedData/*' | sort
   ```

4. **Artifact counts.** Count source files by extension, test files, config files, and migration files. Use `find` with `-name` patterns. This gives a sense of project scale.

5. **Build commands.** Check for:
   - `package.json` `scripts` object
   - `Makefile` / `Justfile` / `Taskfile.yml` targets
   - `Dockerfile` / `docker-compose*` services
   - CI config (`.github/workflows/`, `.gitlab-ci.yml`, `Jenkinsfile`)

6. **Git remote URL.**
   ```bash
   git remote get-url origin 2>/dev/null || echo "No remote configured"
   ```

---

## Phase 3: Generate `.bible.yaml`

Create the config file that drives all other Bible operations.

1. **Load the schema.** Read `references/bible-yaml-schema.md` from the plugin's shared references for the 14 default sections and their structure.

2. **Map sections to detected paths.** For each of the 14 default sections:
   - Match detected directories against the section's default path patterns
   - If relevant code exists, set `active: true` and populate `paths` with actual directories
   - If no relevant code exists, set `active: false`

3. **Adjust for project structure.** Examples:
   - Project uses `src/api/` instead of `api/` -- update the API section paths
   - Project has `supabase/functions/` -- activate the Backend/API section with those paths
   - Monorepo with `packages/` -- map each package to relevant sections

4. **Write `.bible.yaml`** to the project root. Include:
   - `bible_path` (default: `docs/BIBLE.md`)
   - `project_name` (detected in Phase 1)
   - `version` (start at `1.0`)
   - All 14 sections with their `active` status and `paths`

---

## Phase 4: Generate Bible

This is the core phase. Produce a document with real, substantive content -- not placeholder templates.

1. **Load generation guidance.** Read:
   - `references/section-templates.md` -- per-section exploration and documentation prompts
   - `references/default-template.md` (plugin shared references) -- the Bible's structural template

2. **Explore and document each active section.** For each active section in `.bible.yaml`:
   - Launch Explore agents (up to 3 in parallel) to read the relevant source files identified by the section's `paths`
   - Follow the section-specific prompts from `references/section-templates.md`
   - Generate substantive content:
     - **Tables** with actual file names, function signatures, endpoint paths, column names
     - **ASCII architecture diagrams** derived from real directory structure
     - **Real build commands** from detected package.json scripts or Makefile targets
     - **Actual environment variables** from `.env.example`, `docker-compose.yml`, or code-level `process.env` / `os.environ` scans
     - **Dependency lists** with versions from lockfiles
     - **Configuration values** from real config files

3. **Write the Bible file** to the path specified in `.bible.yaml`.
   - Header format:
     ```markdown
     # [Project Name] Bible

     > **Version:** 1.0 · **Date:** [today's date]

     Single source of truth for [project name]. This document covers architecture, conventions, operations, and every integration point a developer needs to understand the system.
     ```
   - Each section gets an H2 heading (`## 1. Project Overview`, etc.)
   - Only include active sections
   - Section order follows the numbering from the schema

4. **Quality standards for generated content:**
   - Every table must contain real data, not example placeholders
   - Every code block must be a real command or real code from the project
   - Architecture descriptions must name actual directories and files
   - If information cannot be determined from the codebase, say so explicitly rather than guessing
   - No emojis. Technical, direct prose.

---

## Phase 5: Commit

Stage and commit both files.

```bash
git add [bible_path] .bible.yaml
git commit -m "docs: initialize project Bible v1.0

Co-Authored-By: Claude <noreply@anthropic.com>"
```

If the commit fails (e.g., pre-commit hook), fix the issue and retry with a new commit -- do not amend.

---

## Phase 6: CLAUDE.md Sync (Optional)

After the Bible is generated, offer to sync operational content to `CLAUDE.md`.

Prompt the user: "Would you like to sync operational Bible content to CLAUDE.md now?"

If yes, invoke the `claude-md-management:claude-md-improver` skill with context about:
- Which sections were generated
- Which sections are marked as `operational: true` in `.bible.yaml`
- The Bible file path for reference

This ensures `CLAUDE.md` stays lean (build commands, gotchas, patterns) while the Bible holds the full picture.

---

## Rules

- **Never hardcode project-specific paths.** Always derive paths from the codebase scan.
- **Content must be substantive.** Real file names, real commands, real configuration. If the scan finds 48 API endpoints, list them. If there are 12 database tables, document them.
- **Match existing style.** If the project already has documentation, match its tone and format conventions.
- **No placeholders.** Every `[TODO]` or `[fill in]` is a failure. If something cannot be determined, state that explicitly: "Not detected -- add manually."
- **Respect .gitignore.** Never read or document contents of files that are gitignored (secrets, local configs, credentials).
- **Idempotent on config.** Running init again with confirmation should regenerate the Bible without breaking the config structure.
- **Section activation is conservative.** Only mark a section active if there is real code or config to document. An empty section is worse than no section.
