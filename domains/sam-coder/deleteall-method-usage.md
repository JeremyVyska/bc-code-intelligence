---
title: "DeleteAll Method Usage"
domain: "sam-coder"
difficulty: "intermediate"
bc_versions: "14+"
tags: ["deleteall", "data-manipulation", "triggers"]
prerequisites: ["table-basics", "record-management"]
samples: "samples/deleteall-usage.md"
---
# DeleteAll Method Usage

## Overview

DeleteAll is a BC method that removes all records from a table or filtered set while executing OnDelete triggers for each record. Unlike direct SQL DELETE operations, DeleteAll maintains full BC business logic compliance by triggering all associated validations and cascading operations.

**Key Characteristic**: Always executes OnDelete triggers - this is fundamental BC behavior that cannot be bypassed.

**Primary Use Cases**:
- Clearing temporary data with proper cleanup
- Bulk deletions requiring business logic validation
- Data maintenance operations preserving referential integrity
- Cleanup operations in test scenarios

**Critical Understanding**: DeleteAll is not a performance optimization - it's a business logic preservation tool that executes triggers for every deleted record.

## Strategic Framework

### When DeleteAll is Appropriate

**Data Integrity Scenarios**:
- Clearing temporary tables where related data must be cleaned up
- Removing test data while maintaining proper audit trails
- Bulk operations where business rules must be enforced
- Scenarios requiring cascade delete operations

**Business Logic Requirements**:
- Operations requiring OnDelete trigger execution
- Scenarios where related table updates are necessary
- Situations demanding proper logging and audit trail maintenance
- Cases where validation logic must run for each deletion

### When DeleteAll is Inappropriate

**Performance-Critical Operations**:
- Large dataset deletions where triggers are unnecessary
- Bulk cleanup operations on isolated data
- System maintenance requiring maximum speed
- Operations on tables without significant business logic

**Alternative Consideration Points**:
- Direct SQL operations for simple, non-validated deletions
- Batch processing approaches for very large datasets
- Specialized cleanup procedures for system maintenance
- Custom deletion logic for complex business scenarios

## Implementation Considerations

### Trigger Execution Impact

**Performance Implications**:
- Each record deletion executes full OnDelete trigger logic
- Related table updates and validations occur for every record
- Memory usage scales with trigger complexity and related data volume
- Network traffic increases with extensive related table operations

**Business Logic Preservation**:
- Maintains complete data integrity across all related tables
- Ensures proper audit trail and logging execution
- Preserves cascade delete operations and referential integrity
- Executes all custom business validation logic

### Error Handling Strategy

**Validation Failures**:
- Any OnDelete trigger failure stops the entire DeleteAll operation
- Partial deletions can occur if errors happen mid-process
- Transaction boundaries depend on implementation context
- Error recovery requires understanding of deletion state

**Recovery Patterns**:
- Implement proper transaction management around DeleteAll calls
- Design retry logic for recoverable validation failures
- Plan for partial completion scenarios and cleanup procedures
- Consider logging and monitoring for deletion operation tracking

## Best Practices

### Effective DeleteAll Usage

**Preparation Steps**:
- Verify that OnDelete triggers contain necessary business logic
- Ensure related data can handle the deletion operations
- Test deletion operations in controlled environments first
- Implement proper error handling and recovery procedures

**Execution Patterns**:
- Use appropriate filters to limit deletion scope
- Consider batch processing for very large datasets
- Implement progress tracking for long-running operations
- Plan for proper transaction boundaries and commit strategies

### Monitoring and Validation
- Monitor deletion progress and performance impact
- Validate that expected business logic executes correctly
- Verify related table cleanup occurs as designed
- Validate data integrity across all affected tables

## Anti-Patterns

### Performance Misunderstanding
**Incorrect Assumption**: Using DeleteAll for performance optimization. DeleteAll executes triggers, making it slower than direct SQL operations.

### Trigger Logic Avoidance
**Incorrect Pattern**: Attempting to bypass OnDelete triggers with DeleteAll. DeleteAll always executes OnDelete triggers - this cannot be disabled.

### Insufficient Error Handling
**Problematic Implementation**: Assuming DeleteAll operations always succeed. Implement comprehensive error handling and recovery procedures.

