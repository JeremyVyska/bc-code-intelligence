---
title: "AL End Else Pairing"
domain: "roger-reviewer"
difficulty: "intermediate"
bc_versions: "14+"
tags: ["end-statements", "else-clauses", "block-structure"]
type: "improvement-pattern"
category: "language-fundamentals"
pattern_type: "good"
severity: "medium"
impact_level: "medium"
improvement_suggestion: "Properly pair end statements with else clauses to maintain clear block structure"
implementation_steps: ["identify-block-boundaries", "align-end-else-pairs", "validate-structure"]
validation_criteria: ["correct-end-else-pairing", "clear-block-boundaries"]
technical_areas: ["al-code"]
---

# AL End Else Pairing

## Overview

Proper pairing of end statements with else clauses in AL code maintains clear block structure and prevents syntax errors. Understanding end-else relationships is crucial for writing maintainable conditional logic and avoiding common AL language pitfalls.

**Key Principle**: Each end statement must properly close its corresponding block before any related else clause can be processed.

## Strategic Framework

### Block Structure Integrity
- **Scope Management**: Clear understanding of block boundaries and closure
- **Syntax Correctness**: Proper AL language structure compliance
- **Error Prevention**: Avoiding common end-else pairing mistakes
- **Code Clarity**: Structure that makes block relationships obvious

### Language Compliance
AL language requires specific patterns for end-else pairing that differ from some other programming languages. Understanding these patterns prevents compilation errors and logic mistakes.

## Architecture Focus

### Pairing Patterns
- **Conditional Blocks**: Proper end placement before else statements
- **Nested Structures**: Correct pairing in complex conditional hierarchies
- **Block Boundaries**: Clear indication of scope closure points
- **Statement Organization**: Logical grouping of related conditional elements

### Syntax Requirements
AL syntax requires end statements to properly close conditional blocks before else clauses can be evaluated. This pattern ensures clear logical flow and prevents ambiguous code interpretation.

## Best Practices

### Correct Pairing Techniques
- **Close blocks completely** before else statements
- **Align end-else pairs** for visual clarity
- **Use consistent indentation** to show block relationships
- **Validate block closure** before adding else clauses
- **Structure complex conditionals** with clear pairing patterns

### Error Prevention
Understand AL language requirements for end-else pairing to avoid compilation errors. Use proper formatting to make block relationships immediately apparent to code reviewers.

### Code Organization
Structure conditional logic with clear end-else pairing that supports easy understanding of decision flow and block boundaries.

## Anti-Patterns

### Avoid These Approaches
- **Missing end statements** before else clauses
- **Improper block closure** creating syntax errors
- **Ambiguous pairing** in nested conditional structures
- **Poor indentation** obscuring block relationships
- **Complex nesting** without clear pairing patterns

Incorrect end-else pairing creates compilation errors and makes conditional logic difficult to understand and maintain.

*Related patterns: al-separate-if-else-formatting.md, al-keyword-indentation-rules.md*
*See also: al-separate-if-else-formatting.md samples for practical examples*