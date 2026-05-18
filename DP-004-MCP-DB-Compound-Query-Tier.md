# DP-004: Database-Backed Model Context Protocol Server with Compound Query Tier

**Defensive Publication**  
**Technology Outlaws LLC**  
**Publication Date:** May 17, 2026  
**Inventor:** Jason Tesso  
**Related Provisional Application:** U.S. Provisional App. No. 64/013,702 (filed March 23, 2026)  

---

## 1. Field of Disclosure

This disclosure relates to a database-backed Model Context Protocol (MCP) server architecture that exposes a compound query tier, enabling a large language model to retrieve complex multi-join, multi-filter data in a single tool call rather than requiring multiple sequential tool invocations.

## 2. Background

The Model Context Protocol (MCP) defines a standard interface through which large language models (LLMs) access external tools and data sources. In a typical MCP implementation, each tool exposes a single atomic operation: retrieve a record by ID, search by keyword, list items with a filter. When an LLM needs to answer a question that requires data from multiple tables, multiple filter conditions, or aggregation across records, the LLM must make multiple sequential MCP tool calls, interpret intermediate results, and compose a final answer from the accumulated data.

This sequential pattern has three problems. First, it consumes multiple inference round-trips, increasing latency and token cost. Second, it requires the LLM to maintain intermediate state across tool calls, introducing the risk of context loss or misinterpretation in long tool-call chains. Third, it creates a large surface area of individual tool definitions that the LLM must reason about, increasing the probability of selecting the wrong tool or composing tool calls incorrectly.

No existing MCP server architecture exposes a compound query tier that allows an LLM to express a multi-table, multi-filter, multi-aggregation query in a single tool call and receive a fully composed result.

## 3. Technical Disclosure

### 3.1 Database-at-MCP-Layer Architecture

The MCP server is backed directly by a database containing the application's structured data. Rather than wrapping individual API endpoints as MCP tools, the server exposes the database query capability itself — constrained by a security layer that enforces tenant isolation, field-level access control, and query complexity limits.

The database is not exposed raw. A query schema defines which tables, fields, joins, and aggregation functions are available through the MCP interface. The LLM cannot execute arbitrary SQL. It can only compose queries within the boundaries of the published query schema.

### 3.2 Compound Query Tool Interface

The compound query tool accepts a structured query definition containing:

- **tables**: An array of table identifiers to query, drawn from the published schema.
- **joins**: An array of join conditions specifying which fields link the tables.
- **filters**: An array of filter conditions applied to any field in any joined table. Filters support equality, range, substring, set membership, and null checks.
- **aggregations**: Optional aggregation functions (count, sum, average, min, max, distinct) applied to specified fields, with optional group-by clauses.
- **projections**: The specific fields to return in the result set, from any joined table.
- **ordering**: Sort order for the result set.
- **limit**: Maximum number of records to return.

The compound query tool validates the incoming query against the published schema, enforces tenant isolation by injecting the tenant identifier as a mandatory filter on every table, applies field-level access control to ensure the requesting user's role permits access to every requested field, enforces query complexity limits (maximum number of joins, maximum result set size, maximum aggregation cardinality), and then executes the validated query against the database.

### 3.3 N-to-1 Tool Call Collapse

The compound query tier collapses what would otherwise be N sequential MCP tool calls into a single call. For example, a question like "which matters have more than 5 documents uploaded this month, grouped by practice area, with the average confidence score for each group" would require at minimum: (1) list matters, (2) for each matter count documents filtered by date, (3) group by practice area, (4) compute average confidence per group. In the compound query tier, this is a single tool call with one join, two filters, one aggregation, and one group-by clause.

### 3.4 Tenant Isolation Enforcement

Every query executed through the compound query tier is automatically scoped to the requesting tenant. The tenant identifier is injected as a mandatory equality filter on every table in the query. This injection occurs after schema validation and before query execution. The LLM cannot override, omit, or modify the tenant filter. A query that would return data from multiple tenants is structurally impossible.

### 3.5 Query Complexity Bounds

The compound query tier enforces configurable complexity limits to prevent resource exhaustion:

- Maximum number of joined tables per query (default: 4).
- Maximum result set size (default: 500 records).
- Maximum aggregation cardinality (default: 100 distinct groups).
- Query execution timeout (default: 10 seconds).

Queries exceeding any limit are rejected before execution with a structured error indicating which limit was exceeded.

## 4. Scope of Disclosure

This disclosure establishes prior art for the concept of a database-backed MCP server exposing a compound query tier that allows large language models to execute multi-table, multi-filter, multi-aggregation queries in a single tool call with mandatory tenant isolation injection, field-level access control, and query complexity bounds.

This disclosure does not cover the integration of this query tier with any specific compliance wire protocol, any specific cryptographic attestation mechanism, or any specific database engine.

---

*© 2026 Technology Outlaws LLC. All rights reserved. This publication is made solely for the purpose of establishing prior art under 35 U.S.C. § 102.*
