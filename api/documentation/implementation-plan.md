# Project Plan: Synthetic Data Generation & Conversational AI

**Overall Status: In Progress**

This document outlines the implementation plan for building a conversational AI application with synthetic data generation and natural language querying capabilities, as defined in the project overview.

---

## 📝 Phase 1: Synthetic Data Generation Engine

**Status: Complete**

**Goal:** To build the core engine that can parse a SQL schema, generate synthetic data based on it, allow for modifications, and expose this functionality via a REST API.

**Note:** The backend returns generated CSV rows in the `DataGenerationResponse`; the frontend is responsible for packaging (ZIP) and providing downloads. Server-side CSV persistence is not implemented by default and is optional if long-term storage is required.

### Feature 1: Initial Data Generation

**Description:** This feature allows a client to upload a raw SQL schema and instructions, and receive a set of CSV files (one per table) containing synthetic data.

- **Task 1.1: Refactor `SchemaAssistantService` Interface**

  - **File:** `src/main/java/com/example/ai/synthetic_data_generator_ai/service/SchemaAssistantService.java`
  - **Action:** The `generateSyntheticData` method accepts a `conversationId`, `schemaFileName`, an `InputStream` for the schema, and a `DataGenerationRequest` DTO containing the prompt, temperature, row count, and other instructions. It returns a `DataGenerationResponse` containing the generated data.

- **Task 1.2: Update `SchemaAssistantServiceImpl`**

  - **File:** `src/main/java/com/example/ai/synthetic_data_generator_ai/service/SchemaAssistantServiceImpl.java`
  - **Action:** Implement the `generateSyntheticData` method. This involves using the LLM to generate data based on the schema and the parameters in the `DataGenerationRequest`.

- **Task 1.3: Update `SchemaAssistantController`**
  - **File:** `src/main/java/com/example/ai/synthetic_data_generator_ai/controller/SchemaAssistantController.java`
  - **Action:** The `/api/v1/schema-assistant/learn` endpoint is a `multipart/form-data` endpoint that accepts a `file` part for the schema, a `parameters` part for the `LearnSchemaRequest` JSON object, and request parameters for `conversationId` and `schemaFileName`. The `/api/v1/schema-assistant/generate` endpoint accepts a `DataGenerationRequest` in the request body. This endpoint handles both initial generation and table-specific regeneration (via the `tableName` field).

### Feature 2: Data Modification and Refinement

**Description:** This feature allows the user to provide feedback on the generated data for a specific table and have it regenerated. This is handled by the existing `/api/v1/schema-assistant/generate` endpoint, which accepts a `tableName` in the `DataGenerationRequest` to target a specific table for regeneration.

---

## [x] 💬 Phase 2: Conversational Core (Talk-to-your-Data)

**Status: Complete**

**Goal:** To implement the conversational AI that allows users to query the database using natural language.

- **Task 4.1: Create DTOs for Querying:**

  - **Files to Create:** `src/main/java/com/example/ai/synthetic_data_generator_ai/dto/QueryRequest.java`, `src/main/java/com/example/ai/synthetic_data_generator_ai/dto/QueryResponse.java`
  - **Action:** Define DTOs to handle the natural language question and the structured AI response (which may include text, tabular data, etc.). `QueryRequest` will contain the user's question and the schema name. `QueryResponse` will contain the generated SQL query and the query result.

- **Task 4.2: Extend LLM Client for NL-to-SQL:**

  - **File:** `src/main/java/com/example/ai/synthetic_data_generator_ai/llm/LLMSchemaAssistantClient.java`
  - **Action:** Add a new method, `generateSqlQuery(String question, NormalizedSchema schema)`, that takes a user's question and a `NormalizedSchema` to generate an executable SQL query. This will involve creating a new prompt template (`nl-to-sql.st`) that instructs the LLM to convert a natural language question into a SQL query based on the provided schema.

- **Task 4.3: Create a Database Query Service:**

  - **File:** `src/main/java/com/example/ai/synthetic_data_generator_ai/service/DynamicDataService.java`
  - **Action:** Enhance the existing `DynamicDataService` to execute the generated SQL query against the appropriate database. This will involve adding a new method, `executeQuery(String schemaName, String query)`, that takes the schema name and the generated SQL query, and returns the query result as a list of maps.

- **Task 4.4: Implement the Query API Endpoint:**

  - **File:** `src/main/java/com/example/ai/synthetic_data_generator_ai/controller/TalkToYourDataController.java`
  - **Action:** Add a new `@PostMapping("questions")` to the `TalkToYourDataController`. This endpoint will orchestrate the entire process:
    1. Receive the `QueryRequest`.
    2. Call the `SchemaAssistantService` to get the `NormalizedSchema`.
    3. Call the `LLMSchemaAssistantClient` to generate the SQL query.
    4. Call the `DynamicDataService` to execute the query.
    5. Return a `QueryResponse` containing the generated SQL and the query result.

- **Task 4.5: Implement Guardrails:**

  - **Action:** Integrate mechanisms to detect prompt injection, ensure the assistant stays on topic, and handle PII data if applicable. This will be implemented in the `LLMSchemaAssistantClientImpl` by adding a system prompt that defines the assistant's persona and constraints.

- **Task 4.6: Setup Observability with Langfuse:**
  - **Action:** Connect the application to Langfuse for tracing and monitoring. This will involve adding the Langfuse SDK to the `pom.xml` and configuring it in the `application.yaml` file. We will then use the `@trace` annotation on the service methods to trace the execution flow.
