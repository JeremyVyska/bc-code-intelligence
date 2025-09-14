---
title: "FieldError Message Construction Patterns"
domain: "error-handling"
difficulty: "intermediate"
bc_versions: "14+"
tags: ["fielderror", "error-messages", "user-experience", "localization", "message-formatting", "validation-feedback"]
prerequisites: ["fielderror-syntax-usage", "text-constants", "label-variables"]
samples: "samples/fielderror-message-construction.md"
related_topics: ["localization-patterns", "user-feedback-design", "validation-error-handling"]
---

# FieldError Message Construction Patterns

## Overview

FieldError message construction combines automatic field context with custom messaging to provide meaningful validation feedback. Understanding how FieldError builds error messages enables developers to create user-friendly validation experiences that guide users toward successful data entry.

The method automatically includes field captions, current values, and record identification, while allowing custom text to provide specific business context and correction guidance.

**Key Principle**: Effective FieldError messages combine system-generated field context with business-specific guidance to create actionable user feedback.

## Automatic Message Components

### Field Context Integration
FieldError automatically includes the field caption from table metadata, ensuring consistent field identification across different user interface languages. The system retrieves localized field captions when multiple languages are configured.

The current field value is included in the error message, providing users with immediate context about the data that triggered the validation failure. This eliminates ambiguity about which specific value caused the problem.

### Record Identification
FieldError includes the table name and primary key values to identify the specific record where the validation failure occurred. This record context helps users navigate back to the problematic data when working with large datasets.

## Default Message Patterns

### Empty Field Messages
When a field is empty and FieldError is called without custom text, the system generates messages following the pattern: "You must specify [Field Caption] in [Table Caption] [Primary Key Values]". This pattern clearly indicates missing required data.

### Invalid Value Messages
When a field contains data and FieldError is called without custom text, the system generates messages following the pattern: "[Field Caption] must not be [Field Value] in [Table Caption] [Primary Key Values]". This pattern indicates that the current value is invalid.

## Custom Message Integration

### Text Parameter Usage
The optional text parameter allows developers to provide specific validation guidance while maintaining automatic field context. Custom text becomes the primary error description, with field context appended for complete user guidance.

### Message Formatting Rules
Custom FieldError text should not begin with capital letters, as the system automatically capitalizes the complete error message. This formatting rule ensures grammatically correct error messages when combined with automatic field context.

### Business Context Addition
Custom messages should provide business rule context that helps users understand why the validation failed and what action they need to take to correct the data.

## Localization Considerations

### Label Variable Integration
Use Label variables for custom FieldError text to support multilingual applications. Label variables automatically provide localized text based on user language preferences, ensuring consistent user experience across language configurations.

### Field Caption Localization
FieldError automatically uses localized field captions from table metadata, ensuring field names appear in the user's preferred language without additional development effort.

### Message Consistency
Maintain consistent message formatting patterns across the application by establishing standard Label variables for common validation scenarios.

## Message Construction Strategies

### Actionable Guidance
Construct custom messages that tell users not only what is wrong but also what they should do to fix the problem. Include specific value ranges, acceptable formats, or procedural steps when appropriate.

### Business Rule Explanation
When FieldError is triggered by complex business rules, provide enough context for users to understand the underlying business logic without overwhelming them with technical details.

### Progressive Information
For complex validation scenarios, consider whether a single FieldError message can adequately explain the problem, or if multiple validation steps with different error messages would provide better user guidance.

## Integration with Validation Logic

### OnValidate Trigger Usage
FieldError integrates seamlessly with OnValidate triggers, where business rule validation typically occurs. The method provides immediate feedback when field changes violate business constraints.

### Posting Routine Integration
Business Central's posting codeunits extensively use FieldError to provide specific feedback about data validation failures during complex business processes. This pattern ensures users receive targeted guidance about posting failures.

### Cross-Field Validation
When validation involves multiple fields, FieldError can focus user attention on the specific field that needs correction while explaining the relationship to other field values in the custom message text.

## User Experience Design

### Clear Problem Identification
Effective FieldError messages clearly identify what specific validation rule was violated, helping users understand the exact nature of the problem.

### Solution Guidance
Include positive instructions about correct data entry rather than just negative statements about what is wrong. Guide users toward successful completion of their data entry task.

### Context Preservation
Leverage FieldError's automatic context inclusion to help users maintain orientation within complex data entry scenarios, especially when working with large forms or multi-step processes.

## Common Message Patterns

### Range Validation
For numeric or date range validation, FieldError messages should specify the acceptable range and explain any business rationale for the constraints.

### Format Validation
For data format requirements, FieldError messages should provide examples of acceptable formats and explain any special formatting rules.

### Dependency Validation
For fields that depend on other field values, FieldError messages should explain the relationship and guide users toward consistent data entry.

