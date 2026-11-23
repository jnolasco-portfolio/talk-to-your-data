# Multi-Modal Rendering Strategy: A Backend-Driven Approach

This document outlines the final, streamlined architecture for integrating **Recharts** (charts) and a **Table Component** (tabular data) across a **React frontend** and an intelligent **Spring Boot/LLM backend**.

The core principle is **Delegation of Intelligence**: The Spring Boot/LLM backend decides _what_ to display, and the React frontend decides _how_ to display it.

## 1\. 🌊 End-to-End Workflow

This visualizes the minimum necessary data transfer required for a successful conversational response.

1.  **React Frontend:** User inputs raw question $\rightarrow$ Sends JSON payload to Spring Boot.
2.  **Spring Boot + LLM:** Backend processes the question against the schema $\rightarrow$ LLM returns the optimal `final_sql_query`, `requested_chart_type`, and key mapping fields.
3.  **Spring Boot Service:** Executes the SQL $\rightarrow$ Maps result to DTOs $\rightarrow$ Bundles DTOs with LLM metadata.
4.  **React Frontend:** Receives unified JSON $\rightarrow$ **`ChartRenderer`** component selects the correct visualization component (Recharts or Table) $\rightarrow$ Renders the final interactive result.

---

## 2\. ⚙️ Backend Logic & Actionable JSON Schema

Since your backend already produces the correct SQL, the LLM's goal is to define the final structure of the result set for the frontend.

### A. Backend LLM Output (Simplified Actionable JSON)

The LLM returns the following JSON after analyzing the user's question, chart type, and generating the SQL.

| Key              | Example (Line Chart)                                                                 | Purpose                                                        |
| :--------------- | :----------------------------------------------------------------------------------- | :------------------------------------------------------------- |
| `content_type`   | `"line"`                                                                             | Directs the React component choice: line, bar, pie, table      |
| `category_field` | `"sale_month"`                                                                       | The field used for X-Axis/Grouping/Slices. Not used on tables. |
| `value_field`    | `"total_sales"`                                                                      | The field used for Y-Axis/Bar/Size. Not used on tables.        |
| `sqlQuery`       | `SELECT DATE_TRUNC('month', date) AS sale_month, SUM(amount) AS total_sales FROM...` | The executable SQL statement.                                  |

Note: LLM has to default to table in case not able to guess proper chart type.

#### A1. Backend LLM Output (Simplified Actionable JSON) for Line Charts

```json
{
  "id": "c40ad7f0-1e00-4741-ad31-2c43eed3e7a2",
  "conversationId": "abc-456",
  "question": "Show the trend of revenue by month.",
  "sqlQuery": "[LLM generated SQL query]",
  "metadata": {
    "content_type": "line",
    "category_key": "sale_month",
    "value_key": "total_sales"
  },
  "result": [
    { "sale_month": "2025-01-01T00:00:00.000Z", "total_sales": 750000 },
    { "sale_month": "2025-02-01T00:00:00.000Z", "total_sales": 850000 },
    { "sale_month": "2025-03-01T00:00:00.000Z", "total_sales": 950000 }
  ]
}
```

#### A2. Backend LLM Output (Simplified Actionable JSON) for Bar Charts

```json
{
  "id": "c40ad7f0-1e00-4741-ad31-2c43eed3e7a2",
  "conversationId": "abc-456",
  "question": "Show the trend of revenue by month.",
  "sqlQuery": "[LLM generated SQL query]",
  "metadata": {
    "content_type": "bar",
    "category_key": "department",
    "value_key": "employee_count"
  },
  "result": [
    { "department": "Sales", "employee_count": 150 },
    { "department": "HR", "employee_count": 50 },
    { "department": "Engineering", "employee_count": 300 },
    { "department": "Marketing", "employee_count": 100 }
  ]
}
```

#### A3. Backend LLM Output (Simplified Actionable JSON) for Pie Charts

```json
{
  "id": "c40ad7f0-1e00-4741-ad31-2c43eed3e7a2",
  "conversationId": "abc-456",
  "question": "Show the trend of revenue by month.",
  "sqlQuery": "[LLM generated SQL query]",
  "metadata": {
    "content_type": "pie",
    "category_key": "expense_type",
    "value_key": "total_amount"
  },
  "result": [
    { "expense_type": "Rent", "total_amount": 15000 },
    { "expense_type": "Salaries", "total_amount": 50000 },
    { "expense_type": "Supplies", "total_amount": 5000 }
  ]
}
```

#### A4. Backend LLM Output (Simplified Actionable JSON) for Table Charts

```json
{
  "id": "c40ad7f0-1e00-4741-ad31-2c43eed3e7a2",
  "conversationId": "abc-457",
  "question": "Who has loaned books?",
  "sqlQuery": "[LLM generated SQL query]",
  "metadata": {
    "content_type": "table",
    "category_key": null, // Not used for tables
    "value_key": null // Not used for tables
  },
  "result": [
    { "first_name": "Alice", "last_name": "Smith" },
    { "first_name": "Bob", "last_name": "Johnson" }
  ]
}
```

### B. Consistent Response Structure (Spring Boot to React)

Spring Boot bundles the LLM's metadata with the SQL results (`chart_data`) into a unified JSON object.

| Key        | Description                                                                     |
| :--------- | :------------------------------------------------------------------------------ |
| `metadata` | Contains the LLM's decisions (`content_type`, `category_key`, `value_key`).     |
| `result`   | The clean array of JSON objects (DTOs), ready for direct use by Recharts/Table. |

---

## 3\. 📊 Full Request and Response Examples

The `chart_data` array structure remains the same for all types—a simple array of flat objects—which is why **Recharts** is the ideal choice.

| Chart Type                    | User Request                                                    | `chart_metadata` Output Highlights                                                               | `chart_data` Structure                                                  |
| :---------------------------- | :-------------------------------------------------------------- | :----------------------------------------------------------------------------------------------- | :---------------------------------------------------------------------- |
| **📉 Line Chart (Trend)**     | "Show the **trend** of revenue **by month**."                   | `"content_type": "line"`, `"category_key": "sale_month"`, `"value_key": "total_revenue"`         | `[{"sale_month": "2025-01", "total_revenue": 750000}, ...]`             |
| **📊 Bar Chart (Comparison)** | "What is the **count** of employees **by department**?"         | `"content_type": "bar"`, `"category_key": "department"`, `"value_key": "employee_count"`         | `[{"department": "Sales", "employee_count": 150}, ...]`                 |
| **🥧 Pie Chart (Proportion)** | "Give me the **percentage share** of products **by category**." | `"content_type": "pie"`, `"category_key": "product_category"`, `"value_key": "total_units_sold"` | `[{"product_category": "Electronics", "total_units_sold": 5000}, ...]`  |
| **📄 Table (List/Detail)**    | "List the **top 10** most recent customer orders."              | `"content_type": "table"`, `"column_keys": ["order_id", "customer_name", "amount"]`              | `[{"order_id": 1001, "customer_name": "Alice", "amount": 150.00}, ...]` |

---

## 4\. ⚛️ Frontend Rendering (Recharts & Table Component)

The React **`ChartRenderer`** component is the single point of visualization logic. It reads the metadata and performs the final binding.

### A. Dynamic Component Selection

The logic uses the `requested_chart_type` to switch the rendered component:

```jsx
// Simplified ChartRenderer Logic
function ChartRenderer({ data, metadata }) {
  switch (metadata.requested_chart_type) {
    case 'bar':
      return (
        <BarChartComponent data={data} categoryKey={metadata.category_key} />
      );
    case 'line':
      return (
        <LineChartComponent data={data} categoryKey={metadata.category_key} />
      );
    case 'pie':
      return (
        <PieChartComponent data={data} categoryKey={metadata.category_key} />
      );
    case 'table':
      // Requires the 'column_keys' and 'chart_data'
      return <ResultTable result={data} />;
    default:
      return <div>Unable to display visualization.</div>;
  }
}
```

### B. Recharts Binding (Minimal Processing)

The simplicity of Recharts is demonstrated by the direct key binding:

| Chart Type   | Recharts Component             | Key Bindings                                                                               |
| :----------- | :----------------------------- | :----------------------------------------------------------------------------------------- |
| **Bar/Line** | `<BarChart /> / <LineChart />` | `<XAxis dataKey={metadata.category_key} />` **and** `<Bar dataKey={metadata.value_key} />` |
| **Pie**      | `<PieChart />`                 | `<Pie dataKey={metadata.value_key} nameKey={metadata.category_key} />`                     |

This architecture ensures a **clean, robust, and scalable** conversational interface.
