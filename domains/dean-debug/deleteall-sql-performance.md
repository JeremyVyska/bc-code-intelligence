---
title: "DeleteAll SQL Translation and Performance Impact"
domain: "dean-debug"
difficulty: "intermediate"
bc_versions: "14+"
tags: ["deleteall", "sql-translation", "performance-optimization"]
samples: "samples/deleteall-performance.md"
type: "improvement-pattern"
category: "performance-optimization"
pattern_type: "good"
severity: "medium"
impact_level: "high"
improvement_suggestion: "Use DeleteAll for bulk deletions to leverage SQL DELETE performance"
implementation_steps: ["identify-bulk-deletion-scenarios", "replace-iterative-loops", "validate-event-dependencies"]
validation_criteria: ["performance-benchmarks", "event-dependency-analysis"]
technical_areas: ["al-code", "database"]
---

# DeleteAll SQL Translation and Performance Impact

## Overview

DeleteAll method translates directly to SQL DELETE statements, providing significant performance advantages over iterative record deletion. This single SQL operation eliminates AL runtime overhead but bypasses table-level AL events, creating critical performance vs. event consistency trade-offs.

**Key Insight**: DeleteAll executes as one database operation rather than individual record processing, dramatically improving performance for large datasets while skipping AL-level validations.

## Strategic Framework

### Performance Characteristics
- **Single SQL DELETE**: Entire operation executes as one database statement
- **Eliminates AL Overhead**: No per-record AL runtime processing
- **Database-Level Execution**: Constraints and triggers still execute
- **Bypasses AL Events**: OnDelete triggers and validations are skipped

### When to Use DeleteAll
- **Large Record Sets**: Performance gains most pronounced with bulk deletions
- **Simple Deletion Logic**: No complex AL validation required
- **Performance Critical**: Operations where speed is prioritized
- **Database Constraints Sufficient**: Referential integrity handled at database level

## Architecture Focus

### SQL Translation Impact
DeleteAll converts to optimized SQL DELETE with WHERE clauses matching record filters. Database engine handles the bulk operation efficiently using index structures and internal optimization strategies.

### Event Processing Differences
Table-level AL events (OnDelete triggers, validation logic) are completely bypassed. Only database-level constraints and triggers execute, creating potential inconsistencies with iterative deletion behavior.

### Memory and Resource Efficiency
Single SQL operation uses minimal AL runtime resources compared to loop-based deletion that processes each record through full AL stack.

## Best Practices

### Performance Optimization
- **Bulk Operations**: Prefer DeleteAll for large dataset removals
- **Filter Optimization**: Ensure filters use indexed fields for optimal SQL execution
- **Transaction Boundaries**: Consider transaction scope for very large deletions
- **Resource Monitoring**: Monitor database resource usage during bulk operations

### Event Dependency Management
- **Validate Dependencies**: Ensure no critical AL logic exists in bypassed events
- **Alternative Patterns**: Implement necessary validations before DeleteAll execution
- **Documentation**: Document event bypass behavior for maintenance clarity

## Anti-Patterns

### Avoid These Approaches
- Using DeleteAll when AL events contain critical business logic
- Iterative deletion loops for performance-critical bulk operations
- Ignoring database constraint dependencies during bulk deletions
- Mixing DeleteAll with individual record processing without understanding event differences

*Implementation examples: see samples/deleteall-performance.md*