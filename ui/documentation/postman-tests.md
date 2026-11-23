# Postman API Test Collection

Here are the cURL commands for testing the API. You can import these directly into Postman to create a test collection.

### How to Import into Postman

1.  Open Postman.
2.  Click the **Import** button in the top-left corner.
3.  In the dialog that appears, select the **Raw Text** tab.
4.  Copy and paste the entire cURL command from below into the text area.
5.  Click **Continue** and then **Import**.

---

### **1. Learn Schema**

This request uploads a SQL schema file, along with generation parameters, and returns the learned schema.

**cURL Command:**

```bash
curl --location 'https://2c7ecc91-0331-4a18-b933-5d95ba55d77d.mock.pstmn.io/api/v1/schema-assistant/learn' \
--form 'file=@"/home/jnolasco/code/grid/gridu/prompt-engineering-ai-apps/ai-synthetic-data-generator-api/src/main/resources/data/sample-schema.sql"' \
--form 'conversationId="12345"' \
--form 'schemaFileName="company_schema.sql"' \
--form 'parameters="{\"prompt\":\"Learn this schema\", \"temperature\": 0.2, \"maxRows\": 20}"'
```

**IMPORTANT:**

- Before importing, you **must** replace `/home/jnolasco/code/grid/gridu/prompt-engineering-ai-apps/ai-synthetic-data-generator-api/schemas/01_company_employee_schema.sql` with the **absolute path** to a schema file on your local machine.
- The `parameters` are sent as a JSON string.

---

### **2. Generate Synthetic Data**

This request generates synthetic data for a specific table based on the learned schema and a prompt.

**cURL Command:**

```bash
curl --location 'https://2c7ecc91-0331-4a18-b933-5d95ba55d77d.mock.pstmn.io/api/v1/schema-assistant/generate' \
--header 'Content-Type: application/json' \
--data
'{
        "server": "mysql",
        "schemaName": "library",
        "tableName": "Authors",
        "instructions": "Generate 10 authors from Spain.",
        "maxRows": 10,
        "temperature": 0.2,
        "tables": [
            {
                "name": "Authors",
                "columns": [
                    {
                        "name": "author_id",
                        "type": "INT",
                        "nullable": false,
                        "defaultValue": null,
                        "autoIncrement": true
                    },
                    {
                        "name": "first_name",
                        "type": "VARCHAR(100)",
                        "nullable": false,
                        "defaultValue": null,
                        "autoIncrement": false
                    },
                    {
                        "name": "last_name",
                        "type": "VARCHAR(100)",
                        "nullable": false,
                        "defaultValue": null,
                        "autoIncrement": false
                    },
                    {
                        "name": "birth_date",
                        "type": "DATE",
                        "nullable": true,
                        "defaultValue": null,
                        "autoIncrement": false
                    },
                    {
                        "name": "death_date",
                        "type": "DATE",
                        "nullable": true,
                        "defaultValue": null,
                        "autoIncrement": false
                    },
                    {
                        "name": "nationality",
                        "type": "VARCHAR(100)",
                        "nullable": true,
                        "defaultValue": null,
                        "autoIncrement": false
                    },
                    {
                        "name": "biography",
                        "type": "TEXT",
                        "nullable": true,
                        "defaultValue": null,
                        "autoIncrement": false
                    }
                ],
                "primaryKey": "author_id",
                "foreignKeys": [],
                "indexes": [
                    {
                        "name": "PK_Authors_author_id",
                        "columns": [
                            "author_id"
                        ],
                        "unique": true,
                        "type": "PRIMARY KEY"
                    }
                ]
            }
        ]
    },

}'
```

**IMPORTANT:**

- The `schema` object in the request body should be the `NormalizedSchema` object returned from the `/learn` endpoint.
- The `tableName` should be one of the tables from the schema.
- The `instructions` are used to guide the data generation.
- `maxRows` is the maximum number of rows to generate.

---

### **3. Talk to Your Data (Query)**

This request sends a natural language question about the data and returns a SQL query and the corresponding results from the database.

**cURL Command:**

```bash
curl --location 'http://localhost:8081/api/v1/data-conversations' \
--header 'Content-Type: application/json' \
--data '{
    "question": "How many authors are there from the United States?",
    "conversationId": "xyz-789",
    "schemaName": "library"
}'
```

**IMPORTANT:**

- The `schema` object in the request body should be the `NormalizedSchema` object returned from the `/learn` endpoint. This provides the necessary context for the LLM to generate the correct SQL query.
- The `question` is the natural language query you want to ask about your data.
- The `conversationId` can be used to maintain context across multiple related queries.
