---
title: "AL Line Comment Formatting and Placement Patterns"
domain: "code-quality"
difficulty: "beginner"
bc_versions: "14+"
tags: ["comments", "code-quality", "readability", "maintainability"]
prerequisites: ["al-basics"]
---

# AL Line Comment Formatting and Placement Patterns

## Overview

Line comments in AL code provide contextual explanation and improve code readability when properly formatted and strategically placed. Effective commenting enhances maintainability without cluttering the codebase.

**Key Principle**: Comments should explain *why* code exists, not *what* it does. Focus on business logic, complex calculations, and non-obvious implementation decisions.

## Strategic Framework

### Comment Placement Strategy
- **Above code blocks**: Explain purpose before implementation
- **Inline with complex logic**: Clarify non-obvious calculations or conditions
- **End-of-line for variables**: Document units, ranges, or special meanings
- **Section headers**: Group related functionality within procedures

### Content Guidelines
- **Business context**: Explain why specific logic is needed
- **Edge cases**: Document special conditions or exceptions
- **Performance notes**: Explain optimization decisions
- **Integration points**: Clarify external system interactions

## Architecture Patterns

### Procedure-Level Comments
Place explanatory comments immediately above procedure declarations to describe purpose, key parameters, and expected outcomes. Use consistent formatting with proper spacing.

### Variable Documentation
Document complex variables with end-of-line comments explaining units, valid ranges, or business significance. This is especially important for calculated fields and configuration values.

### Logic Block Comments
Group related logic with section comments that explain the overall approach. This helps readers understand the flow without parsing individual statements.

## Best Practices

### Formatting Standards
- **Consistent spacing**: Single space after // marker
- **Proper alignment**: Align related inline comments vertically
- **Clear language**: Use complete sentences for complex explanations
- **Standard terminology**: Use consistent BC terminology throughout

### Strategic Commenting
- **Focus on complexity**: Comment complex business rules and calculations
- **Avoid obvious statements**: Don't comment self-evident code
- **Update with changes**: Maintain comment accuracy during code updates
- **Group related items**: Use section comments to organize related functionality

### Maintenance Approach
Keep comments current with code changes. Outdated comments are worse than no comments as they mislead future developers and create maintenance debt.

## Anti-Patterns

### Avoid These Approaches
- **Commenting obvious code**: Don't explain what simple assignments do
- **Excessive commenting**: Every line doesn't need explanation
- **Outdated information**: Comments that no longer match the code
- **Poor grammar/spelling**: Unprofessional comments reduce credibility
- **Code explanation**: Explaining *what* code does instead of *why*

