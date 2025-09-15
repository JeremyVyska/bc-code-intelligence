---
title: "SetLoadFields Primary Key Optimization"
domain: performance
type: code-pattern
pattern_type: good
category: performance
severity: medium
name: setloadfields-primary-key-optimization
regex_patterns:
  - "SetLoadFields\\s*\\([^)]*\\"No\\.\\"[^)]*\\)"
description: "Optimization pattern for including primary key fields in SetLoadFields operations"
impact_level: medium
detection_confidence: high
difficulty: intermediate
bc_versions: "14+"
tags: ["setloadfields", "primary-key", "memory-optimization"]
prerequisites: ["record-variables", "primary-key-concepts"]
samples: "samples/setloadfields-primary-key.md"
---

# SetLoadFields Primary Key Optimization

## Overview

SetLoadFields with primary key fields provides the most fundamental memory optimization in BC development. By loading only the primary key fields, you minimize memory usage while maintaining record identity for subsequent operations.

This optimization is particularly effective when you need to pass records between functions, store record references, or perform record existence checks without requiring full record data.

## Core Concept

Primary key optimization leverages BC's ability to uniquely identify records using only key fields. When you specify primary key fields in SetLoadFields, BC loads minimal data while preserving full record functionality for key-based operations.

The optimization works because BC can reconstruct record identity and perform key-based lookups without loading non-key field data into memory. This creates significant performance benefits in scenarios involving large record sets or memory-constrained operations.

## When to Apply

Use primary key optimization when:
- Passing records between functions where only identity matters
- Building record reference collections
- Performing existence checks or validation
- Processing large datasets where full field loading causes memory pressure
- Creating temporary record variables for subsequent filtering

Avoid when you need non-key field values immediately after the SetLoadFields call, as this forces additional database round trips.

## Performance Impact

Primary key optimization delivers:
- Reduced memory consumption (typically 60-80% reduction)
- Faster record loading from database
- Lower network traffic between BC service and database
- Improved garbage collection performance
- Better scalability with large record volumes

The optimization is most pronounced with tables containing many fields or large field values (text, blob, media fields).

## Implementation Patterns

The basic pattern involves identifying the table's primary key fields and specifying only those fields in the SetLoadFields call. Most BC tables use surrogate keys (single integer field) or composite keys (multiple fields).

For tables with surrogate primary keys, specify the single key field. For tables with composite primary keys, include all fields that comprise the primary key.

## Common Use Cases

**Record Reference Storage**: When maintaining collections of record references where you need identity but not data values.

**Cross-Function Record Passing**: Passing records to functions that will apply their own field loading based on their specific needs.

**Existence Validation**: Checking if records exist without needing their field values.

**Bulk Processing Setup**: Loading minimal record data before applying specific field loading for different processing paths.

## Best Practices

Always identify the correct primary key fields for your target table. Use the table definition or AL development environment to verify key field composition.

Apply primary key optimization early in your data access chain, before other operations that might implicitly load additional fields.

Document your SetLoadFields usage to help other developers understand the optimization strategy and avoid accidentally loading additional fields.

## Integration with Other Optimizations

Primary key optimization combines well with filter placement optimization - apply SetLoadFields with primary keys before setting filters to minimize the data loaded during filter evaluation.

This optimization also supports subsequent field-specific loading where you can add additional fields based on conditional logic or processing requirements.