# Section Templates

Per-section generation prompts for the 14 default Bible sections. Each entry tells an Explore agent what to look for and how to document it.

These prompts assume the section is active (relevant code exists). Skip inactive sections entirely.

---

## Section 1: Project Overview
**Explore:** `README.md`, `CLAUDE.md`, primary manifest (`package.json`, `Cargo.toml`, `go.mod`, etc.), git remote URL, LICENSE file
**Document:** Project name, one-paragraph purpose statement, target users/audience, repository URL, primary language(s), license, current status (active development, maintenance, pre-release)
**Format:** Opening narrative paragraph, then a metadata table with key-value pairs
**Note:** This section sets the tone. Be specific about what the project does, not just what category it falls in.

## Section 2: Architecture
**Explore:** Top-level directory structure, entry points (`main.*`, `index.*`, `app.*`), any existing architecture docs
**Document:** ASCII directory tree (top 2-3 levels with annotations), high-level data flow description, service boundaries, client-server split if applicable
**Format:** Annotated directory tree in a code block, followed by prose explaining the flow. If multiple services exist, list each with its purpose.
**Note:** Name real directories and files. Never use generic placeholders like "source code goes here."

## Section 3: Tech Stack
**Explore:** All detected signature files from tech-detectors, lockfiles for dependency versions, runtime version files (`.node-version`, `.python-version`, `rust-toolchain.toml`)
**Document:** Table of all technologies with category, version, and purpose. Group by: Language, Framework, Database, Infrastructure, Dev Tools.
**Format:** Single table. Include actual version numbers from lockfiles where available.
**Note:** Include build tools, linters, formatters, and CI systems. These are part of the stack.

## Section 4: Setup and Build
**Explore:** `package.json` scripts, `Makefile`/`Justfile`/`Taskfile`, `Dockerfile`, CI configs, any `CONTRIBUTING.md` or setup docs, `.env.example`
**Document:** Prerequisites (runtime versions, system dependencies), installation steps, all build/run/test commands, environment variable table (names and descriptions, never values)
**Format:** Step-by-step numbered list for setup, then a commands table with `command | description` columns, then an env var table
**Note:** Commands must be real and runnable. Copy them from the actual config files.

## Section 5: Data Model
**Explore:** Database schemas (`schema.prisma`, migration files, SQL files, ORM models), type definitions for core domain objects, any ERD docs
**Document:** Core entities/tables with their columns and relationships, enum definitions, key constraints and indexes
**Format:** One table per entity showing column name, type, and notes. Describe relationships in prose after the tables.
**Note:** Focus on domain-critical tables. If there are 50+ tables, group by domain and cover the most important ones in detail.

## Section 6: API Reference
**Explore:** Route definitions, OpenAPI/Swagger specs, API handler files, middleware, serverless function directories
**Document:** All endpoints with method, path, auth requirement, and brief description. Group by domain or resource.
**Format:** Table with `Method | Path | Auth | Description` columns. If the project has an OpenAPI spec, reference its location.
**Note:** Include internal APIs (service-to-service), webhooks, and WebSocket endpoints if they exist.

## Section 7: Authentication and Authorization
**Explore:** Auth middleware, auth service files, JWT/session configuration, RBAC/permission definitions, OAuth configs, auth-related environment variables
**Document:** Auth method (JWT, session, API key, OAuth), token lifecycle, permission model (RBAC, ABAC, etc.), protected vs public routes
**Format:** Prose description of the auth flow, then tables for roles/permissions if RBAC exists
**Note:** Document the actual flow a request takes from unauthenticated to authorized. Include token expiry times and refresh mechanisms.

## Section 8: Services and Integrations
**Explore:** Third-party SDK imports, API client files, webhook handlers, configuration for external services (Stripe, SendGrid, S3, etc.)
**Document:** Every external service with its purpose, SDK/client used, relevant env vars, and any rate limits or quotas
**Format:** Table with `Service | Purpose | SDK/Config | Env Vars` columns
**Note:** Include cloud services (hosting, CDN, monitoring), payment providers, email services, analytics, and any SaaS dependencies.

## Section 9: Testing
**Explore:** Test directories, test config files (`jest.config.*`, `vitest.config.*`, `pytest.ini`, `.rspec`), CI test steps, test utilities/fixtures
**Document:** Test framework(s), test directory structure, how to run tests (unit, integration, e2e), coverage requirements if configured, test data/fixture patterns
**Format:** Commands table for running different test suites, then prose on testing patterns and conventions
**Note:** If no tests exist, say so explicitly. Document the testing gap rather than leaving the section empty.

## Section 10: Deployment
**Explore:** CI/CD configs (`.github/workflows/`, `.gitlab-ci.yml`, `Jenkinsfile`), deploy scripts, hosting configs (`vercel.json`, `firebase.json`, `fly.toml`), Dockerfiles
**Document:** Deployment targets (production, staging URLs if visible), deploy commands, CI/CD pipeline stages, infrastructure requirements
**Format:** Pipeline stages as a numbered list or flow, then a table of deployment targets with their URLs and deploy commands
**Note:** Include both automated (CI/CD) and manual deploy processes. Document any required secrets or deploy keys (names only, never values).

## Section 11: Configuration and Environment
**Explore:** `.env.example`, config files, feature flag systems, environment-specific configs, secrets management approach
**Document:** All configuration sources (env vars, config files, feature flags, secrets vaults), with a master table of every config key, its source, and description
**Format:** Table with `Key | Source | Required | Description` columns. Group by category (database, auth, third-party, feature flags).
**Note:** Never include actual secret values. If `.env.example` exists, use it as the source. Otherwise scan code for `process.env`, `os.environ`, `env::var`, etc.

## Section 12: Conventions and Patterns
**Explore:** Linter configs (`.eslintrc`, `biome.json`, `.swiftlint.yml`), formatter configs (`.prettierrc`), existing code for naming patterns, state management patterns, error handling patterns
**Document:** File naming conventions, code organization patterns, state management approach, error handling strategy, logging approach, import ordering conventions
**Format:** Rules list with short explanations. Include concrete examples from the actual codebase where possible.
**Note:** Derive conventions from the code itself, not just config files. If the project uses consistent patterns (e.g., all services are singletons, all errors extend a base class), document them.

## Section 13: Known Issues and Gotchas
**Explore:** `CLAUDE.md` gotchas sections, TODO/FIXME/HACK comments in code, GitHub issues if accessible, any `KNOWN_ISSUES` docs
**Document:** Operational gotchas (things that break silently), workarounds currently in place, technical debt items, platform-specific quirks
**Format:** Bulleted list. Each item should be a single clear statement of what the gotcha is and why it matters.
**Note:** This section prevents future developers from hitting the same traps. Prioritize items that cause silent failures over items that cause loud errors.

## Section 14: Operational Runbooks
**Explore:** Scripts directory, maintenance docs, monitoring/alerting configs, database migration procedures, incident response docs
**Document:** Common operational tasks (deploy, rollback, database migration, cache clear, log access), monitoring and alerting setup, incident response steps if documented
**Format:** Each runbook as a named subsection with step-by-step commands
**Note:** If the project has no operational procedures documented, create basic ones from the deploy and build information gathered in other sections. At minimum: how to deploy, how to rollback, how to check logs.
