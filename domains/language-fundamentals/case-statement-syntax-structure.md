---
title: "AL Case Statement Syntax and Structure"
domain: "language-fundamentals"
subdomain: "control-structures"
difficulty: "beginner"
bc_versions: "14+"
al_version: "6.0+"
tags: ["case", "switch", "control-flow", "syntax", "branching"]
prerequisites: ["if-statements", "variable-types", "basic-syntax"]
related_topics: ["case-multiple-conditions", "case-performance", "case-error-handling"]
samples: "samples/case-statement-syntax-structure.md"
---

# AL Case Statement Syntax and Structure

## Overview

The AL Case statement provides a structured approach to multi-branch conditional logic in Business Central. It offers cleaner, more readable code compared to multiple IF-THEN-ELSE chains when evaluating a single expression against multiple possible values.

**Core Syntax Pattern:**
```
CASE Expression OF
    Value1: Statement1;
    Value2: Statement2;
    ELSE
        DefaultStatement;
END;
```

The CASE statement evaluates the expression once and executes the matching branch, providing better performance than cascaded IF statements for multiple conditions.

## Fundamental Structure Components

### Expression Evaluation
The case expression is evaluated once at statement entry. Supported expression types include:
- **Integer values** and ranges
- **Enum values** and option strings
- **Text and Code fields** with exact matching
- **Boolean values** (though IF statements are typically preferred)
- **Date and DateTime** values for temporal branching

### Value Matching and Execution
Case values must exactly match the expression and be compile-time constants. Only the first matching branch executes with no fall-through behavior. Each branch can contain single statements or BEGIN-END blocks.

## AL-Specific Implementation Details

### Supported Data Types
AL case statements support integer values, enum types, option fields, text/code fields, boolean values, and date/datetime expressions. Each data type follows specific matching rules and syntax patterns.

### Multi-Statement Branches  
When branches require multiple statements, use BEGIN..END blocks to group related operations. Single statements can be placed directly after the colon without additional grouping.

## Best Practices for Structure

### Expression and Branch Guidelines
- **Simple Expressions**: Use expressions that evaluate to basic types
- **Logical Organization**: Order branches by frequency and group related cases
- **Default Handling**: Always include ELSE clause for robust error handling
- **Readable Code**: Use consistent indentation and descriptive value names

## Common Implementation Patterns

### Enum-Based Switching
Most effective pattern for type-safe branching with IntelliSense support. Use enum values with appropriate ELSE clause for error handling.

### Status Processing
Common pattern for processing different record states with specific handling procedures for each status value.

## Structural Guidelines

### When to Use Case vs IF
- **Use CASE**: 3+ conditions on same expression, enum/option switching
- **Use IF**: Complex boolean logic, different expressions per condition, 1-2 simple conditions

### Nesting Considerations
- **Avoid Deep Nesting**: Prefer flat structure with helper procedures
- **Extract Complex Logic**: Move complex branch logic to separate procedures
- **Maintain Readability**: Keep case statements focused and scannable

## CodeCop Compliance Notes

Case statements should follow Microsoft's AL coding standards, particularly regarding BEGIN..END usage and error message handling. Teams may choose different approaches based on their specific requirements and coding standards.

See companion sample file for CodeCop-compliant examples and alternative approaches.

