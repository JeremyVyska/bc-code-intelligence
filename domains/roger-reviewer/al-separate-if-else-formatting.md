---
title: "AL Separate If Else Formatting"
domain: "roger-reviewer"
difficulty: "intermediate"
bc_versions: "14+"
tags: ["if-statements", "conditional-logic", "formatting"]
samples: "samples/al-separate-if-else-formatting.md"
type: "improvement-pattern"
category: "code-formatting"
pattern_type: "good"
severity: "medium"
impact_level: "medium"
improvement_suggestion: "Format if-else statements with clear separation to show conditional logic structure"
implementation_steps: ["separate-conditional-blocks", "align-if-else-keywords", "validate-readability"]
validation_criteria: ["clear-conditional-separation", "consistent-alignment"]
technical_areas: ["al-code"]
---

# AL Separate If Else Formatting

## Overview

Proper formatting of if-else statements in AL code with clear separation between conditional branches improves readability and helps developers understand decision logic structure. Separate formatting makes complex conditional logic easier to follow and maintain.

**Key Principle**: Format if-else statements with clear visual separation between conditional branches to highlight decision structure.

## Strategic Framework

### Conditional Logic Clarity
- **Branch Separation**: Clear visual distinction between if and else branches
- **Decision Structure**: Formatting that reveals conditional logic organization
- **Readability Focus**: Layout that supports quick comprehension of decision flow
- **Consistency Standards**: Uniform if-else formatting across codebase

### Control Flow Visualization
If-else formatting should make decision paths immediately apparent to code reviewers. Clear separation helps developers understand alternative execution paths and their relationships.

## Architecture Focus

### Formatting Patterns
- **Keyword Alignment**: Consistent positioning of if and else keywords
- **Branch Separation**: Visual spacing between conditional branches
- **Block Structure**: Clear indication of conditional block boundaries
- **Nested Conditionals**: Proper formatting for complex decision trees

### Visual Organization
Structure if-else statements to support quick visual parsing of conditional logic. Use consistent formatting patterns that work well with AL language syntax.

## Best Practices

### Separation Techniques
- **Clear keyword positioning** for if and else statements
- **Appropriate spacing** between conditional branches
- **Consistent indentation** within conditional blocks
- **Visual boundaries** around conditional logic sections
- **Aligned block structures** for easy scanning

### Readability Optimization
Format if-else statements to minimize cognitive load when understanding conditional logic. Use whitespace and alignment effectively to create clear decision structure.

### Complex Conditionals
For nested or complex if-else structures, use additional spacing and indentation to maintain clarity. Consider refactoring overly complex conditional logic into separate methods.

## Anti-Patterns

### Avoid These Approaches
- **Cramped if-else formatting** without adequate separation
- **Inconsistent keyword alignment** obscuring structure
- **Poor indentation** in conditional blocks
- **Unclear branch boundaries** making logic hard to follow
- **Overly complex nested conditionals** without proper formatting

Poor if-else formatting makes conditional logic harder to understand and increases the likelihood of errors during code maintenance.

*Implementation examples: see samples/al-separate-if-else-formatting.md*
*Related patterns: al-end-else-pairing.md, al-keyword-indentation-rules.md*