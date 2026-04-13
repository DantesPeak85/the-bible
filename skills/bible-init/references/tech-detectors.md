# Tech Stack Detectors

Reference for detecting project tech stacks from filesystem signatures. Used by the `bible-init` skill during Phase 2 codebase scan.

---

## Detection Table

Check for these files/directories at the project root (or one level deep for monorepos). If a signature file exists, the stack is present. Multiple stacks can coexist.

| Signature File(s) | Stack | Category | Default Path Patterns |
|---|---|---|---|
| `package.json` | Node.js / JavaScript | Runtime | `src/`, `lib/`, `dist/` |
| `package.json` + `tsconfig.json` | TypeScript | Language | `src/`, `lib/`, `types/` |
| `package.json` + `next.config.*` | Next.js | Framework | `app/`, `pages/`, `api/`, `middleware.*`, `public/` |
| `package.json` + `nuxt.config.*` | Nuxt | Framework | `pages/`, `server/`, `composables/`, `components/` |
| `package.json` + `vite.config.*` | Vite (React/Vue/Svelte) | Build Tool | `src/`, `public/`, `index.html` |
| `package.json` + `remix.config.*` or `app/root.tsx` | Remix | Framework | `app/`, `app/routes/` |
| `package.json` + `astro.config.*` | Astro | Framework | `src/pages/`, `src/layouts/`, `src/components/` |
| `package.json` + `svelte.config.*` | SvelteKit | Framework | `src/routes/`, `src/lib/` |
| `package.json` + `angular.json` | Angular | Framework | `src/app/`, `src/environments/` |
| `package.json` + `expo` in dependencies | Expo / React Native | Mobile | `app/`, `src/`, `assets/` |
| `Cargo.toml` | Rust | Language | `src/`, `crates/`, `tests/`, `benches/` |
| `go.mod` | Go | Language | `cmd/`, `internal/`, `pkg/`, `api/` |
| `pyproject.toml` or `setup.py` or `setup.cfg` | Python | Language | `src/`, `app/`, `tests/`, `scripts/` |
| `requirements.txt` or `Pipfile` or `poetry.lock` | Python (deps) | Deps | (same as above) |
| `manage.py` + `settings.py` (in any subdir) | Django | Framework | `apps/`, `templates/`, `static/`, `manage.py` |
| `*.xcodeproj` or `*.xcworkspace` | Swift / iOS / macOS | Platform | find `.xcodeproj` location, scan sibling dirs |
| `Package.swift` | Swift Package | Language | `Sources/`, `Tests/` |
| `pubspec.yaml` | Dart / Flutter | Mobile | `lib/`, `test/`, `ios/`, `android/`, `web/` |
| `Gemfile` | Ruby | Language | `app/`, `lib/`, `spec/`, `config/`, `db/` |
| `Gemfile` + `config/routes.rb` | Ruby on Rails | Framework | `app/models/`, `app/controllers/`, `app/views/`, `db/migrate/` |
| `*.csproj` or `*.sln` | .NET / C# | Language | `src/`, `Controllers/`, `Services/`, `Models/` |
| `build.gradle*` or `pom.xml` | Java / Kotlin | Language | `src/main/`, `src/test/` |
| `build.gradle*` + `app/src/main/AndroidManifest.xml` | Android | Platform | `app/src/main/java/`, `app/src/main/res/` |
| `Dockerfile` | Docker | Infra | root or per-service Dockerfiles |
| `docker-compose*` | Docker Compose | Infra | `services/`, root-level `docker-compose.yml` |
| `supabase/config.toml` or `supabase/` dir | Supabase | Backend | `supabase/functions/`, `supabase/migrations/` |
| `firebase.json` or `.firebaserc` | Firebase | Backend/Hosting | `functions/`, `public/`, `firestore.rules` |
| `prisma/schema.prisma` | Prisma ORM | Database | `prisma/`, `prisma/migrations/` |
| `drizzle.config.*` | Drizzle ORM | Database | `drizzle/`, `src/db/` |
| `terraform/` or `*.tf` | Terraform | IaC | `terraform/`, `modules/`, `environments/` |
| `serverless.yml` or `serverless.ts` | Serverless Framework | Infra | `functions/`, `src/handlers/` |
| `vercel.json` | Vercel | Hosting | (project-dependent) |
| `netlify.toml` | Netlify | Hosting | (project-dependent) |
| `wrangler.toml` | Cloudflare Workers | Edge | `src/`, `worker/` |

---

## Monorepo Detection

A project is likely a monorepo if any of these are true:

| Signal | Tool |
|---|---|
| `workspaces` field in root `package.json` | npm/Yarn workspaces |
| `pnpm-workspace.yaml` exists | pnpm workspaces |
| `lerna.json` exists | Lerna |
| `nx.json` exists | Nx |
| `turbo.json` exists | Turborepo |
| Multiple `package.json` files at depth 1-2 | Generic monorepo |
| `packages/`, `apps/`, or `services/` directories with independent configs | Convention-based |

When a monorepo is detected:
- Scan each workspace/package independently for its own tech stack
- The Bible should document the overall monorepo structure AND per-package details
- Path patterns in `.bible.yaml` should reference specific packages (e.g., `packages/api/src/`)

---

## Categorization Notes

**Frontend vs Backend vs Fullstack:**
- Frontend only: has UI framework (React, Vue, Svelte, Angular) but no server-side code, no database
- Backend only: has API framework (Express, FastAPI, Rails) or serverless functions but no UI framework
- Fullstack: has both, or uses a fullstack framework (Next.js, Nuxt, SvelteKit, Rails with views)

**Priority when multiple stacks detected:**
- List all detected stacks. Do not pick one.
- Group by category: Language, Framework, Platform, Database, Infra, Hosting
- The Bible sections should reflect ALL detected stacks, not just the primary one

**Edge cases:**
- A `package.json` with only dev dependencies (linting, formatting) does not make it a Node.js project
- A `Dockerfile` alone does not define the tech stack -- check what it builds
- `requirements.txt` with only `black` and `mypy` does not make it a Python project -- check for actual application code
