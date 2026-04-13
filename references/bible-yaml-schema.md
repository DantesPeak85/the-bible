# `.bible.yaml` Configuration Schema

The `.bible.yaml` file lives in the project root and configures how TheBible plugin manages your project's Bible document. All three skills (`/init-bible`, `/update-bible`, `/bible-status`) read this file.

---

## Top-Level Fields

### `bible_path`
- **Type:** string
- **Required:** yes
- **Default:** `docs/BIBLE.md`
- **Description:** Relative path (from project root) to the Bible file. The plugin reads from and writes to this location.
- **Example:** `docs/BIBLE.md`

### `version`
- **Type:** string
- **Required:** yes
- **Default:** `1.0` (set by `/init-bible`)
- **Description:** Current Bible version in `MAJOR.MINOR` format. MAJOR increments on structural changes (new sections, removed sections, reorganization). MINOR increments on content updates within existing sections. The `/update-bible` skill increments this automatically.
- **Example:** `3.2`

### `last_updated`
- **Type:** string (ISO 8601 date)
- **Required:** yes
- **Default:** set to today's date by `/init-bible`
- **Description:** Date of the last Bible update. Used by `/bible-status` for `git log --since` and staleness calculations. The `/update-bible` skill sets this automatically after each update.
- **Example:** `2026-04-13`

---

## `project` Block

Project metadata used in the Bible header and for context during generation.

### `project.name`
- **Type:** string
- **Required:** yes
- **Description:** Project name. Used in the Bible document title (e.g., `# {name} Bible`).
- **Example:** `Acme API`

### `project.description`
- **Type:** string
- **Required:** no
- **Default:** (none)
- **Description:** One-line project description. Included in the Bible's Section 1 (Project Overview) if provided.
- **Example:** `REST API for the Acme e-commerce platform`

---

## `claude_md` Block

Controls whether and how CLAUDE.md is kept in sync with the Bible.

### `claude_md.enabled`
- **Type:** boolean
- **Required:** no
- **Default:** `true`
- **Description:** When `true`, `/update-bible` will regenerate the operational sections of CLAUDE.md after updating the Bible. When `false`, CLAUDE.md is left untouched.
- **Example:** `true`

### `claude_md.path`
- **Type:** string
- **Required:** no
- **Default:** `CLAUDE.md`
- **Description:** Relative path to the project's CLAUDE.md file. Only used when `enabled` is `true`.
- **Example:** `CLAUDE.md`

### `claude_md.operational_sections`
- **Type:** string[]
- **Required:** no
- **Default:** `["Project Overview", "Architecture Overview", "Build & Deploy", "Key Patterns & Conventions", "Gotchas & Pitfalls"]`
- **Description:** Which Bible section names contain operational content that should flow into CLAUDE.md. Section names must exactly match the `sections[].name` values. Only these sections are extracted and condensed into CLAUDE.md — the rest stay Bible-only.
- **Example:**
  ```yaml
  operational_sections:
    - Project Overview
    - Architecture Overview
    - Build & Deploy
    - Key Patterns & Conventions
    - Gotchas & Pitfalls
  ```

---

## `sections` Array

Defines the Bible's structure and maps file paths to sections. Each entry represents one top-level section of the Bible document.

### `sections[].number`
- **Type:** integer
- **Required:** yes
- **Description:** Section number, used for ordering in the Bible document and in status reports. Must be unique across all sections.
- **Example:** `1`

### `sections[].name`
- **Type:** string
- **Required:** yes
- **Description:** Section title. This must exactly match the corresponding heading in the Bible file (e.g., `## 3. Database Schema` has name `Database Schema`). Used by `/bible-status` to report per-section staleness and by `/update-bible` to target specific sections.
- **Example:** `API Endpoints`

### `sections[].description`
- **Type:** string
- **Required:** no
- **Default:** (none)
- **Description:** Brief description of what this section covers. Used by `/init-bible` as guidance when generating initial content and by `/update-bible` to understand section scope.
- **Example:** `All REST endpoints, request/response shapes, auth requirements`

### `sections[].paths`
- **Type:** string[]
- **Required:** yes
- **Description:** Glob patterns that map file changes to this section. When a commit touches files matching these patterns, the section is flagged as potentially stale. Use standard glob syntax (`*`, `**`, `?`). An empty array `[]` means the section is only updated explicitly or when referenced by other sections (useful for overview or philosophy sections that don't map to specific files).
- **Example:**
  ```yaml
  paths:
    - "src/routes/**"
    - "src/middleware/**"
    - "openapi/*.yaml"
  ```

### `sections[].active`
- **Type:** boolean
- **Required:** no
- **Default:** `true`
- **Description:** Set to `false` to exclude this section during `/init-bible` generation and `/update-bible` triage. The section heading remains in the Bible but is marked as inactive. Useful for planned-but-not-yet-relevant sections.
- **Example:** `false`

---

## `impact_rules` Block

Defines how `/update-bible` triage classifies the impact of changes. Each key contains an array of human-readable descriptions that guide the triage assessment.

### `impact_rules.HIGH`
- **Type:** string[]
- **Required:** no
- **Default:** (see below)
- **Description:** Descriptions of changes that constitute HIGH impact — the Bible section is likely wrong or dangerously incomplete without an update.
- **Default values:**
  ```yaml
  HIGH:
    - "New API endpoints, database tables, or services added"
    - "Authentication or authorization changes"
    - "Breaking changes to interfaces or contracts"
    - "Infrastructure or deployment pipeline changes"
    - "Security-critical changes"
  ```

### `impact_rules.MEDIUM`
- **Type:** string[]
- **Required:** no
- **Default:** (see below)
- **Description:** Descriptions of changes that constitute MEDIUM impact — the Bible section is outdated but not dangerously wrong.
- **Default values:**
  ```yaml
  MEDIUM:
    - "New features within existing modules"
    - "Configuration changes"
    - "Dependency version updates"
    - "New environment variables or build flags"
  ```

### `impact_rules.LOW`
- **Type:** string[]
- **Required:** no
- **Default:** (see below)
- **Description:** Descriptions of changes that constitute LOW impact — minor or cosmetic; update at next convenience.
- **Default values:**
  ```yaml
  LOW:
    - "Bug fixes that don't change behavior contracts"
    - "Code refactoring without interface changes"
    - "Test additions or updates"
    - "Comment or documentation updates within code"
  ```

### `impact_rules.NONE`
- **Type:** string[]
- **Required:** no
- **Default:** (see below)
- **Description:** Changes that never affect the Bible and can be safely ignored during triage.
- **Default values:**
  ```yaml
  NONE:
    - "Whitespace or formatting-only changes"
    - "Changes to files in ignore_paths"
    - "CI/CD pipeline tweaks that don't change behavior"
    - "IDE configuration files"
  ```

---

## `ignore_paths` Array

- **Type:** string[]
- **Required:** no
- **Default:** `[]`
- **Description:** Glob patterns for files that should always be excluded from triage and staleness analysis. These files are invisible to both `/bible-status` and `/update-bible`. Common candidates: test fixtures, generated files, lock files, IDE configs.
- **Example:**
  ```yaml
  ignore_paths:
    - "**/*.lock"
    - "**/*.generated.*"
    - ".vscode/**"
    - ".idea/**"
    - "coverage/**"
    - "dist/**"
    - "node_modules/**"
  ```

---

## Full Example

A complete `.bible.yaml` for a hypothetical Node.js/React project:

```yaml
bible_path: docs/BIBLE.md
version: "2.1"
last_updated: "2026-04-13"

project:
  name: Taskflow
  description: Project management API with React dashboard

claude_md:
  enabled: true
  path: CLAUDE.md
  operational_sections:
    - Project Overview
    - Architecture Overview
    - Build & Deploy
    - Key Patterns & Conventions
    - Gotchas & Pitfalls

sections:
  - number: 1
    name: Project Overview
    description: Mission, status, team, and high-level decisions
    paths: []

  - number: 2
    name: Architecture Overview
    description: System diagram, service boundaries, data flow
    paths:
      - "src/app.*"
      - "src/server.*"
      - "docker-compose*.yml"
      - "Dockerfile*"

  - number: 3
    name: Tech Stack
    description: Languages, frameworks, versions, and rationale
    paths:
      - "package.json"
      - "tsconfig*.json"
      - "requirements*.txt"
      - "Cargo.toml"

  - number: 4
    name: Database Schema
    description: Tables, relationships, migrations, enums
    paths:
      - "migrations/**"
      - "prisma/**"
      - "src/models/**"
      - "src/db/**"

  - number: 5
    name: API Endpoints
    description: All REST/GraphQL endpoints, auth, request/response shapes
    paths:
      - "src/routes/**"
      - "src/controllers/**"
      - "src/middleware/auth*"
      - "openapi/**"

  - number: 6
    name: Authentication & Authorization
    description: Auth flows, token handling, RBAC, session management
    paths:
      - "src/auth/**"
      - "src/middleware/auth*"
      - "src/middleware/rbac*"

  - number: 7
    name: Frontend Architecture
    description: Component tree, routing, state management, design system
    paths:
      - "client/src/**"
      - "client/package.json"

  - number: 8
    name: Build & Deploy
    description: Build commands, CI/CD, environments, deploy procedures
    paths:
      - ".github/workflows/**"
      - "Dockerfile*"
      - "docker-compose*.yml"
      - "scripts/deploy*"
      - "Makefile"

  - number: 9
    name: Configuration & Environment
    description: Env vars, config files, secrets management
    paths:
      - ".env.example"
      - "src/config/**"
      - "src/lib/config*"

  - number: 10
    name: Key Patterns & Conventions
    description: Code patterns, naming conventions, architectural decisions
    paths:
      - "src/**"

  - number: 11
    name: Third-Party Integrations
    description: External APIs, webhooks, OAuth providers
    paths:
      - "src/integrations/**"
      - "src/webhooks/**"
      - "src/lib/external*"

  - number: 12
    name: Testing Strategy
    description: Test structure, fixtures, coverage requirements
    paths:
      - "tests/**"
      - "**/*.test.*"
      - "**/*.spec.*"
      - "jest.config*"
      - "vitest.config*"

  - number: 13
    name: Gotchas & Pitfalls
    description: Non-obvious behaviors, known issues, things that break silently
    paths: []

  - number: 14
    name: Roadmap & Status
    description: Current phase, upcoming work, tech debt
    paths: []
    active: false

impact_rules:
  HIGH:
    - "New API endpoints, database tables, or services added"
    - "Authentication or authorization changes"
    - "Breaking changes to interfaces or contracts"
    - "Infrastructure or deployment pipeline changes"
    - "Security-critical changes"
  MEDIUM:
    - "New features within existing modules"
    - "Configuration or environment variable changes"
    - "Dependency version updates with behavioral impact"
    - "New build flags or commands"
  LOW:
    - "Bug fixes that don't change behavior contracts"
    - "Code refactoring without interface changes"
    - "Test additions or updates"
    - "Comment or documentation updates within code"
  NONE:
    - "Whitespace or formatting-only changes"
    - "Changes to files in ignore_paths"
    - "CI/CD tweaks that don't change behavior"
    - "IDE or editor configuration files"

ignore_paths:
  - "**/*.lock"
  - "node_modules/**"
  - "dist/**"
  - "coverage/**"
  - ".vscode/**"
  - ".idea/**"
  - "**/*.generated.*"
```

---

## Validation Rules

When reading `.bible.yaml`, skills should validate:

1. `bible_path` must be a non-empty string pointing to a file with a `.md` extension
2. `version` must match the pattern `\d+\.\d+`
3. `last_updated` must be a valid ISO 8601 date (YYYY-MM-DD)
4. `project.name` must be a non-empty string
5. Every `sections[]` entry must have a unique `number`
6. Every `sections[]` entry must have a non-empty `name`
7. Every `sections[]` entry must have a `paths` array (can be empty `[]`)
8. `claude_md.operational_sections` entries must each match at least one `sections[].name`
9. Glob patterns in `paths` and `ignore_paths` must be valid glob syntax

If validation fails, the skill should report the specific error and stop rather than proceeding with a malformed config.
