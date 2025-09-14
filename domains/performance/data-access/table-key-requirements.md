---
title: "API Page Source Table Key Requirements"
domain: "performance"
difficulty: "advanced"
bc_versions: "14+"
tags: ["api-pages", "table-keys", "performance"]
samples: "samples/table-key-requirements.md"
---

# API Page Source Table Key Requirements

## Overview

API Pages require specific table key configurations to ensure optimal OData query performance and proper data access patterns. Understanding these requirements is critical for high-performance API implementations that scale effectively under production loads.

The BC platform automatically generates OData endpoints based on table key structures, making key design decisions directly impact API query performance. Poor key design can result in full table scans and timeout errors in API consumers.

## Primary Key Requirements

API Pages must have a well-defined primary key that serves as the unique identifier for OData operations. The primary key becomes the basis for individual record retrieval operations through the API endpoint.

Single-field primary keys perform better than composite keys for API operations. The SystemId field, when available, provides optimal performance as it's a GUID field with guaranteed uniqueness and efficient indexing characteristics.

Composite primary keys require careful consideration of field ordering to ensure the most selective fields appear first in the key definition. This ordering directly impacts query performance when API consumers filter on subset of key fields.

## Secondary Key Performance Impact

Secondary keys on API Page source tables significantly impact OData query performance. Each secondary key becomes a potential filter path for API consumers, and poorly designed keys can create performance bottlenecks.

Keys supporting common API query patterns should be prioritized. Consider the typical filtering, sorting, and grouping requirements of API consumers when designing secondary keys for optimal query execution.

The BC platform may automatically select keys for query execution based on OData filter parameters. Understanding this selection logic helps predict and optimize actual query performance under different usage patterns.

## Key Field Selection Criteria

Choose key fields based on query selectivity and data distribution. Fields with high cardinality and even distribution provide better performance than fields with skewed data patterns or low selectivity.

Avoid including large text fields or blob fields in key definitions as these impact both storage requirements and query performance. Keys should focus on efficient identifier and categorization fields.

Consider the impact of key fields on data modification operations. Fields that change frequently should be avoided in secondary keys as this requires key maintenance overhead during data updates.

## OData Query Optimization

API Page key design directly influences OData query compilation and execution. The BC platform analyzes available keys when translating OData filters into SQL query patterns.

Complex OData queries may require specific key combinations to avoid performance issues. Understanding common API consumer query patterns helps identify required key structures during design phase.

Query performance monitoring should include analysis of key usage patterns to identify optimization opportunities. Unused keys consume storage and maintenance overhead without providing performance benefits.

## Multi-Tenancy Considerations

In multi-tenant environments, key design must consider tenant isolation requirements. Keys may need to include tenant-specific fields to ensure proper data segregation and performance isolation.

DataPerCompany table property affects key performance characteristics. Tables with DataPerCompany = false require additional consideration for key design to maintain performance across tenant boundaries.

Global tables require careful key design to prevent cross-tenant data access issues while maintaining query performance for legitimate multi-tenant scenarios.

## Performance Monitoring

Monitor API Page query performance through BC telemetry and database performance metrics. Key performance indicators include query execution time, index usage patterns, and resource consumption.

Identify queries that result in table scans or inefficient key usage. These represent optimization opportunities through key design improvements or query pattern adjustments.

Regular performance analysis should include review of key effectiveness and identification of missing key requirements based on actual usage patterns.

## Common Implementation Issues

Missing or inappropriate primary keys result in poor OData query performance and potential data consistency issues. Ensure every API Page source table has an appropriate primary key definition.

Excessive secondary keys can impact data modification performance without providing corresponding query benefits. Focus key design on actual API consumer requirements rather than theoretical optimization.

Key field data type mismatches between related tables can prevent efficient join operations in complex OData queries. Maintain consistent data types across related table key fields.

## Best Practices

Design keys based on actual API consumer requirements rather than generic optimization principles. Understanding real-world usage patterns drives effective key design decisions.

Balance query performance requirements with data modification overhead. Keys provide query benefits but incur maintenance costs during data updates.

Document key design decisions and performance assumptions for future optimization efforts. This documentation helps maintain design consistency and supports performance troubleshooting.