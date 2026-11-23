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
The `*IT` classes additionally call Gemini and need live GCP credentials.

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

- **`docker compose up -d db` currently crash-loops.** The `mysql:lts-oracle` tag now resolves to MySQL 9.x, where
  `LIBRARY` is a reserved word (`information_schema.keywords.reserved = 1`), so `schemas/00_schemas.sql` fails on
  `create schema library;` and the container dies with `ERROR 1064 … near 'library'`. Fix by backtick-quoting the
  identifier in `schemas/00_schemas.sql` and `schemas/02_library_mgm_schema.sql` (`use \`library\`;`), or by pinning
  the image to a MySQL 8.x tag.
- **Liquibase is disabled** (`spring.liquibase.enabled: false`) and `db/changelog/db.changelog-master.yaml` includes a
  `01_company_employee_schema.sql` that does not exist in the repo. Schema provisioning happens through
  `schemas/*.sql`, which the compose file mounts into `/docker-entrypoint-initdb.d`.
- **CORS is contradictory on purpose.** `CorsConfig` allows only `http://localhost:4000`, but both controllers carry
  `@CrossOrigin()`, which is what lets the Vite dev server on `:5173` call the API. Removing the annotation breaks the
  frontend.
- `db/changelog/`, `application-local.yaml` and the datasource passwords are development-only values; the compose
  `CHANGEME` defaults in `docker-compose.yaml` are the same situation.
