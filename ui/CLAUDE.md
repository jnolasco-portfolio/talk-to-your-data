# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in the `ui/` project.

React 19 + TypeScript 5.9 + Vite 7 frontend. See the [root CLAUDE.md](../CLAUDE.md) for how this fits with the backend.

## Commands

```bash
npm install
npm run dev       # Vite dev server on :5173
npm run build     # tsc -b && vite build
npm run lint      # eslint — currently fails, see Known issues
npm run preview   # serve the production build
```

**There is no test runner.** No vitest/jest is installed — `npm test` does not exist and there are no `*.test.ts`
files. Verification means `npm run build`, which passes.

## Architecture

```text
App.tsx                 BrowserRouter + QueryClientProvider + routes
components/layout/       AppLayout (persistent shell), Sidebar
components/              DataGenerationScreen (/generate), ChatScreen (/chat), ResultTable, DataPreview, ChartRenderer
hooks/                   useLearnDatabase, useGenerateData, useAskQuestion (TanStack Query), useLocalStorage
services/                httpClient (axios instances), DataGenerationService, TalkToYourDataService
schemas/                 Zod schemas — the typed contract with the API
```

Data flows one way: **screen → hook → service → httpClient → API**. Services own the wire calls and Zod parsing;
hooks own the mutation lifecycle; screens own the state and rendering.

### The API contract is validated, not trusted

Every response is parsed with `schema.safeParse(data)`; a mismatch logs and throws
`new Error('Invalid data received from the server.')` (see `services/DataGenerationService.ts` and
`schemas/*Schema.ts`). Types like `GenerateDataResponse` are derived from the Zod schemas via `z.infer`, so the
TypeScript types and the runtime validation can never drift apart. When the backend DTO changes, update the matching
file in `src/schemas/` — a mismatched response fails at runtime, not at compile time.

`src/services/TalkToYourDataService.ts` also normalises three response shapes into one `QuestionResponse`: a JSON
payload with a `result` string (parsed as pipe-delimited rows, `\N` means null), a raw HTML table (parsed with
`DOMParser`), or an already-structured payload.

### Backend URL and CORS

The API base URLs are **hardcoded** in `src/services/httpClient.ts` (`http://localhost:8081/api/v1/…`). There is no
Vite proxy and no env var — changing the backend port means editing that file. Cross-origin calls succeed because the
backend controllers are annotated `@CrossOrigin()`.

### Charts

The backend returns `metadata.content_type` (`bar` | `pie` | `line` | `table`) plus `category_key` and `value_key`;
`components/charts/ChartRenderer.tsx` maps that to the matching Recharts renderer and falls back to a table.

### Styling and state

Pico CSS 2 is loaded from a CDN in `index.html` — there is no npm dependency for it, and no Tailwind. Dark mode is the
`class="dark"` on `<html>` plus Pico's color stylesheet. Conversations and chat history live in `localStorage` through
`hooks/useLocalStorage.ts`.

## Gotchas

- **`npm run lint` fails today** with 10 `@typescript-eslint/no-explicit-any` errors in
  `src/components/ResultTable.tsx` and `src/components/charts/ChartRenderer.tsx`. Pre-existing, not caused by the
  monorepo move — fix the types (or relax the rule) rather than assuming lint is green.
- `tsconfig.app.json` sets `noUnusedLocals` and `noUnusedParameters`, so a leftover import **fails `npm run build`**
  (TS6133). This has already broken the build once — remove unused imports rather than casting them away.
- Generated CSV rows are zipped in the browser (`jszip` in `components/DataPreview.tsx`); the backend does not package
  downloads.
- The frontend never talks to MySQL or Langfuse directly, and the backend does not serve the frontend — they are
  separate processes started separately.
