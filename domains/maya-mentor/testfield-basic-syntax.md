---
title: "TestField Basic Syntax and Usage"
domain: "maya-mentor"
difficulty: "beginner"
bc_versions: "14+"
tags: ["validation", "fields", "error-handling", "data-integrity"]
prerequisites: ["record-variables", "field-access"]
samples: "samples/testfield-basic-syntax.md"
---
# TestField Basic Syntax and Usage

## Overview

TestField validates that a field contains a value before proceeding with business logic. It throws an error if the field is empty, preventing operations that require valid data. Essential for data integrity in BC applications.

**Core Pattern**: `Record.FieldName.TestField()` - Validates field has value, throws error if empty.

## Basic Syntax Forms

### Simple Value Validation
TestField validates that a field contains a non-empty value. Called on any field variable, it checks for empty strings, zero dates, or zero numeric values.

### Expected Value Validation
TestField can validate against specific expected values. Pass the expected value as a parameter to verify the field matches exactly.

### Variable-Based Expected Values
Expected values can be stored in variables before TestField validation, enabling dynamic validation scenarios.

## Field Type Considerations

### Text and Code Fields
Text fields test for non-empty strings. Empty string ('') triggers validation error.

### Option Fields
Option fields test against specific option values. Commonly used to verify workflow states.

### Boolean Fields
Boolean fields can be tested for specific true/false values to enforce business rules.

### Date and DateTime Fields
Date fields test for non-zero dates. 0D (undefined date) triggers validation error.

### Decimal and Integer Fields
Numeric fields test for non-zero values. Zero values trigger validation error.

## Common Usage Patterns

### Mandatory Field Validation
Group related TestField validations to ensure all required fields contain data before processing. Essential for maintaining data integrity in business processes.

### Status Verification
Use TestField with specific option values to verify records are in expected workflow states before proceeding with operations.

### Precondition Checks
Validate all prerequisite field data exists before executing complex business logic. Prevents processing failures and maintains consistent data state.

## Best Practices

### Field Validation Timing
Call TestField early in procedures before complex processing begins. This fails fast and provides clear error context.

### Logical Grouping
Group related TestField calls together for related validations. This creates clear validation blocks.

### User Experience
TestField provides system-generated error messages. For better UX, consider custom error handling for complex business rules.

