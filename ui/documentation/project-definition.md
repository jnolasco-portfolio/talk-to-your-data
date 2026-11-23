# Project Definition: Synthetic Data Generation & Conversational AI - UI

**Overall Status: In Progress**

This document defines the high-level user requirements for a conversational AI application designed for synthetic data generation and natural language-based data querying.

---

## 1. Project Vision

The goal is to create an intuitive web interface that empowers users to:

1.  **Generate Synthetic Data:** Upload a database schema (SQL DDL) and receive high-quality, contextually relevant synthetic data.
2.  **Refine Generated Data:** Iteratively modify and regenerate data for specific tables based on natural language feedback.
3.  **Query Data with Natural Language:** Engage in a conversation with an AI assistant to ask questions about the generated data and receive answers in the form of text, tables, and visualizations.

## 2. High-Level Requirements

The application must provide users with the ability to perform the following key tasks:

- **Schema Ingestion:** The user must be able to upload a `.sql` file containing their database schema to provide context to the system.
- **Instruction-Based Data Generation:** The user must be able to provide natural language instructions to guide the generation of synthetic data (e.g., "generate 100 realistic customer profiles").
- **Data Preview and Validation:** The user must be able to view the generated data for each table in a structured format (e.g., a data grid) to validate its quality.
- **Iterative Data Refinement:** The user must be able to provide follow-up instructions to correct or alter the data for a specific table without starting over.
- **Conversational Querying:** The user must be able to ask questions about their data using a chat-based interface.
- **Multi-Modal Responses:** The system must be able to respond to user queries with various content types, including plain text, formatted tables, and visual chart (pie, bar, line).
- **Conversation History:** The user must be able to view and select from previous conversations.

---

## 3. Implementation Details

For the technical stack, component architecture, development workflows, and other implementation details, please see the [Frontend Implementation Plan](./implementation-plan.md).
