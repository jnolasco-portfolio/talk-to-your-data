# API Contract: Data Assistant Backend

This document outlines the API endpoints and data structures for the communication between the React frontend and the backend service. The contract uses `zod` for schema definitions to ensure type safety.

> **Note:** This document serves as a working agreement. The provided examples are for guidance and should be adapted as needed.

**Base URL:** `https://2c7ecc91-0331-4a18-b933-5d95ba55d77d.mock.pstmn.io/api/v1/schema-assistant`

---

## 1. Schema Upload & Initialization

This endpoint uploads a user's SQL DDL file. The backend parses it into a structured JSON format.

- **Endpoint:** `/learn`
- **Method:** `POST`
- **Content-Type:** `multipart/form-data`

### Request

The request is a multipart form containing the DDL file and associated metadata.

**cURL Example:**

```bash
curl --location 'https://2c7ecc91-0331-4a18-b933-5d95ba55d77d.mock.pstmn.io/api/v1/schema-assistant/learn' \
--form 'file=@"/home/jnolasco/code/grid/gridu/prompt-engineering-ai-apps/ai-synthetic-data-generator-api/src/main/resources/data/sample-schema.sql"' \
--form 'conversationId="12345"' \
--form 'schemaFileName="company_schema.sql"' \
--form 'parameters="{\"prompt\":\"Learn this schema\", \"temperature\": 0.2, \"maxRows\": 20}"'
```

### Response

The response is a JSON object representing the parsed schema.

**Example JSON Response:**

```json
{
  "server": "mysql",
  "schemaName": "library",
  "tables": [
    {
      "name": "Authors",
      "comment": "Stores information about authors.",
      "columns": [
        {
          "name": "author_id",
          "type": "INT",
          "nullable": false,
          "defaultValue": null,
          "autoIncrement": true,
          "comment": "Unique identifier for the author."
        },
        {
          "name": "first_name",
          "type": "VARCHAR(100)",
          "nullable": false,
          "defaultValue": null,
          "autoIncrement": false,
          "comment": "First name of the author."
        },
        {
          "name": "last_name",
          "type": "VARCHAR(100)",
          "nullable": false,
          "defaultValue": null,
          "autoIncrement": false,
          "comment": "Last name of the author."
        },
        {
          "name": "birth_date",
          "type": "DATE",
          "nullable": true,
          "defaultValue": null,
          "autoIncrement": false,
          "comment": "Date of birth of the author."
        },
        {
          "name": "death_date",
          "type": "DATE",
          "nullable": true,
          "defaultValue": null,
          "autoIncrement": false,
          "comment": "Date of death of the author (if applicable)."
        },
        {
          "name": "nationality",
          "type": "VARCHAR(100)",
          "nullable": true,
          "defaultValue": null,
          "autoIncrement": false,
          "comment": "Nationality of the author."
        },
        {
          "name": "biography",
          "type": "TEXT",
          "nullable": true,
          "defaultValue": null,
          "autoIncrement": false,
          "comment": "A brief biography of the author."
        }
      ],
      "primaryKey": "author_id",
      "foreignKeys": [],
      "indexes": [
        {
          "name": "PK_Authors_author_id",
          "columns": ["author_id"],
          "unique": true,
          "type": "PRIMARY KEY"
        }
      ]
    },
    {
      "name": "Publishers",
      "comment": "Stores information about book publishers.",
      "columns": [
        {
          "name": "publisher_id",
          "type": "INT",
          "nullable": false,
          "defaultValue": null,
          "autoIncrement": true,
          "comment": "Unique identifier for the publisher."
        },
        {
          "name": "name",
          "type": "VARCHAR(255)",
          "nullable": false,
          "defaultValue": null,
          "autoIncrement": false,
          "comment": "Name of the publisher."
        },
        {
          "name": "address",
          "type": "VARCHAR(255)",
          "nullable": true,
          "defaultValue": null,
          "autoIncrement": false,
          "comment": "Physical address of the publisher."
        },
        {
          "name": "city",
          "type": "VARCHAR(100)",
          "nullable": true,
          "defaultValue": null,
          "autoIncrement": false,
          "comment": "City where the publisher is located."
        },
        {
          "name": "state",
          "type": "VARCHAR(50)",
          "nullable": true,
          "defaultValue": null,
          "autoIncrement": false,
          "comment": "State where the publisher is located."
        },
        {
          "name": "zip_code",
          "type": "VARCHAR(10)",
          "nullable": true,
          "defaultValue": null,
          "autoIncrement": false,
          "comment": "Zip code of the publisher's address."
        },
        {
          "name": "country",
          "type": "VARCHAR(100)",
          "nullable": true,
          "defaultValue": null,
          "autoIncrement": false,
          "comment": "Country where the publisher is located."
        },
        {
          "name": "contact_email",
          "type": "VARCHAR(255)",
          "nullable": true,
          "defaultValue": null,
          "autoIncrement": false,
          "comment": "Contact email for the publisher."
        },
        {
          "name": "contact_phone",
          "type": "VARCHAR(20)",
          "nullable": true,
          "defaultValue": null,
          "autoIncrement": false,
          "comment": "Contact phone number for the publisher."
        }
      ],
      "primaryKey": "publisher_id",
      "foreignKeys": [],
      "indexes": [
        {
          "name": "PK_Publishers_publisher_id",
          "columns": ["publisher_id"],
          "unique": true,
          "type": "PRIMARY KEY"
        }
      ]
    }
  ]
}
```

---

## 2. Data Generation / Refinement

This endpoint generates synthetic data for a specific table. It can be used for initial generation or for refining data based on new instructions.

- **Endpoint:** `/generate`
- **Method:** `POST`
- **Content-Type:** `application/json`

> **Workflow Clarification:** To ensure data integrity and respect foreign key relationships, the frontend must call this `/generate` endpoint **sequentially** for each table in the database schema. The order of calls should follow the topological sort of the dependency graph of the tables.

### Request (`generateDataRequest.ts`)

The request body contains the full schema, the target table, and generation instructions.

**cURL Example:**

```bash
curl --location 'https://2c7ecc91-0331-4a18-b933-5d95ba55d77d.mock.pstmn.io/api/v1/schema-assistant/generate' \
--header 'Content-Type: application/json' \
--data '{
    "conversationId": "12345",
    "tableName": "Authors",
    "instructions": "Generate data in spanish lenguage",
    "maxRows": 10,
    "schema": { /* full schema from /learn response */ }
}'
```

**Example JSON Response:**

```json
{
  "tableName": "Authors",
  "data": [
    "Gabriel|García Márquez|1927-03-06|2014-04-17|Colombiano|Novelista y cuentista colombiano...",
    "Isabel|Allende|1942-08-02|\\N|Chilena|Escritora chilena...",
    "Jorge Luis|Borges|1899-08-24|1986-06-14|Argentino|Escritor de cuentos...",
    "Laura|Esquivel|1950-09-30|\\N|Mexicana|Escritora mexicana...",
    "Mario|Vargas Llosa|1936-03-28|\\N|Peruano|Escritor peruano...",
    "Federico|García Lorca|1898-06-05|1936-08-19|Español|Poeta, dramaturgo y prosista español...",
    "Clarice|Lispector|1920-12-10|1977-12-09|Brasileña|Escritora brasileña...",
    "Octavio|Paz|1914-03-31|1998-04-19|Mexicano|Poeta, ensayista y diplomático mexicano...",
    "Elena|Poniatowska|1932-05-19|\\N|Mexicana|Periodista y escritora mexicana...",
    "Miguel|de Cervantes|1547-09-29|1616-04-22|Español|Novelista, poeta y dramaturgo español..."
  ]
}
```

**JSON Body:**

| Field       | Type     | Description                                                                                                                                                                   |
| :---------- | :------- | :---------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `tableName` | String   | The name of the table for which data was generated.                                                                                                                           |
| `data`      | String[] | An array of strings, where each string is a comma-separated list of values representing a single data row. The order of values corresponds to the column order in the schema. |

---

## 3. Talk to Your Data (Query)

This endpoint sends a natural language question about the data and returns a SQL query and the corresponding results from the database.

- **Endpoint:** `/query`
- **Method:** `POST`
- **Content-Type:** `application/json`

### Request

The request body contains the user's question, a conversation ID, and the schema context.

**cURL Example:**

```bash
curl --location 'http://localhost:8081/api/v1/schema-assistant/questions' \
--header 'Content-Type: application/json' \
--data '{
    "question": "How many authors are there from the United States?",
    "conversationId": "xyz-789",
    "schemaName": "library"
}'
```

**JSON Body:**

| Field            | Type   | Description                                                                                          |
| :--------------- | :----- | :--------------------------------------------------------------------------------------------------- |
| `question`       | String | The natural language question from the user.                                                         |
| `conversationId` | String | An identifier to maintain context across multiple related queries.                                   |
| `schemaName`     | String | The schema name. This parameter the necessary context for the LLM to generate the correct SQL query. |

### Response

The response contains the generated SQL and the result of its execution.

| Field            | Type                                                         | Description                                                                                                                              |
| :--------------- | :----------------------------------------------------------- | :--------------------------------------------------------------------------------------------------------------------------------------- |
| `id`             | String                                                       | A unique identifier for this specific question-answer interaction.                                                                       |
| `conversationId` | String                                                       | An identifier to maintain context across multiple related queries.                                                                       |
| `question`       | String                                                       | The original natural language question from the user.                                                                                    |
| `sqlQuery`       | String                                                       | The SQL query generated by the AI to answer the question.                                                                                |
| `metadata`       | Object                                                       | An object containing metadata for rendering the results, especially for charts.                                                          |
| `├─ content_type` | String                                                       | The suggested visualization type (`"bar"`, `"table"`, etc.).                                                                             |
| `├─ category_key` | String / Null                                                | The key in the `result` objects to be used for the chart's category axis (e.g., x-axis in a bar chart).                                    |
| `└─ value_key`    | String / Null                                                | The key in the `result` objects to be used for the chart's value axis (e.g., y-axis in a bar chart).                                       |
| `result`         | Array of Objects                                             | An array of objects representing the rows returned by the `sqlQuery`. Each object's keys correspond to the columns in the SQL `SELECT`. |

**Example bar chart JSON Response:**

```json
{
  "id": "5a152d78-bd31-4b38-8add-a3c6cb7358b1",
  "conversationId": "xyz-789",
  "question": "Shome how many authors by country are",
  "sqlQuery": "SELECT nationality, COUNT(author_id) AS author_count FROM Authors WHERE nationality IS NOT NULL GROUP BY nationality ORDER BY author_count DESC",
  "metadata": {
    "content_type": "bar",
    "category_key": "nationality",
    "value_key": "author_count"
  },
  "result": [
    {
      "nationality": "American",
      "author_count": 29
    },
    {
      "nationality": "British",
      "author_count": 24
    },
    {
      "nationality": "Japanese",
      "author_count": 7
    },
    {
      "nationality": "Ancient Greek",
      "author_count": 4
    }
  ]
}
```

**Example table chart JSON Response:**

```json
{
  "id": "a1e023f3-79a7-457d-83fc-a1834b519470",
  "conversationId": "xyz-789",
  "question": "Who has loaned books?",
  "sqlQuery": "SELECT DISTINCT T1.first_name, T1.last_name FROM Library_Members AS T1 JOIN Book_Loans AS T2 ON T1.member_id = T2.member_id",
  "metadata": {
    "content_type": "table",
    "category_key": null,
    "value_key": null
  },
  "result": [
    {
      "first_name": "Alice",
      "last_name": "Smith"
    },
    {
      "first_name": "Bob",
      "last_name": "Johnson"
    }
  ]
}
```
