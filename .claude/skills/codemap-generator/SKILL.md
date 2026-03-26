---
name: codemap-generator
description: Generate a comprehensive system diagram and codemap document for any local codebase. Produces a self-contained markdown file with architecture overview, ASCII component diagrams, data flows, API surface, tech stack, and annotated directory map. Use when user asks for "codemap", "system diagram", "architecture diagram", "codebase overview", "generate a map of this repo", "map this codebase", or any request for a visual/structural overview of a project.
---

# Codemap Generator

Generate a comprehensive, self-contained codemap document for any local codebase. The output is a single markdown file with ASCII diagrams that renders anywhere — no Mermaid, no external tooling required.

## When to Use

- User asks for a "codemap", "system diagram", "architecture map", or "codebase overview"
- User wants a structural reference document for a repo
- User needs to onboard someone and wants a visual architecture doc
- User says "map this repo", "diagram this codebase", "generate an architecture overview"

## Input

The codebase path. Defaults to the current working directory if none is given.

Verify the path is a valid directory before proceeding.

## Output

A single markdown file named `{project-name}-codemap.md` saved to the current working directory (or a user-specified location). The file should be immediately useful as architecture documentation.

## Execution

This is a single-pass document generator. Do NOT ask interactive questions — gather everything, then produce the document.

### Step 1: Discover Project Metadata

Read these files (skip any that don't exist, do not error):

**Must-read (in parallel):**
- Top-level directory listing (1 level deep)
- `README.md` (or `README.rst`, `README.txt`)
- Package manager files: `package.json`, `pyproject.toml`, `Cargo.toml`, `go.mod`, `mix.exs`, `pom.xml`, `build.gradle`
- `Makefile` (or `justfile`, `Taskfile.yml`)

**Read if present (in parallel):**
- `CONTRIBUTING.md`, `ARCHITECTURE.md`, `DESIGN.md`
- `CLAUDE.md`, `AGENTS.md` (at root and in major subdirectories)
- `docker-compose.yml` / `docker-compose.yaml` / `docker/`
- `.env.example`
- `config.example.yaml` / `config.example.json`
- CI config: `.github/workflows/`, `.gitlab-ci.yml`, `Jenkinsfile`

From these, extract:
- Project name and one-line description
- Tech stack (languages, frameworks, key libraries)
- Available dev/build/test commands
- Service ports and URLs
- Major subdirectory purposes

### Step 2: Map the Architecture

For each major subdirectory (e.g., `backend/`, `frontend/`, `server/`, `client/`, `packages/`, `apps/`, `services/`, `src/`):

1. Read its directory listing (2 levels deep)
2. Read its own `README.md`, `CLAUDE.md`, or `AGENTS.md` if present
3. Identify entry points: `main.*`, `index.*`, `app.*`, `server.*`, `__init__.py`, route definitions
4. Identify the layer pattern: is it layered (controllers/services/repos), feature-based, or domain-driven?

Determine:
- The dependency direction between layers (which imports which)
- Any strict boundaries (e.g., "A never imports B" rules)
- Proxy/routing setup (nginx, API gateway, etc.)

### Step 3: Catalog the API Surface

Find and list:
- HTTP route/endpoint definitions (REST routers, GraphQL resolvers, RPC handlers)
- CLI commands or entry points
- Middleware chains or plugin/hook systems
- Event handlers, queue consumers, cron jobs
- WebSocket or streaming endpoints
- Configuration schemas (what config keys exist and what they control)

Group endpoints by module/router. Include HTTP method and path.

### Step 4: Analyze Key Data Flows

Identify the 3-7 most important data flows in the system. For each flow, trace:
- Where data enters (HTTP request, CLI command, IM message, scheduled job)
- What services/components it passes through
- Where it exits (response, file output, database write, external API call)

Common flows to look for:
- The primary user-facing flow (e.g., "chat message" or "API request")
- Configuration/management flow
- File upload/processing flow
- Authentication flow
- Background job/async processing flow

### Step 5: Generate the Codemap Document

Produce the markdown file with the following sections in order. Use ASCII box diagrams (`+--+`, `|  |`, arrows `-->`, `v`, `|`) for all diagrams. Keep each section concise and scannable.

#### Section 1: High-Level Architecture
An ASCII box diagram showing:
- All services/components as boxes
- Ports in parentheses
- Arrows showing connections and their type (HTTP, import, events, proxy)
- External systems (databases, APIs, message queues)

Example style:
```
              +---------------------+
              |   Nginx (Port 80)   |
              +---+------------+----+
                  |            |
           /api/* |            | /*
                  v            v
         +------------+  +-----------+
         | Backend    |  | Frontend  |
         | (Port 8000)|  | (Port 3000)|
         +-----+------+  +-----------+
               |
               v
         +------------+
         | Database   |
         +------------+
```

#### Section 2: Repository Structure
An annotated directory tree (top 2-3 levels) with inline comments explaining each directory's purpose:
```
project/
|-- backend/          # Python API server
|   |-- app/          # Application layer
|   `-- packages/     # Shared libraries
|-- frontend/         # React web UI
`-- scripts/          # Build & deployment scripts
```

#### Section 3: Backend Architecture (if applicable)
- Layer diagram showing internal structure
- Module breakdown with one-line descriptions
- Middleware chain (if present) in execution order
- Key abstractions or patterns used

#### Section 4: Frontend Architecture (if applicable)
- Component hierarchy or page structure
- Data flow pattern (how state is managed, how data fetches work)
- Routing structure

#### Section 5: API Surface
Grouped endpoint listing:
```
Module/Router (base path)
|-- METHOD /path    # Description
|-- METHOD /path    # Description
```

#### Section 6: Configuration Files
What files configure what:
```
config.yaml         -> Models, tools, sandbox, memory settings
extensions.json     -> MCP servers, skill states
.env                -> API keys, secrets
```

#### Section 7: Technology Stack
Table format:
```
| Layer     | Technology              |
|-----------|-------------------------|
| Frontend  | Next.js, React, TS      |
| Backend   | Python, FastAPI         |
```

#### Section 8: Port Map (if multi-service)
```
| Service   | Port | Purpose              |
|-----------|------|----------------------|
| Nginx     | 80   | Reverse proxy        |
| Backend   | 8000 | API server           |
```

#### Section 9: Key Data Flows
Numbered summary of the most important flows:
```
1. CHAT:   Browser -> Nginx -> Backend -> LLM -> SSE Response
2. UPLOAD: Browser -> API -> Storage -> Processing
```

#### Section 10: Built-in Plugins/Skills/Extensions (if applicable)
List of bundled plugins, skills, or extension points with one-line descriptions.

### Step 6: Deliver

1. Save the file as `{project-name}-codemap.md` in the current directory
2. Tell the user the file has been saved and give a brief (2-3 sentence) summary of what the codemap covers

## Format Rules

- **ASCII diagrams only** — no Mermaid, no PlantUML, no external rendering needed
- **Tables** for structured data (tech stack, ports, endpoints)
- **Annotated trees** for directory structure (use `|--` and inline `#` comments)
- **Numbered lists** for data flows
- **Concise** — architecture overview, not code walkthrough. Each section should be scannable in under 30 seconds.
- **Accurate** — only document what you actually found in the codebase. Do not speculate or add generic filler.
- Skip sections that don't apply (e.g., no "Frontend Architecture" for a CLI-only tool, no "Port Map" for a library)

## Anti-Patterns

| Don't | Do Instead |
|-------|-----------|
| Use Mermaid or PlantUML | Use ASCII box diagrams |
| Speculate about architecture | Only document what's in the code |
| Include code-level details | Stay at the architecture/module level |
| Write prose paragraphs | Use diagrams, tables, trees, and bullet points |
| Ask the user interactive questions | Gather everything in one pass, then output |
| Include line-by-line references | Reference directories and files, not lines |
| Skip sections silently | Note "N/A" or omit the section header entirely |
| Generate a generic template | Every section must reflect THIS specific codebase |
