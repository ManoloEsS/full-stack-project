# Project Decisions

Architecture Decision Records (ADRs) for the full-stack application.

## Table of Contents

### Tech Stack

| ADR                                                                     | Title                                           | Status   |
| ----------------------------------------------------------------------- | ----------------------------------------------- | -------- |
| [ADR-001](#adr-001-use-typescript-as-the-primary-language)              | Use TypeScript as the primary language          | Proposed |
| [ADR-002](#adr-002-use-bun-as-the-all-in-one-toolchain)                 | Use Bun as the all-in-one toolchain             | Proposed |
| [ADR-004](#adr-004-use-postgresql-via-docker-for-local-development)     | Use PostgreSQL via Docker for local development | Proposed |
| [ADR-005](#adr-005-use-react-spa-for-the-frontend)                      | Use React SPA for the frontend                  | Proposed |
| [ADR-006](#adr-006-use-zod-for-validation-and-type-inference)           | Use Zod for validation and type inference       | Proposed |
| [ADR-007](#adr-007-use-drizzle-orm-for-database-access)                 | Use Drizzle ORM for database access             | Proposed |
| [ADR-013](#adr-013-use-basic-react-state-management-usestateusecontext) | Use basic React state management                | Proposed |
| [ADR-017](#adr-017-use-dotenv-for-environment-configuration)            | Use dotenv for environment configuration        | Proposed |

### Architecture

| ADR                                                                                         | Title                                        | Status   |
| ------------------------------------------------------------------------------------------- | -------------------------------------------- | -------- |
| [ADR-008](#adr-008-adopt-strict-tdd-as-the-development-approach)                            | Adopt strict TDD as the development approach | Proposed |
| [ADR-009](#adr-009-organize-code-in-vertical-slices-feature-folders--shared-infrastructure) | Organize code in vertical slices             | Proposed |
| [ADR-011](#adr-011-use-rest-api-style)                                                      | Use REST API style                           | Proposed |
| [ADR-014](#adr-014-monorepo-with-bun-workspaces)                                            | Monorepo with Bun workspaces                 | Proposed |
| [ADR-015](#adr-015-use-jwt-for-authentication)                                              | Use JWT for authentication                   | Proposed |
| [ADR-018](#adr-018-react-native-as-a-future-nice-to-have)                                   | React Native as a future nice-to-have        | Proposed |
| [ADR-019](#adr-019-project-scaffolding-with-bun-workspaces)                                 | Project scaffolding with Bun workspaces      | Proposed |

### DevOps & Infra

| ADR                                                              | Title                                        | Status   |
| ---------------------------------------------------------------- | -------------------------------------------- | -------- |
| [ADR-003](#adr-003-set-up-ci-pipeline-early-with-github-actions) | Set up CI pipeline early with GitHub Actions | Accepted |
| [ADR-016](#adr-016-use-neon-postgres-on-render-for-production)   | Use Neon Postgres on Render for production   | Proposed |
| [ADR-020](#adr-020-docker-based-local-database-management)       | Docker-based local database management       | Proposed |

### UX/UI

| ADR                                                    | Title                              | Status   |
| ------------------------------------------------------ | ---------------------------------- | -------- |
| [ADR-012](#adr-012-use-plain-css-for-frontend-styling) | Use plain CSS for frontend styling | Proposed |

### Code Style

| ADR                                                                 | Title                                                                  | Status   |
| ------------------------------------------------------------------- | ---------------------------------------------------------------------- | -------- |
| [ADR-021](#adr-021-code-style-enforcement-with-prettier-and-eslint) | Code style: single quotes, semicolons, enforced with Prettier + ESLint | Proposed |
| [ADR-022](#adr-022-environment-variable-management-with-env-files) | Environment variable management with --env-file flags | Proposed |
| [ADR-023](#adr-023-eslint-per-package-rules-and-import-boundaries) | ESLint per-package rules and import boundaries | Proposed |
| [ADR-024](#adr-024-strict-typescript-configuration) | Strict TypeScript configuration | Proposed |
| [ADR-025](#adr-025-pre-commit-hooks-with-husky-and-lint-staged) | Pre-commit hooks with Husky and lint-staged | Proposed |
| [ADR-026](#adr-026-keep-node_env-in-environment-files) | Keep NODE_ENV in environment files | Proposed |

---

## Template

Each decision follows this format:

- **ADR-ID**: Incremental ID (ADR-XXX)
- **Title**: Short descriptive title
- **Status**: Proposed | Accepted | Deprecated | Superseded
- **Category**: Tech Stack | Architecture | DevOps & Infra | UX/UI
- **Date**: When the decision was made
- **Context**: What is driving this decision
- **Decision**: What was decided
- **Alternatives Considered**: Other options evaluated and why they were not chosen
- **Consequences**: Impacts, tradeoffs, and risks

---

## ADR-001: Use TypeScript as the primary language

- **Status**: Proposed
- **Category**: Tech Stack
- **Date**: 2026-06-02
- **Context**: The project is a full-stack application. Without type safety, runtime errors increase and developer experience degrades at scale. TypeScript provides compile-time checks, better IDE support, and self-documenting code across frontend and backend.
- **Decision**: Use TypeScript for all frontend and backend code.
- **Alternatives Considered**: JavaScript — no serious alternative was evaluated; TypeScript is the clear industry standard for this use case.
- **Consequences**: Improved type safety and DX across the stack; slight build-tooling overhead; shared types between frontend and backend become possible.

---

## ADR-002: Use Bun as the all-in-one toolchain (with Vite for frontend dev/build)

- **Status**: Proposed
- **Category**: Tech Stack
- **Date**: 2026-06-02
- **Context**: Full-stack projects typically require a runtime, package manager, test runner, and bundler — often from different tools (Node.js, npm/pnpm, Jest/Vitest, Vite/Webpack). This fragments tooling, increases config overhead, and slows down dev workflows. While Bun covers most concerns, its browser bundling (`bun build --target browser`) lacks code splitting, CSS extraction, asset hashing, and HMR needed for frontend development.
- **Decision**: Use Bun as the runtime, package manager, and test runner for the entire project. Use `bun build --target bun` for the server bundle. Use Vite for the frontend dev server (HMR) and production builds (code splitting, CSS extraction, asset hashing).
- **Alternatives Considered**: Node.js — mature but requires separate tools for each concern; `bun build --target browser` for frontend — lacks code splitting, CSS handling, and HMR.
- **Consequences**: Bun remains the primary toolchain for server and test concerns; Vite fills the gap for frontend dev experience (HMR) and production optimization; one additional devDependency in `@project/web`; Vite's proxy config routes `/api` requests to the Express server during development; the server still uses `bun build` for its own bundle.

---

## ADR-003: Set up CI pipeline early with GitHub Actions

- **Status**: Accepted
- **Category**: DevOps & Infra
- **Date**: 2026-06-02
- **Context**: Establishing CI early ensures code quality is enforced from the start. Without CI, type errors, lint issues, and test failures go undetected until manually discovered. Since this is a TypeScript full-stack app using Bun, the pipeline should leverage Bun's speed.
- **Decision**: Create a GitHub Actions CI pipeline that runs on every push to `main` and every pull request targeting `main`. Three parallel jobs: (1) Lint & Format — Prettier check + ESLint; (2) TypeCheck — `tsc --noEmit` across all packages; (3) Test — Bun test runner with a PostgreSQL 16 service container providing the `project_test` database.
- **Alternatives Considered**: Minimal pipeline first — adds stages later but risks missing issues early; other CI platforms (GitLab CI, CircleCI, Jenkins) — GitHub Actions is the natural choice if the repo is on GitHub.
- **Consequences**: Catches issues before merge from day one; slightly more upfront setup work; pipeline will need updating as the project grows (e.g., adding build/deploy stages later); Bun support in GitHub Actions requires using `oven-sh/setup-bun` action.

---

## ADR-004: Use PostgreSQL via Docker for local development

- **Status**: Proposed
- **Category**: Tech Stack
- **Date**: 2026-06-02
- **Context**: A full-stack application needs a reliable, relational database. Local development requires a reproducible, easy-to-set-up database instance that matches the production environment. Docker provides consistent, disposable database instances without local installation hassles.
- **Decision**: Use PostgreSQL as the primary database, running via official Docker images for local development. Production database hosting decision is deferred to ADR-016.
- **Alternatives Considered**: MongoDB — document-oriented NoSQL database; doesn't align with the need for relational data integrity and ACID compliance. SQLite — lighter but lacks concurrency and production parity.
- **Consequences**: Consistent local dev environment; easy onboarding (just `docker compose up`); production DB is Neon (ADR-016); Docker dependency for local dev; need to manage data persistence across container restarts.

---

## ADR-005: Use React SPA for the frontend

- **Status**: Proposed
- **Category**: Tech Stack
- **Date**: 2026-06-02
- **Context**: The frontend needs a component-based UI framework. The team has experience with React, reducing onboarding time. A SPA approach keeps the frontend decoupled from the backend, aligning with the full-stack TypeScript setup.
- **Decision**: Use React as a single-page application for the frontend. SSR frameworks like Next.js can be considered later if SEO or initial load performance becomes a concern.
- **Alternatives Considered**: Vue — simpler syntax and less boilerplate, but team has less experience with it; doesn't leverage existing team knowledge.
- **Consequences**: Team productivity from day one due to familiarity; large ecosystem of libraries and components; SPA means client-side routing and initial load considerations; no SSR out of the box (can migrate to Next.js later if needed).

---

## ADR-006: Use Zod for validation and type inference

- **Status**: Proposed
- **Category**: Tech Stack
- **Date**: 2026-06-02
- **Context**: A full-stack TypeScript app needs consistent, type-safe validation on both the frontend (forms) and backend (API input). Without runtime validation, TypeScript types only exist at compile time and can't guarantee incoming data shape at the boundaries. Zod provides a single source of truth for schemas that both validate at runtime and infer TypeScript types.
- **Decision**: Use Zod for API input validation on the backend and form validation on the frontend.
- **Alternatives Considered**: No serious alternative — Zod is the de facto standard for TypeScript-first schema validation with a minimal API and built-in type inference.
- **Consequences**: Shared validation schemas between frontend and backend reduce duplication; Zod schemas double as TypeScript type definitions via `z.infer`; slight runtime overhead on validation; strong ecosystem integration with forms (e.g., react-hook-form) and API frameworks.

---

## ADR-007: Use Drizzle ORM for database access

- **Status**: Proposed
- **Category**: Tech Stack
- **Date**: 2026-06-02
- **Context**: The backend needs a type-safe way to interact with PostgreSQL. ORMs provide query building, migration management, and type safety, but many add heavy abstractions or require code generation steps. Drizzle offers SQL-like syntax with full TypeScript inference and zero runtime overhead beyond query building.
- **Decision**: Use Drizzle ORM as the database access layer for the backend, connecting to PostgreSQL.
- **Alternatives Considered**: Prisma — schema-first approach with a Prisma schema file and code generation step; heavier runtime, slower cold starts, and less SQL-like control. Drizzle was preferred for its lighter footprint, native TypeScript types without codegen, and SQL-like DX.
- **Consequences**: Full type safety on queries and results without code generation; SQL-like syntax feels natural for developers who know SQL; lightweight runtime; Drizzle has a smaller community than Prisma; migration tooling is less mature; schema is defined in TypeScript directly, enabling easy sharing with Zod schemas.

---

## ADR-008: Adopt strict TDD as the development approach

- **Status**: Proposed
- **Category**: Architecture
- **Date**: 2026-06-02
- **Context**: Without a structured testing discipline, code quality and design can degrade as the project grows. TDD forces thinking about interfaces and contracts before implementation, leading to better architecture and fewer bugs. This is especially important for a full-stack app where backend APIs and frontend logic must work reliably together.
- **Decision**: Adopt strict TDD — all code must have tests written first. No code is merged without corresponding tests. Pragmatic scope: TDD applies across the full stack but with priority on core business logic and API contracts.
- **Alternatives Considered**: Test-after approach — leads to lower coverage and tests that verify implementation rather than behavior; no structured testing — relies on developer discretion, inconsistent coverage.
- **Consequences**: Higher code quality and better software design from the start; living documentation through tests; slower initial development pace; requires discipline and team buy-in; some boilerplate for simple cases; CI pipeline (ADR-003) enforces the red-green-refactor cycle; Bun's built-in test runner (ADR-002) will be used as the test framework.

---

## ADR-009: Organize code in vertical slices (feature folders + shared infrastructure)

- **Status**: Proposed
- **Category**: Architecture
- **Date**: 2026-06-02
- **Context**: Traditional layered architecture (controllers/, services/, models/) groups code by technical concern, making it hard to understand a full feature flow. As the project grows, features become scattered across many folders, increasing cognitive load and merge conflicts. Vertical slices organize code by feature, keeping everything related to "users" or "orders" together.
- **Decision**: Organize both frontend and backend code by feature/domain in vertical slices. Each feature folder contains its route, handler, and related UI/components. Shared infrastructure (database, auth, config) remains in common modules. Slices are thin — they compose over a shared service/data layer rather than duplicating it.
- **Alternatives Considered**: Layered architecture — code organized by technical layer; familiar but scatters feature logic across directories and increases cross-cutting maintenance. Full vertical slice architecture — each slice fully independent with its own data; too much duplication for a monorepo-style full-stack app.
- **Consequences**: Feature-related code is co-located and easy to navigate; reduces merge conflicts since teams work in different feature folders; shared data layer (Drizzle) and auth/config remain centralized; easier to reason about a single feature end-to-end; requires discipline to keep slices thin and push shared logic into common modules.

---

## ADR-010: Use Express as the backend framework

- **Status**: Proposed
- **Category**: Tech Stack
- **Date**: 2026-06-02
- **Context**: The backend needs an HTTP framework to serve the REST API. The team is experienced with Express, minimizing onboarding time. Running on Bun (ADR-002) provides the performance benefits of Bun's runtime while keeping the familiar Express API.
- **Decision**: Use Express as the backend HTTP framework.
- **Alternatives Considered**: Hono, Elysia — newer Bun-native frameworks with modern APIs; not chosen due to team experience with Express.
- **Consequences**: Team productive from day one; massive middleware ecosystem; Express is not fully Bun-optimized (Hono/Elysia would leverage Bun APIs more); some middleware may need Bun compatibility checks.

---

## ADR-011: Use REST API style

- **Status**: Proposed
- **Category**: Architecture
- **Date**: 2026-06-02
- **Context**: The frontend and backend need to communicate. REST is the most widely understood and well-documented API style, providing clear resource-based endpoints.
- **Decision**: Use REST for all client-server communication.
- **Alternatives Considered**: None — REST is the standard choice for this project's needs.
- **Consequences**: Predictable URL-based endpoints; easy to cache and scale; no type-safety between client and server (mitigated by shared Zod schemas, ADR-006); tRPC could be considered in the future if end-to-end type safety becomes a priority.

---

## ADR-012: Use plain CSS for frontend styling

- **Status**: Proposed
- **Category**: UX/UI
- **Date**: 2026-06-02
- **Context**: The frontend needs a styling approach. Plain CSS keeps things simple with no build tooling dependencies, no class name hashing, and full control over styles.
- **Decision**: Use plain CSS files for all frontend styling.
- **Alternatives Considered**: CSS Modules — adds scoping but unnecessary for initial simplicity; Tailwind CSS — utility-first approach adds a dependency and learning curve; styled-components — runtime CSS-in-JS overhead.
- **Consequences**: Zero additional dependencies or build steps; full control over styling; no scoped CSS by default — naming conventions or BEM needed to avoid collisions; can migrate to CSS Modules or Tailwind later if needed.

---

## ADR-013: Use basic React state management (useState/useContext)

- **Status**: Proposed
- **Category**: Tech Stack
- **Date**: 2026-06-02
- **Context**: The frontend needs state management. For now, React's built-in useState and useContext are sufficient. No complex global state is anticipated at this stage.
- **Decision**: Use React's built-in state management (useState and useContext) only. No external state library.
- **Alternatives Considered**: Zustand — lightweight but unnecessary for current scope; Redux — overkill for this stage; TanStack Query — can be added later for server state caching if needed.
- **Consequences**: Zero additional dependencies; simplest possible state management; may need to introduce an external library (Zustand, TanStack Query) if server state or complex global state becomes a concern.

---

## ADR-014: Monorepo with Bun workspaces

- **Status**: Proposed
- **Category**: Architecture
- **Date**: 2026-06-02
- **Context**: The full-stack app has a frontend and backend that share types, schemas (Zod, ADR-006), and possibly utilities. A monorepo keeps everything in one repository, enabling code sharing and atomic commits across the stack. Bun workspaces provide native monorepo support without additional tooling.
- **Decision**: Use a monorepo structure managed by Bun workspaces, with separate packages for frontend, backend, and shared code.
- **Alternatives Considered**: Turborepo — adds caching and orchestration but overkill for this project size; Nx — full framework, too heavy; separate repos — loses code sharing and atomic cross-stack commits.
- **Consequences**: Shared types and schemas between frontend and backend; single repo for all code; Bun workspaces are simple and add no extra dependency; no caching or advanced orchestration (can add Turborepo later if needed).

---

## ADR-015: Use JWT for authentication

- **Status**: Proposed
- **Category**: Architecture
- **Date**: 2026-06-02
- **Context**: The application needs an authentication mechanism. JWTs are stateless, easy to implement in a REST API, and work well across both web and potential mobile (React Native) clients.
- **Decision**: Use JWT-based authentication for the API.
- **Alternatives Considered**: Session-based auth — requires server-side session storage, less portable across clients; OAuth provider (Auth0, Clerk) — adds a third-party dependency, can be layered on later if needed.
- **Consequences**: Stateless auth, easy to scale; works seamlessly with future React Native client (ADR-018); must handle token refresh and expiration carefully; no built-in logout/revocation without a token denylist or short expiry + refresh tokens.

---

## ADR-016: Use Neon Postgres on Render for production

- **Status**: Proposed
- **Category**: DevOps & Infra
- **Date**: 2026-06-02
- **Context**: The app needs production hosting. Neon provides a serverless Postgres database with autoscaling and branching, complementing ADR-004 (PostgreSQL). Render offers simple deployment for both the backend API and React SPA.
- **Decision**: Host the production database on Neon Postgres and deploy the application on Render.
- **Alternatives Considered**: Self-hosted Postgres on Render — more ops overhead; AWS RDS — more complex setup and pricing; Supabase — includes auth and realtime features beyond current needs; Vercel — frontend-focused, less suitable for Express backend.
- **Consequences**: Managed database reduces ops burden; Neon's branching feature useful for preview environments; Render provides straightforward deployment with auto-scaling; cost will scale with usage; production DB is Neon (not Docker), so local dev uses ADR-004 Docker setup for parity testing.

---

## ADR-017: Use dotenv for environment configuration

- **Status**: Proposed
- **Category**: Tech Stack
- **Date**: 2026-06-02
- **Context**: The backend needs a way to manage environment-specific configuration (database URLs, JWT secrets, API keys). These values must not be committed to source control.
- **Decision**: Use `.env` files managed by dotenv for environment configuration.
- **Alternatives Considered**: Bun's built-in env loading — Bun reads .env files natively; dotenv provides explicit control and validation; environment-only config — no file, set vars in CI/hosting only; harder for local development.
- **Consequences**: Simple, industry-standard approach; `.env` files excluded from version control; Bun can also read .env natively, so dotenv adds explicit validation; must define a schema for required env vars (can be validated with Zod).

---

## ADR-018: React Native as a future nice-to-have

- **Status**: Proposed
- **Category**: Architecture
- **Date**: 2026-06-02
- **Context**: A mobile client may be desired in the future. React Native shares React knowledge and can reuse API clients, auth (JWT, ADR-015), and Zod schemas (ADR-006) from the web app.
- **Decision**: Keep React Native as a nice-to-have target. No immediate development, but architecture decisions (REST API, JWT auth, monorepo structure) should not preclude a future React Native client.
- **Alternatives Considered**: Flutter — different language (Dart), no React knowledge reuse; native — higher development cost, no code sharing.
- **Consequences**: Current decisions (REST, JWT, monorepo) are mobile-friendly; shared packages in monorepo (ADR-014) will make mobile integration smoother when prioritized; no immediate mobile investment.

---

## ADR-019: Project scaffolding with Bun workspaces

- **Status**: Proposed
- **Category**: Architecture
- **Date**: 2026-06-02
- **Context**: The project needs a monorepo structure (ADR-014) that separates frontend, backend, and shared code. Bun workspaces provide native monorepo support. The backend will serve the built React SPA in production, so the build pipeline must ensure `@project/web` builds before `@project/server` references the output. Vertical slices (ADR-009) dictate feature-based folder organization within each package. Shared schemas and types (ADR-006) need a dedicated package that both web and server depend on.
- **Decision**: Scaffold the project with three workspace packages: `@project/shared` (Zod schemas, shared types), `@project/server` (Express backend with feature folders, Drizzle schema/migrations, DB connection), and `@project/web` (React SPA with feature folders, components, styles). Web builds to its own `dist/`; server references `../web/dist/` as static files. Root `package.json` orchestrates build order (web first, then server) and dev scripts. Drizzle config and migrations live in the server package. Scoped package names (`@project/*`) avoid naming collisions.
- **Alternatives Considered**: Web builds into server directory — simpler deploy but couples build output paths; Turborepo/Nx — adds orchestration tooling overhead unnecessary at this scale; separate repos — loses shared code and atomic commits.
- **Consequences**: Clean separation of concerns with three packages; shared package eliminates type/schema duplication; Bun workspaces resolve `@project/shared` as a local dependency; build order enforced by root scripts; server serves production build from `../web/dist/` (relative path within workspaces); Docker Compose provides local PostgreSQL; monorepo structure supports future React Native package (ADR-018).

---

## ADR-020: Docker-based local database management

- **Status**: Proposed
- **Category**: DevOps & Infra
- **Date**: 2026-06-02
- **Context**: Developers need a PostgreSQL database running locally for development and testing. Docker provides a consistent, disposable database instance. The project needs scripts to start/stop the database, run migrations, and run tests against a separate test database — similar to the knife-roll pattern of `docker start || docker run` for fast iteration.
- **Decision**: Use a `docker start || docker run` pattern via npm scripts for the local database, alongside the existing `docker-compose.yml` for CI or advanced use. Separate `.env.development` and `.env.test` files configure different databases (`project` and `project_test`) on the same PostgreSQL container. Root scripts: `db:start` (start or create container), `db:stop`, `db:migrate`, `dev:full` (start DB, migrate, run dev), and `test` (start DB, use `.env.test`, run tests). Bun's `--env-file` flag loads the appropriate environment per context.
- **Alternatives Considered**: Docker Compose only — requires `docker compose up` every time, slower for quick start/stop; separate Docker containers for test DB — unnecessary resource overhead when a second database on the same container works fine; manual Docker commands — error-prone and inconsistent across team members.
- **Consequences**: Fast database startup with `docker start` (reuses existing container) or `docker run` (creates on first use); test isolation via `project_test` database on the same container; `dev:full` script provides one-command onboarding; `docker-compose.yml` retained for CI or `docker compose up` preference; environment files are gitignored to prevent secrets from being committed; `docker-compose.yml` stays as an alternative for those who prefer it.

---

## ADR-021: Code style — single quotes, semicolons, enforced with Prettier + ESLint

- **Status**: Proposed
- **Category**: Code Style
- **Date**: 2026-06-02
- **Context**: Without enforced formatting rules, code style will drift across the project as different developers write code. Inconsistent quote styles, semicolons, trailing commas, and spacing create noise in diffs and reduce readability. The project needs both automated formatting (Prettier) and code quality rules (ESLint) from the start to support the TDD approach (ADR-008) and keep CI checks consistent.
- **Decision**: Use single quotes, always require semicolons, trailing commas, and enforce these with Prettier and ESLint. Prettier handles formatting (quotes, semicolons, indentation, line width). ESLint handles code quality (unused variables, React hooks rules, TypeScript best practices). Both run as root-level scripts available to all packages. ESLint config uses the new flat config format (`eslint.config.mjs`) with TypeScript and React plugins.
- **Alternatives Considered**: Double quotes — valid choice but Prettier default; single quotes are more common in JS/TS community; no enforcement — relies on discipline, style drifts immediately; Biome — all-in-one formatter+lint but less mature ecosystem than Prettier+ESLint.
- **Consequences**: Consistent code style across the entire project; `bun run format` auto-fixes formatting; `bun run lint` catches code quality issues; `bun run format:check` validates formatting in CI; Prettier and ESLint configs are root-level (shared across all packages); new devs just run `bun run format` and style is handled automatically.

---

## ADR-022: Environment variable management with --env-file flags

- **Status**: Proposed
- **Category**: Architecture
- **Date**: 2026-06-02
- **Context**: Different environments (development, test, production) need different configuration. The server needs `DATABASE_URL`, `JWT_SECRET`, and `PORT`. Drizzle Kit commands need access to `DATABASE_URL` when running locally. Production deployments (Render) set env vars via the platform dashboard — no `.env` files exist. CI sets env vars inline in the workflow.
- **Decision**: Use Bun's built-in `--env-file` flag to load environment files per context. All scripts that need env vars explicitly pass the appropriate file: `--env-file=.env.development` for dev commands, `--env-file=.env.test` for test commands. Production gets env vars from the hosting platform. `drizzle.config.ts` uses `process.env.DATABASE_URL!` directly — no `dotenv` import needed since the `--env-file` flag loads vars before the process starts.
- **Alternatives Considered**: Import `dotenv/config` in `drizzle.config.ts` — works but adds a dependency and conditional logic; separate `.env` for each environment — same pattern but `--env-file` is a Bun native feature; environment-only config — no file, set vars in CI/hosting only; harder for local development.
- **Consequences**: Explicit about which environment each command targets; no conditional env loading logic in application code; `.env.*` patterns are gitignored; `.env.example` is committed as documentation; the `start` script (production) has no `--env-file` flag since the platform provides env vars; test script runs `db:migrate` before tests to ensure schema is up to date.

---

## ADR-023: ESLint per-package rules and import boundaries

- **Status**: Proposed
- **Category**: Architecture
- **Date**: 2026-06-02
- **Context**: The monorepo has three packages with a clear dependency direction: `server ← shared → web`. Without enforcement, developers could import `@project/web` from `@project/server` or vice versa, creating circular dependencies and breaking the architecture. Additionally, React lint rules should only apply to `@project/web`, not `@project/server`.
- **Decision**: Split ESLint configuration into per-package rule sets. `@project/shared` gets TypeScript rules + `no-console: error` + blocked from importing server or web. `@project/server` gets TypeScript rules + `no-console: warn` + blocked from importing web. `@project/web` gets TypeScript + React + React Hooks rules + `no-console: warn` + blocked from importing server. Root config files (vite.config.ts, drizzle.config.ts) allow console. All packages enforce `@typescript-eslint/consistent-type-imports` for proper bundler interop.
- **Alternatives Considered**: Single ESLint config for all packages — applies React rules to server code, creates false positives; no import boundary enforcement — relies on discipline, leads to architectural drift over time; Turborepo/Nx for boundary enforcement — overkill for this project size.
- **Consequences**: Architectural boundaries enforced at lint time; React rules only apply where relevant; `consistent-type-imports` ensures type-only imports are marked with `import type`, which is required for Bun and Vite bundlers; import violations produce clear error messages; `no-console` prevents debug logging from reaching production (error in shared, warn in server/web).

---

## ADR-024: Strict TypeScript configuration

- **Status**: Proposed
- **Category**: Tech Stack
- **Date**: 2026-06-02
- **Context**: TypeScript's `strict: true` enables many safety checks but doesn't cover all cases. `noUncheckedIndexedAccess` prevents accessing array elements or object keys from returning `T` instead of `T | undefined`. `isolatedModules` ensures each file can be transpiled independently, which is required for Bundler module resolution (used by Bun and Vite). `noImplicitOverride` forces explicit `override` keywords on class method overrides, preventing accidental method shadowing.
- **Decision**: Add `noUncheckedIndexedAccess`, `isolatedModules`, and `noImplicitOverride` to `tsconfig.base.json` so all packages inherit these strict settings.
- **Alternatives Considered**: Default `strict: true` only — misses index access safety and bundler compatibility; `verbatimModuleSyntax` instead of `isolatedModules` — stricter but requires `import type` everywhere, which we already enforce via ESLint's `consistent-type-imports`.
- **Consequences**: Array element access and object key access return `T | undefined`, requiring explicit null checks; each TS file is independently transpilable, ensuring compatibility with Bun and Vite; class method overrides must use the `override` keyword; slightly more verbose code but significantly fewer runtime type errors.

---

## ADR-025: Pre-commit hooks with Husky and lint-staged

- **Status**: Proposed
- **Category**: DevOps & Infra
- **Date**: 2026-06-02
- **Context**: Without pre-commit hooks, developers can commit code that fails formatting or lint checks, pushing issues to CI instead of catching them locally. This wastes CI minutes and slows the feedback loop. Running Prettier and ESLint on the entire project on every commit would be too slow for large codebases.
- **Decision**: Use Husky for git hooks and lint-staged for running checks only on staged files. On every `git commit`, lint-staged runs `eslint --fix` and `prettier --write` on staged `*.{ts,tsx}` files, and `prettier --write` on staged `*.{json,md,css,yml}` files. If any check fails, the commit is blocked. Developers can bypass with `--no-verify` for WIP commits.
- **Alternatives Considered**: No pre-commit hooks — relies on CI to catch issues, slower feedback; full-project lint on every commit — too slow as the project grows; Biome lint-staged — would work but project already uses Prettier + ESLint.
- **Consequences**: Formatting and lint issues are caught before push; only staged files are checked, making the hook fast (usually under 1 second); developers can bypass with `git commit --no-verify`; Husky's `prepare` script runs automatically on `bun install`; lint-staged config lives in root `package.json`.

---

## ADR-026: Keep NODE_ENV in environment files

- **Status**: Proposed
- **Category**: Architecture
- **Date**: 2026-06-02
- **Context**: `NODE_ENV` appears in `.env.example`, `.env.development`, `.env.test`, and the CI workflow YAML, but nothing in the current codebase reads or checks it. It could be considered dead weight. However, `NODE_ENV` is a widely recognized convention that Express automatically uses for behavior toggling (detailed errors in development, caching in production), Vite uses to determine build mode, and hosting platforms (Render) set it automatically in production.
- **Decision**: Keep `NODE_ENV` in `.env.example` and `.env.development` as documentation of the convention. Keep it in CI's `.env.test` inline vars. The server entry point should check `NODE_ENV` to determine behavior such as serving static files only in production, enabling detailed error messages in development, and configuring CORS or rate limiting per environment.
- **Alternatives Considered**: Remove `NODE_ENV` entirely — nothing reads it now, but it will be needed as soon as server code is written; set it only in CI and hosting — works but developers lose visibility into what environment they're running locally.
- **Consequences**: `NODE_ENV` serves as documented convention before it's actively used; server code will rely on it for environment-specific behavior (static file serving, error detail, CORS); hosting platforms set it automatically; developers see it in their `.env.development` file and understand it's an expected variable; CI sets it inline in the test job.
