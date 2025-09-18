---
title: "TestField Error Handling and Custom Messages"
domain: "eva-errors"
difficulty: "intermediate"
bc_versions: "14+"
tags: ["error-handling", "validation", "user-experience", "custom-messages"]
prerequisites: ["testfield-basic-syntax", "error-handling-fundamentals"]
samples: "samples/testfield-error-handling.md"
---
# TestField Error Handling and Custom Messages

## Overview

TestField generates automatic error messages, but business scenarios often require custom error handling and messaging. Understanding TestField error behavior enables better user experience and error management strategies.

**Key Principle**: TestField errors are runtime exceptions that halt procedure execution unless caught and handled.

## Default Error Behavior

### Standard Error Messages
TestField generates system messages like "You must specify Name in Customer" when validation fails. These messages include:
- Field caption from table definition
- Table name
- Current record context when available

### Error Flow Impact
TestField errors immediately terminate current procedure execution. Control returns to calling code or user interface with error state.

## Custom Error Handling Patterns

### Try-Function Approach
Wrap TestField calls in TryFunction procedures to catch validation errors without terminating execution. Returns boolean success indicator for error-tolerant processing scenarios.

### Custom Validation with Better Messages
Replace TestField with custom validation logic when business context requires specific error messages. Provides more meaningful feedback to users than system-generated messages.

### Conditional TestField Usage
Apply TestField validation based on business rules or record state. Ensures appropriate validation intensity for different scenarios without over-validating.

## Advanced Error Management

### Error Context Enhancement
Capture TestField errors using TryFunction pattern and enhance with business context before re-throwing. Provides users with meaningful error information including document details.

### Multiple Field Validation with Aggregated Errors
Collect multiple validation failures before reporting to users. Improves user experience by showing all validation issues at once rather than one-by-one error correction.

## Integration with Business Logic

### Workflow Integration
TestField works seamlessly with workflow validations. Use TestField for mandatory data checks within workflow step validations.

### API Integration
For API endpoints, catch TestField errors and return structured error responses rather than allowing raw BC errors to propagate.

### Batch Processing
In batch processing scenarios, use TryFunction pattern to continue processing other records when individual record validation fails.

## User Experience Considerations

### Progressive Validation
Validate fields in logical order so users get meaningful first-error feedback rather than random field validation failures.

### Context-Aware Messages
Include business context in custom error messages to help users understand why specific fields are required.

### Field Focus Integration
Custom validation procedures can integrate with page field focus to direct users to specific fields requiring attention.

## Common Patterns

### Validation Procedures
Create dedicated validation procedures that use TestField internally but provide business-context error messages.

### Status-Based Validation
Use TestField with status checks to enforce business rules at specific workflow stages.

### Cross-Table Validation
Combine TestField with related table lookups to validate complex business relationships.

