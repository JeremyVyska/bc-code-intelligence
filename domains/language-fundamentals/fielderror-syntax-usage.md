---
title: "FieldError Syntax and Usage"
domain: "language-fundamentals"
difficulty: "beginner"
bc_versions: "14+"
tags: ["fielderror", "error-handling", "validation", "record-methods", "user-feedback", "field-validation", "runtime-errors"]
prerequisites: ["record-basics", "field-access"]
samples: "samples/fielderror-syntax-usage.md"
related_topics: ["testfield-method", "error-method", "field-validation", "record-validation"]
---

# FieldError Syntax and Usage

## Overview

FieldError is a fundamental AL method that displays an error dialog with a field-specific error message and terminates current processing. Unlike the generic Error method, FieldError automatically includes the field caption and current field value in the error message, providing better user context for validation failures.

The method belongs to the Record data type and operates on specific fields within record instances. When called, FieldError immediately stops code execution and displays a formatted error message to the user.

**Core Purpose**: Provide field-specific error feedback with automatic context inclusion for better user experience during validation failures.

## Method Signature and Variations

### Basic Syntax
The FieldError method follows the pattern: Record.FieldError(Field [, Text])

The FieldError method accepts the target field as the first parameter and an optional custom error text as the second parameter. When no custom text is provided, BC generates a standard error message using the field's caption and current value.

### Parameter Details
- **Field**: The specific field reference where the error occurred
- **Text**: Optional custom error message to display instead of the default message
- **Return Type**: None (method terminates execution)

## Automatic Message Construction

### Default Message Format
When called without custom text, FieldError constructs messages following the pattern: "Field Caption must not be Value in Table Caption: Primary Key Values". This automatic formatting provides consistent error messaging across the application.

The system automatically retrieves the field caption from the field's metadata, includes the current field value, and adds table context with primary key identification for precise error location.

### Custom Message Integration
When custom text is provided, FieldError still includes field context but uses the custom message as the primary error description. The field caption and value are typically appended to maintain context while allowing specific error guidance.

## Execution Behavior

### Immediate Termination
FieldError causes immediate code execution termination at the point of call. No subsequent code in the current procedure executes after a FieldError call, making it suitable for validation scenarios where further processing is meaningless.

### User Interface Impact
The error displays in a modal dialog that requires user acknowledgment before continuing. Users see the complete error context including field identification, current value, and record location.

### Transaction Handling
FieldError respects current transaction boundaries and triggers appropriate rollback behavior when called within database transactions. This ensures data consistency when validation failures occur during data modification operations.

## Common Usage Patterns

### Field Value Validation
FieldError commonly validates field values against business rules, checking for invalid data ranges, forbidden values, or constraint violations. The method provides immediate feedback when field contents violate established business logic.

### State-Dependent Validation
Fields often require different validation based on record state or related field values. FieldError enables context-sensitive validation that considers the complete record state when evaluating individual field validity.

### Cross-Field Validation
When field validity depends on other field values, FieldError provides targeted feedback while maintaining awareness of the broader record context. This enables complex validation scenarios with clear user guidance.

## Best Practices

### Message Clarity
When providing custom error messages, focus on actionable guidance that helps users understand both the problem and the solution. Combine BC's automatic field context with specific correction instructions.

### Validation Timing
Call FieldError during appropriate validation phases, typically in OnValidate triggers or validation procedures. Avoid FieldError in contexts where graceful error handling might be more appropriate.

### User Experience
Consider the user's workflow when implementing FieldError calls. Provide enough context for users to understand and correct the validation failure without requiring deep system knowledge.

