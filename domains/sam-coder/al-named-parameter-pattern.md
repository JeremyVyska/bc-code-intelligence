---
title: "AL Named Parameter Pattern"
domain: "sam-coder"
difficulty: "intermediate"
bc_versions: "17+"
tags: ["parameters", "code-clarity", "maintainability", "readability"]
type: "improvement-pattern"
category: "code-quality"
pattern_type: "good"
severity: "medium"
impact_level: "medium"
improvement_suggestion: "Use descriptive variable naming to create self-documenting parameter patterns"
implementation_steps: ["identify-complex-calls", "create-descriptive-variables", "validate-clarity"]
validation_criteria: ["improved-call-clarity", "reduced-parameter-errors"]
technical_areas: ["al-code"]
samples: "samples/al-named-parameter-pattern.md"
---

# AL Named Parameter Pattern

## Overview

Named parameter patterns in AL improve code readability, reduce parameter ordering errors, and enhance maintainability by making procedure calls self-documenting. This pattern particularly benefits procedures with multiple parameters of similar types.

**Key Principle**: Use descriptive variable naming to make procedure calls immediately understandable without referring to procedure definitions.

## Strategic Framework

### Implementation Approach
AL doesn't support native named parameters, but naming conventions and parameter objects achieve similar benefits through descriptive variable usage.

### Parameter Object Pattern
Create dedicated parameter records or temporary tables for complex parameter sets, providing natural naming and extensibility.

### Descriptive Variable Naming
Use clearly named local variables that self-document parameter purposes when calling procedures.

### Parameter Documentation
Structure parameter lists with consistent ordering and comprehensive documentation to support clear calling patterns.

## Architecture Focus

### Maintainability Enhancement
**Parameter Evolution**: Parameter objects support adding new parameters without breaking existing calls
**Type Safety**: Structured parameters reduce type mismatch errors
**Documentation**: Self-documenting code through clear parameter naming
**Refactoring Support**: Named patterns simplify procedure signature changes

### Code Clarity Benefits
**Call Site Readability**: Procedure calls clearly communicate their purpose and data flow
**Parameter Validation**: Centralized parameter validation within parameter objects
**Default Handling**: Parameter objects can encapsulate default value logic
**Complex Operations**: Multi-step operations benefit from structured parameter passing

### Performance Considerations
Parameter objects may have slight overhead but improve development velocity and reduce debugging time through clearer interfaces.

## Best Practices

### Variable Naming Strategy
Create descriptive local variables before procedure calls:
```
CustomerNo := 'C0001';
PostingDate := WorkDate();
DocumentType := DocumentType::Invoice;
PostSalesDocument(CustomerNo, PostingDate, DocumentType);
```

### Parameter Object Design
- **Logical Grouping**: Group related parameters into cohesive objects
- **Validation Integration**: Include parameter validation within object methods
- **Default Values**: Implement sensible defaults for optional parameters
- **Clear Naming**: Use business domain terminology for parameter properties

### Procedure Interface Design
- **Consistent Ordering**: Establish standard parameter ordering patterns
- **Optional Parameters**: Place optional parameters at end of parameter list
- **Documentation**: Provide comprehensive parameter documentation
- **Interface Stability**: Design parameter interfaces for long-term stability

### Call Site Organization
- **Descriptive Variables**: Use meaningful variable names that explain parameter purpose
- **Logical Grouping**: Group related parameter assignments together
- **Clear Separation**: Separate parameter preparation from procedure calls
- **Error Context**: Include parameter validation before complex procedure calls

## Anti-Patterns

### Avoid These Approaches
- **Magic Numbers**: Passing literal values without descriptive variable names
- **Parameter Soup**: Long parameter lists without logical organization
- **Type Confusion**: Multiple parameters of same type without clear differentiation
- **Inconsistent Ordering**: Different parameter ordering patterns across similar procedures

Procedures with unclear parameter patterns create debugging difficulties and increase the likelihood of parameter ordering errors during maintenance.

*Related patterns: al-procedure-design.md, al-code-clarity.md*
*Implementation examples: see samples/al-named-parameter-pattern.md*