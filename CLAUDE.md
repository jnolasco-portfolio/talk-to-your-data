# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repository is

A monorepo holding one product — synthetic data generation plus conversational analytics ("talk to your data") — split
into two independently built projects:

| Directory | Project | Toolchain | Docs |
| --- | --- | --- | --- |
| `api/` | Spring Boot 3.5 backend (Java 21, Maven) | `./mvnw`, MySQL, Langfuse | [api/CLAUDE.md](api/CLAUDE.md) |
| `ui/` | React 19 + Vite frontend (TypeScript) | `npm`, Pico CSS | [ui/CLAUDE.md](ui/CLAUDE.md) |

**Read the child `CLAUDE.md` before touching either project.** They hold the commands, architecture and known issues
for their half; this file only covers what spans both.

There is no root build orchestrator — no root `package.json`, no Maven aggregator, no workspace config. Maven and npm
stay separate by design. Run every command from inside `api/` or `ui/`.

## The two halves, end to end

1. **Learn** — the UI uploads a DDL file; the backend asks Gemini to normalize it into a structured model
   (tables, columns, PKs, FKs, indexes) and caches the result.
2. **Generate** — the UI sends table name + natural-language instructions; the backend returns generated rows as CSV
   strings; the UI zips them client-side (JSZip) for download.
3. **Talk** — the UI sends a question; the backend turns it into SQL, executes it against MySQL, and returns rows plus
   `metadata.content_type` (`bar` / `pie` / `line` / `table`) that tells the frontend which chart to draw.

The API contract exists twice and **must be changed in both places**: Java records in `api/src/main/java/.../dto/` and
Zod schemas in `ui/src/schemas/`. The UI parses every response with `safeParse` and throws on mismatch, so a DTO
change without the matching schema change surfaces as a runtime "Invalid data received from the server." error.

## Local services

`api/docker-compose.yaml` provides everything the two projects talk to:

```bash
cd api && docker compose up -d      # MySQL :3306, phpMyAdmin :8090, Langfuse :3001
```

The API serves on `:8081`, the UI on `:5173`. The UI calls the API cross-origin from the browser — there is no Vite
proxy — which works only because the backend controllers are annotated `@CrossOrigin()`.

## Not in this repository

These sit next to the repo and are gitignored on purpose:

- `langfuse/` — upstream clone. The backend consumes Langfuse as Docker Hub images (`docker.io/langfuse/langfuse:3`),
  never from this source tree. Do not vendor it.
- `ai-synthetic-data-generator-ui-template/` — standalone static HTML design mockups, still their own repo.
- `*.mov` — demo screen recordings (~165 MB).

## Git history

The repo was consolidated from two separate repositories. Both original histories are ancestors of `main`, so
`git log --follow -- api/pom.xml` and `git log --follow -- ui/package.json` still walk the pre-consolidation
timelines. Plain `git log -- api/` does *not* show them (path-limited log does not follow the prefix rewrite) — use
`--follow`, or `git log --all --oneline` to see everything.
