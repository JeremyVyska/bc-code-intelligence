---
title: "SetLoadFields Placement Before Filters"
domain: "dean-debug"
difficulty: "beginner"
bc_versions: "14+"
tags: ["setloadfields", "placement", "filters", "order"]
prerequisites: ["setloadfields-basics", "record-filtering"]
samples: "samples/setloadfields-before-filters.md"
---
# SetLoadFields Placement Before Filters

## Overview

SetLoadFields must be placed before filter operations (SETRANGE, SETFILTER) to be effective. This placement ensures that field loading optimization is applied during the database query execution rather than after records have already been loaded with all fields.

Proper placement is critical for realizing the performance benefits of SetLoadFields optimization. Incorrect placement results in no optimization benefit and can actually reduce performance due to unnecessary function calls.

## Execution Order Impact

When SetLoadFields is placed before filters, BC incorporates the field loading specification into the database query planning. The database query is optimized to retrieve only the specified fields for records matching the filter criteria.

When SetLoadFields is placed after filters, BC has already executed the database query with full field loading. The SetLoadFields call becomes ineffective because the records are already loaded in memory with all fields populated.

## BC Query Processing

BC processes record access operations in a specific sequence:
1. SetLoadFields specification (if present)
2. Filter criteria application (SETRANGE, SETFILTER)
3. Database query execution with optimized field loading
4. Record iteration (FIND, FINDSET, etc.)

This sequence ensures that field loading optimization is incorporated into the actual database query rather than applied as a post-processing step.

## Performance Implications

Correct placement delivers significant performance benefits:
- Reduced database I/O for record retrieval
- Lower memory consumption during record processing
- Improved query execution speed
- Better network efficiency between BC service and database

Incorrect placement provides no performance benefit and adds unnecessary processing overhead from the ignored SetLoadFields call.

## Common Placement Errors

The most common error is placing SetLoadFields after SETRANGE or SETFILTER operations. This happens when developers add SetLoadFields optimization to existing code without restructuring the data access logic.

Another common error is placing SetLoadFields immediately before record iteration (FIND, FINDSET) but after filter setup. While this appears logical, it's too late in the execution sequence to provide optimization benefits.

## Verification Techniques

Verify correct placement by reviewing the sequence of your data access operations. SetLoadFields should be the first operation after record variable declaration and before any filter operations.

Use BC performance profiling tools to confirm that field loading optimization is actually taking effect. Properly placed SetLoadFields should show measurable improvements in query execution time and memory usage.

## Code Organization Patterns

Organize your data access code with a consistent pattern:
1. Record variable declaration
2. SetLoadFields specification
3. Filter setup (SETRANGE, SETFILTER)
4. Record iteration and processing

This pattern ensures correct placement and makes the optimization visible to other developers working with the code.

## Integration with Other Optimizations

Proper SetLoadFields placement is foundational for other field loading optimizations. Primary key optimization, filter field exclusion, and conditional field loading all depend on correct placement before filter operations.

Consider establishing coding standards that require SetLoadFields as the first data access operation after record variable declaration to ensure consistent correct placement across your codebase.