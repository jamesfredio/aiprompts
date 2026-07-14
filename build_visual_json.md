# build_visual_json

## Purpose

`build_visual_json` is the final transformation step in the Barnsley
Text-to-SQL workflow.

Its responsibility is to convert the original SQL results into the
frontend chart format using the recommendation returned by the AI.

The AI **does not** build the chart data itself. It only decides
**what** chart should be displayed and **which fields** should be used.

------------------------------------------------------------------------

# Workflow

``` text
SQL Result
      │
      ▼
Visual Recommendation (OpenAI)
      │
      ├── Chart type
      ├── Category field
      ├── Value field
      ├── Title
      └── Description
              │
              ▼
      build_visual_json
              │
              ▼
      Frontend-ready JSON
```

------------------------------------------------------------------------

# Responsibilities

## 1. Read the SQL Result

The node reads the original SQL output from `summary_query_execution`.

``` javascript
const rows = $items("summary_query_execution").map(item => item.json);
```

This ensures the original dataset is always used when building charts.

------------------------------------------------------------------------

## 2. Parse the Visual Recommendation

The node supports the different formats returned by the OpenAI Responses
API, including:

-   `output[0].content[0].text`
-   `output`
-   `text`
-   Already-parsed JSON

This makes the workflow resilient to different response structures.

------------------------------------------------------------------------

## 3. Interpret the Recommendation

The AI returns metadata such as:

``` json
{
  "type": "bar_chart",
  "category_field": "topic",
  "value_field": "total_searches",
  "title": "...",
  "description": "..."
}
```

The node extracts these values and prepares the final visual.

------------------------------------------------------------------------

## 4. Build the Final Chart

Depending on the recommendation it creates:

-   Pie Chart
-   Bar Chart
-   Line Chart
-   Text response

using the **original SQL rows**, not AI-generated data.

------------------------------------------------------------------------

# Helper Functions

The node also:

-   Converts numeric strings into numbers.
-   Normalises dates into `YYYY-MM-DD`.
-   Maps SQL field names into the frontend chart schema.

------------------------------------------------------------------------

# Why this Approach?

## Previous Workflow

``` text
SQL Result
      │
      ▼
OpenAI
      │
      ▼
Complete Chart JSON
```

The model had to:

-   Read every SQL row
-   Transform the dataset
-   Rename fields
-   Produce the final JSON

This consumed a large number of tokens for large result sets.

------------------------------------------------------------------------

## Current Workflow

``` text
SQL Result
      │
      ▼
SQL Metadata
      │
      ▼
Visual Recommendation (OpenAI)
      │
      ├── Chart Type
      ├── Fields
      ├── Title
      └── Description
              │
              ▼
      build_visual_json
              │
              ▼
      Final Chart JSON
```

OpenAI now only decides:

-   Which chart to use
-   Which SQL fields represent the X and Y axes
-   Titles and descriptions

The JavaScript code performs all data transformation deterministically.

------------------------------------------------------------------------

# Benefits

-   Significant reduction in OpenAI token usage.
-   More predictable chart generation.
-   Original SQL data is always preserved.
-   No AI-generated data transformations.
-   Easier to maintain and debug.
-   Faster workflow execution.
-   Consistent frontend JSON output.

------------------------------------------------------------------------

# Summary

`build_visual_json` separates **decision-making** from **data
transformation**.

The AI decides **what** to visualise.

The JavaScript node deterministically builds **how** the visual is
represented using the original SQL result.
