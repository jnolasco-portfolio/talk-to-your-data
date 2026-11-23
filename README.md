<div align="center">

# Data Assistant

**Synthetic data generation and conversational analytics over your own SQL schema.**

`Data Assistant` turns a DDL schema into realistic synthetic data, lets you refine it with natural-language
instructions, and then lets you *talk to that data* — asking questions in plain English and getting answers back as
text, tables, and charts.

[![Java](https://img.shields.io/badge/Java-21-ED8B00?logo=openjdk&logoColor=white)](https://openjdk.org/)
[![Spring Boot](https://img.shields.io/badge/Spring_Boot-3.5-6DB33F?logo=springboot&logoColor=white)](https://spring.io/projects/spring-boot)
[![React](https://img.shields.io/badge/React-19-61DAFB?logo=react&logoColor=black)](https://react.dev/)
[![TypeScript](https://img.shields.io/badge/TypeScript-5.9-3178C6?logo=typescript&logoColor=white)](https://www.typescriptlang.org/)
[![MySQL](https://img.shields.io/badge/MySQL-8-4479A1?logo=mysql&logoColor=white)](https://www.mysql.com/)
[![Langfuse](https://img.shields.io/badge/Langfuse-3-332300?logo=langfuse&logoColor=white)](https://langfuse.com/)

</div>

---

> Upload a `.sql` file, generate realistic rows with a sentence, then ask your data questions and get charts back —
> one pipeline, one schema, no SQL hand-writing.

- **Learn** — an LLM normalizes your DDL into a structured model (tables, columns, types, keys, comments), cached per
  conversation.
- **Generate** — *"10 authors from Spain"* becomes rows that respect the schema's types, primary keys and foreign keys.
  The browser packages them into a downloadable ZIP of CSVs.
- **Talk** — a question becomes SQL, is executed against MySQL, and comes back as prose, a sortable table and a
  bar/pie/line chart.
- **Traced** — every LLM call is observed with Micrometer and exported over OTLP to Langfuse: prompt, completion and
  latency per conversation.
- **Validated** — every payload the UI receives is parsed with a Zod schema; anything that does not match is rejected
  rather than rendered.

## Talk to your data

![Asking "Show me top 5 author's by countries" and getting back SQL plus a bar chart](screenshots/talk-to-your-data.gif)

## How it works

<table>
  <tr>
    <td width="33%" valign="top" align="center"><img src="screenshots/01-learn-schema.png" width="300" alt="Upload DDL schema form"><br><b>1 · Learn</b><br><sub>A DDL file becomes a structured model</sub></td>
    <td width="33%" valign="top" align="center"><img src="screenshots/02-generate-preview.png" width="300" alt="Generated rows previewed in a table"><br><b>2 · Generate</b><br><sub>Plain English becomes valid rows</sub></td>
    <td width="33%" valign="top" align="center"><img src="screenshots/03-talk-bar.png" width="300" alt="A question answered with SQL and a bar chart"><br><b>3 · Talk</b><br><sub>A question becomes SQL, rows and a chart</sub></td>
  </tr>
</table>

## The problem

Teams prototyping a product rarely have realistic data — and the data they *do* have rarely looks like production
data. Meanwhile, business questions still get answered by a human writing SQL.

This project closes both gaps with one pipeline:

1. **Learn** — upload a SQL schema. The backend normalizes the DDL into a structured model (tables, columns, types,
   keys, comments) using an LLM, with the result cached per conversation.
2. **Generate** — ask for rows in plain language ("10 authors from Spain"). The backend produces valid synthetic data
   that respects the schema's constraints, primary keys and foreign keys. The frontend packages the result into a
   downloadable ZIP of CSVs.
3. **Talk** — switch to the *Talk to your data* tab and ask questions about the schema. The backend converts the
   question to SQL (with a safety valve for "cannot answer this"), executes it against the live schema, and returns
   rows plus metadata. The frontend renders the answer as prose, a sortable table, and bar/pie/line charts.

## Walkthrough

### Data generation

Upload a DDL schema, pick a table, describe the rows you want:

![Data generation screen with the DDL upload form and advanced parameters](screenshots/01-learn-schema.png)

The generated rows land in a sortable preview and come out as a ZIP of CSVs — one file per table:

![Generated authors previewed in a table next to a Download All (.zip) button](screenshots/02-generate-preview.png)

### Asking your data questions

Ask in plain English. The backend answers with the SQL it ran, so you can check the reasoning:

![A question answered with the generated SQL and a bar chart of authors by country](screenshots/03-talk-bar.png)

The chart type is not hard-coded — the backend returns `metadata.content_type` and the frontend draws the matching
chart:

![A pie chart of the most lent genres, computed over a three-table join](screenshots/04-talk-pie.png)

![A line chart of monthly loan volume between 2022 and 2024](screenshots/05-talk-line.png)

Not every question is a chart. A simple lookup comes back as a table:

![A table answering "What is stephen king's nationality?"](screenshots/06-talk-table.png)

## Architecture

```mermaid
graph LR
    subgraph UI["ui/ (React 19 + Vite + TS)"]
        direction TB
        G1["/generate\nData Generation"]
        G2["/chat\nTalk to your data"]
    end
    
    subgraph API["api/ (Spring Boot 3.5 + Spring AI)"]
        direction TB
        L1["POST /api/v1/schema-assistant/learn"]
        L2["POST /api/v1/schema-assistant/generate"]
        L3["POST /api/v1/data-conversations/questions"]
    end
    
    UI <-->|HTTP calls| API
    
    style UI fill:#e3f2fd,stroke:#1976d2,stroke-width:2px
    style API fill:#e8f5e9,stroke:#388e3c,stroke-width:2px
    
    OT["Otel → Langfuse"] -->|prompts, traces, latency| LF["Langfuse 3"]
    
    style OT fill:#fff3e0,stroke:#fb8c00,stroke-width:2px
    style LF fill:#e8f5e9,stroke:#388e3c,stroke-width:2px
```

Every LLM call is observed through Micrometer and exported over OTLP to **Langfuse**, so prompts, completions and
latency are traceable per conversation.

## Tech stack

| Layer | Technology |
| --- | --- |
| Frontend | React 19, TypeScript 5.9, Vite 7, React Router 7, TanStack Query 5, React Hook Form, Zod, Recharts, Pico CSS, JSZip, Axios |
| Backend | Java 21, Spring Boot 3.5, Spring AI 1.0 (Vertex AI / Gemini 2.5 Flash), Liquibase, Spring Cache, Maven |
| Observability | Micrometer + OpenTelemetry OTLP → Langfuse 3 (Prompts, traces, latency) |
| Data | MySQL 8.4 (LTS, image pinned in `api/docker-compose.yaml`), phpMyAdmin for schema inspection |
| Local infra | Docker Compose — MySQL, phpMyAdmin, Langfuse (web, worker, Postgres, ClickHouse, MinIO, Redis) |
| Testing | JUnit 5 + Mockito (unit), Maven Failsafe (integration) |

## Repository layout

```mermaid
graph TD
    root["."]
    api["api/\nSpring Boot backend (Maven)"]
    ui["ui/\nReact frontend (Vite)"]
    readme["README.md"]
    
    root --> api
    root --> ui
    root --> readme
    
    api --> dc["docker-compose.yaml\nMySQL + phpMyAdmin + Langfuse stack"]
    api --> schemas["schemas/\nDDL for MySQL bootstrap"]
    api --> prompts["src/main/resources/prompts/\n(.st prompt templates)"]
    api --> docs["documentation/\nBackend plan, Postman checks"]
    
    ui --> components["src/components/\nScreens, layout, charts"]
    ui --> schemas2["src/schemas/\nZod schemas"]
    ui --> services["src/services/\nAxios clients + service layer"]
    ui --> fdocs["documentation/\nAPI contract, frontend plan"]
```

This repository was consolidated from two separate repositories (`…-api` and `…-ui`); their full commit history is
preserved, so `git log -- api/` and `git log -- ui/` still walk the original timelines.

## Getting started

### 1. Infrastructure (MySQL + Langfuse)

```bash
cd api
docker compose up -d
```

- MySQL → `localhost:3306` (root password `1234`), phpMyAdmin → <http://localhost:8090>
- Langfuse → <http://localhost:3001>

### 2. Backend

```bash
cd api
export GOOGLE_CLOUD_PROJECT=<your-gcp-project>
export GOOGLE_CLOUD_LOCATION=us-central1
export LANGFUSE_OTEL_AUTH=<base64 of "pk-…:sk-…">   # Langfuse project API keys
./mvnw spring-boot:run
```

The API serves on <http://localhost:8081>.

### 3. Frontend

```bash
cd ui
npm install
npm run dev
```

The UI serves on <http://localhost:5173> and talks to the API at `http://localhost:8081`
(configured in `ui/src/services/httpClient.ts`).

### Tests

```bash
cd api && ./mvnw verify      # unit + integration (Failsafe) — needs MySQL up and GCP credentials
cd ui  && npm run build      # type-check + bundle (there is no UI test runner yet)
```

Known issues in both projects are tracked in [`api/CLAUDE.md`](api/CLAUDE.md) and [`ui/CLAUDE.md`](ui/CLAUDE.md).

## API surface

| Method | Endpoint | Purpose |
| --- | --- | --- |
| `POST` | `/api/v1/schema-assistant/learn` | Normalize an uploaded DDL schema into a structured model (multipart). |
| `POST` | `/api/v1/schema-assistant/generate` | Generate synthetic rows for a learned schema, driven by natural-language instructions. |
| `POST` | `/api/v1/data-conversations/questions` | Answer a question over the schema: NL → SQL → executed result, with metadata for charting. |

Full request/response shapes live in [`ui/documentation/api-contract.md`](ui/documentation/api-contract.md); the
Postman collection notes are in [`api/documentation/postman-tests.md`](api/documentation/postman-tests.md).

## Design notes

- **Responses are validated at the boundary.** Every payload the UI receives is parsed with a Zod schema; anything
  that does not match is rejected rather than rendered.
- **The DDL is cached, the data is not.** Schema learning is `@Cacheable` per `(conversationId, schemaName, prompt)`;
  generated rows are intentionally not persisted by default.
- **Prompts live in resources, not in code.** `.st` templates under `api/src/main/resources/prompts/` keep the
  NL → SQL and schema → JSON instructions editable without touching Java.
- **The model is told to refuse.** The NL → SQL prompt has an explicit "cannot answer this" path, so an unanswerable
  question returns a message instead of an invented query.
- **The chart is the backend's choice.** `metadata.content_type` (`bar` / `pie` / `line` / `table`) decides what the
  frontend draws, so new chart types do not need a frontend change.

## Recordings

Screen recordings of both flows are kept in the working copy of this repository (`01-data-generation.mov`,
`02-talk-to-you-data.mov`). They are excluded from git because of their size (~165 MB); the screenshots and the
looping GIF above are the public demo. Play them locally with `ffplay 01-data-generation.mov`.

---

*Personal project — synthetic data generation and conversational analytics, built with Spring AI, React and Langfuse.*
