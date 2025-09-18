---
title: "AL Variable Declaration Order"
domain: "roger-reviewer"
difficulty: "beginner"
bc_versions: "14+"
tags: ["variables", "declaration", "code-structure", "organization"]
type: "improvement-pattern"
category: "code-quality"
pattern_type: "good"
severity: "medium"
impact_level: "medium"
improvement_suggestion: "Organize variable declarations in logical order to improve code readability and maintainability"
implementation_steps: ["establish-declaration-order", "group-related-variables", "apply-consistent-patterns"]
validation_criteria: ["consistent-declaration-order", "improved-organization"]
technical_areas: ["al-code"]
samples: "samples/al-variable-declaration-order.md"
---

# AL Variable Declaration Order

## Overview

Systematic variable declaration ordering in AL improves code readability, maintenance efficiency, and team collaboration. Consistent ordering patterns help developers quickly locate and understand variable purposes within procedures and objects.

**Key Principle**: Group related variables together and order by importance, scope, and usage patterns to create predictable code structure.

## Strategic Framework

### Standard Declaration Sequence
1. **Input Parameters**: Method inputs in logical order
2. **Output Parameters**: Return values and output references
3. **Record Variables**: Primary business objects
4. **Temporary Records**: Buffer and temporary storage
5. **Management Objects**: Codeunits, pages, reports
6. **Simple Data Types**: Text, Integer, Decimal, Boolean
7. **System Objects**: Dialog, File, Stream variables

### Grouping Principles
- **Related Functionality**: Group variables serving similar purposes
- **Usage Frequency**: Most-used variables declared first within groups
- **Data Flow**: Input → Processing → Output variable ordering
- **Alphabetical Within Groups**: When no logical order exists, use alphabetical sorting

## Architecture Focus

### Scope-Based Organization
**Global Variables** follow object-level importance - core business records first, followed by supporting objects and simple types.

**Local Variables** prioritize procedure-specific logic flow - parameters first, then main processing variables, finally utility variables.

**Temporary Variables** grouped separately to clearly indicate their transient nature and memory management requirements.

### Type-Specific Patterns
- **Record Variables**: Order by relationship hierarchy (master records before detail records)
- **Management Variables**: Group by functional area (sales management, posting management)
- **Configuration Variables**: Place setup and configuration records after main business records
- **System Variables**: Group system objects (Dialog, File) at end of declarations

## Best Practices

### Logical Grouping
- **Business Context First**: Primary business objects at top of declarations
- **Supporting Objects Next**: Helper records and management objects follow
- **Utility Variables Last**: Counters, flags, and temporary values at end
- **Related Variables Together**: Keep functionally related variables adjacent

### Readability Enhancement
- **Blank Line Separation**: Use blank lines between logical groups
- **Inline Comments**: Document variable groups and their collective purpose
- **Consistent Spacing**: Align variable names and types for visual clarity
- **Descriptive Ordering**: Within groups, order by business logic flow

### Maintenance Considerations
- **Standard Patterns**: Follow consistent ordering across all team code
- **Refactoring Support**: Logical ordering simplifies variable impact analysis
- **Documentation**: Variable order should tell the story of the procedure's logic
- **Review Guidelines**: Include declaration order in code review checklists

## Anti-Patterns

### Avoid These Approaches
- **Random Ordering**: Variables declared in order of coding rather than logical importance
- **Mixed Types**: Simple types scattered between complex objects
- **No Grouping**: Related variables separated by unrelated declarations
- **Inconsistent Patterns**: Different ordering approaches within same codebase

Poorly ordered declarations make code changes more error-prone and time-consuming, with variable relationships becoming unclear and increasing maintenance risk.

*Related patterns: al-variable-naming-conventions.md, al-code-organization.md*
*Implementation examples: see samples/al-variable-declaration-order.md*