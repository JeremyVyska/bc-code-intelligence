---
title: "AL Statement Per Line Rule"
domain: "roger-reviewer"
difficulty: "beginner"
bc_versions: "14+"
tags: ["statements", "line-organization", "readability"]
samples: "samples/al-statement-per-line-rule.md"
type: "improvement-pattern"
category: "code-formatting"
pattern_type: "good"
severity: "medium"
impact_level: "medium"
improvement_suggestion: "Write one AL statement per line to improve readability and debugging capability"
implementation_steps: ["separate-multiple-statements", "format-single-statements", "validate-clarity"]
validation_criteria: ["one-statement-per-line", "improved-readability"]
technical_areas: ["al-code"]
---

# AL Statement Per Line Rule

## Overview

Writing one AL statement per line improves code readability, simplifies debugging, and makes code maintenance more effective. The one-statement-per-line rule is a fundamental formatting practice that enhances code clarity and supports better development workflows.

**Key Principle**: Each line should contain exactly one complete AL statement to maximize readability and debugging effectiveness.

## Strategic Framework

### Readability Enhancement
- **Visual Clarity**: Each statement clearly separated and identifiable
- **Scanning Efficiency**: Easy visual parsing of code logic
- **Debugging Support**: Line-by-line debugging capability
- **Code Review**: Simplified review process with clear statement boundaries

### Maintenance Benefits
Single statements per line make code changes easier to track, debug, and understand. This pattern supports effective version control and collaborative development practices.

## Architecture Focus

### Statement Organization
- **Clear Boundaries**: Each statement on separate line with proper termination
- **Logical Flow**: Sequential statement arrangement for easy following
- **Visual Structure**: Line organization that reflects code logic
- **Error Isolation**: Individual statements for easier error identification

### Development Workflow
One-statement-per-line formatting integrates well with debugging tools, version control systems, and code review processes commonly used in AL development.

## Best Practices

### Statement Formatting
- **Single statement per line** with proper semicolon termination
- **Appropriate indentation** for each statement level
- **Clear statement separation** without cramming multiple operations
- **Logical ordering** of related statements
- **Consistent formatting** across all AL objects

### Code Organization
Organize statements in logical sequence, group related operations appropriately, and use consistent formatting patterns that support easy code comprehension.

### Debugging Support
Structure code to support effective debugging by ensuring each line represents a single operation that can be stepped through individually.

## Anti-Patterns

### Avoid These Approaches
- **Multiple statements per line** reducing readability
- **Cramped code formatting** without proper separation
- **Complex compound statements** difficult to debug
- **Inconsistent line organization** across AL objects
- **Poor statement grouping** without logical organization

Multiple statements per line make code harder to read, debug, and maintain. This pattern increases cognitive load and reduces development efficiency.

*Implementation examples: see samples/al-statement-per-line-rule.md*
*Related patterns: al-readability-optimization.md, al-blank-line-organization.md*