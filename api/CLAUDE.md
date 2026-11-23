# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in the `api/` project.

Spring Boot 3.5.7 / Java 21 / Spring AI 1.0.3 (Vertex AI, Gemini 2.5 Flash). See the [root CLAUDE.md](../CLAUDE.md)
for how this fits with the frontend.

## Commands

```bash
./mvnw spring-boot:run                    # run on :8081
./mvnw -DskipTests package                # build (jar lands in target/)
./mvnw test                               # unit tests (Surefire)
./mvnw verify                             # unit + integration tests (Failsafe)
./mvnw test -Dtest=DynamicDataServiceTest # single unit test class
./mvnw test -Dtest=DynamicDataServiceTest#testExecuteQuery   # single test method
./mvnw verify -Dit.test=SchemaAssistantClientImplIT          # single integration test
docker compose up -d                      # MySQL :3306, phpMyAdmin :8090, Langfuse :3001
```

Required environment variables (no defaults — the context fails to start without them):

- `GOOGLE_CLOUD_PROJECT`, `GOOGLE_CLOUD_LOCATION` — Vertex AI credentials.
- `LANGFUSE_OTEL_AUTH` — base64 of `pk-…:sk-…`, sent as the OTLP `Basic` auth header to Langfuse.

**Tests need MySQL running** (`docker compose up -d db` first). Most test classes are `@SpringBootTest`, so they load
the full application context and fail with "Failed to obtain JDBC Connection" without the database.
`TalkToYourDataControllerTest` is the exception — a `@WebMvcTest` slice with mocked services, so it runs without one.
The `*IT` classes and the two `SchemaAssistant*` classes call Gemini and need live GCP credentials — without
Application Default Credentials they fail with "Your default credentials were not found", which is an environment
problem, not a regression.

**Re-seeding the database**: the `db` service gets an anonymous volume for `/var/lib/mysql` (the mysql image declares
one), and the entrypoint only runs `schemas/*.sql` against an *empty* data directory. `docker compose up -d db` on an
existing volume therefore reuses the old data and silently skips the seed scripts. To re-apply them:

```bash
docker compose up -d --force-recreate --renew-anon-volumes db
```

## Architecture

```text
controller/  SchemaAssistantController (/api/v1/schema-assistant), TalkToYourDataController (/api/v1/data-conversations)
service/     SchemaAssistantServiceImpl, DynamicDataService, DatabaseQueryToolService, DatabaseMetadataToolService
llm/         LLMSchemaAssistantClientImpl — the only class that talks to Gemini
config/      ChatClientConfig, FunctionConfiguration, DataSourceConfiguration, CacheConfig, CorsConfig
dto/         Lombok DTO classes mirroring the wire contract (mirror these in ui/src/schemas/)
```

**Prompts live in `src/main/resources/prompts/*.st`**, not in Java. `LLMSchemaAssistantClientImpl` loads them with
`@Value("classpath:prompts/…")`; edit those files to change model behaviour. `schema-assistant-system.st` is the
system prompt for the schema client, `natural-lenguage-to-sql.st` / `nl-to-sql.st` drive question answering, and
`convert-schema-to-json.st` + `generate-synthetic-data.st` drive learning and generation.

### Schema routing: the bean-name convention

There is no schema registry. A request's `schemaName` is turned into a Spring bean name by concatenation —
`"library"` → `libraryDataSource` — and looked up in the injected `Map<String, DataSource>`
(`DynamicDataService`). Adding a database therefore means **three** edits that must stay in sync:

1. a `@Bean` in `DataSourceConfiguration` (e.g. `warehouseDataSource`),
2. an `app.datasource.warehouse` block in `application-local.yaml`,
3. the schema name in the UI, which sends it verbatim.

`DynamicDataService` reads table/column/PK/FK/index metadata with JDBC `DatabaseMetaData` and executes generated SQL
through `JdbcClient`.

### LLM tool calling

`FunctionConfiguration` publishes two `Function` beans with `@Description` annotations — they are the tool definitions
offered to the model:

- `databaseQueryToolService` — runs a SQL string against a schema.
- `databaseMetadataToolService` — returns the full schema for a schema name.

### Caching and observability

`LLMSchemaAssistantClientImpl.learnSchema` is `@Cacheable` on `(conversationId, schemaName, userPrompt)`; the `test`
profile disables caching (`application-test.yaml`). Every LLM call is observed by Micrometer and exported over OTLP to
Langfuse (`management.tracing.exporters.otlphttp/langfuse`).

## Known issues

- **The MySQL image is pinned to `mysql:8.4` on purpose.** The `mysql:lts-oracle` tag floats: Docker Hub re-pointed it
  at MySQL 9.7 in November 2026, where `LIBRARY` is a reserved word (`information_schema.keywords.reserved = 1`) and
  the seed scripts fail with `ERROR 1064 … near 'library'`. The `library` identifier is now backtick-quoted in
  `schemas/00_schemas.sql` and `schemas/02_library_mgm_schema.sql`, so the seed works on 8.x and 9.x alike — but the
  image stays on the 8.4 LTS line, which is what the app is developed against.
- **Liquibase is disabled** (`spring.liquibase.enabled: false`) and `db/changelog/db.changelog-master.yaml` includes a
  `01_company_employee_schema.sql` that does not exist in the repo. Schema provisioning happens through
  `schemas/*.sql`, which the compose file mounts into `/docker-entrypoint-initdb.d` — `00_schemas.sql` creates the
  `aiuser` role and the three schemas, `01`–`03` create the tables, `05_library_authors_data.sql` seeds 100 sample
  authors that `DynamicDataServiceTest` and `DatabaseQueryToolServiceTest` assert on (row 1 is Stephen King,
  1947-09-21).
- **CORS is contradictory on purpose.** `CorsConfig` allows only `http://localhost:4000`, but both controllers carry
  `@CrossOrigin()`, which is what lets the Vite dev server on `:5173` call the API. Removing the annotation breaks the
  frontend.
- `db/changelog/`, `application-local.yaml` and the datasource passwords are development-only values; the compose
  `CHANGEME` defaults in `docker-compose.yaml` are the same situation.
