---
title: "AL Binary Operator Spacing"
domain: "roger-reviewer"
difficulty: "beginner"
bc_versions: "14+"
tags: ["operators", "spacing", "expressions"]
samples: "samples/al-binary-operator-spacing.md"
type: "improvement-pattern"
category: "code-formatting"
pattern_type: "good"
severity: "medium"
impact_level: "medium"
improvement_suggestion: "Use consistent spacing around binary operators to improve expression readability"
implementation_steps: ["standardize-operator-spacing", "apply-consistent-patterns", "validate-readability"]
validation_criteria: ["consistent-operator-spacing", "improved-expression-clarity"]
technical_areas: ["al-code"]
---

# AL Binary Operator Spacing

## Overview

Consistent spacing around binary operators in AL expressions improves readability and helps developers quickly understand mathematical, logical, and comparison operations. Proper operator spacing is a fundamental formatting practice that enhances code clarity.

**Key Principle**: Use consistent spacing around binary operators to create clear, readable expressions that are easy to understand and maintain.

## Strategic Framework

### Expression Clarity
- **Visual Separation**: Clear spacing around operators for easy parsing
- **Readability Focus**: Formatting that supports quick expression comprehension
- **Consistency Standards**: Uniform spacing patterns across all expressions
- **Mathematical Convention**: Spacing that follows standard mathematical notation

### Code Comprehension
Well-spaced operators help developers quickly identify expression structure, operator precedence, and logical relationships within complex calculations and conditions.

## Architecture Focus

### Operator Categories
- **Arithmetic Operators**: +, -, *, /, mod with consistent spacing
- **Comparison Operators**: =, <>, <, >, <=, >= with clear separation
- **Logical Operators**: and, or, not with appropriate spacing
- **Assignment Operators**: := with consistent formatting

### Spacing Patterns
Establish consistent spacing rules that work well with AL syntax and support effective expression formatting across different operator types and complexity levels.

## Best Practices

### Spacing Guidelines
- **Single space before and after** binary operators
- **Consistent application** across all expression types
- **Clear operator separation** in complex expressions
- **Parentheses coordination** with operator spacing
- **Alignment consideration** in multi-line expressions

### Expression Organization
Structure complex expressions with appropriate spacing to maintain readability. Use consistent patterns for operator spacing that integrate well with overall code formatting.

### Readability Optimization
Format expressions to minimize cognitive load when understanding mathematical and logical operations. Use spacing to create natural reading flow in complex calculations.

## Anti-Patterns

### Avoid These Approaches
- **No spacing around operators** creating cramped expressions
- **Inconsistent spacing** within single expressions
- **Excessive spacing** disrupting expression flow
- **Mixed spacing styles** across different AL objects
- **Poor alignment** in multi-line expressions

Poor operator spacing makes expressions harder to read and increases the likelihood of errors when understanding complex calculations and logical conditions.

*Implementation examples: see samples/al-binary-operator-spacing.md*
*Related patterns: al-comment-spacing-standards.md, al-readability-optimization.md*