# Barnsley AI Search -- Progress Summary

## Overview

Today focused on improving the architecture, reducing OpenAI token
usage, and making the Text-to-SQL workflow more deterministic and
scalable.

------------------------------------------------------------------------

# Text-to-SQL Improvements

## Removed the Data Summary node

The largest source of token usage was removed.

**Before**

    SQL
    ↓
    Data Summary (50k+ tokens)

**After**

    SQL
    ↓
    Metadata

This eliminates the expensive summarisation step while preserving
functionality.

------------------------------------------------------------------------

## Reduced Token Usage

The workflow now processes approximately **3,000--4,000 tokens** instead
of **50,000+ tokens** for equivalent requests.

------------------------------------------------------------------------

## Introduced SQL Result Metadata

Instead of sending the full SQL result to the Visual Recommendation
model, the workflow now sends:

-   Row count
-   Column count
-   Column types
-   Dimensions
-   Measures
-   Example values
-   Sample rows

This provides enough context for chart selection while keeping token
usage low.

------------------------------------------------------------------------

## Added `build_visual_json`

Visual Recommendation now only decides:

-   Visual type
-   Fields to use
-   Titles and descriptions

A JavaScript Code node now:

-   Builds the final chart
-   Maps SQL fields
-   Converts numeric strings
-   Normalises dates
-   Produces frontend-ready JSON

------------------------------------------------------------------------

## Simplified the Visual Recommendation Prompt

The AI no longer transforms datasets.

It now simply returns:

-   Chart type
-   Category field
-   Value field
-   Title
-   Description
-   Confidence

All data transformation is handled deterministically in JavaScript.

------------------------------------------------------------------------

# SQL Generation Improvements

## Improved Relative Date Handling

Updated the SQL prompt to prefer `MAX(created_at)` for "latest" or "last
N periods" unless the user explicitly refers to today's date.

------------------------------------------------------------------------

## Improved Comparison Queries

Added prompt guidance so comparisons such as:

-   Mental health vs breastfeeding
-   Schools vs housing

return aggregated comparison rows instead of one row per individual
search phrase.

------------------------------------------------------------------------

## Improved Categorisation Rules

Clarified that:

-   categorisation
-   themes
-   clustering
-   classification

are **not** the same as grouping by `search_request`.

If categorisation cannot be performed reliably using SQL alone, the
workflow now returns `INVALID_QUERY`.

------------------------------------------------------------------------

## Added Result Size Optimisation

The SQL prompt now encourages:

-   Aggregation
-   GROUP BY
-   LIMIT
-   Returning the minimum rows required

instead of unnecessarily large result sets.

------------------------------------------------------------------------

## Protected Against Unrestricted Queries

Added rules preventing summary queries from returning unrestricted raw
records.

Large export-style requests are summarised or rejected rather than
returning thousands of rows.

------------------------------------------------------------------------

# Workflow Improvements

## Added INVALID_QUERY Handling

If SQL generation cannot safely answer a question:

-   The workflow exits early.
-   SQL is not executed.
-   Visual Recommendation is skipped.
-   A helpful text response is returned.

------------------------------------------------------------------------

## Improved User Feedback

Instead of simply returning `INVALID_QUERY`, the workflow now returns
the assumptions explaining why the request could not be answered.

------------------------------------------------------------------------

# Architecture Improvements

The workflow now follows this structure:

    User
        ↓
    Text → SQL
        ↓
    INVALID?
     ├── Yes → Text response
     └── No
            ↓
     Execute SQL
            ↓
     SQL Metadata
            ↓
     Visual Recommendation
            ↓
     build_visual_json
            ↓
     Webhook Response

------------------------------------------------------------------------

# Architectural Discoveries

Identified two distinct categories of user requests.

## 1. SQL Analytics

Examples:

-   How many searches?
-   Searches by month
-   Average response time
-   Top search requests

Handled through Text-to-SQL.

## 2. AI Analysis

Examples:

-   Categorise search requests
-   Identify themes
-   Cluster similar searches
-   Summarise user intent

These require AI reasoning rather than SQL aggregation.

This provides a foundation for introducing an intent router in the
future.

------------------------------------------------------------------------

# Overall Outcome

Today's work focused on improving architecture rather than adding
visible features.

Key achievements include:

-   Major reduction in OpenAI token usage
-   Cleaner separation of AI and deterministic code
-   More reliable SQL generation
-   Better protection against expensive queries
-   Improved handling of unsupported requests
-   A scalable foundation for future analytics and AI workflows

Overall, the Barnsley analytics workflow has shifted from being heavily
AI-driven to a metadata-driven architecture, making it significantly
more efficient, predictable, and maintainable.
