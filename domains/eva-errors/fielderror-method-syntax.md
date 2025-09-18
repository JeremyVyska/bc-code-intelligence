---
title: "FieldError Method Syntax and Behavior"
domain: "eva-errors"
difficulty: "intermediate"
bc_versions: "14+"
tags: ["fielderror", "validation", "error-messages", "posting"]
samples: "samples/fielderror-syntax.md"
---
# FieldError Method Syntax and Behavior

## Overview

The FieldError method generates field-specific error messages with automatic field name inclusion and record context. Unlike TestField, FieldError assumes validation has already failed and focuses on error message generation rather than condition checking.

**Core Purpose**: Generate contextual error messages that clearly identify the problematic field and its current state.

## Method Signatures

### Basic Syntax
The FieldError method accepts two forms:
- `Record.FieldError(FieldNo: Integer)` - Uses default message generation
- `Record.FieldError(FieldNo: Integer, MsgText: Text)` - Uses custom message text

### Parameter Details
- **FieldNo**: Integer identifying the field causing the error
- **MsgText**: Optional custom error message text (follows lowercase convention)

## Behavioral Characteristics

### Automatic Context Building
FieldError automatically constructs error messages including:
- Field caption from table metadata
- Current field value (when not blank)
- Record identification (primary key values)
- Table name context

### Error Message Structure
Messages follow the pattern: `[Field Caption] [Current Value] [Custom Message] in [Table Name] [Record Key]`

Example output: "Document No. SO-001 must be unique in Sales Header Document Type='Order',No.='SO-001'"

## Implementation Patterns

### Standard Posting Codeunit Usage
FieldError is extensively used in standard BC posting codeunits (12 Gen. Jnl.-Post Line, 80 Sales-Post, 90 Purch.-Post) for validation failure reporting after conditions have been tested.

### Field Validation Context
Common usage involves testing business conditions first, then calling FieldError when validation fails. This pattern separates condition testing from error reporting.

## FieldError vs TestField Comparison

### Use FieldError When
- **Validation already failed**: Error condition has been determined
- **Custom error context needed**: Standard TestField message insufficient
- **Complex validation logic**: Multiple conditions or calculations involved
- **Posting routine errors**: Standard pattern in posting codeunits

### Use TestField When
- **Simple null/empty checks**: Standard "must have a value" scenarios
- **Direct field validation**: Single condition testing
- **Setup validation**: Configuration fields that must be populated

### Behavioral Differences
- **TestField**: Tests condition AND generates error
- **FieldError**: Only generates error (assumes condition already tested)
- **TestField**: Limited to empty/non-empty and value matching
- **FieldError**: Accepts any custom error message

## Best Practices

### Message Design
- **Be specific**: Describe exactly what's wrong and why
- **Include context**: Reference related fields or business rules when relevant
- **Provide guidance**: Suggest how to resolve the issue when possible
- **Follow conventions**: Start custom messages with lowercase letter

### Error Timing
- **Use after validation**: Don't duplicate condition checking
- **Fail fast**: Call FieldError immediately when error is detected
- **Single responsibility**: One FieldError per validation condition

### Performance Considerations
- **Field number constants**: Use field number constants, not field names
- **Message construction**: Keep custom messages concise but informative
- **Early exit**: Place FieldError calls after all condition checks

## Common Implementation Scenarios

### Required Field Validation
Use FieldError without custom message for standard empty field scenarios where default messaging is sufficient.

### Range Validation
Combine condition testing with FieldError for numeric range violations, providing specific boundary information in custom messages.

### Business Rule Violations
Apply FieldError for complex business logic failures, including cross-field dependencies and calculated validation scenarios.

### Cross-Field Dependencies
Validate relationships between fields using FieldError to identify the specific field causing the constraint violation.

## Anti-Patterns

### Avoid These Approaches
- **Redundant condition checking**: Don't test condition if already validated
- **Generic error messages**: Don't use FieldError for non-field-specific errors
- **Message duplication**: Don't repeat information already included automatically
- **Complex message logic**: Keep message construction simple and direct

## Usage Guidelines

FieldError is most effective when integrated into structured validation routines where business logic testing precedes error reporting. The method's strength lies in automatic context generation combined with flexible custom messaging.

For comprehensive implementation examples and working code samples, see the companion AL code file referenced below.

