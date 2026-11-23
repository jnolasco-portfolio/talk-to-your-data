# Project Overview

## Overview

The goal of this practice is to implement a conversational AI application with two primary functionalities: synthetic data generation and natural language data querying. This task is broken down into 3 distinct phases:

• Phase 1 involves developing a core data generation engine. This engine must interpret the provided SQL schemas, identifying tables, columns, data types, and constraints. A data generation module will then use this parsed information to create realistic synthetic data, respecting all defined constraints, especially foreign keys, to ensure data integrity. The system should be capable of generating a configurable amount of data, for instance, a thousand rows per table.

• Phases 2 and 3 focus on implementing the Conversational Core module, which allows users to query SQL data using natural language (talk-to-your-data) and present results in the form of text, tables, and plots.

Functional Requirements:

Phase 1: Synthetic Data Generation

• The system should generate consistent and valid data for the provided DDL schema (up to 5-7 Tables) and instructions [data types, null values, date and time formats, primary and foreign keys, etc].

• The system should allow a user to modify the data through textual feedback from the user.

• Generated data can be downloaded as csv and stored in the system so that later it is accessible in ‘Talk to you data’ tab. The UI will be responsible for packaging the CSV files into a ZIP archive for download.

UI requirements:

1. Sidebar for main tabs: Data Generation, Talk to your data.

2. Data Generation tab:

User can upload as a file with DDL schema (.sql, .txt or .ddl file)

User can add text instructions (prompt) for the data in a text box

User can set additional generation parameters, such as temperature

Generation happens after user clicks “Generate” button

After data is generated, user can check the preview for each of the generated tables

User can apply changes for each of the tables by entering the prompt and clicking Submit button

Phase 2: Chat with Your Data

The system shall provide a conversational interface that allows users to interact with the data using natural language text input.

1. The system should display the conversation history and stream system responses.

2. The system should automatically generate and execute relevant SQL queries against the underlying data source(s).

SQL Joins and Aggregation functions are supported

Both the source query and tabular output are provided in the output

[optional] Queries are modifiable from the UI (?)

3. The system should generate data visualisations using the Python Seaborn library (or similar) and provide the results within the conversational interface.

Implement basic guardrails:

• Detect prompt injection / jailbreaks attempts.

• Make sure your AI assistant stays on topic.

• Optional (if relevant for your schema): PII data tokenization (masking) for user queries.

Observability

• Setup Langfuse and connect it to your application (for tracing):

You can also try to setup some alerts for jailbreak attempts and online evals

The "Phase 1: Synthetic Data Generation" workflow is focused on taking a schema and instructions from the user, generating realistic synthetic data, allowing the user to refine it, and finally storing the result.

The process is divided into three main stages: **Input & Configuration**, **Generation**, and **Review & Refinement**.

---

## ⚙️ Phase 1: Synthetic Data Generation Workflow

[cite_start]The workflow is primarily carried out in the **"Data Generation" tab**[cite: 10], as visually represented in Image 1.

### 1. Input & Configuration

This stage collects all necessary inputs for the generation engine:

| Step                    | User Action (UI Element)                                                                                                                         | System Requirement                                                                                                                     |
| :---------------------- | :----------------------------------------------------------------------------------------------------------------------------------------------- | :------------------------------------------------------------------------------------------------------------------------------------- |
| **1. Provide Schema**   | [cite_start]User clicks "Upload DDL Schema" to upload a file (**.sql**, **.txt**, or **.ddl** file)[cite: 11].                                   | [cite_start]The engine must **interpret the provided SQL schemas**, identifying tables, columns, data types, and constraints[cite: 3]. |
| **2. Add Instructions** | [cite_start]User enters the desired constraints, themes, or rules into the **Prompt** text box[cite: 11].                                        | [cite_start]The system will generate consistent and valid data based on these instructions[cite: 7].                                   |
| **3. Set Parameters**   | [cite_start]User sets **additional generation parameters** like **Temperature** and **Max Tokens** (seen under "Advanced Parameters")[cite: 11]. | [cite_start]This configures the engine's output style and limits the response size[cite: 11].                                          |

---

### 2. Data Generation

| Step                      | Action                                                       | Outcome                                                                                                                                                                                                                                                                 |
| :------------------------ | :----------------------------------------------------------- | :---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **4. Trigger Generation** | [cite_start]User clicks the **"Generate"** button[cite: 11]. | [cite_start]A core data generation engine uses the parsed schema and instructions to **create realistic synthetic data**[cite: 4, 7]. [cite_start]The system should be capable of generating a configurable amount of data, such as a thousand rows per table[cite: 5]. |

---

### 3. Review & Refinement

This stage involves checking the output and making iterative modifications.

| Step                      | User Action (UI Element)                                                                                                                                                                            | System Requirement                                                                                                                                                                                                                                                                                                                                                                                |
| :------------------------ | :-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| **5. Check Preview**      | [cite_start]The user uses the **Data Preview** area and a dropdown (or similar selector) to **check the preview for each of the generated tables** (up to 5-7 tables)[cite: 7, 11].                 | [cite_start]The preview displays the consistent and valid data generated, respecting all defined constraints[cite: 7].                                                                                                                                                                                                                                                                            |
| **6. Modify Data (Loop)** | [cite_start]The user can **apply changes** for the _currently selected_ table by **entering a prompt** into the **"Enter quick instructions..."** input field and clicking **Submit**[cite: 8, 11]. | [cite_start]The system should **allow a user to modify the data through textual feedback**[cite: 8]. The preview must update to show the revised data.                                                                                                                                                                                                                                            |
| **7. Finalize & Store**   | (Implicit Action)                                                                                                                                                                                   | [cite_start]The generated data is returned to the frontend as CSV rows. The frontend is responsible for packaging these files into a **ZIP archive** for the user and providing the download. The backend provides schema metadata and synthetic data payloads but does not persist CSV files by default. The frontend may persist or upload generated datasets if long-term storage is required. |

[cite_start]This workflow outlines the process for "Phase 2: Chat with Your Data" (also known as "Talk to your data"), which is a conversational interface designed to query the stored data using natural language[cite: 11]. This phase relies on the data generated and stored during Phase 1.

The process is an iterative conversational loop:

---

## 💬 Phase 2: Chat with Your Data Workflow

### 1. Initiation and Input

| Step                      | User Action (UI Element)                                                                                                                                                                     | System Requirement                                                                                               |
| :------------------------ | :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | :--------------------------------------------------------------------------------------------------------------- |
| **1. Start Conversation** | User navigates to the **"Talk to your data" tab** (via the sidebar).                                                                                                                         | [cite_start]The system should display the **conversation history** (if any) in the main panel[cite: 12].         |
| **2. Pose Query**         | [cite_start]User enters a natural language question into the text input box at the bottom of the screen (e.g., "What are the top-selling dishes by week over the last 3 months?")[cite: 11]. | [cite_start]The system must be connected to the underlying data source (the stored data from Phase 1)[cite: 13]. |

### 2. Processing and Execution

| Step                                        | System Action                                                                                                                                                                         | Outcome                                                                                                                                |
| :------------------------------------------ | :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | :------------------------------------------------------------------------------------------------------------------------------------- |
| **3. Natural Language (NL) Interpretation** | The Conversational Core module interprets the user's natural language request.                                                                                                        | [cite_start]The system implements guardrails to **detect prompt injection** and ensures the AI assistant **stays on topic**[cite: 15]. |
| **4. Query Generation**                     | [cite_start]The system automatically **generates relevant SQL queries** based on the prompt, utilizing features like **SQL Joins and Aggregation functions** if needed[cite: 13, 14]. | The generated SQL is ready for execution against the database.                                                                         |
| **5. Query Execution**                      | [cite_start]The system **executes the generated SQL query** against the stored data source(s)[cite: 13].                                                                              | Results are retrieved from the database.                                                                                               |

### 3. Output and Visualization

[cite_start]The system streams the response to the user, displaying the original query, the tabular output, or a visualization[cite: 12].

Refer to **multi-modal-rendering-strategy.md** document for more details. |

### 4. Iteration

[cite_start]The system continues to **display the conversation history** and returns to Step 2, allowing the user to enter a follow-up question that builds on the prior context (e.g., "Great. Which of those dishes are available in London branch?")[cite: 12].
