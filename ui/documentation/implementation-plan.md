# Frontend Implementation Plan: Data Assistant

This document outlines the technical implementation strategy for the Data Assistant frontend application. For high-level project goals and user requirements, please see the [Project Definition](./project-definition.md).

## 1. Technical Stack Overview

| Category              | Technology                                | Rationale                                                                                          |
| :-------------------- | :---------------------------------------- | :------------------------------------------------------------------------------------------------- |
| **Core Framework**    | **React (v19.x)**                         | Industry standard for building reactive user interfaces.                                           |
| **Language**          | **TypeScript (v5.x)**                     | Provides static typing for increased code quality, error prevention, and better tooling.           |
| **Styling**           | **Pico.css** + Custom Styling (index.css) | A custom stylesheet is used for styling the application.                                           |
| **Routing**           | **React Router DOM (v6)**                 | Declarative navigation for a Single Page Application (SPA).                                        |
| **Form Management**   | **React Hook Form**                       | High-performance library for handling form state and validation, used in the `DataGenerationForm`. |
| **Schema Validation** | **Zod**                                   | Declarative, type-safe schema validation.                                                          |
| **UI State**          | **React `useState`/`useReducer`**         | For managing local component and screen-level state.                                               |
| **Server State**      | **React Query**                           | For fetching, caching, and managing asynchronous data from the API.                                |
| **Chart Library**     | **Recharts**                              | For fastest path from JSON to a rendered chart with minimal intermediate processing code.          |

## 2. Component Architecture

The application will use a clear, modular architecture following a **Layout > Screen > Component** hierarchy.

### 2.1. Layout Layer (Persistent Structure)

- **`AppLayout.tsx`**: The top-level component defining the two-column grid (Sidebar and Main Content). It will contain the `Sidebar` and receive the currently active `Screen` component as its children (via React Router).
- **`Sidebar.tsx`**: Contains the persistent application title (`Data Assistant`) and the primary navigation links (`Data Generation` and `Talk to your data`), using `react-router-dom`'s `Link` and `useLocation` for active state tracking.

### 2.2. Screen Layer (Route Destinations)

- **`DataGenerationScreen.tsx` (Phase 1)**: Manages the state for synthetic data generation and holds the overall logic for the generation workflow.
- **`ChatScreen.tsx` (Phase 2)**: Manages the conversation state, input/output, and complex content rendering for the natural language data querying feature.

### 2.3. Component Layer (Reusability & Logic)

- **Phase 1 Components:**
  - `DataGenerationForm.tsx`: Handles prompt input, parameter settings (Temperature, Max Tokens), and DDL schema upload.
  - `DataPreview.tsx`: Renders the generated data in a tabular format and can include the `QuickInstructionForm`.
  - `QuickInstructionForm.tsx`: A simple form for sending textual feedback to modify data.
- **Phase 2 Components:**
  - `QuestionForm.tsx`: Handles user input for asking questions.
  - `ConversationSelector.tsx`: Allows users to select from existing conversations or start a new one.
  - `ChatHistoryItem.tsx`: Renders a single item in the chat history, including the user's question, the generated SQL query, and the result table.

## 3. Detailed Workflow 1: Synthetic Data Generation

This workflow outlines the step-by-step process for a user generating synthetic data.

1.  **Initial State:** The user is on the "Data Generation" screen. The `DataGenerationForm` is visible, and the `DataPreview` area is empty.
2.  **Schema Upload:** The user clicks the "Upload DDL" button and selects a `.sql` file.
    - The `useLearDatabase` hook is triggered, sending the file content to the `/learn-database` API endpoint.
    - While the request is in flight, a loading overlay is displayed over the main content area.
3.  **Schema Learning Confirmation:** Upon a successful API response, the loading overlay is removed. A success notification is displayed, and the names of the learned tables are populated in a dropdown or list within the `DataGenerationForm`.
4.  **Instruction Input:** The user types a natural language instruction into the main text area (e.g., "Generate 50 realistic customer profiles based in the United States").
5.  **Data Generation:** The user submits the form.
    - To ensure data integrity and respect foreign key constraints, the tables are first topologically sorted based on their dependencies.
    - The `useGenerateData` hook is then called sequentially for each table, sending the instruction and any selected parameters to the `/generate-data` API endpoint.
    - A loading overlay is again displayed, showing the progress of the generation.
6.  **Data Preview:** Upon a successful response, the `DataPreview` component is rendered.
    - It displays the returned data, with each table's data presented in a separate tab.
    - Each tab contains a paginated, sortable data grid.
7.  **Data Refinement (Loop):**
    - Within a table's tab in the `DataPreview`, the user can type a refinement instruction into the `QuickInstructionForm` (e.g., "For the 'users' table, add more varied email domains").
    - Submitting this form triggers the `useGenerateData` hook again, but with the new, targeted instruction. The process returns to step 6.

## 4. Detailed Workflow 2: Talk to Your Data (Conversational Core)

This workflow describes the user's interaction with the conversational data querying interface.

1.  **Initial State:** The user navigates to the "Talk to your Data" screen. The `ChatHistory` panel is on the left, and the main `ChatScreen` is on the right.
2.  **Conversation Management:**
    - The `ConversationSelector` fetches and displays a list of past conversations.
    - The user can select a past conversation to load its message history or click "New Chat" to start fresh.
    - Conversation history is persisted to the browser's `localStorage` to ensure that chats are not lost between sessions.
3.  **Asking a Question:** The user types a question into the `QuestionForm` at the bottom of the `ChatScreen` (e.g., "How many orders were placed in the last month?").
4.  **API Request:** On submission, the question is sent to the `/questions` API endpoint. The new user message is immediately appended to the chat history in a "pending" state.
5.  **Response Handling:**
    - A loading indicator is shown while waiting for the API.
    - When the response is received, the "pending" message is updated to a "sent" state, and the AI's response is appended to the history.
6.  **Multi-Modal Rendering:** The `ChatHistoryItem` component inspects the `content_type` of the AI's message and renders it accordingly. This backend-driven rendering strategy is detailed in the [Multi-Modal Rendering Strategy documentation](./documentation/multi-modal-rendering-strategy.md). The supported types are:
    - **`table`**: A data grid.
    - **`chart_pie`**: A pie chart.
    - **`chart_bar`**: A bar chart.
    - **`chart_line`** A line chart.
7.  **Continuous Conversation (Loop):** The user can continue asking follow-up questions, repeating the process from step 3.

## 5. API Contract

The detailed API endpoints, request/response payloads, and example `cURL` commands are documented in `api-contract.md`. This document serves as the source of truth for the frontend-backend communication interface.

## 6. Development Workflow & Code Quality

To ensure a high-quality and maintainable codebase, the team will adhere to the following practices:

- **Linting and Formatting:** All code will be validated against the established ESLint rules (`npm run lint`) to enforce a consistent style and catch common errors.
- **TypeScript Best Practices:** We will leverage TypeScript's static typing to its full potential, avoiding `any` types where possible and defining clear interfaces for props, state, and API payloads.
- **AI-Assisted Development:** Developers are encouraged to use AI coding assistants (e.g., Gemini Code Assist) to:
  - Generate boilerplate code for components and functions.
  - Suggest refactoring opportunities for improved readability and performance.
  - Identify potential bugs or edge cases.
  - Get suggestions on best practices and alternative implementations.
- **Critical Review:** All suggestions from AI assistants must be critically reviewed by the developer to ensure they align with the project's architecture, goals, and security standards before being committed.
